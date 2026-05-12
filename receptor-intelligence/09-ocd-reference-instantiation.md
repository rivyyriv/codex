# 09 — OCD Reference Instantiation

The first concrete disorder template. Useful as a worked example for what a populated DisorderTemplate looks like, what authoring decisions get made, and how the framework's primitives apply in practice.

This document describes the canonical OCD template (`ocd_canonical_v2`) — its subsystems, cell coverage, and key authoring choices.

---

## Template metadata

```yaml
id: ocd_canonical_v2
schema_version: "3.0"
disorder: OCD
template_version: "2.0.1"
severity_bucket: moderate            # canonical baseline at moderate severity
phenotype_subtype: null               # template covers OCD broadly; subtypes use modifiers
baseline_ref: healthy_v1
elicitation_map_ref: y-bocs.v1
status: active
last_reviewed: 2026-04-15
reviewer: framework-team
```

The template represents **moderate, unmedicated, trait-baseline OCD**. Severe and mild presentations scale via `severity_factor`; subtype variations (contamination-dominant, hoarding, scrupulosity) come through PatientCellModifier overrides where the literature supports cell-level differentiation.

## Subsystems

OCD's four canonical subsystems:

- **D — Doubt and over-checking.** Intrusive uncertainty, compulsive verification. Maps primarily to OFC, ACC, dorsolateral prefrontal regions and the cortico-striatal-thalamic-cortical (CSTC) loop's serotonergic and glutamatergic inputs.
- **F — Fear of harm / avoidance.** Anxiety, threat appraisal, behavioral inhibition. Maps to amygdala, ventromedial PFC, hippocampus.
- **H — Habit-related compulsivity.** Stimulus-response automation, ritual entrenchment. Maps to dorsal striatum (caudate, putamen) and the habit-learning circuit.
- **T — Tic-spectrum overlap.** Motor and phonic patterns, sensorimotor urges. More prominent in OCD+TS comorbid; minor in pure OCD. Maps to sensorimotor cortex, putamen, GPi/STN.

Every cell carries `subsystem_weights` that distribute its contribution across these four subsystems. Cells without weights — or with weights summing wrong — fail audit rule #1.

## Cell coverage

53 active cells span:

### Cortical regions

- **OFC** (orbitofrontal cortex) — serotonergic post-synaptic deficits (5-HT2A, 5-HT1A, SERT density), glutamatergic findings, metabolic hyperactivity. ~10 cells.
- **ACC** (anterior cingulate) — error-monitoring circuit involvement; 5-HT, Glu, metabolic. ~6 cells.
- **dlPFC** — top-down control deficits; NE, DA, Glu. ~5 cells.
- **vmPFC** — fear extinction circuit; 5-HT, Glu. ~3 cells.

### Subcortical / striatal regions

- **Caudate** — habit and stimulus-response circuit; DA tone, DAT density, metabolic. ~5 cells.
- **Putamen** — motor habit, sensorimotor integration; DA, ACh, GABA. ~5 cells.
- **Ventral Striatum / NAc** — reward and motivation; DA D2/D3, opioid. ~3 cells.

### Limbic regions

- **Amygdala** — threat appraisal; 5-HT2A, GABA-A, mGluR5, NE. ~6 cells.
- **Hippocampus** — context discrimination; 5-HT, GABA. ~3 cells.

### Brainstem / regulatory

- **Raphe (DRN)** — serotonergic source; SERT, 5-HT autoreceptors. ~3 cells.
- **VTA** — dopaminergic source; firing dynamics. ~2 cells.
- **LC** — noradrenergic source; firing tone. ~2 cells.

## Key authoring choices

The OCD template was the pressure-test bed for v2 → v3 schema evolution. Several authoring decisions show up clearly in its cells.

### 1. Subtype-conditional cells were split, not flattened

Original (v1) approach: a single OFC 5-HT2A cell with averaged delta across contamination-OCD, hoarding-OCD, and other subtypes. Pressure test caught this as flattening — averaging conceals subtype-specific findings.

v2 resolution: the OFC 5-HT2A cell carries the consensus finding (contamination-dominant pattern, since it's most-studied). Hoarding-specific deviations are encoded as PatientCellModifiers with `phenotype_subtype` qualification. Future v3+ work may add a `phenotype_subtype` field to the cell itself for explicit subtype templates.

### 2. Density / dynamic / functional / composite sites added

Original schema had only `pre-syn | post-syn | auto | tone` for `site`. OCD literature uses:

- **Cell density measures** — "Cholinergic interneuron count in putamen" → `site: density`
- **Dynamic measures** — "DA cell firing tonic in VTA", "DA release amplitude on amphetamine challenge" → `site: dynamic`
- **Functional output** — "GPi GABAergic output to thalamus" → `site: functional`
- **Composite measures** — "Glu:GABA ratio in striatum" → `site: composite`

The schema was extended to admit these as first-class site values. This was the biggest v2 change driven by OCD instantiation.

### 3. Conflict typology made explicit

OCD has well-known measurement conflicts:

- **ACC GABA** — different studies report different directions depending on MRS protocol. Tagged `contested: methodological`. `delta_best` follows the consensus higher-tier method.
- **Caudate DAT** — varies between studies in remitted vs. acute patients. Tagged `contested: state-trait`. `delta_range` reflects this.

These conflicts surface in BrainMap visualizations as marked cells regardless of magnitude. Clinicians see "we're not certain about this one" without having to read the source list.

### 4. Comorbid-OCD+TS evidence was split out

Wong 2007's comorbid-OCD+TS findings (ventral striatum D2/D3 inversion, putamen CIN density patterns) were originally living in the OCD template as caveats. v3 resolution: these findings live in the OCD+TS curated comorbidity template (`08-comorbidity-templates.md`), not in the pure OCD template. Pure OCD cells reflect pure OCD imaging.

## Drug coverage cells

The OCD template shipped without drug coverage cells in v1; this was identified as a hard blocker for the app. **Status: drug coverage cells are not yet populated.** See `11-readiness-and-blockers.md` §1.

When populated, the priority list is:

- **SSRIs** — sertraline, fluoxetine, escitalopram. ~10–15 coverage cells each (serotonergic post-synaptic, SERT, downstream effects).
- **SNRIs** — venlafaxine, duloxetine. ~10–12 coverage cells each.
- **Glutamatergic augmenters** — memantine, NAC. ~8–10 coverage cells each.
- **Antipsychotic augmenters** — aripiprazole, risperidone (especially relevant for OCD+TS curated template).
- **Other** — clomipramine (older but high-evidence for OCD), clonidine (TS overlap).

Each coverage cell is mechanistically grounded — cited binding affinity data, target densities, or human studies of regional effects.

## ElicitationMap reference

The OCD template uses `y-bocs.v1` as primary instrument. See `04-elicitation-maps.md` for the full Y-BOCS mapping example.

Secondary instruments (not yet in v1 ElicitationMap library):

- **DOCS** (Dimensional Obsessive-Compulsive Scale) — better F-subsystem coverage; 4-subscale structure aligns with OCD subsystems.
- **OCI-R** (Obsessive-Compulsive Inventory-Revised) — broad self-report alternative to Y-BOCS.

Adding DOCS or OCI-R would require ElicitationMap entries and validation against pilot data. Out of scope for v1.

## How a patient profile composes against this template

For a hypothetical Alex, 32, new intake, contamination-dominant OCD, no prior treatment:

1. **Profile creation.** Clinician sets `template_refs: ["ocd_canonical_v2"]`, `severity_per_template: { "ocd_canonical_v2": "moderate" }`. The 53 OCD cells load as Alex's starting profile.
2. **Y-BOCS administration.** Alex scores 24 (moderate-severe). ElicitationMap maps:
   - Obsessions subscale: 14 → D-subsystem modifier `+1`
   - Compulsions subscale: 10 → F-subsystem modifier `+0.3`
3. **Effective deltas computed.** Each OCD cell's effective_delta = template's delta_best × severity_factor + (subsystem_modifier × cell.subsystem_weights). For OFC 5-HT2A (subsystem_weights `{D: 0.7, F: 0.3}`): `-3 × 1.0 + (1.0 × 0.7) + (0.3 × 0.3) = -3 + 0.79 = -2.21` → renders as -2 with a slight upward tick visible in detail panel.
4. **Differential distance ranking** is shown if `diagnosis_status: undifferentiated`, but Alex came in with a clinical OCD impression so the clinician sets `diagnosis_status: provisional` directly.
5. **Treatment fit table** is initially empty (no active treatments). After SSRI initiation, treatment coverage cells subtract from effective deltas, producing a residual map that drives augmentation decisions at follow-up.

## What the OCD template proves

The OCD instantiation is the framework's existence proof. It shows:

- The cell schema accommodates real psychiatric neuroimaging literature without flattening or hand-waving.
- The audit rules catch authoring errors at scale (the v2 → v3 migration surfaced real validation failures).
- The protocol scales — a single disorder is ~50–80 cells, ~2–3 weeks of authoring with the protocol.
- The visualization payloads work — BrainMap, SubsystemHeatmap, and audit dashboards run against the OCD template and produce sensible output.

What it doesn't yet prove:

- Multi-disorder differential distance ranking — needs ≥2 disorder templates.
- Treatment fit calculation — needs drug coverage cells.
- Curated comorbidity templates beyond OCD+TS — needs additional canonical templates.

These are the readiness blockers in `11-readiness-and-blockers.md`. The OCD template is the proof; the remaining work is content authoring, not framework redesign.
