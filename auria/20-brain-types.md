# 20 — Brain Types

A patient-identity layer that sits *above* the cell registry and disorder templates. It does not replace any existing logic. The cell registry remains the source of truth, the brain map continues to render at cell resolution, the treatment fit table continues to rank drugs by mechanism overlap. The brain type adds a sixth surface — a coarse, memorable archetype the patient identifies with and the provider reads in two seconds.

Read this after `00-framework-overview.md` and `14-master-design.md`. The type layer is additive; nothing in the methodology files needs to be rewritten to support it.

---

## Why this exists

The cell registry is mechanistically rigorous and clinically useful, but it is not relatable. A patient cannot internalize "OFC 5HT2A density is reduced by 1.5 standard units"; they can internalize "I'm an Obsessive-Perfectionist type — my brain has trouble closing loops." Body types stick because they give people a relatable identity that explains how their body responds to inputs. A brain-type layer does the same job for neuropsychiatric presentation: it gives the patient a story they can hold, and it gives the provider a fast orientation before drilling into cell-level detail.

The clinical justification for adding this layer is dimensional, not categorical. Recent psychiatric research (Drysdale et al. 2017 on depression biotypes; HiTOP dimensional psychopathology; Cloninger temperament dimensions) has moved away from pure DSM-categorical thinking toward a recognition that real brains cluster along a small number of axes that cut across diagnoses. The brain type layer encodes that clustering as a small editorial typology, anchored in those literatures but authored by the clinical team rather than emerged from an unowned dataset.

## How types compose with the existing schema

The type layer sits between `PatientProfile` and the cell-level state:

```
PatientProfile
  ├── baseline_ref: "healthy_v1"
  ├── template_refs: [DisorderTemplate refs]
  ├── brain_type: BrainType  ← new
  ├── PatientSubsystemModifier[]
  └── PatientCellModifier[]
```

The brain type is a stable trait. It is assigned once, near the end of the first encounter, and rarely changes — only on explicit clinician re-classification. It is not recomputed per visit; it is not a function of the current cell state. This is deliberate. The type is the patient's identity layer; if it shifted under their feet whenever treatment moved their cells, it would lose the property that makes it useful (a handle they can keep across visits and across years).

A patient's type is determined at first assessment by:

1. Computing the patient's effective cell vector (baseline + template_refs + modifiers, before treatment).
2. Comparing that vector to each of the six type archetype patterns.
3. Surfacing the closest match to the provider as a *suggestion*, with the secondary closest as a co-suggestion.
4. The provider confirms or overrides. Type is provider-committed, not auto-assigned.

This preserves the platform-wide principle: AI proposes, clinician commits.

## What the type drives, and what it does not

**The type drives:**

- The header chip on the patient detail screen ("Type 3 · Obsessive-Perfectionist").
- The spine of the Layer-3 patient-facing summary. The patient summary opens with the type's name and one-paragraph description.
- A "Recommendations" panel adjacent to the treatment fit table, surfacing curated lifestyle, supplement, therapy-modality, sleep, and communication-style recommendations specific to the type. Each recommendation carries an evidence-tier badge.
- The patient-facing intake mini-app's post-questionnaire summary screen.

**The type does not drive:**

- The cell-level brain map. The map still renders effective deltas from the cell registry. The type may visually frame the map (chip above, narrative below), but does not alter cell values.
- Drug ranking in the treatment fit table. Drugs continue to rank by mechanism overlap with residual. Types do not boost, reweight, or tiebreak drug scores. This separation is intentional: it preserves the mechanism-grounded reasoning that licenses the CDS-exempt posture, and prevents the type layer from becoming a vehicle for opinion that overrides evidence.
- Diagnostic claims. The type is orthogonal to DSM diagnosis. A patient can be Type 3 (Obsessive-Perfectionist) with no formal OCD diagnosis, and an OCD-diagnosed patient may resolve to Type 5 (Cognitive-Ruminative) if the cognitive component dominates the compulsive one.

The clean division — *type drives lifestyle/supplements, cells drive pharmacology* — is the load-bearing design choice. It keeps the type layer honest about what it is (an identity and narrative scaffold informed by clinical pattern-matching) and keeps the cell registry honest about what it is (a mechanism-traceable computational substrate).

## Evidence base for v1

The v1 typology is anchored in three converging research literatures, not invented. Each type below points at a specific published evidence base, named at the end of the type's section. The strongest anchor is the Williams Stanford 2024 *Nature Medicine* study, which clustered 801 patients with depression or anxiety using task-free and task-evoked fMRI of six brain circuits — default mode, salience, attention, negative affect, positive affect, and cognitive control — and identified six biotypes with distinct symptom profiles and differential treatment response. Four of our six types map onto Williams biotypes directly. The remaining two are anchored in equally well-established literatures: the cortico-striato-thalamo-cortical (CSTC) loop work in OCD (Type 4), and the DSM-5-recognized hyperarousal/dissociative split in PTSD established by Lanius and colleagues (Type 6).

The names, plain-language descriptions, and supplement recommendations are editorial. The circuit patterns and clinical associations are not.

## The six v1 types

Each type carries:

- A short name (clinical / formal) and a patient-facing descriptor.
- A characteristic circuit pattern using the v1 region and system vocabulary (`OFC, ACC, dlPFC, vmPFC, mPFC, Caudate, Putamen, vS, Amygdala, Hippocampus, Raphe, VTA, LC` × `5HT, DA, NE, GABA, Glu, ACh`).
- A plain-language description, written for the patient.
- Common presentations (which DSM-language conditions tend to map here).
- What tends to help, in non-pharm terms.
- Watch-fors (clinical risks for this type).
- Supplement recommendations with evidence tiers (A = multiple RCTs or systematic review support; B = some RCT or strong observational support; C = mechanistic rationale with limited clinical trial data).
- Evidence anchors (the published basis for the type).

---

### Type 1 — Anxious-Vigilant (Negative Affect Reactive)

**Stanford biotype lineage.** NC+ — elevated reactivity in the negative affect circuit.

**Characteristic circuit pattern.** Amygdala hyperreactivity to threat cues; elevated LC NE tone; ventral ACC and subgenual cingulate engagement; HPA-axis dysregulation with elevated CRH; mPFC hypoactivation reducing top-down regulation of limbic structures.

**Plain language.** "Your nervous system runs hot. You feel threat before you've consciously identified it. The same circuitry that makes you anxious also makes you alert, intuitive, and protective of the people you love. The cost is that the system stays on when it should be off."

**Common presentations.** Generalized anxiety disorder, panic disorder, somatic anxiety, anxious depression, health anxiety, social anxiety with prominent physiological reactivity.

**What tends to help.**

- Nervous system regulation work: paced breathing, vagal tone exercises, polyvagal-informed therapy, judicious cold exposure.
- Predictability and routine. Surprise is expensive for this type.
- Rhythmic movement (walking, swimming, yoga) over high-intensity intervals.
- Sleep regularity — symptoms degrade fastest under sleep loss.

**Watch-fors.** Entrenched avoidance behaviors; benzodiazepine reliance; caffeine sensitivity often underestimated; misdiagnosis as bipolar when reactive intensity is mistaken for hypomania.

**Supplement recommendations.** Magnesium glycinate (B), L-theanine (B), ashwagandha (B), passionflower (C). Caffeine reduction is often higher-impact than any supplement addition.

**Evidence anchors.** Williams et al. 2024 (Nature Medicine) negative affect biotype; Charney/Drevets neurocircuitry of anxiety; HPA-axis literature on chronic anxiety regulation.

---

### Type 2 — Anhedonic-Depleted (Positive Affect Hypoactive)

**Stanford biotype lineage.** PC− — reduced reactivity in the positive affect circuit.

**Characteristic circuit pattern.** Low VTA dopaminergic output; reduced NAc D2 density and blunted reward prediction signal; vmPFC hypoactivation; flattened HPA diurnal slope; prefrontal hypometabolism.

**Plain language.** "Reward feels muted. The things that should feel good don't, or take more effort to feel anything at all. Motivation costs you more than it costs others — not because you're lazy, but because the circuitry that converts intention into action is running below the line."

**Common presentations.** Melancholic depression with prominent anhedonia, post-viral or post-illness depression, treatment-resistant depression where anhedonia (not sadness) is the dominant complaint, low-energy atypical depression.

**What tends to help.**

- Morning bright light within an hour of waking.
- Behavioral activation — do first, feel later. Don't wait for motivation.
- Scheduled pleasure: small reliable rewards on a calendar, not on impulse.
- Exercise tied to a stable cue (a class, an outdoor location). Intensity matters less than reliability.

**Watch-fors.** Slow response to first-line SSRIs (serotonergic agents do not directly address the dopaminergic deficit); risk of activation-without-mood-improvement on stimulating antidepressants; isolation reinforcing the anhedonia loop.

**Supplement recommendations.** Omega-3 EPA-dominant (A), SAMe (B), L-tyrosine (C), vitamin D if deficient (A for replacement), methylated B-vitamins (B).

**Evidence anchors.** Williams et al. 2024 positive affect biotype; Pizzagalli et al. on reward circuit dysfunction in depression; melancholic-depression dopaminergic literature (decreased DA transmission in melancholic vs. atypical presentations).

---

### Type 3 — Ruminative-Internal (Default Mode Dominant)

**Stanford biotype lineage.** DC+ — default mode circuit hyperconnectivity. Williams identified a biotype with elevated resting activity across DMN, salience, and attention regions (DC+SC+AC+) that responded preferentially to behavioral talk therapy.

**Characteristic circuit pattern.** Default Mode Network hyperconnectivity (mPFC, posterior cingulate, angular gyrus); reduced switching efficiency between DMN and task-positive networks; ACC top-down attenuation; limbic system largely intact and responsive.

**Plain language.** "You think a lot. The same circuitry that makes you thoughtful, self-aware, and good at understanding other people also keeps you stuck on internal narratives. Your problem isn't that you feel too much — it's that your mind doesn't let go cleanly. Thoughts keep running after you'd rather they stop."

**Common presentations.** Ruminative depression, anxious worry as the dominant anxiety presentation, intellectualized OCD without prominent rituals, "high-functioning" depression, generalized anxiety with cognitive component dominating somatic.

**What tends to help.**

- Mindfulness-Based Cognitive Therapy (MBCT) or metacognitive therapy as primary modalities. This biotype responded preferentially to behavioral talk therapy in the Williams trial.
- Attention training: practiced, repeated redirection rather than insight pursuit.
- Structured journaling — same time, time-boxed, with a closing ritual. Open-ended journaling can feed rumination.
- Exercise demanding present-moment focus (climbing, dance, complex movement) over zone-out cardio.

**Watch-fors.** Insight-oriented therapy can reinforce rumination; this type frequently describes itself as "treatment-resistant" when the issue is modality mismatch.

**Supplement recommendations.** Omega-3 (A), curcumin (B), saffron (B), L-methylfolate (B, especially with MTHFR variant), vitamin D if deficient (A).

**Evidence anchors.** Williams et al. 2024 DC+SC+AC+ biotype (behavioral therapy responsive); DMN hyperconnectivity literature in GAD and depression; metacognitive therapy outcome literature.

---

### Type 4 — Compulsive-Locked (CSTC Loop Hyperactive)

**Stanford biotype lineage.** Not in Williams 2024 (cohort did not include OCD). Anchored independently in the OCD-specific CSTC literature.

**Characteristic circuit pattern.** Cortico-striato-thalamo-cortical loop hyperactivity, with subcircuit involvement varying by symptom dimension: the *affective* circuit (ACC, vmPFC → NAc → thalamus) for contamination and harm obsessions; the *dorsal cognitive* circuit (dlPFC → caudate → thalamus) for checking and "just-right" symptoms; the *ventral cognitive* circuit (anterolateral OFC → putamen) for response-inhibition deficits. Reduced OFC 5HT2A density; striatal glutamatergic dysregulation; Caudate D2 elevation; preserved-to-elevated dlPFC engagement (cognitive overcontrol).

**Plain language.** "Your brain has trouble closing loops. Thoughts get on rails and stay there. The system that should mark something as 'done' isn't firing cleanly, so you keep checking, keep refining, keep replaying. The same circuitry that creates the stuckness can make you precise, conscientious, and reliable — the cost is the cycles you can't turn off."

**Common presentations.** OCD across all symptom dimensions (contamination, checking, symmetry, harm, hoarding adjacent); perfectionistic depression; body-focused repetitive behaviors; restrictive eating with ritualistic features; tic-OCD spectrum.

**What tends to help.**

- Exposure and Response Prevention (ERP) as the primary therapy modality.
- Cognitive defusion (ACT techniques): treating thoughts as mental events rather than truths.
- Time-boxed task closure: explicit "this is done" rituals.
- Reducing decision load in low-stakes domains to preserve capacity for high-stakes ones.

**Watch-fors.** Insight-heavy patients can intellectualize ERP into ineffectiveness; SSRI response typically requires higher doses and longer trials than for depression (often 60–80mg fluoxetine-equivalent); aripiprazole augmentation if comorbid tics.

**Supplement recommendations.** NAC (A for OCD augmentation), inositol (B), zinc (C). Caffeine moderation often helps; alcohol typically worsens the loop.

**Evidence anchors.** CSTC subcircuit literature (Saxena, Pauls, Milad); OCD striatal glutamatergic findings; NAC augmentation RCTs in OCD.

---

### Type 5 — Cognitive-Fogged (Cognitive Control Hypoactive)

**Stanford biotype lineage.** The Williams "cognitive biotype" — characterized by reduced activity and connectivity in the cognitive control circuit (dlPFC, dorsal ACC) and corresponding executive function deficits. Affects ~27% of depressed patients. Predicts poor response to standard SSRIs and preferential response to venlafaxine (SNRI) and to TMS targeting the cognitive control circuit (B-SMART-fMRI trial, *Nature Mental Health* 2024).

**Characteristic circuit pattern.** Cognitive control circuit hypoactivity (dlPFC, dorsal ACC); reduced engagement of frontoparietal network during cognitive tasks; intact limbic and reward circuits; executive dysfunction (working memory, sustained attention, cognitive flexibility) on objective testing.

**Plain language.** "Depression isn't only sadness. For you, it shows up as fog — words don't come, decisions feel impossible, and the executive functions you used to rely on are running on a low battery. Sadness may be present too, but the dominant feature is cognitive: the machinery that normally drives focused work isn't engaging."

**Common presentations.** Cognitive depression ("brain fog" predominant), post-COVID cognitive depression, treatment-resistant depression where standard SSRIs have failed, depression with prominent executive dysfunction, midlife depression with concentration as the chief complaint.

**What tends to help.**

- Structured cognitive engagement (skill-based learning, deliberate practice) rather than passive consumption.
- Aerobic exercise — most robust intervention for cognitive control circuit function.
- Sleep architecture, especially preserving slow-wave sleep.
- Reducing cognitive load and decision fatigue during the worst-affected hours.

**Watch-fors.** Misclassified as treatment-resistant when the issue is biotype-treatment mismatch; SSRI monotherapy often inadequate (this type responded better to venlafaxine in Williams trial); patients often self-blame for cognitive deficits that are biology, not effort.

**Supplement recommendations.** Omega-3 (A), methylated B-vitamins (B), citicoline (B), creatine monohydrate (B), vitamin D if deficient (A).

**Evidence anchors.** Williams 2023 *JAMA Network Open* cognitive subtype paper; Williams et al. 2024 *Nature Medicine* biotype study (CC− biotype; venlafaxine responsiveness); B-SMART-fMRI TMS trial (*Nature Mental Health* 2024).

---

### Type 6 — Trauma-Imprinted (Hyperarousal / Dissociative Dual Pattern)

**Stanford biotype lineage.** Not in Williams 2024 (cohort did not stratify by trauma history). Anchored in the DSM-5-recognized dissociative subtype of PTSD and the dual-pattern neurobiological evidence established by Lanius and colleagues.

**Characteristic circuit pattern.** Two reciprocal patterns, both clinically present in trauma-imprinted patients but typically dominant in one direction at a given time. *Hyperarousal pattern:* elevated amygdala and anterior insula activity; reduced mPFC and rostral ACC engagement (affect overwhelms cognition); elevated LC NE tone; HPA dysregulation with hippocampal functional deficit. *Dissociative pattern:* the reverse — mPFC and rostral ACC oversuppression of limbic structures, producing depersonalization and derealization; reduced DMN connectivity; altered insular interoception. Roughly 15–30% of PTSD patients show the dissociative pattern; the remainder typically show hyperarousal.

**Plain language.** "Your nervous system learned that the world was dangerous, and hasn't fully unlearned it. The body remembers things the conscious mind has filed away. Sometimes the system runs hot — vigilance, startle, hyperarousal. Other times it runs cold — distance from your body, your feelings, the moment. The same wiring that protected you then is now expensive. This is not a character flaw. It is a nervous system that took on a job it can be helped to set down."

**Common presentations.** PTSD (with or without the dissociative specifier); complex trauma / cPTSD; somatic symptom disorder with trauma history; anxious depression with prominent hyperarousal; dissociative presentations not amounting to DID.

**What tends to help.**

- Trauma-specialized therapy: EMDR, somatic experiencing, IFS, trauma-focused CBT, sensorimotor psychotherapy. Modality match matters more than dose.
- Predictable rhythm and environment. Stability of place and people is itself an intervention.
- Body-based work alongside cognitive work. This type tends to over-favor talk therapy and under-utilize somatic modalities.
- Co-regulation: time with calm, attuned others changes nervous-system set-points.

**Watch-fors.** Misdiagnosed as borderline personality, bipolar, or treatment-resistant depression when the trauma frame is missed; activation sensitivity to typical antidepressants; benzodiazepine reliance; the dissociative variant frequently missed because patients don't report it spontaneously.

**Supplement recommendations.** Omega-3 (A), magnesium glycinate (B), adaptogens — rhodiola or ashwagandha (B), L-theanine (B). Avoid stimulant supplements.

**Evidence anchors.** Lanius et al. 2012 (*Depression and Anxiety*) — dissociative subtype neurobiological evidence; DSM-5 dissociative-subtype inclusion based on converging symptom, treatment outcome, and psychobiological evidence; recent (2023) review confirming distinct neurobiological and genetic correlates.

---

## Type → disorder template mapping (v1)

The disorder templates in the v1 set declare a `brain_type_anchor` field (see `30-v1.1-schema-extensions.md`) indicating which type a patient with that diagnosis most often resolves to. The reverse mapping — which disorders most often present as each type — is shown below. This is a clinical-pattern table, not a hard rule; the actual type is determined by the patient's cell vector via closest-match.

| Type | Primary disorder anchors | Secondary / overlap |
|------|--------------------------|---------------------|
| Type 1 — Anxious-Vigilant | GAD, Panic Disorder, Social Anxiety, PTSD (hyperarousal subtype) | Anxious depression |
| Type 2 — Anhedonic-Depleted | MDD (melancholic / atypical with anhedonia dominant) | Social Anxiety (reward-deficit dimension), ADHD (motivation dimension) |
| Type 3 — Ruminative-Internal | GAD (worry-dominant) | MDD ruminative, intellectualized OCD |
| Type 4 — Compulsive-Locked | OCD | — |
| Type 5 — Cognitive-Fogged | MDD (cognitive biotype, ~27%), ADHD | Post-COVID cognitive depression |
| Type 6 — Trauma-Imprinted | PTSD (both hyperarousal and dissociative subtypes) | Complex trauma, somatic-symptom with trauma history |

**Insomnia and Adjustment Disorder** are not anchored to a primary type because their presentations are heterogeneous. The brain type is determined from the patient's actual cell vector, not seeded by the diagnosis. Insomnia driven by hyperarousal often resolves to Type 1; insomnia driven by depression often resolves to Type 2 or Type 5.

## Authoring and extending types

The six types are authored as a v1 starting set. They should be expected to evolve. The protocol for extending or revising:

- Type definitions are versioned. `BrainType v1.0` is the initial release. Revisions get new version numbers; existing patient type assignments pin to the version under which they were assigned.
- New types may be added if cell-pattern clustering on real patient data reveals a stable group that doesn't fit any existing archetype. The bar for adding a seventh type is high: it must be distinguishable on the characteristic pattern, recognizable to patients, and lead to non-trivial recommendation differences.
- Recommendation lists (supplements, lifestyle, therapy modalities) are versioned within the type and may be revised more frequently than the type definitions themselves, subject to evidence review.
- The cell-pattern signature of each type is the authoritative substrate. Plain-language descriptions and recommendations derive from it. If the pattern is revised, the description follows.

## What this resolves and what it leaves open

**Resolves.** The "what does the patient remember and tell their friends?" gap. Cell-level reasoning is too fine for that job. The type provides the handle.

**Resolves.** The undifferentiated-patient v1 gap. The new-patient flow in `16-frontend-ux.md` now offers a "No diagnosis yet — assess only" option at intake. Selecting it creates the patient with `diagnosis_status: undifferentiated` and no `template_refs`. The brain type is assigned from questionnaire results, type-driven lifestyle and supplement recommendations are surfaced, and the brain map renders against the healthy baseline. The drug treatment fit table is hidden until a provisional diagnosis is committed (no residual to rank against without a template). Differential diagnosis ranking itself remains a v2+ concern (needs more than three disorder templates to be useful).

**Leaves open.**

- How prominently the type chip appears on the patient detail screen relative to active diagnoses. Style/UX decision deferred to `16-frontend-ux.md` revision.
- Whether the type appears on the patient-facing intake mini-app's post-questionnaire screen, or only in the post-visit summary. Probably the latter, to avoid pre-empting clinician judgment.
- The evidence-tier authoring protocol for type-specific supplement recommendations. Likely inherits from the drug coverage evidence tiering, but the literature is thinner and the protocol needs a separate review pass.
- Whether comorbid type pairings ("primary Type 3, secondary Type 5") are supported in v1 or deferred. Currently spec'd as primary-only; the schema does not preclude later adding a secondary.

## References

Key publications anchoring the v1 typology. The clinical content team should treat these as the starting bibliography when authoring per-type evidence and updating recommendation lists.

- Williams LM et al. (2024). Personalized brain circuit scores identify clinically distinct biotypes in depression and anxiety. *Nature Medicine.* https://www.nature.com/articles/s41591-024-03057-9 — the primary anchor for Types 1, 2, 3, and 5.
- B-SMART-fMRI trial (2024). A cognitive neural circuit biotype of depression showing functional and behavioral improvement after TMS. *Nature Mental Health.* https://www.nature.com/articles/s44220-024-00271-9 — Type 5 treatment evidence.
- Lanius RA et al. (2012). The dissociative subtype of posttraumatic stress disorder: rationale, clinical and neurobiological evidence, and implications. *Depression and Anxiety.* https://onlinelibrary.wiley.com/doi/10.1002/da.21889 — Type 6 anchor.
- Neurobiological and genetic correlates of the dissociative subtype of PTSD (2023). https://pubmed.ncbi.nlm.nih.gov/37023279/ — Type 6 updated evidence.
- Drysdale AT et al. (2017). Resting-state connectivity biomarkers define neurophysiological subtypes of depression. *Nature Medicine.* https://www.nature.com/articles/nm.4246 — historical anchor for the biotyping approach (note: replication concerns documented in Dinga et al. 2019).
- Cortico-striato-thalamo-cortical circuit literature in OCD (Saxena, Pauls, Milad, and others) — anchors Type 4. See https://pmc.ncbi.nlm.nih.gov/articles/PMC6524661/ for a representative review.
- Hierarchical Taxonomy of Psychopathology (HiTOP): https://www.hitop-system.org/the-framework — dimensional-psychopathology context.
- NIMH Research Domain Criteria (RDoC): https://en.wikipedia.org/wiki/Research_Domain_Criteria — six-domain framework that broadly aligns with the type axes used here.

---

Last updated: 2026-05-12
