# 25 — Social Anxiety Disorder Template

> **CLINICAL REVIEW REQUIRED.** This file is a first-draft DisorderTemplate authored by the framework team for v1 review. Cell deltas, subsystem weights, and elicitation coefficients are starting calibrations grounded in the cited literature but have not been clinician-committed. Do not deploy to patient profiles before a clinical reviewer signs off on `last_reviewed` / `reviewer` fields and a pilot calibration pass against intake data is complete.

A canonical DisorderTemplate for moderate, unmedicated, trait-baseline Social Anxiety Disorder (generalized type). Mirrors the structure, depth, and tone of `09-ocd-reference-instantiation.md`.

---

## Template metadata

```yaml
id: sad_canonical_v1
schema_version: "3.0"
disorder: SAD
template_version: "1.0.0"
dsm5_code: "300.23"
icd11_code: "6B04"
severity_bucket: moderate
phenotype_subtype: generalized       # generalized type as canonical baseline
baseline_ref: healthy_v1
elicitation_map_ref: lsas-sr.v1
brain_type_anchor: type_1            # Anxious-Vigilant (with reward-deficit overlay)
status: draft
last_reviewed: 2026-05-13
reviewer: framework-team
notes: |
  Social Anxiety Disorder (generalized type) presents with persistent fear
  of social evaluation, behavioral avoidance of social-performance
  situations, and a characteristic dual deficit: amygdala-mPFC threat
  hyperreactivity (the anxious vigilance limb) plus ventral striatal reward
  hypoactivation specifically for social rewards (the reward-deficit limb).
  Performance-only (non-generalized) SAD is handled as a phenotype modifier.
  Brain Type 1 (Anxious-Vigilant) is the patient-identity anchor, with
  partial overlap to Type 2 (Anhedonic-Depleted) when reward deficit
  dominates.
```

The template represents **moderate, unmedicated, trait-baseline SAD, generalized type**. Performance-only SAD (fear restricted to public speaking / performance) is a phenotype modifier rather than a separate template — the circuit differences are subtle in pure-form imaging and the literature mostly studies generalized SAD.

## Subsystems

SAD's four canonical subsystems:

- **T — Threat (social-evaluative).** Amygdala-led hyperreactivity to faces, especially angry/disgusted/critical; reduced top-down regulation from mPFC. Maps to amygdala, mPFC, vmPFC, OFC.
- **N — Negative self-referential processing.** Self-focused attention, post-event rumination, mPFC self-referential processing. Maps to mPFC (DMN component), ACC, dlPFC.
- **R — Reward deficit (social-specific).** Ventral-striatal hypoactivation specifically to social rewards (vs intact non-social reward); reduced D2 binding (Schneier 2000, 2009); reduced DAT binding (Tiihonen 1997 SPECT — contested with van der Wee 2008). Maps to vS, VTA, DA system. The distinguishing feature vs GAD/panic.
- **A — Behavioral avoidance / performance.** Conditioned avoidance of social-performance situations; autonomic anticipatory anxiety on performance cues. Maps to vmPFC, hippocampus, amygdala, LC.

Every cell carries `subsystem_weights` distributing its contribution across {T, N, R, A}. Weights for any non-zero cell sum to 1.0 ± 0.01.

## Cell coverage

46 active cells. Coverage by region:

### Cortical regions

- **mPFC** — self-referential processing engaged during social-evaluation tasks; reduced top-down inhibition of amygdala (Goldin 2009; Brühl 2014). 5 cells.
- **ACC** — social-evaluation-error monitoring; reduced 5-HT1A binding (Lanzenberger 2007). 4 cells.
- **OFC** — disrupted effective connectivity with amygdala during emotion discrimination (Sladky 2015). 3 cells.
- **vmPFC** — fear-extinction circuit; reduced extinction recall on social cues. 3 cells.
- **dlPFC** — top-down regulation; modest involvement. 2 cells.

### Limbic regions

- **Amygdala** — central hyperreactivity to social-threat cues (Phan 2006; Etkin 2007 meta-analysis); reduced 5-HT1A binding (Lanzenberger 2007); cortisol-5HT1A inverse correlation. 8 cells.
- **Hippocampus** — context-discrimination deficits; reduced 5-HT1A binding (Lanzenberger 2007). 3 cells.

### Subcortical / reward (this is where SAD differs)

- **vS (ventral striatum / NAc)** — reduced D2 binding (Schneier 2000); reduced reward-anticipation BOLD for social rewards (Richey 2017); intact response to non-social rewards in some studies. 6 cells. This is the disorder-distinguishing region.
- **Caudate** — minor involvement; D2 receptor changes (Schneier 2009). 2 cells.
- **Putamen** — minor involvement; some DA findings. 1 cell.

### Brainstem / regulatory

- **Raphe (DRN)** — serotonergic source; reduced 5-HT1A binding cross-regional (Lanzenberger 2007). 3 cells.
- **LC** — noradrenergic source; modest tonic elevation; less prominent than panic. 2 cells.
- **VTA** — dopaminergic source; reduced phasic DA to social rewards (inferred from striatal findings); central to the reward-deficit limb. 4 cells.

### Cortical-wide / functional

- **Cortex (5HT composite)** — cross-regional reduced 5-HT1A binding pattern (Lanzenberger 2007 PET landmark); reduced 5HT/DA transporter co-expression patterns (Plavén-Sigray 2017). 2 cells.

## Cell registry (excerpted)

| Cell ID | Region | System | Target | Site | δ | range | tier | conf | weights {T,N,R,A} | sources |
|---|---|---|---|---|---|---|---|---|---|---|
| sad.amygdala.5HT.5HT1A.post-syn | Amygdala | 5HT | 5HT1A | post-syn | −2 | [−3,−1] | 1 | H | {0.50, 0.10, 0.10, 0.30} | Lanzenberger 2007 PET landmark |
| sad.amygdala.5HT.5HT2A.post-syn | Amygdala | 5HT | 5HT2A | post-syn | +1 | [0,+2] | 2 | M | {0.50, 0.10, 0.10, 0.30} | inferred from anxiety spectrum |
| sad.amygdala.GABA.GABA-A/BZ.post-syn | Amygdala | GABA | GABA-A/BZ-site | post-syn | −1 | [−2,0] | 3 | L | {0.40, 0.10, 0.10, 0.40} | inferred from anxiety spectrum |
| sad.amygdala.DA.D1.post-syn | Amygdala | DA | D1 | post-syn | −1 | [−2,0] | 3 | L | {0.30, 0.10, 0.30, 0.30} | inferred (Plavén-Sigray 2017 system-level) |
| sad.amygdala.NE.tone.tone | Amygdala | NE | NE tone | tone | +1 | [0,+2] | 2 | L | {0.40, 0.10, 0.10, 0.40} | Goddard 2017 (anxiety-spectrum) |
| sad.amygdala.functional.face-reactivity.functional | Amygdala | Composite | Threat-face (angry/disgust) BOLD reactivity | functional | +3 | [+2,+3] | 1 | H | {0.70, 0.10, 0.05, 0.15} | Phan 2006; Etkin & Wager 2007 meta |
| sad.amygdala.functional.mPFC-connectivity.functional | Amygdala | Composite | Amygdala–mPFC functional connectivity | functional | −2 | [−3,−1] | 1 | M | {0.50, 0.20, 0.10, 0.20} | Goldin 2009; Brühl 2014 |
| sad.amygdala.functional.OFC-connectivity.functional | Amygdala | Composite | Amygdala–OFC effective connectivity (DCM) | functional | −2 | [−3,−1] | 1 | M | {0.50, 0.10, 0.15, 0.25} | Sladky 2015 |
| sad.mPFC.5HT.5HT1A.post-syn | mPFC | 5HT | 5HT1A | post-syn | −2 | [−3,−1] | 1 | H | {0.20, 0.40, 0.10, 0.30} | Lanzenberger 2007 |
| sad.mPFC.functional.self-referential.functional | mPFC | Composite | Self-referential processing BOLD (social-eval tasks) | functional | +2 | [+1,+3] | 1 | M | {0.20, 0.60, 0.10, 0.10} | Blair 2008; Goldin 2009 |
| sad.mPFC.functional.regulation.functional | mPFC | Composite | Top-down regulation of amygdala (PPI) | functional | −2 | [−3,−1] | 1 | M | {0.40, 0.20, 0.10, 0.30} | Goldin 2009 |
| sad.mPFC.GABA.tone.tone | mPFC | GABA | GABA tone (MRS) | tone | −1 | [−2,0] | 2 | L | {0.20, 0.40, 0.10, 0.30} | inferred from GAD literature |
| sad.mPFC.composite.DMN-self.composite | mPFC | Composite | DMN connectivity during social-eval | composite | +1 | [0,+2] | 2 | L | {0.20, 0.60, 0.10, 0.10} | Brühl 2014 |
| sad.ACC.5HT.5HT1A.post-syn | ACC | 5HT | 5HT1A | post-syn | −2 | [−3,−1] | 1 | H | {0.20, 0.40, 0.10, 0.30} | Lanzenberger 2007 |
| sad.ACC.functional.social-error.functional | ACC | Composite | Social-evaluation error monitoring | functional | +2 | [+1,+3] | 2 | M | {0.30, 0.50, 0.10, 0.10} | Amir 2005; Brühl 2014 |
| sad.ACC.GABA.tone.tone | ACC | GABA | GABA tone (MRS) | tone | −1 | [−2,0] | 3 | L | {0.20, 0.40, 0.10, 0.30} | inferred |
| sad.ACC.composite.glu-gaba.composite | ACC | Composite | Glu:GABA ratio | composite | +1 | [0,+2] | 3 | L | {0.20, 0.40, 0.10, 0.30} | inferred from GAD |
| sad.OFC.5HT.5HT2A.post-syn | OFC | 5HT | 5HT2A | post-syn | +1 | [0,+2] | 2 | L | {0.40, 0.30, 0.10, 0.20} | inferred |
| sad.OFC.functional.value-eval.functional | OFC | Composite | Social-value evaluation (BOLD) | functional | −1 | [−2,0] | 2 | L | {0.30, 0.30, 0.30, 0.10} | Sladky 2015 |
| sad.OFC.functional.amygdala-effective.functional | OFC | Composite | OFC→amygdala effective connectivity | functional | −2 | [−3,−1] | 1 | M | {0.50, 0.20, 0.10, 0.20} | Sladky 2015 |
| sad.vmPFC.5HT.5HT1A.post-syn | vmPFC | 5HT | 5HT1A | post-syn | −1 | [−2,0] | 2 | M | {0.20, 0.20, 0.20, 0.40} | Lanzenberger 2007 |
| sad.vmPFC.functional.extinction.functional | vmPFC | Composite | Fear-extinction recall (social cues) | functional | −2 | [−3,−1] | 2 | M | {0.20, 0.10, 0.10, 0.60} | Greenberg 2013 (anxiety extinction) |
| sad.vmPFC.functional.safety-signal.functional | vmPFC | Composite | Safety-signal processing | functional | −1 | [−2,0] | 2 | L | {0.30, 0.20, 0.20, 0.30} | inferred |
| sad.dlPFC.functional.reappraisal.functional | dlPFC | Composite | Cognitive reappraisal engagement | functional | −2 | [−3,−1] | 1 | M | {0.30, 0.20, 0.10, 0.40} | Goldin 2009 |
| sad.dlPFC.NE.alpha2.post-syn | dlPFC | NE | α2A-adrenergic | post-syn | −1 | [−2,0] | 3 | L | {0.20, 0.20, 0.10, 0.50} | inferred (Arnsten 2009) |
| sad.hippocampus.5HT.5HT1A.post-syn | Hippocampus | 5HT | 5HT1A | post-syn | −2 | [−3,−1] | 1 | M | {0.30, 0.30, 0.10, 0.30} | Lanzenberger 2007 |
| sad.hippocampus.functional.social-context.functional | Hippocampus | Composite | Social-context discrimination | functional | −1 | [−2,0] | 2 | L | {0.30, 0.20, 0.10, 0.40} | inferred |
| sad.hippocampus.GABA.tone.tone | Hippocampus | GABA | GABA tone | tone | −1 | [−2,0] | 3 | L | {0.30, 0.20, 0.10, 0.40} | inferred |
| sad.vS.DA.D2.post-syn | vS | DA | D2/D3 | post-syn | −2 | [−3,−1] | 1 | H | {0.10, 0.10, 0.70, 0.10} | Schneier 2000 SPECT |
| sad.vS.DA.D1.post-syn | vS | DA | D1 | post-syn | −1 | [−2,0] | 3 | L | {0.10, 0.10, 0.70, 0.10} | inferred |
| sad.vS.DA.DAT.pre-syn | vS | DA | DAT | pre-syn | −1 | [−2,0] | 1 | M | {0.10, 0.10, 0.70, 0.10} | Tiihonen 1997 SPECT (contested: methodological; van der Wee 2008) |
| sad.vS.DA.tone.tone | vS | DA | DA tone (social-reward-specific) | tone | −2 | [−3,−1] | 2 | M | {0.10, 0.10, 0.70, 0.10} | Richey 2017; Cremers 2015 |
| sad.vS.functional.social-reward.functional | vS | Composite | Social-reward anticipation BOLD | functional | −2 | [−3,−1] | 1 | H | {0.05, 0.10, 0.80, 0.05} | Richey 2017; Cremers 2015 |
| sad.vS.functional.nonsocial-reward.functional | vS | Composite | Non-social-reward anticipation BOLD | functional | 0 | [−1,+1] | 2 | M | {0.10, 0.10, 0.70, 0.10} | Richey 2017 (intact non-social reward) |
| sad.caudate.DA.D2.post-syn | Caudate | DA | D2/D3 | post-syn | −1 | [−2,0] | 2 | M | {0.10, 0.20, 0.50, 0.20} | Schneier 2009 |
| sad.caudate.functional.habit.functional | Caudate | Composite | Habitual avoidance pattern engagement | functional | +1 | [0,+2] | 3 | L | {0.10, 0.10, 0.10, 0.70} | inferred |
| sad.putamen.DA.D2.post-syn | Putamen | DA | D2/D3 | post-syn | 0 | [−1,+1] | 3 | L | {0.20, 0.20, 0.40, 0.20} | inferred; minimal in SAD |
| sad.raphe.5HT.5HT1A.auto | Raphe | 5HT | 5HT1A | auto | +1 | [0,+2] | 2 | M | {0.40, 0.20, 0.10, 0.30} | Lanzenberger 2007 (autoreceptor inference) |
| sad.raphe.5HT.SERT.pre-syn | Raphe | 5HT | SERT | pre-syn | 0 | [−1,+1] | 1 | M | {0.30, 0.20, 0.20, 0.30} | Plavén-Sigray 2017 (contested: methodological) |
| sad.raphe.5HT.tone.tone | Raphe | 5HT | 5HT tone (cortical projection) | tone | −1 | [−2,0] | 2 | M | {0.30, 0.30, 0.10, 0.30} | Lanzenberger 2007 implication |
| sad.LC.NE.tone.tone | LC | NE | NE tone | tone | +1 | [0,+2] | 2 | L | {0.40, 0.10, 0.10, 0.40} | Goddard 2017 |
| sad.LC.NE.dynamic.dynamic | LC | NE | LC tonic firing | dynamic | +1 | [0,+2] | 3 | L | {0.40, 0.10, 0.10, 0.40} | inferred |
| sad.VTA.DA.dynamic.dynamic | VTA | DA | VTA phasic firing to social rewards | dynamic | −2 | [−3,−1] | 2 | M | {0.05, 0.10, 0.80, 0.05} | inferred from striatal findings; Pohlack 2012 |
| sad.VTA.DA.tone.tone | VTA | DA | VTA tonic firing | tone | −1 | [−2,0] | 3 | L | {0.10, 0.10, 0.70, 0.10} | inferred |
| sad.VTA.DA.D2.auto | VTA | DA | D2 (autoreceptor) | auto | +1 | [0,+2] | 3 | L | {0.10, 0.10, 0.70, 0.10} | inferred |
| sad.VTA.DA.functional.reward-prediction.functional | VTA | Composite | Social-reward prediction error signal | functional | −2 | [−3,−1] | 2 | M | {0.05, 0.10, 0.80, 0.05} | Richey 2017 derivation |
| sad.cortex.5HT.5HT1A.density | Cortex (composite) | 5HT | 5HT1A density (cross-regional aggregate) | density | −2 | [−3,−1] | 1 | H | {0.30, 0.30, 0.10, 0.30} | Lanzenberger 2007 |
| sad.cortex.composite.cortisol-5ht1a.composite | Cortex (composite) | Composite | Cortisol-to-5HT1A inverse correlation | composite | −1 | [−2,0] | 2 | M | {0.40, 0.20, 0.10, 0.30} | Lanzenberger 2010 |

Subsystem weights cross-check: each row sums to 1.0 ± 0.01.

## Key authoring choices

### 1. The social-vs-nonsocial reward split is encoded explicitly

`sad.vS.functional.social-reward.functional` (δ = −2) and `sad.vS.functional.nonsocial-reward.functional` (δ = 0) live as two separate cells. This is unusual — most templates have one cell per region-system-target tuple — but the SAD literature is clear that the reward deficit is *social-specific*, not a generalized reward dysfunction (Richey 2017; Cremers 2015). The two cells differ only by `target` ("social-reward anticipation" vs "non-social-reward anticipation"), satisfying the single-tuple rule. This distinguishes SAD from major depression's pan-anhedonia and from generic anxiety. It also licenses the partial-overlap Brain Type chip behavior: SAD pulls slightly toward Type 2 (Anhedonic-Depleted) only on the social-reward dimension.

### 2. DAT in vS is `contested: methodological`

Tiihonen 1997 SPECT reported decreased DAT binding; van der Wee 2008 reported increased; subsequent meta-analyses are inconclusive. `sad.vS.DA.DAT.pre-syn` is `delta_best: −1`, `delta_range: [−2, 0]`, `contested: methodological`. Notes carry the conflict explanation. The visualization marks the cell regardless of direction.

### 3. The Lanzenberger 2007 PET landmark is the spine of the 5-HT1A cells

The Lanzenberger 2007 *Biol Psychiatry* PET study reported reduced 5-HT1A binding across limbic regions (amygdala, ACC, mPFC, hippocampus, dorsal raphe, insula) in SAD. This finding is the strongest single PET landmark in the SAD literature. The template encodes the regional findings as separate cells rather than aggregating, because regional differentiation matters for the cell-level math even when the source is single. The cortex-composite cell (`sad.cortex.5HT.5HT1A.density`) carries the cross-regional pattern as an additional summary row.

### 4. Cortisol-5HT1A inverse correlation cell

Lanzenberger's 2010 follow-up reported negative correlation between serum cortisol and 5-HT1A binding in amygdala and hippocampus in SAD. This is encoded as a composite cell with notes flagging the composite nature. It is included because it has clinical implications (HPA-axis modifying interventions may shift 5-HT1A function) that don't fall out of any single receptor cell.

### 5. VTA cells for the reward-deficit limb

Standard receptor cells alone can't carry the reward-deficit story (which is fundamentally a *signaling dynamics* finding, not a density finding). `sad.VTA.DA.dynamic.dynamic` and `sad.VTA.DA.functional.reward-prediction.functional` encode the phasic-to-social-rewards finding. Weights skew heavily to the `R` subsystem.

### 6. Performance-only SAD as modifier, not template

The phenotype_subtype field is set to "generalized" as the canonical baseline. Performance-only SAD (formerly "discrete" or "non-generalized" SAD; DSM-5 still has a "performance only" specifier) is handled as a PatientSubsystemModifier rather than a separate template — the imaging literature on pure performance-only SAD is thin and the circuit differences appear to be largely magnitude differences rather than directional differences. A clinical reviewer should validate this decision.

### 7. Brain Type 1 anchor with Type 2 overlap

SAD primarily anchors to Type 1 (Anxious-Vigilant) but the reward-deficit limb pulls partially toward Type 2 (Anhedonic-Depleted). The type chip preselects Type 1, with Type 2 as a possible secondary if the reward-deficit cells dominate the effective vector at intake. This is handled at the type-resolution step, not at the cell level.

## ElicitationMap reference

```yaml
id: lsas-sr.v1
instrument: LSAS-SR (Liebowitz Social Anxiety Scale, self-report)
applies_to_templates: [sad_canonical_v1]
recency_window_days: 14
license_status: free-clinical
source_citation: "Liebowitz MR (1987). Social Phobia. Mod Probl Pharmacopsychiatry. \
                  Self-report version: Fresco DM et al. (2001) Psychol Med. \
                  https://pubmed.ncbi.nlm.nih.gov/11722154/"
```

### Scoring rules

LSAS-SR has 24 items, each rated for both fear (0–3) and avoidance (0–3). Total range 0–144.

- `total` = sum of all fear + avoidance items, range [0, 144]
- `fear_total` = sum of fear items, range [0, 72]
- `avoidance_total` = sum of avoidance items, range [0, 72]
- `social_interaction_fear` = sum of fear items for the 11 social-interaction items
- `performance_fear` = sum of fear items for the 13 performance items
- `social_interaction_avoidance` = sum of avoidance items for social interaction
- `performance_avoidance` = sum of avoidance items for performance

### Subsystem mappings

| Scoring | Subsystem | Formula | Evidence | Confidence | Rationale |
|---|---|---|---|---|---|
| fear_total | T | (score − 25) / 15 | inferred | M | Fear items index threat-circuit recruitment (moderate threshold ~25) |
| social_interaction_fear | R | (score − 12) / 12 | inferred | M | Social-interaction fear correlates with reward deficit (less reward signal → more aversive interaction) |
| avoidance_total | A | (score − 20) / 15 | inferred | M | Avoidance items index conditioned-avoidance circuit |
| total | N | (score − 50) / 30 | inferred | L | Higher total scores correlate with rumination / post-event processing |
| performance_fear | T | (score − 15) / 10 | inferred | L | Half-weight to T; mainly informative for performance-only subtype |

All coefficients are starting calibrations and require pilot validation.

### Cell-level mappings

None in v1. LSAS-SR does not have multi-study replication supporting cell-level overrides. All effects route through subsystem weights.

### AI extraction targets

- **Pattern: fear of negative evaluation across social situations.** Cells: `sad.amygdala.functional.face-reactivity.functional`, `sad.mPFC.functional.self-referential.functional`. Phrasings: "always worried what people think", "convinced they're judging me", "replaying conversations afterward".
- **Pattern: performance-only fear (public speaking, performing).** Cells: `sad.amygdala.functional.face-reactivity.functional`, `sad.LC.NE.tone.tone`. Phrasings: "fine one-on-one but freeze in front of groups", "panic before any presentation".
- **Pattern: social-reward deficit / social anhedonia.** Cells: `sad.vS.functional.social-reward.functional`, `sad.VTA.DA.functional.reward-prediction.functional`. Phrasings: "compliments don't feel good", "don't get the rush other people seem to get from socializing".
- **Pattern: post-event rumination.** Cells: `sad.mPFC.functional.self-referential.functional`, `sad.ACC.functional.social-error.functional`. Phrasings: "go over conversations for days", "cringe at things I said years ago".
- **Pattern: pervasive avoidance / safety behaviors.** Cells: `sad.vmPFC.functional.extinction.functional`, `sad.caudate.functional.habit.functional`. Phrasings: "won't make eye contact", "always have an escape planned", "drink to get through events".

## Narrative summary

SAD's signature is **threat-circuit hyperreactivity to social-evaluation cues, paired with a reward circuit that signals less for social rewards than for non-social ones.** Three observations anchor the template.

First, the **amygdala-mPFC threat-evaluation circuit is dysregulated**: amygdala face-reactivity is elevated (Phan 2006; Etkin & Wager 2007 meta-analysis), amygdala-mPFC functional connectivity is reduced (Goldin 2009; Brühl 2014), and effective connectivity from OFC to amygdala is altered (Sladky 2015). This is the Type-1-style anxious vigilance limb. The patient experiences faces — especially neutral, angry, or disgusted faces — as threats requiring vigilance.

Second, the **5-HT1A landmark** (Lanzenberger 2007 PET) reports reduced 5-HT1A binding across limbic and cortical regions (amygdala, ACC, mPFC, hippocampus, raphe, insula). This is the strongest pharmacology-relevant finding in SAD and is the mechanistic anchor for SSRI efficacy: SSRIs gradually restore 5-HT1A function via autoreceptor desensitization. Lanzenberger 2010 added the cortisol-5HT1A inverse correlation, which connects HPA-axis status to receptor function.

Third — and this is what distinguishes SAD from GAD or panic — the **reward circuit shows a social-specific deficit**: ventral-striatal D2 binding is reduced (Schneier 2000 SPECT), ventral-striatal BOLD to social-reward anticipation is reduced (Richey 2017; Cremers 2015), but non-social reward anticipation is largely intact. This is the Type-2 overlap finding: SAD patients are not pan-anhedonic, but social engagement specifically fails to produce the reward signal that would normally reinforce approach behavior. The behavioral consequence — avoidance — gets a circuit explanation.

Caudate D2 reductions (Schneier 2009) extend the dopaminergic story modestly. Cortical and hippocampal cells largely follow the amygdala/limbic 5-HT1A pattern. The LC noradrenergic component is present but secondary; SAD's autonomic component is real but not the brainstem-dominated phenotype of panic.

The treatment implication: first-line SSRIs (paroxetine, sertraline, escitalopram) hit the cortical and limbic 5-HT1A cells, with onset matching the 4–6 week receptor-resensitization timeline. Venlafaxine extends coverage to the LC/NE cells. Pregabalin / gabapentin cover the GABA cells (less direct evidence in SAD than in GAD, but mechanism-consistent). MAOIs (phenelzine) have historically strong SAD evidence and hit both serotonergic and dopaminergic systems — particularly relevant given the reward-deficit limb. Beta-blockers (propranolol) cover peripheral autonomic symptoms for performance-only presentations. CBT (cognitive restructuring + exposure) addresses the negative self-referential subsystem (`N`) and the conditioned-avoidance subsystem (`A`) through experiential rather than pharmacological mechanisms.

## v1 readiness

**Ready:**

- 46 active cells covering threat, negative self-referential, reward-deficit, and avoidance subsystems with explicit subsystem weights.
- LSAS-SR ElicitationMap drafted with subsystem-default mappings; coefficients calibrated to literature direction.
- Brain Type 1 (Anxious-Vigilant) anchor populated, with Type 2 overlap on reward-deficit dimension flagged.
- The social-vs-nonsocial reward split is explicitly encoded — a v1-distinctive authoring decision.
- Lanzenberger 2007 5-HT1A landmark spine encoded across regional cells.

**Not ready (blockers tracked in `11-readiness-and-blockers.md`):**

- **Drug coverage cells not yet populated.** First priority: paroxetine (best SAD evidence), sertraline, escitalopram, venlafaxine, phenelzine (historical first-line; MAOI), pregabalin, propranolol (performance-only).
- **Pilot calibration of LSAS-SR → subsystem coefficients.** Formula coefficients are first-pass.
- **DAT contested cell.** `sad.vS.DA.DAT.pre-syn` should remain flagged until a clinical reviewer audits the methodology-vs-state-trait distinction (Tiihonen vs van der Wee).
- **Performance-only SAD subtype.** Currently handled as PatientSubsystemModifier. Pilot data may justify a separate phenotype branch.
- **Insula vocabulary gap.** Insula is referenced in Lanzenberger 2007 as carrying part of the 5-HT1A pattern but is not in v1 region vocabulary. Currently encoded under Cortex (composite); pending v2 vocabulary extension.
- **Comorbidity composition.** SAD + MDD is the most prevalent comorbidity (40%+); composition rules will likely produce sensible composite vectors but pilot validation is required. Comorbid SAD + avoidant personality disorder is a known stable phenotype that may warrant a curated comorbidity template.

## References (anchoring sources, partial; full source list lives in registry)

- Lanzenberger RR et al. (2007). Reduced serotonin-1A receptor binding in social anxiety disorder. *Biol Psychiatry.* https://pubmed.ncbi.nlm.nih.gov/16996023/ — PET landmark.
- Lanzenberger R et al. (2010). Cortisol plasma levels in social anxiety disorder patients correlate with serotonin-1A receptor binding in limbic brain regions. *Int J Neuropsychopharmacol.* https://pubmed.ncbi.nlm.nih.gov/20846460/
- Schneier FR et al. (2000). Low dopamine D2 receptor binding potential in social phobia. *Am J Psychiatry.* https://pubmed.ncbi.nlm.nih.gov/10739431/ — SPECT D2 landmark.
- Schneier FR et al. (2009). Dopamine transporters, D2 receptors, and dopamine release in generalized social anxiety disorder. *Depress Anxiety.* https://pmc.ncbi.nlm.nih.gov/articles/PMC2679094/
- Tiihonen J et al. (1997). Dopamine reuptake site densities in patients with social phobia. *Am J Psychiatry.* https://pubmed.ncbi.nlm.nih.gov/9090349/ — SPECT DAT (contested).
- van der Wee NJ et al. (2008). Increased serotonin and dopamine transporter binding in psychotropic medication-naïve patients with generalized social anxiety disorder shown by 123I-β-(4-iodophenyl)-tropane SPECT. *J Nucl Med.* — DAT contested counter-finding.
- Plavén-Sigray P et al. (2017). Expression and co-expression of serotonin and dopamine transporters in social anxiety disorder: a multitracer positron emission tomography study. *Mol Psychiatry.* https://www.nature.com/articles/s41380-019-0618-7
- Cervenka S et al. (2012). Changes in dopamine D2-receptor binding are associated to symptom reduction after psychotherapy in social anxiety disorder. *Transl Psychiatry.* https://www.nature.com/articles/tp201240
- Phan KL et al. (2006). Association between amygdala hyperactivity to harsh faces and severity of social anxiety in generalized social phobia. *Biol Psychiatry.* https://pubmed.ncbi.nlm.nih.gov/16139816/
- Etkin A, Wager TD (2007). Functional neuroimaging of anxiety: a meta-analysis. *Am J Psychiatry.* https://pubmed.ncbi.nlm.nih.gov/17898336/
- Goldin PR et al. (2009). Neural mechanisms of cognitive reappraisal of negative self-beliefs in social anxiety disorder. *Biol Psychiatry.* https://pubmed.ncbi.nlm.nih.gov/19632436/
- Brühl AB et al. (2014). Neuroimaging in social anxiety disorder — a meta-analytic review resulting in a new neurofunctional model. *Neurosci Biobehav Rev.* https://pubmed.ncbi.nlm.nih.gov/25124509/
- Sladky R et al. (2015). Disrupted Effective Connectivity Between the Amygdala and Orbitofrontal Cortex in Social Anxiety Disorder During Emotion Discrimination. *Soc Cogn Affect Neurosci.* https://pmc.ncbi.nlm.nih.gov/articles/PMC4379995/
- Richey JA et al. (2017). Neural mechanisms of reward processing in adolescents with and without social anxiety disorder. *Soc Cogn Affect Neurosci.*
- Cremers HR et al. (2015). Social anxiety, the amygdala, and the mesolimbic dopamine system. *Cortex.*
- Pohlack ST et al. (2012). Neural mechanism of a sex-specific risk variant for posttraumatic stress disorder in the type I receptor of the pituitary adenylate cyclase activating polypeptide. *Biol Psychiatry.* (cited for VTA inference in anxiety spectrum).
- Frick A et al. (2015). Serotonin synthesis and reuptake in social anxiety disorder: a positron emission tomography study. *JAMA Psychiatry.*
- Akimova E et al. (2009). The serotonin-1A receptor in anxiety disorders. *Biol Psychiatry.* https://pubmed.ncbi.nlm.nih.gov/19625006/
- NIH-PMC overview: *Neurobiological correlates of social anxiety disorder: an update.* https://pubmed.ncbi.nlm.nih.gov/24571962/
- Frontiers review: *Reward Circuitry and Motivational Deficits in Social Anxiety Disorder: What Can Be Learned From Mouse Models?* https://www.frontiersin.org/journals/neuroscience/articles/10.3389/fnins.2020.00154/full
- Fresco DM et al. (2001). The Liebowitz Social Anxiety Scale: a comparison of the psychometric properties of self-report and clinician-administered formats. *Psychol Med.* https://pubmed.ncbi.nlm.nih.gov/11722154/

---

Last updated: 2026-05-13 · Draft awaiting clinical review.
