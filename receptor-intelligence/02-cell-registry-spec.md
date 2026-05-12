# 02 — Cell Registry Spec

The cell shape itself — every field, every enum, validation rules, and the audit checklist that runs against the registry.

The cell is the fundamental unit. Disorders, drug coverages, and patient overrides are all expressed in this shape.

---

## The DisorderCell

```typescript
interface DisorderCell {
  // Identifiers
  id: string;                    // "{disorder}.{region}.{system}.{target}.{site}"
  disorder: Disorder;             // enum
  region: Region;                 // enum
  system: System;                 // enum
  target: string;                 // free text — receptor subtype, transporter, cell type
  site: Site;                     // enum

  // What's being measured
  measure_type: MeasureType;
  applicable: boolean;            // false = "not applicable to this disorder"
                                  //         (e.g., a system not implicated)

  // The delta itself
  delta_best: number;             // -3 to +3 integer (display); float internally
  delta_range: [number, number];  // confidence interval / range across studies

  // Evidence metadata
  tier: 1 | 2 | 3;                // evidence tier (1 = strongest)
  confidence: "H" | "M" | "L";    // overall confidence in this cell's value
  evidence_status: EvidenceStatus; // evidenced | inferred | no-data | not-applicable
  contested: Contested;            // null | methodological | subtype | state-trait
  state: CellState;               // active | superseded | deprecated

  // Aggregation weights
  subsystem_weights: Record<string, number>; // e.g. { "D": 0.6, "F": 0.4 }
                                              // sum must equal 1.0 ± 0.01

  // Sources
  sources: Source[];

  // Audit
  last_reviewed: string;           // ISO date
  reviewer: string;
  notes: string;
}

interface Source {
  id: string;       // DOI or PMID
  type: string;      // "PET-imaging", "MRS-human", "post-mortem-immuno",
                    // "RCT", "open-label-trial", "case-series",
                    // "animal-model-rodent", "computational-model", etc.
  tier: 1 | 2 | 3;
  year: number;
  n?: number;        // sample size if applicable
  url?: string;
}

type CellState = "active" | "superseded" | "deprecated";
```

## The DrugCoverageCell

Identical shape to DisorderCell, except keyed on agent rather than disorder. Fields that differ in interpretation:

- `id` is `"{agent}.{region}.{system}.{target}.{site}"`
- `delta_best` represents the **coverage** the agent provides (a counter-vector). Sign matters: an SSRI raising synaptic 5-HT is `+1` to `+2` on serotonergic post-synaptic cells; a D2 antagonist reducing dopamine signaling is `-2` on D2 cells. The runtime subtracts treatment vectors from disorder vectors to compute residual.
- `tier`, `confidence`, `evidence_status` follow the same evidence rules but reference pharmacology literature, not psychiatric phenotype literature.
- `applicable` is `false` for cells outside the agent's mechanism.

```typescript
interface DrugCoverageCell {
  id: string;
  agent: string;                   // e.g. "sertraline", "aripiprazole"
  agent_class: string;              // e.g. "SSRI", "atypical_antipsychotic"
  region: Region;
  system: System;
  target: string;
  site: Site;
  measure_type: MeasureType;
  applicable: boolean;
  delta_best: number;               // coverage vector value
  delta_range: [number, number];
  tier: 1 | 2 | 3;
  confidence: "H" | "M" | "L";
  evidence_status: EvidenceStatus;
  dose_dependence: string;          // "linear" | "ceiling" | "biphasic" | "unknown"
  sources: Source[];
  last_reviewed: string;
  reviewer: string;
  notes: string;
}
```

## Field-by-field semantics

### `id`

Stable across versions. Built from the 5-tuple. Lowercase, dot-separated, alphanumeric. Whitespace not allowed. Used as foreign key in modifiers and composition rules.

Examples:
- `ocd.ofc.5HT.5HT2A.post-syn`
- `ocd.putamen.ACh.CIN.density`
- `sertraline.raphe.5HT.SERT.pre-syn`

### `target`

Free text by design. Cell-population qualifiers go here when needed:

- `5HT2A` — simple receptor subtype
- `D2/D3 (heteromer)` — receptor heteromer
- `GABA-A (on MSNs)` — receptor with cell-population qualifier
- `Adenosine A2A (on D2-bearing MSNs)` — receptor with subpopulation conditioning

Subpopulation-conditioned targets are how the schema avoids flattening cell-type-specific findings while keeping the row keyed on a single tuple.

### `site` — and the four "non-synaptic" sites

The original (v1) schema had only `pre-syn | post-syn | auto | tone`. v2 added four more for measurements that aren't strictly synaptic:

- `density` — cell-density measures (e.g., "Cholinergic interneuron count in putamen")
- `dynamic` — temporal/firing measures ("DA cell firing tonic", "DA release amplitude on amphetamine challenge")
- `functional` — output/projection measures ("GPi GABAergic output to thalamus")
- `composite` — derived measures ("Glu:GABA ratio")

The `hetero` site is also explicitly admitted: heteroreceptors (e.g., `5-HT1B` on glutamatergic terminals) are post-synaptic on a different cell than they regulate. Tag as `post-syn / heteroreceptor` in `target` notes; `site: hetero` if the measurement specifically tracks heteroreceptor function.

### `measure_type`

Categorizes what kind of finding the cell represents:

```
MeasureType:
  receptor_density           // PET binding, autoradiography
  receptor_function          // signaling pathway readout
  transporter_density         // DAT, SERT, NET density
  transporter_function        // amphetamine challenge, etc.
  neurotransmitter_tone        // synaptic concentration / overall tone
  cell_density                // CIN count, MSN count
  firing_rate                 // tonic / phasic dynamics
  output_strength              // projection efficacy
  ratio                       // composite (Glu:GABA, etc.)
```

This field exists so visualizations can group cells by what's actually being measured, not just by anatomy.

### `applicable`

`false` when the cell's tuple doesn't apply to this disorder (e.g., a system that's not implicated). Distinguishes from `evidence_status: no-data` (where it might apply but hasn't been studied).

### `delta_best` and `delta_range`

`delta_best` is the consensus value across studies — typically integer `-3` to `+3`. `delta_range` is `[lower_bound, upper_bound]` reflecting either confidence interval or cross-study variability. Display rounds; internal arithmetic uses floats.

### `tier` and `confidence`

Evidence tier (1 = direct human imaging or clinical trial, 2 = post-mortem or animal-model, 3 = inferred / computational) is independent from confidence (H/M/L). A tier-1 finding can still be M-confidence if studies disagree. A tier-3 finding can't be H-confidence.

### `subsystem_weights`

Distributes the cell's contribution across the disorder's subsystems. For OCD's four subsystems (D, F, H, T), a serotonergic OFC cell might be `{ "D": 0.6, "F": 0.4 }` — primarily contributing to "Doubt and over-checking" with secondary contribution to "Fear of harm."

Weights for any cell with non-zero `delta_best` must sum to 1.0 ± 0.01. Cells with `delta_best == 0` may have empty `subsystem_weights`.

This is how subsystem-level questionnaire modifiers redistribute onto cells. Without subsystem_weights, the modifier-to-cell math doesn't work.

### `state`

- `active` — current canonical entry.
- `superseded` — replaced by a newer cell (e.g., when a study contradicts a prior finding). Kept in registry for provenance; queries filter to `state: active` by default.
- `deprecated` — the cell's tuple is no longer used (e.g., target was renamed in the field; no longer studied).

### `contested`

`null` if uncontested. Otherwise typed:

- `methodological` — different studies disagree because of measurement approach (PET vs MRS)
- `subtype` — value differs by phenotype subtype (contamination-OCD vs hoarding-OCD)
- `state-trait` — value differs by patient state at measurement (acute vs remitted)

Cells with non-null `contested` get a marker in visualizations regardless of magnitude.

### `last_reviewed` / `reviewer`

ISO date and reviewer identity. Cells older than 12 months show in the staleness audit. Cells over 24 months are flagged for re-review.

## Audit checklist (automatable queries)

Every cell must satisfy these. Cells that fail are flagged in the audit dashboard.

| # | Rule | Severity | Query |
|---|------|----------|-------|
| 1 | `subsystem_weights` sums to 1.0 ± 0.01 (when delta_best ≠ 0) | high | `SUM(weights) ∉ [0.99, 1.01]` |
| 2 | `id` is unique across registry | high | `COUNT(*) GROUP BY id HAVING > 1` |
| 3 | `id` follows the 5-tuple pattern | high | regex match |
| 4 | `delta_best` ∈ `delta_range` | high | `delta_best < range[0] OR delta_best > range[1]` |
| 5 | `evidence_status: evidenced` requires ≥1 source | high | `evidence_status = 'evidenced' AND len(sources) = 0` |
| 6 | `tier: 1` cells must have ≥1 tier-1 source | medium | source tier check |
| 7 | `confidence: H` requires `evidence_status: evidenced` | medium | combination check |
| 8 | `applicable: false` cells must have `delta_best = 0` | medium | combination check |
| 9 | `last_reviewed` within 12 months | low | date diff |
| 10 | `contested != null` requires explanation in `notes` | medium | text length check |

The audit checklist runs as queries against the registry. Failures aren't blockers per se — they're gaps surfaced for the clinical team to triage.

## Authoring rules (the protocol)

These are the rules a clinical author follows when adding or editing cells. The audit checklist above is the automated enforcement; the protocol below is the human-facing version.

### Conflict typology

When multiple studies disagree about a cell's value, flatten only when the disagreement is genuinely noise. Otherwise, **split**.

- **Subtype/population structure** → split into separate cells. Don't average. A finding that holds in contamination-OCD but not hoarding-OCD is two cells (or one cell with `phenotype_subtype` qualification on the parent template), not one averaged cell.
- **Methodological disagreement** → flag `contested: methodological`, pick the higher-tier method's value as `delta_best`, encode the disagreement in `delta_range` and `notes`.
- **State-trait** → flag `contested: state-trait`. The measurement's stability across remission/acute is a real finding worth surfacing.

### When in doubt, mark inferred

A cell's `evidence_status` should reflect what's actually in the literature. If a finding is computationally derived from connected cells (e.g., "if input is reduced and the receptor is up-regulated, downstream firing is probably reduced"), the cell is `inferred`, not `evidenced` — even if the inference is sound.

### Single tuple rule

One row per `(disorder, region, system, target, site)` tuple. Heteromers are a single target string with the heteromer notation. Subpopulation cells use the qualifier in `target`. Density measures use `site: density`, not a separate field.

### Source tier weights

When computing aggregate confidence:
- Tier 1 (human PET, MRS, RCT) — weight 1.0
- Tier 2 (post-mortem, animal model with strong human correlate) — weight 0.5
- Tier 3 (animal-only, computational, mechanistic inference) — weight 0.25

Confidence is downstream of source tier, not arbitrary.

### Updates

When new evidence updates a cell's `delta_best`:
1. Don't overwrite the prior cell. Set its `state: superseded`.
2. Create a new cell with the same `id` (the registry permits this — uniqueness check is on `(id, state: active)`).
3. Add the new source. Update `last_reviewed` and `reviewer`.
4. The audit dashboard surfaces the supersession for review.

This preserves provenance. Migration scripts can re-render historical PatientStates against the cell as-it-was at the encounter time.

## Drug coverage cell authoring

Drug coverage cells follow the same rules with three additions:

1. **Mechanism statement required.** The cell's `notes` must contain a one-line mechanism description ("SERT inhibition increases synaptic 5-HT").
2. **Dose-dependence labeled.** The `dose_dependence` field is required: `linear`, `ceiling`, `biphasic`, or `unknown`. Affects how runtime applies coverage at non-canonical doses.
3. **Class inheritance noted.** SSRIs share most coverage cells. The framework permits `agent_class`-keyed coverage cells with per-agent overrides. v1 implementation: each agent has its own full coverage cell set; class-level inheritance is a v2 optimization.

## Cell registry as queryable database

The registry is currently in Notion as a database. The runtime expects it as queryable JSON or Postgres. Migration path:

- Notion remains source-of-truth for clinical reviewers (preserves their editing workflow).
- A build pipeline exports Notion → JSON → Postgres on a schedule (every 6 hours) or on-demand.
- Schema validation runs on every export. Failures block the deployment.

Or: registry migrates to Postgres directly, with a small admin UI for clinical reviewers. See `12-app-architecture-decisions.md` §4 for the trade-off.

## Reserved field semantics for visualization

When the front-end consumes a cell via a payload, these fields are **always present** and have these display meanings:

- `applicable: false` cells render in a different visual state (gray/dashed) regardless of delta.
- `evidence_status: no-data` cells render with reduced opacity.
- `contested != null` cells get a marker (icon, hatching) regardless of magnitude.
- `state != active` cells are filtered from default views; surfaced only in audit dashboard.

These rules are part of the primitive contract (`06-frontend-primitives-spec.md`) and are not optional.
