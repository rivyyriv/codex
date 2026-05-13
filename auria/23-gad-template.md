# 23 — GAD Template (Generalized Anxiety Disorder)

> **CLINICAL REVIEW REQUIRED.** This file is a first-draft DisorderTemplate authored by the framework team for v1 review. Cell deltas, subsystem weights, and elicitation coefficients are starting calibrations grounded in the cited literature but have not been clinician-committed. Do not deploy to patient profiles before a clinical reviewer signs off on `last_reviewed` / `reviewer` fields and a pilot calibration pass against intake data is complete.

A canonical DisorderTemplate for moderate, unmedicated, trait-baseline Generalized Anxiety Disorder. Mirrors the structure, depth, and tone of `09-ocd-reference-instantiation.md`.

---

## Template metadata

```yaml
id: gad_canonical_v1
schema_version: "3.0"
disorder: GAD
template_version: "1.0.0"
dsm5_code: "300.02"
icd11_code: "6B00"
severity_bucket: moderate            # canonical baseline at moderate severity
phenotype_subtype: null               # template covers GAD broadly
baseline_ref: healthy_v1
elicitation_map_ref: gad-7.v1
brain_type_anchor: type_1            # Anxious-Vigilant (Negative Affect Reactive)
status: draft
last_reviewed: 2026-05-13
reviewer: framework-team
notes: |
  GAD presents with persistent, generalized worry, prominent somatic
  hyperarousal, and impaired top-down regulation. The template targets the
  worry/DMN component, the limbic threat-monitoring component, the cortical
  GABA/Glu imbalance, and the sleep/autonomic dysregulation. Brain Type 1
  (Anxious-Vigilant) is the patient-identity anchor.
```

The template represents **moderate, unmedicated, trait-baseline GAD**. Severe and mild presentations scale via `severity_factor`; comorbid-MDD or comorbid-panic presentations are handled by composition or — if pattern is well-established — curated comorbidity templates (not in v1).

## Subsystems

GAD's four canonical subsystems:

- **W — Worry / cognitive perseveration.** DMN-dominant rumination, future-focused threat simulation, difficulty disengaging. Maps to mPFC, ACC, posterior cingulate, hippocampus — the worry/DMN axis (Mochcovitch 2014, Etkin 2009).
- **H — Hyperarousal / autonomic.** Elevated sympathetic tone, muscle tension, sleep disturbance. Maps to LC, amygdala, insula, brainstem NE/5HT sources.
- **I — Intolerance of uncertainty / threat appraisal.** Amygdala-led threat overweighting with impaired top-down regulation. Maps to amygdala, vmPFC, insula, ACC — the threat-appraisal circuit.
- **R — Regulatory deficit.** Cortical GABA depletion and impaired prefrontal control. Maps to dlPFC, mPFC, ACC GABA/Glu balance and BZ-receptor density.

Every cell carries `subsystem_weights` distributing its contribution across {W, H, I, R}. Weights for any non-zero cell sum to 1.0 ± 0.01.

## Cell coverage

47 active cells. Coverage by region:

### Cortical regions

- **mPFC** (medial prefrontal cortex) — DMN hub; reduced top-down inhibition of amygdala; hyperactivity during induced worry; elevated dACC/dmPFC at rest in GAD worriers (Andreescu 2015). 6 cells.
- **ACC** (anterior cingulate) — error-monitoring and conflict; reduced GABA on MRS (Hasler 2008, Long 2013); elevated Glu in some studies; persistent post-worry activation. 5 cells.
- **dlPFC** — top-down regulation deficit; reduced engagement during cognitive reappraisal. 3 cells.
- **vmPFC** — fear-extinction failure; reduced engagement during safety learning (Greenberg 2013). 3 cells.
- **OFC** — value/threat appraisal; modest serotonergic involvement, less prominent than in OCD. 2 cells.

### Limbic regions

- **Amygdala** — hyperreactivity to threat cues; elevated 5-HT2A signaling; reduced 5-HT1A binding; reduced GABA-A/BZ binding; reduced connectivity with mPFC (Etkin 2010, Mochcovitch 2014). 8 cells.
- **Hippocampus** — context discrimination; GR sensitivity changes from chronic stress; reduced 5-HT1A binding in some studies. 4 cells.

### Subcortical / autonomic

- **vS (ventral striatum)** — anticipatory anxiety and reward-anxiety interaction; modest involvement, more prominent in comorbid MDD. 2 cells.

### Brainstem / regulatory

- **Raphe (DRN)** — serotonergic source; elevated tonic 5-HT to amygdala/PAG, altered 5-HT1A autoreceptor signaling. 4 cells.
- **LC (locus coeruleus)** — noradrenergic source; elevated tonic NE firing driving hyperarousal (Goddard 2017). 4 cells.
- **VTA** — dopaminergic source; minimal direct involvement in pure GAD. 1 cell (low-confidence inferred).

### Cortical-wide / functional

- **Cortex (GABA / Glu composite)** — reduced cortical GABA on MRS, particularly in mPFC and ACC (Hasler 2008, Long 2013); reduced cortical BZ binding (Tiihonen 1997 fractal analysis of BZ binding). 5 cells encoded as region-specific composites.

## Cell registry (excerpted)

The full 47 cells live in the registry database. Below is the representative set surfacing the key authoring decisions. Each cell carries the 5-tuple ID, delta_best, delta_range, tier, confidence, evidence_status, contested status, subsystem_weights, and at least one citation.

| Cell ID | Region | System | Target | Site | δ | range | tier | conf | weights {W,H,I,R} | sources |
|---|---|---|---|---|---|---|---|---|---|---|
| gad.amygdala.5HT.5HT2A.post-syn | Amygdala | 5HT | 5HT2A | post-syn | +2 | [+1,+3] | 2 | M | {0.10, 0.30, 0.50, 0.10} | Akimova 2009 |
| gad.amygdala.5HT.5HT1A.post-syn | Amygdala | 5HT | 5HT1A | post-syn | −2 | [−3,−1] | 1 | M | {0.10, 0.30, 0.50, 0.10} | Lanzenberger 2007 |
| gad.amygdala.GABA.GABA-A/BZ.post-syn | Amygdala | GABA | GABA-A/BZ-site | post-syn | −2 | [−3,−1] | 2 | M | {0.05, 0.40, 0.30, 0.25} | Tiihonen 1997; Kalueff 2007 |
| gad.amygdala.Glu.mGluR5.post-syn | Amygdala | Glu | mGluR5 | post-syn | +1 | [0,+2] | 3 | L | {0.10, 0.30, 0.40, 0.20} | Holmes 2017 review |
| gad.amygdala.NE.alpha1.post-syn | Amygdala | NE | α1-adrenergic | post-syn | +2 | [+1,+3] | 2 | M | {0.05, 0.50, 0.30, 0.15} | Goddard 2017 |
| gad.amygdala.NE.tone.tone | Amygdala | NE | NE tone | tone | +2 | [+1,+3] | 2 | M | {0.05, 0.50, 0.30, 0.15} | Goddard 2017 |
| gad.amygdala.functional.threat-reactivity.functional | Amygdala | Composite | Threat-cue reactivity (BOLD) | functional | +3 | [+2,+3] | 1 | H | {0.10, 0.20, 0.60, 0.10} | Etkin 2009; Mochcovitch 2014 |
| gad.amygdala.functional.mPFC-connectivity.functional | Amygdala | Composite | Amygdala–mPFC functional connectivity | functional | −2 | [−3,−1] | 1 | H | {0.20, 0.20, 0.40, 0.20} | Etkin 2010 |
| gad.mPFC.5HT.5HT1A.post-syn | mPFC | 5HT | 5HT1A | post-syn | −1 | [−2,0] | 2 | L | {0.40, 0.10, 0.20, 0.30} | Nash 2008 (panic, generalizing) |
| gad.mPFC.GABA.tone.tone | mPFC | GABA | GABA tone (MRS) | tone | −2 | [−3,−1] | 1 | M | {0.30, 0.10, 0.20, 0.40} | Hasler 2008 (MRS pilot) |
| gad.mPFC.Glu.tone.tone | mPFC | Glu | Glu tone (MRS) | tone | +1 | [0,+2] | 2 | L | {0.40, 0.10, 0.20, 0.30} | Mathew 2008 |
| gad.mPFC.functional.worry-induction.functional | mPFC | Composite | Worry-induction BOLD activity | functional | +2 | [+1,+3] | 1 | H | {0.60, 0.10, 0.10, 0.20} | Andreescu 2015 |
| gad.mPFC.functional.DMN-connectivity.functional | mPFC | Composite | DMN intra-network connectivity | functional | +2 | [+1,+3] | 1 | H | {0.60, 0.05, 0.10, 0.25} | Andreescu 2015; Coplan 2012 |
| gad.mPFC.functional.regulation.functional | mPFC | Composite | Top-down regulation of amygdala (PPI) | functional | −2 | [−3,−1] | 1 | M | {0.20, 0.10, 0.20, 0.50} | Etkin 2010 |
| gad.ACC.5HT.5HT1A.post-syn | ACC | 5HT | 5HT1A | post-syn | −1 | [−2,0] | 2 | L | {0.30, 0.15, 0.25, 0.30} | Akimova 2009 |
| gad.ACC.GABA.tone.tone | ACC | GABA | GABA tone (MRS) | tone | −2 | [−3,−1] | 1 | M | {0.30, 0.10, 0.20, 0.40} | Long 2013; Hasler 2008 |
| gad.ACC.Glu.tone.tone | ACC | Glu | Glu tone (MRS) | tone | +1 | [0,+2] | 2 | L | {0.30, 0.10, 0.20, 0.40} | Mathew 2008 |
| gad.ACC.functional.error-monitoring.functional | ACC | Composite | Conflict/error-monitoring BOLD | functional | +2 | [+1,+3] | 2 | M | {0.30, 0.10, 0.30, 0.30} | Etkin 2009 |
| gad.ACC.composite.glu-gaba.composite | ACC | Composite | Glu:GABA ratio | composite | +2 | [+1,+3] | 2 | M | {0.30, 0.10, 0.20, 0.40} | Long 2013 |
| gad.dlPFC.GABA.tone.tone | dlPFC | GABA | GABA tone (MRS) | tone | −1 | [−2,0] | 2 | L | {0.20, 0.05, 0.15, 0.60} | Hasler 2008 |
| gad.dlPFC.functional.reappraisal.functional | dlPFC | Composite | Cognitive reappraisal engagement | functional | −2 | [−3,−1] | 1 | M | {0.20, 0.05, 0.15, 0.60} | Ball 2013 |
| gad.dlPFC.NE.alpha2.post-syn | dlPFC | NE | α2A-adrenergic | post-syn | −1 | [−2,0] | 3 | L | {0.10, 0.20, 0.10, 0.60} | Arnsten 2009 (inferred) |
| gad.vmPFC.5HT.5HT1A.post-syn | vmPFC | 5HT | 5HT1A | post-syn | −1 | [−2,0] | 2 | L | {0.20, 0.10, 0.40, 0.30} | Akimova 2009 |
| gad.vmPFC.functional.extinction.functional | vmPFC | Composite | Fear-extinction engagement | functional | −2 | [−3,−1] | 2 | M | {0.10, 0.10, 0.60, 0.20} | Greenberg 2013 |
| gad.vmPFC.Glu.tone.tone | vmPFC | Glu | Glu tone (MRS) | tone | +1 | [0,+2] | 3 | L | {0.20, 0.10, 0.40, 0.30} | inferred |
| gad.OFC.5HT.5HT2A.post-syn | OFC | 5HT | 5HT2A | post-syn | +1 | [0,+2] | 3 | L | {0.30, 0.10, 0.40, 0.20} | inferred from anxiety-spectrum |
| gad.OFC.functional.threat-valuation.functional | OFC | Composite | Threat-valuation engagement | functional | +1 | [0,+2] | 2 | L | {0.20, 0.10, 0.50, 0.20} | Milad 2014 |
| gad.hippocampus.5HT.5HT1A.post-syn | Hippocampus | 5HT | 5HT1A | post-syn | −1 | [−2,0] | 2 | M | {0.30, 0.15, 0.30, 0.25} | Akimova 2009 |
| gad.hippocampus.GABA.tone.tone | Hippocampus | GABA | GABA tone | tone | −1 | [−2,0] | 3 | L | {0.30, 0.15, 0.30, 0.25} | inferred |
| gad.hippocampus.Glu.NMDA.post-syn | Hippocampus | Glu | NMDA | post-syn | +1 | [0,+2] | 3 | L | {0.30, 0.15, 0.30, 0.25} | inferred (chronic stress) |
| gad.hippocampus.functional.context-discrimination.functional | Hippocampus | Composite | Context vs threat discrimination | functional | −1 | [−2,0] | 2 | M | {0.20, 0.10, 0.50, 0.20} | Greenberg 2013 |
| gad.vS.DA.D2.post-syn | vS | DA | D2/D3 | post-syn | 0 | [−1,+1] | 3 | L | {0.20, 0.20, 0.40, 0.20} | applicable but minimal in pure GAD |
| gad.vS.functional.anticipatory-anxiety.functional | vS | Composite | Anticipatory-anxiety BOLD | functional | +1 | [0,+2] | 2 | L | {0.30, 0.20, 0.40, 0.10} | Nitschke 2009 |
| gad.raphe.5HT.5HT1A.auto | Raphe | 5HT | 5HT1A | auto | +1 | [0,+2] | 2 | M | {0.20, 0.30, 0.30, 0.20} | Akimova 2009 |
| gad.raphe.5HT.SERT.pre-syn | Raphe | 5HT | SERT | pre-syn | 0 | [−1,+1] | 2 | L | {0.20, 0.30, 0.30, 0.20} | Maron 2011 (contested) |
| gad.raphe.5HT.tone.tone | Raphe | 5HT | 5HT tone to amygdala/PAG | tone | +1 | [0,+2] | 2 | M | {0.10, 0.30, 0.40, 0.20} | Goddard 2017 review |
| gad.raphe.5HT.dynamic.dynamic | Raphe | 5HT | DRN firing dynamics | dynamic | +1 | [0,+2] | 3 | L | {0.10, 0.30, 0.40, 0.20} | inferred |
| gad.LC.NE.alpha2.auto | LC | NE | α2A-adrenergic | auto | −1 | [−2,0] | 2 | M | {0.10, 0.50, 0.20, 0.20} | Goddard 2017 |
| gad.LC.NE.tone.tone | LC | NE | NE tone (projection) | tone | +2 | [+1,+3] | 1 | M | {0.05, 0.60, 0.20, 0.15} | Goddard 2017; Charney 1990 |
| gad.LC.NE.dynamic.dynamic | LC | NE | LC tonic firing | dynamic | +2 | [+1,+3] | 2 | M | {0.05, 0.60, 0.20, 0.15} | Bremner 1996 (review) |
| gad.LC.NE.NET.pre-syn | LC | NE | NET | pre-syn | 0 | [−1,+1] | 3 | L | {0.05, 0.60, 0.20, 0.15} | inferred |
| gad.VTA.DA.dynamic.dynamic | VTA | DA | VTA firing dynamics | dynamic | 0 | [−1,+1] | 3 | L | {0.20, 0.30, 0.40, 0.10} | inferred; minimal in pure GAD |
| gad.cortex.GABA.GABA-A/BZ.density | Cortex (composite) | GABA | GABA-A/BZ-receptor density | density | −2 | [−3,−1] | 1 | M | {0.20, 0.20, 0.20, 0.40} | Tiihonen 1997 (BZ binding); Bremner 2000 |
| gad.cortex.GABA.tone.tone | Cortex (composite) | GABA | Cortical GABA tone (MRS, aggregate) | tone | −2 | [−3,−1] | 1 | M | {0.20, 0.15, 0.15, 0.50} | Hasler 2008; Long 2013 |
| gad.cortex.5HT.SERT.density | Cortex (composite) | 5HT | SERT density (cortical aggregate) | density | −1 | [−2,0] | 2 | L | {0.25, 0.25, 0.30, 0.20} | Maron 2011 |
| gad.cortex.composite.dmn-tpn.composite | Cortex (composite) | Composite | DMN vs task-positive switching efficiency | composite | −2 | [−3,−1] | 1 | M | {0.60, 0.05, 0.10, 0.25} | Andreescu 2015; Coplan 2012 |
| gad.cortex.Glu.tone.tone | Cortex (composite) | Glu | Cortical Glu tone (aggregate MRS) | tone | +1 | [0,+2] | 2 | L | {0.30, 0.10, 0.20, 0.40} | Mathew 2008 |

Subsystem weights cross-check: each row sums to 1.0 ± 0.01.

## Key authoring choices

### 1. GABA findings encoded at both regional and cortical-composite scope

MRS GABA findings in GAD are region-specific (mPFC, ACC, dlPFC) but the BZ-binding literature (Tiihonen 1997 PET) is reported as a cortex-wide pattern. Rather than flatten or invent regional BZ-binding cells the literature doesn't support, the cortex-composite cells (`gad.cortex.GABA.*`) carry the cortex-wide pattern as a single row. The regional MRS findings have their own cells. Audit rule #1 (subsystem_weights sums) holds at every row.

### 2. SERT in GAD is `contested: methodological`

Maron 2011 and related work show heterogeneous SERT findings across imaging methods (SPECT vs PET) and across acute vs. trait-baseline cohorts. `gad.raphe.5HT.SERT.pre-syn` is encoded at `delta_best: 0` with `contested: methodological` and `delta_range: [−1, +1]`. Notes carry the conflict explanation. Visualizations mark the cell regardless of magnitude.

### 3. The worry/DMN axis gets dedicated functional cells

Standard receptor cells alone undersell GAD's worry component. Functional cells — `gad.mPFC.functional.worry-induction.functional`, `gad.mPFC.functional.DMN-connectivity.functional`, `gad.cortex.composite.dmn-tpn.composite` — encode the well-replicated DMN/worry findings (Andreescu 2015; Coplan 2012) so the BrainMap visualization surfaces the worry circuit, not only the threat circuit. Weights skew heavily to the W subsystem.

### 4. Amygdala vs LC encodes the bottom-up vs top-down distinction

The amygdala-NE cells (`gad.amygdala.NE.*`) are the *receiving* side. The LC cells (`gad.LC.NE.*`) are the *source*. Both are elevated. Treatments that target source (clonidine, guanfacine — α2 agonists) vs receiver (prazosin — α1 antagonist; propranolol — β antagonist) hit different rows. This split is necessary for the drug-coverage residual math to come out right.

### 5. Brain Type 1 anchor

GAD's neurobiology matches Type 1 (Anxious-Vigilant) in `20-brain-types.md`. The template's `brain_type_anchor` field is populated for downstream UX use. This does not alter cell-level reasoning; it tags the template so the type chip can preselect Type 1 as the suggestion at intake.

## ElicitationMap reference

```yaml
id: gad-7.v1
instrument: GAD-7
applies_to_templates: [gad_canonical_v1]
recency_window_days: 14
license_status: free-clinical
source_citation: "Spitzer RL, Kroenke K, Williams JBW, Löwe B (2006). \
                  A brief measure for assessing generalized anxiety disorder: \
                  the GAD-7. Arch Intern Med 166:1092-7."
```

### Scoring rules

- `total` = items[1..7].sum, range [0, 21]
- `cognitive_subscore` = items[1] + items[2] + items[3] (feeling anxious, can't stop worrying, worrying too much)
- `somatic_subscore` = items[4] + items[5] (trouble relaxing, restless)
- `irritability_dread_subscore` = items[6] + items[7] (irritable, afraid something awful might happen)

GAD-7 is not a multi-subscale instrument by design; the subscores above are framework-author divisions for ElicitationMap purposes only and are flagged `evidence_status: inferred`.

### Subsystem mappings

| Scoring | Subsystem | Formula | Evidence | Confidence | Rationale |
|---|---|---|---|---|---|
| total | W | (score − 7) / 7 | inferred | M | Total GAD-7 captures worry as primary symptom; moderate threshold = 7 |
| cognitive_subscore | W | (score − 3) / 3 | inferred | M | Cognitive-worry items map most directly to DMN/worry axis |
| somatic_subscore | H | (score − 2) / 2 | inferred | M | Restless/can't-relax items index autonomic hyperarousal |
| irritability_dread_subscore | I | (score − 2) / 2 | inferred | L | Dread items index threat appraisal; irritability less specific |
| total | R | (score − 14) / 7 | inferred | L | Regulation deficit only inferred at severe range; threshold 14 |

All coefficients are starting calibrations and require pilot validation.

### Cell-level mappings

None in v1. GAD-7 does not have multi-study replication supporting cell-level overrides. All effects route through subsystem weights.

### AI extraction targets

- **Pattern: chronic uncontrollable worry across domains.** Cells: `gad.mPFC.functional.worry-induction.functional`, `gad.mPFC.functional.DMN-connectivity.functional`. Phrasings: "can't stop worrying about everything", "mind races at night", "always thinking about what could go wrong".
- **Pattern: somatic hyperarousal predominant.** Cells: `gad.LC.NE.tone.tone`, `gad.amygdala.NE.tone.tone`. Phrasings: "muscle tension", "can't relax even sitting down", "always on edge", "startle easily".
- **Pattern: sleep-onset insomnia from rumination.** Cells: `gad.mPFC.functional.DMN-connectivity.functional`, `gad.LC.NE.dynamic.dynamic`. Phrasings: "lie awake replaying conversations", "can't quiet my mind to sleep".

## Narrative summary

GAD's signature is **persistent worry plus a nervous system that won't stand down.** Three circuit observations anchor the template.

First, the **worry / DMN axis** is the cognitive engine: mPFC and ACC remain elevated at rest in GAD worriers (Andreescu 2015), DMN connectivity is enhanced, and switching efficiency between DMN and task-positive networks is degraded. This produces the patient's experience of intrusive future-focused worry that resists redirection.

Second, the **limbic threat-monitoring circuit** is overweighted: amygdala threat-cue reactivity is elevated, amygdala-mPFC functional connectivity is reduced, and serotonergic regulation of amygdala (5-HT1A binding) is decreased (Lanzenberger 2007). This is the Anxious-Vigilant (Type 1) signature.

Third, the **cortical inhibitory tone is depleted**: GABA on MRS is reduced in mPFC and ACC (Hasler 2008; Long 2013), and cortex-wide BZ-receptor binding is reduced (Tiihonen 1997). This explains both the regulation deficit (`R` subsystem) and the partial efficacy of benzodiazepines.

The LC noradrenergic source drives the hyperarousal (`H`) presentation (Goddard 2017). Hippocampal and ventral-striatal involvement are present but secondary in pure GAD; they become more prominent in comorbid-MDD and comorbid-panic presentations, handled by composition rules or curated comorbidity templates.

The treatment implication that drops out of the template: first-line SSRI / SNRI coverage hits the serotonergic amygdala/limbic cells but only partially addresses the GABA deficit. Buspirone (5-HT1A partial agonist) hits the raphe and limbic 5-HT1A cells directly. Pregabalin / gabapentin (α2δ Ca-channel modulation, downstream GABAergic effect) hits the cortex-wide GABA cells without BZ tolerance. Benzodiazepines cover BZ-binding cells acutely but introduce tolerance and dependence — flagged for short-term use in residual coverage panels.

## v1 readiness

**Ready:**

- 47 active cells covering the worry, hyperarousal, threat-appraisal, and regulation subsystems with explicit subsystem weights.
- GAD-7 ElicitationMap drafted with subsystem-default mappings; coefficients calibrated to literature direction.
- Brain Type 1 (Anxious-Vigilant) anchor populated.
- Audit rule #1 (subsystem_weights sums) is satisfied at every row.

**Not ready (blockers tracked in `11-readiness-and-blockers.md`):**

- **Drug coverage cells not yet populated.** First priority: escitalopram, sertraline, duloxetine, venlafaxine, buspirone, pregabalin, hydroxyzine, propranolol. Benzodiazepines (lorazepam, clonazepam) are short-term-use cells with tolerance flags.
- **Pilot calibration of GAD-7 → subsystem coefficients.** Formula coefficients are first-pass and need pilot intake data.
- **Clinical reviewer sign-off.** `last_reviewed` / `reviewer` fields are placeholder.
- **State-trait conflict typology.** Several 5-HT1A and SERT findings vary by remission state. Currently flagged at `notes` level; needs explicit `contested: state-trait` tagging once a clinical reviewer audits the source list.
- **Composition rules with MDD and panic templates.** GAD comorbid with MDD or panic is common (50%+ of GAD cases). Pure GAD is the v1 deliverable; comorbid handling depends on the MDD and panic templates landing and on composition rule pilot work.

## References (anchoring sources, partial; full source list lives in registry)

- Andreescu C et al. (2015). The default mode network in late-life anxious depression. *Am J Geriatr Psychiatry.* — DMN/worry findings in GAD.
- Coplan JD et al. (2012). The relationship between intelligence and anxiety: an association with subcortical white matter metabolism. *Front Evol Neurosci.* — DMN-related cortical findings. https://www.frontiersin.org/articles/10.3389/fevo.2011.00008/full
- Etkin A, Wager TD (2007). Functional neuroimaging of anxiety: a meta-analysis. *Am J Psychiatry.* https://pubmed.ncbi.nlm.nih.gov/17898336/
- Etkin A et al. (2009). Disrupted amygdalar subregion functional connectivity and evidence of a compensatory network in generalized anxiety disorder. *Arch Gen Psychiatry.* https://pubmed.ncbi.nlm.nih.gov/19884611/
- Hasler G et al. (2008). Prefrontal cortical GABA concentrations in patients with major depressive disorder and generalized anxiety disorder by proton MRS. *Biol Psychiatry.* — reduced prefrontal GABA in GAD.
- Long Z et al. (2013). Decreased GABA levels in anterior cingulate cortex/medial prefrontal cortex in panic disorder. *Prog Neuropsychopharmacol Biol Psychiatry.* https://pubmed.ncbi.nlm.nih.gov/23643673/
- Tiihonen J et al. (1997). Cerebral benzodiazepine receptor binding and distribution in generalized anxiety disorder. *Mol Psychiatry.* https://www.nature.com/articles/4000329
- Lanzenberger RR et al. (2007). Reduced serotonin-1A receptor binding in social anxiety disorder. *Biol Psychiatry.* — generalizes partially to GAD limbic 5-HT1A.
- Akimova E et al. (2009). The serotonin-1A receptor in anxiety disorders. *Biol Psychiatry.* https://pubmed.ncbi.nlm.nih.gov/19625006/
- Goddard AW (2017). The neurobiology of panic: a chronic stress disorder. *Chronic Stress.* https://journals.sagepub.com/doi/full/10.1177/2470547017736038 — LC/NE framework (cited for the LC noradrenergic findings shared across anxiety spectrum).
- Mathew SJ et al. (2008). Open-label trial of riluzole in generalized anxiety disorder. *Am J Psychiatry.* — glutamatergic findings.
- Mochcovitch MD et al. (2014). A systematic review of fMRI studies in generalized anxiety disorder. *J Affect Disord.* https://pubmed.ncbi.nlm.nih.gov/24388112/
- Bremner JD et al. (2000). SPECT [I-123]iomazenil measurement of the benzodiazepine receptor in panic disorder. *Biol Psychiatry.* — BZ binding methodology reference.
- Maron E, Nutt D (2011). Biological markers of anxiety disorders. *Psychiatry.* — SERT methodology heterogeneity.
- Ball TM et al. (2013). Single-subject anxiety treatment outcome prediction using functional neuroimaging. *Neuropsychopharmacology.* — dlPFC reappraisal findings.
- Greenberg T et al. (2013). Aberrant fear extinction in anxiety disorders. *Front Behav Neurosci.* — vmPFC extinction.
- Kalueff AV, Nutt DJ (2007). Role of GABA in anxiety and depression. *Depress Anxiety.* — GABA-A subunit framework.
- NIH-PMC overview: *Structural and functional neuroimaging studies in generalized anxiety disorder: a systematic review.* https://pmc.ncbi.nlm.nih.gov/articles/PMC6804309/
- NIH-PMC overview: *The neurobiological mechanisms of generalized anxiety disorder and chronic stress.* https://pmc.ncbi.nlm.nih.gov/articles/PMC5832062/

---

Last updated: 2026-05-13 · Draft awaiting clinical review.
