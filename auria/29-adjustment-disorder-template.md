# 29 — Adjustment Disorder Template (Draft)

> **CLINICAL REVIEW REQUIRED.** This is an AI-assisted first draft. Cell-level deltas, subsystem weights, evidence tiers, and elicitation coefficients have NOT been validated by a clinical advisor. Adjustment Disorder (AjD) has a notably smaller cell-level neurobiology evidence base than OCD or MDD, and many cells below are tier-2 / tier-3 inferred from the broader stress-response literature rather than directly evidenced in AjD samples. Do not use for patient-facing decisions, treatment ranking, or production renders until a credentialed reviewer has signed off. The honest read of this template: AjD's cell-level coverage is **substantially weaker** than OCD's or MDD's, and v1 will need to ship with prominent UI cues about the evidence quality.

This document describes the canonical Adjustment Disorder template (`adjustment_disorder_canonical_v1`) — its subsystems, cell coverage, authoring choices, and the explicit acknowledgment of evidence limitations.

---

## Template metadata

```yaml
id: adjustment_disorder_canonical_v1
schema_version: "3.0"
disorder: AjD
template_version: "1.0.0-draft"
severity_bucket: moderate            # canonical baseline at moderate severity
phenotype_subtype: null               # subtypes (with-depressed-mood, with-anxiety, mixed, with-conduct) live as modifiers
baseline_ref: healthy_v1
elicitation_map_ref: adnm-20.v1       # primary; ADNM-8 short form via adnm-8.v1
status: draft
last_reviewed: 2026-05-13
reviewer: framework-team (draft, pending clinical review)
notes: |
  AjD is a stress-response syndrome with a small direct-imaging literature.
  v1 cells are largely tier-2/tier-3 inferred from the broader HPA-axis,
  acute-stress, and prolonged-stress-response neurobiology. The framework
  encodes this honestly: many cells are evidence_status: inferred and the
  narrative summary calls out the evidence asymmetry vs MDD / PTSD.
```

The template represents **moderate, unmedicated, within-stressor-window AjD** in an adult outpatient — symptoms emerging within 1 month of an identifiable stressor (per ICD-11) or 3 months (per DSM-5), expected to remit within 6 months of stressor resolution, not meeting full criteria for MDD, GAD, or PTSD.

DSM-5 code: **309.x** (subtype-suffixed: .0 with depressed mood, .24 with anxiety, .28 with mixed anxiety and depressed mood, .3 with disturbance of conduct, .4 with mixed disturbance of emotions and conduct, .9 unspecified). ICD-11 code: **6B43**. Paired instrument: **ADNM-20** (Adjustment Disorder New Module 20, self-report, validated against ICD-11 criteria), with ADNM-8 short-form fallback.

## Subsystems

AjD's three canonical subsystems for v1. The choice of three (vs OCD's / MDD's four) reflects the smaller evidence base — there's not enough cell-level differentiation to justify a fourth subsystem at v1. The subsystem set is built around the ICD-11 core features (preoccupation with stressor, failure to adapt) and the major phenotypic subtypes (depressed mood, anxious, mixed):

- **P — Preoccupation / rumination.** Stressor-focused intrusive thoughts, recurrent distressing cognitions, inability to disengage from the stressor. Per ICD-11, this is the *required* feature of AjD (which distinguishes it from MDD where preoccupation isn't required). Maps to mPFC, vmPFC, default-mode network connectivity, hippocampus (stressor-memory consolidation), and rostral ACC.
- **S — Stress-response / HPA-axis dysregulation.** Acute and prolonged HPA activation, cortisol-driven structural and functional changes, NE / LC arousal. Maps to amygdala, hippocampus, LC, and brainstem regulatory regions. This subsystem carries most of the V- (vegetative) symptom load (sleep, somatic arousal, autonomic).
- **F — Failure-to-adapt / mood-anxiety surface symptoms.** The phenotypic surface — depressed mood, anxious affect, irritability, behavioral acting-out. This subsystem captures whatever subtype the patient presents with (with-depressed-mood, with-anxiety, mixed, with-conduct disturbance) and serves as the modifier landing site for ADNM subscale scores.

Cell weights distribute across P, S, F. Cells whose primary signal is preoccupation/rumination weight toward P; HPA-axis cells weight toward S; symptom-surface cells (the ones that "look like" MDD or GAD cells but are state-driven rather than trait) weight toward F.

Subsystem identity notes:
- S is the most cell-evidenced subsystem, drawing on the rich HPA-axis / acute-stress / chronic-stress literature applied to AjD by analogy.
- P is grounded in the ICD-11 criterion but cell-level evidence specifically in AjD samples is thin. Most P cells are `evidence_status: inferred` from the stress-rumination and DMN literature.
- F is the thinnest cell-wise — by design, since "failure-to-adapt" is largely a clinical-phenomenology construct and the subtype symptomatology overlaps heavily with MDD / GAD cells.

## Cell coverage

32 active cells span. (Fewer than OCD's 53 or MDD's 48 — an honest reflection of the literature, not a v1 authoring shortfall.)

### Cortical regions

- **mPFC** — default-mode self-referential processing, stressor preoccupation. Glu, GABA, functional. ~4 cells.
- **vmPFC** — top-down regulation of amygdala; stress-response modulation. Functional, 5-HT, Glu. ~3 cells.
- **ACC** — affective dorsal/rostral; stress-affect interface. 5-HT, functional. ~3 cells.
- **OFC** (medial OFC) — emotional processing per the limited AjD fMRI literature. Functional. ~2 cells.
- **dlPFC** — top-down control under stress; modest involvement. Functional, NE. ~2 cells.

### Subcortical / striatal regions

- **vS** — modest involvement; reward-circuit cells primarily relevant to AjD with-depressed-mood subtype. ~2 cells.

### Limbic regions

- **Amygdala** — stress-reactive hyperarousal; primary AjD substrate per stress-imaging literature. GABA-A, NE, glucocorticoid, functional. ~6 cells.
- **Hippocampus** — HPA-axis target; stressor-memory consolidation; structural and functional changes. Glu, GABA, 5-HT, functional. ~5 cells.

### Brainstem / regulatory

- **LC** — noradrenergic arousal; stress-hyperresponse. NE tone, firing. ~3 cells.
- **Raphe** — modest 5-HT signal under acute stress. 5-HT firing. ~1 cell.
- **VTA** — minimal involvement; one cell as placeholder for with-depressed-mood subtype reward signal. ~1 cell.

### Cells

Cells follow the 5-tuple `(disorder, region, system, target, site)` convention. Display fields per cell: `delta_best`, `delta_range`, `tier`, `confidence`, `subsystem_weights`, primary sources, and `evidence_status` since many cells are inferred. Sign convention: positive = elevated relative to healthy baseline; negative = reduced.

#### S subsystem — stress-response / HPA-axis

1. **`adjd.Amygdala.functional.functional.functional`** — Amygdala hyperactivation to stressor-relevant cues; documented in stress-paradigm fMRI studies and reasonable in AjD by analogy. `delta_best: +2`, `delta_range: [+1, +3]`, tier 2, confidence M, weights `{S: 0.70, P: 0.20, F: 0.10}`, `evidence_status: inferred` (inferred from broader stress-response literature applied to AjD). Source: van Marle / van Stegeren stress-amygdala model; Hermans HPA-amygdala work. URL: [Amygdala in stress](https://pmc.ncbi.nlm.nih.gov/articles/PMC5987037/).
2. **`adjd.Amygdala.NE.tone.tone`** — Elevated noradrenergic drive to amygdala during the stress window; well-supported mechanistically. `delta_best: +2`, `delta_range: [+1, +3]`, tier 2, confidence M, weights `{S: 0.85, F: 0.15}`, `evidence_status: inferred`. Source: NE-glucocorticoid amygdala interaction model. URL: [NE-glucocorticoid model](https://pmc.ncbi.nlm.nih.gov/articles/PMC6671813/).
3. **`adjd.Amygdala.GABA.GABA-A.post-syn`** — Reduced GABA-A inhibitory tone on amygdala under stress; partial disinhibition. `delta_best: -1`, `delta_range: [-2, 0]`, tier 3, confidence L, weights `{S: 0.85, F: 0.15}`, `evidence_status: inferred`.
4. **`adjd.Amygdala.Glu.tone.tone`** — Elevated glutamatergic input to amygdala under acute stress; well-supported in rodent models, inferred in humans. `delta_best: +1`, `delta_range: [0, +2]`, tier 3, confidence L, weights `{S: 0.75, P: 0.20, F: 0.05}`, `evidence_status: inferred`. Source: Musazzi / acute-stress Glu model. URL: [Acute stress PFC glutamate](https://www.pnas.org/doi/10.1073/pnas.0906791106).
5. **`adjd.Amygdala.5HT.5HT2A.post-syn`** — Modest 5-HT2A signaling changes under chronic stress; direction inconsistent. `delta_best: 0`, `delta_range: [-1, +1]`, tier 3, confidence L, weights `{S: 0.55, F: 0.45}`, `evidence_status: inferred`, `contested: methodological`. *Note: delta_best of 0 means this cell is functionally inactive at v1 — kept in registry as evidence-gap marker.*
6. **`adjd.LC.NE.tone.tone`** — Elevated LC noradrenergic tone during the stress window. `delta_best: +2`, `delta_range: [+1, +3]`, tier 2, confidence M, weights `{S: 0.80, F: 0.10, P: 0.10}`, `evidence_status: inferred`. Source: stress-LC literature; Aston-Jones & Cohen model.
7. **`adjd.LC.NE.firing_rate.dynamic`** — Elevated LC phasic firing under stressor exposure. `delta_best: +1`, `delta_range: [0, +2]`, tier 3, confidence L, weights `{S: 0.85, F: 0.15}`, `evidence_status: inferred`.
8. **`adjd.LC.NE.alpha2A.auto`** — α2A autoreceptor binding under acute / sub-acute stress; not yet established for AjD specifically but direction inferable from chronic-stress animal models. `delta_best: 0`, `delta_range: [-1, +2]`, tier 3, confidence L, weights `{S: 0.85, F: 0.15}`, `evidence_status: inferred`, `contested: state-trait`. *Note: this is a stark contrast to MDD where post-mortem α2A is elevated — the AjD state is acute/sub-acute and may not show the chronic-deprivation upregulation pattern. Cell kept at 0 with wide range pending evidence.*
9. **`adjd.Hippocampus.Glu.tone.tone`** — Elevated extracellular Glu in hippocampus under glucocorticoid stress; risk for stress-related dendritic atrophy. `delta_best: +1`, `delta_range: [0, +2]`, tier 3, confidence L, weights `{S: 0.80, P: 0.20}`, `evidence_status: inferred`. Source: McEwen stress-hippocampus literature. URL: [Stress neuronal structure](https://pmc.ncbi.nlm.nih.gov/articles/PMC4677120/).
10. **`adjd.Hippocampus.GABA.GABA-A.post-syn`** — Reduced GABA-A interneuron function under stress. `delta_best: -1`, `delta_range: [-2, 0]`, tier 3, confidence L, weights `{S: 0.70, P: 0.30}`, `evidence_status: inferred`.
11. **`adjd.Hippocampus.functional.functional.functional`** — Mild hippocampal functional change with stress; volumetric changes are *not* expected at AjD timescales (those need months-to-years). `delta_best: -1`, `delta_range: [-2, 0]`, tier 3, confidence L, weights `{S: 0.55, P: 0.30, F: 0.15}`, `evidence_status: inferred`. *Note: this cell intentionally does NOT carry the -2/-3 hippocampal-volume delta seen in MDD because the AjD timeframe is too short for atrophy.*
12. **`adjd.Hippocampus.5HT.5HT1A.post-syn`** — Modest 5-HT1A signaling change; unlikely strong AjD effect. `delta_best: 0`, `delta_range: [-1, +1]`, tier 3, confidence L, weights `{S: 0.65, F: 0.35}`, `evidence_status: inferred`.

#### P subsystem — preoccupation / rumination

13. **`adjd.mPFC.functional.default-mode.functional`** — Default-mode network hyperconnectivity reflecting stressor preoccupation and ruminative processing. `delta_best: +2`, `delta_range: [+1, +3]`, tier 2, confidence M, weights `{P: 0.85, F: 0.15}`, `evidence_status: inferred`. Source: stress-rumination DMN literature; AjD-specific evidence sparse. URL: [Stress psychobiology](https://www.tandfonline.com/doi/full/10.1080/15622975.2018.1459049).
14. **`adjd.mPFC.Glu.composite.composite`** — mPFC Glx changes under stress; direction acute (elevated) vs sub-acute (mixed). `delta_best: +1`, `delta_range: [-1, +2]`, tier 3, confidence L, weights `{P: 0.70, S: 0.20, F: 0.10}`, `evidence_status: inferred`, `contested: state-trait`. Source: acute-stress PFC Glu (Yuen, Arnsten). URL: [Acute stress glutamate PFC](https://www.pnas.org/doi/10.1073/pnas.0906791106).
15. **`adjd.mPFC.GABA.GABA-A.post-syn`** — Modestly reduced mPFC GABA tone under prolonged stress. `delta_best: -1`, `delta_range: [-2, 0]`, tier 3, confidence L, weights `{P: 0.65, S: 0.20, F: 0.15}`, `evidence_status: inferred`.
16. **`adjd.vmPFC.functional.functional.functional`** — Reduced vmPFC top-down inhibition of amygdala under stress; allows the amygdala hyperreactivity in cell #1. `delta_best: -1`, `delta_range: [-2, 0]`, tier 2, confidence M, weights `{P: 0.50, S: 0.45, F: 0.05}`, `evidence_status: inferred`. Source: Arnsten chronic-stress PFC review. URL: [Chronic stress PFC](https://journals.sagepub.com/doi/10.1177/24705470211029254).
17. **`adjd.vmPFC.5HT.5HT1A.post-syn`** — Modest 5-HT1A change in vmPFC; AjD-specific evidence absent. `delta_best: 0`, `delta_range: [-1, +1]`, tier 3, confidence L, weights `{P: 0.55, F: 0.45}`, `evidence_status: inferred`.
18. **`adjd.ACC.functional.rostral.functional`** — Rostral ACC reactivity to self-referential stress cues. `delta_best: +1`, `delta_range: [0, +2]`, tier 3, confidence L, weights `{P: 0.65, F: 0.25, S: 0.10}`, `evidence_status: inferred`.
19. **`adjd.OFC.functional.medial.functional`** — Medial OFC structural / functional change reported in limited AjD fMRI (one small study). `delta_best: -1`, `delta_range: [-2, +1]`, tier 3, confidence L, weights `{P: 0.45, F: 0.45, S: 0.10}`, `evidence_status: inferred`, `contested: methodological`. Source: Zhou (small fMRI series cited in AjD review). URL: [AjD psychobiology review](https://www.tandfonline.com/doi/full/10.1080/15622975.2018.1459049). *Note: this is one of the few AjD-direct neuroimaging findings; cell carries the asterisk that the single primary source is a small study.*
20. **`adjd.Hippocampus.Glu.composite.composite`** — Glx in hippocampus reflecting stressor-related encoding load. `delta_best: +1`, `delta_range: [0, +2]`, tier 3, confidence L, weights `{P: 0.65, S: 0.35}`, `evidence_status: inferred`.

#### F subsystem — failure-to-adapt / surface symptoms

21. **`adjd.vmPFC.Glu.tone.tone`** — Glutamatergic tone in vmPFC reflecting affective dysregulation; inferred. `delta_best: 0`, `delta_range: [-1, +1]`, tier 3, confidence L, weights `{F: 0.55, P: 0.30, S: 0.15}`, `evidence_status: inferred`.
22. **`adjd.ACC.5HT.SERT.pre-syn`** — Modest ACC SERT change; AjD-specific evidence absent — inferred placeholder. `delta_best: 0`, `delta_range: [-1, +1]`, tier 3, confidence L, weights `{F: 0.65, P: 0.25, S: 0.10}`, `evidence_status: inferred`.
23. **`adjd.ACC.5HT.5HT1A.post-syn`** — Post-synaptic 5-HT1A change; inferred from broader mood-anxiety literature, not AjD-specific. `delta_best: 0`, `delta_range: [-1, +1]`, tier 3, confidence L, weights `{F: 0.60, P: 0.40}`, `evidence_status: inferred`.
24. **`adjd.dlPFC.functional.functional.functional`** — dlPFC hypoactivation under cognitive control demand in stress. `delta_best: -1`, `delta_range: [-2, 0]`, tier 3, confidence L, weights `{F: 0.55, S: 0.30, P: 0.15}`, `evidence_status: inferred`.
25. **`adjd.dlPFC.NE.tone.tone`** — Stress-altered NE tone in dlPFC; biphasic (acute facilitation → sub-acute decrement under prolonged stress). `delta_best: -1`, `delta_range: [-2, +1]`, tier 3, confidence L, weights `{F: 0.55, S: 0.35, P: 0.10}`, `evidence_status: inferred`, `contested: state-trait`. Source: Arnsten PFC-NE review. URL: [Chronic stress PFC](https://journals.sagepub.com/doi/10.1177/24705470211029254).
26. **`adjd.vS.DA.tone.tone`** — Reduced ventral-striatal DA tone in AjD-with-depressed-mood subtype; in non-depressed AjD subtypes, this cell is closer to 0. `delta_best: -1`, `delta_range: [-2, 0]`, tier 3, confidence L, weights `{F: 0.85, P: 0.15}`, `evidence_status: inferred`, `contested: subtype`.
27. **`adjd.vS.DA.D2/D3.post-syn`** — D2/D3 receptor changes in vS; minimal in AjD vs MDD. `delta_best: 0`, `delta_range: [-1, +1]`, tier 3, confidence L, weights `{F: 0.85, P: 0.15}`, `evidence_status: inferred`, `contested: subtype`.
28. **`adjd.VTA.DA.firing_rate.dynamic`** — VTA firing under sub-acute stress; literature predicts increased phasic firing in susceptible individuals (chronic social defeat model), but AjD is broadly less severe and the direction is uncertain. `delta_best: 0`, `delta_range: [-1, +1]`, tier 3, confidence L, weights `{F: 0.75, S: 0.25}`, `evidence_status: inferred`, `contested: state-trait`.
29. **`adjd.Raphe.5HT.firing_rate.dynamic`** — Raphe 5-HT firing during acute / sub-acute stress; literature mixed. `delta_best: 0`, `delta_range: [-1, +1]`, tier 3, confidence L, weights `{F: 0.45, S: 0.30, P: 0.25}`, `evidence_status: inferred`.
30. **`adjd.Amygdala.5HT.SERT.pre-syn`** — SERT in amygdala in AjD; literature absent — flagged as inferred placeholder. `delta_best: 0`, `delta_range: [-1, 0]`, tier 3, confidence L, weights `{F: 0.55, S: 0.45}`, `evidence_status: inferred`. *Note: contrast with MDD where amygdala SERT is reduced by meta-analysis (Gryglewski). AjD may not show this effect at the disorder's timescale.*
31. **`adjd.Hippocampus.functional.HPA-target.functional`** — Hippocampal HPA-target functional change reflecting glucocorticoid receptor downregulation under sustained stress. `delta_best: -1`, `delta_range: [-2, 0]`, tier 3, confidence L, weights `{S: 0.55, P: 0.30, F: 0.15}`, `evidence_status: inferred`. Source: HPA-axis dysregulation review. URLs: [HPA dysregulation timescale model](https://pmc.ncbi.nlm.nih.gov/articles/PMC7364861/), [HPA review](https://link.springer.com/article/10.1007/s43440-025-00782-x).
32. **`adjd.mPFC.functional.cognitive-control.functional`** — mPFC engagement during reappraisal / emotion regulation; reduced in AjD. `delta_best: -1`, `delta_range: [-2, 0]`, tier 3, confidence L, weights `{F: 0.45, P: 0.40, S: 0.15}`, `evidence_status: inferred`.

## Key authoring choices

### 1. The evidence asymmetry is acknowledged, not hidden

AjD has a smaller direct-imaging literature than OCD, MDD, GAD, or PTSD. There is no large AjD-specific PET or MRS meta-analysis. Most cell-level claims in this template are inferred from one of three sources:

- **HPA-axis / stress-response neurobiology** (animal models with human correlates) — used for S subsystem cells.
- **Acute-stress PFC literature** (Arnsten, Yuen, Musazzi) — used for P / S subsystem cells.
- **Stress-rumination / default-mode literature** — used for P subsystem cells.

The result: **most AjD cells are `tier: 3` and `confidence: L`**. This is correct. A v1 that shipped tier-1 / confidence-H cells for AjD would be misleading. The honesty is intentional and surfaces in visualizations through the reduced-opacity rendering rule (`02-cell-registry-spec.md` §"Reserved field semantics").

### 2. Cells with `delta_best: 0` are kept in registry

Several cells (5, 8, 17, 21, 22, 23, 27, 28, 29, 30) carry `delta_best: 0` — meaning at v1 we have no defensible direction-and-magnitude claim. Rather than dropping these cells, they're kept as **evidence-gap markers**. They satisfy audit rule #1 trivially (sum of weights still equals 1.0 since `delta_best: 0` cells may have weights for distribution-shape consistency, or the audit rule waives the check — implementation decision). The clinical reviewer can promote them to non-zero deltas as evidence accumulates.

### 3. Hippocampal volume cell deliberately omitted

MDD's template carries `mdd.Hippocampus.functional.functional.functional` at -2/-3 reflecting the 8–10% volumetric reduction seen meta-analytically. AjD's analogous cell (#11) sits at -1 with a tighter range, because the AjD timeframe (weeks to months, often resolving within 6 months) is too short for atrophic processes. This was a deliberate authoring decision and is one of the clearest cell-level differentiations between AjD and MDD.

### 4. Subtype variation via modifiers, not subtype templates

AjD's six DSM-5 subtypes (with-depressed-mood, with-anxiety, mixed anxiety-and-depressed, with-conduct, mixed emotion-and-conduct, unspecified) are encoded as PatientSubsystemModifiers on the F subsystem, weighted by ADNM-20 subscale scores. The cell-level neurobiology doesn't (yet) differ enough between AjD-with-anxiety and AjD-with-depressed-mood to justify separate templates. Cells where the subtype matters (#26, #27, #28) are tagged `contested: subtype`.

### 5. State-trait conflicts are pervasive

AjD is **by definition** a state-driven, time-limited condition. Almost every cell is state-rather-than-trait. The decision: don't tag every cell `contested: state-trait` (which would defeat the marker's purpose). Instead, tag only cells where state vs trait direction or magnitude *meaningfully differs* (#8, #14, #25, #28). The template-level metadata makes the global state-driven nature explicit.

### 6. Differential-distance ranking against MDD and GAD is the headline use case

The single most important downstream behavior for AjD is differential distance: "is this MDD, GAD, or AjD?" The template is structured so cell-by-cell comparison against `mdd_canonical_v1` and `gad_canonical_v1` (when authored) produces a sensible distance ranking. Key differentiating cells:

- **Hippocampal volume** — AjD modest, MDD substantial.
- **Subgenual ACC hypermetabolism** — strong MDD signal, weak/absent in AjD.
- **Reward-circuit cells** (vS DA) — strong MDD signal, subtype-only in AjD.
- **Amygdala hyperactivation** — present in both, but AjD's is *stressor-coupled* (decays as the stressor remits), MDD's is more persistent into remission.

The differential-distance primitive (`13-api-endpoint-derivation.md`) is the right vehicle for surfacing these contrasts; the AjD template is authored to feed that primitive cleanly.

### 7. Honest about what AjD is *not*

AjD is not PTSD. The stressor doesn't need to meet the PTSD A-criterion threshold. AjD cells should NOT carry the strongly-replicated PTSD findings (e.g., reduced hippocampal volume from trauma, fear-extinction deficits in vmPFC). Where the AjD literature is silent, cells are inferred from the *general* stress-response literature, not from PTSD-specific findings. This is a notable cell-level differentiation point and will need careful reviewer attention.

## Drug coverage cells

**Status: drug coverage cells are not yet populated.** This is a particularly important gap for AjD because:

- AjD has **no FDA-approved pharmacotherapy** for the disorder per se.
- First-line treatment in most guidelines is **psychotherapy** (CBT, problem-solving therapy, brief supportive therapy), not pharmacotherapy.
- Symptomatic pharmacotherapy is sometimes used (short-course SSRI, short-course benzodiazepine, sleep aids), but the evidence base is sparse and often borrowed from MDD / GAD.

When populated, the priority order is unusual relative to MDD / OCD:

- **Psychotherapy coverage cells** — CBT, problem-solving therapy, brief supportive therapy. The framework's open question of how to encode psychotherapy hits hardest for AjD.
- **Short-course SSRI** — sertraline, escitalopram (low-dose, time-limited). Some evidence for AjD-with-depressed-mood subtype.
- **Short-course anxiolytic** — generally to be avoided (dependence risk, AjD's expected remission), but appears in some guidelines.
- **Sleep aids** — symptomatic.

The framework's treatment ranking for AjD should heavily privilege psychotherapy and watchful waiting. The drug-coverage cell layer needs an explicit non-pharmacological-first policy at the template or composition-rule level.

## ElicitationMap reference

The AjD template uses `adnm-20.v1` as primary instrument. The ADNM-20 (Adjustment Disorder New Module 20) is validated against ICD-11 AjD criteria and has reasonable psychometric properties (87% sensitivity at cut-off 47.5).

### ADNM-20 subsystem mappings (draft coefficients — pilot validation required)

```yaml
id: adnm-20.v1
instrument: ADNM-20
applies_to_templates: ["adjustment_disorder_canonical_v1"]
scoring:
  - name: total
    formula: items[1..20].sum
    range: [20, 80]
  - name: preoccupation_subscale
    formula: items[1..4].sum                # ADNM items 1-4 cover preoccupation
    range: [4, 16]
  - name: failure_to_adapt_subscale
    formula: items[5..8].sum                # items 5-8 cover failure-to-adapt
    range: [4, 16]
  - name: avoidance_subscale
    formula: items[9..12].sum               # items 9-12 cover avoidance
    range: [4, 16]
  - name: depressive_reactions_subscale
    formula: items[13..16].sum              # items 13-16 cover depressive reactions
    range: [4, 16]
  - name: anxiety_subscale
    formula: items[17..20].sum              # items 17-20 cover anxious reactions
    range: [4, 16]
subsystem_mappings:
  - scoring_name: preoccupation_subscale
    template_ref: adjustment_disorder_canonical_v1
    subsystem: P
    formula: "(score - 8) / 4"              # ADNM preoccupation 8 → 0; 16 → +2
    evidence_status: inferred
    confidence: M
    rationale: |
      ADNM-20 preoccupation subscale directly probes intrusive thoughts
      about the stressor (rumination, recurrent distressing thoughts). This
      is the closest instrument-level mapping to the P subsystem.
      ICD-11 specifies preoccupation as the *required* AjD feature, so this
      subscale carries the most diagnostic weight.
  - scoring_name: failure_to_adapt_subscale
    template_ref: adjustment_disorder_canonical_v1
    subsystem: F
    formula: "(score - 8) / 4"
    evidence_status: inferred
    confidence: M
    rationale: |
      Failure-to-adapt items capture functional impairment, inability to
      proceed with normal life. Maps to F subsystem as the surface-symptom
      severity driver.
  - scoring_name: avoidance_subscale
    template_ref: adjustment_disorder_canonical_v1
    subsystem: S
    formula: "(score - 8) / 6"              # smaller coefficient — avoidance is a downstream sign
    evidence_status: inferred
    confidence: L
    rationale: |
      Avoidance is a stress-response symptom but at instrument level it's
      a downstream behavioral indicator rather than direct HPA-axis index.
      Mapped to S with reduced coefficient. Pilot data will determine
      whether to remap to F or split across S+F.
  - scoring_name: depressive_reactions_subscale
    template_ref: adjustment_disorder_canonical_v1
    subsystem: F
    formula: "(score - 8) / 6"
    evidence_status: inferred
    confidence: M
    rationale: |
      Depressive-reactions items load onto F (surface symptoms). When this
      subscale is dominant, the patient is likely AjD-with-depressed-mood
      subtype — flagged for clinician review of subtype assignment.
      *Subtype tag: if depressive_reactions_subscale ≥ 12 AND
      anxiety_subscale < 10, flag with-depressed-mood subtype.*
  - scoring_name: anxiety_subscale
    template_ref: adjustment_disorder_canonical_v1
    subsystem: F
    formula: "(score - 8) / 6"
    evidence_status: inferred
    confidence: M
    rationale: |
      Anxiety items load onto F. When this subscale dominates,
      AjD-with-anxiety subtype is indicated.
      *Subtype tag: if anxiety_subscale ≥ 12 AND
      depressive_reactions_subscale < 10, flag with-anxiety subtype.*
      *Mixed: if both ≥ 10, flag mixed subtype.*
cell_mappings: []                           # no cell-level overrides at v1 from ADNM alone
ai_extraction_targets:
  - pattern_description: |
      Clinician note documents an identifiable stressor (job loss, divorce,
      bereavement, illness, relocation) within 1-3 months of symptom onset.
      Reinforces P subsystem weighting and AjD diagnosis vs MDD.
    cell_ids: ["adjd.mPFC.functional.default-mode.functional",
               "adjd.Amygdala.functional.functional.functional"]
    example_phrasings:
      - "started after layoff three weeks ago"
      - "since her mother's death in February"
      - "after the divorce was finalized"
    evidence_strength_required: explicit_test
  - pattern_description: |
      Clinician note documents preoccupation with the stressor (recurrent
      thoughts, inability to disengage). Per ICD-11 this is the required
      AjD feature. Reinforces P subsystem.
    cell_ids: ["adjd.mPFC.functional.default-mode.functional",
               "adjd.ACC.functional.rostral.functional"]
    example_phrasings:
      - "can't stop thinking about it"
      - "keeps replaying the conversation"
      - "wakes up at 3am thinking about work"
    evidence_strength_required: any
recency_window_days: 14
license_status: free-clinical
source_citation: |
  Lorenz L, Bachem RC, Maercker A. The Adjustment Disorder–New Module 20 as
  a Screening Instrument: Cluster Analysis and Cut-off Values. Int J Occup
  Environ Med. 2016;7(4):215-220.
  URL: https://pubmed.ncbi.nlm.nih.gov/27651082/
  Also: ADNM-20 questionnaire, University of Zurich.
  URL: https://www.psychology.uzh.ch/dam/jcr:15220404-d1b2-4d9a-9661-f1709b4ca3f4/ADNM_20_Homepage_English.pdf
notes: |
  All coefficients are starting calibrations. Validation against pilot
  intake data required before clinical use. The ADNM-20 is validated for
  ICD-11 AjD criteria — DSM-5 AjD profiles may produce somewhat different
  loading patterns. Subtype-tagging rules at the end of mappings are
  implementation hints, not part of the formal ElicitationMap schema.
```

The ADNM-8 short form is available with similar mappings at reduced precision; a separate `adnm-8.v1` map would mirror the structure with fewer scoring subscales.

### Why not PHQ-9 or GAD-7 for AjD elicitation?

PHQ-9 and GAD-7 are cross-disorder instruments and a patient with AjD will produce non-zero scores. Per the cross-disorder questionnaire rule in `04-elicitation-maps.md`, those instruments map only to their primary templates (MDD, GAD). For an AjD-only patient, PHQ-9 / GAD-7 scores are recorded but produce no modifiers until an MDD or GAD template is added. The ADNM instruments are AjD-specific and produce AjD-template modifiers.

For an AjD patient who *also* meets MDD criteria, both templates compose — but at that point the differential-distance primitive should flag whether AjD or MDD is the better-fit single diagnosis. AjD persisting >6 months post-stressor or where MDD criteria are met should generally be re-coded as MDD.

## Narrative summary

Adjustment Disorder is encoded at v1 as a three-subsystem stress-response syndrome — preoccupation with stressor (P), HPA-axis-driven stress-response activation (S), and surface symptoms of failure-to-adapt (F, encompassing the with-depressed-mood / with-anxiety / mixed subtypes). The most cell-evidenced findings are stress-driven amygdala hyperactivation, elevated noradrenergic / locus-coeruleus tone, acute-stress-driven glutamatergic upregulation in mPFC and amygdala, reduced vmPFC top-down regulation, and elevated default-mode connectivity reflecting stressor preoccupation. The bulk of cell-level evidence is **inferred from the broader stress-response, HPA-axis, and rumination neurobiology literature** rather than from AjD-specific imaging studies, which remain sparse. Cell-level findings clearly differentiating AjD from MDD include the absence of substantial hippocampal volume loss (the AjD timescale is too short for atrophy), the absence of subgenual ACC hypermetabolism, and the absence of the chronic-deprivation NE-receptor upregulation pattern seen post-mortem in MDD. Reward-circuit cells (ventral striatum DA) carry only modest deltas, and only in the AjD-with-depressed-mood subtype. The state-driven nature of AjD means almost every cell is acute-state-dependent; cells with materially different state vs trait magnitude are tagged `contested: state-trait` explicitly. The primary instrument is ADNM-20 with five subscales; subtype assignment (with-depressed-mood vs with-anxiety vs mixed) flows from the subscale-balance comparison. The framework deliberately privileges psychotherapy and watchful waiting in treatment ranking — drug coverage cells when authored should reflect that pharmacotherapy is second-line at best. Differential distance against MDD and GAD is the headline downstream use case for this template.

## v1 readiness

**What's authored:** 32 disorder cells (lower count than OCD's 53 or MDD's 48, by design), 3 subsystems (one fewer than OCD/MDD, by design), ADNM-20 elicitation sketch with subtype-tagging hints, narrative summary, key authoring choices documented with explicit evidence-asymmetry acknowledgment, all citations real and URL-linked where possible.

**What needs clinical review before v1 ship:**

- Every `delta_best`, `delta_range`, `tier`, `confidence`, and `subsystem_weight` value. These are AI-drafted, often inferred from broader stress-response literature, and need a credentialed reviewer to confirm.
- Cells at `delta_best: 0` — are these evidence-gap markers worth keeping, or should they be dropped to reduce noise? Schema-level decision needed.
- The audit-rule-#1 question for `delta_best: 0` cells: should `subsystem_weights` still sum to 1.0 for these cells, or is the check waived? Implementation decision.
- The boundary between AjD and prolonged-grief / acute-stress / mild-MDD presentations. The template should not pull in PTSD-specific findings; reviewer needs to confirm no PTSD findings leaked in.
- Treatment-ranking policy: how strongly should psychotherapy be privileged vs pharmacotherapy in AjD recommendations? This is partly a clinical / framework policy question, not just cell-level.
- ADNM-20 elicitation coefficients are starting calibrations needing pilot data validation, especially the subtype-tagging thresholds.

**What's missing entirely:**

- Drug coverage cells (entire layer, with the additional open question of how to represent psychotherapy / brief problem-solving therapy as a "coverage" entity).
- Composition rules for AjD + MDD (when both meet criteria — common clinical scenario), AjD + GAD, AjD + insomnia.
- Time-decay logic for cells. AjD is time-limited; the framework's persistent PatientState model may need a `stressor_resolution_date` or similar to allow cells to attenuate as the stressor resolves. Open framework question, not just template content.
- An explicit "this patient should be re-evaluated for MDD if symptoms persist beyond 6 months post-stressor" decision rule in the visualization layer.

**Honesty note:** AjD's evidence base is thinner than OCD's or MDD's, and the v1 template reflects that honestly via tier-3 cells, low confidence ratings, and pervasive `evidence_status: inferred`. This is the right framing — but the UI / clinician-facing layer must communicate the reduced evidence quality clearly (reduced-opacity rendering per `02-cell-registry-spec.md`, evidence-tier badges on cells, an explicit "evidence base for this disorder is limited" advisory at the disorder-card level).

**Status: draft. Do not promote to `active` until clinical advisor sign-off — and the sign-off bar should be higher for AjD than for OCD / MDD given the more inferential cell-level content.**

## Sources (primary)

- Casey P, Bailey S. Adjustment Disorder: Current Diagnostic Status. *Indian J Psychol Med* 2011. [Link](https://pmc.ncbi.nlm.nih.gov/articles/PMC3701359/)
- O'Donnell ML et al. Adjustment Disorder: Current Developments and Future Directions. *Int J Environ Res Public Health* 2019. [Link](https://www.mdpi.com/1660-4601/16/14/2537)
- Maercker A et al. Adjustment disorder: implications for ICD-11 and DSM-5. *Br J Psychiatry* 2013. [Link](https://www.cambridge.org/core/journals/the-british-journal-of-psychiatry/article/adjustment-disorder-implications-for-icd11-and-dsm5/65DD5402BB0F00F4C9329E97758AA3DA)
- Lorenz L, Bachem RC, Maercker A. The ADNM-20 as a screening instrument. *Int J Occup Environ Med* 2016. [Link](https://pubmed.ncbi.nlm.nih.gov/27651082/)
- Lorenz L et al. Measuring the ICD-11 AjD concept: ADNM validation. 2019. [Link](https://pmc.ncbi.nlm.nih.gov/articles/PMC6877162/)
- McEwen BS et al. Stress effects on neuronal structure: hippocampus, amygdala, PFC. *Neuropsychopharmacology* 2015. [Link](https://pmc.ncbi.nlm.nih.gov/articles/PMC4677120/)
- Woo E, Sansing LH, Arnsten AFT, Datta D. Chronic stress weakens connectivity in PFC. 2021. [Link](https://journals.sagepub.com/doi/10.1177/24705470211029254)
- Yuen EY et al. Acute stress enhances glutamatergic transmission in PFC and facilitates working memory. *PNAS* 2009. [Link](https://www.pnas.org/doi/10.1073/pnas.0906791106)
- van Marle / van Stegeren noradrenergic-glucocorticoid amygdala model. *Neuropsychopharmacology* 2010. [Link](https://pmc.ncbi.nlm.nih.gov/articles/PMC6671813/)
- Roozendaal B et al. Stress-induced functional alterations in amygdala. *Front Cell Neurosci* 2018. [Link](https://pmc.ncbi.nlm.nih.gov/articles/PMC5987037/)
- Karin O et al. New model for HPA axis explains dysregulation on the timescale of weeks. *Mol Syst Biol* 2020. [Link](https://pmc.ncbi.nlm.nih.gov/articles/PMC7364861/)
- Cortisol axis and psychiatric disorders updated review. *Pharmacol Rep* 2025. [Link](https://link.springer.com/article/10.1007/s43440-025-00782-x)
- Carta MG et al. The psychobiology of stress, depression, adjustment disorders and resilience. *World J Biol Psychiatry* 2018. [Link](https://www.tandfonline.com/doi/full/10.1080/15622975.2018.1459049)
- O'Donnell ML et al. Longitudinal study of adjustment disorder after trauma exposure. *AJP* 2016. [Link](https://psychiatryonline.org/doi/10.1176/appi.ajp.2016.16010071)
