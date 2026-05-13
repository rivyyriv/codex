# 06 — Front-End Primitives — Engineering Spec

The React component contracts that consume the typed payloads from `05-visualization-api-payloads.md`. The primitives are the API surface for rendering — composed into clinician, patient, and audit UIs.

---

## Scope

**What this covers:** the front-end primitives, their props, their behavior contracts, and how they compose into UI surfaces.

**What's deferred:** specific charting library (Recharts vs D3 vs Plotly), state management (Redux vs Zustand vs Context), accessibility patterns, internationalization. These are downstream engineering decisions; the contracts here are stable across them.

**Audience:** the front-end engineer building the clinician and patient interfaces.

## Key invariant

The front-end never reaches into the registry directly. All visualization is a function of typed payloads. Every payload is reproducible from the same `(PatientState, query)` inputs. This makes screenshots auditable and bugs debuggable.

## Primitive list

One primitive per payload type. Built as React components with the payload as the only required prop.

### `<BrainMap />`

Renders the per-region/per-receptor brain map.

```typescript
interface BrainMapProps {
  payload: BrainMapPayload;
  projection?: "flat" | "anatomical-2d" | "anatomical-3d" | "hex-grid";
  colorBy?: "effective_delta" | "residual" | "confidence";
  onCellClick?: (cell: BrainMapCell) => void;
  highlightCellIds?: string[];
}
```

**Behavior contract:**

- Default projection: `flat` (region grid, simplest implementation).
- Cell click opens a detail panel showing all schema fields including sources.
- Color scale uses `legend.delta_scale` and `legend.subsystem_colors` from the payload — component does not invent its own colors.
- Cells where `applicable: false` render as a different visual state (gray/dashed) regardless of delta.
- Cells where `evidence_status: no-data` render with reduced opacity.
- Contested cells (`contested != null`) get a marker (icon, hatching) regardless of magnitude.
- Tooltip shows: cell_id, target, effective_delta, residual, confidence, top sources.

**Do not:**

- Store or fetch registry data — only consume the payload.
- Round delta values for display before payload generation — the payload's `effective_delta` is what renders, even if continuous internally.

### `<SubsystemHeatmap />`

Region × subsystem heatmap.

```typescript
interface SubsystemHeatmapProps {
  payload: SubsystemHeatmapPayload;
  orientation?: "region-rows" | "subsystem-rows";
  showAggregateBar?: boolean;
  onCellClick?: (region: string, subsystem: string) => void;
}
```

**Behavior:**

- Region-rows is default.
- Aggregate bar at edge shows subsystem totals across all regions.
- Click drills into the contributing cells for that (region, subsystem) intersection.
- Cells with magnitude 0 render empty, not at minimum color.

### `<ResidualGapView />`

The "what's not covered" view. Drives next-line treatment thinking.

```typescript
interface ResidualGapViewProps {
  payload: ResidualGapPayload;
  groupBy?: "subsystem" | "region";
  showCandidateAgents?: boolean;
  onAgentSelect?: (agentId: string) => void;
}
```

**Behavior:**

- Default: group by subsystem (clinically more useful for treatment planning).
- Each gap row shows: cell, residual magnitude, candidate agents.
- Candidate agent selection bubbles up via `onAgentSelect` — parent screen handles "preview adding this agent."
- Cells with residual below threshold are filtered by the payload generator — the primitive renders what's given.

### `<DifferentialDistanceRanking />`

For undifferentiated profiles: rank candidate disorders by distance.

```typescript
interface DifferentialDistanceRankingProps {
  payload: DifferentialDistancePayload;
  showDistinguishingCells?: boolean;
  onTemplateSelect?: (templateId: string) => void;
}
```

**Behavior:**

- Templates rank by similarity score (1.0 = identical).
- Top 3 templates show distinguishing cells inline; rest expand on click.
- Selecting a template bubbles up — parent handles "set this as a provisional template_ref."

### `<TreatmentFitTable />`

Candidate agents ranked by predicted residual reduction.

```typescript
interface TreatmentFitTableProps {
  payload: TreatmentFitPayload;
  showMechanismSummary?: boolean;
  onAgentSelect?: (agentId: string) => void;
}
```

**Behavior:**

- Sort by ranking_rationale's score (residual reduction × confidence × evidence weight).
- Show contraindications inline as warnings.
- Expand row → mechanism summary, evidence sources, expected coverage by subsystem.
- Never display prescribing instructions. The component surfaces fit; the clinician converts that into a prescription.

### `<PatientFacingSummary />`

Layer 3 plain language. Patient-facing.

```typescript
interface PatientFacingSummaryProps {
  payload: PatientFacingPayload;
}
```

**Behavior:**

- Renders Most Affected Systems, Treatment Summary, Next Steps, Glossary.
- Glossary terms inline link to glossary entries on hover/tap.
- No technical neuroanatomy jargon. No cell IDs. No tier/confidence indicators.
- Layer-3 templates are reviewed by a clinical advisor before delivery — the primitive renders what's given without mediation.

### `<AuditDashboard />`

Internal-facing. Registry health.

```typescript
interface AuditDashboardProps {
  payload: AuditPayload;
  groupBy?: "rule" | "severity" | "cell_id";
  showSummaryCards?: boolean;
}
```

**Behavior:**

- Default group: severity (high → low).
- Each failure row: cell_id, rule, current value, expected, suggested fix.
- Staleness panel separate.

## Composition into UI surfaces

Three main surfaces compose the primitives. Each is a route in the application.

### Clinician intake surface

```
IntakeScreen
  ├─ QuestionnaireForm (out of scope here — covered by intake pipeline)
  ├─ <BrainMap payload={brainMap} colorBy="effective_delta" />
  ├─ <SubsystemHeatmap payload={heatmap} />
  └─ <DifferentialDistanceRanking payload={differential} />
      // shown only when diagnosis_status === 'undifferentiated'
```

### Clinician treatment surface

```
TreatmentScreen
  ├─ RegimenEditor (out of scope here)
  ├─ <BrainMap payload={brainMap} colorBy="residual" />
  ├─ <ResidualGapView payload={residualGap} groupBy="subsystem" />
  └─ <TreatmentFitTable payload={treatmentFit} />
```

### Patient surface

```
PatientScreen
  └─ <PatientFacingSummary payload={patientFacing} />
```

### Internal audit surface

```
AuditScreen
  └─ <AuditDashboard payload={audit} />
```

## Implementation sequence

Order by user value. Build top to bottom.

1. **Payload contracts package** (`@receptor-intelligence/contracts`) — just the TypeScript types from `05-visualization-api-payloads.md`. Pure data, zero dependencies. Locks the contract before any rendering work.
2. **Mock payload generator** — hardcoded fixtures matching the contracts, derived from the OCD canonical template. Lets front-end work begin before the runtime is built.
3. **`<BrainMap />`** with flat projection only. Highest user-value primitive. Anatomical projections are nice-to-have.
4. **`<SubsystemHeatmap />`** — small, useful, validates the payload generator.
5. **`<ResidualGapView />` and `<TreatmentFitTable />`** — these unlock the "pick next-line drug" workflow which is goal #2 from the project priorities.
6. **`<DifferentialDistanceRanking />`** — unlocks the diagnostic workflow for undifferentiated profiles.
7. **`<PatientFacingSummary />`** — the Layer 3 deliverable. Last to build because it depends on Layer 0–2 stability.
8. **`<AuditDashboard />`** — internal tool, can lag.

## Open implementation questions

Decisions deferred to engineering, not blocking the contract:

- **Charting library.** Recharts is light; D3 is flexible; Plotly is heavy. BrainMap may justify D3 for SVG control; heatmap is simple enough for any.
- **Anatomical projection assets.** 3D brain meshes (Three.js + brain atlas) vs 2D anatomical SVG vs hex-grid simplification. Hex grid is fastest and surprisingly readable.
- **State management.** Redux is overkill. Zustand or React Query for server state, Context for theme/user.
- **Rendering performance.** ~100 cells × a few visualizations is small; no virtualization needed. If the cell count grows past ~500, revisit.
- **Animation.** Should delta changes between PatientState snapshots animate? Useful for showing treatment trajectory. Defer to v2.
- **Mobile.** All primitives should be responsive; complex projections may degrade gracefully to flat on small screens. Specific breakpoints TBD.

## Acceptance criteria

A primitive is complete when:

1. It accepts the payload type as the only required prop.
2. It renders correctly against the mock payload generator's fixtures.
3. It does not fetch data, mutate the registry, or hold registry state.
4. Color scales, labels, and thresholds come from the payload, not hardcoded.
5. It handles `applicable: false`, `evidence_status: no-data`, and `contested != null` cells per the behavior contract.
6. It exports a Storybook story per significant state.

## What this spec does NOT define

- Authentication or PHI handling — architecture concern, separate spec.
- Backend API endpoints — the runtime exposes payload generators; HTTP/GraphQL transport is downstream.
- Database connection — the runtime reads from the registry export; live database connection is not required.
- The intake form UI — questionnaire rendering and AI extraction review queue are out of scope.
- Notion integration — the registry export from Notion is a build step, not a runtime concern.
