# 00 — Framework Overview

The methodology, the mental model, and what the framework is and isn't trying to do.

---

## Why this framework exists

Psychiatric medication is prescribed in a feedback-poor environment. A clinician picks a first-line agent, the patient takes it for 6–12 weeks, response is judged on subjective scales, and if it fails the next agent is chosen by guideline plus pattern-matching against prior failures. The mechanistic reasoning — *why* this drug should hit this patient's specific neurobiology — happens informally if at all, because the field doesn't have a shared way to represent the patient's neurobiology in operational terms.

This framework supplies that representation. It compiles peer-reviewed neuropsychiatric findings into a structured registry of cells, where each cell is one well-defined neurobiological measurement (a receptor in a region, a transporter density, a circuit's tone) with a delta value relative to a healthy baseline. Disorders, individual patients, and pharmacological treatments are all expressed in the same cell shape, which makes the math simple:

- A disorder is a vector over cells.
- A patient is `disorder vector + individual modifiers`.
- A treatment is a vector that subtracts coverage from the patient vector.
- The residual is what's left uncovered.

The framework's clinical claim is modest and explicit: it provides decision support, not diagnostic or prescribing decisions. Clinicians use it to reason about coverage gaps, mechanistic fit of next-line options, and the distance between candidate diagnoses. It never recommends prescribing actions; it surfaces mechanism-level fit and lets clinicians convert that into prescriptions.

## The core mental model

Three ideas, in order:

### 1. The brain as a cell registry

Every measurable feature of the brain that's been studied at the receptor or circuit level can be tagged with five identifiers: which **disorder** the cell describes, which **region** it sits in, which neurotransmitter or signaling **system** it's part of, the specific **target** (receptor subtype, transporter, cell type), and the **site** (pre-synaptic, post-synaptic, autoreceptor, density measure, dynamic measure, etc.).

A cell is one row keyed on that 5-tuple. It carries a delta value (e.g., `-2` = moderately reduced relative to healthy), a confidence rating, evidence tier, sources, and metadata about how it was reviewed.

Roughly 50–80 cells describe a typical disorder template at adequate coverage. The OCD canonical template, which is the first concrete instantiation, has 53 cells.

### 2. Patients are deltas on top of a template

A patient with OCD doesn't get the OCD template assigned to them as their state. They get the OCD template as a *reference*, and their actual cells are computed as the template plus patient-specific modifiers. This separation is what makes the system extensible — when the template is updated (new evidence), patient profiles can re-baseline against the new template without losing what's specific to the individual.

Two modifier types exist:

- **PatientSubsystemModifier** — produced automatically from validated questionnaires (Y-BOCS, MADRS, etc.) using elicitation maps. This is the common path. Clinician doesn't author cell-level data; they administer instruments and the system maps scores to subsystem-level deltas, weighted onto cells by each cell's `subsystem_weights`.
- **PatientCellModifier** — clinician-authored, evidence-required, for findings that point to a specific cell (family history of TS implicating CIN density; imaging showing reduced putamen volume). Schema enforces that `evidence` is non-empty.

The split keeps clinicians out of cell-level reasoning by default. Subsystems collapse complexity.

### 3. Treatments are vectors too

Each medication has its own coverage vector — a set of cells the agent's mechanism touches. SSRIs have coverage on serotonergic cells across multiple regions. Aripiprazole has coverage on D2/5HT2A cells. NAC has coverage on glutamatergic cells.

Effective patient state, with treatment, is the patient vector minus the active treatments. The residual — what's left uncovered — is computable, displayable, and actionable.

This is what makes "next-line treatment" not a guess. Given a patient vector and a candidate agent, the runtime computes what coverage the agent would add and what residual would remain. Candidates rank by residual reduction × confidence × evidence weight.

## The four tiers

| Tier | Type | Built by | Lifecycle |
|------|------|----------|-----------|
| 0 | HealthyBaselineTemplate | Framework team | Versioned, rare changes |
| 1 | DisorderTemplate | Clinical team | Versioned per disorder, periodic review |
| 2 | PatientProfile | Clinician + patient | Per-patient, lifelong with revisions |
| 3 | PatientState | Clinician at encounter | Per encounter, longitudinal record |

A profile *always* references the baseline. It references zero, one, or more DisorderTemplates depending on `diagnosis_status`:

- `diagnosis_status: undifferentiated` — no template_refs, baseline only. Differential distance ranking surfaces likely templates.
- `diagnosis_status: provisional` — one or more template_refs, but uncomposed or unconfirmed.
- `diagnosis_status: confirmed` — clinician affirms.
- `diagnosis_status: rule_out` — explicitly excluded.

Diagnostic distance — comparing a profile to disorder templates by cell-level distance — is a core primitive, not an optional artifact. It's how undifferentiated profiles get oriented.

## Three layers of presentation

The cell registry is the source of truth, but it's not what users see. Three presentation layers translate from registry to audience:

- **Layer 0 — Cell registry (machine).** JSON-shaped, queryable. Audience: validators, queries, computational tools. Not human-readable in raw form. Every other layer derives from this.
- **Layer 1 — Region/subsystem aggregations (semi-structured).** Region-by-region summaries, subsystem heatmaps. Audience: clinicians who want mechanistic detail. The brain map and subsystem heatmap visualizations live here.
- **Layer 2 — Narrative summaries (clinician-friendly prose).** A plain-language description of the patient state in terms a clinician would write in a note. Composed from templates fed by the registry, with cell-level detail collapsed into mechanism statements.
- **Layer 3 — Patient-facing summary (lay language).** "Most affected systems," "what your treatment covers," "what's still uncovered" — no neuroanatomy jargon. The clinical operational target. Generated from a small set of templates, reviewed by a human before delivery.

Layers 1–3 are deterministic functions of Layer 0. None of them are stored independently; they're computed and rendered.

## What the framework does not try to answer

These are deliberately outside scope:

- **Prescribing decisions.** The framework surfaces fit, not prescriptions. It never says "give X." It says "X covers these gaps; Y covers those."
- **Diagnostic claims.** The framework supports differential reasoning by computing distance between profile and template vectors. Diagnostic claims are clinician-made.
- **Genetic/pharmacogenomic factors.** CYP2D6, COMT, etc. are out of scope for v1. They modify how treatment vectors apply (metabolism rate) but the framework's vector representation doesn't yet incorporate this.
- **Network-level / functional connectivity.** Out of scope for v1. The framework is regional and receptor-level.
- **Acute pharmacodynamics.** Treatment vectors are steady-state, not minute-by-minute kinetics.
- **Causal claims.** A delta is what's measured, not what causes what. The framework assembles measurements; it doesn't claim mechanism causation beyond what individual citations support.

## Audience targeting

The framework is built to serve, in priority order:

1. **Patients (via clinician)** mapping their own brain and understanding their treatment.
2. **Prescribing clinicians** picking next-line agents, augmentation, or evaluating coverage.
3. **Differential diagnosis support** for clinicians evaluating undifferentiated patients.
4. **Mechanism explanation** — *why* a given drug is being prescribed, to whom.

The long-term form is an enterprise app for patients and providers. The short-term form is the methodology spec, registry data, and engineering contracts in this package.

## What "robust" means here

The framework is robust in three senses, which are not the same and shouldn't be conflated:

- **Methodologically robust** — the schema, audit rules, and protocols withstand pressure-testing. The cell schema covers density, dynamic, functional, and composite measurements (not just synaptic site). Subtype-conditional values split rather than flatten. Conflicts are typed and flagged. Profile-template hierarchy supports cascading updates (parked but planned). This is the work this package documents.
- **Content-robust** — having enough populated registry data to compute useful answers. Currently: only OCD is populated. See `11-readiness-and-blockers.md` for what's missing.
- **App-robust** — production-grade infrastructure: PHI handling, auth, audit logging, regulatory posture. Out of scope for the framework; in scope for app architecture. See `12-app-architecture-decisions.md`.

The methodological robustness is what licenses building the app. The other two are work that happens during and before app launch.
