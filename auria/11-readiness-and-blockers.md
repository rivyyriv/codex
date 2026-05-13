# 11 — Readiness and Blockers

What the framework is ready to support, what it isn't, and what content gaps starve the app even with perfect engineering.

This is the handoff doc for the engineer or product person walking into app planning. Read it before architecture decisions lock.

---

## TL;DR

The framework is a complete methodology spec: schema, protocol, audit rules, composition algebra, payload contracts, AI extraction discipline. **Engineering can start now in parallel with content population.**

But three content gaps will starve the app if not closed before launch, and six app-layer decisions sit outside the framework's scope and need explicit calls during app planning. Both lists below.

## Hard blockers — content gaps

The framework is a working machine with empty tanks. Until these close, the app can render but won't compute anything useful.

### 1. Drug Coverage cells (zero populated)

**Why it matters.** Residual computation is `disorder_vector − Σ treatment_vectors`. With zero coverage cells, Σ treatments = 0, so residual = disorder. "Coverage gap" reduces to "the disorder." "Next-line treatment" returns nothing. These are the headline clinician features.

**What's needed.** ~10 first-line agents, each with 10–30 coverage cells:

- sertraline, fluoxetine, escitalopram (SSRIs)
- venlafaxine, duloxetine (SNRIs)
- memantine, NAC (glutamatergic augmentation)
- aripiprazole (D2/5HT2A augmentation — critical for OCD+TS)
- clonidine (α2A — tics)
- lamotrigine (Glu modulation — bipolar/augmentation)

**Effort.** Mechanical content work. Schema is defined. Maybe a day per agent with PubMed surveys.

### 2. Only one disorder template populated (OCD)

**Why it matters.** Differential diagnosis distance needs neighbors to compute against. Today the app can answer "how OCD-like is this profile?" — not "is this MDD vs GAD vs OCD?" The undifferentiated-patient flow has nothing to match against.

**What's needed.** At minimum: MDD, GAD, ADHD, plus one of (PTSD, TS). Each is ~50 cells + ElicitationMap + DisorderTemplate metadata. Protocol is proven from OCD instantiation.

**Effort.** ~2–3 days each with the protocol. Five disorders ≈ 10–15 working days of content work.

### 3. HealthyBaselineTemplate not built

**Why it matters.** Schema v3 says every PatientProfile has `baseline_ref: "healthy_v1"`. That object doesn't exist yet. The effective-delta formula's first term has nothing to read against.

**What's needed.** A registry artifact `healthy_v1` with every cell at Δ=0, `evidence_status: "evidenced"`, across the standard anatomy × receptor inventory.

**Effort.** Mostly mechanical — half a day to enumerate. Cells inherit from the standard inventory.

## App-layer decisions

These aren't framework gaps. They're outside the framework's scope by design. Each is a decision that shapes app architecture.

### 1. PHI separation

The framework deals in registry data (no PHI) and patient data (PHI: profiles, modifiers, states, notes). The app must keep them separate at the storage layer, both for HIPAA and for blast-radius reasons.

**Decision needed:** what's the storage architecture for PHI vs. registry?

Options:
- **(a) Separate Postgres clusters** — one for registry (read-heavy, public-ish), one for PHI (encrypted, BAA-covered).
- **(b) Single cluster, separate schemas** — `registry.*` vs. `phi.*` schemas with different RLS policies.
- **(c) Single schema, table-level encryption** — simplest but riskiest.

Recommendation: (a) for v1. Easier to defend in audit, easier to scope BAAs, easier to swap providers.

### 2. Auth + RBAC

Three principal roles, and a forth optional:

- **Patient** — sees own profile, own state history, own patient-facing summary.
- **Clinician** — sees patients in their caseload, can write modifiers, can run treatment narrowing.
- **Clinical reviewer / admin** — edits the registry (cells, rules, templates), reviews proposed modifiers from AI extraction.
- **(Optional) Researcher** — read-only access to deidentified registry; no PHI access.

**Decision needed:** auth provider (Supabase Auth, Auth0, Clerk, custom), and the RBAC matrix mapping role × resource × action.

### 3. Regulatory posture

The framework provides decision support, not prescribing decisions. Whether the app counts as Software-as-Medical-Device (SaMD) depends on jurisdiction and how features are framed.

Examples:
- **Decision support tool** (clinician-in-loop, no specific recommendations) → typically lower-risk classification.
- **Diagnostic ranking tool** with claims of accuracy → potentially SaMD.
- **Treatment recommendation tool** with specific drug suggestions → potentially SaMD.

**Decision needed:** counsel review of the planned UI claims against FDA / EMA / TGA SaMD guidance. The framework's design (mechanism-level fit, not prescribing decisions; differential distance, not diagnostic claims) is intentionally conservative — but the framing in the UI is what regulators read.

### 4. Registry source-of-truth

The registry currently lives in Notion (cells, composition rules, curated templates). The app needs it as queryable JSON/Postgres.

**Two paths:**

- **(A) Notion remains source of truth.** Build a Notion → Postgres export pipeline. Clinical reviewers keep editing in Notion, app reads from Postgres.
  - Pros: clinical staff can edit without engineering.
  - Cons: export pipeline is its own moving part; schema drift risk.
- **(B) Migrate registry to Supabase, Notion becomes documentation only.** Reviewers use a small admin UI.
  - Pros: single source of truth, version control via DB migrations, transactional safety.
  - Cons: clinical reviewers need an admin UI to edit cells (or stay in Notion and you periodically reconcile).

**Recommendation:** (B), with a small admin UI for reviewers. Notion stays for methodology docs and decision provenance, registry edits move to Supabase. (A) is appealing but Notion's schema and Postgres's diverge fast in practice.

### 5. Cascading template updates

Parked in the framework. When `ocd_canonical_v2` → `v3`, what happens to existing PatientProfiles?

**Options:**

- **(a) Pin-by-default** — profiles stay on the version they were created with. Clinician opts in to migrate.
- **(b) Auto-migrate with diff review** — profiles re-baseline against the new template, modifier cells preserved, clinician reviews changed effective deltas before commit.
- **(c) Weighted cascade** — modifiers proportionally rescale to preserve effective delta when template changes are minor.

**Recommendation:** (a) for v1. Conservative, explainable, audit-friendly. Revisit when shipping the second template version.

### 6. Clinical advisory / sign-off

Not a framework gap, but a launch-readiness gap: at some point a real psychiatrist needs to sign off on the cell values and composition rules, or the app ships un-vetted neuroscience to clinicians.

**Decisions needed:**

- Who plays this role (named clinical advisor, advisory board, or formal medical director).
- What's their review cadence (per-disorder template release? quarterly?).
- How are their changes recorded (the cell schema already has `reviewer` and `last_reviewed`; use them).
- How is disagreement handled between advisor and existing literature (the protocol is designed around evidence; advisor input is one more source, not an override).

**Why it matters before launch:** liability, credibility, and frankly the moral claim. Advisor reads the framework page once, signs off on protocol; reviews each disorder template before release.

## Pre-handoff framework fixes

Small additions worth making while the framework is fresh in your head. All easier now than after the app exists and depends on them.

- **`recency_window_days` field on PatientSubsystemModifier.** Y-BOCS asks past week, ASRS asks past 6 months. Modifier outputs should carry the recency window so downstream code can weight chronic vs. episodic appropriately. Schema supports it; needs to flow through the intake pipeline.
- **Multi-instrument administration rule.** When MADRS and PHQ-9 are both administered, modifiers do not stack — the higher-confidence source takes precedence. Resolution rule lives in the intake pipeline contract, not yet in code.
- **Region enum additions.** CMPf, mPFC, Cerebellum, PAG are referenced in OCD content but not all of them are in the formal enum. Add them.
- **More instruments.** YGTSS (tics), PCL-5 (PTSD), DOCS or OCI-R (OCD F-subsystem). Each is an ElicitationMap addition.
- **Multi-instrument precedence in pipeline.** Sketched in `04-elicitation-maps.md`; needs to be coded.
- **Drug coverage authoring.** The first major content lift (see Blocker #1).

## Suggested v1 scope

Opinionated take on what to ship vs. defer in v1.

### Ship in v1

- Patient profile with single-disorder template_ref support.
- Y-BOCS, MADRS, PHQ-9, GAD-7, ASRS instruments operational.
- BrainMap, SubsystemHeatmap, ResidualGapView primitives.
- TreatmentFitTable for first-line agents (requires Blocker #1 closed).
- DifferentialDistanceRanking for undifferentiated profiles (requires Blocker #2 closed).
- PatientFacingSummary (Layer 3) for confirmed profiles.
- Audit dashboard for clinical team.
- AI extraction OFF (questionnaires + clinician-direct only).
- Curated comorbidity template `ocd_plus_ts_v1` operational.

### Defer to v2

- AI extraction service.
- Cascading template updates beyond pin-by-default.
- DOCS, OCI-R, YGTSS, PCL-5 instruments.
- Class-level drug coverage inheritance.
- Three-way comorbid composition.
- Anatomical 3D brain projections (use flat or hex-grid in v1).
- Treatment trajectory animation.

### Defer to v3+

- Genetic / pharmacogenomic integration (CYP2D6, COMT).
- Functional connectivity / network-level overlays.
- Acute pharmacodynamics (kinetics).
- Multi-language support.
- Patient-direct authoring (currently clinician-mediated only).

## Risk register

Seven likely failure modes. Each has a mitigation.

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| Drug coverage cells take 3× as long as estimated | Medium | High | Start with 3 agents (sertraline, aripiprazole, NAC) before committing to all 10 |
| Pilot clinical advisor disagrees with cell values | Medium | High | Ship draft templates first; advisor review before active promotion |
| AI extraction proposals are too noisy in pilot | High | Medium | Ship without AI extraction in v1; tune in v2 |
| PHI compliance gap discovered in audit | Low | Critical | Architecture review with counsel before launch |
| Notion → Postgres export pipeline drifts | Medium | Medium | Choose (B) above (Postgres as source of truth) |
| Differential distance ranking misranks at low cell counts | Medium | Medium | Show similarity score, not just rank; surface "low confidence" warnings |
| Treatment fit table promotes prescribing claim accidentally | Low | Critical | UI review with counsel; mechanism-only language enforced in templates |

## Done-criteria checklist (run before scheduling app launch)

- [ ] HealthyBaselineTemplate registry artifact built
- [ ] Drug coverage cells populated for ≥8 agents
- [ ] At least 3 disorder templates beyond OCD (MDD, GAD, ADHD recommended)
- [ ] PHI separation designed and reviewed
- [ ] Auth roles defined and reviewed
- [ ] Registry source-of-truth decision made (Notion vs Supabase)
- [ ] Cascading update policy decided (recommend pin-by-default for v1)
- [ ] Clinical advisor identified and onboarded
- [ ] BAA executed with infrastructure providers (Supabase, Anthropic if applicable)
- [ ] SaMD posture reviewed by counsel
- [ ] Audit log on every modifier write
- [ ] AI extraction off in v1 (or BAA + evidence-text enforcement on if enabled)
- [ ] Layer-3 patient summary reviewed by clinical advisor on dummy patients
- [ ] Pilot patients (advisor's own caseload) onboarded before external clinicians

## What's NOT here yet

Items the framework deliberately doesn't try to provide. The app architecture supplies these:

- Database schema (Postgres DDL) — needs to be derived from JSON Schema during app architecture.
- API endpoint specification — derived from Visualization API payload contracts (`13-api-endpoint-derivation.md` has a starting sketch).
- Auth / RBAC matrix — needs to be authored in app planning.
- PHI separation design — needs to be authored in app planning.
- Registry export pipeline (Notion → Postgres) or migration plan — needs to be authored in app planning.

## Bottom line

The framework is robust enough to start app planning in parallel with content population. You don't have to wait. But ship-readiness requires the three content blockers closed and the six app-layer decisions made.

Two natural next moves to unblock the app:

- Populate `healthy_v1` (mostly mechanical, half a day).
- Start drafting MDD or GAD disorder templates (the next big content lift).

Both unblock the app meaningfully. Order is reversible.
