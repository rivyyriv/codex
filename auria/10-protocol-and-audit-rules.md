# 10 — Protocol and Audit Rules

The authoring protocol clinical reviewers follow, and the audit checklist that runs against the registry as automatable queries.

The protocol is the human-facing contract; the audit checklist is its automated counterpart. Both exist so registry quality is testable, not aspirational.

---

## The protocol — five rules

Every cell added or edited follows these rules. Violations are flagged by audit.

### Rule 1 — Single tuple per cell

One row per `(disorder, region, system, target, site)` tuple. No averaging across regions, no merging across systems, no combining sites that have different evidence.

When the same target shows up in different cell populations (e.g., GABA-A on MSNs vs. on interneurons), the cell-population qualifier goes in the `target` field. The 5-tuple stays unique.

When measurements come from different sites for the same target (e.g., post-synaptic 5-HT2A density vs. functional 5-HT2A signaling), they are separate cells with the same target but different `site` values.

### Rule 2 — Subtype/population structure splits, not flattens

When a finding holds in one subtype but not another, **split into separate rows**, don't average. A finding that's `+2` in contamination-OCD and `0` in hoarding-OCD is two cells (or one cell with subtype-qualified PatientCellModifier overrides). Never `+1` averaged.

The same applies to demographic subgroups (age, sex), state-trait splits (acute vs. remitted), and methodological subgroups (PET vs. MRS findings of the same target).

### Rule 3 — Conflicts are typed and flagged

When studies disagree, classify the conflict:

- **`methodological`** — different measurement approaches give different values. Pick the higher-tier method's value as `delta_best`; encode disagreement in `delta_range` and `notes`.
- **`subtype`** — value differs by phenotype subtype. See Rule 2.
- **`state-trait`** — value differs by patient state at measurement.

Set `contested` accordingly. The registry surfaces contested cells in visualizations regardless of magnitude.

If the conflict is genuinely noise (same method, same subtype, just variance), tag `contested: null` and let `delta_range` capture the spread.

### Rule 4 — Inferred is honest

A cell's `evidence_status` reflects what's actually in the literature.

- **`evidenced`** — direct measurement reported in human studies. Multiple studies preferred; single-study findings can be `evidenced` if tier 1.
- **`inferred`** — computationally derived from connected cells, mechanistic reasoning, or animal-only data extrapolated to human. Even if the inference is sound.
- **`no-data`** — the cell is in the standard inventory but has no published findings for this disorder.
- **`not-applicable`** — the system isn't implicated in this disorder.

Inferred cells can carry `delta_best` values; `confidence` should be `M` or `L`. Not honest about inference status is the easiest way to corrupt the registry. The audit dashboard surfaces inferred cells weighted by what they affect downstream.

### Rule 5 — Updates preserve provenance

When new evidence updates a cell's value:

1. **Don't overwrite.** Set the existing cell's `state: superseded`.
2. **Create a new active cell** with the same `id` (uniqueness is on `(id, state: active)`).
3. **Add the new source.** Update `last_reviewed` and `reviewer`.
4. **Audit dashboard** surfaces the supersession for review.

This preserves provenance. Migration scripts can re-render historical PatientStates against the cell as-it-was at the encounter time. Rule 5 is what makes the registry safe to update in production.

## Source tier weights

When computing aggregate confidence, source tiers weight as:

| Tier | Weight | Examples |
|------|--------|----------|
| 1 | 1.0 | Human PET imaging, MRS, RCT, large-N clinical trial |
| 2 | 0.5 | Post-mortem human, animal model with strong human correlate, open-label trial |
| 3 | 0.25 | Animal-only model, computational simulation, mechanistic inference |

Confidence is downstream of source tier. A cell with only tier-3 sources can't be `confidence: H`. A cell with multiple tier-1 sources should be `confidence: H` unless there's specific reason to discount (small N, contradictory findings).

## Drug coverage authoring additions

Drug coverage cells follow Rules 1–5 with three additions:

### A. Mechanism statement required

Every drug coverage cell's `notes` must contain a one-line mechanism description. "SERT inhibition increases synaptic 5-HT" is sufficient. "It works on serotonin" is not.

### B. Dose-dependence labeled

`dose_dependence` is required: `linear`, `ceiling`, `biphasic`, or `unknown`. The runtime applies coverage at non-canonical doses using this label.

- **linear** — coverage scales linearly with dose up to typical maximum.
- **ceiling** — coverage saturates at a typical clinical dose; higher doses don't add coverage.
- **biphasic** — coverage direction reverses at high dose (rare; used for some atypical agents).
- **unknown** — explicitly mark; runtime treats as ceiling for safety.

### C. Class inheritance noted

Many agents in the same class share most coverage cells. The framework permits class-keyed coverage with per-agent overrides. v1 implementation: each agent has its own full coverage cell set; class-level inheritance is a v2 optimization.

If a per-agent cell is identical to the class default, note `inherits_from_class: true` for future migration.

## Audit checklist (automated queries)

Every audit rule runs as a SQL or pandas query against the registry. Failures populate the AuditPayload (`05-visualization-api-payloads.md`).

| # | Rule | Severity | Failure condition |
|---|------|----------|-------------------|
| 1 | `subsystem_weights_sum_to_one` | high | `\|sum(weights) - 1.0\| > 0.01` and `delta_best != 0` |
| 2 | `unique_id` | high | `count(*) group by id having > 1` (filtered to `state: active`) |
| 3 | `id_format` | high | `id` doesn't match `{disorder/agent}.{region}.{system}.{target}.{site}` |
| 4 | `delta_in_range` | high | `delta_best < range[0] OR delta_best > range[1]` |
| 5 | `evidenced_has_sources` | high | `evidence_status = 'evidenced' AND length(sources) = 0` |
| 6 | `tier1_has_tier1_source` | medium | `tier = 1 AND no tier-1 source in sources[]` |
| 7 | `confidence_h_requires_evidenced` | medium | `confidence = 'H' AND evidence_status != 'evidenced'` |
| 8 | `not_applicable_has_zero_delta` | medium | `applicable = false AND delta_best != 0` |
| 9 | `staleness_12mo` | low | `last_reviewed older than 12 months` |
| 10 | `contested_has_notes` | medium | `contested != null AND length(notes) < 20` |
| 11 | `coverage_cell_has_mechanism` | medium | drug coverage cell with `length(notes) < 10` (mechanism statement check) |
| 12 | `coverage_cell_has_dose_dependence` | high | drug coverage cell with `dose_dependence = null` |
| 13 | `composition_rule_template_pair_consistent` | high | rule's template_a/template_b not in alphabetical order |
| 14 | `composition_rule_novel_has_value` | high | `interaction = 'novel' AND novel_value = null` |
| 15 | `composition_rule_multiplicative_has_factor` | high | `interaction = 'multiplicative' AND factor = null` |

Failures aren't automatic blockers. They populate the audit dashboard for the clinical team to triage. Some staleness is expected; some inferred cells are valid. The dashboard separates "fix this now" (high) from "review this quarter" (medium) from "consider this" (low).

## Registry-build checks (outside JSON Schema)

Two checks run at registry-load time, before any payload generation:

1. **`subsystem_weights` sums to 1.0** for every cell with `delta_best ≠ 0`. (Same as audit rule #1; runs at load to fail fast.)
2. **`id` values unique** across the registry. (Same as audit rule #2.)

These run at boot. A registry that fails these can't be served — the runtime refuses to start. This is intentional: subtle weight or ID errors corrupt every downstream payload.

## Provisional flag policy

A PatientProfile is marked `diagnosis_status: provisional` when:

- Composition is in use (multi-template) AND any cell falls through to global default rule.
- Composition is in use AND any applied rule has `confidence: L` or `evidence_status: inferred`.
- Composition is in use AND any applied rule has `interaction: unknown`.
- A clinician explicitly sets it (during initial intake before workup is complete).

Promotion to `confirmed` requires explicit clinician affirmation. Provisional → confirmed is an audited event; the ProfileEditLog records timestamp and clinician identity.

`provisional` profiles can still produce all payloads — they just carry a flag visible in the UI ("This profile uses composed disorder data; some cells aren't fully evidenced for the comorbid pattern").

## Audit dashboard — recommended cadence

For pilot and active clinical use:

- **Daily** — high-severity failures. Auto-page clinical team if failures exceed threshold.
- **Weekly** — medium-severity review. One reviewer's session reviews accumulated medium failures.
- **Monthly** — low-severity (staleness) review. Quarterly clinical review of cells nearing 12-month staleness.
- **Per-template-update** — full audit run before promoting a new template version from `draft` to `active`.

## Reviewer responsibility

Every cell has a `reviewer` field. The reviewer is the clinician (or framework team member) who last affirmed the cell's value. Multiple reviewers can have edited a cell over its lifetime; the audit log preserves the chain.

For active clinical deployment, a designated **clinical advisor** (named psychiatrist) signs off on every active-status template before deployment. The advisor's identity is on the template's `reviewer` field. Their review cadence (per-disorder template, quarterly) is set by the project's clinical advisory framework — not specified by this framework.

This is one of the six app-layer decisions in `12-app-architecture-decisions.md`.
