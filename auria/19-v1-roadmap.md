# 19 — v1 Roadmap

The week-by-week build plan for a fully capable v1 with AI scribe, at solo part-time pace with Claude as primary coding partner.

This is the practical "what do I do tomorrow morning" document. Read after `14-master-design.md`.

---

## Pace assumptions

**Effort budget**: 10-15 hours/week of focused engineering, accounting for context-switching cost from a day job.

**Multiplier with Claude as coding partner**: ~2.5x weighted across the project. Code generation 3-5x; debugging 1.5-2x; clinical content authoring ~1.2x (literature review can't compress as much); UX iteration 1.2x (real users can't be rushed).

**Realistic calendar**: ~16-20 weeks (4-5 months) to a fully capable v1 ready for friendly pilot. Each phase below is sized accordingly. Track actual hours in the first phase to recalibrate; the multiplier is a starting estimate.

If life intervenes (it will) and the pace drops, double the calendar. The work is sequenced so any given week's deliverable is meaningful even if the next week slips.

---

## Phases at a glance

| Phase | Weeks | Focus | Outputs |
|-------|-------|-------|---------|
| 0 — Setup | 1 | Infra, repos, auth, dev environment | Backend running locally, deployable, auth flow |
| 1 — Foundation | 2-3 | Postgres schema, registry seeding, patient CRUD | OCD template + healthy_v1 in DB, patient creation |
| 2 — Computation | 4-5 | Effective-delta library, Y-BOCS intake, payload generators | Backend computes patient state from questionnaire |
| 3 — Frontend foundation | 6-8 | React app, brain map renderer, patient detail page | UI renders BrainMapPayload from backend |
| 4 — Visit flow | 9-11 | Treatment fit, prescribing, predicted layer, retest comparison | Full canonical visit flow without scribe |
| 5 — AI scribe | 12-14 | Audio capture, Whisper, Claude extraction, proposal queue | Working scribe end-to-end |
| 6 — Polish + content | 15-17 | Clinical review of 8 AI-drafted templates, dose-band drug coverage, supplements + brain types + undifferentiated path, scribe partial-salvage UI, auto-approve tuning | Production-ready for friendly pilot |
| 7 — Pilot | 18+ | Onboard 1-3 psychiatrists, iterate from feedback | Real-world validation |

Total to friendly pilot: ~17 weeks (4 months) of build, then pilot is open-ended. The actual launch trigger is "build done + first willing provider lined up," which can happen earlier or later than week 17 depending on pilot recruitment in parallel.

---

## Phase 0 — Setup (Week 1)

**Goal**: infrastructure ready to receive code.

- Create accounts: Supabase (free tier), Vercel (free), Fly.io or Railway, Sentry (free), GitHub.
- Buy domain. Configure DNS.
- Initialize monorepo: `pnpm` workspaces with `apps/backend` and `apps/frontend`.
- Backend: Fastify + TypeScript + Drizzle + Vitest. Hello-world endpoint.
- Frontend: Vite + React 18 + TypeScript + Tailwind + shadcn/ui CLI. Hello-world page.
- Configure Supabase Auth with email/password + TOTP MFA.
- JWT verification middleware in Fastify (verify against Supabase JWKS).
- Audit log middleware skeleton.
- Sentry SDK both projects.
- CI: lint + typecheck + test on every PR.
- Deploy both: backend to Fly.io/Railway, frontend to Vercel. Confirm production environment works.

**End of Phase 0:** you can log in, your IP is in the audit log, the deploy pipeline works.

---

## Phase 1 — Foundation (Weeks 2-3)

**Goal**: registry exists in Postgres, you can create a patient profile.

### Week 2: schema migrations

- Full DDL from `17-backend-stack.md`: `001_registry_schema.sql`, `002_phi_schema.sql`.
- RLS policies on every PHI table.
- Drizzle schema files matching the SQL; generate TS types via `drizzle-kit`.
- Seed scripts:
  - `registry.regions` for v1 brain regions (OFC, ACC, dlPFC, vmPFC, mPFC, Caudate, Putamen, vS, Amygdala, Hippocampus, Raphe, VTA, LC).
  - `registry.systems` (5HT, DA, NE, GABA, Glu, ACh).
  - `registry.healthy_baseline` (every cell at 0).
  - OCD canonical template (~53 cells from `09-ocd-reference-instantiation.md`).

### Week 3: clinician + patient CRUD + registry reads

- `src/modules/patients`: create, get, list, update.
- Caseload membership model (RLS enforced).
- Endpoint to add `template_ref` to a patient.
- `src/modules/registry`: read endpoints for `disorders`, `cells`, `regions`, `systems`, `healthy-baseline`.
- ETag + cache-control headers.
- Frontend: basic patient list page, can create a patient via UI.

**End of Phase 1**: patient creation works in browser; OCD template attached; data shows up in DB.

---

## Phase 2 — Computation (Weeks 4-5)

**Goal**: when a patient takes Y-BOCS, the backend computes their effective-delta state and you can fetch it.

### Week 4: effective-delta library + Y-BOCS

- `src/lib/effective-delta.ts` — pure function from inputs to deltas per `17-backend-stack.md`. Heavily unit-tested (~50 tests covering simple, multi-disorder, composition rules, modifiers, treatment subtraction, edge cases).
- Y-BOCS elicitation map seeded into `registry.elicitation_maps`.
- `POST /intake/instruments/ybocs/responses` endpoint: stores responses, computes subsystem modifiers, snapshots a new `patient_state`, returns state ID.

### Week 5: payload generators

- `src/lib/payloads/brainMap.ts`, `subsystemHeatmap.ts`, `residualGap.ts`.
- Each generator: pure function from `(patient_state, registry)` to typed payload per `05-visualization-api-payloads.md`.
- Endpoints: `GET /patients/:id/state/:stateId/brain-map`, etc.
- Unit tests against canned patient states.
- Integration smoke: create patient → submit Y-BOCS → fetch brain map → assert shape.
- ETag-based caching for payload endpoints.

**End of Phase 2**: curl your API, get a valid BrainMapPayload back.

---

## Phase 3 — Frontend foundation (Weeks 6-8)

**Goal**: brain map renders in the browser from real backend data.

### Week 6: app skeleton + react-query setup

- Routes: `/login`, `/dashboard`, `/patients`, `/patients/:id`.
- Generated TS types from backend OpenAPI (or hand-written).
- React Query hooks: `usePatient`, `useBrainMap`, `useSubsystemHeatmap`, `useResidualGap`.
- Auth context: hold session, attach JWT to API calls.

### Week 7: brain map renderer (SVG hex grid)

- `src/components/brainmap/HexGrid.tsx` — hex layout per `16-frontend-ux.md`.
- `HexCell.tsx` — color-encoded hex.
- `ColorScale.tsx` — legend.
- Click → side panel with cell-level detail.
- Test against canned BrainMapPayload fixtures.

### Week 8: layer system + patient detail polish

- Layer toggles (effective / disorder-only / coverage / residual / predicted), stacking via SVG groups with alpha.
- Animation between layer changes.
- Subsystem heatmap (Recharts bar chart).
- Residual gap list with click-to-highlight.
- Patient detail layout per `16-frontend-ux.md` wireframe. Loading/empty/error states.

**End of Phase 3**: open a patient in browser, see full brain map, navigate the layers.

---

## Phase 4 — Visit flow (Weeks 9-11)

**Goal**: full canonical patient flow works end-to-end (without scribe yet).

### Week 9: drug coverage data + treatment fit

- Author drug coverage cells for sertraline, fluoxetine, escitalopram (3 SSRIs). Literature review + cell-level entries with source citations. This is content work; budget the time honestly.
- `TreatmentFitPayload` generator + `GET /patients/:id/state/:stateId/treatment-fit` endpoint.
- Frontend: `TreatmentFitTable` component below brain map.

### Week 10: prescribing + predicted layer

- `PrescribingDialog` UI: select agent + dose, see predicted brain map preview.
- Backend: `POST /patients/:id/regimen/agents` creates regimen entry, freezes `TreatmentPrediction`.
- New patient state snapshot includes treatment coverage.
- Predicted layer toggle on brain map renderer.

### Week 11: follow-up retest

- Patient retakes Y-BOCS at follow-up.
- Backend detects matching `TreatmentPrediction` (within expected_evaluation_at window ± 2 weeks).
- Generates predicted-vs-observed comparison.
- Frontend: comparison view per `16-frontend-ux.md` follow-up screen.
- Provider confirms → `ObservedOutcome` written.

**End of Phase 4**: full visit flow works end-to-end. Manual modifier entry only; scribe comes next.

---

## Phase 5 — AI scribe (Weeks 12-14)

**Goal**: provider clicks "Start scribe," has a conversation, gets a reviewed proposal queue at the end.

### Week 12: audio capture + transcription pipeline

- Frontend: MediaRecorder API to capture visit audio. UI: Start/Pause/End buttons during visit.
- Backend: chunked upload endpoint accepting audio segments.
- OpenAI Whisper API integration (HIPAA tier with BAA).
- `phi.encounter_transcripts` table per `15-schema-extensions.md` (encrypted at rest).
- End-of-visit hook: full transcript saved, linked to encounter.

### Week 13: Claude extraction + proposal queue

- Implement modifier extraction pattern from `18-structured-ai.md` Pattern 2.
- Substring guard for evidence quotes (anti-hallucination).
- `phi.modifier_proposals` table (queue with `pending` / `approved` / `rejected` / `edited` statuses).
- Background job: on encounter end, transcribe → extract → enqueue proposals.
- `phi.ai_call_log` writes for every Claude call.

### Week 14: proposal review UI

- Frontend: review queue surfaces at end of visit.
- Each proposal: subsystem or cell, magnitude/direction, evidence quote highlighted in transcript context, confidence.
- Actions per proposal: Approve (writes modifier), Edit (adjust magnitude/cell before approving), Reject (logged with optional reason).
- Live brain map update as proposals are approved.
- Audit log captures every action.

**End of Phase 5**: working scribe. Provider has a 30-min visit, audio is captured, transcript and proposals are ready by the time they end the visit.

---

## Phase 6 — Polish + content (Weeks 15-17)

**Goal**: ready for pilot. Clinical review of the expanded disorder set, dose-banded drug coverage, supplements alongside drugs, brain types layer, undifferentiated patient flow, scribe partial-salvage and auto-approve tuning, patient-facing intake mini-app.

### Week 15: clinical review of AI-drafted templates + dose-band drug coverage

- Walk a clinical reviewer through the 8 AI-drafted disorder templates (`22`–`29`: MDD, GAD, Panic Disorder, Social Anxiety, PTSD, ADHD, Insomnia, Adjustment Disorder). Capture corrections in versioned template diffs; nothing ships unreviewed.
- Apply subtype_overrides per `30-v1.1-schema-extensions.md` (PTSD dissociative; MDD melancholic / atypical).
- Finalize drug coverage authoring with discrete dose bands (low / medium / high) for ~10 agents: sertraline, fluoxetine, escitalopram, venlafaxine, duloxetine, bupropion, lamotrigine, aripiprazole, clonidine, NAC. One TreatmentCoverage row per agent×band.
- PHQ-9, GAD-7, MADRS, and the additional instruments tied to the expanded set, elicitation maps and intake routes.
- Composition tests across the new templates.

### Week 16: supplements + brain types + undifferentiated path + patient-facing intake

- Treatment fit table: render supplements alongside drugs, ranked together, with evidence-tier badges (A / B / C). One unified ranking surface.
- Brain types layer (`20-brain-types.md`): assignment flow from intake signals, 6-archetype display on patient detail, and the **lifestyle + supplements recommendations** panel driven by the assigned type. Brain type is a stable trait, not a drug-ranking input.
- Undifferentiated patient flow: "No diagnosis yet — assess only" intake option (`16-frontend-ux.md`). Patient progresses through assessment without a `template_ref`; provider assigns one later, or keeps the patient in assessment-only mode.
- Patient-facing intake mini-app at `/intake/:token`. Token-scoped auth, mobile-first responsive, single instrument per session.
- Provider-side: "send pre-visit link" generates and emails the token (Resend or AWS SES).

### Week 17: scribe partial-salvage, auto-approve tuning, pre-pilot QA

- Scribe partial-salvage UI: when transcription or extraction fails on a span, commit the surviving segments and render the gap inline with timestamped markers (`[gap 12:04–14:30 — transcription unavailable]`). Provider can manually fill or accept the gap.
- Auto-approve threshold tuning surface: per-provider slider/setting for the confidence threshold (default 0.85). Tuning events are logged. High-stakes actions always queue regardless of threshold (see `18-structured-ai.md`).
- Layer-3 patient-facing summary generator (Pattern 2 from `18-structured-ai.md`). Drafted by AI; **provider reviews and edits before sending**. Forbidden-term checks remain enforced.
- Audit timeline UI in patient settings.
- Browser testing (Chrome, Safari, Firefox).
- Mobile device testing for patient-facing intake.
- Security review: every endpoint authed, every PHI table RLS'd, no PHI in logs.
- Identify clinical advisor candidate. Draft BAA template.

**End of Phase 6**: platform is pilot-ready.

---

## Phase 7 — Pilot (Weeks 18+)

**Goal**: real psychiatrists use the platform on real patients with informed consent and CDS-exempt framing.

### Onboarding cadence

- Week 18-19: first provider — recruit, onboard, walk through the platform, set expectations. They start with 2-3 willing patients. Daily check-ins for first week.
- Weeks 20-23: iterate from feedback. Bug fixes prioritized by patient-facing severity. UX adjustments based on usage. Resist scope creep.
- Week 24+: second + third provider if first pilot is successful.

The pilot phase is open-ended. The launch trigger to consider v2 work is: ~10+ patients across 2-3 providers with at least one full prediction-vs-observation cycle completed, providing real signal on what's used and what's not.

---

## Done-criteria for v1 launch

Before declaring v1 done and starting pilot:

- [ ] Healthy baseline template populated.
- [ ] All 9 disorder templates populated and clinically reviewed (OCD, MDD, GAD, Panic Disorder, Social Anxiety, PTSD, ADHD, Insomnia, Adjustment Disorder).
- [ ] 10 drug coverage profiles populated with source citations, in discrete dose bands (low / med / high).
- [ ] Supplements integrated into the treatment fit table alongside drugs, with A/B/C evidence-tier badges.
- [ ] Brain types layer implemented: 6 archetypes, assignment flow, lifestyle + supplements recommendations panel.
- [ ] Undifferentiated patient flow works end-to-end (intake without `template_ref`, later assignment).
- [ ] Y-BOCS, PHQ-9, GAD-7, MADRS and the instruments for the expanded disorder set operational end-to-end.
- [ ] Brain map renders correctly across 5+ patient scenarios.
- [ ] Treatment fit table shows mechanism reasoning at point of decision.
- [ ] Predicted post-treatment overlay works.
- [ ] AI scribe captures audio, transcribes, extracts proposals, tiered review queue.
- [ ] Auto-approve at 0.85 confidence works (tunable per provider; high-stakes actions always queue); tuning events logged.
- [ ] Scribe partial-salvage failure mode works (surviving segments commit; timestamped gap markers).
- [ ] Patient-facing summary is AI-drafted and provider-reviewed before sending.
- [ ] Proposal review UI: approve / edit / reject all functional, with audit log entries.
- [ ] Follow-up retest comparison works.
- [ ] RLS policies on every PHI table; tested with two clinician users not seeing each other's patients.
- [ ] Patient-facing intake mini-app works on iOS Safari and Android Chrome.
- [ ] Sentry capturing errors in production.
- [ ] Backups configured and tested (restore from a snapshot at least once).
- [ ] CDS-exempt framing in UI: "decision support" language, every recommendation shows its basis.
- [ ] Privacy policy + terms of service drafted (counsel-reviewed if affordable).
- [ ] BAA template drafted for pilot providers.

---

## Risk register

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| Effective-delta computation has subtle bug | Medium | Critical | Heavy unit testing in Phase 2; manual sanity checks against authored OCD reference |
| Drug coverage cell authoring takes 2-3x estimated time | Medium | High | Start with 3 SSRIs in Week 9; reassess pace before committing to all 10 |
| AI scribe extraction quality is poor | Medium | Medium | Proposal queue is the safety net — provider rejects bad proposals, no harm done. Iterate prompts based on real transcripts. |
| Whisper transcription is poor in noisy clinic | Medium | Medium | Test in actual clinic environment in Phase 5; consider better mic recommendations to providers |
| Brain map UX confuses providers | Medium | High | Quick informal user-test with 1-2 psychiatrist friends in Phase 3 before committing to layout |
| Patient-facing summary generates inappropriate language | Low | Critical | Forbidden-term checks per `18-structured-ai.md`; opt-in only |
| RLS policy gap exposes PHI | Low | Critical | Security review at Week 17; pen-test before pilot |
| Pilot psychiatrist disengages | High | Medium | Recruit 2 from start; weekly check-ins |
| Burnout from solo part-time pace | High | Critical | Buffer weeks built into roadmap; strict cap on hours; the 2.5x Claude multiplier means progress per hour is high — preserve that by not over-investing in any single week |
| Predictions wildly mis-calibrated against early observations | Medium | Medium | Expected — that's the data signal you're collecting. Track per-cell error patterns |
| Anthropic / OpenAI API pricing changes | Low | Low | Token usage is small; route low-stakes calls to cheaper models |

---

## What's deliberately not on the roadmap

Things you might be tempted to add but should resist in v1:

- **EHR integration.** Capital and certification cost. The architecture supports it; building it is a v2 decision.
- **Public API as developer product.** No external consumers yet; defer the developer portal work.
- **DifferentialDistance ranking.** Even with 9 v1 templates the calibration data isn't yet there; revisit post-pilot.
- **Mobile native app.** Browser-responsive is fine for v1.
- **Multi-clinic / enterprise auth.** Single-clinic deployment for pilot; multi-tenant in v2.
- **Real-time scribe transcript display during visit.** Post-visit batch processing is much simpler and gets 80% of the value.
- **Marketing site / pricing pages.** Defer until you have product to sell.

If something feels critical and isn't here, ask: would the platform be unusable without it? Probably no. Add it after the pilot says it's needed.

---

## Bottom line

~17 weeks of build at solo part-time pace with Claude as primary coding partner. AI scribe is in v1. The full canonical flow (intake → brain map → treatment fit → prescribe → predicted layer → follow-up → predicted-vs-observed) works end-to-end with conversation capture.

After pilot, choices fork: bootstrap further, raise capital to expand, partner with an academic medical center or pharma. The pilot data informs that decision.

Start with Phase 0, Week 1. The fastest way to make this real is to write the first commit.

## Cross-references

- `14-master-design.md`
- `15-schema-extensions.md`
- `16-frontend-ux.md`
- `17-backend-stack.md`
- `18-structured-ai.md`
