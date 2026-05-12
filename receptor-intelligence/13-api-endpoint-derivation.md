# 13 — API Endpoint Derivation

The suggested REST surface for the Receptor Intelligence runtime, derived from the seven typed payloads in `05-visualization-api-payloads.md` and the data shapes in `01-schema-v3.md` and `02-cell-registry-spec.md`.

This is a starting sketch, not a fixed contract. The payload contracts in `05` are the stable interface; transport (REST vs GraphQL vs RPC) is a choice that doesn't change what's computed, only how it's requested.

---

## Design principles

Five principles drive every endpoint shape below.

**Payloads are the unit of response.** Every visualization endpoint returns one of the seven typed payloads from `05`. The front-end has no need for arbitrary registry queries; if a screen needs a different cut of data, it's a new payload type, not a new query parameter.

**Patient state is the unit of identity for compute.** Every visualization is "given this PatientState, render that payload." The PatientState is the lens; the payload is the output. URLs reflect this: `/patients/:id/state/:state_id/payload/<name>`.

**Reads are deterministic and cacheable.** Same `(patient_state, registry_version)` always produces the same payload. The `registry_version` and `patient_state_ref` are part of every payload's metadata. Cache keys can include them.

**Writes are narrow, audited, and explicit.** Modifier writes carry evidence. Profile updates carry timestamps and authors. AI-proposed modifiers go to a queue, never directly to state. Audit log on every mutation.

**Registry is read-mostly, separately authenticated.** Most clients only read the registry. Registry editing is an admin surface, not a clinician surface, and uses different routes.

---

## Resource taxonomy

Six resource families, each backed by schema from `01`:

| Resource | Scope | Backing schema |
|---|---|---|
| `registry` | Cells, rules, templates, elicitation maps | `DisorderCell`, `CompositionRule`, `DisorderTemplate`, `ElicitationMap`, `HealthyBaselineTemplate`, `CuratedComorbidityTemplate` |
| `patients` | Profiles, modifiers, states, regimens | `PatientProfile`, `PatientCellModifier`, `PatientSubsystemModifier`, `PatientState` |
| `intake` | Questionnaire submission, AI-extracted note proposal queue | `ElicitationMap` outputs, `ProposedModifier` |
| `payloads` | The seven typed payloads (always derived, never stored) | `BrainMapPayload`, `SubsystemHeatmapPayload`, `ResidualGapPayload`, `DifferentialDistancePayload`, `TreatmentFitPayload`, `PatientFacingPayload`, `AuditPayload` |
| `audit` | Mutation log, registry version history | derived |
| `auth` | Sessions, MFA, caseload memberships | per Decision 2 in `12-app-architecture-decisions.md` |

URL prefix convention: `/api/v1/<resource>/...`. Version is in the URL because the schema version changes meaningfully and clients need to pin.

---

## Registry endpoints (read-mostly)

The registry endpoints are how the front-end discovers the available disorders, instruments, and composition rules. Cells themselves are rarely fetched directly by the clinician UI — payloads embed the relevant cell snapshots — but the admin UI needs full read/write.

### Read routes

```
GET  /registry/version
GET  /registry/disorders
GET  /registry/disorders/:id
GET  /registry/disorders/:id/cells
GET  /registry/cells/:cell_key
GET  /registry/composition-rules
GET  /registry/composition-rules/:id
GET  /registry/elicitation-maps
GET  /registry/elicitation-maps/:instrument
GET  /registry/comorbidity-templates
GET  /registry/comorbidity-templates/:id
GET  /registry/healthy-baseline
GET  /registry/drug-coverage
GET  /registry/drug-coverage/:agent_id
```

`GET /registry/version` returns:

```json
{
  "registry_version": "2026-04-29.1",
  "schema_version": "3.0",
  "generated_at": "2026-04-29T14:00:00Z"
}
```

Clients pin to a registry version when computing a payload, so they can detect when their cached payload is out of date.

### Write routes (admin only)

```
POST   /registry/cells
PUT    /registry/cells/:cell_key
DELETE /registry/cells/:cell_key            # soft-delete; sets evidence_status
POST   /registry/composition-rules
PUT    /registry/composition-rules/:id
POST   /registry/disorders
PUT    /registry/disorders/:id
POST   /registry/comorbidity-templates
PUT    /registry/comorbidity-templates/:id
POST   /registry/elicitation-maps
PUT    /registry/elicitation-maps/:instrument
POST   /registry/drug-coverage
PUT    /registry/drug-coverage/:agent_id
```

Every write requires reviewer auth and writes an audit log entry. Schema validation runs synchronously and returns 422 with field-level errors on violation. The audit checklist in `10-protocol-and-audit-rules.md` runs as a CI step before PR merge if the registry is in version control, or as a write-time validator if the registry is database-native.

### Bulk export

```
GET /registry/export?since=:registry_version
```

Returns the full registry or the diff since a prior version. Used by the front-end to refresh its local copy and by analytics pipelines.

---

## Patient endpoints

The patient endpoints handle profile creation, modifier writes, and state transitions. Every read is gated by caseload membership (Decision 2) and audited.

### Profile management

```
POST   /patients                            # create profile
GET    /patients/:id
PUT    /patients/:id                        # metadata only; modifiers via separate route
GET    /patients?clinician=me               # caseload list
POST   /patients/:id/template-refs          # add a disorder template ref
DELETE /patients/:id/template-refs/:ref_id  # remove (rare; usually marked rule_out)
PUT    /patients/:id/template-refs/:ref_id  # update severity, diagnosis_status, treatment_status
```

`POST /patients` body:

```json
{
  "demographics": { "age_range": "30-39", "sex_at_birth": "F" },
  "baseline_ref": "healthy_v1",
  "template_refs": [],
  "created_by": "clinician_42"
}
```

Note that `demographics` is intentionally coarse-grained — age range, not date of birth — to limit PHI surface in the payloads. The full DOB lives in the patient identity table elsewhere, not in the profile blob the runtime sees.

### Modifier management

```
POST   /patients/:id/modifiers/cell         # PatientCellModifier (clinician-direct)
PUT    /patients/:id/modifiers/cell/:mod_id
DELETE /patients/:id/modifiers/cell/:mod_id
GET    /patients/:id/modifiers              # list both kinds
```

Cell modifier write body requires evidence:

```json
{
  "cell_key": "ocd|OFC|5HT|5HT2A|post-syn",
  "delta": -1,
  "evidence": {
    "type": "imaging",
    "source_id": "doi:10.xxxx/xxxxx",
    "note": "Patient PET scan 2026-03-15 showing 5HT2A reduction"
  },
  "authored_by": "clinician_42",
  "authored_at": "2026-04-29T14:00:00Z"
}
```

Subsystem modifiers are not written directly via this route — they come from intake (next section).

### State management

`PatientState` is computed from `PatientProfile + active modifiers + active regimen` per the effective-delta formula in `01-schema-v3.md`. It's a snapshot, not directly editable.

```
POST /patients/:id/state                    # snapshot current state, save with timestamp
GET  /patients/:id/state                    # latest snapshot
GET  /patients/:id/state/:state_id          # historical snapshot
GET  /patients/:id/state/history            # state timeline
```

State snapshots are immutable. To "edit" a state, write a new modifier, then snapshot again. The audit story is the snapshot timeline.

### Regimen management

```
POST   /patients/:id/regimen/agents         # add an active treatment
PUT    /patients/:id/regimen/agents/:rx_id  # update dose, status
DELETE /patients/:id/regimen/agents/:rx_id  # discontinue (soft)
GET    /patients/:id/regimen
```

A regimen entry references a `drug_coverage` record by agent_id, plus dose and start/stop dates. The runtime applies coverage when computing residual.

---

## Intake endpoints

The intake routes handle questionnaire ingestion and AI extraction proposals. Per `04-elicitation-maps.md` and `07-ai-extraction-spec.md`.

### Questionnaire submission

```
POST /intake/instruments/:instrument/responses
```

Body:

```json
{
  "patient_id": "p_abc123",
  "instrument": "ybocs",
  "responses": { "obsessions_total": 14, "compulsions_total": 13, "insight": 1 },
  "administered_at": "2026-04-29T13:30:00Z",
  "administered_by": "clinician_42"
}
```

Server applies the relevant `ElicitationMap`, computes `PatientSubsystemModifier`s, writes them, returns a summary plus a fresh state snapshot.

Response:

```json
{
  "subsystem_modifiers_written": [
    { "subsystem": "D", "delta": +1, "confidence": "M", "recency_window_days": 7 },
    { "subsystem": "F", "delta": +0.5, "confidence": "M", "recency_window_days": 7 }
  ],
  "new_state_id": "s_xyz789"
}
```

Instruments supported in v1: `ybocs`, `phq9`, `gad7`, `madrs`, `asrs`. The ElicitationMap registry is the source of truth for what's supported and how each maps.

When multiple instruments are administered, precedence rules from `04-elicitation-maps.md` apply server-side. Higher-confidence source wins; modifiers do not stack.

### AI extraction (v2)

The full spec is in `07-ai-extraction-spec.md`. Endpoint sketch:

```
POST /intake/ai-extraction/notes            # submit a clinical note
GET  /intake/ai-extraction/queue            # admin: review queue
POST /intake/ai-extraction/queue/:id/approve
POST /intake/ai-extraction/queue/:id/reject
```

The submit endpoint returns a job id; the extraction is async. Proposed modifiers land in the queue. Approval moves them to actual modifiers; rejection logs the rejection.

Critical constraint from `07`: an AI proposal never becomes a real modifier without explicit clinician approval. The endpoints enforce this.

---

## Payload endpoints

The seven payload endpoints. Each is a deterministic GET of `(patient_state, optional query)`. URL pattern: `/patients/:id/state/:state_id/<payload>`.

If the client wants the latest state, the convenience pattern `/patients/:id/state/latest/<payload>` resolves at request time.

### 1. BrainMap

```
GET /patients/:id/state/:state_id/brain-map
```

Query params:

| Param | Default | Notes |
|---|---|---|
| `mode` | `effective` | `effective` \| `disorder-only` \| `treatment-only` \| `residual` |
| `region_filter` | none | Comma-separated region codes |
| `system_filter` | none | Comma-separated system codes |

Returns a `BrainMapPayload` (full type in `05-visualization-api-payloads.md`).

### 2. SubsystemHeatmap

```
GET /patients/:id/state/:state_id/subsystem-heatmap
```

Query params:

| Param | Default | Notes |
|---|---|---|
| `subsystems` | all from active templates | Comma-separated subsystem codes |
| `mode` | `effective` | Same modes as brain-map |

Returns a `SubsystemHeatmapPayload`.

### 3. ResidualGap

```
GET /patients/:id/state/:state_id/residual-gap
```

Query params:

| Param | Default | Notes |
|---|---|---|
| `threshold` | `0.5` | Minimum |delta| to include in the gap list |
| `top_n` | none | If set, returns top-N largest gaps |

Returns a `ResidualGapPayload`.

### 4. DifferentialDistance

```
GET /patients/:id/state/:state_id/differential-distance
```

Query params:

| Param | Default | Notes |
|---|---|---|
| `against` | all active disorder templates | Comma-separated template ids |
| `metric` | `cosine` | `cosine` \| `weighted-l2` |

Returns a `DifferentialDistancePayload` with similarity scores. Per Decision 3, the response framing is similarity, not diagnosis.

### 5. TreatmentFit

```
GET /patients/:id/state/:state_id/treatment-fit
```

Query params:

| Param | Default | Notes |
|---|---|---|
| `agents` | all populated | Comma-separated agent ids; if empty, scores all |
| `top_n` | `10` | Top-N agents by fit score |
| `exclude_active_regimen` | `false` | Skip currently-active drugs |
| `exclude_contraindicated` | `true` | Filter agents flagged as contraindicated |

Returns a `TreatmentFitPayload` with mechanism overlap, residual coverage, and ranking. Per Decision 3, framing is "mechanism fit" not "recommendation."

### 6. PatientFacing

```
GET /patients/:id/state/:state_id/patient-facing
```

Query params:

| Param | Default | Notes |
|---|---|---|
| `language` | `en` | i18n; v1 supports en only |
| `reading_level` | `8th` | grade level; affects template selection |

Returns a `PatientFacingPayload`. This is the Layer-3 plain-language summary. Per Decision 3, must never name a specific drug, never give numeric diagnostic confidence.

### 7. Audit

```
GET /patients/:id/state/:state_id/audit
```

Query params:

| Param | Default | Notes |
|---|---|---|
| `since` | profile creation | Filter by timestamp |
| `actor` | none | Filter by clinician/admin id |

Returns an `AuditPayload` showing the chain of modifiers, their evidence, who wrote them, and how they affect effective deltas.

---

## Caching and conditional requests

All payload responses include:

```
ETag: "<sha of patient_state_ref + registry_version>"
Cache-Control: private, max-age=60, must-revalidate
```

Clients send `If-None-Match` on subsequent requests; server returns 304 if the patient state and registry version are unchanged. Cache duration is short (60s) because state can change quickly during an active clinical session.

`registry_version` in the URL of the registry export endpoint enables CDN caching of registry data. Patient payloads cannot be CDN-cached (PHI).

---

## Error model

All errors return JSON of shape:

```json
{
  "error": {
    "code": "REGISTRY_VERSION_MISMATCH",
    "message": "Computation requested against registry_version 2026-04-29.1 but server is on 2026-04-30.1",
    "details": { "requested": "2026-04-29.1", "current": "2026-04-30.1" }
  }
}
```

HTTP status conventions:

- 400: malformed request
- 401: not authenticated
- 403: authenticated but lacking authorization (caseload, role, etc.)
- 404: resource not found (also returned for resources outside caseload, to avoid leaking existence)
- 409: write conflict (e.g., concurrent modifier edit)
- 422: schema validation failed (returns field-level errors)
- 500: internal error
- 503: registry export in progress, retry after a moment

---

## Pagination

List endpoints (`/registry/cells`, `/audit`, `/patients`) use cursor-based pagination:

```
GET /registry/cells?cursor=:opaque&limit=100
```

Response includes `next_cursor`. No offset-based pagination — registry sizes will grow and offsets degrade.

---

## Observability headers

Every request and response includes:

```
X-Registry-Version: 2026-04-29.1
X-Schema-Version: 3.0
X-Request-Id: <uuid>
```

Server-side logs include the patient id (hashed), state id, payload type, and computation time per request. Per Decision 1, logs go to a PHI-aware log sink.

---

## GraphQL alternative

Some teams will prefer GraphQL. The mapping is straightforward:

- Each payload becomes a query: `brainMap(patientId, stateId, mode)`, `subsystemHeatmap(...)`, etc.
- Each registry resource becomes a type with a query and a mutation set.
- Modifier writes become mutations.
- Subscriptions can stream new payload computations as state changes.

The deciding factor: if clients are mostly fetching one payload per screen (which is the dominant case here), REST is simpler. If a screen needs to compose three payloads in one request (which happens for the Layer-1 clinician dashboard), GraphQL is more efficient. A pragmatic compromise: REST for v1, with an optional `/payloads/batch` REST endpoint that accepts a list of payload requests and returns them in one response.

---

## Implementation sequence

A suggested order for an engineer building this surface:

1. **Registry read endpoints** — get the cell registry queryable from the front-end. Unblocks UI exploration with no patient state involved.
2. **Patient profile + state CRUD** — write patient creation, profile read, state snapshot. Without payloads, this is just bookkeeping, but it's the spine.
3. **Effective-delta computation library** — the function from `(PatientProfile, regimen)` to `effective(cell)`. Pure function, easy to test against the formula in `01-schema-v3.md`. Heavy on unit tests.
4. **First payload: BrainMap** — the simplest. Implement the generator, wire to endpoint. Front-end can render.
5. **SubsystemHeatmap, ResidualGap** — straightforward extensions of the same machinery.
6. **Intake endpoint for Y-BOCS** — proves the questionnaire → modifier → state pipeline end-to-end.
7. **TreatmentFit** — depends on drug coverage cells (Blocker #1 in `11-readiness-and-blockers.md`).
8. **DifferentialDistance** — depends on multiple disorder templates (Blocker #2).
9. **PatientFacing** — separate template-rendering layer; can be developed in parallel with payloads 1-3.
10. **Audit** — last because it depends on the audit log infrastructure being in place.
11. **AI extraction routes** — v2 feature per `07-ai-extraction-spec.md`.

Steps 1-6 are the v1 critical path. Steps 7-10 land as the content gaps close.

---

## What this doc deliberately doesn't specify

- **Authentication transport.** Bearer tokens vs cookies vs mTLS — that's a Decision 2 outcome.
- **Rate limiting.** Necessary in production; orthogonal to the API shape.
- **Webhooks.** Not required for v1. If needed for partner integrations later, add a webhook resource.
- **Bulk patient import.** Not in v1. If hospital partner pilots require it, add `/patients/bulk` with idempotency.
- **The actual Postgres DDL.** Derive from the JSON Schema noted in `01-schema-v3.md`. The DDL is implementation, not interface.

---

## Cross-references

- `01-schema-v3.md` — the data shapes referenced in every body and response
- `05-visualization-api-payloads.md` — the payload contracts that drive every payload endpoint
- `04-elicitation-maps.md` — what the intake endpoints transform questionnaire responses into
- `07-ai-extraction-spec.md` — full spec for the AI proposal queue endpoints
- `10-protocol-and-audit-rules.md` — the audit checks that validate registry writes
- `12-app-architecture-decisions.md` — Decisions 1, 2, and 4 directly shape the auth and storage layers below this API
