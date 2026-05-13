# 24 — Panic Disorder Template

> **CLINICAL REVIEW REQUIRED.** This file is a first-draft DisorderTemplate authored by the framework team for v1 review. Cell deltas, subsystem weights, and elicitation coefficients are starting calibrations grounded in the cited literature but have not been clinician-committed. Do not deploy to patient profiles before a clinical reviewer signs off on `last_reviewed` / `reviewer` fields and a pilot calibration pass against intake data is complete.

A canonical DisorderTemplate for moderate, unmedicated, trait-baseline Panic Disorder (with or without agoraphobia). Mirrors the structure, depth, and tone of `09-ocd-reference-instantiation.md`.

---

## Template metadata

```yaml
id: panic_canonical_v1
schema_version: "3.0"
disorder: Panic
template_version: "1.0.0"
dsm5_code: "300.01"                  # 300.22 if with agoraphobia
icd11_code: "6B01"
severity_bucket: moderate
phenotype_subtype: null               # baseline; agoraphobia handled by modifier
baseline_ref: healthy_v1
elicitation_map_ref: pdss-sr.v1
brain_type_anchor: type_1            # Anxious-Vigilant (shared with GAD)
status: draft
last_reviewed: 2026-05-13
reviewer: framework-team
notes: |
  Panic disorder presents with recurrent unexpected panic attacks plus the
  meta-anxiety about attack recurrence (anticipatory anxiety). The
  neurobiology centers on the brainstem fear circuit (LC, raphe, PAG),
  amygdala-driven suffocation-alarm activation, and interoceptive
  hypervigilance via the insula. Brain Type 1 (Anxious-Vigilant) is the
  patient-identity anchor, though panic's autonomic dominance distinguishes
  it from GAD's worry dominance.
```

The template represents **moderate, unmedicated, trait-baseline Panic Disorder**. Agoraphobia is handled as a phenotype modifier rather than a separate template (DSM-5 split them but the neurobiology shares the same core). Severity scales via `severity_factor`.

## Subsystems

Panic disorder's four canonical subsystems:

- **P — Panic attack core circuit.** Acute brainstem-driven fear response: LC tonic and phasic firing, amygdala output to autonomic centers, PAG defensive activation, raphe failure-of-inhibition. The phenotype of the attack itself.
- **A — Anticipatory anxiety.** Inter-attack worry about recurrence; overlaps with GAD's worry axis but is more interoceptively framed. Maps to mPFC, ACC, insula, amygdala.
- **I — Interoceptive hypervigilance.** Heightened awareness of bodily signals (heartbeat, breath); the cognitive substrate for fear-of-fear. Maps to anterior insula, ACC, brainstem afferents. In our v1 region vocabulary, insular findings are encoded via vmPFC/ACC and amygdala proxy cells with notes flagging the insular origin until the vocabulary expands.
- **C — Conditioned avoidance.** Agoraphobic avoidance, situational escape. Maps to vmPFC fear extinction failure, hippocampal context discrimination, amygdala fear consolidation.

Every cell carries `subsystem_weights` distributing its contribution across {P, A, I, C}. Weights for any non-zero cell sum to 1.0 ± 0.01.

## Cell coverage

48 active cells. Coverage by region:

### Cortical regions

- **mPFC** — anticipatory worry, interoceptive monitoring (proxy for insular cells); reduced top-down inhibition of amygdala. 4 cells.
- **ACC** — interoceptive salience; reduced GABA on MRS in panic (Long 2013, Hasler 2008); elevated error-monitoring on attack signals. 5 cells.
- **vmPFC** — fear-extinction circuit; reduced extinction recall in panic (Milad 2014). 3 cells.
- **dlPFC** — top-down regulation; modest involvement vs GAD. 2 cells.
- **OFC** — interoceptive valuation; minor. 1 cell.

### Limbic regions

- **Amygdala** — central panic generator; reduced 5-HT1A binding (Neumeister 2004 PET); reduced GABA-A/BZ binding (Hasler 2008 SPECT, Malizia 1998); elevated reactivity to interoceptive cues. 8 cells.
- **Hippocampus** — context-discrimination deficits driving agoraphobic generalization; 5-HT1A reductions; volume reductions in some studies. 4 cells.

### Subcortical / autonomic

- **vS** — anticipatory and conditioned-reward-loss; minor in pure panic. 2 cells.

### Brainstem / regulatory (this is where panic differs from GAD)

- **Raphe (DRN)** — serotonergic source; the CO2-sensing serotonergic neurons (Severson 2003, Hodges & Richerson 2010); deficient inhibition of PAG (Maron 2011, Graeff 2012). Encoded with explicit dorsal-raphe CO2-chemoreceptor cell. 5 cells.
- **LC** — noradrenergic source; tonic firing elevated; phasic response to interoceptive stimuli amplified. 4 cells.
- **PAG (periaqueductal gray)** — defensive reaction generator; the suffocation-alarm and flight circuit. v1 vocabulary does not include PAG as a region — findings are encoded under Raphe (functional projection) cells with `target` carrying the PAG specificity and notes flagging the vocabulary gap. 3 cells (proxied via Raphe and Midbrain functional cells).
- **VTA** — dopaminergic source; minimal in pure panic. 1 cell low-confidence.

### Cortical-wide / functional

- **Cortex (GABA / BZ composite)** — reduced cortical BZ binding (Malizia 1998 [11C]flumazenil PET; Hasler 2008; Cameron 2007). 4 cells encoded as region-specific composites.
- **Cortex (interoceptive / insular composite)** — insular gray matter increase (Uchida 2008), interoceptive BOLD elevation (Critchley 2004). 2 cells encoded as composite functional cells until vocabulary expands.

## Cell registry (excerpted)

| Cell ID | Region | System | Target | Site | δ | range | tier | conf | weights {P,A,I,C} | sources |
|---|---|---|---|---|---|---|---|---|---|---|
| panic.amygdala.5HT.5HT1A.post-syn | Amygdala | 5HT | 5HT1A | post-syn | −2 | [−3,−1] | 1 | M | {0.40, 0.20, 0.20, 0.20} | Neumeister 2004; Nash 2008 |
| panic.amygdala.5HT.5HT2A.post-syn | Amygdala | 5HT | 5HT2A | post-syn | +2 | [+1,+3] | 2 | M | {0.40, 0.15, 0.25, 0.20} | Maron 2011 |
| panic.amygdala.GABA.GABA-A/BZ.post-syn | Amygdala | GABA | GABA-A/BZ-site | post-syn | −2 | [−3,−1] | 1 | M | {0.40, 0.15, 0.20, 0.25} | Hasler 2008; Malizia 1998 |
| panic.amygdala.NE.alpha1.post-syn | Amygdala | NE | α1-adrenergic | post-syn | +2 | [+1,+3] | 2 | M | {0.50, 0.15, 0.20, 0.15} | Goddard 2017 |
| panic.amygdala.NE.tone.tone | Amygdala | NE | NE tone | tone | +2 | [+1,+3] | 2 | M | {0.50, 0.15, 0.20, 0.15} | Goddard 2017 |
| panic.amygdala.Glu.NMDA.post-syn | Amygdala | Glu | NMDA | post-syn | +1 | [0,+2] | 3 | L | {0.30, 0.20, 0.20, 0.30} | inferred |
| panic.amygdala.functional.interoceptive-reactivity.functional | Amygdala | Composite | Interoceptive-cue reactivity (BOLD) | functional | +3 | [+2,+3] | 1 | H | {0.40, 0.10, 0.40, 0.10} | Domschke 2010; Critchley 2004 |
| panic.amygdala.functional.mPFC-connectivity.functional | Amygdala | Composite | Amygdala–mPFC functional connectivity | functional | −2 | [−3,−1] | 1 | M | {0.30, 0.20, 0.20, 0.30} | Pannekoek 2013 |
| panic.mPFC.5HT.5HT1A.post-syn | mPFC | 5HT | 5HT1A | post-syn | −1 | [−2,0] | 1 | M | {0.20, 0.40, 0.20, 0.20} | Nash 2008 PET |
| panic.mPFC.GABA.tone.tone | mPFC | GABA | GABA tone (MRS) | tone | −2 | [−3,−1] | 1 | M | {0.20, 0.40, 0.10, 0.30} | Long 2013; Hasler 2008 |
| panic.mPFC.functional.anticipatory.functional | mPFC | Composite | Anticipatory-anxiety BOLD | functional | +2 | [+1,+3] | 2 | M | {0.10, 0.60, 0.20, 0.10} | Boshuisen 2002 |
| panic.mPFC.functional.regulation.functional | mPFC | Composite | Top-down regulation of amygdala | functional | −2 | [−3,−1] | 1 | M | {0.20, 0.30, 0.20, 0.30} | Pannekoek 2013 |
| panic.ACC.5HT.5HT1A.post-syn | ACC | 5HT | 5HT1A | post-syn | −1 | [−2,0] | 2 | M | {0.20, 0.30, 0.30, 0.20} | Nash 2008 |
| panic.ACC.GABA.tone.tone | ACC | GABA | GABA tone (MRS) | tone | −2 | [−3,−1] | 1 | M | {0.20, 0.30, 0.20, 0.30} | Long 2013 |
| panic.ACC.functional.interoceptive-salience.functional | ACC | Composite | Interoceptive-salience BOLD | functional | +2 | [+1,+3] | 1 | M | {0.20, 0.20, 0.50, 0.10} | Critchley 2004 |
| panic.ACC.functional.error-monitoring.functional | ACC | Composite | Error/conflict monitoring | functional | +1 | [0,+2] | 2 | L | {0.20, 0.30, 0.30, 0.20} | inferred from anxiety meta-analyses |
| panic.ACC.composite.glu-gaba.composite | ACC | Composite | Glu:GABA ratio | composite | +2 | [+1,+3] | 2 | M | {0.20, 0.30, 0.20, 0.30} | Long 2013 |
| panic.vmPFC.5HT.5HT1A.post-syn | vmPFC | 5HT | 5HT1A | post-syn | −1 | [−2,0] | 2 | M | {0.10, 0.20, 0.20, 0.50} | Nash 2008 |
| panic.vmPFC.functional.extinction.functional | vmPFC | Composite | Fear-extinction recall (BOLD) | functional | −2 | [−3,−1] | 1 | M | {0.10, 0.10, 0.20, 0.60} | Milad 2014 |
| panic.vmPFC.Glu.tone.tone | vmPFC | Glu | Glu tone (MRS) | tone | +1 | [0,+2] | 3 | L | {0.20, 0.20, 0.20, 0.40} | inferred |
| panic.dlPFC.GABA.tone.tone | dlPFC | GABA | GABA tone | tone | −1 | [−2,0] | 2 | L | {0.10, 0.40, 0.10, 0.40} | Hasler 2008 |
| panic.dlPFC.functional.reappraisal.functional | dlPFC | Composite | Reappraisal engagement | functional | −1 | [−2,0] | 2 | L | {0.10, 0.40, 0.10, 0.40} | inferred |
| panic.OFC.functional.interoceptive-valuation.functional | OFC | Composite | Interoceptive-valuation engagement | functional | +1 | [0,+2] | 3 | L | {0.20, 0.20, 0.50, 0.10} | inferred |
| panic.hippocampus.5HT.5HT1A.post-syn | Hippocampus | 5HT | 5HT1A | post-syn | −1 | [−2,0] | 1 | M | {0.20, 0.20, 0.20, 0.40} | Nash 2008; Neumeister 2004 |
| panic.hippocampus.GABA.tone.tone | Hippocampus | GABA | GABA tone | tone | −1 | [−2,0] | 2 | L | {0.20, 0.20, 0.20, 0.40} | inferred |
| panic.hippocampus.functional.context-discrimination.functional | Hippocampus | Composite | Context vs threat discrimination | functional | −2 | [−3,−1] | 1 | M | {0.10, 0.20, 0.10, 0.60} | Lissek 2010 |
| panic.hippocampus.composite.volume.composite | Hippocampus | Composite | Hippocampal volume | composite | −1 | [−2,0] | 2 | L | {0.20, 0.30, 0.20, 0.30} | Massana 2003 (contested) |
| panic.vS.DA.D2.post-syn | vS | DA | D2/D3 | post-syn | 0 | [−1,+1] | 3 | L | {0.20, 0.40, 0.20, 0.20} | inferred; minimal in pure panic |
| panic.vS.functional.anticipatory.functional | vS | Composite | Anticipatory-anxiety BOLD | functional | +1 | [0,+2] | 2 | L | {0.20, 0.50, 0.20, 0.10} | Nitschke 2009 |
| panic.raphe.5HT.5HT1A.auto | Raphe | 5HT | 5HT1A | auto | +2 | [+1,+3] | 1 | M | {0.40, 0.20, 0.20, 0.20} | Neumeister 2004; Akimova 2009 |
| panic.raphe.5HT.SERT.pre-syn | Raphe | 5HT | SERT | pre-syn | 0 | [−1,+1] | 2 | L | {0.30, 0.20, 0.30, 0.20} | Maron 2011 (contested: methodological) |
| panic.raphe.5HT.tone.tone | Raphe | 5HT | 5HT tone to PAG/amygdala | tone | −1 | [−2,0] | 2 | M | {0.50, 0.10, 0.20, 0.20} | Graeff 2012; Maron 2011 |
| panic.raphe.5HT.CO2-chemoreceptor.dynamic | Raphe | 5HT | CO2-sensing 5HT neurons (DRN) | dynamic | +2 | [+1,+3] | 2 | M | {0.70, 0.05, 0.20, 0.05} | Severson 2003; Hodges & Richerson 2010 |
| panic.raphe.functional.PAG-inhibition.functional | Raphe | Composite | DRN→PAG inhibitory output (proxy) | functional | −2 | [−3,−1] | 2 | M | {0.60, 0.10, 0.20, 0.10} | Graeff 2012; Schenberg 2014 |
| panic.LC.NE.alpha2.auto | LC | NE | α2A-adrenergic | auto | −1 | [−2,0] | 2 | M | {0.30, 0.30, 0.20, 0.20} | Goddard 2017 |
| panic.LC.NE.tone.tone | LC | NE | NE tone (projection) | tone | +2 | [+1,+3] | 1 | M | {0.40, 0.20, 0.20, 0.20} | Charney 1990; Goddard 2017 |
| panic.LC.NE.dynamic.dynamic | LC | NE | LC tonic + phasic firing | dynamic | +2 | [+1,+3] | 1 | M | {0.50, 0.20, 0.20, 0.10} | Bremner 1996 |
| panic.LC.NE.NET.pre-syn | LC | NE | NET | pre-syn | 0 | [−1,+1] | 3 | L | {0.30, 0.30, 0.20, 0.20} | inferred |
| panic.VTA.DA.dynamic.dynamic | VTA | DA | VTA firing dynamics | dynamic | 0 | [−1,+1] | 3 | L | {0.20, 0.30, 0.20, 0.30} | inferred; minimal in pure panic |
| panic.cortex.GABA.GABA-A/BZ.density | Cortex (composite) | GABA | GABA-A/BZ-receptor density (PET [11C]flumazenil) | density | −2 | [−3,−1] | 1 | H | {0.30, 0.30, 0.10, 0.30} | Malizia 1998; Hasler 2008; Cameron 2007 |
| panic.cortex.GABA.tone.tone | Cortex (composite) | GABA | Cortical GABA tone (MRS aggregate) | tone | −2 | [−3,−1] | 1 | M | {0.25, 0.30, 0.15, 0.30} | Long 2013; Hasler 2008 |
| panic.cortex.5HT.5HT1A.density | Cortex (composite) | 5HT | 5HT1A density (cortical aggregate) | density | −2 | [−3,−1] | 1 | H | {0.30, 0.25, 0.25, 0.20} | Neumeister 2004; Nash 2008 |
| panic.cortex.composite.insular-interoception.composite | Cortex (composite) | Composite | Insular interoceptive BOLD (proxy region) | composite | +2 | [+1,+3] | 1 | H | {0.20, 0.10, 0.60, 0.10} | Critchley 2004; Domschke 2010 |
| panic.cortex.composite.insular-volume.composite | Cortex (composite) | Composite | Insular gray-matter volume (proxy region) | composite | +1 | [0,+2] | 2 | M | {0.20, 0.20, 0.50, 0.10} | Uchida 2008 |
| panic.cortex.Glu.tone.tone | Cortex (composite) | Glu | Cortical Glu tone (aggregate) | tone | +1 | [0,+2] | 2 | L | {0.20, 0.30, 0.20, 0.30} | Maddock 2013 |
| panic.cortex.composite.lactate-sensitivity.composite | Cortex (composite) | Composite | Lactate-infusion BOLD sensitivity (provocation paradigm) | composite | +2 | [+1,+3] | 1 | M | {0.50, 0.10, 0.30, 0.10} | Maddock 2013; Goddard 2017 |
| panic.cortex.composite.co2-sensitivity.composite | Cortex (composite) | Composite | CO2-challenge response (provocation paradigm) | composite | +3 | [+2,+3] | 1 | H | {0.70, 0.05, 0.20, 0.05} | Klein 1993 (suffocation alarm); Battaglia 2019 |

Subsystem weights cross-check: each row sums to 1.0 ± 0.01.

## Key authoring choices

### 1. PAG and insula encoded as proxy/composite cells with vocabulary-gap notes

The v1 region vocabulary (per the task brief) is OFC, ACC, dlPFC, vmPFC, mPFC, Caudate, Putamen, vS, Amygdala, Hippocampus, Raphe, VTA, LC. **PAG and insula are not in the vocabulary** but are central to panic neurobiology. Resolution:

- PAG findings are encoded under Raphe with `target` strings carrying the projection specificity (`5HT tone to PAG/amygdala`, `DRN→PAG inhibitory output`) and notes flagging "PAG specificity; pending v2 vocabulary extension." This preserves the Raphe→PAG inhibition finding without inventing a region.
- Insular findings are encoded as `Cortex (composite)` rows with `composite.insular-*` targets and notes flagging the same gap. The interoceptive subsystem (`I`) carries most of its weight in these cells plus the ACC interoceptive-salience functional cell.

Action item: propose `Insula` and `PAG` for v2 region vocabulary in `15-schema-extensions.md`. Until then, the proxy encoding is auditable but lossy.

### 2. The CO2 / suffocation alarm cell is the panic-specific signature

`panic.raphe.5HT.CO2-chemoreceptor.dynamic` and `panic.cortex.composite.co2-sensitivity.composite` together encode the Klein 1993 / Battaglia / Hodges-Richerson framework. These cells are weighted heavily to the `P` (panic-attack core) subsystem. Their delta_best is `+2` and `+3` respectively, reflecting the well-replicated CO2-challenge hypersensitivity. This is what distinguishes panic from GAD at the cell level.

### 3. Raphe→PAG inhibition is encoded as a directional functional cell

`panic.raphe.functional.PAG-inhibition.functional` carries `delta_best: −2` (reduced inhibition). This is the Graeff "deficient serotonergic inhibition of PAG" framework — central to panic — and could not be expressed as a receptor density alone. The functional site type is doing the load-bearing work here.

### 4. SERT in panic is `contested: methodological`

Identical reasoning to GAD's SERT cell: methodological heterogeneity. `panic.raphe.5HT.SERT.pre-syn` has `delta_best: 0`, `delta_range: [−1, +1]`, `contested: methodological`, notes explaining the conflict. Visualization marks it regardless of magnitude.

### 5. Provocation-paradigm cells (lactate, CO2) are first-class

Panic literature has a rich provocation paradigm tradition (sodium lactate, CO2, yohimbine, doxapram) which produces reproducible activation patterns. Two composite cells encode the lactate and CO2 provocations explicitly. This is unusual relative to other disorders — most disorder templates don't have provocation cells — but panic's provocation literature is strong enough to warrant the encoding.

### 6. Agoraphobia is a phenotype modifier, not a separate template

DSM-5 splits Panic Disorder (300.01) from Agoraphobia (300.22) but the neurobiology is largely shared. v1 handles agoraphobia as a PatientSubsystemModifier on the `C` (conditioned avoidance) subsystem rather than a separate DisorderTemplate. A v2 curated comorbidity template (`panic_plus_agoraphobia_v1`) is plausible if pilot data justifies it.

### 7. Brain Type 1 anchor (shared with GAD)

Both panic and GAD anchor to Type 1 (Anxious-Vigilant). The type chip preselects Type 1 when either template is referenced. The cell-level differentiation (panic's brainstem dominance and interoceptive cells vs GAD's worry/DMN dominance) is preserved at the cell registry level, not in the type — which is by design (type is the coarse handle; cells are the fine grain).

## ElicitationMap reference

```yaml
id: pdss-sr.v1
instrument: PDSS-SR (Panic Disorder Severity Scale, self-report)
applies_to_templates: [panic_canonical_v1]
recency_window_days: 14
license_status: free-clinical
source_citation: "Shear MK et al. (1997, 2001). Reliability and validity of \
                  a structured interview guide for the Hamilton Anxiety Rating \
                  Scale. Self-report PDSS validated by Houck PR et al. 2002 \
                  in Depression and Anxiety. https://pubmed.ncbi.nlm.nih.gov/12203671/"
```

### Scoring rules

PDSS-SR has 7 items, each 0–4, total 0–28.

- `total` = items[1..7].sum, range [0, 28]
- `attack_severity` = items[1] + items[2] (frequency, distress during attacks)
- `anticipatory` = items[3] (anticipatory anxiety)
- `phobic_avoidance` = items[4] + items[5] (situational avoidance, interoceptive avoidance)
- `impairment` = items[6] + items[7] (work, social impairment)

### Subsystem mappings

| Scoring | Subsystem | Formula | Evidence | Confidence | Rationale |
|---|---|---|---|---|---|
| attack_severity | P | (score − 3) / 3 | inferred | M | Attack frequency × distress indexes brainstem fear-circuit recruitment |
| anticipatory | A | (score − 1) / 1.5 | inferred | M | Single item; coarse but specific to A subsystem |
| phobic_avoidance | C | (score − 3) / 3 | inferred | M | Avoidance × interoceptive-avoidance indexes conditioned-avoidance circuit |
| phobic_avoidance | I | (score − 3) / 6 | inferred | L | Half-weight to interoceptive; item 5 (interoceptive avoidance) specifically |
| total | I | (score − 14) / 7 | inferred | L | Interoceptive vigilance scales with overall severity |

All coefficients are starting calibrations and require pilot validation.

### Cell-level mappings

None in v1. PDSS-SR does not have multi-study replication supporting cell-level overrides. All effects route through subsystem weights.

### AI extraction targets

- **Pattern: discrete panic attacks with autonomic surge.** Cells: `panic.LC.NE.dynamic.dynamic`, `panic.amygdala.functional.interoceptive-reactivity.functional`, `panic.cortex.composite.co2-sensitivity.composite`. Phrasings: "heart racing out of nowhere", "felt like I couldn't breathe", "thought I was having a heart attack".
- **Pattern: fear of recurrence / interoceptive vigilance.** Cells: `panic.cortex.composite.insular-interoception.composite`, `panic.ACC.functional.interoceptive-salience.functional`. Phrasings: "constantly checking my pulse", "every twinge feels like the start of one".
- **Pattern: situational avoidance (agoraphobic features).** Cells: `panic.vmPFC.functional.extinction.functional`, `panic.hippocampus.functional.context-discrimination.functional`. Phrasings: "won't go to the supermarket", "always need an escape route", "haven't driven on the highway in years".
- **Pattern: nocturnal panic / sleep-onset attacks.** Cells: `panic.LC.NE.dynamic.dynamic`, `panic.raphe.functional.PAG-inhibition.functional`. Phrasings: "wake up gasping", "panic attacks in my sleep".

## Narrative summary

Panic disorder's signature is a **brainstem fear circuit that fires inappropriately, plus a cortex that learns to fear the firing.** Four observations anchor the template.

First, the **panic attack itself is a brainstem event.** LC tonic and phasic noradrenergic output is elevated (Charney 1990; Goddard 2017). The dorsal-raphe serotonergic neurons that normally inhibit PAG defensive output are deficient (Graeff 2012). The CO2-chemosensing 5-HT neurons in the raphe are hypersensitive (Hodges & Richerson 2010), supporting Klein's 1993 suffocation-alarm hypothesis: panic patients respond to subtle hypercapnia or hypoxia as if it were imminent suffocation.

Second, the **amygdala is the central recipient and amplifier.** Reduced 5-HT1A binding (Neumeister 2004 PET) and reduced GABA-A/BZ binding (Malizia 1998 [11C]flumazenil PET, Hasler 2008) leave the amygdala under-inhibited. Interoceptive cues recruit amygdala BOLD above controls (Domschke 2010). Amygdala-mPFC connectivity is reduced (Pannekoek 2013).

Third, the **insula and ACC encode the interoceptive hypervigilance** that becomes fear-of-fear. Insular gray-matter volume is elevated (Uchida 2008), interoceptive BOLD is elevated (Critchley 2004). The patient becomes acutely sensitive to their own bodily signals, and any deviation registers as potential threat. In v1 vocabulary these findings live as composite proxies pending insula/PAG addition to the region enum.

Fourth, the **cortical inhibitory deficit parallels GAD's.** Reduced cortical GABA on MRS (Long 2013; Hasler 2008) and reduced cortical BZ-receptor binding (Malizia 1998; Cameron 2007) are shared. This is why benzodiazepines work acutely in both disorders, and why both share Brain Type 1 (Anxious-Vigilant) at the type layer.

What distinguishes panic from GAD at the cell level: the brainstem dominance (LC, raphe, PAG-proxy), the CO2/suffocation-alarm cells, the insular interoceptive cells, and the conditioned-avoidance hippocampal/vmPFC cells. GAD's signature is worry/DMN; panic's signature is interoceptive paroxysm.

The treatment implication: first-line SSRIs (escitalopram, sertraline) and SNRIs (venlafaxine) hit the serotonergic limbic and raphe cells; the 4-6 week onset matches the time required to restore 5-HT1A function and downstream amygdala inhibition. Benzodiazepines (clonazepam, alprazolam) cover the cortex-wide BZ-binding cells acutely — useful in titration period and PRN — but tolerance is real and dependence risk high; the residual coverage panel should flag long-term use. CBT (interoceptive exposure plus cognitive restructuring) covers the conditioned-avoidance subsystem (`C`) and the interoceptive subsystem (`I`) through experiential rather than pharmacological mechanisms. Propranolol covers peripheral β-adrenergic symptoms but not the brainstem source.

## v1 readiness

**Ready:**

- 48 active cells covering the panic-attack core, anticipatory anxiety, interoceptive hypervigilance, and conditioned-avoidance subsystems with explicit subsystem weights.
- PDSS-SR ElicitationMap drafted with subsystem-default mappings; coefficients calibrated to literature direction.
- Brain Type 1 (Anxious-Vigilant) anchor populated.
- CO2 / lactate provocation cells encoded as composite cells with the strongest panic-specific evidence.

**Not ready (blockers tracked in `11-readiness-and-blockers.md`):**

- **Drug coverage cells not yet populated.** First priority: escitalopram, sertraline, paroxetine, venlafaxine, clonazepam, alprazolam, propranolol, hydroxyzine. Benzodiazepines need tolerance-flagged cells and explicit short-term-use status.
- **Region vocabulary gap.** Insula and PAG are central to panic but not in the v1 region vocabulary. Proxy encoding via Raphe and Cortex composites is auditable but loses anatomic specificity. Tracked as a vocabulary-extension proposal for v2.
- **Agoraphobia subtype encoding.** Currently handled as PatientSubsystemModifier; pilot data may justify a curated `panic_plus_agoraphobia_v1` template.
- **PDSS-SR ElicitationMap calibration.** Formula coefficients are first-pass.
- **State-trait conflict typology.** 5-HT1A findings vary by acute vs. inter-attack state; needs explicit `contested: state-trait` tagging.
- **Provocation-paradigm cells** are unusual relative to other templates. May need a `measure_type: provocation` extension to the enum if other disorder templates adopt the pattern.
- **Composition rules with GAD, MDD, agoraphobia templates.** Panic + GAD is common comorbidity. Pure panic is the v1 deliverable.

## References (anchoring sources, partial; full source list lives in registry)

- Klein DF (1993). False suffocation alarms, spontaneous panics, and related conditions. *Arch Gen Psychiatry.* — suffocation alarm hypothesis.
- Battaglia M, Khan WU (2019). Panic disorder, suffocation-fear, and the neurobiology of anxiety. *Curr Top Behav Neurosci.* — updated suffocation-alarm framework.
- Severson CA, Wang W, Pieribone VA et al. (2003). Mid-hindbrain serotonergic neurons are central respiratory chemoreceptors. *Nat Neurosci.* — DRN CO2 chemoreceptor cells.
- Hodges MR, Richerson GB (2010). Medullary serotonin neurons and their roles in central respiratory chemoreception. *Respir Physiol Neurobiol.* — extended chemoreception framework.
- Maddock RJ et al. (2013). Elevated brain lactate responses to neural activation in panic disorder. *Mol Psychiatry.* — lactate provocation; cortical lactate.
- Maron E, Nutt D (2011). Biological markers of anxiety disorders. *Psychiatry.* https://pubmed.ncbi.nlm.nih.gov/22055194/ — SERT methodology heterogeneity.
- Malizia AL et al. (1998). Decreased brain GABA-A benzodiazepine receptor binding in panic disorder. *Arch Gen Psychiatry.* https://pubmed.ncbi.nlm.nih.gov/9783559/ — [11C]flumazenil PET landmark.
- Cameron OG et al. (2007). Reduced GABA-A benzodiazepine binding sites in insular cortex of individuals with panic disorder. *Psychiatry Res.*
- Neumeister A et al. (2004). Reduced serotonin type 1A receptor binding in panic disorder. *J Neurosci.* https://pubmed.ncbi.nlm.nih.gov/14724240/
- Nash JR, Sargent PA et al. (2008). Serotonin 5-HT1A receptor binding in people with panic disorder. *Br J Psychiatry.*
- Akimova E et al. (2009). The serotonin-1A receptor in anxiety disorders. *Biol Psychiatry.* https://pubmed.ncbi.nlm.nih.gov/19625006/
- Goddard AW (2017). The neurobiology of panic: a chronic stress disorder. *Chronic Stress.* https://journals.sagepub.com/doi/full/10.1177/2470547017736038
- Charney DS, Heninger GR (1990). Noradrenergic function in panic disorder. *J Clin Psychiatry.*
- Bremner JD et al. (1996). Noradrenergic mechanisms in stress and anxiety. *Synapse.*
- Graeff FG, Zangrossi H (2012). The dual role of serotonin in defense and the mode of action of antidepressants on generalized anxiety and panic disorders. *Curr Top Behav Neurosci.* — DRN→PAG framework.
- Schenberg LC (2014). Panic disorder: from the period mechanisms to the panic gene. *Trends Pharmacol Sci.*
- Pannekoek JN et al. (2013). Aberrant resting-state functional connectivity in limbic and salience networks in treatment-naïve clinically depressed adolescents. *J Child Psychol Psychiatry.* — amygdala-mPFC connectivity.
- Boshuisen ML et al. (2002). rCBF differences between panic disorder patients and control subjects during anticipatory anxiety and rest. *Biol Psychiatry.*
- Critchley HD et al. (2004). Neural systems supporting interoceptive awareness. *Nat Neurosci.* https://www.nature.com/articles/nn1176
- Domschke K et al. (2010). Interoceptive sensitivity in anxiety and anxiety disorders. *Biol Psychol.*
- Uchida RR et al. (2008). Regional gray matter abnormalities in panic disorder: a voxel-based morphometry study. *Psychiatry Res Neuroimaging.*
- Milad MR et al. (2014). Deficits in conditioned fear extinction in obsessive-compulsive disorder and neurobiological changes in the fear circuit. *JAMA Psychiatry.* — extinction circuit (cited across anxiety templates).
- Lissek S et al. (2010). Generalization of conditioned fear-potentiated startle in humans: experimental validation and clinical relevance. *Behav Res Ther.* — context discrimination.
- Massana G et al. (2003). Amygdalar atrophy in panic disorder patients detected by volumetric magnetic resonance imaging. *NeuroImage.*
- Long Z et al. (2013). Decreased GABA levels in anterior cingulate cortex/medial prefrontal cortex in panic disorder. *Prog Neuropsychopharmacol Biol Psychiatry.* https://pubmed.ncbi.nlm.nih.gov/23643673/
- Hasler G et al. (2008). Prefrontal cortical GABA concentrations in patients with major depressive disorder and generalized anxiety disorder by proton MRS. *Biol Psychiatry.*
- Houck PR et al. (2002). Reliability of the self-report version of the Panic Disorder Severity Scale. *Depress Anxiety.* https://pubmed.ncbi.nlm.nih.gov/12203671/
- NIH-PMC overview: *Fear Circuits in Panic Disorder: An Update.* https://pmc.ncbi.nlm.nih.gov/articles/PMC12231371/
- NIH-PMC overview: *Brain Mechanisms Underlying Panic Attack and Panic Disorder.* https://pmc.ncbi.nlm.nih.gov/articles/PMC11178723/

---

Last updated: 2026-05-13 · Draft awaiting clinical review.
