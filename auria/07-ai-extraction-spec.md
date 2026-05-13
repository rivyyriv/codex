# 07 — AI Extraction Service — Engineering Spec

The pipeline that turns clinician free-form notes into ProposedModifier records for clinician review.

This is a v2 capability. v1 ships without AI extraction — questionnaire-driven subsystem modifiers and clinician-direct cell modifiers cover the intake path. AI extraction is a force multiplier for v2.

---

## Scope

**What this covers:** end-to-end pipeline from clinician note ingestion through clinician review to committed PatientCellModifier or PatientSubsystemModifier records. Architecture, data contracts, prompt design pattern, review queue mechanics, audit trail.

**What's deferred:** specific LLM provider selection, prompt-tuning iterations on real clinical notes, the review-queue UI styling. Those are downstream engineering decisions; the contracts here are stable across them.

**Audience:** the engineer building the AI extraction service.

## Hard rule — the only rule that matters

**AI proposes. Clinician commits.**

The AI never writes to the cell registry. Every AI-extracted modifier sits in a proposal queue until a human reviews it.

This rule is non-negotiable. The framework's clinical claim depends on every cell-level data point being human-affirmed. AI as a drafting tool: yes. AI as authoring authority: no.

## Pipeline flow

```
Clinician note (text input)
  ↓
LLM extraction with structured output
  ↓
ProposedModifier records (queued, status: 'pending_review')
  ↓
Clinician review UI
  ↓
Decision: accept | reject | edit
  ↓
If accept: write PatientCellModifier or PatientSubsystemModifier to registry
           (with source: 'ai_extracted_clinician_review')
If reject: mark proposal status: 'rejected', retain for audit
If edit:   write modified version, retain original proposal for audit
```

Every step is logged. The audit trail preserves: original note text, full LLM prompt, full LLM response, proposed modifier, clinician decision, final committed modifier (if any), timestamp, clinician identity.

## Data contracts

### ProposedModifier

```typescript
interface ProposedModifier {
  id: string;                              // UUID
  schema_version: "3.0";
  patient_id: string;
  profile_id: string;

  proposal_type: "subsystem" | "cell";
  proposal_payload: ProposedSubsystemModifier | ProposedCellModifier;

  source_note: SourceNote;
  source_sentences: string[];               // sentences from the note that justify this proposal
  ai_confidence: number;                    // 0-1, model's self-reported confidence
  ai_reasoning: string;                     // why this proposal (model output)

  llm_metadata: {
    model: string;                          // e.g. "claude-sonnet-4-20250514"
    prompt_version: string;
    request_id: string;
    timestamp: string;
  };

  status: "pending_review" | "accepted" | "rejected" | "edited";
  reviewed_by: string | null;
  reviewed_at: string | null;
  review_decision: ReviewDecision | null;

  committed_modifier_id: string | null;     // FK to actual PatientModifier if accepted
}

interface ProposedSubsystemModifier {
  template_ref: string;
  subsystem: string;
  delta_modifier: number;
  recency_window_days: number;
}

interface ProposedCellModifier {
  cell_id: string;
  delta_modifier: number;
  evidence: string;                         // AI-drafted evidence sentence
  evidence_sources: Source[];               // optional, AI-extracted citations
}

interface SourceNote {
  note_id: string;
  note_text_hash: string;                   // SHA-256 of the source text (for audit)
  encounter_id: string | null;
  authored_by: string;
  authored_at: string;
}

interface ReviewDecision {
  decision: "accept" | "reject" | "edit";
  notes: string;                             // clinician's reasoning
  edits?: Partial<ProposedSubsystemModifier | ProposedCellModifier>;
}
```

## LLM prompt pattern

The extraction prompt has three parts: scope constraint, output schema, and provenance demand.

### Scope constraint

The prompt is **disorder-template-keyed**. For a patient with `template_refs: ["ocd_canonical_v2"]`, the prompt loads the OCD ElicitationMap's `ai_extraction_targets` — these are the cells and subsystems the AI is allowed to propose modifiers for. The model isn't free-roaming over the registry; it's constrained to a list of plausible targets.

Pattern:

```
You are extracting structured patient modifiers from a clinical note.

The patient has these active disorder templates:
- ocd_canonical_v2 (moderate)

For OCD, you may propose modifiers to these subsystems:
- D (Doubt and over-checking)
- F (Fear of harm / avoidance)
- H (Habit-related compulsivity)
- T (Tic-spectrum overlap)

You may propose cell-level modifiers ONLY for these cells (with these patterns):
- ocd.caudate.DA.tone.tone — propose if note describes severe contamination
  compulsions or hyperactive checking pattern (FDG-PET evidence base)
- ocd.putamen.ACh.CIN.density — propose if note describes family history of TS
  or motor tic onset
- ... (full list from ElicitationMap)

For any other proposed modifier, return subsystem-level only.
```

### Output schema

The model returns structured JSON matching `ProposedSubsystemModifier[]` and `ProposedCellModifier[]`. The prompt provides the schema and example outputs.

### Provenance demand

The prompt requires the model to:

- Quote the source sentence(s) that justify each proposal — verbatim from the input note.
- Provide a one-sentence reasoning for each proposal.
- Self-report confidence (0–1).

Proposals without source sentences are rejected before queuing — there's no extraction without provenance.

## Review queue mechanics

The review queue is the clinician's UI for triaging proposals.

### Queue shape

Sorted by:

1. Pending proposals first (status: `pending_review`).
2. Within pending: by patient (clinician reviews all pending for one patient at a time).
3. Within patient: by ai_confidence ascending (low-confidence first — those need the most review).

### Review actions

For each proposal, the clinician can:

- **Accept** — proposal is committed as a PatientModifier with `source: ai_extracted_clinician_review`. The committed modifier carries forward the clinician identity, decision timestamp, and a reference to the original ProposedModifier.
- **Reject** — proposal stays in the queue with `status: rejected` for audit. Not committed to registry.
- **Edit** — clinician modifies the proposed payload and accepts. The edited version is committed; both original proposal and edits are preserved.

### Bulk operations

Clinicians can accept/reject in bulk for a single patient's session. Bulk accept requires confirming the patient identifier — no accidental "accept all proposals across all patients" path.

### Quality gates

- Proposals with `ai_confidence < 0.5` cannot be bulk-accepted; require individual review.
- Proposals modifying critical cells (e.g., cells with `evidence_status: evidenced` and `confidence: H` in the template) require explicit confirmation: "This cell has high-confidence template data. Are you sure your modifier overrides it?"
- The review UI displays the source sentence highlighted in the original note context — never just the extracted snippet.

## Audit trail

Every proposal lifecycle event is logged:

- Proposal creation (with full LLM input/output).
- Queue movements.
- Reviewer assignments.
- Decisions (accept/reject/edit) with reviewer ID, timestamp, notes.
- Committed modifier ID (if accepted).
- Edits to proposals before acceptance.

Audit log is immutable. Append-only. Stored separately from the registry (different table, different retention policy — clinical decision audit retains longer than registry data).

## Failure modes and mitigations

### 1. Model hallucinates cell IDs

The constraint prompt narrows the search space, but models can still invent cells. **Mitigation:** validate every proposed `cell_id` against the registry before queuing. Invalid IDs reject the proposal (don't even queue it).

### 2. Model misreads sentence

The provenance demand catches some of this — the source sentence is human-readable. The clinician sees what the model thinks justifies the proposal. **Mitigation:** require quoted source sentence for every proposal; clinician can spot-check.

### 3. Model proposes too many modifiers

For long notes, the model may produce 20+ proposals. **Mitigation:** the queue UI groups by source note; the clinician reviews per-note, not per-proposal. Bulk-accept respects this grouping.

### 4. Model misses important findings

Worse than hallucination. The clinician thinks AI extraction is complete and misses a finding. **Mitigation:** AI extraction is **never** the only intake path. Questionnaires are mandatory. Clinician-direct entry is always available. The AI is supplementary, not exclusive.

### 5. PHI in LLM payload

Clinical notes contain PHI. Sending them to a hosted LLM raises BAA and data-flow questions. **Mitigation:** see `12-app-architecture-decisions.md` §3 (regulatory posture) and §4 (registry source-of-truth). v1 ships without AI extraction; v2 enables it only after BAA execution and PHI-handling design.

## What's NOT in this spec

- The exact LLM provider — Anthropic's Claude is the default given the broader stack, but the contract is provider-agnostic.
- Prompt-tuning iterations — those happen during pilot. The pattern is here; the specific phrasing iterates.
- The review queue UI styling — it's a list view with detail panel and accept/reject/edit buttons. Behavior contract is here; styling is downstream.
- The training/fine-tuning pipeline if it materializes. Initial deployment uses the prompt pattern, not fine-tuning.
- The runtime that applies committed modifiers when computing effective deltas. That's the Visualization API generator (`05-visualization-api-payloads.md`).

## v1 vs v2

**v1 (recommended scope):** AI extraction off. Intake = questionnaires + clinician-direct cell modifiers only. Ships without the proposal queue.

**v2 (when BAA + PHI design ready):** AI extraction enabled. Proposal queue + review UI in clinician workflow.

The contracts here are stable across v1 → v2. v1 implementation can stub the AI extraction path; turning it on later is a feature flag, not a rewrite.
