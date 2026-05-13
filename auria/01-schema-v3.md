# 01 — Schema v3

The full schema for the four-tier architecture: HealthyBaselineTemplate, DisorderTemplate, PatientProfile, PatientState — plus modifier types, identifiers, and the effective-delta formula.

This file defines the **types**. The cell shape itself (the most-used type) is broken out into `02-cell-registry-spec.md` because it's large enough to deserve its own document.

---

## Schema scope

v3 is a strict superset of v2. The v2 cell schema is preserved unchanged inside v3; v3 wraps v2 with a patient-layer hierarchy.

- **v1** — deprecated.
- **v2** — single-tier disorder schema. Cell shape, drug coverage cell shape. Still the source of truth for cell-level data.
- **v3** — adds the patient layer (HealthyBaselineTemplate, DisorderTemplate as v2 wrapper, PatientProfile, PatientState, modifier types, ElicitationMap reference, CompositionRule reference).

`schema_version: "3.0"` is required on all v3 records.

## Identifiers

Stable IDs across all tiers. Used as foreign keys in the runtime database and as `$id` references in JSON Schema.

```
HealthyBaselineTemplate.id    →  "healthy_v1"
DisorderTemplate.id           →  "ocd_canonical_v2", "mdd_canonical_v1", etc.
PatientProfile.id             →  UUID
PatientState.id               →  UUID
PatientSubsystemModifier.id   →  UUID
PatientCellModifier.id        →  UUID
Cell.id                       →  "{disorder}.{region}.{system}.{target}.{site}"
                                  e.g. "ocd.ofc.5HT.5HT2A.post-syn"
DrugCoverageCell.id           →  "{agent}.{region}.{system}.{target}.{site}"
                                  e.g. "sertraline.raphe.5HT.SERT.pre-syn"
CompositionRule.id            →  "{template_a}.{template_b}.{scope}.{scope_value}"
                                  e.g. "ocd.ts.cell.ocd.putamen.DA.tone"
ElicitationMap.id             →  "{instrument}.v{version}"
                                  e.g. "y-bocs.v1", "phq-9.v1"
```

Cell IDs sort alphabetically within tier; comparison is meaningful (same template + region groups together).

## Tier 0 — HealthyBaselineTemplate

```typescript
interface HealthyBaselineTemplate {
  id: "healthy_v1";              // singleton; only one healthy baseline
  schema_version: "3.0";
  description: string;
  cells: HealthyBaselineCell[];  // every cell at delta_best = 0
  last_reviewed: string;          // ISO date
  reviewer: string;
}

interface HealthyBaselineCell {
  id: string;                     // "healthy.{region}.{system}.{target}.{site}"
  region: Region;
  system: System;
  target: string;
  site: Site;
  measure_type: MeasureType;
  delta_best: 0;                  // always zero by definition
  delta_range: [0, 0];
  evidence_status: "evidenced";
  applicable: true;
  sources: Source[];
  last_reviewed: string;
}
```

Every PatientProfile references this template by `baseline_ref: "healthy_v1"`. The baseline is a constant — every cell is at zero relative to itself. It exists so the effective-delta formula has a well-defined starting term and so unit tests on delta arithmetic can verify against a known zero.

The healthy baseline is also the universe of cells: a cell that exists in the baseline can carry deltas in disorder templates and modifiers. A cell that doesn't exist in the baseline can't appear elsewhere.

**Status:** schema defined; not yet populated. See `11-readiness-and-blockers.md`.

## Tier 1 — DisorderTemplate

A populated v2 cell registry, plus template metadata.

```typescript
interface DisorderTemplate {
  id: string;                     // e.g. "ocd_canonical_v2"
  schema_version: "3.0";
  disorder: Disorder;             // enum: OCD, MDD, GAD, ADHD, TS, ASD, ...
  template_version: string;        // semver, e.g. "2.0.1"
  severity_bucket: SeverityBucket; // subclinical | mild | moderate | severe | chronic_severe
  phenotype_subtype: string | null; // e.g. "contamination-dominant" for OCD
  baseline_ref: "healthy_v1";
  subsystems: Subsystem[];          // 2–6 subsystems for this disorder
  elicitation_map_ref: string;       // e.g. "y-bocs.v1"
  cells: DisorderCell[];             // ~50–80 cells typically
  drug_coverage_cells: DrugCoverageCell[]; // optional, can be in separate registry
  status: "draft" | "active" | "deprecated";
  last_reviewed: string;
  reviewer: string;
  notes: string;
}
```

The cell shape (DisorderCell, DrugCoverageCell) is fully specified in `02-cell-registry-spec.md`.

Templates are clinical-team-curated. The team owns the canonical form for each disorder. Templates are versioned; old versions are preserved for migration.

**Cascading update policy** is parked for v1. Three options sketched (pin-by-default, auto-migrate with diff review, weighted cascade). Recommendation: pin-by-default for v1; revisit when shipping the second template version.

## Tier 2 — PatientProfile

```typescript
interface PatientProfile {
  id: string;                            // UUID
  schema_version: "3.0";
  patient_id: string;                    // FK to patient record (PHI; out of scope here)
  baseline_ref: "healthy_v1";
  template_refs: string[];               // 0 or more disorder template IDs
  diagnosis_status: DiagnosisStatus;      // undifferentiated | provisional | confirmed | rule_out
  severity_per_template: Record<string, SeverityBucket>; // e.g. { "ocd_canonical_v2": "moderate" }
  cell_modifiers: PatientCellModifier[];
  subsystem_modifiers: PatientSubsystemModifier[];
  curated_comorbidity_template_ref: string | null;
                                          // when set, system uses this curated template
                                          // *instead of* composing template_refs
                                          // e.g. "ocd_plus_ts_v1"
  created_at: string;
  last_updated: string;
  reviewer: string;
}

type DiagnosisStatus = "undifferentiated" | "provisional" | "confirmed" | "rule_out";
```

A profile lives across the patient's lifetime. Every encounter produces a PatientState (Tier 3) snapshot but the profile is the persistent identity.

**`template_refs.length === 0`** → undifferentiated. The patient has presented but no disorder match has been made. Differential distance ranking is the primary primitive for this state.

**`template_refs.length === 1`** → standard single-disorder profile.

**`template_refs.length > 1`** → comorbid. Resolved as either:
- **Curated comorbidity template** if `curated_comorbidity_template_ref` is set.
- **Composition** otherwise, using rules from `03-composition-rules.md`.

## Tier 3 — PatientState

A snapshot at a moment in time. The thing visualizations actually render.

```typescript
interface PatientState {
  id: string;                            // UUID
  schema_version: "3.0";
  profile_ref: string;                   // FK to PatientProfile
  encounter_id: string | null;            // optional FK to clinical encounter record
  active_treatments: ActiveTreatment[];
  acute_modifiers: PatientSubsystemModifier[]; // e.g. acute stress, sleep deprivation
  recorded_at: string;                    // ISO datetime
  recorded_by: string;
}

interface ActiveTreatment {
  agent_id: string;                      // FK to agent (e.g. "sertraline")
  dose_mg: number;
  route: "PO" | "SL" | "IM" | "IV" | "TD";
  duration_days: number;                  // for steady-state coverage assumptions
  started_at: string;
  notes: string;
}
```

Every PatientState is a deterministic function of `(profile + treatments + acute_modifiers + registry snapshot)`. Re-render the same state in 6 months with the same registry version and you get the same visualizations. That's what makes screenshots auditable.

## Modifier types

Patient-specific deltas. Two flavors.

### PatientSubsystemModifier — the common path

Produced automatically from validated questionnaires via ElicitationMaps.

```typescript
interface PatientSubsystemModifier {
  id: string;                       // UUID
  schema_version: "3.0";
  profile_id: string;
  subsystem: string;                 // disorder-relative subsystem key
                                     // e.g. "D" (Doubt) for OCD, "A" (Anhedonia) for MDD
  template_ref: string;              // which disorder template this subsystem belongs to
  delta_modifier: number;             // continuous; rounded only at render
  source: ModifierSource;
  source_ref: string;                 // e.g. "Y-BOCS:28" — the score that produced this
  recency_window_days: number;        // e.g. 7 for Y-BOCS, 180 for ASRS
  applied_at: string;
  reviewer: string;
}

type ModifierSource =
  | "questionnaire"             // automatic from ElicitationMap
  | "clinician_direct"           // manual entry
  | "ai_extracted_clinician_review"; // AI-proposed, clinician-committed
```

The modifier is applied weighted onto every cell whose `subsystem_weights[subsystem]` is non-zero. A patient with `delta_modifier: +1` on OCD's `D` subsystem and a cell with `subsystem_weights: {"D": 0.6, "F": 0.4}` gets `+0.6` added to that cell's effective delta from this modifier alone.

### PatientCellModifier — the exception path

Clinician-authored, evidence-required, for findings that point to a specific cell.

```typescript
interface PatientCellModifier {
  id: string;
  schema_version: "3.0";
  profile_id: string;
  cell_id: string;                  // exact cell ID to override
  delta_modifier: number;
  evidence: string;                  // REQUIRED, non-empty — clinician's explanation
  evidence_sources: Source[];        // optional citations
  source: "clinician_direct" | "ai_extracted_clinician_review";
  applied_at: string;
  reviewer: string;
}
```

`evidence` is a hard schema requirement. A cell-level modifier without explanation is rejected at validation. This is what keeps the system auditable and prevents clinicians from making cell-level claims on hunches.

### Multi-instrument administration

When two instruments map to the same subsystem (e.g., PHQ-9 and MADRS both score depression severity), modifiers do not stack. The higher-confidence source takes precedence (clinician-rated MADRS over self-rated PHQ-9 in most cases). The resolution rule lives in the intake pipeline, not in the ElicitationMap.

### Cross-disorder questionnaires

Y-BOCS scoring is OCD-specific, but a patient with depression and OCD comorbidity will produce a non-zero Y-BOCS. The ElicitationMap is keyed to the disorder template, so Y-BOCS modifiers only apply when an OCD template is referenced in the profile.

## Effective delta formula (canonical)

For any cell `c` at any point in time:

```
effective(c) = baseline(c)
             + Σ_{t ∈ profile.template_refs}
                 t.cells[c].delta_best × severity_factor(t)
             + Σ_{m ∈ profile.subsystem_modifiers + state.acute_modifiers}
                 m.delta_modifier × c.subsystem_weights[m.subsystem]
             + cell_modifier(c)               // if exists
             - Σ_{tx ∈ state.active_treatments}
                 tx.coverage_cells[c].delta_best
```

Where:
- `baseline(c)` is always 0 by Tier-0 definition.
- `severity_factor` scales the template's stored delta to the patient's severity bucket. Implementation choices: linear scaling, lookup table per template, or always 1.0 with severity stored as part of `delta_modifier`. v1 uses linear scaling, with a per-template lookup if needed.
- The summation over modifiers uses `subsystem_weights` to redistribute subsystem-level deltas across cells. Sum of subsystem_weights for any single cell must equal 1.0 ± 0.01.
- For comorbid profiles (`template_refs.length > 1`), the second summation is replaced by composition rules (`03-composition-rules.md`).

The computation is **continuous**. Internal arithmetic uses floats. The `-3 to +3` integer scale is for display only — applied at the rendering step, not before.

`Residual` for treatment-fit purposes is the same formula minus active treatments; if the result is non-zero, that's a coverage gap. Coverage gap magnitude is `|residual|`. Direction matters — a cell with `effective(c) = +2` and treatment coverage `+1` has residual `+1` (still elevated but partially covered); coverage `+3` has residual `-1` (over-corrected).

## Severity bucket mapping

```
SeverityBucket: subclinical | mild | moderate | severe | chronic_severe
```

`severity_factor` defaults:
- `subclinical`: 0.25
- `mild`: 0.5
- `moderate`: 1.0       (the canonical baseline used in templates)
- `severe`: 1.5
- `chronic_severe`: 2.0 (clipped to the cell's `delta_range` if exceeded)

These are starting calibrations. Pilot data is needed to validate.

## Curated comorbidity templates

For high-prevalence comorbid pairs where evidence supports a specific comorbid pattern (not just the composition of two disorders), the framework supports **curated comorbidity templates** as first-class DisorderTemplate-shaped entities.

When `curated_comorbidity_template_ref` is set, the system uses this template directly *instead of* composing the template_refs. The comorbid profile gets `diagnosis_status: confirmed` if instruments support it; no provisional flag.

See `08-comorbidity-templates.md` for the OCD+TS curated template as a worked example.

## JSON Schema location

The full JSON Schema (draft 2020-12) for v3 lives in the schema repo (TBD when registry migrates from Notion to a build-time export pipeline). Two registry-build checks live outside JSON Schema and run at load time:

- `subsystem_weights` sums to 1.0 ± 0.01 for every cell with `delta_best ≠ 0`
- `id` values are unique across the registry

JSON Schema is suitable for config validation in tooling, app backends, and structured-data pipelines.

## Versioning policy

- **Schema versions** are global: `1.0`, `2.0`, `3.0`. New top-level fields, type-shape changes, or breaking validation rules increment the major. Additive non-breaking refinements increment the minor.
- **Template versions** are per-template, semver. `ocd_canonical_v2` to `ocd_canonical_v2.1` is a calibration update; to `v3` is a structural change.
- **Patient profile versions** are not versioned; `last_updated` timestamps and an audit log of modifier writes provide history.

## Reserved enums

These enums are referenced throughout v3 and have a stable ordering. They live in the schema repo:

```
Disorder:        OCD | MDD | GAD | ADHD | TS | ASD | PTSD | BPD | OCD+TS (curated)
Region:          OFC | ACC | dlPFC | vmPFC | SMA | M1 | Caudate | Putamen
                 Ventral Striatum | NAc | GPi | GPe | STN | SNr | SNc | VTA
                 Thalamus | Amygdala | Hippocampus | Insula | Hypothalamus
                 Midbrain | LC | DRN | MRN | CMPf | mPFC | Cerebellum | PAG
System:          5-HT | DA | NE | GABA | Glu | ACh | Histamine | Adenosine
                 Opioid | Cannabinoid | Composite | Metabolism
Site:            pre-syn | post-syn | auto | hetero | tone
                 density | dynamic | functional | composite
Tier (evidence): 1 | 2 | 3
Confidence:      H | M | L
EvidenceStatus:  evidenced | inferred | no-data | not-applicable
Contested:       null | "methodological" | "subtype" | "state-trait"
SeverityBucket:  subclinical | mild | moderate | severe | chronic_severe
DiagnosisStatus: undifferentiated | provisional | confirmed | rule_out
```

Adding a new enum value is non-breaking; removing or renaming requires a major schema bump.
