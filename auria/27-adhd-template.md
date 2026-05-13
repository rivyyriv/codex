# 27 — ADHD Template (Canonical v1, Draft)

> **Clinical review required.** This is a first-draft DisorderTemplate produced by an authoring agent against the v3 schema. Cell deltas, evidence tiers, and subsystem weights are starting calibrations grounded in cited literature, but have not been reviewed or signed off by the clinical team. Do not use for patient-facing rendering until clinical review is complete and the template is moved from `status: draft` to `status: active`.

The second concrete disorder template after OCD. Modeled after `09-ocd-reference-instantiation.md` in structure, depth, and authoring tone. Covers ADHD combined presentation as canonical; inattentive variant noted via subsystem weighting; hyperactive-impulsive presentation flagged as a variant.

---

## Template metadata

```yaml
id: adhd_canonical_v1
schema_version: "3.0"
disorder: ADHD
template_version: "1.0.0"
severity_bucket: moderate             # canonical baseline at moderate severity
phenotype_subtype: combined            # combined presentation; inattentive/hyperactive variants via modifiers
baseline_ref: healthy_v1
elicitation_map_ref: asrs.v1
status: draft
last_reviewed: 2026-05-13
reviewer: authoring-agent
```

The canonical template represents **moderate, unmedicated, trait-baseline ADHD combined presentation in adults**. ADHD is a chronic trait disorder; the 180-day ASRS recency window reflects this. Severity scaling and presentation variants (inattentive-dominant, hyperactive-impulsive-dominant) compose via `PatientSubsystemModifier` redistribution on the four subsystems below.

## Subsystems

ADHD's four canonical subsystems. These follow the dominant network/circuit decomposition in the literature (Castellanos & Proal 2012; Sonuga-Barke & Castellanos 2007):

- **A — Attention / sustained vigilance.** Difficulty sustaining focus, distractibility, default-mode interference during goal-directed tasks. Maps primarily to dlPFC, ACC, parietal attention network, and the LC-NE tonic/phasic arousal system. **This is the dominant subsystem in the inattentive presentation.**
- **E — Executive control / working memory.** Top-down inhibition, working memory, planning, response selection. Maps to dlPFC, ACC, OFC, and frontostriatal loops. Implicated across all presentations.
- **M — Motivation / reward sensitivity.** Reward delay aversion, anhedonic-like under-engagement with non-salient stimuli, sluggish reward prediction. Maps to vS/NAc, VTA, OFC, mPFC reward pathway.
- **H — Hyperactivity / motor impulsivity.** Motor restlessness, impulsive action, response inhibition failure. Maps to putamen, caudate (dorsal), SMA, and the motor inhibition circuit. **This is the dominant subsystem in the hyperactive-impulsive presentation; secondary in inattentive.**

Subsystem weighting choices below reflect the combined presentation. Inattentive variants weight A more heavily and H less; hyperactive-impulsive variants do the reverse. The ASRS subscales (Part A inattention items vs. hyperactivity items) drive this redistribution via the ElicitationMap (see §"Elicitation map" below).

## Cell coverage summary

47 active cells in this draft span:

### Cortical regions
- **dlPFC** — top-down executive control deficits; NE α2A, DA D1, Glu, metabolic. ~8 cells.
- **ACC** — error monitoring and conflict resolution; NE, DA, Glu, GABA. ~6 cells.
- **OFC** — reward valuation, behavioral inhibition; DA D1/D2, 5-HT. ~4 cells.
- **vmPFC** — DMN node, reward integration; DA, Glu. ~3 cells.
- **mPFC** — DMN interference; Glu, GABA. ~3 cells.

### Subcortical / striatal regions
- **Caudate** — frontostriatal executive loop; DAT density, DA tone. ~5 cells.
- **Putamen** — motor habit / hyperactivity circuit; DA, GABA. ~4 cells.
- **vS (ventral striatum / NAc)** — reward pathway; DA D2/D3, DAT. ~4 cells.

### Limbic regions
- **Amygdala** — emotional dysregulation comorbid component; NE, 5-HT2A. ~3 cells.
- **Hippocampus** — working memory binding; ACh, Glu. ~2 cells.

### Brainstem / regulatory
- **LC** — noradrenergic tonic/phasic dysregulation; NE firing dynamics. ~3 cells.
- **VTA** — dopaminergic source; DA firing. ~2 cells.

## The cells

Format per cell: `id` — `target / site` — `delta_best [range]` — `tier/confidence` — subsystem_weights — citation(s) — note.

Where `delta_best` is on the −3 to +3 integer scale (display); ranges encode cross-study variability. `tier`: 1 (direct human imaging / RCT), 2 (post-mortem / animal with strong human correlate), 3 (inferred / computational).

### dlPFC (dorsolateral prefrontal cortex)

| Cell ID | Target / site | δ_best | range | tier/conf | weights | sources |
|---|---|---|---|---|---|---|
| `adhd.dlpfc.NE.alpha2A.post-syn` | α2A adrenergic / post-syn | −2 | [−3,−1] | 1/H | A:0.4, E:0.5, H:0.1 | Arnsten 2009 [PMC2863119](https://pmc.ncbi.nlm.nih.gov/articles/PMC2863119/) |
| `adhd.dlpfc.DA.D1.post-syn` | D1 / post-syn | −2 | [−3,−1] | 2/M | A:0.3, E:0.6, M:0.1 | Arnsten 2009 [PMC2863119](https://pmc.ncbi.nlm.nih.gov/articles/PMC2863119/) |
| `adhd.dlpfc.DA.tone.tone` | DA synaptic tone | −2 | [−3,−1] | 1/H | A:0.3, E:0.5, M:0.2 | Volkow 2009 [PubMed 20856250](https://pubmed.ncbi.nlm.nih.gov/20856250/) |
| `adhd.dlpfc.NE.tone.tone` | NE synaptic tone | −2 | [−3,−1] | 1/H | A:0.5, E:0.4, H:0.1 | Bymaster 2002 [Nature 1395936](https://www.nature.com/articles/1395936) |
| `adhd.dlpfc.Glu.NMDA.post-syn` | NMDA / post-syn | −1 | [−2,0] | 2/M | A:0.3, E:0.7 | Maltezos 2014 [PubMed 35322293](https://pubmed.ncbi.nlm.nih.gov/35322293/) |
| `adhd.dlpfc.Glu.tone.tone` | Glu/Gln synaptic tone | −1 | [−2,+1] | 2/M | A:0.3, E:0.7 | Maltezos meta 2022 [PubMed 35322293](https://pubmed.ncbi.nlm.nih.gov/35322293/) |
| `adhd.dlpfc.GABA.GABA-A.post-syn` | GABA-A / post-syn | +1 | [0,+2] | 3/L | E:0.7, H:0.3 | inferred from disinhibition framework |
| `adhd.dlpfc.Composite.bold.functional` | task-evoked BOLD on inhibitory tasks | −2 | [−3,−1] | 1/H | A:0.3, E:0.6, H:0.1 | Hart 2013 [PubMed 24819224](https://pubmed.ncbi.nlm.nih.gov/24819224/); Castellanos & Proal 2012 [link](https://einsteinmed.edu/uploadedFiles/departments/neurology/Divisions/Child_Neurology/Child_Neurology_References/ADHD/Castellanos%202012.pdf) |

### ACC (anterior cingulate cortex)

| Cell ID | Target / site | δ_best | range | tier/conf | weights | sources |
|---|---|---|---|---|---|---|
| `adhd.acc.NE.alpha2A.post-syn` | α2A / post-syn | −2 | [−3,−1] | 1/M | A:0.5, E:0.5 | Arnsten 2009 [PMC2863119](https://pmc.ncbi.nlm.nih.gov/articles/PMC2863119/) |
| `adhd.acc.DA.tone.tone` | DA synaptic tone | −1 | [−2,0] | 2/M | A:0.4, E:0.6 | Volkow 2009 [PubMed 20856250](https://pubmed.ncbi.nlm.nih.gov/20856250/) |
| `adhd.acc.Glu.tone.tone` | Glu/Gln synaptic tone | −1 | [−2,+1] | 1/M | A:0.3, E:0.7 | Maltezos 2022 [PubMed 35322293](https://pubmed.ncbi.nlm.nih.gov/35322293/) |
| `adhd.acc.GABA.tone.tone` | GABA synaptic tone | +1 | [0,+2] | 3/L | E:0.6, H:0.4 | inferred; contested: methodological |
| `adhd.acc.Composite.bold.functional` | task-evoked BOLD on Stroop/error tasks | −2 | [−3,−1] | 1/H | A:0.3, E:0.6, H:0.1 | Bush 1999 [PubMed 10376114](https://pubmed.ncbi.nlm.nih.gov/10376114/); Rubia 2008 [PubMed 18759938](https://pubmed.ncbi.nlm.nih.gov/18759938/) |
| `adhd.acc.Composite.dmn.functional` | DMN-task switching efficiency | −2 | [−3,−1] | 1/H | A:0.6, E:0.4 | Sonuga-Barke & Castellanos 2007; Liu 2024 [PMC11325164](https://pmc.ncbi.nlm.nih.gov/articles/PMC11325164/) |

### OFC (orbitofrontal cortex)

| Cell ID | Target / site | δ_best | range | tier/conf | weights | sources |
|---|---|---|---|---|---|---|
| `adhd.ofc.DA.D1.post-syn` | D1 / post-syn | −1 | [−2,0] | 2/M | E:0.4, M:0.6 | Tripp & Wickens 2008 [link](https://einsteinmed.edu/uploadedFiles/departments/neurology/Divisions/Child_Neurology/Child_Neurology_References/Executive_Fnc/Tripp.ADHD.pdf) |
| `adhd.ofc.DA.D2.post-syn` | D2 / post-syn | −1 | [−2,0] | 2/M | E:0.3, M:0.7 | Volkow 2009 [PubMed 20856250](https://pubmed.ncbi.nlm.nih.gov/20856250/) |
| `adhd.ofc.DA.tone.tone` | DA synaptic tone | −1 | [−2,0] | 2/M | E:0.3, M:0.7 | Volkow 2009 [PubMed 20856250](https://pubmed.ncbi.nlm.nih.gov/20856250/) |
| `adhd.ofc.Composite.bold.functional` | reward-evaluation BOLD | −1 | [−2,0] | 1/M | E:0.3, M:0.7 | Plichta & Scheres 2014 meta-analysis |

### vmPFC

| Cell ID | Target / site | δ_best | range | tier/conf | weights | sources |
|---|---|---|---|---|---|---|
| `adhd.vmpfc.DA.tone.tone` | DA synaptic tone | −1 | [−2,0] | 2/M | M:0.7, E:0.3 | Volkow 2011 [Journal of Neuroscience](https://www.jneurosci.org/content/32/3/841) |
| `adhd.vmpfc.Glu.tone.tone` | Glu/Gln synaptic tone | 0 | [−1,+1] | 3/L | A:0.4, E:0.3, M:0.3 | no-data; inferred |
| `adhd.vmpfc.Composite.dmn.functional` | DMN node activity (rest vs task) | +2 | [+1,+3] | 1/H | A:0.7, E:0.3 | Sonuga-Barke & Castellanos 2007; [PMC5568884](https://pmc.ncbi.nlm.nih.gov/articles/PMC5568884/) |

### mPFC

| Cell ID | Target / site | δ_best | range | tier/conf | weights | sources |
|---|---|---|---|---|---|---|
| `adhd.mpfc.Glu.tone.tone` | Glu/Gln synaptic tone | −1 | [−2,+1] | 1/M | A:0.4, E:0.6 | Salavert 2018 [link](https://journals.sagepub.com/doi/10.1177/1087054715611492); Maltezos 2022 [PubMed 35322293](https://pubmed.ncbi.nlm.nih.gov/35322293/) |
| `adhd.mpfc.GABA.tone.tone` | GABA tone | +1 | [0,+2] | 3/L | A:0.5, E:0.5 | inferred; contested: methodological |
| `adhd.mpfc.Composite.dmn.functional` | DMN interference index | +2 | [+1,+3] | 1/H | A:0.7, E:0.3 | Sonuga-Barke & Castellanos 2007; Liu 2024 [PMC11325164](https://pmc.ncbi.nlm.nih.gov/articles/PMC11325164/) |

### Caudate

| Cell ID | Target / site | δ_best | range | tier/conf | weights | sources |
|---|---|---|---|---|---|---|
| `adhd.caudate.DA.DAT.pre-syn` | DAT density / pre-syn | +1 | [−1,+2] | 1/M | A:0.3, E:0.4, H:0.3 | Spencer 2007 [PMC2715944](https://pmc.ncbi.nlm.nih.gov/articles/PMC2715944/); Fusar-Poli 2012 meta [link](https://psychiatryonline.org/doi/10.1176/appi.ajp.2011.11060940) — *contested: state-trait (medication exposure confound)* |
| `adhd.caudate.DA.D2.post-syn` | D2 / post-syn | −1 | [−2,0] | 1/M | M:0.4, E:0.3, H:0.3 | Volkow 2009 [PubMed 20856250](https://pubmed.ncbi.nlm.nih.gov/20856250/) |
| `adhd.caudate.DA.tone.tone` | DA synaptic tone | −2 | [−3,−1] | 1/H | A:0.3, E:0.4, H:0.3 | Volkow 2009; Tripp & Wickens 2008 |
| `adhd.caudate.GABA.GABA-A.post-syn` | GABA-A on MSNs / post-syn | 0 | [−1,+1] | 3/L | E:0.5, H:0.5 | no-data; inferred |
| `adhd.caudate.Composite.bold.functional` | task-evoked BOLD on go/no-go | −2 | [−3,−1] | 1/H | A:0.3, E:0.4, H:0.3 | Hart 2013 [PubMed 24819224](https://pubmed.ncbi.nlm.nih.gov/24819224/) |

### Putamen

| Cell ID | Target / site | δ_best | range | tier/conf | weights | sources |
|---|---|---|---|---|---|---|
| `adhd.putamen.DA.DAT.pre-syn` | DAT density / pre-syn | +1 | [−1,+2] | 1/M | H:0.6, E:0.3, A:0.1 | Spencer 2007 [PMC2715944](https://pmc.ncbi.nlm.nih.gov/articles/PMC2715944/) — *contested: state-trait* |
| `adhd.putamen.DA.tone.tone` | DA synaptic tone | −1 | [−2,0] | 1/M | H:0.6, E:0.3, A:0.1 | Volkow 2009 |
| `adhd.putamen.GABA.tone.tone` | GABA tone | −1 | [−2,+1] | 2/M | H:0.7, E:0.3 | Edden 2012 [link](https://www.sciencedirect.com/science/article/abs/pii/S0925492720300548) (children, 7T) |
| `adhd.putamen.Composite.bold.functional` | motor-inhibition BOLD | −2 | [−3,−1] | 1/H | H:0.7, E:0.3 | Hart 2013 |

### vS (ventral striatum / NAc)

| Cell ID | Target / site | δ_best | range | tier/conf | weights | sources |
|---|---|---|---|---|---|---|
| `adhd.vS.DA.D2.post-syn` | D2 / post-syn | −2 | [−3,−1] | 1/H | M:0.8, E:0.2 | Volkow 2009 [PubMed 20856250](https://pubmed.ncbi.nlm.nih.gov/20856250/) |
| `adhd.vS.DA.D3.post-syn` | D3 / post-syn | −2 | [−3,−1] | 1/M | M:0.8, E:0.2 | Volkow 2009 |
| `adhd.vS.DA.DAT.pre-syn` | DAT density / pre-syn | +1 | [−1,+2] | 1/M | M:0.5, H:0.3, A:0.2 | Volkow 2009; Spencer 2007 |
| `adhd.vS.DA.tone.tone` | DA synaptic tone | −2 | [−3,−1] | 1/H | M:0.7, E:0.2, A:0.1 | Volkow 2011 [J Neuroscience](https://www.jneurosci.org/content/32/3/841) |

### Amygdala

| Cell ID | Target / site | δ_best | range | tier/conf | weights | sources |
|---|---|---|---|---|---|---|
| `adhd.amygdala.NE.alpha2A.post-syn` | α2A / post-syn | −1 | [−2,0] | 2/M | A:0.3, H:0.7 | inferred from arousal regulation literature |
| `adhd.amygdala.5HT.5HT2A.post-syn` | 5-HT2A / post-syn | 0 | [−1,+1] | 3/L | H:0.7, A:0.3 | no-data; inferred |
| `adhd.amygdala.Composite.bold.functional` | task-evoked BOLD on emotional faces | +1 | [0,+2] | 1/M | H:0.7, A:0.3 | Brotman 2010 [PMC3155780](https://pmc.ncbi.nlm.nih.gov/articles/PMC3155780/) — emotional dysregulation component |

### Hippocampus

| Cell ID | Target / site | δ_best | range | tier/conf | weights | sources |
|---|---|---|---|---|---|---|
| `adhd.hippocampus.ACh.tone.tone` | ACh tone | −1 | [−2,0] | 3/L | A:0.5, E:0.5 | inferred from working-memory ACh literature |
| `adhd.hippocampus.Glu.tone.tone` | Glu tone | 0 | [−1,+1] | 3/L | A:0.4, E:0.6 | no-data |

### LC (locus coeruleus)

| Cell ID | Target / site | δ_best | range | tier/conf | weights | sources |
|---|---|---|---|---|---|---|
| `adhd.lc.NE.firing.tonic.dynamic` | NE tonic firing rate | +2 | [+1,+3] | 2/H | A:0.6, H:0.3, E:0.1 | Bari & Aston-Jones 2013 [PMC3445720](https://pmc.ncbi.nlm.nih.gov/articles/PMC3445720/); Aston-Jones & Cohen 2005 — **excessive tonic, reduced phasic ratio is the core LC abnormality** |
| `adhd.lc.NE.firing.phasic.dynamic` | NE phasic firing on task | −2 | [−3,−1] | 2/H | A:0.7, E:0.2, H:0.1 | Bari & Aston-Jones 2013 |
| `adhd.lc.NE.tone.tone` | NE tone (composite) | 0 | [−1,+1] | 2/M | A:0.5, E:0.3, H:0.2 | composite: tonic up, phasic down; net tone disputed |

### VTA

| Cell ID | Target / site | δ_best | range | tier/conf | weights | sources |
|---|---|---|---|---|---|---|
| `adhd.vta.DA.firing.tonic.dynamic` | DA tonic firing | −1 | [−2,0] | 2/M | M:0.6, A:0.2, E:0.2 | Tripp & Wickens 2008 — reward-prediction dynamic model |
| `adhd.vta.DA.firing.phasic.dynamic` | DA phasic firing to reward cues | −2 | [−3,−1] | 2/M | M:0.8, E:0.2 | Tripp & Wickens 2008; [eNeuro 2018](https://www.eneuro.org/content/5/2/ENEURO.0007-18.2018) |

---

## Key authoring choices

### 1. The DAT-density direction is contested

Direction of the striatal DAT-density finding in ADHD is the most contested cell in the template. The Fusar-Poli 2012 meta-analysis ([link](https://psychiatryonline.org/doi/10.1176/appi.ajp.2011.11060940)) concluded that elevated striatal DAT density seen in many studies is at least partly an adaptation to prior psychostimulant exposure rather than a primary ADHD finding. We encode `delta_best: +1` (the consensus finding for unmedicated adults in PET studies; Spencer 2007) with `delta_range: [−1, +2]` and `contested: state-trait`. The medication-naive subgroup may have lower deltas; this is an open question.

### 2. LC tonic vs. phasic was split into two cells, not flattened

The LC-NE finding in ADHD is not "low NE" — it is **excessive tonic firing with reduced phasic-to-tonic ratio** (Aston-Jones & Cohen 2005; Bari & Aston-Jones 2013). Flattening this to a single `lc.NE.tone.tone` cell would conceal the mechanism that explains why atomoxetine and α2A agonists (guanfacine, clonidine) work: they correct the ratio, not the level. We split into `firing.tonic.dynamic` (+2) and `firing.phasic.dynamic` (−2) with a separate composite tone cell at 0. This is the same authoring pattern the OCD template uses for VTA firing dynamics.

### 3. DMN interference cells are encoded as composite/functional

The default-mode-interference hypothesis (Sonuga-Barke & Castellanos 2007) is a network-level finding, not a single-receptor finding. We encode it as `Composite.dmn.functional` cells on the vmPFC, mPFC, and ACC. These cells contribute heavily to the A (attention) subsystem and carry tier-1 evidence from multiple meta-analyses ([PMC5568884](https://pmc.ncbi.nlm.nih.gov/articles/PMC5568884/); [PMC11325164](https://pmc.ncbi.nlm.nih.gov/articles/PMC11325164/)). This pattern — encoding network-level findings as composite cells — is also how the OCD template handles CSTC loop measures.

### 4. Inattentive vs. combined: subsystem weighting, not separate templates

The literature is split on whether ADHD-I and ADHD-C are distinct entities or a severity continuum ([PubMed 35219873](https://pubmed.ncbi.nlm.nih.gov/35219873/)). v1 takes the continuum position: one canonical template, with the ASRS Part B subscales producing subsystem modifiers that redistribute weight across A vs. H. The inattentive presentation is approximated by `subsystem_modifier{A: +1, H: −0.5}`; hyperactive-impulsive by the inverse. This is encoded directly in the ElicitationMap rather than as `phenotype_subtype` field on the template — keeping the template count down. If pilot data shows the continuum model is wrong, splitting into `adhd_inattentive_v1` and `adhd_combined_v1` is a v2 option.

### 5. Glutamate findings are flagged contested

ADHD glutamate findings in the medial prefrontal cortex are notoriously inconsistent — Maltezos 2022 ([PubMed 35322293](https://pubmed.ncbi.nlm.nih.gov/35322293/)) found a small Glu/Gln imbalance favoring lower glutamate but with high cross-study variability. We encode `delta_best: −1` with `delta_range: [−2, +1]` and tag `contested: methodological`. Clinicians using the template at moderate severity should see this as a marked cell.

---

## Elicitation map

The ADHD template uses `asrs.v1` as primary instrument. ASRS-v1.1 is free for clinical/research use ([WHO/Kessler 2005 link](https://contentmanager.med.uvm.edu/docs/default-source/ahec-documents/adult_adhd_self_report_scale.pdf?sfvrsn=2)). Mapping sketch (full ElicitationMap entry to be added to `04-elicitation-maps.md`):

```typescript
{
  id: "asrs.v1",
  instrument: "ASRS-v1.1",
  applies_to_templates: ["adhd_canonical_v1"],
  scoring: [
    { name: "part_a",      formula: "items[1..6].sum",  range: [0, 24] },
    { name: "inattention", formula: "items[1..4]+items[7..11].sum (subset of 18-item)", range: [0, 36] },
    { name: "hyperactivity_motor", formula: "items[5..6]+items[12..14].sum", range: [0, 20] },
    { name: "hyperactivity_verbal", formula: "items[15..18].sum", range: [0, 16] },
    { name: "total_18",     formula: "items[1..18].sum", range: [0, 72] }
  ],
  subsystem_mappings: [
    {
      scoring_name: "inattention",
      template_ref: "adhd_canonical_v1",
      subsystem: "A",
      formula: "(score - 18) / 6",   // ~moderate (18) → 0; severe (36) → +3
      evidence_status: "inferred",
      confidence: "M",
      rationale: "ASRS inattention subscale items map directly onto sustained-attention \
                  and distractibility constructs; calibrated against Part A clinical \
                  threshold (≥4 of 6 = positive screen)."
    },
    {
      scoring_name: "inattention",
      template_ref: "adhd_canonical_v1",
      subsystem: "E",
      formula: "(score - 18) / 9",   // half-weight on executive
      evidence_status: "inferred",
      confidence: "M",
      rationale: "Inattention items overlap with executive (working memory, planning, \
                  task-switching) constructs. Half-weight relative to A."
    },
    {
      scoring_name: "hyperactivity_motor",
      template_ref: "adhd_canonical_v1",
      subsystem: "H",
      formula: "(score - 10) / 3.3", // ~moderate (10) → 0; severe (20) → +3
      evidence_status: "inferred",
      confidence: "M",
      rationale: "Motor hyperactivity subscale captures restlessness, fidgeting, \
                  inability to relax — maps to H subsystem (putamen, motor inhibition)."
    },
    {
      scoring_name: "hyperactivity_verbal",
      template_ref: "adhd_canonical_v1",
      subsystem: "H",
      formula: "(score - 8) / 2.7",
      evidence_status: "inferred",
      confidence: "M",
      rationale: "Verbal-impulsivity subscale (interrupting, talking over) reflects \
                  response-inhibition failure. Aggregates with motor into H."
    },
    {
      scoring_name: "total_18",
      template_ref: "adhd_canonical_v1",
      subsystem: "M",
      formula: "(score - 36) / 12",
      evidence_status: "inferred",
      confidence: "L",
      rationale: "ASRS does not directly score reward/motivation symptoms; \
                  M-subsystem modifier is approximated from total severity. \
                  Clinician override recommended where motivation is dominant phenotype."
    }
  ],
  cell_mappings: [
    {
      scoring_name: "hyperactivity_motor",
      cell_id: "adhd.caudate.DA.DAT.pre-syn",
      formula: "(score - 12) / 4",
      threshold: 12,
      evidence_status: "inferred",
      rationale: "Hyperactivity subscale has strongest multi-study evidence linking \
                  to caudate DAT availability (Spencer 2007; Fusar-Poli 2012 meta).",
      sources: [/* Spencer 2007, Fusar-Poli 2012 */]
    }
  ],
  ai_extraction_targets: [
    {
      pattern_description: "Patient reports being late, missing appointments, losing \
                            objects, difficulty initiating tasks — points to A/E subsystems.",
      cell_ids: ["adhd.dlpfc.NE.alpha2A.post-syn", "adhd.lc.NE.firing.phasic.dynamic",
                 "adhd.mpfc.Composite.dmn.functional"],
      example_phrasings: [
        "always running late",
        "can't get started on work",
        "loses keys, phone constantly",
        "mind wanders during meetings"
      ],
      evidence_strength_required: "history"
    },
    {
      pattern_description: "Patient reports motor restlessness, can't sit still, \
                            interrupts others — points to H subsystem.",
      cell_ids: ["adhd.putamen.DA.DAT.pre-syn", "adhd.putamen.Composite.bold.functional"],
      example_phrasings: [
        "fidgets constantly",
        "interrupts in conversation",
        "can't sit through a movie"
      ],
      evidence_strength_required: "history"
    },
    {
      pattern_description: "Patient reports anhedonia-like under-engagement with \
                            non-novel tasks, novelty-seeking, reward delay aversion — \
                            points to M subsystem.",
      cell_ids: ["adhd.vS.DA.D2.post-syn", "adhd.vta.DA.firing.phasic.dynamic"],
      example_phrasings: [
        "can't get motivated unless it's interesting",
        "needs deadlines to function",
        "boredom is intolerable"
      ],
      evidence_strength_required: "explicit_test"
    }
  ],
  recency_window_days: 180,
  license_status: "free-clinical",
  source_citation: "Kessler et al. 2005, Psychol Med 35:245-256",
  notes: "All formula coefficients are starting calibrations. ASRS captures inattention \
          and hyperactivity directly; motivation (M) subsystem requires clinician override \
          or AI-extracted history because ASRS items don't cover it well."
}
```

## Inattentive vs. combined presentation handling

The combined presentation is the canonical baseline. The two variants compose as follows:

**Inattentive-dominant** (DSM-5 314.00 / 6A05.0): The patient's ASRS produces high inattention subscale (≥15 on the 9-item inattention subscale) and low hyperactivity subscale (<6 combined motor + verbal). The ElicitationMap maps these to `subsystem_modifier{A: +2, H: −1}`, which redistributes onto cells via subsystem_weights. The net effect: dlPFC NE α2A and LC firing dynamics deltas amplify; putamen and dorsal-caudate motor-circuit deltas attenuate.

**Hyperactive-impulsive-dominant** (DSM-5 314.01 / 6A05.1, rare in adults — typically a childhood presentation that converts to combined or inattentive in adolescence): Inverse weighting. Encoded as a variant rather than a separate template because adult prevalence is low and the underlying circuit findings overlap heavily.

**Combined** (DSM-5 314.01 / 6A05.2): The template's canonical baseline. No subsystem modifier needed beyond severity.

If pilot data shows the inattentive and combined presentations diverge sharply on cells the template currently shares — for example, if inattentive-dominant patients show *no* striatal DAT elevation while combined patients do — the split into separate templates becomes warranted. v1 ships with the unified template and revisits at v2.

## DSM-5 / ICD-11 codes

- **DSM-5**: 314.00 (Predominantly inattentive) / 314.01 (Combined or Predominantly hyperactive-impulsive). Single ADHD diagnosis with presentation specifier.
- **ICD-11**: 6A05.0 (inattentive presentation) / 6A05.1 (hyperactive-impulsive) / 6A05.2 (combined). Codes are sibling under 6A05 Attention deficit hyperactivity disorder.

## Narrative summary

ADHD's neurobiology centers on three interacting circuit-level findings, each well-replicated in human imaging:

1. **Reduced dopaminergic signaling in the reward pathway** — Volkow's PET series ([PubMed 20856250](https://pubmed.ncbi.nlm.nih.gov/20856250/); [J Neurosci 2011](https://www.jneurosci.org/content/32/3/841)) showed reduced D2/D3 receptor availability and reduced DA release in the ventral striatum (NAc), midbrain, and caudate in adults with ADHD, correlated with motivation and attention ratings. This is the core motivational deficit — the M subsystem in this template — and underlies the reward-delay-aversion phenotype and the response to stimulant augmentation of DA signaling.

2. **Prefrontal hypoactivation during executive tasks** — Castellanos & Proal 2012 ([link](https://einsteinmed.edu/uploadedFiles/departments/neurology/Divisions/Child_Neurology/Child_Neurology_References/ADHD/Castellanos%202012.pdf)), Hart 2013 ([PubMed 24819224](https://pubmed.ncbi.nlm.nih.gov/24819224/)), and others document task-evoked hypoactivation in dlPFC, ACC, and right IFC during inhibition, working memory, and time-discrimination tasks. The downstream mechanism in the v1 vocabulary: reduced NE α2A signaling (Arnsten 2009) and reduced D1 stimulation in dlPFC, expressed as both receptor-level deltas and composite BOLD-functional cells. This is the E subsystem and overlaps heavily with A.

3. **LC-NE tonic/phasic dysregulation** — Aston-Jones & Cohen 2005 and Bari & Aston-Jones 2013 ([PMC3445720](https://pmc.ncbi.nlm.nih.gov/articles/PMC3445720/)) showed that ADHD-like attentional dysfunction is associated with excessive tonic LC firing and reduced phasic-to-tonic ratio, not simply "low NE." Atomoxetine corrects the ratio. This is the A subsystem's brainstem driver and is encoded as two split cells (tonic up, phasic down) rather than a flattened tone cell.

4. **Default-mode interference** — Sonuga-Barke & Castellanos 2007 and a now-large meta-analytic base ([PMC5568884](https://pmc.ncbi.nlm.nih.gov/articles/PMC5568884/); [PMC11325164](https://pmc.ncbi.nlm.nih.gov/articles/PMC11325164/)) show that ADHD patients fail to fully suppress default-mode activity (vmPFC, mPFC, PCC) when transitioning to task-positive states, producing attentional lapses. This is the A subsystem's cortical signature and is encoded as composite/dmn-functional cells.

Striatal DAT findings are the most-discussed-but-most-contested cells in the template. We follow Fusar-Poli 2012's meta-analytic conclusion ([link](https://psychiatryonline.org/doi/10.1176/appi.ajp.2011.11060940)) and flag `contested: state-trait`.

Coverage cells (stimulants — methylphenidate, mixed amphetamine salts, lisdexamfetamine; non-stimulants — atomoxetine, guanfacine, clonidine; off-label — bupropion, modafinil) are out of scope for this draft and will be added in a separate authoring pass. Stimulants are the highest-priority coverage entries because they map directly onto the M and A subsystems via DA tone and DAT inhibition.

## v1 readiness

**Ready for clinical review.** The template is at the same level of completeness as the OCD canonical v2 was at clinical-review handoff: ~47 cells with subsystem weights summing to 1.0, evidence-cited deltas, contested cells flagged, and an ElicitationMap mapping ASRS-v1.1 onto the four subsystems.

**Blockers before `status: active`:**

1. **Clinical review of every cell.** Particularly the contested DAT cells, the LC firing-dynamic split, and the DMN composite cells — each carries authoring decisions that need clinical sign-off.
2. **Drug coverage cells absent.** ADHD is treated; an ADHD template without stimulant + atomoxetine + guanfacine coverage cells can render BrainMap but cannot compute treatment fit. This blocks the same primitive as in OCD (see `11-readiness-and-blockers.md` §1).
3. **ElicitationMap pilot calibration.** All ASRS subsystem-mapping coefficients are starting calibrations. The motivation (M) subsystem mapping in particular is weak — ASRS does not directly index reward/motivation symptoms. Pilot data is required before clinical use.
4. **Inattentive-presentation cell-level validation.** v1 ships with the unified template and presentation-as-subsystem-modifier approach. If pilot data shows inattentive patients diverge on specific cells (e.g., no DAT elevation in pure inattentive cases), splitting into separate templates becomes warranted.

**Not blockers for review:**

- The healthy baseline doesn't exist yet (every PatientProfile references `healthy_v1` as a constant; arithmetic is unaffected).
- Cascading update policy is parked (pin-by-default at v1).
- Curated comorbidity templates with MDD or anxiety (high-prevalence ADHD comorbidities) are v2 scope.

This is the second canonical disorder template after OCD. Together with insomnia (`28-insomnia-template.md`) and the existing OCD template, the v1 portfolio reaches three disorders — the minimum for multi-disorder differential distance ranking to be a meaningful primitive.
