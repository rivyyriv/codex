# 26 — PTSD Template (with Dissociative Subtype)

> **CLINICAL REVIEW REQUIRED.** This file is a first-draft DisorderTemplate authored by the framework team for v1 review. Cell deltas, subsystem weights, subtype-conditional cells, and elicitation coefficients are starting calibrations grounded in the cited literature but have not been clinician-committed. The DSM-5 dissociative subtype is encoded as a phenotype branch with reversed-direction cells; the subtype switch logic requires clinical reviewer audit before deployment. Do not deploy to patient profiles before sign-off and a pilot calibration pass against intake data.

A canonical DisorderTemplate for moderate, unmedicated, trait-baseline PTSD, with explicit handling of the DSM-5 dissociative subtype. Mirrors the structure, depth, and tone of `09-ocd-reference-instantiation.md`.

---

## Template metadata

```yaml
id: ptsd_canonical_v1
schema_version: "3.0"
disorder: PTSD
template_version: "1.0.0"
dsm5_code: "309.81"
icd11_code: "6B40"                   # also 6B41 (complex PTSD); 6B40.1 (with dissociative)
severity_bucket: moderate
phenotype_subtype: hyperarousal       # canonical baseline; dissociative is a sibling branch
subtype_branches: [hyperarousal, dissociative]
baseline_ref: healthy_v1
elicitation_map_ref: pcl-5.v1
brain_type_anchor: type_6            # Trauma-Imprinted (Hyperarousal / Dissociative dual pattern)
status: draft
last_reviewed: 2026-05-13
reviewer: framework-team
notes: |
  PTSD presents across four DSM-5 symptom clusters (re-experiencing,
  avoidance, negative cognition/mood, hyperarousal) with an explicit
  dissociative subtype recognized in DSM-5. The hyperarousal pattern is
  encoded as the canonical baseline. The dissociative pattern is encoded
  as a phenotype branch via subtype-conditional cells with directionally
  reversed deltas on the affect-regulation circuit (mPFC/rACC vs amygdala).
  Both branches share most cells. Brain Type 6 (Trauma-Imprinted) is the
  patient-identity anchor and is the brain type that natively encodes
  this dual-pattern phenotype.
```

The template represents **moderate, unmedicated, trait-baseline PTSD, hyperarousal pattern** as the canonical baseline. Severity scales via `severity_factor`. The dissociative subtype is encoded as a phenotype branch — see "Subtype-conditional cells" below for the schema mechanism. Complex PTSD (cPTSD; ICD-11 6B41) shares most cells but adds chronicity / developmental cells; v1 handles cPTSD as a phenotype modifier rather than a separate template.

## Subsystems

PTSD's five canonical subsystems (DSM-5-aligned plus the dissociative dimension):

- **H — Hyperarousal.** Hypervigilance, exaggerated startle, sleep disturbance, irritability. LC NE elevation, amygdala-driven autonomic output. Maps to LC, amygdala, insula (composite proxy), hippocampus.
- **R — Re-experiencing / intrusion.** Flashbacks, intrusive memories, trauma-cued reactivity. Maps to amygdala, hippocampus (context-failure), vmPFC (extinction failure), Glu/NMDA-mediated reconsolidation.
- **V — Avoidance.** Behavioral and cognitive avoidance of trauma reminders. Maps to vmPFC, hippocampus, amygdala fear-consolidation, OFC.
- **N — Negative cognition / mood.** Persistent negative beliefs, distorted blame, anhedonia, detachment. Maps to mPFC, ACC, vS reward-deficit, vmPFC. Overlaps with MDD on anhedonia cells.
- **D — Dissociative (subtype-specific).** Depersonalization, derealization, emotional numbing. Maps to mPFC, rostral ACC overactivation, reduced insular interoception, altered DMN connectivity. **Active only for the dissociative phenotype branch.** In the hyperarousal branch this subsystem is dormant (weight 0 on all cells).

Every cell carries `subsystem_weights` distributing its contribution across {H, R, V, N, D}. Weights for any non-zero cell sum to 1.0 ± 0.01.

## Phenotype-branch handling: how the dissociative subtype is encoded

The dissociative subtype is not a separate disorder; it shares ~75% of its cells with the hyperarousal baseline (the trauma-history substrate is common to both). The remaining ~25% are cells where the literature documents a **directional reversal**:

- **Hyperarousal pattern:** amygdala over-activation, mPFC / rostral ACC under-activation. "Bottom-up" — affect overwhelms cognition.
- **Dissociative pattern:** amygdala under-activation, mPFC / rostral ACC *over*-activation ("emotional overmodulation"). "Top-down" — cognition suppresses affect, producing depersonalization and derealization.

(Lanius 2010, 2012; Daniels et al. 2016; Nicholson et al. 2015 dynamic causal modeling.)

### Schema mechanism

The v3 schema's `phenotype_subtype` field on DisorderTemplate is currently a single string. To support sibling phenotype branches sharing one template, we introduce `subtype_branches` (proposed v3.1 additive field) listing the supported branches, and tag subtype-conditional cells with a `phenotype_branch` qualifier in the cell's `notes` and an explicit `subtype_overrides` block:

```yaml
# Example: amygdala-mPFC connectivity cell with both branches
- id: ptsd.amygdala.functional.mPFC-connectivity.functional
  delta_best: -2                          # hyperarousal (canonical) baseline
  delta_range: [-3, -1]
  phenotype_branch: hyperarousal           # the canonical branch this row encodes
  subtype_overrides:
    dissociative:
      delta_best: +2                       # mPFC oversuppresses amygdala
      delta_range: [+1, +3]
      notes: "Reversed in dissociative subtype; top-down overmodulation \
              (Lanius 2010; Nicholson 2015 DCM)."
  ...
```

The runtime selects the active branch from the PatientProfile's `phenotype_subtype` field (a per-profile string; e.g., `"dissociative"`). If `phenotype_subtype == "dissociative"`, the override block applies; otherwise the cell renders at `delta_best`. This is a minor schema extension; the alternative (two full sibling templates) was rejected because ~75% of cells are shared and maintenance would double.

**Tracked in `15-schema-extensions.md` as proposal v3.1.SUB-1.** Until that lands, the template ships with an authoring convention: subtype-conditional cells carry both values in `notes` and the runtime applies the override via a thin lookup table. The visualization renders the branch the profile selects.

## Cell coverage

51 active cells. Coverage by region:

### Cortical regions

- **mPFC** — central to the dual pattern. Hyperarousal: reduced engagement, reduced regulation of amygdala. Dissociative: **increased** engagement, oversuppression of amygdala. 6 cells, 3 with dissociative overrides.
- **rACC** (rostral ACC) — same dual pattern. Encoded under ACC region with `target` carrying the rostral specificity. 4 cells, 2 with dissociative overrides.
- **ACC** (dorsal ACC) — error monitoring; conflict-related elevation across both subtypes. 3 cells.
- **OFC** — appraisal of trauma-related cues. 2 cells.
- **vmPFC** — fear-extinction circuit; reduced extinction recall (Milad 2009). Shared across both subtypes. 4 cells.
- **dlPFC** — top-down regulation deficit (working memory and attention deficits in PTSD). 3 cells.

### Limbic regions

- **Amygdala** — central in hyperarousal pattern; hyperreactivity to trauma cues and threat (Etkin & Wager 2007; Pitman 2012). Reversed in dissociative subtype (under-activation; Lanius 2010). 8 cells, 4 with dissociative overrides.
- **Hippocampus** — volume reduction (Bremner 1995; Gilbertson 2002; Logue 2018 ENIGMA mega-analysis); context-discrimination deficits; GR sensitivity changes. Shared across both subtypes. 6 cells.

### Subcortical / autonomic

- **vS / NAc** — reward-deficit cells overlap with MDD; anhedonia is a core PTSD symptom. 3 cells.

### Brainstem / regulatory

- **LC** — central in hyperarousal pattern; elevated tonic firing, elevated NE release; partial normalization in dissociative subtype. 4 cells, 1 with dissociative override.
- **Raphe (DRN)** — serotonergic source; reduced 5-HT1A binding across regions; reduced 5-HT tone hypothesized but evidence mixed. 3 cells.
- **VTA** — dopaminergic source; reduced phasic to rewards; overlaps with MDD. 2 cells.

### HPA / endocrine (composite proxy)

- **HPA axis** — distinctive cortisol pattern in PTSD (often *low* basal cortisol with elevated GR sensitivity; Yehuda 2002, 2009). HPA findings are not a "region" in v1 vocabulary; encoded as Cortex (composite) and Hippocampus (composite) cells with explicit HPA-axis targets. 3 cells.

### Cortical-wide / functional

- **Cortex (composite — insular proxy)** — anterior insula central to interoception and the hyperarousal/dissociative distinction (Lanius 2010); over-engaged in hyperarousal, under-engaged in dissociative. 2 cells with full dissociative overrides.
- **DMN composite** — altered DMN connectivity in PTSD, more pronounced in dissociative subtype (Bluhm 2009). 2 cells, 1 with dissociative override.

## Cell registry (excerpted)

| Cell ID | Region | System | Target | Site | δ (hyper) | δ (dissoc) | range | tier | conf | weights {H,R,V,N,D} | sources |
|---|---|---|---|---|---|---|---|---|---|---|---|
| ptsd.amygdala.5HT.5HT1A.post-syn | Amygdala | 5HT | 5HT1A | post-syn | −2 | −2 | [−3,−1] | 2 | M | {0.30, 0.20, 0.20, 0.20, 0.10} | Bonne 2005; Sullivan 2013 |
| ptsd.amygdala.GABA.GABA-A/BZ.post-syn | Amygdala | GABA | GABA-A/BZ-site | post-syn | −2 | −2 | [−3,−1] | 2 | M | {0.30, 0.20, 0.20, 0.20, 0.10} | Geuze 2008 (BZ binding); inferred |
| ptsd.amygdala.NE.alpha1.post-syn | Amygdala | NE | α1-adrenergic | post-syn | +2 | +1 | [+1,+3] | 2 | M | {0.50, 0.20, 0.10, 0.10, 0.10} | Pitman 2012; Raskind 2013 (prazosin) |
| ptsd.amygdala.NE.tone.tone | Amygdala | NE | NE tone | tone | +2 | +1 | [+1,+3] | 2 | M | {0.50, 0.20, 0.10, 0.10, 0.10} | Pitman 2012 |
| ptsd.amygdala.Glu.NMDA.post-syn | Amygdala | Glu | NMDA | post-syn | +1 | +1 | [0,+2] | 2 | M | {0.20, 0.50, 0.10, 0.10, 0.10} | Rosso 2017 MRS |
| ptsd.amygdala.functional.trauma-reactivity.functional | Amygdala | Composite | Trauma-cue BOLD reactivity | functional | +3 | −1 | [−2,+3] | 1 | H | {0.40, 0.40, 0.10, 0.05, 0.05} | Lanius 2010 meta; Hayes 2012 |
| ptsd.amygdala.functional.mPFC-connectivity.functional | Amygdala | Composite | Amygdala–mPFC functional connectivity | functional | −2 | +2 | [−3,+3] | 1 | H | {0.30, 0.20, 0.10, 0.15, 0.25} | Lanius 2010; Nicholson 2015 DCM |
| ptsd.amygdala.functional.startle.functional | Amygdala | Composite | Startle-potentiation BOLD | functional | +2 | 0 | [−1,+3] | 1 | M | {0.60, 0.20, 0.10, 0.05, 0.05} | Pole 2007 |
| ptsd.mPFC.5HT.5HT1A.post-syn | mPFC | 5HT | 5HT1A | post-syn | −1 | −1 | [−2,0] | 2 | L | {0.20, 0.20, 0.15, 0.30, 0.15} | Sullivan 2013 |
| ptsd.mPFC.functional.regulation.functional | mPFC | Composite | Top-down regulation of amygdala | functional | −2 | +2 | [−3,+3] | 1 | H | {0.20, 0.20, 0.10, 0.20, 0.30} | Lanius 2010; Nicholson 2015 |
| ptsd.mPFC.functional.self-referential.functional | mPFC | Composite | Self-referential / negative-cognition BOLD | functional | +1 | +2 | [0,+3] | 2 | M | {0.10, 0.10, 0.05, 0.50, 0.25} | Bluhm 2009; Lanius 2014 |
| ptsd.mPFC.functional.depersonalization.functional | mPFC | Composite | Depersonalization-related mPFC activation | functional | 0 | +3 | [0,+3] | 1 | H | {0.05, 0.05, 0.05, 0.10, 0.75} | Lanius 2010; Hopper 2007 |
| ptsd.mPFC.GABA.tone.tone | mPFC | GABA | GABA tone | tone | −1 | −1 | [−2,0] | 2 | L | {0.20, 0.15, 0.15, 0.30, 0.20} | Rosso 2017 MRS |
| ptsd.mPFC.Glu.tone.tone | mPFC | Glu | Glu tone | tone | +1 | +1 | [0,+2] | 2 | L | {0.20, 0.20, 0.15, 0.30, 0.15} | Rosso 2017 |
| ptsd.ACC.5HT.5HT1A.post-syn | ACC | 5HT | 5HT1A (rostral) | post-syn | −1 | −1 | [−2,0] | 2 | M | {0.20, 0.20, 0.15, 0.30, 0.15} | Sullivan 2013 |
| ptsd.ACC.functional.rACC-regulation.functional | ACC | Composite | Rostral ACC regulation of limbic activity | functional | −2 | +2 | [−3,+3] | 1 | H | {0.20, 0.20, 0.10, 0.20, 0.30} | Lanius 2010; Hopper 2007 |
| ptsd.ACC.functional.dACC-error.functional | ACC | Composite | Dorsal ACC error/conflict monitoring | functional | +2 | +1 | [0,+3] | 2 | M | {0.30, 0.20, 0.20, 0.20, 0.10} | Etkin 2011 (anxiety meta) |
| ptsd.ACC.GABA.tone.tone | ACC | GABA | GABA tone | tone | −1 | −1 | [−2,0] | 2 | L | {0.20, 0.20, 0.15, 0.30, 0.15} | Rosso 2017 |
| ptsd.OFC.functional.appraisal.functional | OFC | Composite | Threat-appraisal BOLD | functional | +1 | 0 | [−1,+2] | 2 | L | {0.30, 0.30, 0.20, 0.10, 0.10} | inferred |
| ptsd.OFC.5HT.5HT2A.post-syn | OFC | 5HT | 5HT2A | post-syn | +1 | +1 | [0,+2] | 3 | L | {0.30, 0.30, 0.20, 0.10, 0.10} | inferred from anxiety spectrum |
| ptsd.vmPFC.5HT.5HT1A.post-syn | vmPFC | 5HT | 5HT1A | post-syn | −1 | −1 | [−2,0] | 2 | M | {0.15, 0.20, 0.40, 0.15, 0.10} | Sullivan 2013 |
| ptsd.vmPFC.functional.extinction.functional | vmPFC | Composite | Fear-extinction recall | functional | −2 | −2 | [−3,−1] | 1 | H | {0.10, 0.30, 0.50, 0.05, 0.05} | Milad 2009 |
| ptsd.vmPFC.functional.safety-signal.functional | vmPFC | Composite | Safety-signal processing | functional | −1 | −1 | [−2,0] | 2 | M | {0.20, 0.20, 0.40, 0.10, 0.10} | Jovanovic 2010 |
| ptsd.vmPFC.composite.volume.composite | vmPFC | Composite | vmPFC gray-matter volume | composite | −1 | 0 | [−2,+1] | 2 | M | {0.20, 0.20, 0.30, 0.20, 0.10} | Karl 2006 meta |
| ptsd.dlPFC.functional.working-memory.functional | dlPFC | Composite | Working memory BOLD | functional | −2 | −1 | [−3,0] | 1 | M | {0.30, 0.10, 0.10, 0.40, 0.10} | Aupperle 2012 |
| ptsd.dlPFC.NE.alpha2.post-syn | dlPFC | NE | α2A-adrenergic | post-syn | −1 | −1 | [−2,0] | 2 | L | {0.30, 0.10, 0.10, 0.40, 0.10} | Arnsten 2015 |
| ptsd.dlPFC.functional.reappraisal.functional | dlPFC | Composite | Cognitive reappraisal engagement | functional | −2 | −1 | [−3,0] | 2 | M | {0.20, 0.20, 0.10, 0.40, 0.10} | New 2009 |
| ptsd.hippocampus.composite.volume.composite | Hippocampus | Composite | Hippocampal volume | composite | −2 | −2 | [−3,−1] | 1 | H | {0.20, 0.30, 0.20, 0.20, 0.10} | Bremner 1995; Gilbertson 2002; Logue 2018 ENIGMA |
| ptsd.hippocampus.5HT.5HT1A.post-syn | Hippocampus | 5HT | 5HT1A | post-syn | −1 | −1 | [−2,0] | 2 | M | {0.20, 0.30, 0.20, 0.20, 0.10} | Sullivan 2013 |
| ptsd.hippocampus.Glu.NMDA.post-syn | Hippocampus | Glu | NMDA | post-syn | +1 | +1 | [0,+2] | 2 | M | {0.20, 0.30, 0.20, 0.20, 0.10} | Rosso 2017; chronic-stress framework |
| ptsd.hippocampus.composite.GR-sensitivity.composite | Hippocampus | Composite | Glucocorticoid receptor sensitivity | composite | +2 | +2 | [+1,+3] | 1 | H | {0.20, 0.20, 0.15, 0.30, 0.15} | Yehuda 2002, 2009 |
| ptsd.hippocampus.functional.context-discrimination.functional | Hippocampus | Composite | Context vs threat discrimination | functional | −2 | −1 | [−3,0] | 1 | M | {0.20, 0.40, 0.20, 0.10, 0.10} | Lissek 2010; Acheson 2015 |
| ptsd.hippocampus.GABA.tone.tone | Hippocampus | GABA | GABA tone | tone | −1 | −1 | [−2,0] | 3 | L | {0.20, 0.20, 0.20, 0.20, 0.20} | inferred |
| ptsd.vS.DA.D2.post-syn | vS | DA | D2/D3 | post-syn | −1 | −1 | [−2,0] | 2 | L | {0.05, 0.05, 0.10, 0.70, 0.10} | inferred from anhedonia overlap with MDD |
| ptsd.vS.functional.reward-anticipation.functional | vS | Composite | Reward-anticipation BOLD | functional | −2 | −2 | [−3,−1] | 1 | M | {0.05, 0.05, 0.10, 0.70, 0.10} | Elman 2009; Hägele 2015 |
| ptsd.vS.DA.tone.tone | vS | DA | DA tone | tone | −1 | −1 | [−2,0] | 2 | L | {0.05, 0.05, 0.10, 0.70, 0.10} | inferred |
| ptsd.LC.NE.tone.tone | LC | NE | NE tone | tone | +3 | +1 | [0,+3] | 1 | H | {0.60, 0.20, 0.05, 0.10, 0.05} | Geracioti 2001 CSF NE; Strawn 2008 |
| ptsd.LC.NE.dynamic.dynamic | LC | NE | LC tonic + phasic firing | dynamic | +2 | +1 | [0,+3] | 1 | M | {0.60, 0.20, 0.05, 0.10, 0.05} | Pitman 2012 review |
| ptsd.LC.NE.alpha2.auto | LC | NE | α2A-adrenergic | auto | −1 | −1 | [−2,0] | 2 | M | {0.50, 0.20, 0.10, 0.10, 0.10} | Southwick 1997 (yohimbine) |
| ptsd.LC.NE.NET.pre-syn | LC | NE | NET | pre-syn | 0 | 0 | [−1,+1] | 3 | L | {0.50, 0.20, 0.10, 0.10, 0.10} | inferred |
| ptsd.raphe.5HT.5HT1A.auto | Raphe | 5HT | 5HT1A | auto | +1 | +1 | [0,+2] | 2 | L | {0.20, 0.20, 0.20, 0.30, 0.10} | inferred |
| ptsd.raphe.5HT.SERT.pre-syn | Raphe | 5HT | SERT | pre-syn | 0 | 0 | [−1,+1] | 2 | L | {0.20, 0.20, 0.20, 0.30, 0.10} | Murrough 2011 (contested) |
| ptsd.raphe.5HT.tone.tone | Raphe | 5HT | 5HT tone (cortical) | tone | −1 | −1 | [−2,0] | 2 | L | {0.20, 0.20, 0.20, 0.30, 0.10} | inferred |
| ptsd.VTA.DA.dynamic.dynamic | VTA | DA | Phasic firing to rewards | dynamic | −1 | −1 | [−2,0] | 3 | L | {0.05, 0.05, 0.10, 0.70, 0.10} | inferred from striatal findings |
| ptsd.VTA.DA.tone.tone | VTA | DA | VTA tonic firing | tone | −1 | −1 | [−2,0] | 3 | L | {0.05, 0.05, 0.10, 0.70, 0.10} | inferred |
| ptsd.cortex.composite.cortisol-basal.composite | Cortex (composite) | Composite | Basal cortisol (HPA proxy) | composite | −1 | −1 | [−2,0] | 1 | H | {0.30, 0.10, 0.10, 0.30, 0.20} | Yehuda 2002, 2009 (paradoxically low basal) |
| ptsd.cortex.composite.cortisol-dex-suppression.composite | Cortex (composite) | Composite | Cortisol dex-suppression (enhanced) | composite | +2 | +2 | [+1,+3] | 1 | H | {0.20, 0.20, 0.10, 0.30, 0.20} | Yehuda 2002 |
| ptsd.cortex.composite.CRH-tone.composite | Cortex (composite) | Composite | CRH tone (CSF) | composite | +2 | +1 | [0,+3] | 2 | M | {0.40, 0.20, 0.10, 0.20, 0.10} | Bremner 1997 CSF CRH |
| ptsd.cortex.composite.insular-interoception.composite | Cortex (composite) | Composite | Anterior insular interoception (proxy region) | composite | +2 | −2 | [−3,+3] | 1 | H | {0.30, 0.20, 0.10, 0.10, 0.30} | Lanius 2010, 2014; Hopper 2007 |
| ptsd.cortex.composite.insular-volume.composite | Cortex (composite) | Composite | Insular gray-matter volume | composite | +1 | 0 | [−1,+2] | 2 | L | {0.30, 0.20, 0.10, 0.20, 0.20} | Kasai 2008 |
| ptsd.cortex.composite.DMN-connectivity.composite | Cortex (composite) | Composite | DMN intra-network connectivity | composite | −1 | −2 | [−3,0] | 2 | M | {0.10, 0.10, 0.10, 0.30, 0.40} | Bluhm 2009; Daniels 2010 |
| ptsd.cortex.composite.salience-network.composite | Cortex (composite) | Composite | Salience network connectivity | composite | +1 | −1 | [−2,+2] | 2 | M | {0.30, 0.20, 0.10, 0.20, 0.20} | Sripada 2012 |

Subsystem weights cross-check: each row sums to 1.0 ± 0.01. Rows with subtype overrides apply the override delta when `phenotype_subtype == "dissociative"` is active on the profile.

## Key authoring choices

### 1. Subtype-conditional cells encode the reversed circuit pattern, not separate templates

The DSM-5 dissociative subtype shares ~75% of cells with the hyperarousal canonical baseline. The remaining ~25% — primarily the amygdala-mPFC affect-regulation circuit, the insular interoception cells, the rACC regulation cell, and selected LC tonic cells — show **directionally reversed** values in the dissociative pattern. Two design options were considered:

**Option A (rejected):** Two sibling templates (`ptsd_hyperarousal_v1`, `ptsd_dissociative_v1`). Rejected because of the ~75% cell overlap; maintenance burden doubles and audit-rule false-positives surge whenever shared cells need updates.

**Option B (chosen):** One template with `subtype_branches` and per-cell `subtype_overrides`. Cells without overrides apply identically to both branches. Cells with overrides apply the override delta when the profile's `phenotype_subtype` matches. This is a minor schema extension (proposed v3.1 additive) — tracked in `15-schema-extensions.md` as v3.1.SUB-1. Until landed, the template uses an authoring convention with values in `notes` and a thin runtime lookup table.

The 13 cells with dissociative overrides cluster around four anatomic-functional patterns:

- **Amygdala under-activation** (dissociative) vs over-activation (hyperarousal): `ptsd.amygdala.functional.trauma-reactivity.functional`, `ptsd.amygdala.functional.startle.functional`.
- **mPFC / rACC oversuppression** (dissociative) vs hypoactivation (hyperarousal): `ptsd.mPFC.functional.regulation.functional`, `ptsd.ACC.functional.rACC-regulation.functional`, `ptsd.amygdala.functional.mPFC-connectivity.functional`.
- **Depersonalization-specific cell** (dissociative only): `ptsd.mPFC.functional.depersonalization.functional`.
- **Insular interoception** (over-engaged in hyperarousal, under-engaged in dissociative): `ptsd.cortex.composite.insular-interoception.composite`.

### 2. The D subsystem is dormant in hyperarousal-branch profiles

For a profile with `phenotype_subtype == "hyperarousal"`, subsystem `D` (dissociative) is dormant: PCL-5 dissociation items still score, but the elicitation map applies the D-subsystem delta_modifier at 0 (no effect) for the hyperarousal branch. For `phenotype_subtype == "dissociative"`, the D subsystem is active and PCL-5 dissociation items drive D-subsystem deltas. This is encoded in the ElicitationMap, not in cell weights — cell weights are static across branches; the branch determines which subsystem modifiers actually fire.

### 3. Hippocampal volume reduction encoded as a `composite` cell, not a receptor cell

`ptsd.hippocampus.composite.volume.composite` carries delta_best = −2 (range [−3, −1]) with high confidence (Bremner 1995, Gilbertson 2002 twin study, Logue 2018 ENIGMA mega-analysis N>1800). This is the most-replicated structural finding in PTSD and gets first-class encoding via the `composite` site type. Gilbertson's twin study suggests hippocampal volume reduction is partly a pre-existing vulnerability marker — encoded in notes; pending a `state-trait` flag.

### 4. HPA-axis cells encoded as Cortex (composite) proxies

The HPA axis is central to PTSD neurobiology (low basal cortisol with elevated GR sensitivity, enhanced dexamethasone suppression — Yehuda 2002, 2009; elevated CSF CRH — Bremner 1997). v1 region vocabulary does not include "Hypothalamus" or "Pituitary" or "Adrenal." HPA findings are encoded as Cortex (composite) cells with explicit HPA-axis targets and notes flagging the vocabulary gap. The hippocampal GR-sensitivity cell sits in the Hippocampus region where it anatomically belongs. Action item: add Hypothalamus to v2 vocabulary; pending in `15-schema-extensions.md`.

### 5. Anhedonia cells overlap with MDD

`ptsd.vS.functional.reward-anticipation.functional` carries delta_best = −2 and overlaps mechanistically with MDD's reward-deficit cells. The N subsystem (negative cognition/mood) inherits much of MDD's neurobiology. This is a known overlap; the composition rules with MDD will need to handle it without double-counting. Pure-PTSD weight is 0.70 on N for these cells; in a comorbid profile, the MDD template's parallel cells get suppressed by the composition rule (to be specified).

### 6. LC NE elevation is the strongest pharmacology-actionable finding

`ptsd.LC.NE.tone.tone` (delta_best = +3 in hyperarousal, +1 in dissociative) and `ptsd.LC.NE.dynamic.dynamic` together encode the Geracioti 2001 CSF NE finding plus the broader noradrenergic literature. These cells are the mechanistic anchor for prazosin (α1 antagonist) and clonidine/guanfacine (α2 agonists). The dissociative-branch override (+1 vs +3) reflects literature suggesting reduced noradrenergic outflow in dissociative presentations (Lanius 2010 review).

### 7. SERT in PTSD is `contested: methodological`

Same authoring decision as GAD, panic, and SAD. Murrough 2011 and related PET work show heterogeneous SERT findings across methods and across acute vs. chronic cohorts. `ptsd.raphe.5HT.SERT.pre-syn` is `delta_best: 0`, `contested: methodological`, notes carry the conflict explanation.

### 8. Brain Type 6 anchor

PTSD's neurobiology is the native anchor for Type 6 (Trauma-Imprinted) in `20-brain-types.md`. The type description in §20 already articulates the dual-pattern phenotype; this template is the cell-level realization of that type. The type chip preselects Type 6 when the PTSD template is referenced. The dissociative phenotype branch is the type's "dissociative" variant — natively encoded.

## ElicitationMap reference

```yaml
id: pcl-5.v1
instrument: PCL-5 (PTSD Checklist for DSM-5)
applies_to_templates: [ptsd_canonical_v1]
recency_window_days: 14
license_status: free-clinical
source_citation: "Weathers FW, Litz BT, Keane TM, Palmieri PA, Marx BP, Schnurr PP \
                  (2013). The PTSD Checklist for DSM-5 (PCL-5). National Center \
                  for PTSD. https://www.ptsd.va.gov/professional/assessment/\
                  adult-sr/ptsd-checklist.asp"
```

### Scoring rules

PCL-5 has 20 items, each 0–4, total 0–80.

- `total` = items[1..20].sum, range [0, 80]
- `cluster_B_reexperiencing` = items[1..5].sum (5 items, range [0, 20])
- `cluster_C_avoidance` = items[6..7].sum (2 items, range [0, 8])
- `cluster_D_negative_cognition` = items[8..14].sum (7 items, range [0, 28])
- `cluster_E_hyperarousal` = items[15..20].sum (6 items, range [0, 24])
- `dissociation_proxy` = items[8] (negative beliefs about self/world; weakest proxy) + items[10] (blame, weak proxy)
  - **Note:** PCL-5 does not have dedicated dissociation items. Branch assignment to "dissociative" subtype should not rely on PCL-5 alone; recommend MDI or DES-II as a paired instrument. v1 ships with the PCL-5 ElicitationMap and adds a placeholder dissociation_proxy that is flagged `evidence_status: inferred`, `confidence: L`.

### Subsystem mappings

| Scoring | Subsystem | Formula | Evidence | Confidence | Rationale |
|---|---|---|---|---|---|
| cluster_E_hyperarousal | H | (score − 8) / 6 | inferred | M | Hyperarousal cluster directly indexes LC/amygdala-driven autonomic output |
| cluster_B_reexperiencing | R | (score − 7) / 5 | inferred | M | Re-experiencing cluster indexes amygdala-hippocampal intrusion circuit |
| cluster_C_avoidance | V | (score − 3) / 3 | inferred | M | Avoidance cluster (2 items only); coarse but specific |
| cluster_D_negative_cognition | N | (score − 10) / 8 | inferred | M | Negative cognition/mood cluster indexes mPFC/vS reward-deficit pattern |
| dissociation_proxy | D | (score − 2) / 4 | inferred | L | PCL-5 weak proxy; only applied for `phenotype_subtype == "dissociative"`; otherwise dormant |
| total | H | (score − 33) / 20 | inferred | L | Total scaling at moderate severity threshold (~33, common clinical cutoff) |

All coefficients are starting calibrations and require pilot validation. **The dissociation_proxy mapping should be replaced or supplemented by MDI/DES-II in v2.**

### Cell-level mappings

None in v1. PCL-5 does not have multi-study replication supporting cell-level overrides. All effects route through subsystem weights.

### AI extraction targets

- **Pattern: hyperarousal symptoms predominant.** Cells: `ptsd.LC.NE.tone.tone`, `ptsd.amygdala.functional.startle.functional`. Phrasings: "always on guard", "startle at everything", "can't sleep through the night".
- **Pattern: re-experiencing / intrusions.** Cells: `ptsd.amygdala.functional.trauma-reactivity.functional`, `ptsd.hippocampus.functional.context-discrimination.functional`. Phrasings: "flashbacks", "nightmares about it", "smells/sounds put me right back there".
- **Pattern: avoidance.** Cells: `ptsd.vmPFC.functional.extinction.functional`, `ptsd.vmPFC.functional.safety-signal.functional`. Phrasings: "won't drive past that street", "can't watch the news", "avoid anything that reminds me".
- **Pattern: negative cognition / anhedonia / detachment.** Cells: `ptsd.vS.functional.reward-anticipation.functional`, `ptsd.mPFC.functional.self-referential.functional`. Phrasings: "feel numb", "don't enjoy anything anymore", "the world is dangerous".
- **Pattern: dissociation (depersonalization / derealization).** Cells: `ptsd.mPFC.functional.depersonalization.functional`, `ptsd.cortex.composite.insular-interoception.composite`. Phrasings: "feel outside my body", "things don't feel real", "watching myself from outside", "go blank when triggered". `evidence_strength_required: explicit_test` — patient often does not volunteer dissociation; clinician should screen with MDI or DES-II before branch assignment.

## Branch-assignment logic

The phenotype branch (hyperarousal vs dissociative) drives override application. Authoring convention for v1 (until v3.1.SUB-1 lands in schema):

1. **Default branch is `hyperarousal`.** New PTSD profiles initialize to this branch.
2. **Branch is reassigned to `dissociative`** when one of:
   - MDI (Multiscale Dissociation Inventory) total ≥ 60 (clinically significant dissociation)
   - DES-II (Dissociative Experiences Scale) total ≥ 30
   - PCL-5 alone is insufficient; the `dissociation_proxy` score is informative only as a screen-positive flag triggering MDI/DES-II administration
   - Clinician direct assignment based on history (depersonalization/derealization episodes during trauma response)
3. **Branch is clinician-committed.** AI proposes from instrument scores; clinician confirms.
4. **Branch can switch** if presentation changes across visits, but this is a meaningful re-classification event tracked in the audit log (parallels brain-type reassignment in `20-brain-types.md`).

## Narrative summary

PTSD's neurobiology is the trauma-imprinted nervous system: a fear circuit that learned the world was dangerous and hasn't unlearned it. Four observations anchor the template.

First, the **fear circuit centerpiece is the amygdala-mPFC-hippocampus triad.** In the hyperarousal pattern (canonical, ~70–85% of patients), amygdala reactivity to trauma cues is elevated (Lanius 2010 meta-analysis; Hayes 2012); mPFC and rostral ACC top-down regulation is reduced; hippocampal context-discrimination is impaired (Lissek 2010; Acheson 2015). The amygdala is loud; the prefrontal regulator is quiet.

Second, the **dissociative subtype reverses this pattern.** In ~15–30% of PTSD patients (Lanius 2010; Wolf 2012; DSM-5 inclusion), the mPFC and rostral ACC oversuppress the amygdala — emotional overmodulation rather than overflow. Patients describe feeling "outside themselves," watching events from a distance, experiencing depersonalization and derealization. Dynamic causal modeling (Nicholson 2015) shows the connectivity direction is reversed: top-down (mPFC → amygdala) in dissociative, bottom-up (amygdala → mPFC) in hyperarousal. The insula is over-engaged in hyperarousal (vivid interoception) and under-engaged in dissociative (interoceptive blunting). This is the rationale for encoding the dissociative subtype as a phenotype branch with directionally reversed cells on the affect-regulation circuit, not as a separate disorder template.

Third, the **hippocampal volume reduction is the most-replicated structural finding** in PTSD (Bremner 1995; Gilbertson 2002 twin study; Logue 2018 ENIGMA mega-analysis). The Gilbertson twin design suggests this is partly a vulnerability marker, not purely a consequence — relevant for predicting onset risk. The hippocampal GR sensitivity is elevated (Yehuda 2002, 2009), part of the broader HPA-axis dysregulation: paradoxically low basal cortisol, enhanced dexamethasone suppression, elevated CSF CRH (Bremner 1997). This dissociates PTSD's HPA pattern from major depression's (where cortisol is typically elevated with reduced suppression).

Fourth, the **LC noradrenergic system is hyperactive in the hyperarousal pattern** (Geracioti 2001 CSF NE; Strawn 2008 review). This is the strongest pharmacology-actionable finding: prazosin (α1 antagonist) reduces nightmares and sleep-onset hyperarousal (Raskind 2013 — though replication concerns in subsequent VA trials); clonidine and guanfacine (α2 agonists) modestly reduce hyperarousal. The dissociative subtype shows less prominent noradrenergic elevation, consistent with its parasympathetic-dominant "shutdown" phenotype.

The treatment implication: first-line SSRIs (sertraline, paroxetine — both FDA-approved for PTSD) and SNRIs (venlafaxine) hit the serotonergic limbic cells. Prazosin covers the α1 cells for the hyperarousal subtype specifically. Trauma-focused psychotherapy (prolonged exposure, CPT, EMDR) covers the conditioned-avoidance (`V`) and re-experiencing (`R`) subsystems through experiential mechanisms. The **dissociative subtype requires different treatment sequencing** — direct trauma exposure can worsen dissociation before improving it; stabilization-phase work (somatic, IFS, grounding skills) typically precedes processing work. The treatment-fit table should respect the branch.

## v1 readiness

**Ready:**

- 51 active cells covering hyperarousal, re-experiencing, avoidance, negative-cognition, and dissociative subsystems with explicit subsystem weights.
- Subtype-conditional cells encoded for the dissociative branch with reversed deltas on 13 cells.
- PCL-5 ElicitationMap drafted with subsystem-default mappings; coefficients calibrated to literature direction.
- Brain Type 6 (Trauma-Imprinted) anchor populated and matches the type's native dual-pattern phenotype.
- Hippocampal volume cell encoded with highest-confidence evidence (ENIGMA mega-analysis).

**Not ready (blockers tracked in `11-readiness-and-blockers.md`):**

- **Schema extension v3.1.SUB-1.** `subtype_branches` and per-cell `subtype_overrides` need formal schema admission. Until then, the template uses an authoring convention with notes-embedded override values and a thin runtime lookup table.
- **Dissociation screening instrument.** PCL-5 alone is insufficient to assign the dissociative branch. The v2 ElicitationMap library should add MDI or DES-II. Until then, branch assignment defaults to hyperarousal unless the clinician overrides based on history.
- **Drug coverage cells not yet populated.** First priority: sertraline (FDA-approved for PTSD), paroxetine (FDA-approved), venlafaxine, prazosin (hyperarousal-specific), topiramate, propranolol. Mirtazapine for sleep. Benzodiazepines flagged as **relatively contraindicated** in PTSD (worsening evidence — Guina 2015 meta).
- **Region vocabulary gaps.** Insula, hypothalamus, pituitary — all referenced in PTSD literature, all encoded via Cortex (composite) proxies. Pending v2 vocabulary extension.
- **Pilot calibration of PCL-5 → subsystem coefficients.** First-pass.
- **Complex PTSD (cPTSD).** ICD-11 introduces cPTSD as a distinct entity; v1 handles it as a PatientSubsystemModifier rather than a separate template. Pilot data may justify a curated cPTSD template, especially for childhood-onset trauma populations.
- **Composition rules with MDD, anxiety templates.** PTSD + MDD is highly prevalent (~50%); avoid double-counting anhedonia cells. Composition pilot work pending.
- **State-trait conflict typology.** Several cells (hippocampal volume — vulnerability vs. consequence; LC NE — acute vs. chronic) need explicit `contested: state-trait` tagging once a clinical reviewer audits.

## References (anchoring sources, partial; full source list lives in registry)

- Lanius RA et al. (2010). Emotion modulation in PTSD: clinical and neurobiological evidence for a dissociative subtype. *Am J Psychiatry.* https://pubmed.ncbi.nlm.nih.gov/20360318/
- Lanius RA et al. (2012). The dissociative subtype of posttraumatic stress disorder: rationale, clinical and neurobiological evidence, and implications. *Depress Anxiety.* https://onlinelibrary.wiley.com/doi/10.1002/da.21889
- Lanius RA, Bluhm RL, Frewen PA (2014). How understanding the neurobiology of complex post-traumatic stress disorder can inform clinical practice: a social cognitive and affective neuroscience approach. *Acta Psychiatr Scand.*
- Nicholson AA et al. (2015). Dynamic causal modeling in PTSD and its dissociative subtype: bottom-up versus top-down processing within fear and emotion regulation circuitry. *Hum Brain Mapp.* https://pubmed.ncbi.nlm.nih.gov/28836726/
- Hopper JW, Frewen PA, van der Kolk BA, Lanius RA (2007). Neural correlates of reexperiencing, avoidance, and dissociation in PTSD: symptom dimensions and emotion dysregulation in responses to script-driven trauma imagery. *J Trauma Stress.*
- Lebois LAM et al. (2023). Neurobiological and Genetic Correlates of the Dissociative Subtype of PTSD. https://pmc.ncbi.nlm.nih.gov/articles/PMC10286858/
- Daniels JK et al. (2016). Switching between executive and default mode networks in posttraumatic stress disorder. *J Psychiatry Neurosci.*
- Wolf EJ et al. (2012). A latent class analysis of dissociation and posttraumatic stress disorder. *Arch Gen Psychiatry.* https://pubmed.ncbi.nlm.nih.gov/22566561/ — DS prevalence.
- Bremner JD et al. (1995). MRI-based measurement of hippocampal volume in patients with combat-related PTSD. *Am J Psychiatry.*
- Gilbertson MW et al. (2002). Smaller hippocampal volume predicts pathologic vulnerability to psychological trauma. *Nat Neurosci.* https://pubmed.ncbi.nlm.nih.gov/12379862/ — twin study.
- Logue MW et al. (2018). Smaller hippocampal volume in posttraumatic stress disorder: a multisite ENIGMA-PGC study. *Biol Psychiatry.* https://pubmed.ncbi.nlm.nih.gov/29217296/ — N>1800 mega-analysis.
- Karl A et al. (2006). A meta-analysis of structural brain abnormalities in PTSD. *Neurosci Biobehav Rev.*
- Kasai K et al. (2008). Evidence for acquired pregenual anterior cingulate gray matter loss from a twin study of combat-related posttraumatic stress disorder. *Biol Psychiatry.*
- Yehuda R (2002, 2009). Status of cortisol findings in PTSD. *Endocrinol Metab Clin North Am / Psychiatr Clin North Am.* — HPA axis framework.
- Bremner JD et al. (1997). Elevated CSF corticotropin-releasing factor concentrations in posttraumatic stress disorder. *Am J Psychiatry.*
- Geracioti TD Jr et al. (2001). CSF norepinephrine concentrations in posttraumatic stress disorder. *Am J Psychiatry.* https://pubmed.ncbi.nlm.nih.gov/11481156/
- Strawn JR, Geracioti TD Jr (2008). Noradrenergic dysfunction and the psychopharmacology of posttraumatic stress disorder. *Depress Anxiety.*
- Raskind MA et al. (2013). A trial of prazosin for combat trauma PTSD with nightmares in active-duty soldiers returned from Iraq and Afghanistan. *Am J Psychiatry.*
- Pitman RK et al. (2012). Biological studies of post-traumatic stress disorder. *Nat Rev Neurosci.* https://pubmed.ncbi.nlm.nih.gov/22968743/
- Milad MR et al. (2009). Neurobiological basis of failure to recall extinction memory in posttraumatic stress disorder. *Biol Psychiatry.* https://pubmed.ncbi.nlm.nih.gov/19748081/
- Etkin A, Wager TD (2007). Functional neuroimaging of anxiety: a meta-analysis of emotional processing in PTSD, social anxiety disorder, and specific phobia. *Am J Psychiatry.* https://pubmed.ncbi.nlm.nih.gov/17898336/
- Hayes JP, Hayes SM, Mikedis AM (2012). Quantitative meta-analysis of neural activity in posttraumatic stress disorder. *Biol Mood Anxiety Disord.*
- Bluhm RL et al. (2009). Alterations in default network connectivity in posttraumatic stress disorder related to early-life trauma. *J Psychiatry Neurosci.*
- Daniels JK et al. (2010). Default mode alterations in posttraumatic stress disorder related to early-life trauma. *J Psychiatry Neurosci.*
- Sripada RK et al. (2012). Neural dysregulation in posttraumatic stress disorder: evidence for disrupted equilibrium between salience and default mode brain networks. *Psychosom Med.*
- Rosso IM et al. (2017). Cortical inhibition deficits in posttraumatic stress disorder revealed by single-voxel proton MRS. *Mol Psychiatry.*
- Bonne O et al. (2005). Reduced posterior hippocampal volume in posttraumatic stress disorder. *J Clin Psychiatry.*
- Sullivan GM et al. (2013). PET imaging of CB1 cannabinoid receptor and SERT in PTSD. *Mol Psychiatry / J Nucl Med.*
- Geuze E et al. (2008). Reduced GABAA benzodiazepine receptor binding in veterans with post-traumatic stress disorder. *Mol Psychiatry.* https://pubmed.ncbi.nlm.nih.gov/17389905/
- Southwick SM et al. (1997). Yohimbine use in a natural setting: effects in posttraumatic stress disorder. *Biol Psychiatry.*
- Murrough JW et al. (2011). The effect of early trauma exposure on serotonin type 1B receptor expression revealed by reduced selective radioligand binding. *Arch Gen Psychiatry.*
- Acheson DT et al. (2015). The effect of pretrauma cortisol and trauma-induced changes in cortisol on adult hippocampal volume in young men. *Psychoneuroendocrinology.*
- Lissek S et al. (2010). Overgeneralization of conditioned fear as a pathogenic marker of PTSD. *Am J Psychiatry.*
- Aupperle RL et al. (2012). Neural systems underlying approach and avoidance in anxiety disorders. *Dialogues Clin Neurosci.*
- Pole N (2007). The psychophysiology of posttraumatic stress disorder: a meta-analysis. *Psychol Bull.*
- New AS et al. (2009). A functional magnetic resonance imaging study of deliberate emotion regulation in resilience and posttraumatic stress disorder. *Biol Psychiatry.*
- Elman I et al. (2009). Functional neuroimaging of reward circuitry responsivity to monetary gains and losses in posttraumatic stress disorder. *Biol Psychiatry.*
- Hägele C et al. (2015). Dimensional psychiatry: reward dysfunction and depressive mood across psychiatric disorders. *Psychopharmacology.*
- Guina J et al. (2015). Benzodiazepines for PTSD: a systematic review and meta-analysis. *J Psychiatr Pract.* — benzodiazepine harm evidence.
- Weathers FW et al. (2013). The PTSD Checklist for DSM-5 (PCL-5). National Center for PTSD. https://www.ptsd.va.gov/professional/assessment/adult-sr/ptsd-checklist.asp
- NIH-PMC overview: *The Dissociative Subtype of PTSD: Unique Resting-State Functional Connectivity of Basolateral and Centromedial Amygdala Complexes.* https://www.nature.com/articles/npp201579
- Frontiers review: *The neural circuits and molecular mechanisms underlying fear dysregulation in posttraumatic stress disorder.* https://www.frontiersin.org/journals/neuroscience/articles/10.3389/fnins.2023.1281401/full
- VA *Research Quarterly: The Dissociative Subtype of PTSD.* https://www.ptsd.va.gov/publications/rq_docs/V29N3.pdf

---

Last updated: 2026-05-13 · Draft awaiting clinical review.
