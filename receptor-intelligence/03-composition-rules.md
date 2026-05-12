# 03 — Composition Rules

How disorder templates compose when a patient has more than one. The five interaction types, the resolution algorithm, and the seeded rules for known comorbidities.

---

## Why composition rules exist

When a PatientProfile has `template_refs.length > 1`, the registry has multiple disorder templates touching the same anatomy. The naive composition is "add them up" — but this is wrong in most clinically relevant cases.

Three real-world examples that prove additive composition fails:

- **OCD + TS at putamen DA tone.** OCD = +2, TS = +3. Sum clipped to +3. Reality (Wong 2007 PET imaging): roughly +3 — not +5, not the sum. Effect = **ceiling** (system saturates).
- **OCD + TS at ventral striatum D2/D3.** OCD alone = −2, TS alone = −2. Naive sum = −3. Reality (Wong 2007): comorbid shows **+1 in left ventral striatum** — the opposite direction. Effect = **novel** (qualitatively different from either alone).
- **MDD + GAD at HPA axis.** Both elevated. Reality: roughly additive. Effect = **additive**.

The composition rule registry encodes which interaction type applies at which scope. A patient's effective vector is computed by walking each cell, finding the most specific applicable rule, and applying it.

## The five interaction types

```typescript
type InteractionType =
  | "additive"           // mechanisms independent; deltas sum
  | "multiplicative"     // mechanisms amplify; delta scales
  | "ceiling"            // system saturates; second disorder doesn't compound
  | "novel"              // qualitatively different value when comorbid
  | "unknown";           // no evidence; flag profile provisional
```

Formula for each:

```typescript
// additive — mechanisms are independent
effective = clip(a + b, -3, +3)

// multiplicative — mechanisms amplify each other
effective = clip(a * factor, -3, +3)   // factor > 1

// ceiling — second disorder doesn't compound much
effective = clip(max_by_magnitude(a, b) ± 0.5, -3, +3)

// novel — comorbid produces a qualitatively different value
effective = novel_value                 // stored on the rule

// unknown — no evidence; profile flagged provisional
effective = clip(a + b, -3, +3)         // fallback to additive
```

Where `a` and `b` are the deltas from the two templates at that cell.

## Resolution algorithm

For each cell where two or more `template_refs` contribute, the runtime applies the **most specific applicable rule**:

1. **cell-scoped rule** — exact match on `cell_id`. If exists, use it.
2. **region-scoped rule** — applies to all cells in this region.
3. **system-scoped rule** — applies to all cells in this neurotransmitter system.
4. **subsystem-scoped rule** — applies to any cell with this subsystem in its weights.
5. **global default** — `additive` clipped to ±3.

Specificity beats confidence — a cell-scoped `unknown` rule beats a region-scoped `additive` rule. (The clinical reviewer chose to record the cell-level uncertainty deliberately.)

Pseudocode:

```python
def resolve_rule(cell, template_a, template_b, rule_registry):
    """Return the most specific composition rule for this cell pair."""
    sorted_pair = sorted([template_a, template_b])
    for scope in ['cell', 'region', 'system', 'subsystem']:
        rule = rule_registry.find_first(
            template_a=sorted_pair[0],
            template_b=sorted_pair[1],
            scope=scope,
            scope_value_matches=cell
        )
        if rule:
            return rule
    return rule_registry.global_default()


def apply_rule(rule, delta_a, delta_b):
    if rule.interaction == 'additive':
        return clip(delta_a + delta_b, -3, +3)
    if rule.interaction == 'multiplicative':
        return clip(delta_a * rule.factor, -3, +3)
    if rule.interaction == 'ceiling':
        a, b = delta_a, delta_b
        bigger = a if abs(a) >= abs(b) else b
        sign = +0.5 if abs(a + b) > abs(bigger) else -0.5
        return clip(bigger + sign, -3, +3)
    if rule.interaction == 'novel':
        return rule.novel_value
    if rule.interaction == 'unknown':
        return clip(delta_a + delta_b, -3, +3)  # additive fallback
```

The result replaces the modifier-summation step in the effective-delta formula for cells governed by composition.

## CompositionRule record shape

```typescript
interface CompositionRule {
  id: string;                              // "{a}.{b}.{scope}.{scope_value}"
  template_a: string;                       // alphabetical ordering for dedup
  template_b: string;
  scope: "cell" | "region" | "system" | "subsystem" | "global";
  scope_value: string;                       // cell_id, region, system, or subsystem key
                                             // empty string for global
  interaction: InteractionType;
  formula: string;                            // human-readable expression
  factor?: number;                            // for multiplicative
  novel_value?: number;                       // for novel
  novel_range?: [number, number];              // optional CI for novel value
  confidence: "H" | "M" | "L";                 // confidence in the rule itself
  evidence_status: "evidenced" | "inferred" | "no-data";
  sources: Source[];
  rationale: string;                           // plain-English why this rule applies
  last_reviewed: string;
  reviewer: string;
  notes: string;
}
```

## Provisional flag

A composed PatientProfile is marked `diagnosis_status: provisional` if **any** of:

- A cell falls through to the global default rather than a specific rule.
- Any rule applied has `confidence: L` or `evidence_status: inferred`.
- Any rule applied has `interaction: unknown`.

Clinician must affirm the comorbid profile to promote to `confirmed`. The rationale: composed comorbidities without specific evidence are the system's best guess, and clinicians need to know that.

## Curated comorbidity templates as alternative

When a comorbidity is well-characterized in the literature (multiple imaging studies in patients with the comorbid pair), the framework supports **curated comorbidity templates** as first-class DisorderTemplate-shaped entities. Setting `curated_comorbidity_template_ref` on the profile bypasses composition entirely — the curated template is used directly.

See `08-comorbidity-templates.md` for OCD+TS as the worked example.

## Seeded rules

The composition rule registry has been seeded with rules across five comorbidity pairs. Below: the eleven rules currently authored.

| # | Comorbidity | Scope | Cell / Target | Interaction | Confidence |
|---|-------------|-------|---------------|-------------|------------|
| 1 | (any+any) | global | — | additive (clipped) | M |
| 2 | OCD+TS | cell | Putamen DA tone | ceiling | H |
| 3 | OCD+TS | cell | Putamen D2/D3 | ceiling | H |
| 4 | OCD+TS | cell | Ventral Striatum D2/D3 | **novel** (+1, Wong 2007) | M |
| 5 | OCD+TS | cell | Putamen CIN density | **novel** | H |
| 6 | OCD+TS | cell | Ventral Striatum DA phasic | **novel** | H |
| 7 | MDD+GAD | system | 5-HT | ceiling | M |
| 8 | MDD+GAD | cell | Hypothalamus HPA | additive | M |
| 9 | GAD+OCD | region | Amygdala | additive | M |
| 10 | MDD+OCD | system | 5-HT | ceiling | M |
| 11 | ADHD+OCD | region | dlPFC | multiplicative (factor 1.3) | L |

The Wong 2007 finding (the "+1 ventral striatum" novel value) is the most evidence-rich and most counterintuitive. It's the kind of finding that justifies the existence of curated comorbidity templates over compositional rules — the comorbid pattern isn't "the sum of the parts."

## Authoring new rules

Rules are added when:

- A cell falls through to global default for a comorbidity that's clinically common.
- Imaging evidence supports a specific composition pattern.
- Clinical experience flags a known interaction the literature confirms (e.g., serotonergic ceiling effects in MDD+anxiety).

Rules are not added speculatively. An empty rule is worse than no rule because it implies decision support that isn't actually decision-supported.

## What this enables

For a patient with `template_refs: [ocd_canonical_v2, ts_canonical_v1]`, the runtime can:

1. Walk every cell where both templates contribute.
2. Look up the most specific applicable rule.
3. Apply the rule's formula to compute the comorbid effective delta.
4. Flag the profile provisional if any cell falls through to global default.

The result is a deterministic, auditable BrainMapPayload for the comorbid profile. The visualization shows ceiling effects, novel findings, and additive overlaps separately — clinicians can see *which* cells the system trusts vs. defaulted on.

## What's not yet built

- **Subsystem-scoped rules.** The subsystem keys aren't standardized across templates yet (OCD has D/F/H/T; MDD has A/N/V/C). Subsystem rules become useful when 3+ disorders share subsystem structure. v2 work.
- **Three-way composition.** A patient with OCD+ADHD+MDD has three template_refs. Pairwise composition is well-defined but the order of application matters when interactions chain. v1 implementation: pairwise composition applied in alphabetical order, with provisional flag on the whole profile. Three-way curated templates are the long-term answer.
- **Rule conflicts.** Two cell-scoped rules at the same cell shouldn't both exist; if they do, the registry validation catches it. There's no UI yet for resolving conflicts.

## Reference: the OCD+TS case as worked example

To make this concrete: a 28-year-old with treatment-naive OCD and motor tics presents.

**Path A — Composition** (without curated template):

Profile gets `template_refs: [ocd_canonical_v2, ts_canonical_v1]`. Runtime walks ~80 cells where both templates contribute. For most cells, no specific rule exists — falls through to global default (`additive`, clipped). Profile flagged `provisional` because additive composition is the system's best guess, not evidenced.

Specific rules exist for the 5 OCD+TS cell rules (rules #2–#6 above). Those cells get evidenced composition.

Result: profile is computable but **provisional**. Clinician sees "this is OCD + TS but the system isn't fully confident in how they compose." Most of the brain map is "best guess by addition."

**Path B — Curated comorbidity template:**

Profile gets `curated_comorbidity_template_ref: "ocd_plus_ts_v1"`. The curated template *is* a populated cell registry — it contains:

- Cells imported unchanged from OCD (OFC 5-HT2A, ACC Glu, amygdala mGluR5)
- Cells imported unchanged from TS (putamen CIN density at −3, ventral striatum phasic DA at +3)
- Modified cells where evidence supports a specific comorbid value (putamen DA tone at +3, ventral striatum D2/D3 at +1 from Wong 2007)
- Its own ElicitationMap that scores Y-BOCS and YGTSS together
- Its own severity calibration

Result: profile gets `diagnosis_status: confirmed` immediately if instruments support it. No provisional flag.

**Treatment narrowing is different between the paths.** Pure OCD's first-line is SSRI. Pure TS often gets alpha-2 agonists. For OCD+TS, the literature strongly supports SSRI + low-dose antipsychotic augmentation (aripiprazole, risperidone) — and the evidence is for the *comorbid state specifically*, not the additive composition. Path B captures this; Path A doesn't.

The framework supports both paths. Curated templates exist for high-prevalence pairs; composition is the fallback. Both produce computable, auditable profiles.
