# 22 — MDD Template (Draft)

> **CLINICAL REVIEW REQUIRED.** This is an AI-assisted first draft. Cell-level deltas, subsystem weights, evidence tiers, and elicitation coefficients have NOT been validated by a clinical advisor. Do not use for patient-facing decisions, treatment ranking, or production renders until a credentialed reviewer has signed off. This document exists to accelerate authoring — every numeric value below should be considered a starting point, not a clinical claim.

The second concrete disorder template after OCD. This document describes the canonical Major Depressive Disorder template (`mdd_canonical_v1`) — its subsystems, cell coverage, and authoring choices specific to MDD's evidence base.

---

## Template metadata

```yaml
id: mdd_canonical_v1
schema_version: "3.0"
disorder: MDD
template_version: "1.0.0-draft"
severity_bucket: moderate            # canonical baseline at moderate severity
phenotype_subtype: null               # broad MDD; melancholic, atypical, and biotype variants live as modifiers
baseline_ref: healthy_v1
elicitation_map_ref: phq-9.v1         # primary; MADRS via madrs.v1 takes precedence when both administered
status: draft
last_reviewed: 2026-05-13
reviewer: framework-team (draft, pending clinical review)
```

The template represents **moderate, unmedicated, trait-baseline MDD** in an adult outpatient. Severe (e.g., melancholic with prominent psychomotor features) and mild (subthreshold persistent depressive presentations) scale via `severity_factor`. The Williams 2024 biotypes — and the older melancholic/atypical/anxious distinctions — are encoded through PatientCellModifiers and PatientSubsystemModifiers, not as separate templates, until the empirical biotype literature stabilizes enough to justify subtype-specific templates.

DSM-5 code: **296.2x / 296.3x** (single episode / recurrent). ICD-11 code: **6A70** (single episode) / **6A71** (recurrent). Paired instrument: PHQ-9 (self-report) and MADRS (clinician-rated). Multi-instrument precedence rule (`04-elicitation-maps.md`) gives MADRS over PHQ-9 when both are within window.

## Subsystems

MDD's four canonical subsystems for v1. Labels mix classical melancholic/atypical phenomenology with the Williams biotype literature, weighted toward what cell-level evidence actually maps onto:

- **A — Anhedonia / reward-circuit dysfunction.** Reduced hedonic capacity, blunted reward learning, motivational anergy. Maps primarily to ventral striatum (vS), VTA, mPFC, OFC and dopaminergic / mesolimbic circuitry. The Pizzagalli reward-deficit framework lives here. Williams' "Positive affect / reward" biotype dimension maps largely to this subsystem.
- **N — Negative affect / threat reactivity.** Sad mood, ruminative negative cognition, threat hypervigilance. Maps to amygdala, subgenual / rostral ACC, vmPFC, hippocampus, and limbic 5-HT signaling. Williams' "Anxious avoidance," "Negative affect," and "Rumination" biotype dimensions all load primarily here. SSRI response is most tightly linked to this subsystem in the SSRI-treatment-prediction literature.
- **C — Cognitive control / executive dysfunction.** Concentration impairment, decision-making slowing, dlPFC hypoactivation, top-down control deficit. Maps to dlPFC, ACC (dorsal), and frontoparietal noradrenergic / glutamatergic signaling. The Williams "Cognitive biotype" maps here.
- **V — Vegetative / somatic.** Sleep, appetite, psychomotor changes, fatigue, HPA-axis dysregulation. Maps to hippocampus, hypothalamic-pituitary inputs (proxied here via hippocampus and LC), Raphe (sleep regulation), and brainstem NE/5-HT. PHQ-9 "somatic" factor loads here.

Every cell carries `subsystem_weights` distributing its contribution across A, N, C, V. Cells without weights — or with weights summing wrong — fail audit rule #1.

Subsystem identity notes:
- A and N are the most cell-evidenced subsystems. Cells in vS, VTA, OFC, vmPFC, amygdala, and subgenual ACC carry strong A or N weights.
- C is well-evidenced at the regional/functional level (dlPFC hypoactivation is one of the most replicated MDD findings) but lighter on neurochemistry-specific cells.
- V is the most heterogeneous and the most clinically heterogeneous across patients. v1 weights are first-pass.

## Cell coverage

48 active cells span:

### Cortical regions

- **OFC** (orbitofrontal cortex) — reward valuation, hedonic representation. 5-HT post-synaptic deficits, glutamatergic abnormalities, metabolic findings. ~5 cells.
- **ACC** (anterior cingulate) — error monitoring, conflict; dorsal (cognitive control) and rostral/subgenual (affective) components. Glx reductions, 5-HT, glucose-metabolism findings. ~6 cells.
- **dlPFC** — top-down control, cognitive interference. NE, DA, Glu/GABA balance. ~5 cells.
- **vmPFC** — fear extinction, self-referential negative processing, reward valuation. 5-HT, Glu. ~4 cells.
- **mPFC** — broad self-referential / default-mode involvement. Glu, GABA. ~3 cells.

### Subcortical / striatal regions

- **vS** (ventral striatum, including NAc) — reward, motivation, anhedonia. D2/D3, DA tone, DA dynamic. ~5 cells.
- **Caudate** — reward-prediction error, instrumental learning; modest involvement in MDD outside reward tasks. ~3 cells.
- **Putamen** — motor and habit; primarily psychomotor symptom carrier. ~2 cells.

### Limbic regions

- **Amygdala** — threat appraisal, negative-affect bias; one of the most replicated regional findings in MDD. 5-HT2A, SERT, GABA-A, glucocorticoid. ~6 cells.
- **Hippocampus** — context, memory, HPA-axis target; volumetric loss replicated meta-analytically. 5-HT1A, GABA, Glu, neurotrophic. ~4 cells.

### Brainstem / regulatory

- **Raphe (DRN)** — serotonergic source; 5-HT1A autoreceptor controversy; SERT. ~3 cells.
- **VTA** — dopaminergic source; firing-rate dynamics from chronic stress models. ~2 cells.
- **LC** — noradrenergic source; α2-adrenergic upregulation finding (Ordway et al. post-mortem); NET. ~3 cells.

### Cells

Cells follow the 5-tuple `(disorder, region, system, target, site)` convention. Display fields shown per cell: `delta_best`, `delta_range`, `tier`, `confidence`, `subsystem_weights`, and primary source(s). Sign convention: positive = elevated relative to healthy baseline; negative = reduced.

#### A subsystem — anhedonia / reward circuit

1. **`mdd.vS.DA.D2/D3.post-syn`** — Reduced D2/D3 receptor binding in ventral striatum during reward anticipation. `delta_best: -2`, `delta_range: [-3, -1]`, tier 1, confidence M, weights `{A: 0.85, N: 0.10, V: 0.05}`. Source: Pizzagalli 2014 review; Pizzagalli 2022 AJP. URLs: [Pizzagalli 2014](https://pmc.ncbi.nlm.nih.gov/articles/PMC3972338/), [Pizzagalli 2022](https://cdasr.mclean.harvard.edu/wp-content/uploads/2022/07/Pizzagalli_AJP22.pdf).
2. **`mdd.vS.DA.tone.tone`** — Reduced tonic dopamine tone in ventral striatum / NAc. `delta_best: -2`, `delta_range: [-3, -1]`, tier 2, confidence M, weights `{A: 0.80, V: 0.20}`. State-trait contested (recovers partially with successful treatment). Sources: Nestler & Carlezon mesolimbic review; Pizzagalli 2014. URL: [Pizzagalli 2014](https://pmc.ncbi.nlm.nih.gov/articles/PMC3972338/).
3. **`mdd.vS.DA.DAT.pre-syn`** — Modestly elevated DAT availability in NAc in subgroups; literature is mixed. `delta_best: +1`, `delta_range: [-1, +2]`, tier 2, confidence L, weights `{A: 0.85, V: 0.15}`, `contested: methodological`. Direction unclear across SPECT vs PET; note ambiguity in NOTES.
4. **`mdd.VTA.DA.dynamic.dynamic`** — Altered phasic firing of VTA DA neurons. Direction is state- and subtype-dependent: increased in chronic-social-defeat susceptibility models, decreased in chronic-unpredictable-mild-stress models. `delta_best: -1`, `delta_range: [-2, +2]`, tier 3, confidence L, weights `{A: 0.75, N: 0.15, V: 0.10}`, `contested: state-trait`. Source: Friedman/Russo 2014 *Nature*; Chaudhury 2013 *Nature*. URL: [Friedman/Russo](https://pmc.ncbi.nlm.nih.gov/articles/PMC3554860/).
5. **`mdd.VTA.DA.firing_rate.dynamic`** — Aggregate VTA DA tonic firing reduced in melancholic-anhedonic subtype. `delta_best: -1`, `delta_range: [-2, 0]`, tier 3, confidence L, weights `{A: 0.85, V: 0.15}`, `contested: subtype`. Source: Russo & Nestler 2013 *Nat Rev Neurosci*.
6. **`mdd.OFC.5HT.5HT2A.post-syn`** — Reduced 5-HT2A binding in OFC; medial OFC implicated in reward valuation. `delta_best: -1`, `delta_range: [-2, 0]`, tier 1, confidence M, weights `{A: 0.55, N: 0.40, V: 0.05}`. Source: Meyer et al. (PET review); Murrough 2011.
7. **`mdd.OFC.DA.tone.tone`** — Reduced DAergic tone in medial OFC consistent with blunted reward valuation. `delta_best: -1`, `delta_range: [-2, 0]`, tier 3, confidence L, weights `{A: 0.85, N: 0.15}`.
8. **`mdd.OFC.Glu.composite.composite`** — Modestly reduced Glx in medial OFC / vmPFC. `delta_best: -1`, `delta_range: [-2, 0]`, tier 1, confidence M, weights `{A: 0.55, N: 0.40, C: 0.05}`. Source: Tao 2025 MRS meta-analysis; Moriguchi 2018 MRS meta-analysis. URLs: [Tao 2025](https://onlinelibrary.wiley.com/doi/10.1155/da/5180077), [Moriguchi 2018](https://www.nature.com/articles/s41380-018-0252-9).
9. **`mdd.Caudate.DA.tone.tone`** — Reduced reward-prediction-error response in caudate. `delta_best: -1`, `delta_range: [-2, 0]`, tier 1, confidence M, weights `{A: 0.85, C: 0.15}`. Source: Pizzagalli 2009 AJP. URL: [Pizzagalli 2009](https://pmc.ncbi.nlm.nih.gov/articles/PMC2735451/).
10. **`mdd.vS.Opioid.MOR.post-syn`** — Reduced mu-opioid receptor signaling on social-reward processing; emerging evidence. `delta_best: -1`, `delta_range: [-2, 0]`, tier 2, confidence L, weights `{A: 0.80, N: 0.20}`. *Note: opioid system is outside v1 system vocabulary; cell flagged as `not-applicable` until opioid is added to system enum, OR remapped onto a DA proxy. Reviewer decision required.*

#### N subsystem — negative affect / threat reactivity

11. **`mdd.Amygdala.5HT.5HT2A.post-syn`** — Altered 5-HT2A binding in amygdala; direction inconsistent across PET studies. `delta_best: +1`, `delta_range: [-1, +2]`, tier 1, confidence M, weights `{N: 0.85, V: 0.15}`, `contested: methodological`. Source: Meyer PET reviews; Bhagwagar 2007.
12. **`mdd.Amygdala.5HT.SERT.pre-syn`** — Reduced amygdala SERT availability. `delta_best: -1`, `delta_range: [-2, 0]`, tier 1, confidence H, weights `{N: 0.85, V: 0.15}`. Source: Gryglewski et al. 2014 SERT meta-analysis. URL: [Gryglewski 2014](https://pmc.ncbi.nlm.nih.gov/articles/PMC4083395/).
13. **`mdd.Amygdala.GABA.GABA-A.post-syn`** — Reduced GABA-A signaling on amygdala interneurons; partial disinhibition contributing to hyperreactivity. `delta_best: -1`, `delta_range: [-2, 0]`, tier 2, confidence L, weights `{N: 0.90, V: 0.10}`.
14. **`mdd.Amygdala.Glu.tone.tone`** — Elevated glutamatergic drive to amygdala under negative-affect challenge. `delta_best: +2`, `delta_range: [+1, +3]`, tier 1, confidence M, weights `{N: 0.95, V: 0.05}`. Source: fMRI reactivity meta-analyses; Murrough/Sanacora glutamate hypothesis. URL: [Frontiers GABA/Glu](https://www.frontiersin.org/journals/psychiatry/articles/10.3389/fpsyt.2021.637863/full).
15. **`mdd.Amygdala.NE.tone.tone`** — Elevated noradrenergic drive to amygdala under stress; α2 / glucocorticoid interactions. `delta_best: +1`, `delta_range: [0, +2]`, tier 2, confidence M, weights `{N: 0.85, V: 0.15}`. Source: van Marle / van Stegeren NE-glucocorticoid model. URL: [Modeling amygdala bias](https://pmc.ncbi.nlm.nih.gov/articles/PMC6671813/).
16. **`mdd.Amygdala.functional.functional.functional`** — Amygdala hyperactivation to negative emotional stimuli, persistent into remission for many patients. `delta_best: +2`, `delta_range: [+1, +3]`, tier 1, confidence H, weights `{N: 0.95, V: 0.05}`. Source: Hamilton 2012 ALE meta-analysis; Lieberz 2024. URL: [Persistent amygdala hyperactivity](https://www.nature.com/articles/s41380-024-02429-4).
17. **`mdd.ACC.5HT.5HT2A.post-syn`** — Reduced 5-HT2A in subgenual / rostral ACC. `delta_best: -1`, `delta_range: [-2, 0]`, tier 1, confidence M, weights `{N: 0.70, A: 0.20, C: 0.10}`.
18. **`mdd.ACC.Glu.composite.composite`** — Reduced Glx in subgenual ACC, replicated meta-analytically; some unmedicated studies show elevation — state-trait. `delta_best: -1`, `delta_range: [-2, +1]`, tier 1, confidence M, weights `{N: 0.55, A: 0.20, C: 0.25}`, `contested: state-trait`. Source: Tao 2025; Moriguchi 2018. URL: [Tao 2025](https://onlinelibrary.wiley.com/doi/10.1155/da/5180077).
19. **`mdd.ACC.functional.functional.functional`** — Subgenual ACC (BA 25) hypermetabolism in treatment-resistant depression; the Mayberg DBS target. `delta_best: +2`, `delta_range: [+1, +3]`, tier 1, confidence H, weights `{N: 0.75, A: 0.20, V: 0.05}`. Source: Mayberg et al. (DBS literature).
20. **`mdd.vmPFC.5HT.5HT1A.post-syn`** — Reduced 5-HT1A post-synaptic binding in vmPFC. `delta_best: -1`, `delta_range: [-2, 0]`, tier 1, confidence M, weights `{N: 0.55, A: 0.30, V: 0.15}`. Source: Drevets / Sargent 5-HT1A literature.
21. **`mdd.vmPFC.Glu.tone.tone`** — Altered vmPFC glutamatergic tone; mixed directionality. `delta_best: -1`, `delta_range: [-2, +1]`, tier 1, confidence M, weights `{N: 0.55, A: 0.40, C: 0.05}`, `contested: methodological`.
22. **`mdd.Hippocampus.5HT.5HT1A.post-syn`** — Reduced hippocampal 5-HT1A binding. `delta_best: -1`, `delta_range: [-2, 0]`, tier 1, confidence M, weights `{N: 0.55, V: 0.30, C: 0.15}`. Source: Drevets 1999; Sargent 2000.
23. **`mdd.Hippocampus.functional.functional.functional`** — Reduced hippocampal volume and neurogenesis-linked function; meta-analytic 8–10% reduction. `delta_best: -2`, `delta_range: [-3, -1]`, tier 1, confidence H, weights `{N: 0.40, V: 0.40, C: 0.20}`. Source: Videbech & Ravnkilde 2004. URL: [Hippocampus meta-analysis](https://psychiatryonline.org/doi/10.1176/appi.ajp.161.11.1957).
24. **`mdd.Raphe.5HT.5HT1A.auto`** — Elevated 5-HT1A autoreceptor binding in raphe of unmedicated MDD, on most-recent multivariate analyses; consistent with reduced raphe firing. `delta_best: +1`, `delta_range: [0, +2]`, tier 1, confidence M, weights `{N: 0.55, A: 0.25, V: 0.20}`, `contested: methodological`. Source: Parsey et al.; recent hierarchical multivariate reanalysis. URLs: [Wang 2016 meta-analysis](https://link.springer.com/article/10.1186/s12888-016-1025-0), [Multivariate reanalysis](https://pubmed.ncbi.nlm.nih.gov/40800447/).
25. **`mdd.Raphe.5HT.SERT.pre-syn`** — Modestly reduced SERT availability at raphe / brainstem. `delta_best: -1`, `delta_range: [-2, 0]`, tier 1, confidence M, weights `{N: 0.50, A: 0.25, V: 0.25}`. Source: Gryglewski 2014. URL: [Gryglewski 2014](https://pmc.ncbi.nlm.nih.gov/articles/PMC4083395/).
26. **`mdd.vmPFC.functional.functional.functional`** — Reduced vmPFC top-down regulation of amygdala. `delta_best: -1`, `delta_range: [-2, 0]`, tier 1, confidence M, weights `{N: 0.75, C: 0.25}`.

#### C subsystem — cognitive control / executive dysfunction

27. **`mdd.dlPFC.functional.functional.functional`** — dlPFC hypoactivation during cognitive control / interference; one of the most replicated MDD findings. `delta_best: -2`, `delta_range: [-3, -1]`, tier 1, confidence H, weights `{C: 0.85, N: 0.15}`. Source: Fales 2008; Williams 2024 cognitive biotype. URLs: [Fales 2008](https://pmc.ncbi.nlm.nih.gov/articles/PMC2825146/), [Williams 2024 Nat Med](https://www.nature.com/articles/s41591-024-03057-9).
28. **`mdd.dlPFC.NE.tone.tone`** — Reduced effective dlPFC noradrenergic tone in cognitive biotype. `delta_best: -1`, `delta_range: [-2, 0]`, tier 2, confidence L, weights `{C: 0.85, V: 0.15}`. Source: Williams BIG-trial α2A agonist work. URL: [BIG study](https://pmc.ncbi.nlm.nih.gov/articles/PMC11888545/).
29. **`mdd.dlPFC.DA.D1.post-syn`** — Reduced D1 signaling on dlPFC pyramidal neurons; inferred from cognitive-PFC literature. `delta_best: -1`, `delta_range: [-2, 0]`, tier 3, confidence L, weights `{C: 0.90, A: 0.10}`. `evidence_status: inferred`.
30. **`mdd.dlPFC.Glu.composite.composite`** — Reduced Glx in lateral PFC; modest meta-analytic effect. `delta_best: -1`, `delta_range: [-2, 0]`, tier 1, confidence M, weights `{C: 0.75, N: 0.25}`. Source: Tao 2025. URL: [Tao 2025](https://onlinelibrary.wiley.com/doi/10.1155/da/5180077).
31. **`mdd.dlPFC.GABA.GABA-A.post-syn`** — Reduced GABA tone in dlPFC; pronounced in treatment-refractory subgroup. `delta_best: -1`, `delta_range: [-2, 0]`, tier 1, confidence M, weights `{C: 0.70, N: 0.30}`. Source: Sanacora et al.; JAMA Psych 2022 prefrontal GABA/Glu. URLs: [Altered connectivity review](https://pmc.ncbi.nlm.nih.gov/articles/PMC6450409/), [JAMA Psychiatry 2022](https://jamanetwork.com/journals/jamapsychiatry/fullarticle/2797208).
32. **`mdd.ACC.functional.dorsal-cognitive.functional`** — Dorsal ACC hypoactivation on cognitive-control demand. `delta_best: -1`, `delta_range: [-2, 0]`, tier 1, confidence M, weights `{C: 0.85, N: 0.15}`.
33. **`mdd.mPFC.Glu.composite.composite`** — Reduced Glx in medial PFC; complementary to ACC finding. `delta_best: -1`, `delta_range: [-2, 0]`, tier 1, confidence M, weights `{C: 0.50, N: 0.40, A: 0.10}`. Source: Moriguchi 2018. URL: [Moriguchi 2018](https://www.nature.com/articles/s41380-018-0252-9).
34. **`mdd.mPFC.GABA.GABA-A.post-syn`** — Reduced mPFC GABA, more pronounced in treatment-refractory MDD. `delta_best: -2`, `delta_range: [-3, -1]`, tier 1, confidence M, weights `{C: 0.50, N: 0.40, A: 0.10}`. Source: Price 2009; Sanacora.

#### V subsystem — vegetative / somatic

35. **`mdd.LC.NE.alpha2A.auto`** — Elevated α2A autoreceptor binding in locus coeruleus, post-mortem; implies premortem NE deficiency. `delta_best: +2`, `delta_range: [+1, +3]`, tier 2, confidence M, weights `{V: 0.55, N: 0.25, C: 0.20}`. Source: Ordway et al. 2003 post-mortem. URL: [Ordway 2003](https://pubmed.ncbi.nlm.nih.gov/12586450/).
36. **`mdd.LC.NE.NET.pre-syn`** — Reduced NET availability in LC region. `delta_best: -1`, `delta_range: [-2, 0]`, tier 2, confidence M, weights `{V: 0.50, C: 0.30, N: 0.20}`. Source: post-mortem LC noradrenergic review. URL: [LC noradrenergic review](https://pmc.ncbi.nlm.nih.gov/articles/PMC3508310/).
37. **`mdd.LC.NE.firing_rate.dynamic`** — Reduced phasic/effective LC firing despite elevated tyrosine hydroxylase (a compensatory failure pattern). `delta_best: -1`, `delta_range: [-2, 0]`, tier 2, confidence M, weights `{V: 0.55, C: 0.30, N: 0.15}`. Source: Frontiers LC α2A model. URL: [α2A LC model](https://pmc.ncbi.nlm.nih.gov/articles/PMC5410613/).
38. **`mdd.Hippocampus.GABA.GABA-A.post-syn`** — Reduced hippocampal GABA-A interneuron function; implicates HPA dysregulation. `delta_best: -1`, `delta_range: [-2, 0]`, tier 2, confidence L, weights `{V: 0.60, N: 0.30, C: 0.10}`.
39. **`mdd.Hippocampus.Glu.tone.tone`** — Elevated extracellular Glu under HPA stress; reflects glucocorticoid-driven excitotoxicity. `delta_best: +1`, `delta_range: [0, +2]`, tier 3, confidence L, weights `{V: 0.55, N: 0.30, C: 0.15}`. `evidence_status: inferred` from animal models with strong human structural correlate.
40. **`mdd.Raphe.5HT.firing_rate.dynamic`** — Reduced raphe 5-HT firing, downstream of 5-HT1A autoreceptor upregulation. `delta_best: -1`, `delta_range: [-2, 0]`, tier 2, confidence M, weights `{V: 0.40, N: 0.40, A: 0.20}`. `evidence_status: inferred`.
41. **`mdd.Putamen.DA.tone.tone`** — Reduced putamenal DA tone in psychomotor-retarded melancholic subtype. `delta_best: -1`, `delta_range: [-2, 0]`, tier 2, confidence L, weights `{V: 0.75, A: 0.25}`, `contested: subtype`. Source: melancholic-depression NAc/striatal literature. URL: [Melancholic NAc connectivity](https://www.sciencedirect.com/science/article/abs/pii/S002839082300388X).
42. **`mdd.Putamen.functional.functional.functional`** — Reduced putamenal activation during motor tasks; psychomotor slowing correlate. `delta_best: -1`, `delta_range: [-2, 0]`, tier 1, confidence M, weights `{V: 0.85, A: 0.15}`, `contested: subtype`.
43. **`mdd.Caudate.DA.D2/D3.post-syn`** — Modestly reduced D2/D3 in caudate; partly subtype-driven. `delta_best: -1`, `delta_range: [-2, 0]`, tier 2, confidence L, weights `{V: 0.35, A: 0.45, C: 0.20}`, `contested: subtype`.

#### Cross-subsystem cells (whole-brain / network-level)

44. **`mdd.mPFC.functional.default-mode.functional`** — Default-mode network hyperconnectivity / sustained self-referential activation. `delta_best: +1`, `delta_range: [0, +2]`, tier 1, confidence M, weights `{N: 0.55, C: 0.30, A: 0.15}`. Source: Hamilton DMN meta-analyses; Williams 2024.
45. **`mdd.ACC.5HT.SERT.pre-syn`** — Reduced ACC SERT availability. `delta_best: -1`, `delta_range: [-2, 0]`, tier 1, confidence M, weights `{N: 0.55, C: 0.20, A: 0.25}`. Source: Gryglewski 2014.
46. **`mdd.vS.5HT.5HT1B.hetero`** — Reduced 5-HT1B heteroreceptor binding on glutamatergic terminals in NAc; reward-modulatory. `delta_best: -1`, `delta_range: [-2, 0]`, tier 2, confidence L, weights `{A: 0.85, N: 0.15}`.
47. **`mdd.OFC.functional.functional.functional`** — Medial OFC hypoactivation during reward valuation. `delta_best: -1`, `delta_range: [-2, 0]`, tier 1, confidence M, weights `{A: 0.85, N: 0.15}`. Source: Pizzagalli 2009; reward-processing meta-analyses.
48. **`mdd.Hippocampus.5HT.SERT.pre-syn`** — Reduced hippocampal SERT availability; smaller effect than amygdala. `delta_best: -1`, `delta_range: [-2, 0]`, tier 1, confidence M, weights `{V: 0.50, N: 0.30, C: 0.20}`. Source: Gryglewski 2014.

## Key authoring choices

### 1. Biotype dimensions encoded as modifiers, not subtype templates

Williams 2024 identifies six biotypes — but they're empirically derived clusters that may not survive replication unchanged, and the cell-level neurobiology for each biotype is still being mapped ([B-SMART-fMRI trial](https://www.nature.com/articles/s44220-024-00271-9), [BIG α2A study](https://pmc.ncbi.nlm.nih.gov/articles/PMC11888545/)). Authoring decision: encode the biotypes as **subsystem-modifier patterns**, not as separate canonical templates. A "cognitive biotype" patient gets a heavier weight on the C subsystem; an "anxious-avoidance" patient on N. When the literature stabilizes (and ideally when replication studies converge), subtype templates can be split off.

This is analogous to OCD's contamination-vs-hoarding choice: keep one canonical template, encode subtype variation in modifiers, revisit when evidence supports cell-level differentiation.

### 2. Melancholic vs atypical also encoded as modifiers

Melancholic depression has well-replicated cell-level findings (heavier vS DA dysfunction, putamenal psychomotor signal, distinct cortisol pattern). Atypical depression's neurobiology is less well-mapped but plausibly differs (reward responsiveness more preserved, higher prevalence of leaden-paralysis features). v1 carries the consensus moderate-MDD finding as `delta_best` and tags subtype-divergent cells with `contested: subtype`. Subtype variation flows through PatientCellModifiers with `phenotype_subtype` evidence.

### 3. The serotonin "umbrella review" caveat

Moncrieff et al. 2022 ([Mol Psych umbrella review](https://www.nature.com/articles/s41380-022-01661-0)) argues the simple "low 5-HT" model of MDD is not strongly supported by direct evidence. v1 takes the position that:

- **Cell-level 5-HT findings are encoded as found** — meta-analytic SERT reduction (especially in amygdala), 5-HT1A autoreceptor elevation, 5-HT2A post-synaptic reductions — with their actual evidence tier and effect-size literature.
- **The aggregate "is MDD a 5-HT disorder" claim is not encoded** because it's a clinical-narrative aggregate, not a cell-level finding. The template lets the cells speak.

This is the same authoring principle as OCD: encode what the literature actually measures, not the higher-order claim.

### 4. State-trait conflicts surfaced explicitly

Several MDD findings differ acutely vs in remission: amygdala hyperactivity (partially recovers), hippocampal volume (may partially recover with successful treatment), Glx in subgenual ACC, raphe firing patterns. These cells are tagged `contested: state-trait` and have wider `delta_range` reflecting the cross-state variability. This was a major v3 schema improvement and MDD makes heavy use of it.

### 5. PHQ-9 vs MADRS precedence

Both instruments score depression severity. v1 elicitation:

- **PHQ-9 is the default screen** because it's free, fast, self-report, and most clinic intake workflows already collect it.
- **MADRS supersedes PHQ-9 when both are within window** because it's clinician-rated and weights anhedonia / cognitive items more heavily.
- The two-factor PHQ-9 (somatic vs cognitive-affective) lets us distribute the score across the V and A/N/C subsystems rather than treating it as a single severity dial.

### 6. Subsystem name choice

A/N/C/V was picked over alternatives (e.g., reward/affect/cognition/somatic, or the Williams six dimensions) because (a) it stays at four subsystems matching OCD's information density, (b) it maps cleanly onto the most-replicated MDD neurobiology, (c) it covers what both PHQ-9 and MADRS actually score. The Williams six dimensions are richer but harder to constrain at v1 with the available cell-level literature.

## Drug coverage cells

**Status: drug coverage cells are not yet populated.** See `11-readiness-and-blockers.md`.

When populated, the priority list is:

- **SSRIs** — sertraline, escitalopram, fluoxetine, citalopram, paroxetine. ~12–15 coverage cells each.
- **SNRIs** — venlafaxine, duloxetine. ~12 cells each.
- **Atypical antidepressants** — bupropion (DA/NE; strong A-subsystem coverage), mirtazapine (α2 antagonist; strong N+V coverage), vortioxetine (multimodal 5-HT).
- **NMDA modulators** — ketamine, esketamine, dextromethorphan-bupropion. ~10 cells each, with strong A and C coverage.
- **TCAs** — clomipramine, nortriptyline, amitriptyline. Older but high-evidence for melancholic subtype.
- **Augmentation agents** — aripiprazole, brexpiprazole, lithium, T3.

Notable open question: how to encode psychotherapy / TMS / ECT coverage. TMS over left dlPFC is a strong C-subsystem intervention with growing evidence; should appear as a coverage entity in v2.

## ElicitationMap reference

The MDD template uses `phq-9.v1` as primary instrument and `madrs.v1` as clinician-rated supersedant. See `04-elicitation-maps.md` for the multi-instrument precedence rule.

### PHQ-9 subsystem mappings (draft coefficients — pilot validation required)

```yaml
id: phq-9.v1
instrument: PHQ-9
applies_to_templates: ["mdd_canonical_v1"]
scoring:
  - name: total
    formula: items[1..9].sum
    range: [0, 27]
  - name: somatic_factor
    formula: items[3,4,5].sum            # sleep, fatigue, appetite (PHQ items 3,4,5)
    range: [0, 9]
  - name: cognitive_affective_factor
    formula: items[6,7,8,9].sum          # self-worth, concentration, psychomotor, suicidal
    range: [0, 12]
  - name: anhedonia_item
    formula: items[1]                     # anhedonia
    range: [0, 3]
  - name: mood_item
    formula: items[2]                     # depressed mood
    range: [0, 3]
subsystem_mappings:
  - scoring_name: anhedonia_item
    template_ref: mdd_canonical_v1
    subsystem: A
    formula: "score * 0.6"                # 0–1.8 modifier range
    evidence_status: inferred
    confidence: M
    rationale: |
      PHQ-9 item 1 directly probes anhedonia ("little interest or pleasure").
      Single-item index of reward-circuit symptom load. Anhedonia is the
      most consistently reward-circuit-linked symptom in the Pizzagalli
      literature; weighting to A is high-confidence directionally though
      the coefficient is a starting calibration.
  - scoring_name: mood_item
    template_ref: mdd_canonical_v1
    subsystem: N
    formula: "score * 0.5"
    evidence_status: inferred
    confidence: M
    rationale: |
      Item 2 captures sad-mood / negative-affect load.
  - scoring_name: cognitive_affective_factor
    template_ref: mdd_canonical_v1
    subsystem: C
    formula: "(score - 4) / 4"            # PHQ cognitive-affective factor 4 → 0, 12 → +2
    evidence_status: inferred
    confidence: M
    rationale: |
      Two-factor PHQ-9 cognitive-affective factor loads concentration, low
      self-worth, psychomotor disturbance, suicidal ideation. Maps onto
      C subsystem (cognitive control / executive) with secondary contribution
      to N. v1 implementation maps it entirely to C; if clinical pilot suggests
      a split is needed (likely for suicidal-ideation item), revisit.
  - scoring_name: somatic_factor
    template_ref: mdd_canonical_v1
    subsystem: V
    formula: "(score - 3) / 3"
    evidence_status: inferred
    confidence: M
    rationale: |
      PHQ somatic factor (sleep, fatigue, appetite) maps to the vegetative
      subsystem. Note: appetite is bidirectional in PHQ — neither direction
      is preferentially weighted at v1.
cell_mappings: []                          # no cell-level overrides at v1 from PHQ alone
recency_window_days: 14
license_status: free-clinical
source_citation: |
  Kroenke K, Spitzer RL, Williams JBW. The PHQ-9: validity of a brief
  depression severity measure. J Gen Intern Med. 2001;16(9):606-613.
  URL: https://pubmed.ncbi.nlm.nih.gov/11556941/
notes: |
  All coefficients are starting calibrations. Validation against pilot
  intake data required before clinical use.
```

### MADRS subsystem mappings (clinician-rated; supersedes PHQ-9)

MADRS items map by content as follows (full instrument-level draft to be authored in `04-elicitation-maps.md`):

- **Apparent sadness, reported sadness, inner tension, pessimistic thoughts, suicidal thoughts** → N subsystem
- **Reduced sleep, reduced appetite** → V subsystem
- **Lassitude, inability to feel** → A subsystem (strong anhedonia loading on "inability to feel")
- **Concentration difficulties** → C subsystem

A single-cell mapping is justified for MADRS item 8 ("inability to feel") → `mdd.vS.DA.D2/D3.post-syn` if score ≥ 4 of 6 — strong content validity for ventral-striatal reward-circuit signal. This is the kind of cell-level mapping that's defensible per `04-elicitation-maps.md` rules.

## Narrative summary

Major depressive disorder is encoded at v1 as a four-subsystem condition with reward-circuit hypofunction (A), limbic / cortical-affective overactivity (N), prefrontal cognitive-control hypoactivity (C), and brainstem / hippocampal vegetative dysregulation (V). The most cell-evidenced findings are amygdala hyperactivity to negative stimuli, dlPFC hypoactivation under cognitive control, reduced ventral-striatal reward signaling (especially in anhedonic and melancholic presentations), reduced amygdala / brainstem SERT availability (Gryglewski meta-analysis), reduced Glx in subgenual ACC and medial PFC (Moriguchi 2018, Tao 2025 meta-analyses), reduced prefrontal GABA (Sanacora, JAMA Psych 2022), elevated α2A binding in locus coeruleus post-mortem (Ordway), and 8–10% reduced hippocampal volume (Videbech & Ravnkilde 2004). The Williams 2024 biotype framework is encoded as differential subsystem weighting rather than separate templates, because the cell-level neurobiology for each biotype is still being mapped. Melancholic and atypical subtypes flow through PatientCellModifiers tagged `contested: subtype` until subtype-specific templates are justified. State-trait conflicts (amygdala, hippocampus, subgenual ACC) are surfaced explicitly via `contested: state-trait`. The serotonin "is MDD a 5-HT disorder" debate is sidestepped: the framework encodes the cell-level 5-HT findings that exist (post-synaptic reductions, transporter availability, autoreceptor elevation) without making the aggregate claim. PHQ-9 is the default elicitation instrument; MADRS supersedes when both are administered. Drug-coverage cells are not yet authored.

## v1 readiness

**What's authored:** 48 disorder cells, 4 subsystems, PHQ-9 + MADRS elicitation sketches, narrative summary, key authoring choices documented, all citations real and URL-linked where possible.

**What needs clinical review before v1 ship:**

- Every `delta_best`, `delta_range`, `tier`, `confidence`, and `subsystem_weight` value. These are AI-drafted from the literature and need a credentialed reviewer to confirm.
- Cell #10 (mu-opioid receptor) flagged as outside the v1 system vocabulary — reviewer must decide to drop it, remap onto DA, or extend the system enum.
- Elicitation coefficients (every formula in PHQ-9 mapping) are starting calibrations needing pilot data.
- State-trait flagged cells (`contested: state-trait`) need a decision: is `delta_best` the acute value or a state-averaged value? Implications for treatment-response visualization.
- Subtype-conditional cells (`contested: subtype`) need a decision on whether to split into melancholic-specific cells via a child template or to keep flattened-with-modifiers.
- Williams biotype representation — is the subsystem-modifier approach sufficient, or does v1.x need explicit biotype subtype templates once replication evidence matures?

**What's missing entirely:**

- Drug coverage cells for SSRIs / SNRIs / atypicals / NMDA modulators / TMS / ECT.
- Composition rules for MDD + comorbid OCD, MDD + GAD, MDD + ADHD (high-prevalence pairs).
- Treatment-coverage cells for psychotherapy (CBT, IPT, behavioral activation) — open framework question of how to represent.

**Status: draft. Do not promote to `active` until clinical advisor sign-off.**

## Sources (primary)

- Williams LM et al. Personalized brain circuit scores identify clinically distinct biotypes in depression and anxiety. *Nature Medicine* 2024. [Link](https://www.nature.com/articles/s41591-024-03057-9)
- Pizzagalli DA. Depression, stress, and anhedonia: toward a synthesis and integrated model. *Annu Rev Clin Psychol* 2014. [Link](https://pmc.ncbi.nlm.nih.gov/articles/PMC3972338/)
- Pizzagalli DA. Toward a better understanding of the mechanisms and pathophysiology of anhedonia. *Am J Psychiatry* 2022. [Link](https://cdasr.mclean.harvard.edu/wp-content/uploads/2022/07/Pizzagalli_AJP22.pdf)
- Pizzagalli DA et al. Reduced caudate and nucleus accumbens response to rewards in unmedicated MDD. *AJP* 2009. [Link](https://pmc.ncbi.nlm.nih.gov/articles/PMC2735451/)
- Gryglewski G et al. Meta-analysis of molecular imaging of serotonin transporters in major depression. *J Cereb Blood Flow Metab* 2014. [Link](https://pmc.ncbi.nlm.nih.gov/articles/PMC4083395/)
- Moriguchi S et al. Glutamatergic neurometabolite levels in MDD: MRS meta-analysis. *Mol Psychiatry* 2018. [Link](https://www.nature.com/articles/s41380-018-0252-9)
- Tao et al. Meta-analysis of MRS glutamatergic neurometabolite levels in MDD. *Depression and Anxiety* 2025. [Link](https://onlinelibrary.wiley.com/doi/10.1155/da/5180077)
- Videbech P & Ravnkilde B. Hippocampal volume and depression: meta-analysis. *AJP* 2004. [Link](https://psychiatryonline.org/doi/10.1176/appi.ajp.161.11.1957)
- Ordway GA et al. Elevated α2-adrenoceptor agonist binding in LC in MDD. *Biol Psychiatry* 2003. [Link](https://pubmed.ncbi.nlm.nih.gov/12586450/)
- Cottingham C & Wang Q. α2A adrenergic receptor dysregulation in depression. *Pharmacol Ther* 2012. [Link](https://pmc.ncbi.nlm.nih.gov/articles/PMC3508310/)
- Friedman AK / Russo SJ et al. Rapid regulation of depression-related behaviors by control of midbrain DA neurons. *Nature* 2014. [Link](https://pmc.ncbi.nlm.nih.gov/articles/PMC3554860/)
- Lieberz J et al. Persistence of amygdala hyperactivity to subliminal negative emotion in long-term MDD course. *Mol Psychiatry* 2024. [Link](https://www.nature.com/articles/s41380-024-02429-4)
- Wang L et al. Serotonin-1A receptor alterations in depression: imaging meta-analysis. *BMC Psychiatry* 2016. [Link](https://link.springer.com/article/10.1186/s12888-016-1025-0)
- Fales CL et al. Antidepressant treatment normalizes dlPFC hypoactivity. 2009. [Link](https://pmc.ncbi.nlm.nih.gov/articles/PMC2825146/)
- Sanacora G & Saricicek A. Altered connectivity in depression: GABA/Glu deficits review. 2019. [Link](https://pmc.ncbi.nlm.nih.gov/articles/PMC6450409/)
- Moncrieff J et al. The serotonin theory of depression: umbrella review. *Mol Psychiatry* 2022. [Link](https://www.nature.com/articles/s41380-022-01661-0) — referenced for the "is MDD a 5-HT disorder" framing question, not as cell-level evidence
- Williams BIG study (α2A agonist for cognitive biotype). [Link](https://pmc.ncbi.nlm.nih.gov/articles/PMC11888545/)
- B-SMART-fMRI (TMS for cognitive biotype). [Link](https://www.nature.com/articles/s44220-024-00271-9)
- Hu et al. Increased NAc functional connectivity in melancholic depression. [Link](https://www.sciencedirect.com/science/article/abs/pii/S002839082300388X)
- Kroenke K et al. PHQ-9 validity. *J Gen Intern Med* 2001. [Link](https://pubmed.ncbi.nlm.nih.gov/11556941/)
