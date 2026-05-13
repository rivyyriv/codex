# 08 — Curated Comorbidity Templates

When a comorbidity is well-characterized in the literature, it gets its own first-class template — not a composition of two single-disorder templates.

---

## Why curated templates exist

The composition rule system (`03-composition-rules.md`) handles arbitrary comorbid pairs by walking each cell and applying the most specific applicable rule. This works for rare or under-studied combinations. But for high-prevalence pairs with strong imaging evidence — OCD+TS, MDD+GAD, ADHD+ASD, BPD+PTSD — the comorbid state is studied directly. The literature reports patterns that aren't recoverable from composition.

Examples:

- **OCD+TS** — Wong 2007 found ventral striatum D2/D3 *increased* in comorbid patients, opposite to either pure form. Neither additive nor ceiling composition would predict this.
- **MDD+GAD** — Treatment response patterns differ from pure MDD. Specific SSRIs (escitalopram, duloxetine) have indications for both; benzodiazepine use is more nuanced.
- **OCD+TS treatment** — First-line is SSRI + low-dose antipsychotic augmentation (aripiprazole, risperidone). Pure OCD is SSRI alone; pure TS is alpha-2 agonists. The comorbid first-line is neither.

Curated templates capture these patterns. They're built when:

- Multiple imaging studies in the comorbid pair exist.
- Treatment response evidence is comorbidity-specific.
- The comorbid pattern is clinically common enough to justify the authoring effort.

## How curated templates are used

Setting `curated_comorbidity_template_ref` on a PatientProfile **replaces** composition. The system uses the curated template directly:

```typescript
{
  patient_id: "...",
  baseline_ref: "healthy_v1",
  template_refs: ["ocd_canonical_v2", "ts_canonical_v1"],  // recorded for provenance
  curated_comorbidity_template_ref: "ocd_plus_ts_v1",       // ← this is what's used
  diagnosis_status: "confirmed",
  // ...
}
```

When `curated_comorbidity_template_ref` is set:

- The runtime uses the curated template's cell vector as the patient's template contribution.
- Composition rules are bypassed.
- Profile can reach `diagnosis_status: confirmed` if instruments support it; no provisional flag from composition gaps.
- The original `template_refs` are kept for provenance (the patient *is* OCD+TS, even if we use the curated template).

## CuratedComorbidityTemplate shape

Identical to `DisorderTemplate` (`01-schema-v3.md`) with three additions:

```typescript
interface CuratedComorbidityTemplate extends DisorderTemplate {
  component_templates: string[];           // template IDs being combined
                                           // e.g. ["ocd_canonical_v2", "ts_canonical_v1"]
  comorbidity_severity: SeverityBucket;     // calibrated for the comorbid state
  treatment_notes: string;                  // first-line for the comorbidity
                                            // (different from composing parent first-lines)
}
```

`cells` are populated independently of the component templates — they're not inherited automatically. Authoring steps:

1. Start from the union of cells in the component templates.
2. For each cell, decide: import unchanged, modify based on comorbid evidence, or remove if not applicable.
3. Add comorbidity-specific cells the parent templates don't have.
4. Author an ElicitationMap that scores both component instruments together.
5. Calibrate severity for the comorbid state.

## v1 curated templates

Status:

| ID | Pair | Status | Notes |
|----|------|--------|-------|
| `ocd_plus_ts_v1` | OCD + Tourette Syndrome | **active** | Fully populated. First-line: SSRI + low-dose antipsychotic. |
| `mdd_plus_gad_v1_draft` | MDD + GAD | draft | Pending MDD canonical template. |
| `ptsd_plus_mdd_v1_draft` | PTSD + MDD | draft | Pending PTSD canonical template. |
| `adhd_plus_asd_v1_draft` | ADHD + ASD | draft | Pending ASD canonical template. |
| `bpd_plus_ptsd_v1_draft` | BPD + PTSD | draft | Pending both parent templates. |

## OCD+TS (`ocd_plus_ts_v1`) — worked example

The only fully populated curated template. Reference for authoring others.

### Component templates

- `ocd_canonical_v2` — moderate baseline OCD
- `ts_canonical_v1` — moderate baseline Tourette Syndrome (note: TS canonical template is itself a v1 draft pending full population)

### Subsystems

The comorbid template inherits subsystems from both parents:

- **D** — Doubt and over-checking (from OCD)
- **F** — Fear of harm / avoidance (from OCD)
- **H** — Habit-related compulsivity (from OCD)
- **T** — Tic-spectrum overlap (from TS, formalized as a primary subsystem)

### Cell pattern

53+ cells, drawn from three sources:

- **Cells imported unchanged from OCD** — OFC 5-HT2A, ACC Glu, amygdala mGluR5, etc. These don't change in the comorbid state. ~30 cells.
- **Cells imported unchanged from TS** — putamen CIN density at −3, ventral striatum phasic DA at +3, motor cortex hyperexcitability cells. ~12 cells.
- **Modified cells where evidence supports a specific comorbid value** — putamen DA tone at +3 (ceiling effect; Wong 2007), ventral striatum D2/D3 at +1 left-lateralized (novel finding; Wong 2007), comorbid-specific corticostriatal connectivity changes. ~10 cells.

### Modified-cell highlights

These are the cells where the curated template visibly diverges from the composition that pairwise rules would produce:

| Cell | OCD alone | TS alone | Composition would give | Curated template |
|------|-----------|----------|------------------------|------------------|
| Putamen DA tone | +2 | +3 | +5 → clipped to +3 | +3 (ceiling) |
| Putamen D2/D3 | −2 | −2 | −4 → clipped to −3 | −3 (ceiling) |
| Ventral Striatum D2/D3 (left) | −2 | −2 | −4 → clipped to −3 | **+1** (novel; Wong 2007) |
| Putamen CIN density | 0 (n/a) | −3 | −3 | **−3** (TS-specific cell now active) |
| Ventral Striatum DA phasic | 0 (n/a) | +3 | +3 | **+3** (TS-specific cell now active) |

The "novel" cells — particularly the ventral striatum D2/D3 inversion — are why the curated template exists. Composition wouldn't get this right.

### ElicitationMap

The curated template uses a comorbidity-specific ElicitationMap that scores Y-BOCS *and* YGTSS together:

- Y-BOCS obsessions → D subsystem (same coefficient as pure OCD)
- Y-BOCS compulsions → F subsystem (same)
- YGTSS motor + phonic → T subsystem (TS-specific)
- Composite score → severity calibration for the comorbid state

This is what allows the comorbid profile to reach `diagnosis_status: confirmed` from instrument scores alone.

### Treatment notes

First-line for OCD+TS is SSRI + low-dose antipsychotic augmentation (aripiprazole 2–5mg, or risperidone 0.5–2mg). Reasoning:

- SSRI covers the OCD-side serotonergic deficits (OFC 5-HT2A, ACC 5-HT, etc.).
- Low-dose antipsychotic covers the TS-side hyperdopaminergic state (putamen DA tone, motor cortex hyperexcitability).
- Combined coverage maps better to the comorbid residual than either monotherapy.

This is *different* from composing parent treatment recommendations. Pure OCD's first-line is SSRI alone. Pure TS's first-line is alpha-2 agonists (clonidine, guanfacine). The comorbid first-line is neither — it's the SSRI from OCD plus a different augmenter than either parent uses.

The framework's TreatmentFitPayload, run against `ocd_plus_ts_v1`, ranks SSRI + low-dose antipsychotic candidates higher than SSRI alone or alpha-2 agonists alone. Pure-form treatment recommendations would miss this.

## Authoring guidance for new curated templates

1. **Confirm the evidence base.** ≥3 imaging studies in the comorbid pair, or strong treatment-response data, or both. Anecdote or single-study findings aren't enough.
2. **Start from union, not sum.** Begin with all cells from both component templates as draft rows. Mark each: import unchanged, modify, or remove.
3. **Name the modifications explicitly.** Each modified cell needs a `notes` field explaining why this cell deviates from composition. Cite sources.
4. **Author the ElicitationMap.** A curated template needs its own map that scores both instruments. Single-disorder maps don't apply directly.
5. **Calibrate comorbid severity.** Severe-OCD + severe-TS isn't necessarily "severe-comorbid." Use comorbid-state validation studies.
6. **Document treatment notes.** First-line for the comorbid state, with the difference from parent first-lines explained.
7. **Mark status `draft` until clinical advisor signs off.** Curated templates ship `active` only after named clinical review.

## When NOT to author a curated template

- The comorbid pair is rare (<5% of either parent's prevalence).
- Imaging evidence is thin (one study, conflicting findings).
- Treatment response evidence doesn't differ from parent first-lines.
- The comorbid pattern is well-captured by a few specific composition rules at the cell level.

In these cases, author composition rules instead. Composition is cheaper to author and easier to update.

## Reference: how the runtime decides composition vs curated

```python
def resolve_template(profile: PatientProfile) -> List[Cell]:
    if profile.curated_comorbidity_template_ref:
        # Curated template exists; use it directly
        return registry.get_template(profile.curated_comorbidity_template_ref).cells
    elif len(profile.template_refs) == 0:
        # Undifferentiated; baseline only
        return registry.get_baseline().cells
    elif len(profile.template_refs) == 1:
        # Single disorder
        return registry.get_template(profile.template_refs[0]).cells
    else:
        # Comorbid; compose
        return compose_templates(
            [registry.get_template(t) for t in profile.template_refs],
            registry.composition_rules
        )
```

`curated_comorbidity_template_ref` is the highest-priority resolution path. When set, no composition logic runs.
