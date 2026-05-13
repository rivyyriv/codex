# 17 — Backend Stack and Implementation

The Node.js backend implementing the API surface defined in `13-api-endpoint-derivation.md` against the schema in `01-schema-v3.md` and `15-schema-extensions.md`. Concrete enough to start coding.

---

## Stack

| Concern | Choice | Why |
|---------|--------|-----|
| Runtime | Node.js 20 LTS | Mature, broad ecosystem, fits Fastify and Supabase JS |
| Framework | Fastify | Faster than Express, built-in JSON schema validation, smaller surface area |
| Language | TypeScript (strict, `noImplicitAny`) | Type safety end-to-end |
| Validation | zod (request/response) + ajv (Fastify schema) | zod for app logic, ajv for fast wire validation |
| Database | Supabase (Postgres 15 + PgBouncer + RLS) | One service for db + auth + storage |
| Migrations | Supabase CLI migrations / drizzle-kit | Version-controlled DDL |
| ORM | drizzle-orm | TS-first, generated types match schema; lighter than Prisma at solo scale |
| Auth | Supabase Auth (JWT) | Verified server-side via JWKS; same provider as frontend |
| Background jobs | pg-boss (Postgres-backed) | No additional infra; survives restarts; sufficient for v1 throughput |
| AI integration | Anthropic SDK (`@anthropic-ai/sdk`) | Per `18-structured-ai.md` |
| Logging | pino (built into Fastify) | Structured JSON logs, fast |
| Observability | Sentry (error tracking) | Free tier sufficient for v1 |
| Hosting | Fly.io or Railway | Single-region single-machine fine for v1; both have generous free/cheap tiers |
| Testing | Vitest + supertest | Fast, TS-native |

Solo-friendly choices throughout: minimal services, all managed where possible, ergonomic at small scale.

## Project structure

```
backend/
├── src/
│   ├── server.ts              # Fastify bootstrap, plugin registration
│   ├── env.ts                 # zod-validated env vars
│   ├── plugins/
│   │   ├── auth.ts            # Supabase JWT verification
│   │   ├── audit.ts           # audit log middleware
│   │   ├── error.ts           # error handling
│   │   └── cors.ts
│   ├── modules/
│   │   ├── registry/          # Cells, rules, templates, elicitation maps
│   │   │   ├── routes.ts
│   │   │   ├── service.ts
│   │   │   ├── repository.ts
│   │   │   └── schema.ts
│   │   ├── patients/          # Profiles, modifiers, states, regimens
│   │   ├── intake/            # Questionnaire submission
│   │   ├── scribe/            # Audio upload, transcript storage, proposal queue
│   │   ├── payloads/          # The 7 typed payloads
│   │   ├── prediction/        # TreatmentPrediction + ObservedOutcome
│   │   ├── audit/             # Audit log queries
│   │   └── adverse-events/    # AE reporting
│   ├── lib/
│   │   ├── effective-delta.ts # The schema-v3 effective(cell) computation
│   │   ├── composition.ts     # Composition rules application
│   │   ├── elicitation.ts     # Questionnaire → modifiers
│   │   ├── ai/                # Anthropic + OpenAI integration
│   │   │   ├── client.ts
│   │   │   ├── extractModifiers.ts
│   │   │   ├── patientSummary.ts
│   │   │   └── intakeAssistant.ts
│   │   ├── payloads/          # One generator per payload type
│   │   │   ├── brainMap.ts
│   │   │   ├── subsystemHeatmap.ts
│   │   │   ├── residualGap.ts
│   │   │   ├── differentialDistance.ts
│   │   │   ├── treatmentFit.ts
│   │   │   ├── patientFacing.ts
│   │   │   └── audit.ts
│   │   └── fhir.ts            # FHIR adapter (no-op v1, hook in v2)
│   ├── jobs/
│   │   ├── transcribeEncounter.ts        # Whisper transcription
│   │   ├── extractModifiers.ts           # Claude extraction
│   │   ├── computePredictionAccuracy.ts  # nightly aggregation
│   │   └── snapshotPatientStates.ts
│   └── db/
│       ├── schema.ts          # drizzle schema
│       └── migrations/        # versioned SQL
├── package.json
├── tsconfig.json
└── drizzle.config.ts
```

Module-per-resource-family keeps boundaries clean. Each module owns its routes, service logic, repository (DB access), and validation schemas.

## Postgres schema

Two top-level schemas in one Supabase project: `registry` (no PHI) and `phi` (encrypted, RLS-protected).

### Registry schema (no PHI)

```sql
-- Source-of-truth registry, edited via admin UI
SCHEMA registry;

CREATE TABLE registry.disorder_templates (
  id text PRIMARY KEY,                    -- "ocd_canonical_v2"
  schema_version text NOT NULL,            -- "3.0"
  name text NOT NULL,
  description text,
  registry_version text NOT NULL,
  active boolean NOT NULL DEFAULT true,
  created_at timestamptz NOT NULL DEFAULT now(),
  metadata jsonb
);

CREATE TABLE registry.disorder_cells (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  cell_key text NOT NULL,                  -- "ocd|OFC|5HT|5HT2A|post-syn"
  template_id text NOT NULL REFERENCES registry.disorder_templates(id),
  region text NOT NULL,
  system text NOT NULL,
  target text,
  site text NOT NULL,
  baseline_delta numeric NOT NULL,
  subsystem_weights jsonb NOT NULL,        -- { D: 0.6, F: 0.3, H: 0.1, T: 0.0 }
  evidence_status text NOT NULL,           -- 'evidenced' | 'inferred' | 'no-data' | 'not-applicable'
  confidence text NOT NULL,                -- 'H' | 'M' | 'L'
  contested text,                          -- null | 'methodological' | 'subtype' | 'state-trait'
  sources jsonb NOT NULL,                  -- [{id, type, tier, year, n}]
  last_reviewed timestamptz,
  reviewer text,
  notes text,
  active_in_registry boolean NOT NULL DEFAULT true,
  registry_version text NOT NULL,
  UNIQUE (cell_key, template_id, registry_version)
);

CREATE INDEX idx_cells_template ON registry.disorder_cells (template_id);
CREATE INDEX idx_cells_region_system ON registry.disorder_cells (region, system);

CREATE TABLE registry.regions (
  code text PRIMARY KEY,                   -- "OFC", "thyroid", etc.
  name text NOT NULL,
  anatomical_system text NOT NULL,         -- "CNS" | "endocrine" | ...
  parent_region text REFERENCES registry.regions(code),
  rendering_hints jsonb,                   -- { map_layer, hex_position, ... }
  active_in_registry boolean NOT NULL DEFAULT true
);

CREATE TABLE registry.systems (
  code text PRIMARY KEY,
  name text NOT NULL,
  category text NOT NULL,                  -- 'neurotransmitter' | 'hormone' | ...
  has_targets boolean NOT NULL DEFAULT true,
  has_sites boolean NOT NULL DEFAULT true
);

CREATE TABLE registry.composition_rules (
  id text PRIMARY KEY,
  scope jsonb NOT NULL,                    -- which disorder pair, which cells/subsystems
  interaction_type text NOT NULL,          -- 'additive' | 'multiplicative' | 'ceiling' | 'novel' | 'unknown'
  rule_data jsonb NOT NULL,
  evidence jsonb NOT NULL,
  active boolean NOT NULL DEFAULT true,
  registry_version text NOT NULL
);

CREATE TABLE registry.elicitation_maps (
  instrument text PRIMARY KEY,             -- "ybocs", "phq9", etc.
  loinc_code text,                         -- For FHIR adapter
  schema_version text NOT NULL,
  mappings jsonb NOT NULL,                 -- the question→subsystem mappings
  recency_window_days int NOT NULL,
  registry_version text NOT NULL
);

CREATE TABLE registry.drug_coverage (
  agent_id text NOT NULL,                  -- "sertraline"
  dose_bucket text NOT NULL,               -- "low" | "standard" | "high"
  cell_key text NOT NULL,
  coverage_delta numeric NOT NULL,
  confidence text NOT NULL,
  sources jsonb NOT NULL,
  registry_version text NOT NULL,
  PRIMARY KEY (agent_id, dose_bucket, cell_key, registry_version)
);

CREATE TABLE registry.healthy_baseline (
  cell_key text PRIMARY KEY,
  baseline_value numeric NOT NULL DEFAULT 0,
  evidence_status text NOT NULL,
  registry_version text NOT NULL
);

CREATE TABLE registry.versions (
  version text PRIMARY KEY,                -- "2026-04-29.1"
  schema_version text NOT NULL,
  generated_at timestamptz NOT NULL,
  notes text
);
```

### PHI schema (RLS-protected)

```sql
SCHEMA phi;

CREATE TABLE phi.clinicians (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  supabase_user_id uuid UNIQUE NOT NULL,   -- Links to auth.users
  npi text,                                 -- National Provider Identifier
  name text NOT NULL,
  email text NOT NULL,
  role text NOT NULL,                      -- 'clinician' | 'admin' | 'reviewer'
  mfa_enrolled boolean NOT NULL DEFAULT false,
  created_at timestamptz NOT NULL DEFAULT now()
);

CREATE TABLE phi.patients (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  external_id text,                        -- For FHIR sync
  birth_year int NOT NULL,                 -- Coarse for PHI minimization
  sex_at_birth text,
  pronouns text,
  preferred_language text DEFAULT 'en',
  baseline_ref text NOT NULL DEFAULT 'healthy_v1',
  created_by uuid REFERENCES phi.clinicians(id),
  created_at timestamptz NOT NULL DEFAULT now(),
  archived_at timestamptz
);

CREATE TABLE phi.caseload_memberships (
  clinician_id uuid REFERENCES phi.clinicians(id),
  patient_id uuid REFERENCES phi.patients(id),
  granted_at timestamptz NOT NULL DEFAULT now(),
  granted_by uuid REFERENCES phi.clinicians(id),
  revoked_at timestamptz,
  PRIMARY KEY (clinician_id, patient_id)
);

CREATE TABLE phi.template_refs (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  patient_id uuid REFERENCES phi.patients(id),
  template_id text NOT NULL,
  template_version_pin text NOT NULL,      -- per Decision 5: pin-by-default
  diagnosis_status text NOT NULL,          -- 'undifferentiated' | 'provisional' | 'confirmed' | 'rule_out'
  severity_bucket text NOT NULL,           -- 'subclinical' | 'mild' | 'moderate' | 'severe' | 'chronic_severe'
  treatment_status text NOT NULL,          -- 'untreated' | 'undertreated' | 'on-treatment' | 'remitted'
  added_at timestamptz NOT NULL DEFAULT now(),
  added_by uuid REFERENCES phi.clinicians(id)
);

CREATE TABLE phi.encounters (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  patient_id uuid REFERENCES phi.patients(id),
  clinician_id uuid REFERENCES phi.clinicians(id),
  started_at timestamptz NOT NULL,
  ended_at timestamptz,
  encounter_type text NOT NULL,            -- 'initial' | 'follow-up' | 'retest'
  notes text                               -- Encrypted at rest
);

CREATE TABLE phi.questionnaire_responses (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  patient_id uuid REFERENCES phi.patients(id),
  encounter_id uuid REFERENCES phi.encounters(id),
  instrument text NOT NULL,
  loinc_code text,                         -- For FHIR adapter
  responses jsonb NOT NULL,
  administered_at timestamptz NOT NULL,
  administered_by uuid REFERENCES phi.clinicians(id),
  administration_mode text NOT NULL        -- 'in-office' | 'pre-visit-link'
);

CREATE TABLE phi.subsystem_modifiers (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  patient_id uuid REFERENCES phi.patients(id),
  source_response_id uuid REFERENCES phi.questionnaire_responses(id),
  subsystem text NOT NULL,                 -- 'D' | 'F' | 'H' | 'T' | etc.
  delta numeric NOT NULL,
  confidence text NOT NULL,
  recency_window_days int,
  active boolean NOT NULL DEFAULT true,
  superseded_by uuid REFERENCES phi.subsystem_modifiers(id),
  created_at timestamptz NOT NULL DEFAULT now()
);

CREATE TABLE phi.cell_modifiers (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  patient_id uuid REFERENCES phi.patients(id),
  cell_key text NOT NULL,
  delta numeric NOT NULL,
  evidence jsonb NOT NULL,                 -- { type, source_id, note }
  authored_by uuid REFERENCES phi.clinicians(id),
  authored_at timestamptz NOT NULL DEFAULT now(),
  active boolean NOT NULL DEFAULT true,
  superseded_by uuid REFERENCES phi.cell_modifiers(id)
);

CREATE TABLE phi.regimen_agents (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  patient_id uuid REFERENCES phi.patients(id),
  agent_id text NOT NULL,                  -- references registry.drug_coverage(agent_id)
  dose text NOT NULL,                      -- "50mg daily"
  dose_bucket text NOT NULL,
  started_at timestamptz NOT NULL,
  stopped_at timestamptz,
  prescribed_by uuid REFERENCES phi.clinicians(id),
  notes text
);

CREATE TABLE phi.patient_states (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  patient_id uuid REFERENCES phi.patients(id),
  computed_at timestamptz NOT NULL DEFAULT now(),
  registry_version text NOT NULL,
  effective_deltas jsonb NOT NULL,         -- { cell_key: delta, ... }
  active_modifier_ids uuid[],
  active_regimen_ids uuid[],
  triggering_event_type text,              -- 'questionnaire' | 'modifier-write' | 'regimen-change'
  triggering_event_id uuid
);

CREATE INDEX idx_states_patient_time ON phi.patient_states (patient_id, computed_at DESC);

CREATE TABLE phi.treatment_predictions (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  patient_id uuid REFERENCES phi.patients(id),
  patient_state_ref uuid REFERENCES phi.patient_states(id),
  prescribed_agent_id text NOT NULL,
  prescribed_dose text NOT NULL,
  prescribed_at timestamptz NOT NULL,
  expected_evaluation_at timestamptz NOT NULL,
  prescribing_clinician_id uuid REFERENCES phi.clinicians(id),
  predicted_deltas jsonb NOT NULL,
  registry_version text NOT NULL,
  notes text
);

CREATE TABLE phi.observed_outcomes (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  prediction_id uuid REFERENCES phi.treatment_predictions(id),
  patient_id uuid REFERENCES phi.patients(id),
  observed_at timestamptz NOT NULL,
  observation_state_ref uuid REFERENCES phi.patient_states(id),
  observed_deltas jsonb NOT NULL,
  treatment_adherence text,
  side_effects_reported text[],
  treatment_changed boolean NOT NULL DEFAULT false,
  notes text
);

CREATE TABLE phi.adverse_events (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  patient_id uuid REFERENCES phi.patients(id),
  patient_state_ref uuid REFERENCES phi.patient_states(id),
  reported_by uuid REFERENCES phi.clinicians(id),
  reported_at timestamptz NOT NULL DEFAULT now(),
  event_type text NOT NULL,
  severity text NOT NULL,
  outcome text NOT NULL,
  related_treatment_id uuid REFERENCES phi.regimen_agents(id),
  related_prediction_id uuid REFERENCES phi.treatment_predictions(id),
  description text NOT NULL,
  reporter_attributed_to_platform boolean NOT NULL DEFAULT false,
  notes text
);

CREATE TABLE phi.audit_log (
  id bigserial PRIMARY KEY,
  occurred_at timestamptz NOT NULL DEFAULT now(),
  actor_id uuid REFERENCES phi.clinicians(id),
  actor_role text NOT NULL,
  action text NOT NULL,                    -- 'create' | 'update' | 'delete' | 'read' | 'login' | ...
  resource_type text NOT NULL,
  resource_id text NOT NULL,
  patient_id uuid REFERENCES phi.patients(id),
  request_id text,
  ip_address inet,
  user_agent text,
  before_state jsonb,
  after_state jsonb,
  notes text
);

CREATE INDEX idx_audit_patient_time ON phi.audit_log (patient_id, occurred_at DESC);
CREATE INDEX idx_audit_actor_time ON phi.audit_log (actor_id, occurred_at DESC);
```

### Row-level security

PHI tables are read-protected at the row level. A clinician can only read patient rows they have caseload membership for.

```sql
ALTER TABLE phi.patients ENABLE ROW LEVEL SECURITY;

CREATE POLICY caseload_read ON phi.patients
  FOR SELECT
  USING (
    EXISTS (
      SELECT 1 FROM phi.caseload_memberships m
      JOIN phi.clinicians c ON c.id = m.clinician_id
      WHERE m.patient_id = phi.patients.id
        AND c.supabase_user_id = auth.uid()
        AND m.revoked_at IS NULL
    )
  );

-- Similar policies on every PHI table joined through patient_id.
```

Admin role bypasses RLS for registry edits (separate policy). Audit log writes happen with elevated privilege via a database function.

## Effective-delta computation library

The pure function at the heart of the system. From `01-schema-v3.md`:

```typescript
effective(cell) = baseline(cell)                          // always 0 from healthy_v1
                + Σ template_refs.delta(cell) × severity  // active disorders
                + Σ subsystem_modifiers × subsystem_weights[cell]
                + cell_modifier(cell)                     // individual override
                - Σ active_treatment.coverage(cell)       // treatment effect
```

Implementation:

```typescript
// src/lib/effective-delta.ts
import type { 
  PatientProfile, RegimenAgent, ActiveModifiers, 
  RegistrySnapshot, SeverityBucket, CompositionRule
} from '../types/schema';

const SEVERITY_FACTOR: Record<SeverityBucket, number> = {
  subclinical: 0.25,
  mild: 0.5,
  moderate: 1.0,
  severe: 1.5,
  chronic_severe: 2.0,
};

interface EffectiveDeltaInput {
  profile: PatientProfile;
  templateRefs: TemplateRef[];        // from phi.template_refs
  subsystemModifiers: SubsystemModifier[];
  cellModifiers: CellModifier[];
  activeRegimen: RegimenAgent[];
  registry: RegistrySnapshot;          // pinned to a registry_version
}

export function computeEffectiveDeltas(input: EffectiveDeltaInput): Map<string, number> {
  const result = new Map<string, number>();
  
  // For each template_ref, accumulate base disorder contribution
  for (const ref of input.templateRefs) {
    if (ref.diagnosis_status === 'rule_out') continue;
    const cells = input.registry.cellsByTemplate.get(ref.template_id);
    if (!cells) continue;
    const sev = SEVERITY_FACTOR[ref.severity_bucket];
    for (const cell of cells) {
      const current = result.get(cell.cell_key) ?? 0;
      result.set(cell.cell_key, current + cell.baseline_delta * sev);
    }
  }
  
  // Apply composition rules (multiplicative, ceiling, novel, unknown)
  // Default behavior is additive — already handled above
  applyCompositionRules(result, input.templateRefs, input.registry.compositionRules);
  
  // Apply subsystem modifiers via per-cell subsystem_weights
  for (const mod of input.subsystemModifiers) {
    if (!mod.active) continue;
    // For every cell that has a non-zero weight for this subsystem
    for (const cell of input.registry.allCells) {
      const w = cell.subsystem_weights[mod.subsystem] ?? 0;
      if (w === 0) continue;
      const current = result.get(cell.cell_key) ?? 0;
      result.set(cell.cell_key, current + mod.delta * w);
    }
  }
  
  // Apply cell-level modifiers (clinician-direct overrides)
  for (const mod of input.cellModifiers) {
    if (!mod.active) continue;
    const current = result.get(mod.cell_key) ?? 0;
    result.set(mod.cell_key, current + mod.delta);
  }
  
  // Subtract active treatment coverage
  for (const agent of input.activeRegimen) {
    if (agent.stopped_at && new Date(agent.stopped_at) < new Date()) continue;
    const coverage = input.registry.drugCoverage.get(`${agent.agent_id}|${agent.dose_bucket}`);
    if (!coverage) continue;
    for (const [cellKey, deltaCoverage] of coverage) {
      const current = result.get(cellKey) ?? 0;
      result.set(cellKey, current - deltaCoverage);
    }
  }
  
  return result;
}
```

This function is the soul of the system. Heavily unit-tested. It's pure: same inputs always produce the same output. No clock, no IO, no randomness.

## Endpoint implementation pattern

Each endpoint follows this pattern:

```typescript
// src/modules/payloads/routes.ts
import type { FastifyInstance } from 'fastify';
import { z } from 'zod';
import { brainMapSchema, brainMapQuery } from './schema';
import { generateBrainMap } from '../../lib/payloads/brainMap';
import { loadPatientStateForRequest } from './service';

export async function payloadRoutes(app: FastifyInstance) {
  app.get<{
    Params: { id: string; stateId: string };
    Querystring: z.infer<typeof brainMapQuery>;
  }>('/patients/:id/state/:stateId/brain-map', {
    schema: {
      params: { type: 'object', properties: { id: { type: 'string' }, stateId: { type: 'string' } }, required: ['id', 'stateId'] },
      querystring: brainMapQuery.shape,
      response: { 200: brainMapSchema },
    },
    preHandler: [app.requireAuth, app.requireCaseloadAccess('id')],
  }, async (req, reply) => {
    const { id, stateId } = req.params;
    const query = req.query;
    const { state, registry } = await loadPatientStateForRequest({ patientId: id, stateId });
    const payload = generateBrainMap({ state, registry, query });
    return payload;  // Fastify validates against response schema
  });
}
```

Key elements:

- Zod schemas for input validation (then converted to JSON schema for Fastify).
- `preHandler` middleware enforces auth + caseload access.
- Service layer loads data, library generates payload deterministically.
- Response schema validation guarantees clients get well-shaped data.

## Audit middleware

Every PHI write produces an audit log entry. Implemented as a Fastify hook on routes that mutate.

```typescript
// src/plugins/audit.ts
app.addHook('onResponse', async (request, reply) => {
  if (!request.routeOptions.config?.auditable) return;
  if (reply.statusCode >= 400) return;  // Don't audit errors
  
  await db.insert(audit_log).values({
    actor_id: request.user.id,
    actor_role: request.user.role,
    action: routeToAction(request.method, request.url),
    resource_type: request.routeOptions.config.resourceType,
    resource_id: extractResourceId(request, reply),
    patient_id: request.params.id ?? null,
    request_id: request.id,
    ip_address: request.ip,
    user_agent: request.headers['user-agent'],
    before_state: request.auditBefore,    // Set by route handler if needed
    after_state: request.auditAfter,
  });
});
```

Routes opt in via `config: { auditable: true, resourceType: 'patient' }`. Before/after states captured by handlers when relevant for diff display in audit timeline.

## Background jobs

pg-boss for queue-backed jobs. Jobs are Postgres rows; workers consume them. Survives restarts. Sufficient throughput for v1.

Two scheduled jobs in v1:

```typescript
// src/jobs/computePredictionAccuracy.ts
// Runs nightly, computes PredictionAccuracyAggregate per (cell_key, agent_id, dose_bucket)
// from all (TreatmentPrediction, ObservedOutcome) pairs older than 6 weeks.

// src/jobs/snapshotPatientStates.ts
// Optional: pre-compute PatientState snapshots for active patients to reduce request latency.
```

Other jobs added in v2:

- AI extraction processing (Whisper transcription → Claude extraction → proposal queue).
- FHIR sync (push/pull deltas to/from connected EHRs).

## Testing strategy

**Unit tests**: every function in `src/lib/`. The effective-delta computation in particular needs ~50 unit tests covering:
- Simple single-disorder cases
- Multi-disorder additive composition
- Multi-disorder with composition rules (ceiling, multiplicative, novel)
- Subsystem modifier propagation
- Cell modifier override
- Treatment coverage subtraction
- Edge cases (zero deltas, missing cells, deprecated cells, ruled-out templates)

**Integration tests**: Fastify app with test database (real Postgres in Docker, not mocks). Cover each endpoint family.

**E2E**: skip in v1 unless team grows.

## Local development

```bash
# Spin up Postgres + Supabase locally
supabase start

# Apply migrations
npm run db:migrate

# Seed registry with OCD canonical template + healthy_v1
npm run db:seed

# Run backend in watch mode
npm run dev

# Run tests
npm test
```

## Production deployment

v1 single-region single-machine on Fly.io or Railway:

- Single Fastify process (Node 20).
- Supabase managed Postgres (also has connection pooling, RLS enforcement).
- Sentry for errors.
- Vercel for frontend.
- Domain via your registrar; HTTPS via the host.

Backups: Supabase automatic daily, with 7-day retention on free tier (upgrade as needed).

Cost at v1 scale: <$50/month for everything.

## Migration to scale

When v1 outgrows single-machine:

- Add Fastify replicas behind a load balancer (Fly.io supports this natively).
- Promote Supabase Postgres tier.
- Add a managed Redis (Upstash) for session cache and pg-boss replacement.
- Move PHI to a separate Supabase project per `12-app-architecture-decisions.md` Decision 1.

None of these require app rewrites. The architecture supports horizontal scaling because the API is stateless and the heavy logic is pure functions.

## Cross-references

- `01-schema-v3.md` — schema this DDL implements
- `15-schema-extensions.md` — predicted/observed types in `phi.treatment_predictions`, `phi.observed_outcomes`
- `13-api-endpoint-derivation.md` — endpoints implemented here
- `05-visualization-api-payloads.md` — payload contracts the generators produce
- `18-structured-ai.md` — AI integration (v2)
- `19-v1-roadmap.md` — when each module gets built
