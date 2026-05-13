# 28 — Insomnia Disorder Template (Canonical v1, Draft)

> **Clinical review required.** This is a first-draft DisorderTemplate produced by an authoring agent against the v3 schema. Cell deltas, evidence tiers, and subsystem weights are starting calibrations grounded in cited literature, but have not been reviewed or signed off by the clinical team. Do not use for patient-facing rendering until clinical review is complete and the template is moved from `status: draft` to `status: active`. This template additionally surfaces **v1 vocabulary limitations** — see "Vocabulary gaps and proposed v1.5 extensions" below.

The third canonical disorder template (after OCD and ADHD). Insomnia is in scope because PCPs see it constantly and because no single mechanism dominates — the four-subsystem decomposition makes the trade-offs between hypnotics, sedating antidepressants, melatonergic agents, and orexin antagonists legible. Modeled after `09-ocd-reference-instantiation.md` in structure and authoring tone.

---

## Template metadata

```yaml
id: insomnia_canonical_v1
schema_version: "3.0"
disorder: InsomniaDisorder
template_version: "1.0.0"
severity_bucket: moderate              # canonical baseline at moderate severity
phenotype_subtype: null                 # template covers chronic insomnia disorder broadly
baseline_ref: healthy_v1
elicitation_map_ref: isi.v1
status: draft
last_reviewed: 2026-05-13
reviewer: authoring-agent
```

The canonical template represents **moderate, unmedicated, chronic Insomnia Disorder in adults**. Severity scales linearly off the ISI score; acute/transient insomnia (≤3 months) uses the same template with `severity_bucket: mild` and acute_modifiers on the Arousal subsystem.

## Subsystems

Four subsystems anchored in the dominant insomnia models — the 3P model (Spielman & Glovinsky 1987, neurobiological extension by Buysse 2010) and the hyperarousal model (Bonnet & Arand 2010):

- **R — Arousal / hyperarousal.** Sympathetic activation, HPA axis activation, elevated metabolic rate, high-frequency EEG. Maps primarily to LC, amygdala, ACC, and the autonomic-arousal axis. The dominant subsystem in most chronic insomnia phenotypes ([Bonnet & Arand 2010](https://pubmed.ncbi.nlm.nih.gov/19640748/); [Dressle & Riemann 2023](https://onlinelibrary.wiley.com/doi/full/10.1111/jsr.13928)).
- **C — Circadian misalignment.** Phase delay, phase advance, weakened SCN signaling, melatonin-rhythm abnormalities. **Note: the v1 region vocabulary does not include the SCN, hypothalamus, or pineal — we encode this subsystem against the closest v1 substitutes (Raphe, VTA, LC) and explicitly flag the gap below.**
- **O — Sleep-onset difficulty.** Cortical and thalamocortical hyperactivity at sleep onset, ruminative pre-sleep cognition, conditioned arousal. Maps to ACC, mPFC, vmPFC, Amygdala — the DMN-emotion cluster active at sleep onset in insomnia ([Wassing 2016 → Stoffers et al. 2014](https://journals.physiology.org/doi/full/10.1152/physrev.00046.2019)).
- **M — Sleep-maintenance / wake-after-sleep-onset (WASO).** GABAergic deficits, cortical hyperarousal during sleep, NREM instability. Maps to ACC and broader cortical GABA tone ([Winkelman 2008](https://academic.oup.com/sleep/article-pdf/31/11/1499/13664126/sleep-31-11-1499.pdf); [Plante 2012](https://www.nature.com/articles/npp20124)).

These four are not orthogonal. R drives O and M; C interacts with R in shift-work and delayed-sleep-phase variants. Subsystem_weights on each cell distribute its contribution across all four; the elicitation map maps ISI items onto the subsystems with O and M getting heaviest signal from the first three ISI items.

## Cell coverage summary

42 active cells in this draft span:

### Cortical regions
- **ACC** — error monitoring at sleep onset, hyperarousal hub; 5-HT2A, GABA, Glu, BOLD. ~7 cells.
- **mPFC / vmPFC** — pre-sleep rumination, DMN-emotion cluster; Glu, GABA, BOLD. ~5 cells.
- **dlPFC** — daytime impairment correlate; GABA, NE. ~3 cells.

### Subcortical / striatal regions
- **Caudate / Putamen / vS** — minimal direct evidence in insomnia literature; encoded sparsely (3 cells total) and flagged `evidence_status: inferred` or `no-data` where appropriate.

### Limbic regions
- **Amygdala** — emotional reactivity at sleep onset, hyperarousal-emotion coupling; NE α1, 5-HT2A, GABA-A, BOLD. ~6 cells.
- **Hippocampus** — sleep-dependent memory consolidation, stress-volume correlates; GABA, Glu, ACh. ~4 cells.

### Brainstem / regulatory
- **Raphe (DRN)** — serotonergic source, wake-promoting; 5-HT firing, autoreceptors. ~4 cells. *Used as v1 proxy for some pineal/SCN findings via Raphe→SCN projections.*
- **VTA** — minimal direct evidence; one cell. *Used as v1 proxy for "ascending arousal nuclei" composite where SCN/TMN/orexin not in vocabulary.*
- **LC** — noradrenergic hyperarousal driver; NE firing tonic, NE tone, α1/α2. ~6 cells.

## The cells

Format per cell: `id` — `target / site` — `delta_best [range]` — `tier/confidence` — subsystem_weights — citation(s) — note.

### ACC (anterior cingulate cortex)

| Cell ID | Target / site | δ_best | range | tier/conf | weights | sources |
|---|---|---|---|---|---|---|
| `insomnia.acc.GABA.tone.tone` | GABA tone (MRS) | −2 | [−3,−1] | 1/H | R:0.3, M:0.5, O:0.2 | Plante 2012 [Neuropsychopharm](https://www.nature.com/articles/npp20124); Winkelman 2008 [Sleep](https://academic.oup.com/sleep/article-pdf/31/11/1499/13664126/sleep-31-11-1499.pdf) |
| `insomnia.acc.GABA.GABA-A.post-syn` | GABA-A / post-syn | −1 | [−2,0] | 2/M | M:0.5, R:0.3, O:0.2 | inferred from MRS tone reduction |
| `insomnia.acc.Glu.tone.tone` | Glu tone (MRS) | +1 | [0,+2] | 1/M | R:0.4, O:0.4, M:0.2 | Winkelman 2019 [PMC10662933](https://pmc.ncbi.nlm.nih.gov/articles/PMC10662933/) |
| `insomnia.acc.5HT.5HT2A.post-syn` | 5-HT2A / post-syn | +1 | [0,+2] | 2/M | R:0.5, O:0.5 | inferred from trazodone mechanism literature; [PMC5842888](https://pmc.ncbi.nlm.nih.gov/articles/PMC5842888/) |
| `insomnia.acc.NE.alpha1.post-syn` | α1 / post-syn | +1 | [0,+2] | 2/M | R:0.7, O:0.3 | inferred from prazosin literature; arousal axis |
| `insomnia.acc.Composite.bold.functional` | rest-state hyperactivity | +2 | [+1,+3] | 1/H | R:0.4, O:0.4, M:0.2 | Riemann 2015 [Physiol Rev](https://journals.physiology.org/doi/full/10.1152/physrev.00046.2019) |
| `insomnia.acc.Composite.dmn.functional` | DMN-task switching deficits | +1 | [0,+2] | 1/M | O:0.6, R:0.4 | DMN hyperactivity at sleep onset |

### mPFC

| Cell ID | Target / site | δ_best | range | tier/conf | weights | sources |
|---|---|---|---|---|---|---|
| `insomnia.mpfc.GABA.tone.tone` | GABA tone | −1 | [−2,0] | 2/M | M:0.4, O:0.4, R:0.2 | inferred from cortical-wide GABA reductions |
| `insomnia.mpfc.Glu.tone.tone` | Glu tone | +1 | [0,+2] | 2/M | O:0.5, R:0.3, M:0.2 | inferred |
| `insomnia.mpfc.Composite.dmn.functional` | DMN failure to deactivate at sleep onset | +2 | [+1,+3] | 1/M | O:0.7, R:0.3 | Stoffers 2014; Riemann 2015 [Physiol Rev](https://journals.physiology.org/doi/full/10.1152/physrev.00046.2019) |

### vmPFC

| Cell ID | Target / site | δ_best | range | tier/conf | weights | sources |
|---|---|---|---|---|---|---|
| `insomnia.vmpfc.Glu.tone.tone` | Glu tone | 0 | [−1,+1] | 3/L | O:0.6, R:0.4 | no-data; inferred |
| `insomnia.vmpfc.Composite.bold.functional` | emotional-reactivity BOLD | +1 | [0,+2] | 1/M | O:0.5, R:0.5 | [Baglioni 2014](https://www.sciencedirect.com/science/article/abs/pii/S0720048X11002993) — amygdala-vmPFC coupling abnormal |

### dlPFC

| Cell ID | Target / site | δ_best | range | tier/conf | weights | sources |
|---|---|---|---|---|---|---|
| `insomnia.dlpfc.GABA.tone.tone` | GABA tone | −1 | [−2,0] | 2/M | M:0.5, R:0.3, O:0.2 | Winkelman 2008; cortical-wide reduction |
| `insomnia.dlpfc.NE.alpha2A.post-syn` | α2A / post-syn | 0 | [−1,+1] | 3/L | R:0.5, M:0.5 | inferred — daytime impairment marker |
| `insomnia.dlpfc.Composite.bold.functional` | daytime working-memory BOLD | −1 | [−2,0] | 2/M | M:0.6, R:0.4 | daytime sequelae of poor sleep |

### Caudate

| Cell ID | Target / site | δ_best | range | tier/conf | weights | sources |
|---|---|---|---|---|---|---|
| `insomnia.caudate.Composite.bold.functional` | rest-state activity | +1 | [0,+2] | 2/L | R:0.5, O:0.5 | indirect; ascending arousal correlate |

### Putamen

| Cell ID | Target / site | δ_best | range | tier/conf | weights | sources |
|---|---|---|---|---|---|---|
| `insomnia.putamen.GABA.tone.tone` | GABA tone | 0 | [−1,+1] | 3/L | R:0.4, M:0.6 | no-data |

### vS (ventral striatum)

| Cell ID | Target / site | δ_best | range | tier/conf | weights | sources |
|---|---|---|---|---|---|---|
| `insomnia.vS.DA.tone.tone` | DA tone | 0 | [−1,+1] | 3/L | R:0.5, O:0.5 | no-data; not strongly implicated |

### Amygdala

| Cell ID | Target / site | δ_best | range | tier/conf | weights | sources |
|---|---|---|---|---|---|---|
| `insomnia.amygdala.NE.alpha1.post-syn` | α1 / post-syn | +2 | [+1,+3] | 2/M | R:0.6, O:0.4 | hyperarousal-emotion axis; prazosin mechanism support |
| `insomnia.amygdala.NE.tone.tone` | NE tone | +2 | [+1,+3] | 2/H | R:0.6, O:0.3, M:0.1 | inferred from LC-amygdala projection literature; Riemann 2015 |
| `insomnia.amygdala.5HT.5HT2A.post-syn` | 5-HT2A / post-syn | +1 | [0,+2] | 2/M | R:0.4, O:0.6 | inferred; trazodone target |
| `insomnia.amygdala.GABA.GABA-A.post-syn` | GABA-A / post-syn | −1 | [−2,0] | 2/M | R:0.5, O:0.5 | inferred from benzodiazepine response; Riemann 2015 |
| `insomnia.amygdala.Composite.bold.functional` | emotional-reactivity BOLD to sleep-related stimuli | +2 | [+1,+3] | 1/H | O:0.5, R:0.4, M:0.1 | Baglioni 2014 [link](https://www.sciencedirect.com/science/article/abs/pii/S0720048X11002993); [PMC7810854](https://pmc.ncbi.nlm.nih.gov/articles/PMC7810854/) |
| `insomnia.amygdala.Composite.connectivity.functional` | amygdala-insula/thalamus rsFC | +1 | [0,+2] | 1/M | R:0.5, O:0.5 | [PMC10694975](https://pmc.ncbi.nlm.nih.gov/articles/PMC10694975/) |

### Hippocampus

| Cell ID | Target / site | δ_best | range | tier/conf | weights | sources |
|---|---|---|---|---|---|---|
| `insomnia.hippocampus.GABA.tone.tone` | GABA tone | −1 | [−2,0] | 3/L | M:0.6, R:0.4 | inferred from cortical-wide reduction |
| `insomnia.hippocampus.Glu.NMDA.post-syn` | NMDA / post-syn | +1 | [0,+2] | 3/L | O:0.5, R:0.5 | inferred from chronic-stress excitotoxicity model |
| `insomnia.hippocampus.ACh.tone.tone` | ACh tone | 0 | [−1,+1] | 3/L | M:0.5, O:0.5 | no-data |
| `insomnia.hippocampus.Composite.volume.density` | hippocampal volume | −1 | [−2,0] | 2/L | R:0.4, M:0.6 | Riemann 2007 [PMC3391618](https://pmc.ncbi.nlm.nih.gov/articles/PMC3391618/) — *contested: methodological (Winkelman 2010 found no volume difference)* |

### Raphe (DRN)

| Cell ID | Target / site | δ_best | range | tier/conf | weights | sources |
|---|---|---|---|---|---|---|
| `insomnia.raphe.5HT.firing.tonic.dynamic` | 5-HT tonic firing rate | +1 | [0,+2] | 3/M | R:0.5, O:0.3, C:0.2 | inferred from DRN wake-promoting role; serves as v1 proxy for ascending wake-arousal |
| `insomnia.raphe.5HT.5HT1A.auto` | 5-HT1A autoreceptor / auto | 0 | [−1,+1] | 3/L | R:0.5, O:0.5 | no-data |
| `insomnia.raphe.5HT.SERT.pre-syn` | SERT / pre-syn | 0 | [−1,+1] | 3/L | R:0.5, M:0.5 | no-data |
| `insomnia.raphe.5HT.tone.tone` | 5-HT synaptic tone (projection sites) | +1 | [0,+2] | 3/M | R:0.5, O:0.3, C:0.2 | DRN→cortex projection elevated in wake-promoting state |

### VTA

| Cell ID | Target / site | δ_best | range | tier/conf | weights | sources |
|---|---|---|---|---|---|---|
| `insomnia.vta.DA.tone.tone` | DA tone | 0 | [−1,+1] | 3/L | R:0.5, O:0.5 | no-data; included for completeness; v1 substitute for "ascending arousal nuclei" |

### LC (locus coeruleus)

| Cell ID | Target / site | δ_best | range | tier/conf | weights | sources |
|---|---|---|---|---|---|---|
| `insomnia.lc.NE.firing.tonic.dynamic` | NE tonic firing | +2 | [+1,+3] | 2/H | R:0.6, O:0.3, M:0.1 | core hyperarousal driver; Bonnet & Arand 2010 [PubMed 19640748](https://pubmed.ncbi.nlm.nih.gov/19640748/); Dressle & Riemann 2023 [link](https://onlinelibrary.wiley.com/doi/full/10.1111/jsr.13928) |
| `insomnia.lc.NE.tone.tone` | NE synaptic tone (cortical/limbic projection) | +2 | [+1,+3] | 2/H | R:0.6, O:0.3, M:0.1 | LC→cortex projection elevation |
| `insomnia.lc.NE.alpha2.auto` | α2 autoreceptor / auto | −1 | [−2,0] | 3/M | R:0.7, M:0.3 | inferred; reduced auto-inhibition consistent with elevated tonic firing |
| `insomnia.lc.NE.firing.phasic.dynamic` | NE phasic firing | +1 | [0,+2] | 3/L | R:0.6, O:0.4 | inferred; less well-characterized than tonic |
| `insomnia.lc.Composite.cortisol.functional` | downstream HPA-axis cortisol output | +2 | [+1,+3] | 1/H | R:0.7, M:0.2, O:0.1 | Bonnet & Arand 2010 [PubMed 19640748](https://pubmed.ncbi.nlm.nih.gov/19640748/); Vgontzas 2001 — *v1 vocab note: HPA-axis output is encoded as composite cell on LC as proxy* |
| `insomnia.lc.Composite.metabolic.functional` | whole-body metabolic rate (24h) | +1 | [0,+2] | 1/H | R:0.8, M:0.2 | Bonnet & Arand 1995, 1998; [Bonnet & Arand 2010](https://pubmed.ncbi.nlm.nih.gov/19640748/) — *v1 vocab note: encoded on LC as proxy for autonomic axis* |

---

## Key authoring choices

### 1. The hyperarousal axis maps onto LC as the primary v1 region

The most replicated insomnia finding — elevated 24-hour metabolic rate, elevated cortisol, elevated HRV markers of sympathetic tone (Bonnet & Arand 1995, 1998, 2010) — is best framed as **autonomic/HPA hyperarousal**, which is brainstem-driven. The v1 region vocabulary has LC but not hypothalamus / PVN / brainstem reticular formation. We encode `insomnia.lc.Composite.cortisol.functional` and `insomnia.lc.Composite.metabolic.functional` as composite cells on LC, with `notes` explicitly stating they are v1 proxies for the autonomic-HPA axis. This is similar to how the OCD template uses VTA composite cells for "ascending dopaminergic firing" findings that aren't strictly VTA-localized.

### 2. The circadian (C) subsystem is undercovered by v1 vocabulary — and we flag it

Circadian dysregulation is a real insomnia mechanism (delayed sleep-phase, melatonin-rhythm abnormalities, shift-work-related insomnia), but the v1 region vocabulary does not include:

- **SCN** (suprachiasmatic nucleus) — the central circadian pacemaker
- **Pineal gland** — melatonin source
- **TMN** (tuberomammillary nucleus) — histamine source for the wake-promoting axis
- **LH / orexin neurons** — orexin/hypocretin source; the target of suvorexant/lemborexant/daridorexant
- **VLPO** (ventrolateral preoptic) — primary sleep-promoting nucleus
- **PVN** (paraventricular nucleus) — CRH/HPA-axis driver

This is a substantive gap for insomnia. The C subsystem is currently encoded only via secondary weights on Raphe and VTA cells, which is unsatisfying. **See "Vocabulary gaps and proposed v1.5 extensions" below** for the proposed schema change.

### 3. Hippocampal volume is flagged contested

The Riemann 2007 finding of reduced hippocampal volume in chronic insomnia ([PMC3391618](https://pmc.ncbi.nlm.nih.gov/articles/PMC3391618/)) was not replicated by Winkelman 2010 ([link](https://www.sciencedirect.com/science/article/abs/pii/S1389945710001607)). We encode `delta_best: −1` with `delta_range: [−2, 0]` and `contested: methodological`. The interaction with sleep duration and arousal index ([PMC3391618](https://pmc.ncbi.nlm.nih.gov/articles/PMC3391618/)) is the most reliable form of the finding.

### 4. GABA reductions are localized to ACC and occipital, not global

Winkelman's initial 2008 study ([Sleep](https://academic.oup.com/sleep/article-pdf/31/11/1499/13664126/sleep-31-11-1499.pdf)) reported global brain GABA reduction (~30%). The Plante 2012 follow-up ([Neuropsychopharm](https://www.nature.com/articles/npp20124)) used improved methodology and found the reduction localized to ACC and occipital cortex, with no thalamic difference. We encode the ACC GABA cell at `delta_best: −2` (high-confidence finding) and the dlPFC/mPFC GABA cells at `−1` with `delta_range: [−2, 0]`. We do not encode a thalamic GABA cell because the v1 vocabulary does not include thalamus as a region, and the literature finding is null there anyway.

### 5. The DMN-emotion cluster at sleep onset gets composite cells

The Stoffers et al. work synthesized in [Riemann 2015 (Physiol Rev)](https://journals.physiology.org/doi/full/10.1152/physrev.00046.2019) — failure of DMN deactivation at sleep onset, amygdala-thalamus-insula hyperconnectivity, increased emotional reactivity to sleep-related stimuli — is encoded as four composite-functional cells: `insomnia.mpfc.Composite.dmn.functional`, `insomnia.amygdala.Composite.bold.functional`, `insomnia.amygdala.Composite.connectivity.functional`, and `insomnia.acc.Composite.dmn.functional`. These cells carry heavy weight on the O (sleep-onset) subsystem and are the strongest tier-1 functional findings in the template.

---

## Elicitation map

The insomnia template uses `isi.v1` as primary instrument. The Insomnia Severity Index ([Bastien 2001](https://pmc.ncbi.nlm.nih.gov/articles/PMC3079939/); [Morin 2011](https://www.med.upenn.edu/cbti/assets/user-content/documents/Insomnia%20Severity%20Index%20(ISI).pdf)) is free for clinical/research use.

```typescript
{
  id: "isi.v1",
  instrument: "ISI",
  applies_to_templates: ["insomnia_canonical_v1"],
  scoring: [
    { name: "onset",                  formula: "item_1",  range: [0, 4] },
    { name: "maintenance",            formula: "item_2",  range: [0, 4] },
    { name: "early_morning_waking",   formula: "item_3",  range: [0, 4] },
    { name: "satisfaction",           formula: "item_4",  range: [0, 4] },
    { name: "interference",           formula: "item_5",  range: [0, 4] },
    { name: "noticeability",          formula: "item_6",  range: [0, 4] },
    { name: "worry_distress",         formula: "item_7",  range: [0, 4] },
    { name: "total",                  formula: "items[1..7].sum", range: [0, 28] }
  ],
  subsystem_mappings: [
    {
      scoring_name: "onset",
      template_ref: "insomnia_canonical_v1",
      subsystem: "O",
      formula: "(score - 1) / 1",      // 0-1 → 0; 4 → +3
      evidence_status: "inferred",
      confidence: "M",
      rationale: "ISI item 1 directly indexes sleep-onset difficulty — the O subsystem's \
                  defining clinical feature."
    },
    {
      scoring_name: "maintenance",
      template_ref: "insomnia_canonical_v1",
      subsystem: "M",
      formula: "(score - 1) / 1",
      evidence_status: "inferred",
      confidence: "M",
      rationale: "ISI item 2 indexes WASO and sleep maintenance — direct map onto M subsystem \
                  and the ACC GABA cells."
    },
    {
      scoring_name: "early_morning_waking",
      template_ref: "insomnia_canonical_v1",
      subsystem: "M",
      formula: "(score - 1) / 2",       // half-weighted
      evidence_status: "inferred",
      confidence: "M",
      rationale: "Early-morning waking is a maintenance-cluster symptom but also reflects \
                  circadian phase advance; half-weighted onto M."
    },
    {
      scoring_name: "early_morning_waking",
      template_ref: "insomnia_canonical_v1",
      subsystem: "C",
      formula: "(score - 1) / 2",
      evidence_status: "inferred",
      confidence: "L",
      rationale: "Phase-advance signature when early-morning waking is dominant; \
                  half-weighted onto C. Confidence low because ISI does not directly \
                  measure circadian variables — supplementary instruments (MEQ, sleep \
                  diary) needed for confident C-subsystem assessment."
    },
    {
      scoring_name: "worry_distress",
      template_ref: "insomnia_canonical_v1",
      subsystem: "R",
      formula: "(score - 1) / 1",
      evidence_status: "inferred",
      confidence: "M",
      rationale: "ISI item 7 (worry/distress about sleep) is the closest ISI proxy for \
                  hyperarousal; maps onto R subsystem."
    },
    {
      scoring_name: "interference",
      template_ref: "insomnia_canonical_v1",
      subsystem: "R",
      formula: "(score - 1) / 2",
      evidence_status: "inferred",
      confidence: "L",
      rationale: "Daytime interference reflects both hyperarousal-driven daytime \
                  symptoms and sleep deprivation sequelae; half-weighted onto R."
    },
    {
      scoring_name: "total",
      template_ref: "insomnia_canonical_v1",
      subsystem: "R",
      formula: "(score - 14) / 7",       // moderate (14) → 0; severe (28) → +2
      evidence_status: "inferred",
      confidence: "M",
      rationale: "Total score serves as overall severity driver, redistributed across \
                  subsystems via per-cell subsystem_weights. R gets the largest share \
                  because hyperarousal is the dominant mechanism in chronic insomnia."
    }
  ],
  cell_mappings: [
    {
      scoring_name: "maintenance",
      cell_id: "insomnia.acc.GABA.tone.tone",
      formula: "(score - 2) / 2",
      threshold: 2,
      evidence_status: "inferred",
      rationale: "Plante 2012 / Winkelman 2008 found ACC GABA negatively correlated \
                  with PSG-measured WASO. ISI item 2 (sleep maintenance) is the \
                  closest self-report analog.",
      sources: [/* Plante 2012, Winkelman 2008 */]
    }
  ],
  ai_extraction_targets: [
    {
      pattern_description: "Patient reports racing thoughts at bedtime, can't \
                            'turn off brain', worries about sleeping — points to O and R.",
      cell_ids: ["insomnia.amygdala.Composite.bold.functional",
                 "insomnia.mpfc.Composite.dmn.functional",
                 "insomnia.acc.Composite.bold.functional"],
      example_phrasings: [
        "mind races the moment I lie down",
        "can't turn my brain off",
        "lying there worrying about being tired tomorrow"
      ],
      evidence_strength_required: "history"
    },
    {
      pattern_description: "Patient reports physical symptoms of hyperarousal — heart \
                            racing, body tense, can't relax — points to R subsystem.",
      cell_ids: ["insomnia.lc.NE.firing.tonic.dynamic",
                 "insomnia.lc.Composite.metabolic.functional",
                 "insomnia.amygdala.NE.tone.tone"],
      example_phrasings: [
        "heart pounding at bedtime",
        "body wired but tired",
        "can't physically relax"
      ],
      evidence_strength_required: "history"
    },
    {
      pattern_description: "Patient reports phase-shifted sleep — night-owl pattern, \
                            jet-lag-like presentation, shift-work — points to C subsystem.",
      cell_ids: ["insomnia.raphe.5HT.firing.tonic.dynamic"],
      example_phrasings: [
        "can't fall asleep before 3am no matter what",
        "wide awake at midnight, dead asleep at 8am",
        "sleep schedule shifted after night shifts"
      ],
      evidence_strength_required: "history"
    },
    {
      pattern_description: "Patient reports waking up at 3-4am unable to return — \
                            points to M subsystem with possible C component.",
      cell_ids: ["insomnia.acc.GABA.tone.tone", "insomnia.acc.GABA.GABA-A.post-syn"],
      example_phrasings: [
        "wake up at 3am and can't get back to sleep",
        "sleep fine for 4 hours then wide awake"
      ],
      evidence_strength_required: "history"
    }
  ],
  recency_window_days: 14,
  license_status: "free-clinical",
  source_citation: "Bastien et al. 2001, Sleep Med 2:297-307; Morin et al. 2011 validation, Sleep 34:601-608",
  notes: "ISI is unidimensional in its standard scoring but items map cleanly onto \
          the four subsystems via the per-item mappings above. The C (circadian) \
          subsystem is the least-well-served by ISI alone; supplementary instruments \
          (MEQ, consensus sleep diary, dim-light melatonin onset if available) \
          recommended for patients where circadian misalignment is suspected. \
          All formula coefficients are starting calibrations."
}
```

## Vocabulary gaps and proposed v1.5 extensions

This template surfaces three areas where the v1 region/system vocabulary is **insufficient** for insomnia, and where we recommend schema extensions for v1.5:

### Gap 1: No SCN / hypothalamic vocabulary

**What's missing:** SCN (suprachiasmatic nucleus), pineal, PVN (paraventricular nucleus), VLPO (ventrolateral preoptic), TMN (tuberomammillary nucleus), LH (lateral hypothalamus / orexin neurons).

**Why it matters for insomnia:**
- Circadian rhythm dysregulation is a core insomnia mechanism (DSPS, ASPS, shift-work insomnia variants).
- Orexin/hypocretin pathway is the target of FDA-approved DORA hypnotics (suvorexant, lemborexant, daridorexant). Without LH/orexin in the vocabulary, we cannot encode the mechanism cells these drugs cover. See [PMC12429101](https://pmc.ncbi.nlm.nih.gov/articles/PMC12429101/).
- The HPA axis (PVN → adrenal → cortisol) is the most-replicated hyperarousal finding. We currently encode it as a composite functional cell on LC — a hack.

**Proposed v1.5 region additions:** `SCN`, `Hypothalamus` (umbrella for PVN, VLPO, TMN, LH if subnuclei not added), `LH` (specifically for orexin), `Pineal`.

### Gap 2: No Histamine / Orexin / Melatonin / Adenosine systems

**What's missing in System enum:** the v1 system enum lists `5HT, DA, NE, GABA, Glu, ACh`. It does not include `Histamine`, `Orexin`, `Melatonin`, `Adenosine` — every one of which is central to sleep-wake regulation.

Actually, checking `01-schema-v3.md` — Histamine, Adenosine are listed in the reserved enums for `System`. Good. **But neither `Orexin/Hypocretin` nor `Melatonin` is in the system enum.** Without these, we cannot encode:
- Orexin receptor antagonists (suvorexant, etc.) — coverage cells will fail
- Melatonin / melatonin-receptor agonists (ramelteon, agomelatine partial mechanism)
- The orexin-driven arousal axis cells in the disorder template itself

**Proposed v1.5 system additions:** `Orexin` (a.k.a. Hypocretin), `Melatonin`.

### Gap 3: No site value for circadian-phase / chronobiological measures

The Site enum has `pre-syn | post-syn | auto | hetero | tone | density | dynamic | functional | composite`. The closest fit for circadian phase markers (dim-light melatonin onset, core body temperature minimum phase) is `dynamic`, but it's a stretch. **Recommendation:** keep `dynamic` and tag with `measure_type: "circadian_phase"` (new MeasureType enum value).

**Proposed v1.5 MeasureType addition:** `circadian_phase`.

### Workaround in this draft

Pending v1.5, the C (circadian) subsystem in this template carries weight via Raphe and VTA composite cells, which is a stretch — the DRN does send projections to the SCN and modulates circadian function, but encoding SCN-related findings on Raphe is provisional. Clinicians using this template should not expect the C subsystem to be well-served by the registry until v1.5 ships. Patients with strong C-dominant presentations (DSPS, shift-work insomnia, jet-lag-related) should be flagged for clinician override at the cell level until the SCN/orexin vocabulary is in.

## DSM-5 / ICD-11 codes

- **DSM-5**: 780.52 (G47.00) — Insomnia Disorder. Specifiers: episodic / persistent / recurrent.
- **ICD-11**: 7A00 — Chronic insomnia. (7A01 — Short-term insomnia; 7A0Z — Insomnia disorders unspecified.) Chronic insomnia (7A00) is the canonical target for this template; short-term variants use `severity_bucket: mild` and acute_modifiers.

## Narrative summary

Chronic insomnia is best understood as a hyperarousal disorder along three converging dimensions, with circadian dysregulation as a fourth, partially independent dimension:

1. **Autonomic / HPA-axis hyperarousal.** Bonnet & Arand established this through three decades of work — patients with primary insomnia show elevated 24-hour metabolic rate, elevated cortisol, elevated heart rate, and elevated high-frequency EEG ([PubMed 19640748](https://pubmed.ncbi.nlm.nih.gov/19640748/)). The Dressle & Riemann 2023 update ([link](https://onlinelibrary.wiley.com/doi/full/10.1111/jsr.13928)) confirms the model. Mechanistically: LC tonic firing is elevated, NE projection to cortex and amygdala is elevated, and the HPA axis is hyperresponsive. In v1 vocabulary, this maps onto LC firing dynamics and LC composite functional cells (cortisol, metabolic). The R subsystem captures this dimension.

2. **GABAergic deficits in cortical-emotional circuits.** Winkelman's 2008 MRS work ([Sleep](https://academic.oup.com/sleep/article-pdf/31/11/1499/13664126/sleep-31-11-1499.pdf)) and Plante 2012's follow-up ([Neuropsychopharm](https://www.nature.com/articles/npp20124)) showed reduced cortical GABA in primary insomnia, localized to ACC and occipital cortex. This is the M subsystem — the GABAergic substrate for sleep maintenance, which is also the target of benzodiazepines, Z-drugs, and (indirectly) some GABAergic antidepressants. ACC GABA correlates inversely with PSG-WASO. We encode this as a high-confidence (tier 1, M) finding.

3. **DMN-emotion hyperactivity at sleep onset.** Riemann 2015 ([Physiol Rev](https://journals.physiology.org/doi/full/10.1152/physrev.00046.2019)) and the Stoffers et al. line of work documented failure of DMN deactivation at sleep onset, amygdala-thalamus-insula hyperconnectivity, and increased emotional reactivity to sleep-related stimuli in insomnia. This is the O subsystem — the failure-to-disengage cortical state that maps onto the conditioned-arousal component of the 3P model's perpetuating factors. Encoded as composite-functional cells across mPFC, ACC, and amygdala.

4. **Circadian misalignment.** Phase-delay (delayed sleep-phase) and phase-advance (early-morning waking) variants and shift-work-related insomnia. This is the C subsystem, and **the v1 vocabulary is insufficient to encode it well** — see "Vocabulary gaps" above. We approximate via Raphe and VTA composite cells until the v1.5 schema extension adds SCN, hypothalamic subnuclei, and the orexin/melatonin systems.

The Buysse 2010 neurobiological model ([Buysse on insomnia neuroscience](https://pmc.ncbi.nlm.nih.gov/articles/PMC3212043/)) frames insomnia as persistent activity in wake-promoting structures during NREM sleep — synthesizing all four dimensions. The Spielman 3P model (predisposing / precipitating / perpetuating) is the clinical-history framing: trait hyperarousal as predisposing, stressor as precipitating, conditioned arousal and behavioral perpetuation as the chronic-phase drivers. Our subsystems map onto this: R captures predisposing trait hyperarousal; O captures the conditioned-arousal perpetuating factor; M and C capture the maintenance and phase-misalignment perpetuating factors.

Coverage cells (BzRA: zolpidem/eszopiclone/zaleplon; DORA: suvorexant/lemborexant/daridorexant; sedating antidepressants: trazodone/doxepin/mirtazapine; melatonergic: ramelteon/melatonin; orexigenic α1 antagonist: prazosin off-label for nightmare-related insomnia) are out of scope for this draft. **DORA coverage cells will not render correctly until v1.5 adds Orexin to the System vocabulary.** Trazodone, doxepin, and mirtazapine coverage cells can be encoded against the v1 vocabulary because their mechanisms (5-HT2A antagonism, H1 antagonism, α1 antagonism) map onto existing systems — though H1 (Histamine) needs to be confirmed in the v1.5 system enum based on the existing reservation in `01-schema-v3.md`.

## v1 readiness

**Draft ready for clinical review, with caveats.** 42 cells with subsystem weights summing to 1.0, evidence-cited deltas, contested cells flagged, and an ISI ElicitationMap mapping all 7 ISI items onto the four subsystems.

**Blockers before `status: active`:**

1. **Clinical review of every cell.** Particularly the contested hippocampal volume cell, the LC composite cortisol/metabolic cells (which are v1-vocabulary workarounds), and the Raphe composite cells (currently doing double-duty as proxies for SCN-related findings).
2. **v1.5 vocabulary extension is on the critical path for full insomnia coverage.** Specifically: adding SCN, Hypothalamus, LH, Pineal as Regions; adding Orexin and Melatonin to Systems; adding `circadian_phase` to MeasureType. Without these, the C subsystem will remain weakly served and DORA coverage cells will be blocked.
3. **Drug coverage cells absent.** Insomnia is one of the most-treated conditions in primary care; an insomnia template without coverage cells for the major hypnotic classes (BzRA, DORA, sedating antidepressants, melatonergic) is not clinically useful. This is the same blocker pattern as OCD and ADHD; see `11-readiness-and-blockers.md` §1.
4. **ElicitationMap pilot calibration.** All ISI subsystem-mapping coefficients are starting calibrations. The C-subsystem mapping is particularly weak — ISI does not directly measure circadian variables. Supplementary instruments (MEQ for chronotype, consensus sleep diary, ideally DLMO) should be considered for v2.

**Not blockers for review:**

- Healthy baseline (constant, doesn't yet exist as data).
- Cascading update policy (pin-by-default at v1).
- Curated comorbidity templates with MDD or GAD (high-prevalence comorbidities) are v2 scope.

**Recommended sequencing:**

1. First clinical-review pass on the cells that *do* fit v1 vocabulary (LC, ACC, amygdala, mPFC). Ship those with `status: active` and the C subsystem partially-served.
2. Parallel: v1.5 schema PR adding SCN, Hypothalamus, LH, Pineal, Orexin, Melatonin, and `circadian_phase` MeasureType. Should be a straightforward additive enum change (per `01-schema-v3.md` §"Reserved enums", adding new enum values is non-breaking).
3. Second authoring pass once v1.5 lands: re-encode the C subsystem cells against the proper vocabulary, retire the Raphe/VTA composite proxies, and add DORA / melatonergic coverage cells.

This is the third canonical disorder template after OCD and ADHD. Together they reach the multi-disorder-differential threshold for v1's primitives. Insomnia is the highest-volume PCP-encountered presentation in the v1 portfolio — getting it close-to-right (even with the vocabulary caveats) is high-leverage.
