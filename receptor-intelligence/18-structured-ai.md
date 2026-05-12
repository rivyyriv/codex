# 18 — Structured AI Integration

How the platform uses Claude (Anthropic API) and Whisper (OpenAI) for the AI scribe, intake assistance, and patient-facing summary generation. Concrete prompt templates, JSON schemas for structured outputs, and validation patterns.

This file makes the AI extraction service spec from `07-ai-extraction-spec.md` concrete with prompts and code, and operationalizes the scribe pattern as a v1 feature.

---

## Where AI is used in v1

Three patterns, all in v1:

| Pattern | Stakes | Phase |
|---------|--------|-------|
| AI scribe — transcript → proposed modifiers, clinician-reviewed | Medium (proposes data writes; queue with explicit approval) | Phase 5 |
| Layer-3 patient-facing summary generation | Low (read-only, post-visit, no PHI in prompts) | Phase 6 |
| Intake follow-up question suggestions | Low (suggests next questionnaire item) | Phase 6, optional |

Deferred to v2:
- Real-time scribe transcript display during visit (v1 is post-visit batch processing).
- Multi-speaker diarization.
- Differential diagnosis hypothesis surfacing (SaMD territory regardless).
- Embedding-based registry search.

## Core principles

These hold for every AI call in the system:

**Structured outputs only.** Every Claude call uses tool use to produce a typed JSON object. No free-text outputs that downstream code has to parse with regex. Tool definitions enforce shape.

**Validation always.** Every AI output is validated against a zod schema before any downstream action. If validation fails, the proposal is rejected and logged, not used.

**Evidence required.** Every modifier proposal includes the verbatim source text from the transcript or note that justified it. No "AI thinks the patient is anxious" without a quote.

**AI proposes, clinician commits.** Per `07-ai-extraction-spec.md`. Proposals land in a queue. No write to `phi.cell_modifiers` or `phi.subsystem_modifiers` without explicit human approval action.

**Audit trail mandatory.** Every Claude call (request and response) is logged to `phi.ai_call_log` with the prompt, the response, the model version, the temperature, and the resulting clinician action.

**Token budget discipline.** Each call has an explicit max-tokens budget. Long inputs get summarized first. Prompts are small (<2K tokens) where possible.

## Anthropic SDK setup

```typescript
// src/lib/ai/client.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';
import { env } from '../../env';

export const anthropic = new Anthropic({
  apiKey: env.ANTHROPIC_API_KEY,
});

export const openai = new OpenAI({
  apiKey: env.OPENAI_API_KEY,
  // HIPAA-eligible enterprise tier with BAA executed
});

export const MODEL_VERSION = 'claude-sonnet-4-5-20250929';
export const FALLBACK_MODEL = 'claude-haiku-4-5-20251001';
export const TRANSCRIPTION_MODEL = 'whisper-1';

export const DEFAULTS = {
  max_tokens: 2048,
  temperature: 0,
};
```

Pin specific model versions for reproducibility. When new models are released, evaluate before upgrading.

---

## Pattern 1: AI scribe (the centerpiece)

The scribe captures conversation audio during a visit, transcribes it, extracts proposed clinical modifiers, and surfaces them in a review queue for the clinician.

### End-to-end pipeline

```
Browser MediaRecorder
  ↓ (chunked audio uploads)
Backend audio receiver
  ↓ (encrypt + store)
phi.encounter_transcripts
  ↓ (on encounter end)
Whisper transcription job
  ↓ (full transcript)
Claude extraction job
  ↓ (proposed modifiers with evidence quotes)
phi.modifier_proposals (queue)
  ↓ (frontend polls or websocket)
Provider review UI
  ↓ (per-proposal: approve / edit / reject)
phi.subsystem_modifiers / phi.cell_modifiers (writes)
  ↓ (triggers state recomputation)
Updated brain map
```

Each arrow is an audit-logged step. The whole pipeline is replayable: given a transcript and registry version, the same proposals come back.

### Stage 1: audio capture (frontend)

```typescript
// Simplified sketch for the visit-mode component
const [recorder, setRecorder] = useState<MediaRecorder | null>(null);
const [audioChunks, setAudioChunks] = useState<Blob[]>([]);

async function startScribe(encounterId: string) {
  const stream = await navigator.mediaDevices.getUserMedia({ audio: true });
  const rec = new MediaRecorder(stream, { mimeType: 'audio/webm' });
  
  rec.ondataavailable = async (e) => {
    if (e.data.size > 0) {
      // Upload chunk immediately for resilience
      await uploadAudioChunk(encounterId, e.data, audioChunks.length);
      setAudioChunks(prev => [...prev, e.data]);
    }
  };
  
  rec.start(30_000);  // 30-second chunks
  setRecorder(rec);
}

async function endScribe() {
  recorder?.stop();
  // Backend handles concatenation, transcription, extraction
  await api.post('/encounters/end', { encounter_id });
}
```

Notes:
- Chunked uploads survive network blips. If upload fails on chunk 5, chunk 6 still goes.
- Audio is captured at the browser; on iOS Safari, MediaRecorder support requires polyfill or `MediaRecorder.isTypeSupported('audio/mp4')` fallback.
- Visual indicator while recording (red dot, timer). Easy stop button always visible.

### Stage 2: transcription

```typescript
// src/jobs/transcribeEncounter.ts
import { openai, TRANSCRIPTION_MODEL } from '../lib/ai/client';

export async function transcribeEncounterJob({ encounterId }: { encounterId: string }) {
  // Fetch all audio chunks, concatenate
  const audioBuffer = await assembleAudio(encounterId);
  
  const transcript = await openai.audio.transcriptions.create({
    file: audioBuffer,
    model: TRANSCRIPTION_MODEL,
    language: 'en',
    response_format: 'verbose_json',  // Includes timestamps
    timestamp_granularities: ['segment'],
  });
  
  await db.insert(encounter_transcripts).values({
    encounter_id: encounterId,
    transcript_text: transcript.text,
    transcript_segments: transcript.segments,
    transcribed_at: new Date(),
    transcription_model: TRANSCRIPTION_MODEL,
    duration_seconds: transcript.duration,
  });
  
  // Enqueue extraction job
  await pgBoss.send('extract_modifiers', { encounterId });
}
```

The transcript is encrypted at rest (Supabase column-level encryption or app-level via libsodium).

### Stage 3: extraction

```typescript
// src/jobs/extractModifiers.ts
import { extractModifiers } from '../lib/ai/extractModifiers';

export async function extractModifiersJob({ encounterId }: { encounterId: string }) {
  const transcript = await db.select().from(encounter_transcripts)
    .where(eq(encounter_transcripts.encounter_id, encounterId)).limit(1);
  
  const encounter = await db.select().from(encounters)
    .where(eq(encounters.id, encounterId)).limit(1);
  
  const patient = await db.select().from(patients)
    .where(eq(patients.id, encounter[0].patient_id)).limit(1);
  
  const patientContext = await buildPatientContext(patient[0].id);
  
  const proposalEntry = await extractModifiers(
    transcript[0].transcript_text,
    patientContext,
    { encounterId, patientId: patient[0].id }
  );
  
  // Write each proposal to the queue
  for (const proposal of proposalEntry.proposed_modifiers) {
    await db.insert(modifier_proposals).values({
      encounter_id: encounterId,
      patient_id: patient[0].id,
      ...proposal,
      review_status: 'pending',
      created_at: new Date(),
    });
  }
  
  // Notify the frontend (via realtime or polling)
  await notifyProvider(encounter[0].clinician_id, 'proposals_ready', { encounterId });
}
```

### Tool definition for extraction

```typescript
const extractModifiersTool = {
  name: 'extract_clinical_modifiers',
  description: 'Extract proposed cell or subsystem modifiers from clinical observations in a transcript or note.',
  input_schema: {
    type: 'object',
    properties: {
      proposed_subsystem_modifiers: {
        type: 'array',
        items: {
          type: 'object',
          properties: {
            subsystem: {
              type: 'string',
              enum: ['D', 'F', 'H', 'T', 'mood-low', 'mood-high', 'anxiety', 'arousal'],
              description: 'Subsystem code',
            },
            delta_direction: {
              type: 'string',
              enum: ['increased', 'decreased', 'no-change'],
            },
            magnitude: {
              type: 'string',
              enum: ['mild', 'moderate', 'severe'],
            },
            evidence_quote: {
              type: 'string',
              description: 'Verbatim quote from the source supporting this modifier (max 200 chars)',
            },
            confidence: { type: 'string', enum: ['H', 'M', 'L'] },
          },
          required: ['subsystem', 'delta_direction', 'magnitude', 'evidence_quote', 'confidence'],
        },
      },
      proposed_cell_modifiers: {
        type: 'array',
        items: {
          type: 'object',
          properties: {
            cell_key_hint: { type: 'string' },
            delta_direction: { type: 'string', enum: ['increased', 'decreased'] },
            magnitude: { type: 'string', enum: ['mild', 'moderate', 'severe'] },
            evidence_quote: { type: 'string' },
            confidence: { type: 'string', enum: ['H', 'M', 'L'] },
          },
          required: ['cell_key_hint', 'delta_direction', 'magnitude', 'evidence_quote', 'confidence'],
        },
      },
      no_extraction_made: {
        type: 'boolean',
        description: 'True if the source text contains no clinically relevant content for extraction',
      },
      reasoning: {
        type: 'string',
        description: 'Brief explanation of the extraction approach (max 500 chars)',
      },
    },
    required: ['proposed_subsystem_modifiers', 'proposed_cell_modifiers', 'no_extraction_made', 'reasoning'],
  },
};
```

### Prompt template

```typescript
const EXTRACTION_SYSTEM_PROMPT = `You extract structured clinical observations from a psychiatric visit transcript or note.

Your job: identify observations that map to specific subsystems or receptor-level cells in our framework.

Subsystem reference (OCD context):
- D (Doubt): obsessional doubt, checking, mental rituals, "what if" thinking
- F (Fear): contamination fears, harm avoidance, panic
- H (Habit): compulsive behaviors, rigid routines
- T (Tic-spectrum): motor tics, vocalizations, urge-driven movements

Subsystem reference (mood/anxiety context):
- mood-low: anhedonia, low mood, hopelessness, suicidal ideation
- mood-high: elevated mood, grandiosity, decreased sleep need
- anxiety: generalized worry, somatic anxiety, panic
- arousal: hypervigilance, sleep disturbance, irritability

Magnitude calibration:
- mild: occasional, low impact on function
- moderate: frequent, noticeable impact
- severe: pervasive, marked functional impairment

Confidence calibration:
- H: explicit, unambiguous statement
- M: clearly implied, multiple corroborating mentions
- L: ambiguous, single mention, possible alternative interpretations

Critical rules:
1. Every proposed modifier must include a verbatim quote from the source as evidence.
2. If the source contains no clinical observations, set no_extraction_made: true and return empty arrays.
3. Do not infer beyond what's stated. If the patient says "I'm tired," do not infer "mood-low" without supporting context.
4. Cell-level proposals are uncommon in conversation; most observations are subsystem-level. Only propose cell modifiers when the source is unusually specific (e.g., "PET scan showed 5HT2A reduction in OFC").
5. Be conservative. False positives are worse than false negatives because every proposal costs the clinician review time.

Use the extract_clinical_modifiers tool to return your output.`;

const EXTRACTION_USER_PROMPT = (note: string, patientContext: PatientContext) => `Patient context:
- Active disorder template(s): ${patientContext.activeTemplates.join(', ')}
- Severity: ${patientContext.severityBucket}
- Active regimen: ${patientContext.activeRegimen.map(a => `${a.agent_id} ${a.dose}`).join(', ') || 'none'}

Source (visit transcript or clinical note):
"""
${note}
"""

Extract proposed modifiers based on observations in the source. Be conservative.`;
```

### Calling code with validation

```typescript
const extractionOutputSchema = z.object({
  proposed_subsystem_modifiers: z.array(z.object({
    subsystem: z.enum(['D', 'F', 'H', 'T', 'mood-low', 'mood-high', 'anxiety', 'arousal']),
    delta_direction: z.enum(['increased', 'decreased', 'no-change']),
    magnitude: z.enum(['mild', 'moderate', 'severe']),
    evidence_quote: z.string().max(200),
    confidence: z.enum(['H', 'M', 'L']),
  })),
  proposed_cell_modifiers: z.array(z.object({
    cell_key_hint: z.string(),
    delta_direction: z.enum(['increased', 'decreased']),
    magnitude: z.enum(['mild', 'moderate', 'severe']),
    evidence_quote: z.string().max(200),
    confidence: z.enum(['H', 'M', 'L']),
  })),
  no_extraction_made: z.boolean(),
  reasoning: z.string().max(500),
});

export async function extractModifiers(
  note: string,
  patientContext: PatientContext,
  callContext: CallContext
): Promise<ProposalQueueEntry> {
  const startedAt = new Date();
  
  const response = await anthropic.messages.create({
    model: MODEL_VERSION,
    max_tokens: 2048,
    temperature: 0,
    system: EXTRACTION_SYSTEM_PROMPT,
    messages: [{ role: 'user', content: EXTRACTION_USER_PROMPT(note, patientContext) }],
    tools: [extractModifiersTool],
    tool_choice: { type: 'tool', name: 'extract_clinical_modifiers' },
  });
  
  const toolUseBlock = response.content.find(b => b.type === 'tool_use');
  if (!toolUseBlock || toolUseBlock.type !== 'tool_use') {
    throw new AIValidationError('No tool use in response');
  }
  
  const parsed = extractionOutputSchema.safeParse(toolUseBlock.input);
  if (!parsed.success) {
    throw new AIValidationError('Invalid extraction shape', parsed.error);
  }
  
  // Anti-hallucination: every evidence_quote must be a substring of the source
  const noteText = note.toLowerCase();
  for (const mod of [...parsed.data.proposed_subsystem_modifiers, ...parsed.data.proposed_cell_modifiers]) {
    if (!noteText.includes(mod.evidence_quote.toLowerCase())) {
      throw new AIValidationError(`Evidence quote "${mod.evidence_quote}" not found in source`);
    }
  }
  
  const proposalEntry: ProposalQueueEntry = {
    source_note: note,
    extracted_at: startedAt,
    completed_at: new Date(),
    model_version: MODEL_VERSION,
    extraction_reasoning: parsed.data.reasoning,
    proposed_modifiers: [
      ...parsed.data.proposed_subsystem_modifiers.map(m => ({
        type: 'subsystem' as const,
        subsystem: m.subsystem,
        delta: magnitudeToDelta(m.magnitude, m.delta_direction),
        confidence: m.confidence,
        evidence_quote: m.evidence_quote,
      })),
      ...parsed.data.proposed_cell_modifiers.map(m => ({
        type: 'cell' as const,
        cell_key_hint: m.cell_key_hint,
        delta: magnitudeToDelta(m.magnitude, m.delta_direction),
        confidence: m.confidence,
        evidence_quote: m.evidence_quote,
      })),
    ],
    review_status: 'pending',
  };
  
  await logAICall({
    purpose: 'modifier_extraction',
    model: MODEL_VERSION,
    started_at: startedAt,
    input: { note, patientContext },
    raw_response: response,
    parsed_output: parsed.data,
    context: callContext,
  });
  
  return proposalEntry;
}

function magnitudeToDelta(mag: 'mild' | 'moderate' | 'severe', dir: string): number {
  const m = { mild: 0.5, moderate: 1.0, severe: 1.5 }[mag];
  return dir === 'decreased' ? -m : (dir === 'no-change' ? 0 : m);
}
```

### The substring guard

The substring check on `evidence_quote` is critical. Without it, Claude can hallucinate quotes that sound plausible but didn't come from the transcript. The check forces every proposed modifier to be backed by text actually in the source. Anti-hallucination defense, mandatory.

In production, the match is slightly fuzzy to allow for whitespace and minor punctuation differences, but the principle stays. Hallucinated proposals are validation failures, never reach the queue, and surface as a generic error to the user.

### Provider review UI

The proposal queue surfaces at end of visit. For each proposal:

- **Subsystem or cell name** (with mechanism context: "5HT raphe → cortical input")
- **Direction and magnitude** (e.g., "increased, moderate")
- **Evidence quote highlighted in transcript context** — the surrounding 1-2 sentences from the transcript with the quote bolded
- **Confidence indicator**
- **Three actions**:
  - Approve → writes modifier, brain map updates live
  - Edit → adjust subsystem, magnitude, or evidence before approving
  - Reject → discarded, optional reason logged

The transcript itself is also visible (collapsible side panel) so the provider can verify the quote in full context.

---

## Pattern 2: Patient-facing summary (v1)

The Layer-3 patient-facing summary generated post-visit. Plain language, never names specific drugs, never states a diagnosis with a confidence number.

### Tool definition

```typescript
const patientSummaryTool = {
  name: 'generate_patient_summary',
  description: 'Generate a patient-facing summary of their current brain-map state and treatment trajectory.',
  input_schema: {
    type: 'object',
    properties: {
      summary_paragraphs: {
        type: 'array',
        items: { type: 'string' },
        description: '3-5 paragraphs in plain language at 8th-grade reading level',
      },
      key_points: {
        type: 'array',
        items: { type: 'string' },
        description: '3-5 short bullet-style highlights for the patient',
      },
      questions_to_ask_provider: {
        type: 'array',
        items: { type: 'string' },
        description: 'Suggested questions the patient might raise at next visit',
      },
    },
    required: ['summary_paragraphs', 'key_points', 'questions_to_ask_provider'],
  },
};
```

### Prompt

```typescript
const SUMMARY_SYSTEM_PROMPT = `You generate patient-facing summaries explaining brain-mapping assessment results in plain language.

Constraints (non-negotiable):
- Do not name specific medications.
- Do not give numeric diagnostic confidences (e.g. "75% likely to be MDD").
- Do not say "you have X disorder."
- Do not give specific medical advice or recommend specific actions.
- Use plain language at 8th-grade reading level.
- Use second person ("your brain shows...") respectfully and accurately.
- Frame findings as "patterns observed" not "diagnoses."
- Acknowledge uncertainty where the data is uncertain.

Use the generate_patient_summary tool to return your output.`;
```

### Forbidden-term defense

```typescript
const FORBIDDEN_TERMS = [
  'sertraline', 'zoloft', 'fluoxetine', 'prozac', 'escitalopram', 'lexapro',
  'venlafaxine', 'effexor', 'duloxetine', 'cymbalta', 'bupropion', 'wellbutrin',
  'aripiprazole', 'abilify', 'risperidone', 'risperdal', 'olanzapine', 'zyprexa',
  '% likely', '% probability', 'definitive diagnosis',
  'you should take', 'you must take', 'recommended dose',
];

function containsForbiddenTerms(text: string): boolean {
  return FORBIDDEN_TERMS.some(term => text.toLowerCase().includes(term));
}
```

Three layers of defense: system prompt forbids it, tool description forbids it, validator rechecks. Patient-facing surface is the highest-stakes UI; belt-and-suspenders.

---

## Pattern 3: Intake assistant (optional, v1 Phase 6)

After the patient submits a questionnaire, suggest a follow-up instrument or specific clinician questions based on the response pattern. Lower stakes than the scribe; the suggestion goes to the clinician dashboard, not directly into the patient record.

```typescript
const intakeAssistantTool = {
  name: 'suggest_follow_up_intake',
  input_schema: {
    type: 'object',
    properties: {
      suggested_instruments: {
        type: 'array',
        items: { 
          type: 'string',
          enum: ['ybocs', 'phq9', 'gad7', 'madrs', 'asrs', 'pcl5', 'ygtss', 'oci-r', 'docs'],
        },
      },
      reasoning: { type: 'string' },
      provider_questions: {
        type: 'array',
        items: { type: 'string' },
      },
    },
    required: ['suggested_instruments', 'reasoning'],
  },
};
```

Standard call pattern from there. Implement in Phase 6 if time allows; otherwise defer.

---

## AI call logging

Every AI call writes to `phi.ai_call_log`:

```sql
CREATE TABLE phi.ai_call_log (
  id bigserial PRIMARY KEY,
  occurred_at timestamptz NOT NULL DEFAULT now(),
  patient_id uuid REFERENCES phi.patients(id),
  encounter_id uuid REFERENCES phi.encounters(id),
  purpose text NOT NULL,
  model_version text NOT NULL,
  temperature numeric NOT NULL,
  input_tokens int,
  output_tokens int,
  prompt_inputs jsonb,
  raw_response jsonb,
  parsed_output jsonb,
  validation_errors jsonb,
  resulting_action text,
  resulting_action_at timestamptz,
  resulting_actor_id uuid REFERENCES phi.clinicians(id)
);
```

This log makes the scribe pipeline reproducible. Given an encounter, you can replay exactly what the AI proposed and what the clinician did with it.

## Cost management

Token and API cost per visit at v1 scale:

| Call | Tokens / size | Cost |
|------|---------------|------|
| Whisper transcription (30 min visit) | ~30 min audio | ~$0.18 |
| Modifier extraction | ~3000 input + ~1000 output | ~$0.03 |
| Patient summary | ~1500 input + ~1000 output | ~$0.02 |
| Total per visit | | ~$0.23 |

At v1 scale (1-3 providers, ~50-150 visits/month), AI cost is <$50/month.

## Failure handling

- **Whisper timeout / failure**: retry once, then surface "transcription failed, please re-record or enter notes manually." Encounter still completes; modifiers can be added manually.
- **Claude validation failure**: log to `ai_call_log.validation_errors`; surface "AI extraction unavailable for this visit" — provider proceeds with manual modifier entry.
- **Hallucinated evidence quote**: caught by substring guard, treated as validation failure.
- **API rate limit**: exponential backoff. If sustained, surface "service temporarily degraded."

The platform must continue to function without AI. AI is augmentation, not foundation.

## What's not specified here

Real-time streaming responses (low-latency UX during visit), multi-turn conversational extraction, fine-tuning, embedding-based registry search, multi-speaker diarization. All v2+ considerations.

## Cross-references

- `07-ai-extraction-spec.md` — methodology spec
- `15-schema-extensions.md` — `phi.ai_call_log`, `phi.modifier_proposals`, `phi.encounter_transcripts`
- `17-backend-stack.md` — `src/lib/ai/` and `src/jobs/` directories
- `19-v1-roadmap.md` Phase 5 — implementation timeline
