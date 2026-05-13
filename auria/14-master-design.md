# 14 — Master Design

The single orchestrating document for the Auria platform and API. Ties the framework methodology (`00`–`13`) to a concrete buildable product with all architectural decisions resolved.

Read this first if orienting to "what are we shipping?"

---

## Product vision

A platform that lets a psychiatrist (and eventually any prescribing provider) *see* what's happening in a patient's brain at the receptor level, decide on a treatment with mechanism-grounded reasoning, predict the post-treatment state, and validate that prediction at the next visit. A real-time clinician tool that bridges the abstraction between neuroscience research and prescribing decisions.

The canonical patient flow:

1. Patient takes a validated questionnaire (Y-BOCS, PHQ-9, etc.) — pre-visit on their own device or in-office on a tablet.
2. The platform renders a **brain heatmap** showing the patient's effective receptor-level state vs. healthy baseline. Color encodes magnitude and direction of dysregulation.
3. Below the heatmap, **treatment recommendations** appear, ranked by mechanism overlap with the patient's specific dysregulation pattern. Each recommendation shows the cells it targets and the evidence behind those targets.
4. Provider has a clinical conversation with the patient. The **AI scribe** captures audio, transcribes via Whisper, and structured Claude calls extract clinical observations as proposed cell modifiers. Proposals appear in a review queue with the verbatim transcript quote that justified each one.
5. Provider reviews the queue, approves or rejects each proposal. Approved modifiers update the brain map live.
6. Provider selects a medication and dose. The platform overlays a **predicted post-treatment layer** showing how the brain map is expected to change after 6-8 weeks at that dose, derived from the drug coverage cells.
7. Patient returns at follow-up. They retake the questionnaire. The platform compares **predicted vs. observed** deltas and stores the prediction-accuracy pair. Over time, predicted-vs-observed pairs tighten registry calibration and personalize predictions for that patient.

Same engine eventually extends from brain to full body: HPA-axis, immune, gut, endocrine. The schema is system-agnostic by design; only the registry content is brain-specific in v1.

## Confirmed decisions

| Area | Decision |
|------|----------|
| Primary user (v1) | Psychiatrists / psych NPs first; PCPs in v2 with simplified mode |
| Regulatory framing (v1) | CDS-exempt decision support |
| Medication recommendations (v1) | Specific drugs ranked, with mechanism reasoning visible |
| Patient-facing surface | Both pre-visit and in-office |
| Predicted treatment layer | Population average only |
| Anatomy in v1 | Brain only |
| Build resources | Solo, part-time, Claude as primary coding partner |
| Stack | React + Vite + TypeScript + Tailwind + shadcn/ui (frontend); Node + Fastify + TypeScript + Supabase (backend); Anthropic API + Whisper for AI |
| Brain visualization | Stylized anatomical brain rendering with color-encoded regions and multi-layer stacking; hex-grid retained as the underlying data model. See `21-ux-north-star.md` for the visual spec. |
| AI scribe | **In v1.** Audio capture → Whisper transcription → Claude structured extraction → tiered review queue (auto-approve above 0.85 confidence with reversible audit trail; below-threshold and high-stakes proposals queue for explicit review) |
| Scribe failure mode | Partial salvage on transcription/extraction failure — surviving segments commit with timestamped gap markers for the irrecoverable spans |
| Schema source-of-truth | Supabase Postgres |
| PHI separation | Single Supabase project, separate `phi.*` and `registry.*` schemas + strict RLS |
| Auth | Supabase Auth, MFA required for clinician/admin roles |
| Cascading template updates | Pin-by-default |
| Brain types layer | Stable patient trait; 6 authored archetypes (`20-brain-types.md`). Drives lifestyle and supplement recommendations; does not rank drugs. |
| Supplements | Ranked alongside drugs in the same treatment fit table, distinguished by evidence-tier badges (A / B / C). |
| Drug coverage granularity | Discrete dose bands (low / medium / high) per agent. |
| Undifferentiated patient path | "No diagnosis yet — assess only" option at intake (`16-frontend-ux.md`); patient flows through assessment without a template_ref until one is assigned. |
| Patient summary | AI-drafted post-visit; provider reviews and edits before sending. |

## v1 scope — the full capable platform

The goal is a fully capable tool a psychiatrist can run their practice on. Not an MVP that demonstrates a concept; a real product.

What ships in v1:

- Provider auth, single-clinic deployment.
- Patient profiles with multi-disorder template_ref support.
- Questionnaires: Y-BOCS, PHQ-9, GAD-7, MADRS — administered pre-visit (secure link to patient phone) or in-office (tablet).
- Brain map heatmap with multi-layer stacking (effective / disorder-only / coverage / residual / predicted post-Rx).
- Subsystem heatmap and residual gap list.
- Treatment fit table naming specific drugs, with mechanism reasoning and evidence visible at the point of decision.
- Predicted post-treatment overlay (population average).
- Manual cell modifier entry (clinician-direct, in addition to AI-extracted).
- **AI scribe**: audio capture during visit, Whisper transcription, Claude-based structured extraction of proposed modifiers, clinician review queue with approve/reject/edit per proposal.
- Follow-up retest comparison (predicted vs. observed deltas).
- Audit trail viewer.
- Layer-3 patient-facing summary (post-visit, plain language, no specific drug names).
- 9 disorder templates: OCD, MDD, GAD, Panic Disorder, Social Anxiety, PTSD, ADHD, Insomnia, Adjustment Disorder. OCD is the clinically authored reference; the other 8 are AI-drafted (files `22`–`29`) pending clinical review before pilot.
- ~10 first-line drug coverage profiles (sertraline, fluoxetine, escitalopram, venlafaxine, duloxetine, bupropion, lamotrigine, aripiprazole, clonidine, NAC).
- Healthy baseline template populated.

Out of v1 (genuinely deferred):

- Multi-clinic / enterprise auth.
- EHR integration.
- Public API as a separate product.
- DifferentialDistance ranking (needs more disorder templates).
- Confidence intervals on predictions.
- Individual-tuned predictions (requires accumulated outcome data).
- Three-way comorbid composition.
- 3D anatomical brain rendering.
- Mobile native apps.

## v2 directions

Post-v1, based on pilot signal:

- Additional disorder templates beyond the v1 nine (e.g., BD, OUD) and further instruments.
- More drug coverage agents (15-25 total).
- DifferentialDistance ranking.
- PCP simplified mode.
- Public API as a developer product (separate auth, rate limiting, billing).
- More sophisticated AI scribe: real-time transcript display during visit; multi-speaker diarization; longer-context summarization.
- Pharmacogenomic integration (CYP2D6, COMT) where it changes drug coverage estimates.

## Architectural posture toward future upgrades

The architecture is built with good engineering practices that happen to also support two future upgrades cheaply if pursued:

**Future EHR integration.** API data shapes mirror FHIR resources from day one — `Patient`, `Observation`, `MedicationStatement`, `Condition`, `Encounter`. Even though no FHIR client connects in v1, the database schema and API surface are FHIR-shaped. When EHR integration becomes worthwhile, it's a mapping layer, not a redesign. See `15-schema-extensions.md` for the resource mappings.

**Future SaMD clearance.** The architecture has audit trails, versioned reasoning, evidence chains on every cell, deterministic computation, and a clean separation between AI-generated suggestions and clinician-committed data. These are good engineering practices for a clinical tool regardless of regulatory ambition. They also happen to be what an FDA submission would require, so a future clearance pursuit doesn't need a technology rewrite — just QMS work, clinical validation studies, and a Medical Director, which are capital and team concerns.

Neither upgrade is the headline plan. The headline plan is to build the platform. These notes exist so the architecture isn't accidentally painted into a corner.

## Recent v1.1 additions

A set of late-stage v1 expansions (still inside the v1 envelope, hence "v1.1") that are reflected in this document and the roadmap:

- **Brain types layer** — `20-brain-types.md`. 6 authored archetypes, stable patient trait, drives lifestyle and supplements (not drug ranking).
- **Expanded disorder set** — `22`–`29`. Eight AI-drafted templates beyond OCD (MDD, GAD, Panic, Social Anxiety, PTSD, ADHD, Insomnia, Adjustment Disorder), pending clinical review.
- **UX north star** — `21-ux-north-star.md`. Locks the stylized anatomical brain as the visual direction; hex-grid is retained as the data model only.
- **Schema extensions for v1.1** — `30-v1.1-schema-extensions.md`. Adds sleep/circadian vocabulary (SCN, Hypothalamus, LH, Pineal regions; Orexin, Melatonin systems), the `subtype_overrides` pattern (used for PTSD dissociative subtype and MDD melancholic/atypical), and a `brain_type_anchor` field on `DisorderTemplate`.

## Document map

The bundle is 21 files. Reading paths by role:

**Solo founder orienting to the product:**
1. `README.md`
2. `14-master-design.md` (this file)
3. `19-v1-roadmap.md` (what to build first)
4. `00-framework-overview.md`

**Engineer implementing the backend:**
1. `14-master-design.md`
2. `01-schema-v3.md`
3. `15-schema-extensions.md`
4. `17-backend-stack.md`
5. `13-api-endpoint-derivation.md`
6. `05-visualization-api-payloads.md`

**Engineer implementing the frontend:**
1. `14-master-design.md`
2. `16-frontend-ux.md`
3. `06-frontend-primitives-spec.md`
4. `05-visualization-api-payloads.md`

**Engineer working on AI scribe and extraction:**
1. `07-ai-extraction-spec.md`
2. `18-structured-ai.md`

**Clinical reviewer / advisor:**
1. `00-framework-overview.md`
2. `09-ocd-reference-instantiation.md`
3. `10-protocol-and-audit-rules.md`
4. `08-comorbidity-templates.md`

## What changed from `00`–`13`

The reference bundle (`00`–`13`) is the methodology. Files `14`–`19` are the buildable product. Where they conflict, `14`–`19` are correct because they have all decisions resolved.

Specific updates:

- `12-app-architecture-decisions.md` Decision 1 (PHI separation): downgraded from separate clusters to single-project + RLS.
- `11-readiness-and-blockers.md` v1 scope: replaced by `19-v1-roadmap.md` Phase contents.
- `13-api-endpoint-derivation.md`: still correct; `17-backend-stack.md` is the concrete implementation.
- `01-schema-v3.md`: extended by `15-schema-extensions.md`. Original schema unchanged.
- `06-frontend-primitives-spec.md`: still correct; `16-frontend-ux.md` composes primitives into screens.
- `07-ai-extraction-spec.md`: methodology stays; `18-structured-ai.md` makes it concrete and shippable in v1.

## Critical design principles

These hold across every file in the bundle and every line of code in the build:

**Determinism.** Same `(PatientState, registry_version)` always produces the same payloads. Same payloads always render the same UI. No randomness in clinical reasoning, no time-dependent computation outside explicit modifier timestamps. Auditable, debuggable, reproducible.

**Evidence visible at the point of decision.** Every recommendation shows its basis. Every cell value links to its sources. CDS exemption requires this; clinical trust requires this.

**AI proposes, clinician commits.** The system never auto-writes patient data from an AI inference. Every proposed modifier (from the AI scribe or any other source) goes to a queue. Clinician explicitly approves or rejects. The audit trail records both the proposal and the decision.

**Versioning everywhere.** Cells, templates, composition rules, elicitation maps, drug coverage profiles, payload generators all carry explicit version IDs. Nothing deleted, only deprecated.

**Full-body-ready even when only brain populated.** Region and system are extensible enums. Anatomical rendering is a pluggable asset layer. v1 ships brain-only content; the architecture is ready for thyroid, gut, immune, etc.

## Bottom line

v1 is a fully capable platform: brain heatmap, treatment fit, predicted post-treatment overlay, AI scribe, follow-up retest comparison. ~16-20 weeks to ship at solo part-time pace with Claude as primary coding partner.

The architecture supports future EHR integration and SaMD clearance as additive upgrades, but those aren't the v1 plan. The v1 plan is to build the platform.

Next: read `19-v1-roadmap.md` for the week-by-week build plan.
