# 15 — Schema Extensions

Additions to the `01-schema-v3.md` core schema needed to support the platform features. Three groups: the predicted-vs-observed feedback loop (new), FHIR resource mappings (architectural hedge for v2 EHR integration), and full-body extensibility (architectural hedge for non-brain expansion).

The original schema in `01-schema-v3.md` is unchanged. This file adds new types and explains how they fit into the existing tier hierarchy.

---

## 1. Predicted-vs-observed feedback loop

The platform's data moat. When a provider prescribes, the system computes a predicted post-treatment state. At the next visit, the patient retests and an observed post-treatment state is captured. The pair (predicted, observed) is stored. Over time:

- Per-patient: prediction error feeds individual tuning (v3 feature).
- Per-drug: aggregate prediction error tightens population-average estimates (improves the registry).
- Per-cell: error patterns surface where the literature was wrong or population variance is high.

Three new entities:

### `TreatmentPrediction`

A snapshot of what the platform predicted at the moment a treatment was prescribed.

```typescript
interface TreatmentPrediction {
  id: string;
  patient_state_ref: string;           // PatientState at prescribing moment
  patient_id: string;
  prescribed_agent_id: string;         // references DrugCoverageCell agent
  prescribed_dose: string;             // free text in v1, structured later
  prescribed_at: string;               // ISO datetime
  expected_evaluation_at: string;      // ISO datetime, typically +6 to +8 weeks
  prescribing_clinician_id: string;

  // The actual prediction: predicted effective deltas per cell after treatment
  predicted_deltas: Array<{
    cell_key: string;
    pre_treatment_delta: number;       // effective(cell) at prescribed_at
    post_treatment_delta: number;      // computed = pre - drug_coverage(cell, dose)
    drug_coverage_value: number;       // for audit/explanation
  }>;

  registry_version: string;            // freeze for reproducibility
  notes?: string;                      // optional clinician note
}
```

The prediction is computed once at prescribing time and frozen. It's not recomputed even if the registry updates — because the *retrospective* question is "did our prediction at the time of prescribing match what happened?" not "would today's registry version have predicted differently?"

### `ObservedOutcome`

What the questionnaires actually showed at the follow-up visit.

```typescript
interface ObservedOutcome {
  id: string;
  prediction_id: string;               // links to TreatmentPrediction
  patient_id: string;
  observed_at: string;                 // ISO datetime of follow-up retest
  observation_state_ref: string;       // PatientState computed from follow-up questionnaires

  // The observed deltas, parallel structure to TreatmentPrediction.predicted_deltas
  observed_deltas: Array<{
    cell_key: string;
    pre_treatment_delta: number;       // copied from prediction for diff display
    post_treatment_delta: number;      // observed at this visit
    error: number;                     // observed - predicted, signed
  }>;

  // What was happening
  treatment_adherence?: "high" | "moderate" | "low" | "unknown";
  side_effects_reported?: string[];    // free text
  treatment_changed?: boolean;         // did clinician modify regimen between prescribing and observation
  notes?: string;
}
```

Adherence and side effects are captured because they're confounders. A prediction missing because the patient didn't take the medication isn't the registry being wrong.

### `PredictionAccuracyAggregate`

Per-cell aggregation of prediction errors across all (TreatmentPrediction, ObservedOutcome) pairs in the system. Computed as a background job, not in real-time.

```typescript
interface PredictionAccuracyAggregate {
  cell_key: string;                    // disorder|region|system|target|site
  agent_id: string;                    // which drug
  dose_bucket: string;                 // "low" | "standard" | "high" — coarsened
  
  n_pairs: number;                     // sample size
  mean_error: number;                  // bias: positive = under-predicted coverage
  std_error: number;                   // variance
  median_absolute_error: number;       // robust accuracy metric
  
  computed_at: string;
  registry_version_at_computation: string;
}
```

Two uses:

1. **Diagnostic dashboard** (admin / clinical reviewer): which cells / drugs are mis-predicting consistently? Suggests cell value revisions.
2. **Future individual tuning** (v3): the patient's own (TreatmentPrediction, ObservedOutcome) pairs become Bayesian evidence for predictions on their next prescribing decision.

In v1 only the storage and computation are built. The diagnostic dashboard ships as part of `06-frontend-primitives-spec.md` AuditPayload renderer.

### Where these fit in the tier hierarchy

```
Tier 0:  HealthyBaselineTemplate
Tier 1:  DisorderTemplate
Tier 2:  PatientProfile
Tier 3:  PatientState

Tier 3a: TreatmentPrediction      ← new, attached to PatientState at prescribing
Tier 3b: ObservedOutcome          ← new, attached to PatientState at follow-up
Tier 3c: PredictionAccuracyAggregate  ← new, registry-level aggregation
```

`TreatmentPrediction` and `ObservedOutcome` are Tier-3 because they're patient-specific; `PredictionAccuracyAggregate` is registry-level because it aggregates across patients (de-identified at aggregation time).

## 2. FHIR resource mappings

Even though v1 is standalone, the API and database schemas are designed so each entity maps cleanly to a FHIR R4 resource. v2 EHR integration becomes a field-mapping layer, not a redesign.

| Platform entity | FHIR R4 resource | Notes |
|-----------------|-------------------|-------|
| `PatientProfile` | `Patient` | Demographics fields align; clinical observations live in separate resources |
| `PatientProfile.template_refs[].diagnosis_status` | `Condition` | One Condition per template_ref; diagnosis_status maps to Condition.verificationStatus |
| Questionnaire response (Y-BOCS, etc.) | `Observation` | Each instrument's score(s) become an Observation with LOINC code |
| Active regimen agent | `MedicationStatement` | Or `MedicationRequest` if the platform writes prescriptions back (v2 bidirectional) |
| Visit / encounter | `Encounter` | Time-bounded interaction; questionnaires, prescriptions, notes attach to it |
| `TreatmentPrediction` | `RiskAssessment` | FHIR's general "predicted future state" resource; custom prediction structure in `prediction.outcome` |
| `ObservedOutcome` | `Observation` (a follow-up panel) | Same shape as initial questionnaire, references the prior prediction |
| Audit log entry | `AuditEvent` | FHIR's standard audit resource |

### Concrete mapping example: Y-BOCS response

Platform internal:

```typescript
{
  patient_id: "p_abc123",
  instrument: "ybocs",
  responses: { obsessions_total: 14, compulsions_total: 13, insight: 1 },
  administered_at: "2026-04-29T13:30:00Z",
  administered_by: "clinician_42"
}
```

FHIR `Observation` (when v2 syncs to EHR):

```json
{
  "resourceType": "Observation",
  "status": "final",
  "category": [{ "coding": [{ "system": "http://terminology.hl7.org/CodeSystem/observation-category", "code": "survey" }] }],
  "code": { "coding": [{ "system": "http://loinc.org", "code": "70241-2", "display": "Yale-Brown Obsessive-Compulsive Scale" }] },
  "subject": { "reference": "Patient/p_abc123" },
  "encounter": { "reference": "Encounter/e_xyz789" },
  "effectiveDateTime": "2026-04-29T13:30:00Z",
  "performer": [{ "reference": "Practitioner/clinician_42" }],
  "component": [
    { "code": { "coding": [{ "system": "http://loinc.org", "code": "70243-8" }] }, "valueInteger": 14 },
    { "code": { "coding": [{ "system": "http://loinc.org", "code": "70244-6" }] }, "valueInteger": 13 },
    { "code": { "text": "insight" }, "valueInteger": 1 }
  ]
}
```

The platform stores its internal shape; a FHIR adapter (in v2) projects to/from the standard. Internal shape stays small and ergonomic; FHIR shape is verbose but interoperable.

### Database design implication

Postgres tables align with FHIR resource concepts even when v1 doesn't expose FHIR. Examples of the alignment:

```sql
-- Aligns with FHIR Patient
CREATE TABLE phi.patients (
  id uuid PRIMARY KEY,
  external_id text,                    -- FHIR resource id when synced
  birth_year int,                      -- Coarse for PHI minimization
  sex_at_birth text,
  race text[],                         -- Multi-value per FHIR
  ethnicity text,
  preferred_language text,
  ...
);

-- Aligns with FHIR Observation (questionnaire panel)
CREATE TABLE phi.questionnaire_responses (
  id uuid PRIMARY KEY,
  patient_id uuid REFERENCES phi.patients(id),
  encounter_id uuid REFERENCES phi.encounters(id),
  instrument_code text,                -- "ybocs", "phq9", etc.
  loinc_code text,                     -- For FHIR adapter: "70241-2" for Y-BOCS
  responses jsonb,                     -- Internal flat structure
  administered_at timestamptz,
  administered_by uuid REFERENCES phi.clinicians(id),
  ...
);

-- Aligns with FHIR Encounter
CREATE TABLE phi.encounters (
  id uuid PRIMARY KEY,
  patient_id uuid REFERENCES phi.patients(id),
  clinician_id uuid REFERENCES phi.clinicians(id),
  started_at timestamptz,
  ended_at timestamptz,
  encounter_type text,                 -- "initial" | "follow-up" | "retest"
  ...
);
```

Notes about this:

- Platform stays ergonomic; the FHIR adapter is its own concern.
- LOINC codes pre-recorded on instrument definitions so the FHIR adapter doesn't have to look them up at runtime.
- `external_id` field on every PHI table for FHIR resource id when synced.
- Birth year not full DOB: PHI minimization for runtime data; full DOB lives in a separate identity table only if needed.

## 3. Full-body extensibility

The schema is already system-agnostic. To make it explicitly extensible without committing to non-brain v1 content, three things harden up:

### Region taxonomy is open enum, not closed

Original schema treats `region` as a string. Make it explicit:

```typescript
interface RegionDefinition {
  code: string;                        // "OFC", "Caudate", "thyroid", "gut_wall"
  name: string;                        // human-readable
  anatomical_system: AnatomicalSystem; // "CNS" | "endocrine" | "immune" | "GI" | "cardiovascular" | ...
  parent_region?: string;              // for hierarchical anatomy (OFC ⊂ PFC)
  rendering_hints?: {
    map_layer: "brain" | "body";       // which anatomical layer to render on
    hex_position?: { x: number; y: number };  // for hex-grid rendering
    svg_path?: string;                 // for anatomical SVG (later)
  };
  active_in_registry: boolean;         // gates whether v1 surfaces this region
}

type AnatomicalSystem = 
  | "CNS" 
  | "PNS" 
  | "endocrine" 
  | "immune" 
  | "GI" 
  | "cardiovascular" 
  | "renal" 
  | "musculoskeletal";
```

In v1, only regions with `active_in_registry: true` and `anatomical_system: "CNS"` populate. v3+ flips other systems live as content gets authored. No code changes needed.

### System taxonomy similarly open

```typescript
interface SystemDefinition {
  code: string;                        // "5HT", "DA", "GABA", "HPA", "cortisol", "TNF-alpha"
  name: string;
  category: SystemCategory;
  
  // Whether the system has receptor-level subtypes that live in `target`
  has_targets: boolean;
  
  // Whether the system has site-level distinctions (pre-syn vs post-syn etc.)
  has_sites: boolean;
}

type SystemCategory = 
  | "neurotransmitter"                 // 5HT, DA, NE, GABA, Glu, ACh
  | "neuromodulator"                   // peptides, neuroactive steroids
  | "hormone"                          // cortisol, T3/T4, sex steroids
  | "cytokine"                         // TNF-alpha, IL-6
  | "metabolite"                       // lactate, kynurenine
  | "axis";                            // HPA, HPT, HPG
```

The cell schema's site enum (pre-syn / post-syn / auto / hetero / etc.) was designed for receptors but it generalizes: an immune cytokine has "circulating" vs "tissue-resident" sites; a hormone has "free" vs "bound" forms. The taxonomy stretches.

### Anatomical asset layer is pluggable

The brain map renderer in `16-frontend-ux.md` is a function of `(RegionDefinition, delta_value) → visual`. The renderer doesn't know anything brain-specific:

- v1 brain hex layer: render regions where `rendering_hints.map_layer === "brain"`.
- v2+ body hex layer: render regions where `rendering_hints.map_layer === "body"`.
- v3+ multi-system view: composite renderer combining brain + body + endocrine + immune.

Asset layer is content, not code. New disorder template + new region/system entries + new rendering hints = system extension without rebuild.

### Disorder templates are domain-agnostic

A `DisorderTemplate` for "primary hypothyroidism" looks structurally identical to one for OCD:

```typescript
{
  id: "primary_hypothyroidism_v1",
  schema_version: "3.0",
  name: "Primary Hypothyroidism",
  subsystems: [
    { code: "thyroid_axis_failure", name: "HPT axis decompensation", weight: 0.7 },
    { code: "peripheral_resistance", name: "Tissue T4→T3 conversion", weight: 0.3 }
  ],
  cells: [
    { region: "thyroid", system: "T4", target: null, site: "circulating", baseline_delta: -3, ... },
    { region: "anterior_pituitary", system: "TSH", target: null, site: "circulating", baseline_delta: +3, ... },
    ...
  ],
  ...
}
```

Same structure, same composition rules apply, same payload generators work. v1 doesn't ship this. v3+ can add it without changing schema.

### What this costs in v1

Almost nothing. The region and system definitions live in their own registry tables (already a good idea anyway). The renderer takes anatomical hints as input rather than hardcoding brain layout. The `anatomical_system` field on regions just defaults to "CNS" for v1 content. Roughly an extra day of design work, zero ongoing cost.

What it buys: when you decide to extend (likely: inflammation in MDD via TNF-alpha and IL-6 cells, gut-brain via 5-HT in enterochromaffin cells), the work is content authoring + rendering hints + a body asset layer. Not a rebuild.

## 4. Adverse event surface

For SaMD-readiness (Track B), a structured place to capture unexpected patient responses:

```typescript
interface AdverseEventReport {
  id: string;
  patient_id: string;
  patient_state_ref?: string;          // PatientState at time of event, if computable
  reported_by: string;                 // clinician_id
  reported_at: string;
  
  event_type: AdverseEventType;
  severity: "mild" | "moderate" | "severe" | "life-threatening";
  outcome: "recovered" | "recovering" | "ongoing" | "fatal" | "unknown";
  
  related_treatment_id?: string;       // active regimen agent at time of event
  related_prediction_id?: string;      // if event contradicts a TreatmentPrediction
  
  description: string;                 // narrative
  reporter_attributed_to_platform: boolean;  // did the clinician feel the platform's recommendation contributed
  
  notes?: string;
}

type AdverseEventType = 
  | "side-effect"
  | "treatment-failure"
  | "unexpected-improvement"           // worth tracking; also informs cell calibration
  | "deterioration"
  | "interaction"
  | "other";
```

In v1 this is a passive surface — clinicians can log events, but no proactive surveillance. In v3 (SaMD), this feeds a formal pharmacovigilance / post-market surveillance pipeline.

Storing this in v1 costs ~1 table and a basic CRUD endpoint. Retrofitting later is much harder because there's no historical signal to learn from.

## 5. Versioning the new types

All three new entity types (`TreatmentPrediction`, `ObservedOutcome`, `AdverseEventReport`) are immutable once written. Updates produce new records with `supersedes: <prior_id>` pointers. `PredictionAccuracyAggregate` is regenerable from the underlying pairs, so it carries `computed_at` and `registry_version_at_computation` for cache invalidation.

Schema version of these extensions: **3.1**. Original schema (`01-schema-v3.md`) stays at 3.0; the bump signals an additive extension that 3.0 clients can ignore safely.

## Cross-references

- `01-schema-v3.md` — original schema, unchanged
- `02-cell-registry-spec.md` — cell schema referenced by `predicted_deltas[].cell_key` and `observed_deltas[].cell_key`
- `13-api-endpoint-derivation.md` — the API surface that needs new endpoints for prediction/outcome
- `17-backend-stack.md` — Postgres DDL implementation of these types
- `16-frontend-ux.md` — UI surfaces that render predicted/observed comparisons
