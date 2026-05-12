# 05 — Visualization API — Typed Payloads

The seven typed payloads the runtime emits. Each is a deterministic function of `(PatientState, registry snapshot)`. Same inputs always produce the same payload — that's what makes screenshots reproducible and bugs debuggable.

The front-end never reaches into the registry directly. All visualization is a function of these payloads.

---

## Architecture overview

```
Notion / Database (source of truth)
  ↓  registry export
Registry (Layer 0 — cells, rules, templates)
  ↓  effective(c) computation per Schema v3 formula
PatientState (regimen + acute modifiers applied)
  ↓  payload generators (deterministic functions)
Visualization Payloads (this document)
  ↓  rendered by
Front-end primitives (06-frontend-primitives-spec.md)
  ↓  composed into
Clinician UI / Patient UI / Audit UI
```

Key invariant: every payload is reproducible from the same `(PatientState, query)` inputs.

## Common types

```typescript
type Delta = number; // continuous internally; rounded to int [-3, +3] at render
type Confidence = "H" | "M" | "L";
type EvidenceStatus = "evidenced" | "inferred" | "no-data" | "not-applicable";
type Tier = 1 | 2 | 3;
type Site =
  | "pre-syn" | "post-syn" | "auto" | "hetero" | "tone"
  | "density" | "dynamic" | "functional" | "composite";
type Contested = null | "methodological" | "subtype" | "state-trait";

interface Source {
  id: string;          // DOI or PMID
  type: string;        // PET-imaging, MRS-human, etc.
  tier: Tier;
  year: number;
  n?: number;
}

interface PayloadMetadata {
  patient_state_ref: string;
  generated_at: string;        // ISO datetime
  registry_version: string;
  schema_version: "3.0";
}
```

`PayloadMetadata` is on every payload. `registry_version` tags which export of the registry was used; payloads with different `registry_version` aren't directly comparable.

## 1. BrainMapPayload

The region/receptor brain map. The headline visualization.

```typescript
interface BrainMapPayload {
  metadata: PayloadMetadata;
  cells: BrainMapCell[];
  legend: {
    delta_scale: { min: -3; max: 3; midpoint: 0 };
    subsystem_colors: Record<string, string>;  // e.g. { "D": "#1f77b4", "F": "#ff7f0e" }
  };
}

interface BrainMapCell {
  cell_id: string;
  region: string;
  system: string;
  target: string;
  site: Site;

  template_delta: Delta;        // contribution from template_refs (composed)
  modifier_delta: Delta;         // contribution from patient modifiers
  treatment_coverage: Delta;     // contribution from active treatments
  effective_delta: Delta;        // sum (rounded for display)
  residual_delta: Delta;         // effective - treatment_coverage

  applicable: boolean;
  evidence_status: EvidenceStatus;
  contested: Contested;
  confidence: Confidence;
  subsystem_weights: Record<string, number>;
  sources: Source[];
}
```

Each cell carries its full provenance: where the delta came from (template, modifier, treatment), its evidence status, sources. The primitive uses this to decide rendering state (gray for `applicable: false`, low-opacity for `no-data`, hatched for contested).

## 2. SubsystemHeatmapPayload

Region × subsystem aggregation.

```typescript
interface SubsystemHeatmapPayload {
  metadata: PayloadMetadata;
  regions: string[];
  subsystems: string[];
  matrix: SubsystemHeatmapCell[][];   // [region][subsystem]
  aggregates: {
    by_region: Record<string, Delta>;    // sum across subsystems for each region
    by_subsystem: Record<string, Delta>;  // sum across regions for each subsystem
  };
}

interface SubsystemHeatmapCell {
  region: string;
  subsystem: string;
  aggregated_delta: Delta;             // weighted sum of contributing cell effective_deltas
  contributing_cells: string[];        // cell_ids that fed this aggregation
  confidence: Confidence;              // worst confidence among contributors
}
```

Aggregation formula:

```
matrix[region][subsystem].aggregated_delta = Σ_{cells in (region, subsystem)}
  cell.effective_delta × cell.subsystem_weights[subsystem]
```

## 3. ResidualGapPayload

The "what's not covered" view. Drives next-line treatment thinking.

```typescript
interface ResidualGapPayload {
  metadata: PayloadMetadata;
  threshold: number;                    // e.g. 1.0 — minimum |residual| to surface
  gaps: ResidualGap[];
  summary: {
    total_gap_magnitude: number;        // Σ |residual| above threshold
    most_affected_subsystems: { subsystem: string; magnitude: number }[];
    most_affected_regions: { region: string; magnitude: number }[];
  };
}

interface ResidualGap {
  cell_id: string;
  region: string;
  system: string;
  target: string;
  effective_delta: Delta;
  treatment_coverage: Delta;
  residual_delta: Delta;
  candidate_agents: CandidateAgent[];   // agents whose coverage would close this gap
}

interface CandidateAgent {
  agent_id: string;
  agent_class: string;
  expected_coverage: Delta;             // how much this cell's residual would change
  confidence: Confidence;
  evidence_tier: Tier;
}
```

## 4. DifferentialDistancePayload

Compares the current profile against candidate disorder templates. Drives differential diagnosis support.

```typescript
interface DifferentialDistancePayload {
  metadata: PayloadMetadata;
  comparator_template_ids: string[];
  rankings: DifferentialRanking[];
}

interface DifferentialRanking {
  template_id: string;
  template_label: string;               // human-readable disorder name
  distance: number;                     // Euclidean distance over cell deltas
  similarity_score: number;             // 0-1, derived from distance
  matching_cells: string[];             // cells where profile and template agree
  distinguishing_cells: DistinguishingCell[]; // cells where they disagree
}

interface DistinguishingCell {
  cell_id: string;
  profile_delta: Delta;
  template_delta: Delta;
  difference: Delta;                    // |profile - template|
  evidence_status: EvidenceStatus;
}
```

Distance metric (default): Euclidean over the intersection of cells with `applicable: true` in both profile and template. Distinguishing cells are those with the largest `|profile_delta - template_delta|`.

## 5. TreatmentFitPayload

For each candidate agent, expected coverage gain per subsystem and predicted residual after addition.

```typescript
interface TreatmentFitPayload {
  metadata: PayloadMetadata;
  current_regimen: ActiveTreatmentSummary[];
  candidates: TreatmentFitCandidate[];
}

interface ActiveTreatmentSummary {
  agent_id: string;
  agent_class: string;
  cells_covered: number;
  total_coverage_magnitude: number;
}

interface TreatmentFitCandidate {
  agent_id: string;
  agent_class: string;
  expected_coverage_gain_by_subsystem: Record<string, number>;
  predicted_residual_after_addition: number;
  contraindications: string[];
  evidence_tier: Tier;
  evidence_source: string;              // 'RCT' | 'open-label' | 'imaging' | 'mechanistic'
  mechanism_summary: string;
  ranking_rationale: string;
}
```

Output explicitly excludes prescribing instructions — it surfaces mechanism-level fit; clinicians convert that into prescriptions.

## 6. PatientFacingPayload

Layer 3 plain language. Patient-facing, lay terms.

```typescript
interface PatientFacingPayload {
  metadata: PayloadMetadata;
  most_affected_subsystems: PatientSubsystemSummary[];
  current_treatment_summary: PatientTreatmentSummary;
  next_steps_plain: string;
  glossary: Record<string, string>;     // technical_term → plain definition
}

interface PatientSubsystemSummary {
  plain_label: string;                   // 'Doubt and over-checking', not 'D'
  severity_text: string;                  // 'Moderately elevated'
  plain_explanation: string;              // 1-2 sentence narrative
}

interface PatientTreatmentSummary {
  agents: { name: string; what_it_targets_plain: string }[];
  what_remains_uncovered_plain: string;
}
```

Templates feeding this are reviewed by a human (clinical advisor) before delivery. Layer 3 is the **clinical operational target** — the artifact the framework exists to produce for end users.

## 7. AuditPayload

Registry health. Internal-facing.

```typescript
interface AuditPayload {
  metadata: PayloadMetadata;
  failures: AuditFailure[];
  staleness: StalenessRow[];
  summary: {
    total_cells: number;
    failures_by_severity: Record<"high" | "medium" | "low", number>;
  };
}

interface AuditFailure {
  cell_id: string;
  rule: string;                          // 'subsystem_weights_sum_to_one' etc.
  severity: "high" | "medium" | "low";
  current_value: unknown;
  expected: string;
  suggested_fix: string;
}

interface StalenessRow {
  cell_id: string;
  last_reviewed: string;
  age_days: number;
}
```

The full audit checklist with rule names and severities is in `02-cell-registry-spec.md`.

## Payload generator contract

Each payload has exactly one canonical generator on the runtime side. The back-end exposes:

```typescript
interface PayloadGenerators {
  brainMap(state: PatientState, opts?: BrainMapOpts): BrainMapPayload;
  subsystemHeatmap(state: PatientState): SubsystemHeatmapPayload;
  residualGap(state: PatientState, threshold?: number): ResidualGapPayload;
  differentialDistance(
    state: PatientState,
    comparator_template_ids: string[]
  ): DifferentialDistancePayload;
  treatmentFit(
    state: PatientState,
    candidate_agent_ids?: string[]
  ): TreatmentFitPayload;
  patientFacing(state: PatientState): PatientFacingPayload;
  audit(): AuditPayload;
}
```

Generators are **pure functions of (PatientState, registry snapshot)**. Same inputs always produce the same payload. This is what makes screenshots reproducible and bugs debuggable.

`audit()` is a registry-only function (no PatientState argument) — it tests the registry's internal consistency.

## Generator implementation notes

### Effective-delta computation

The core computation underlying every patient-state payload is:

```
For each applicable cell c:
  template_delta = composed_template_delta(c, profile.template_refs, severity)
  modifier_delta = subsystem_modifier_contribution(c, profile.subsystem_modifiers)
                 + cell_modifier(c)              # if exists
  treatment_coverage = Σ active_treatments.coverage(c)
  effective_delta = template_delta + modifier_delta
  residual_delta = effective_delta - treatment_coverage
```

`composed_template_delta` is straight delta lookup for single-template profiles, composition rules for comorbid profiles (`03-composition-rules.md`).

### Performance shape

Typical computation: ~50–80 cells per template × 1–3 active templates × dozen modifiers = a few hundred multiplications. No virtualization needed. If cell count grows past ~500, revisit.

Caching: payload generators should cache on `(PatientState.id, registry_version)`. Same inputs → cached payload. Modifier writes invalidate the cache for that profile.

### Determinism requirements

Generators must be:

- **Pure** — no time-of-day variation, no randomness, no external API calls.
- **Versioned** — `registry_version` in metadata identifies which registry export was used.
- **Replayable** — given a stored PatientState and registry version, generate the same payload as the original render.

This is non-negotiable for clinical use. A clinician must be able to ask "what did this look like at the encounter on 2026-03-15?" and get the exact view that drove that day's decision.

## Suggested transport

The payloads are the contract. Transport is downstream — REST, GraphQL, or RPC are all viable. See `13-api-endpoint-derivation.md` for a derived REST surface.

The contracts are **stable across transport**. The same TypeScript types render in JSON over HTTP, MessagePack over WebSocket, or as direct function returns in a monolithic app.

## What this spec does NOT define

- Authentication or PHI handling — architecture concern (`12-app-architecture-decisions.md`).
- Backend API endpoints — the runtime exposes payload generators; HTTP/GraphQL transport is downstream (`13-api-endpoint-derivation.md`).
- Database connection — the runtime reads from the registry export; live database connection is not required.
- The intake form UI — questionnaire rendering and AI extraction review queue are out of scope (covered in `07-ai-extraction-spec.md`).
- Notion integration — the registry export from Notion is a build step, not a runtime concern.
