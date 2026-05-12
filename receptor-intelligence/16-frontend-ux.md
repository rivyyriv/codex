# 16 — Frontend UX Specification

The React platform UX, screen by screen. Composes the primitive component contracts from `06-frontend-primitives-spec.md` into the actual screens a clinician uses. Separate from the patient-facing intake mini-app, also specified here.

The frontend is one of three deliverables in v1: backend (`17-backend-stack.md`), frontend (this file), and clinical content (drug coverage cells + 3 disorder templates).

---

## Stack

| Concern | Choice | Why |
|---------|--------|-----|
| Framework | React 18 + Vite | Fast dev server, minimal config, ubiquitous |
| Language | TypeScript (strict) | Type safety against the schema; compiler catches half your bugs |
| Styling | Tailwind CSS | No CSS files, fast iteration, ergonomic at solo scale |
| Component library | shadcn/ui | Headless, customizable, owned-not-imported (copy components into your repo) |
| Charts | Recharts | Decent SVG charts; for brain map use custom SVG |
| Forms | react-hook-form + zod | Schema-driven validation; aligns naturally with the platform schema |
| Routing | React Router v6 | Stable, well-documented |
| State | Zustand for global, react-query for server state | Avoid Redux ceremony at solo scale |
| API client | Generated TS types from OpenAPI spec | Single source of truth for endpoint signatures; covered in `17` |
| Brain rendering | Custom SVG (hex grid) | Avoids licensing of anatomical assets, scales to full-body |
| Auth | Supabase JS client | Matches backend auth choice |
| Hosting | Vercel | Optimal for Vite + React, free tier sufficient for v1 |
| Testing | Vitest + React Testing Library | Vite-native, fast |
| E2E | Playwright | Optional in v1 |

The whole stack is solo-friendly and free at v1 traffic levels.

## Architecture

```
src/
├── api/                  # Generated client + react-query hooks
│   ├── client.ts         # axios/fetch wrapper with auth interceptor
│   ├── generated/        # OpenAPI-generated types
│   └── hooks/            # one hook per endpoint family
│       ├── usePatient.ts
│       ├── usePayloads.ts
│       ├── useIntake.ts
│       └── useRegistry.ts
├── components/
│   ├── ui/               # shadcn primitives (button, card, dialog, etc.)
│   ├── brainmap/         # the centerpiece visualization
│   │   ├── BrainMap.tsx
│   │   ├── HexCell.tsx
│   │   ├── HexGrid.tsx
│   │   ├── ColorScale.tsx
│   │   └── LayerStack.tsx
│   ├── intake/
│   │   ├── QuestionnaireRunner.tsx
│   │   ├── YBOCSPanel.tsx
│   │   ├── PHQ9Panel.tsx
│   │   ├── GAD7Panel.tsx
│   │   └── MADRSPanel.tsx
│   ├── treatment/
│   │   ├── TreatmentFitTable.tsx
│   │   ├── PredictedLayerToggle.tsx
│   │   └── PrescribingDialog.tsx
│   ├── scribe/
│   │   ├── ScribeRecorder.tsx        # MediaRecorder wrapper with start/pause/end
│   │   ├── ScribeStatus.tsx          # red-dot + elapsed timer
│   │   ├── ProposalQueue.tsx         # post-visit review list
│   │   ├── ProposalCard.tsx          # single proposal with approve/edit/reject
│   │   └── TranscriptPanel.tsx       # collapsible transcript with quote highlights
│   └── audit/
│       ├── AuditTimeline.tsx
│       └── EvidenceTooltip.tsx
├── pages/                # Route-level screens
│   ├── auth/
│   ├── dashboard/
│   ├── patients/
│   │   ├── PatientList.tsx
│   │   ├── NewPatient.tsx
│   │   ├── PatientDetail.tsx       # The big one
│   │   └── VisitFlow.tsx
│   ├── patient-intake/             # Patient-facing mini-app
│   │   └── PreVisitQuestionnaire.tsx
│   └── admin/                      # Registry editing (later)
├── lib/
│   ├── deltaUtils.ts               # color/scale helpers
│   ├── fhir.ts                     # FHIR adapter (no-op in v1, hook in v2)
│   └── auth.ts
└── types/
    └── schema.ts                   # mirrors backend schema
```

## Screen catalog

Six top-level screens in v1, organized by user journey.

### 1. Login / landing

Standard email/password + MFA prompt for clinicians. Provider lands on dashboard; admin role lands on admin dashboard.

Patients reach the platform via a pre-visit secure link; they don't have accounts in v1. The link includes a one-time JWT scoped to a single questionnaire submission.

### 2. Dashboard

Provider's home screen.

Sections:
- **Today's schedule** — upcoming visits with patient name, time, last-completed questionnaire (if pre-visit administered).
- **Recent patients** — last 10 patients seen, sorted by last activity.
- **Pending actions** — patients with completed pre-visit questionnaires not yet reviewed; predicted-vs-observed comparisons ready (follow-up visits with retest data).
- **Quick search** — patient name autocomplete.

Visual: dense, information-rich. Practitioners triage from this screen. No empty states for power users.

### 3. New patient flow

Multi-step modal:

1. **Demographics** — name, DOB, sex at birth, preferred pronouns, MRN if available.
2. **Initial assessment context** — chief complaint (free text), provisional diagnosis selection from dropdown of active disorder templates (OCD, MDD, GAD in v1), severity sense ("subclinical / mild / moderate / severe" — sets initial severity bucket per `01-schema-v3.md`).
3. **Initial questionnaire** — pick which instrument to administer. Defaults based on provisional diagnosis. Provider can administer in-office on tablet OR send a secure pre-visit link to the patient's phone.
4. **Confirm** — review summary, create patient.

After creation, lands on the patient detail screen.

### 4. Patient detail (the central screen)

The screen the clinician spends most time on. Multiple panels arranged for at-a-glance assessment.

```
┌─────────────────────────────────────────────────────────────┐
│ [Patient name]  [DOB]  [MRN]              [⚙ Settings]      │
│ Active disorders: OCD (severe, confirmed)                   │
│ Active regimen: Sertraline 50mg (12 days)                   │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│   ┌──────────────────────────┐  ┌─────────────────────────┐ │
│   │                          │  │ Subsystem heatmap       │ │
│   │   BRAIN MAP              │  │  D ▮▮▮▮▮▮▮ +2.4         │ │
│   │   (hex grid, color =     │  │  F ▮▮▮▮▮ +1.8           │ │
│   │    delta from baseline)  │  │  H ▮▮▮ +0.9             │ │
│   │                          │  │  T ▮ +0.2               │ │
│   │   Layers:                │  │                         │ │
│   │   ☑ Effective state      │  │ Residual gap            │ │
│   │   ☐ Disorder only        │  │  OFC 5HT2A: -1.5        │ │
│   │   ☐ Treatment coverage   │  │  Caud D2: +1.8          │ │
│   │   ☑ Predicted post-Rx    │  │  vS D2: +0.6            │ │
│   │                          │  │  ...                    │ │
│   └──────────────────────────┘  └─────────────────────────┘ │
│                                                              │
│   ┌─────────────────────────────────────────────────────┐   │
│   │ Treatment fit (mechanism overlap with residual)     │   │
│   │ ─────────────────────────────────────────────────── │   │
│   │ Sertraline 50mg          ████████████░░░  78%      │   │
│   │   Targets: OFC 5HT2A, dlPFC 5HT2C, ...              │   │
│   │   [ Prescribe ▸ ]                                   │   │
│   │ ─────────────────────────────────────────────────── │   │
│   │ Aripiprazole 5mg         ████████░░░░░░░  62%      │   │
│   │   Targets: vS D2, Putamen DA tone (OCD+TS-rel)      │   │
│   │   [ Prescribe ▸ ]                                   │   │
│   │ ─────────────────────────────────────────────────── │   │
│   │ ...                                                 │   │
│   └─────────────────────────────────────────────────────┘   │
│                                                              │
│   [+ New questionnaire]   [+ Note]   [▶ Start visit]        │
└─────────────────────────────────────────────────────────────┘
```

Notes on this layout:

- The brain map is the headline visual; large enough to read region/cell labels.
- Layer toggles let provider see effective state, disorder-attributable, treatment coverage, residual, and predicted post-Rx independently or stacked.
- Subsystem heatmap is a smaller secondary view — bar chart of subsystem-level deltas (D/F/H/T for OCD).
- Residual gap is a list, sorted by `|delta|` descending. Each entry is clickable → highlights the corresponding cell on the brain map.
- Treatment fit table sorts by mechanism overlap percentage. Each row shows the targeted cells; hovering shows the evidence behind those targets — this is where the CDS exemption "independent review of the basis" gets satisfied.
- Footer actions: new questionnaire (administer additional or different instrument), new note (free text + AI summary in v2), start visit (begins encounter).

### 5. Visit flow

When the provider clicks "Start visit," the screen enters visit mode. Persistent timer in the corner. Encounter record is created.

In-visit panels:

- **Scribe panel** — record/pause/end controls plus a recording-state indicator (red dot, elapsed time). Audio is captured via MediaRecorder, chunked-uploaded to the backend continuously. At end of visit, the transcript is generated and proposals are extracted (Pattern 1 in `18-structured-ai.md`).
- **Notes panel** — provider types observations during the conversation. Textarea. Notes get fed alongside the transcript into modifier extraction.
- **Modifier proposals panel** — appears at end of visit once extraction completes. Each proposal shows the verbatim transcript quote, subsystem/cell, magnitude, confidence. Provider approves / edits / rejects each one. Approved modifiers write through and the brain map updates live. Manual modifier entry is also available for things the scribe missed (autocomplete by region/target, delta input, evidence text required per `07-ai-extraction-spec.md`).
- **Prescribing dialog** — when "Prescribe ▸" is clicked on a treatment row:
  1. Confirm agent + dose (provider can adjust dose; predicted layer recomputes live).
  2. Show predicted brain map alongside current.
  3. Confirm prescribing date and expected evaluation date (defaults to +6 weeks).
  4. Optional notes.
  5. Confirm → backend creates `TreatmentPrediction` record (frozen snapshot per `15-schema-extensions.md`), updates regimen, returns updated state.

After ending the visit (button: "End visit") the encounter is sealed, audit log captures all changes, and a Layer-3 patient-facing summary is generated.

### 6. Follow-up retest

When a patient with an active prediction returns and retakes the questionnaire, the platform detects the matching `TreatmentPrediction` and surfaces a comparison view:

```
Predicted vs. Observed — sertraline 50mg, 6 weeks elapsed
─────────────────────────────────────────────────────────
Cell                    Predicted       Observed     Error
OFC 5HT2A (post-syn)    -2 → -0.5       -2 → -0.7    +0.2
Caudate D2 (post-syn)   +2 → +1         +2 → +1.5    +0.5
vS D2 (post-syn)        +1 → 0          +1 → +0.4    +0.4
...

Overall mean error: +0.3 (slightly under-coverage)

[ Confirm and store ▸ ]   [ Adjust regimen ▸ ]
```

Provider reviews, confirms storage of the (`TreatmentPrediction`, `ObservedOutcome`) pair. The pair becomes a row in `prediction_outcome_pairs` table; later aggregated into `PredictionAccuracyAggregate`.

If the provider adjusts regimen at this visit, a new `TreatmentPrediction` is created for the new agent.

### 7. Patient-facing intake (mini-app, separate route)

Routed under `/intake/:token` — token is a one-time JWT included in the secure link sent to the patient.

Three screens:

1. **Welcome** — "Your provider [Name] has asked you to complete a brief assessment. This will take 5-10 minutes. Your responses are private and shared only with your provider."
2. **Questionnaire** — single instrument, one question per screen (mobile-friendly), progress bar, no back-navigation lock (they can correct).
3. **Done** — "Thanks! Your provider will review your responses at your visit on [date]." No clinical interpretation shown to the patient at this stage — that's a Layer-3 summary, post-visit.

Stack is the same React app, separate route tree, no auth (token-scoped). Mobile-first responsive.

## The brain map renderer in detail

The single most important visualization. Worth specifying carefully.

### Approach: hex grid by region

Rather than anatomically accurate brain SVG (licensing pain, hard to author), use a hex grid where each hex represents one anatomical region. Multiple hexes can group into "lobes" or "systems" via color-bordering.

For OCD-relevant regions, a layout might be:

```
        ┌───────────┐
        │    PFC    │   (mPFC, dlPFC clustered)
        │  ⬡ ⬡ ⬡  │
        └─────┬─────┘
   ┌──────────┴──────────┐
   │    OFC     ACC      │
   │     ⬡       ⬡       │
   │                     │
   │   Caudate  Putamen  │
   │     ⬡       ⬡       │
   │                     │
   │      vS    Amyg     │
   │       ⬡     ⬡       │
   │                     │
   │  Hippo  Raphe  VTA  │
   │   ⬡      ⬡    ⬡    │
   └─────────────────────┘
```

Each hex's color encodes the maximum-magnitude delta across all cells in that region (or a chosen aggregate like mean). Click a hex to drill into per-system, per-target deltas for that region.

Color scale:

- 0 (no deviation): neutral gray
- Positive delta (excess function): graduated blue, saturated at +3
- Negative delta (reduced function): graduated red, saturated at -3
- "No data" cells: white with diagonal hatch pattern (distinct from zero)
- "Inferred" cells (vs evidenced): same color but with reduced opacity or a subtle dashed border

### Layer system

Multiple deltas per cell exist depending on what's being viewed:

- **Effective**: `effective(cell)` per the schema-v3 formula. Default view.
- **Disorder-only**: just the disorder template's contribution, no modifiers, no treatment.
- **Treatment coverage**: `Σ active_treatment.coverage(cell)` — what's being addressed.
- **Residual**: effective minus treatment coverage = the gap.
- **Predicted post-treatment**: effective + drug coverage of a hypothetical or active treatment at a chosen dose.

Each layer has a checkbox; layers stack via alpha blending (semi-transparent hexes overlay). Toggle individual layers on/off.

### Drill-down

Click any hex → opens a side panel showing:
- Region name and anatomical context
- All cells in that region (system + target + site + delta)
- For each cell: evidence sources, confidence, last reviewed
- Modifier history for this region for this patient

This is where the CDS independent-review requirement is most visibly satisfied.

### Implementation sketch

```typescript
interface BrainMapProps {
  payload: BrainMapPayload;            // from API
  layers: LayerConfig[];               // user-toggled layers
  onCellClick?: (cellKey: string) => void;
  onRegionClick?: (region: string) => void;
}

interface LayerConfig {
  id: "effective" | "disorder-only" | "coverage" | "residual" | "predicted";
  enabled: boolean;
  opacity: number;
}

// Renders to <svg viewBox="0 0 800 600">
//   <HexGrid regions={...}>
//     {layers.map(layer => 
//       <LayerStack key={layer.id} layer={layer} cells={payload.cells} />
//     )}
//   </HexGrid>
//   <ColorScale />
// </svg>
```

The renderer is a function of `(payload, layer config)` to SVG. No imperative state. Reproducible: same payload + same config → same SVG. Useful for screenshots, audit, and SaMD validation.

### Animation between states

When a provider toggles "predicted post-treatment" layer, hex colors animate from current to predicted. ~600ms ease, no overshoot. The animation is decorative; underlying values are the truth and rendered in the side panel without animation.

### Full-body extension

Per `15-schema-extensions.md`, regions carry `rendering_hints.map_layer: "brain" | "body"`. v3 adds a body view tab next to the brain view; same renderer, different region set, different hex layout. No code changes to the renderer.

## Component contract recap

Tying back to `06-frontend-primitives-spec.md`:

| Primitive | This UX spec implements as |
|-----------|----------------------------|
| `BrainMapRenderer` | `BrainMap` component above |
| `SubsystemHeatmap` | Bar chart in patient detail right column |
| `ResidualGapList` | List in patient detail right column |
| `TreatmentFitTable` | Table below brain map |
| `DifferentialDistanceRanking` | Not in v1 (needs more disorder templates) |
| `PatientFacingSummary` | Layer-3 panel post-visit, also separate mini-app screen |
| `AuditTrail` | Audit timeline in patient settings |

The primitives have stable contracts; UI screens compose them.

## Accessibility

- Color-only encoding fails for color-blind users. Hex shapes carry text labels for value (`+2`, `-1.5`) at any zoom level. Pattern (dot vs line vs solid) is an alternative encoding.
- All interactions keyboard-accessible. Tab order follows visual flow.
- ARIA labels on hexes describe region and value.
- Min font size 14px.
- WCAG AA contrast ratios.

## Performance

The brain map at v1 has ~50 cells across ~12 regions. Trivial render time. Concerns become real at v3 full-body scale (potentially 500+ regions). The renderer should virtualize at that scale, but v1 doesn't need to.

## Mobile

The clinician UI is desktop-first (psychiatrists use laptops in office). Mobile is a v3 nice-to-have. The patient-facing intake mini-app is mobile-first.

## What's not specified here

Theme/dark mode, branding, marketing site, billing UI. Those are product/design decisions outside the master design.

## Cross-references

- `06-frontend-primitives-spec.md` — the component contracts
- `05-visualization-api-payloads.md` — the data shapes that flow into these components
- `13-api-endpoint-derivation.md` — endpoints called by react-query hooks
- `15-schema-extensions.md` — predicted/observed types referenced by visit flow and follow-up screens
