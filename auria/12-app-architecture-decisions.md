# 12 — App Architecture Decisions

Six decisions outside the framework's scope that need to be made before app architecture locks. Each is non-trivial, each shapes downstream code, and none of them have a single obviously-right answer — but each has a defensible recommendation.

This doc expands on the summary in `11-readiness-and-blockers.md` with the actual tradeoff space, the recommendation, and the cost of getting it wrong.

---

## How to use this document

Treat each section as a decision record. By the end of app planning each section should have:

1. The chosen option (one of the listed paths or a documented alternative).
2. Who made the call and when.
3. The constraints that drove it (compliance, team size, time-to-launch, cost).
4. The conditions under which it would be revisited.

The framework is deliberately silent on these — they're product/infra decisions, not methodology decisions. But the framework does have opinions about which choices preserve auditability, which preserve PHI integrity, and which avoid expensive rework. Those opinions are surfaced as "recommendation" below.

---

## Decision 1 — PHI separation

**The question.** How is patient health information (PHI) separated from non-PHI registry data at the storage layer?

### What's PHI in this app

- `PatientProfile` records (linkable to a real person via clinician's caseload)
- `PatientState` records (current effective deltas + active regimen)
- `PatientSubsystemModifier` and `PatientCellModifier` records
- Clinical notes that feed AI extraction
- Questionnaire responses (Y-BOCS, MADRS, etc.)
- Any audit log entry referencing the above

### What's not PHI

- The cell registry (`DisorderCell`, `DrugCoverageCell`)
- `DisorderTemplate`, `HealthyBaselineTemplate`, `CuratedComorbidityTemplate`
- `ElicitationMap` definitions
- `CompositionRule` definitions
- Schema documents themselves

The asymmetry matters: registry data is read-mostly, version-controlled, useful to reference researchers. PHI is write-heavy, encrypted at rest, BAA-scoped. Mixing them in storage forces the strictest policy on the most permissive data.

### Options

**(a) Separate Postgres clusters.** One cluster for the registry (no BAA needed, public-readable for research collaborations), one for PHI (BAA-covered, encrypted at rest, restricted network).

- Pros: cleanest blast radius. A registry vulnerability never exposes PHI. BAA scope is small. Easy to scope a security audit.
- Cons: cross-cluster joins require app-layer composition. Two sets of backups, two sets of monitoring. Marginally higher infra cost.

**(b) Single cluster, separate schemas with row-level security.** `registry.*` and `phi.*` schemas with strict RLS policies; `phi.*` only writable from authenticated clinician sessions.

- Pros: single ops surface, joins are cheap, single backup story.
- Cons: every audit conversation now has to explain the RLS posture. A misconfigured policy is a PHI leak. Same disk, same backup snapshots — harder to scope BAAs.

**(c) Single schema, table-level encryption.** Everything in one place; PHI columns encrypted at rest with separate keys.

- Pros: simplest dev experience. Cheapest to start.
- Cons: hardest to defend in audit. Key management becomes the entire compliance story. Not recommended for HIPAA workloads.

### Recommendation

**(a) Separate Postgres clusters** for v1.

The defensibility argument is the deciding one. When a security questionnaire asks "how is PHI isolated from non-PHI data?" the answer "different clusters in different VPCs with different access controls" is short and obviously correct. (b) requires a half-page explanation. (c) requires a multi-page explanation.

Cost difference is real but small at v1 scale (two small clusters cost only modestly more than one). Operational complexity is real but bounded — you're adding a second connection pool to the API, not a second product.

If the team has strong opinions toward (b) for cost or simplicity, that's defensible — but require a written RLS audit before launch and quarterly thereafter.

### Cost of getting this wrong

- Wrong choice (a) → mild over-engineering; modest infra spend; never a security incident from this surface.
- Wrong choice (b) without RLS audit → potential HIPAA exposure; potential breach notification; reputational and regulatory cost.
- Wrong choice (c) without strong key management → as above, worse.

---

## Decision 2 — Authentication and RBAC

**The question.** Who can do what, and how do they prove who they are?

### Principal roles

The framework implies four roles:

- **Patient.** Sees their own profile, their own state history, their own Layer-3 patient-facing summary. Cannot see registry internals, cannot see other patients, cannot see clinician-only annotations.
- **Clinician.** Sees patients in their caseload. Can read registry. Can write `PatientCellModifier`, run treatment-fit and differential-distance queries, edit profile metadata. Cannot edit registry.
- **Clinical reviewer / registry admin.** Edits the registry: cells, composition rules, templates, elicitation maps. Reviews proposed modifiers from the AI extraction queue. Has read access to anonymized audit logs but not to identifiable patient data unless also a clinician.
- **(Optional) Researcher.** Read-only access to the registry and to fully de-identified aggregate data. Never PHI.

### What needs to be decided

1. **Auth provider.** Supabase Auth, Auth0, Clerk, or self-hosted (Keycloak, Ory).
2. **MFA policy.** Required for clinician and admin roles? Recommended yes.
3. **Session lifetimes.** Clinician sessions in clinical settings → typically short (15–60 min). Patient sessions → longer. Admin sessions → short.
4. **The full RBAC matrix.** Role × resource × action. Roughly 4 roles × 8 resource families × 4 actions = 128 cells in the matrix.
5. **Caseload boundary enforcement.** Clinician sees patient X iff a `caseload_membership` record exists. Where does that record live, who creates it, who removes it?
6. **Break-glass policy.** Can a clinician outside the caseload access in an emergency? If yes, what's the audit story?

### Options for auth provider

**(a) Supabase Auth.** Tightly integrated if you use Supabase elsewhere. Free tier generous. JWT-based. Some weakness on enterprise SSO and MFA features at the lower tiers.

**(b) Auth0.** Mature. Great enterprise SSO. Costs ramp fast at MAU scale. Excellent audit features.

**(c) Clerk.** Modern DX. Good for B2C. Less battle-tested for healthcare compliance specifically.

**(d) Self-hosted Keycloak / Ory.** Full control. Significant ops burden. Defensible for healthcare.

### Recommendation

**Supabase Auth** for v1, with the explicit upgrade path to Auth0 once enterprise customers (hospital systems) require SSO/SAML.

Reasoning: if Decision 4 lands on Supabase as registry source-of-truth, Supabase Auth is already in the stack. The MFA story is adequate for v1 (TOTP via authenticator apps). The migration to Auth0 later is well-trodden.

For MFA: **required** for clinician and admin, **optional** for patient.

For session lifetimes: clinician 30 min idle, admin 15 min idle, patient 12 hours rolling.

For caseload: a `caseload_memberships(clinician_id, patient_id, granted_at, granted_by)` table. Any read of PHI joins through it. Audit logs every caseload membership creation.

For break-glass: yes, with a forced reason field, written to a separate audit log, with email notification to the practice administrator. This is operationally important for ER and on-call scenarios.

### Cost of getting this wrong

- Under-engineered RBAC → patient data visible to wrong clinician → critical incident.
- Over-engineered → clinicians work around the system, accuracy of caseload data degrades, then audit logs become unreliable.

The middle path: simple model, strict enforcement, generous break-glass with strict logging.

---

## Decision 3 — Regulatory posture (SaMD)

**The question.** Does this app count as Software-as-Medical-Device under FDA / EMA / TGA / equivalent rules, and what does that imply for design and launch?

### Why this is non-trivial

The framework was designed conservatively for a reason. The cell-level mechanism representation is mechanism-of-action, not diagnostic claim. "Coverage gap" is residual delta, not "you need this drug." Differential distance is similarity score, not diagnosis.

But the UI is what regulators read. An identical computation surfaced as "similarity to MDD: 0.87" is decision support; surfaced as "Diagnosis: MDD (87% confidence)" is a diagnostic claim and likely SaMD.

### The classification axes

- **Decision support tool, clinician-in-loop, no specific recommendations.** Generally lowest-risk classification. Most decision support tools live here.
- **Diagnostic ranking with stated accuracy claims.** Steps into SaMD territory in many jurisdictions.
- **Treatment recommendation with specific drug suggestions and dose.** SaMD almost certainly.

The framework supports all three modes mechanically. The regulatory posture decides which UI surfaces are exposed and how they're framed.

### Options

**(a) Conservative posture (decision support only).** UI surfaces show mechanism-level fit, residual gaps, and ranked similarity. Never names a diagnosis with a confidence score. Never recommends a specific drug. Always renders "discuss with your clinician" framing in patient-facing surfaces.

**(b) Mid-risk posture (diagnostic and treatment ranking, framed as guidance).** UI surfaces include differential ranking with confidence and treatment fit table with named agents. Heavy disclaiming. Probably triggers SaMD review in EU and AU; likely fine as decision support in US under current FDA enforcement.

**(c) Full SaMD posture.** Pursue de novo classification or 510(k); commit to QMS, post-market surveillance, the full apparatus. Year-plus delay to launch.

### Recommendation

**(a) for v1, with a documented path to (b)** for clinician-side surfaces under counsel review, and (c) only if a partner (pharma, large hospital system) requires it for procurement.

The framework's design supports (a) trivially: ResidualGapPayload and TreatmentFitPayload exist. They're framed as "coverage gap" and "mechanism overlap," not "diagnosis" and "prescription." Keeping that framing in the UI keeps you out of SaMD scope in most jurisdictions.

The single most important rule: **the patient-facing surface (PatientFacingPayload) never names a specific drug, never gives a specific recommendation, never renders a numeric diagnostic confidence.** It explains mechanisms in plain language. Anything stronger is a launch blocker.

### Cost of getting this wrong

- Over-conservative → less differentiated product, harder to pitch to providers.
- Under-conservative → enforcement action, mandatory recall, reputational damage that ends the company.

The asymmetry says start conservative. You can always relax framing under counsel review; you can't undo a 510(k) requirement.

---

## Decision 4 — Registry source of truth

**The question.** Where do the cells, rules, and templates actually live, and how do they get to the running app?

### The current state

The registry was authored in Notion. Cells, composition rules, comorbidity templates, elicitation maps — all currently Notion databases. Notion is great for collaborative authoring with non-engineers. It is not a runtime data store.

### Options

**(A) Notion remains source of truth.** Build a Notion → Postgres export pipeline that runs on a schedule (or webhook trigger). Clinical reviewers continue editing in Notion; app reads from Postgres.

- Pros: zero workflow change for clinical reviewers. They already know Notion. No new admin UI to build or maintain.
- Cons: the export pipeline becomes a critical path. Notion's schema and Postgres's drift over time and reconciliation is painful. Notion is not transactional — concurrent edits during export are a class of bugs. Notion API rate limits are real. No row-level audit of who changed what (Notion has page-level history but not field-level).

**(B) Migrate registry to Supabase, Notion becomes documentation only.** Reviewers use a small admin UI built specifically for cell/rule/template editing. Notion archives live as the methodology and decision-provenance docs (this very bundle), not as the runtime registry.

- Pros: single source of truth, transactional safety, real audit log per row, schema enforced by the database. Migrations are version-controlled. CI can run schema audits.
- Cons: clinical reviewers need to learn the admin UI. Need to actually build the admin UI. Methodology docs and registry data drift apart unless explicitly maintained.

**(C) Hybrid: Notion for narrative templates and decision provenance; Supabase for cells.** The cell registry is too structured for Notion to remain authoritative; the comorbidity templates and decision rationales are too narrative for a relational store. Split them.

- Pros: each store does what it's good at.
- Cons: the boundary is fuzzy. CuratedComorbidityTemplate has both a structured cell-overlay and narrative rationale. Where does it live?

### Recommendation

**(B) with a small admin UI** for v1.

Reasoning: the registry has reached a size and structure where Notion's lack of schema enforcement is starting to bite. A clinician adding a cell with `region: "ofc"` (lowercase) when the enum is `"OFC"` doesn't surface as an error in Notion; it surfaces as a missing payload in the running app weeks later. That class of bug compounds.

The admin UI doesn't need to be elegant for v1. A typed React form with dropdowns for enums, an autocomplete for sources (PMIDs), and a save button is enough. Authoring volume is bounded — tens of cells per week at peak, not thousands.

Notion stays valuable for: methodology docs (this bundle), decision provenance, ADRs, comorbidity narrative rationales, advisor sign-off correspondence. Treat it as the lab notebook, not the database.

If the team strongly prefers (A) — clinical reviewers won't accept a new tool — make the export pipeline transactional (snapshot the entire Notion DB, validate against schema, then atomically swap the read-table) and require schema validation as a build gate.

### Cost of getting this wrong

- Wrong (A) → schema drift, runtime errors, gradual erosion of registry quality.
- Wrong (B) without budgeting admin-UI build time → admin UI ships rough, reviewers complain, edits stall.
- Wrong (C) → eternal arguments about which store owns which fields.

---

## Decision 5 — Cascading template updates

**The question.** When `ocd_canonical_v2` becomes `v3`, what happens to the thousands of `PatientProfile` records that reference v2?

### Why it's hard

A patient's profile is `template_ref + modifiers`. The effective deltas depend on the template's content. If the template changes, the patient's effective deltas change — without anyone editing the patient.

That's both a feature and a hazard. Feature: improvements to the registry automatically improve patient analyses. Hazard: a clinician opens a patient chart and sees a different residual gap than yesterday, with no audit trail explaining why.

### Options

**(a) Pin-by-default.** Profiles stay on the version they were created with. A clinician opts in to migrate per patient. The migration is a deliberate act, logged.

- Pros: profiles are stable. Audit trail is clean. No one is surprised.
- Cons: registry improvements don't propagate. Old templates accumulate. Eventually you have profiles on v2 while the registry's at v6.

**(b) Auto-migrate with diff review.** When a new template version ships, profiles re-baseline against the new template, modifiers preserved. Clinician sees a diff before commit on next patient open.

- Pros: improvements propagate quickly. Cliniician stays informed.
- Cons: clinician workflow becomes "review N diffs before getting to today's work." Painful at scale.

**(c) Weighted cascade.** When changes are minor (no cells added/removed, deltas adjusted within ±0.5), modifiers proportionally rescale to preserve effective delta. When changes are major, fall back to (a).

- Pros: best of both worlds for small updates.
- Cons: defining "minor" is fraught. The proportional rescaling math is real but subtle and the audit story is harder.

### Recommendation

**(a) Pin-by-default** for v1. Revisit when shipping the second template version.

Reasoning: a v1 app that loses clinician trust because effective deltas drift overnight is dead. Pinning preserves trust at the cost of slower registry-improvement propagation. That cost is acceptable in v1 because the registry isn't iterating fast enough to matter.

When the second template version ships, the friction will surface naturally and the team can decide between (b) and (c) with real data on what's changing.

### Implementation note

Schema v3 already supports this. Every `PatientProfile.template_refs[].template_version_pin` is an explicit field. Default it to the version current at profile creation. The migration tooling becomes "set `template_version_pin` to the new version, recompute, render diff, commit on confirm." Build the diff renderer as a v2 feature, not v1.

### Cost of getting this wrong

- Wrong (a) → registry improvements don't propagate; gradually a stale-data product.
- Wrong (b) at v1 → clinicians hit a wall of diffs and lose trust.
- Wrong (c) at v1 → audit trail unclear, edge cases produce wrong rescalings.

---

## Decision 6 — Clinical advisory and sign-off

**The question.** Who is the named clinical authority for the cell values, composition rules, and ranges?

### Why this is a hard requirement

The app produces decision-support output that clinicians will use to think about patient care. The cell values were derived from literature and authoring judgment. At some point, a credentialed psychiatrist needs to sign off on the methodology and on each disorder template.

Without that sign-off:

- Liability is unclear. If the app's output influences a treatment decision and the underlying cell value was wrong, the company has no defense beyond "we read papers."
- Credibility is fragile. Clinicians on pilot ask "who reviewed this?" and the answer needs to be a name.
- The moral claim — "this is grounded in the literature" — has no human accountable for it.

### Decisions needed

1. **Who plays the role.** A named clinical advisor (one person) or an advisory board (small panel)? Or a formal Medical Director (employee or contracted)?
2. **Cadence.** Per-disorder template? Quarterly review? Per-major-version of a template?
3. **Authority boundary.** Does the advisor have veto over a template release? Can they require revisions? Can they require additional evidence?
4. **Compensation and contract.** Equity? Hourly? Pro bono with attribution? This shapes who you can attract.
5. **How disagreement with literature is recorded.** When the advisor disagrees with a cell value supported by Tier-1 evidence, the protocol weights evidence. Is the advisor an evidence source (added to the cell's `sources` array) or an override?

### Options

**(a) Named advisor model.** One credentialed psychiatrist, paid as advisor (equity + small retainer or hourly), reviews each disorder template before activation. Reviews methodology once, then per-template thereafter.

- Pros: single point of accountability. Faster reviews. Easier to onboard one person deeply.
- Cons: single-point-of-failure if they leave. Their bias is the company's bias.

**(b) Advisory board model.** Three to five psychiatrists, each with subspecialty depth (mood, anxiety, psychotic, neurodevelopmental, addiction). Reviews are by-quorum or by-subspecialty.

- Pros: deeper coverage. Multiple perspectives reduce bias risk. Better procurement story for hospital partners.
- Cons: slower reviews. Board management overhead. Higher total comp.

**(c) Medical Director.** A psychiatrist on staff or formally contracted, with sign-off authority on releases and ongoing post-market involvement.

- Pros: strongest accountability. Clearest regulatory story (especially relevant for SaMD).
- Cons: significant cost. Possibly premature for v1.

### Recommendation

**(a) for v1.** Move toward (b) or (c) at the inflection where you're shipping templates faster than one person can review (probably ~6 templates / year, or when entering a new subspecialty area).

Concretely: identify a candidate advisor early (during this app planning phase, not during pilot). They review the framework methodology once, sign off on the protocol. Then they review each disorder template before it goes live in the registry. Their sign-off is recorded against the template version.

Their disagreement with literature is treated as another evidence source — added to the cell's `sources` with `type: "advisor-judgment"`, `tier: 2`, with a brief note. Not an override. The framework is designed around evidence, and the advisor is a high-quality but fallible source.

### Cost of getting this wrong

- No advisor → liability exposure, credibility gap, the moral problem.
- Wrong advisor (one whose judgment is itself biased) → systematically tilted templates that look authoritative but aren't.
- Right advisor, wrong cadence (annual review for a fast-moving registry) → registry ships ahead of sign-off and the advisor becomes a rubber-stamp at best.

The model that works: short-list candidates now, contract one before pilot, build the per-template sign-off workflow into the admin UI from day one.

---

## Decisions matrix — quick reference

| # | Decision | Recommendation | Cost-of-wrong | Latest defensible time to decide |
|---|---|---|---|---|
| 1 | PHI separation | Separate Postgres clusters | Critical (compliance) | Before any PHI is written |
| 2 | Auth + RBAC | Supabase Auth, MFA on clinician/admin | Critical (data leak) | Before pilot |
| 3 | SaMD posture | Conservative (decision support) | Critical (regulatory) | Before public framing locks |
| 4 | Registry source of truth | Supabase + admin UI | Medium (rework cost) | Before second disorder template authored |
| 5 | Cascading updates | Pin-by-default | Medium (clinician trust) | Before second template version ships |
| 6 | Clinical advisor | Named advisor for v1 | Critical (credibility) | Before pilot |

The pattern: four of six are "before pilot" decisions. The other two have natural triggers in the work that buy time.

---

## Decisions deliberately not made here

These are real questions but not for app planning:

- **Charting library.** A v1 implementation choice; the payload contracts in `05` are stable across libraries.
- **State management.** React component contracts in `06` don't dictate Redux vs Zustand vs Jotai.
- **Anatomical brain assets.** Hex grid, flat anatomical SVG, or 3D — primitive contracts permit all three. Pick the one your team can ship in v1 and upgrade later.
- **Hosting.** Vercel, Fly, Render, AWS — orthogonal to everything in this bundle.
- **Observability stack.** Sentry, Datadog, OpenTelemetry — orthogonal.

If any of these become decisions worth recording, do so in app-architecture docs, not here.

---

## Cross-references

- `11-readiness-and-blockers.md` — the original summary list of these decisions
- `13-api-endpoint-derivation.md` — the API surface that sits on top of these decisions
- `02-cell-registry-spec.md` and `01-schema-v3.md` — the data shape that Decision 4 has to support
- `07-ai-extraction-spec.md` — its rollout interacts with Decision 3 (regulatory posture)
