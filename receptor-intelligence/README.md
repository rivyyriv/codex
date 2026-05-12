# Receptor Intelligence — Brain Mapping Framework + Platform Master Design

A schema, protocol, engineering spec, and master product design for representing psychiatric disorders, individual patient profiles, and pharmacological treatments as machine-queryable cell-level deltas across brain regions and neurotransmitter systems — and the buildable platform and API that surface this to clinicians.

---

## What this is

Three things in one bundle:

1. **A methodology framework** (`00`–`13`) for representing the brain as a registry of cells, with disorders, patients, and drugs all expressed in the same cell shape.
2. **A master product design** (`14`–`19`) for the buildable platform — the full canonical flow from intake through brain map through treatment fit through AI scribe through follow-up retest.
3. **A practical roadmap** for solo part-time delivery to friendly pilot in ~17 weeks with Claude as primary coding partner.

## Reading order by role

**Founder / product owner orienting:**
1. `README.md` (this file)
2. `14-master-design.md` — vision, decisions, phasing
3. `19-v1-roadmap.md` — week-by-week plan
4. `00-framework-overview.md` — the methodology

**Engineer building the backend:**
1. `14-master-design.md`
2. `01-schema-v3.md` — data shapes
3. `15-schema-extensions.md` — predicted/observed types, FHIR mappings, full-body extensibility
4. `17-backend-stack.md` — concrete Node + Fastify + Supabase build
5. `13-api-endpoint-derivation.md` — endpoint surface
6. `05-visualization-api-payloads.md` — payload contracts

**Engineer building the frontend:**
1. `14-master-design.md`
2. `16-frontend-ux.md` — screens, brain map renderer, scribe UX, component design
3. `06-frontend-primitives-spec.md` — component contracts
4. `05-visualization-api-payloads.md` — what flows into the UI

**Engineer working on AI scribe and extraction:**
1. `07-ai-extraction-spec.md` — methodology
2. `18-structured-ai.md` — concrete prompt architecture, scribe pipeline, code

**Clinical reviewer / advisor:**
1. `00-framework-overview.md`
2. `09-ocd-reference-instantiation.md` — worked example
3. `10-protocol-and-audit-rules.md` — authoring discipline
4. `08-comorbidity-templates.md`

## Full file index

| # | File | What it contains |
|---|------|------------------|
| 0 | `00-framework-overview.md` | Why the framework exists, the core mental model |
| 1 | `01-schema-v3.md` | Full schema for all four tiers |
| 2 | `02-cell-registry-spec.md` | The cell shape — every field, every enum, validation rules |
| 3 | `03-composition-rules.md` | How disorders compose for comorbid profiles |
| 4 | `04-elicitation-maps.md` | Questionnaire → cell/subsystem mappings |
| 5 | `05-visualization-api-payloads.md` | The seven typed payloads the runtime emits |
| 6 | `06-frontend-primitives-spec.md` | React component contracts for each payload |
| 7 | `07-ai-extraction-spec.md` | Clinician note → ProposedModifier methodology |
| 8 | `08-comorbidity-templates.md` | Curated templates for high-prevalence pairs |
| 9 | `09-ocd-reference-instantiation.md` | OCD as a worked example |
| 10 | `10-protocol-and-audit-rules.md` | Authoring protocol + audit queries |
| 11 | `11-readiness-and-blockers.md` | What's ready, what's missing, content gaps |
| 12 | `12-app-architecture-decisions.md` | Six app-architecture decisions and rationales |
| 13 | `13-api-endpoint-derivation.md` | REST/GraphQL surface from payload contracts |
| **14** | **`14-master-design.md`** | **Orchestrating master design — vision, decisions, phasing** |
| **15** | **`15-schema-extensions.md`** | **Predicted/observed deltas, FHIR mappings, full-body extensibility** |
| **16** | **`16-frontend-ux.md`** | **React UX spec — screens, brain map renderer, scribe UX** |
| **17** | **`17-backend-stack.md`** | **Node + Fastify + Supabase implementation, full Postgres DDL** |
| **18** | **`18-structured-ai.md`** | **AI scribe pipeline, Claude integration, prompt templates** |
| **19** | **`19-v1-roadmap.md`** | **Solo part-time week-by-week plan with Claude as coding partner** |

Files `14`–`19` are the master design built atop `00`–`13`. Where they conflict, `14`–`19` are correct because they have all decisions resolved.

## Confirmed product decisions

| Area | v1 Decision |
|------|----|
| Primary user | Psychiatrists / psych NPs first; PCPs in v2 |
| Regulatory framing | CDS-exempt decision support |
| Medication recommendations | Specific drugs ranked, with mechanism reasoning visible |
| Patient-facing | Both pre-visit secure link and in-office tablet |
| Predicted treatment layer | Population average only |
| Anatomy | Brain only; full-body architectural extensibility baked in |
| Build resources | Solo part-time, Claude as primary coding partner |
| Stack | React + Vite + TypeScript + Tailwind; Node + Fastify + Supabase; Anthropic API + Whisper |
| **AI scribe** | **In v1.** Audio capture → Whisper → Claude extraction → clinician-reviewed proposal queue |
| EHR integration | Out of v1; FHIR-shaped data architectures supports future integration |

Full decision matrix in `14-master-design.md`.

## The mental model in 60 seconds

Every disorder, patient, and treatment is a vector over the same set of cells. A cell is one tuple of `(disorder, region, system, target, site)` with a delta from healthy baseline. Healthy baseline is a constant: every cell at zero.

A patient has zero, one, or more `DisorderTemplate` references plus their own modifier cells. Treatments have coverage cells.

Effective patient state at any moment is computed:

```
effective(cell) = baseline(cell)                                   # always 0
                + Σ template_refs.delta(cell) × severity_factor   # active disorders
                + Σ subsystem_modifiers × subsystem_weights[cell] # questionnaire input
                + cell_modifier(cell)                              # individual override
                - Σ active_treatment.coverage(cell)                # treatment effect
```

The residual = effective minus treatment coverage. The residual drives next-line treatment recommendations and explains why current treatment is or isn't enough. v1 surfaces this as a brain heatmap, treatment fit table, predicted post-treatment overlay, and AI scribe with reviewed proposal queue.

## v1 scope — full capable platform

Not an MVP. A real product a psychiatrist can run their practice on:

- Provider auth (single-clinic)
- Patient profiles with multi-disorder support
- Y-BOCS, PHQ-9, GAD-7, MADRS instruments — pre-visit or in-office
- Brain map with multi-layer stacking
- Subsystem heatmap, residual gap list
- Treatment fit table with mechanism reasoning at point of decision
- Predicted post-treatment overlay
- **AI scribe**: audio capture → Whisper transcription → Claude extraction → reviewed proposal queue
- Manual modifier entry (alongside scribe)
- Follow-up retest comparison (predicted vs. observed)
- Audit trail viewer
- Patient-facing summary (post-visit, plain language)
- Patient-facing intake mini-app
- 3 disorder templates (OCD, MDD, GAD), 10 first-line drug coverage profiles

## How to start building

If you're at week 0 of the roadmap:

1. Read `14-master-design.md` end-to-end.
2. Read `19-v1-roadmap.md` end-to-end.
3. Skim `17-backend-stack.md`, `16-frontend-ux.md`, `18-structured-ai.md` for the stack you'll be working in.
4. Begin Phase 0, Week 1: account setup, repo initialization, hello-world deploys.

## Versioning

- **Schema v3** is current core (in `01-schema-v3.md`).
- **Schema v3.1** adds predicted/observed and FHIR/full-body extensions in `15-schema-extensions.md`. Backward-compatible.
- **Master design v1** is in `14`–`19`.
- The architecture supports future EHR integration and SaMD clearance as additive upgrades but those are not the v1 plan.

---

Last updated: 2026-05-02
