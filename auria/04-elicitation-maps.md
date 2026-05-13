# 04 — Elicitation Maps

The translation layer between intake instruments and the cell registry. Without this, the framework can describe a disorder but not map an individual brain.

---

## What an ElicitationMap is

ElicitationMap answers one question:

> When this questionnaire produces this score, which cells/subsystems change, and by how much?

It lives on each DisorderTemplate. It's data, not code — a lookup table. Each entry is one of:

1. **Questionnaire mapping** (deterministic math) — "Y-BOCS obsessions subscale × formula = D-subsystem modifier." This is what populates `PatientSubsystemModifier` automatically. No human in the loop after the questionnaire is scored.
2. **AI extraction target** — "When parsing clinician notes for this disorder, look for evidence pointing to these specific cells." Shapes what the AI tries to extract from free-form text. A constraint on AI — narrowing the search space so the model isn't proposing modifiers to random cells.

## Why it must exist

Without an ElicitationMap, there's a gap:

```
Y-BOCS total: 24                          ← what the patient produces
                  ???
ocd.OFC.5HT.5HT2A.post-syn Δ: -3          ← what the registry needs
```

Something has to bridge those two. ElicitationMap is that bridge, made explicit and auditable.

The alternatives are worse:

- **Hard-coded in app logic** — not auditable, not citable, can't change per disorder.
- **AI infers each time** — non-deterministic, not reproducible, regulatory nightmare.
- **Clinician fills in cells manually each intake** — defeats the point of using questionnaires.

## ElicitationMap schema

```typescript
interface ElicitationMap {
  id: string;                          // "{instrument}.v{version}", e.g. "y-bocs.v1"
  schema_version: "3.0";
  instrument: string;                  // e.g. "Y-BOCS", "PHQ-9"
  instrument_version: string;
  applies_to_templates: string[];       // disorder template IDs this map applies to
  scoring: ScoringRule[];
  subsystem_mappings: SubsystemMapping[];
  cell_mappings: CellMapping[];          // sparse — only for items with strong cell-level evidence
  ai_extraction_targets: AIExtractionTarget[];
  recency_window_days: number;            // e.g. 7 for Y-BOCS, 180 for ASRS
  license_status: "free-clinical" | "permission-required" | "proprietary";
  source_citation: string;
  notes: string;
}

interface ScoringRule {
  name: string;                          // "obsessions subscale", "total score"
  formula: string;                        // expression over item responses
  range: [number, number];
}

interface SubsystemMapping {
  scoring_name: string;                   // refers to a ScoringRule.name
  template_ref: string;                   // disorder template
  subsystem: string;                      // e.g. "D" for OCD's Doubt
  formula: string;                         // expression mapping score → delta_modifier
                                           // e.g. "(score - 8) / 8" maps Y-BOCS 8-24 to 0-2
  evidence_status: "evidenced" | "inferred";
  confidence: "H" | "M" | "L";
  rationale: string;
}

interface CellMapping {
  scoring_name: string;
  cell_id: string;                         // exact cell to override
  formula: string;
  threshold: number | null;                // optional minimum score to apply
  evidence_status: "evidenced" | "inferred";
  rationale: string;
  sources: Source[];                       // CITED — cell-level mappings need evidence
}

interface AIExtractionTarget {
  pattern_description: string;             // human-readable: what to look for in notes
  cell_ids: string[];                       // cells the pattern points to
  example_phrasings: string[];               // training examples
  evidence_strength_required: "any" | "explicit_test" | "imaging" | "history";
}
```

## v1 instruments

The v1 ElicitationMap library covers five validated instruments. All are free for clinical/research use:

| Instrument | What it scores | Applies to | Recency window |
|------------|----------------|------------|----------------|
| Y-BOCS | OCD severity (obsessions + compulsions subscales) | OCD templates | 7 days |
| PHQ-9 | Depression severity, self-report | MDD templates, screen for any | 14 days |
| MADRS | Depression severity, clinician-rated | MDD templates | 7 days |
| GAD-7 | Generalized anxiety severity | GAD templates | 14 days |
| ASRS | Adult ADHD screening | ADHD templates | 180 days (chronic trait) |

Additional instruments planned for v2: YGTSS (tic severity), DOCS / OCI-R (OCD second-line, F-subsystem coverage), PCL-5 (PTSD).

## Multi-instrument administration rule

When a patient has multiple instruments mapping to the same subsystem (e.g., PHQ-9 and MADRS both score depression severity), modifiers do not stack. The higher-confidence source takes precedence — clinician-rated MADRS supersedes self-rated PHQ-9 in most cases.

The resolution rule lives in the intake pipeline, not in the ElicitationMap itself. Pipeline logic:

```
For each (template_ref, subsystem) pair with multiple instruments:
  1. Filter to instruments whose recency_window_days hasn't expired.
  2. Pick the highest-confidence source available.
     Confidence ranking:
       clinician-rated > self-report
       within-window > expired-window
       newer > older
  3. Drop the others. Don't average, don't sum.
```

## Cross-disorder questionnaires

Y-BOCS scoring is OCD-specific, but a patient with depression and OCD comorbidity will produce a non-zero Y-BOCS. The ElicitationMap is keyed to the disorder template (`applies_to_templates`), so Y-BOCS modifiers only apply when an OCD template (or curated OCD-comorbid template) is referenced in the profile.

A patient without an OCD template_ref can still complete Y-BOCS — the score is recorded — but produces no modifiers until an OCD template is added.

## Subsystem-default vs cell-level overrides

The default approach is **subsystem-level mapping**. A questionnaire score modifies a subsystem (e.g., Y-BOCS obsessions → OCD's D subsystem), and that modifier redistributes onto cells via each cell's `subsystem_weights`.

This is the common path because:

- Subsystems are clinically meaningful aggregates the field already uses.
- Cell-level claims from a questionnaire score are usually unjustified — Y-BOCS doesn't directly measure OFC 5-HT2A density.
- Subsystem mapping keeps clinicians out of cell-level reasoning by default.

Cell-level mappings are reserved for findings with **multi-study replication and clear effect direction**. Examples worth encoding at cell-level:

- ASRS hyperactivity subscale → caudate DA transporter (multiple imaging studies).
- Y-BOCS compulsions subscale → caudate metabolic activity (FDG-PET evidence).

When in doubt, encode at subsystem level. Cell-level overrides accumulate; the audit dashboard surfaces cell mappings for review.

## ElicitationMap example — Y-BOCS sketch

```typescript
{
  id: "y-bocs.v1",
  instrument: "Y-BOCS",
  applies_to_templates: ["ocd_canonical_v2", "ocd_plus_ts_v1"],
  scoring: [
    {
      name: "obsessions_subscale",
      formula: "items[1..5].sum",
      range: [0, 20]
    },
    {
      name: "compulsions_subscale",
      formula: "items[6..10].sum",
      range: [0, 20]
    },
    {
      name: "total",
      formula: "items[1..10].sum",
      range: [0, 40]
    }
  ],
  subsystem_mappings: [
    {
      scoring_name: "obsessions_subscale",
      template_ref: "ocd_canonical_v2",
      subsystem: "D",                    // Doubt and over-checking
      formula: "(score - 8) / 6",        // ~moderate (8) → 0; severe (20) → +2
      evidence_status: "inferred",
      confidence: "M",
      rationale: "Y-BOCS obsessions subscale captures intrusive thoughts and \
                  doubt-related symptoms. Calibration is starting; pilot data needed."
    },
    {
      scoring_name: "compulsions_subscale",
      template_ref: "ocd_canonical_v2",
      subsystem: "F",                    // Fear of harm / avoidance
      formula: "(score - 8) / 6",
      evidence_status: "inferred",
      confidence: "M",
      rationale: "Compulsions subscale captures behavioral output of fear-driven \
                  avoidance and harm-prevention rituals."
    }
  ],
  cell_mappings: [
    {
      scoring_name: "compulsions_subscale",
      cell_id: "ocd.caudate.DA.tone.tone",
      formula: "(score - 12) / 8",
      threshold: 12,
      evidence_status: "inferred",
      rationale: "FDG-PET evidence in OCD compulsions correlates with caudate \
                  metabolic activity above moderate-severity threshold.",
      sources: [/* cited */]
    }
  ],
  ai_extraction_targets: [
    {
      pattern_description: "Patient reports specific contamination concern \
                            (handwashing, household cleaning) — points to D subsystem.",
      cell_ids: ["ocd.ofc.5HT.5HT2A.post-syn", "ocd.acc.5HT.5HT2A.post-syn"],
      example_phrasings: [
        "washes hands until raw",
        "can't touch doorknobs without sanitizer",
        "spends hours cleaning kitchen"
      ],
      evidence_strength_required: "explicit_test"
    }
  ],
  recency_window_days: 7,
  license_status: "free-clinical",
  source_citation: "Goodman et al. 1989, Arch Gen Psychiatry 46:1006-1011",
  notes: "All formula coefficients are starting calibrations. Validation against \
          pilot intake data required before clinical use."
}
```

The actual v1 ElicitationMap library has full content for Y-BOCS, MADRS, PHQ-9, GAD-7, and ASRS. Each follows this shape.

## Status and ground rules

- **All formula coefficients are starting calibrations, not validated truth.** Literature gives directional evidence and rough effect sizes; coefficient values are first-pass and require clinical pilot data to validate. Flagged via `evidence_status: inferred` on subsystem mappings until pilot calibration completes.
- **Subsystem mapping is the default.** Cell-level overrides are reserved for findings with multi-study replication and clear effect direction.
- **No PHI on ElicitationMap pages.** Reference data only.
- **Licensing matters.** Y-BOCS, PHQ-9, GAD-7, ASRS are free for clinical/research use; some instruments (MMSE, WAIS) require licenses and aren't included in v1.

## Open items before v1 → v2

- Finalize MDD, GAD, ADHD canonical subsystem labels (currently working hypotheses).
- Validate all coefficients against pilot intake data — every coefficient is a starting calibration.
- Add second-line OCD instruments (DOCS, OCI-R) for F-subsystem coverage.
- Add YGTSS for tic-spectrum subsystem (T) elicitation.
- Add PCL-5 for PTSD coverage.
- Decide intake pipeline rule for multi-instrument precedence (sketched above; not yet committed in code).
- Add `recency_window_days` field to PatientSubsystemModifier writes from the pipeline.
