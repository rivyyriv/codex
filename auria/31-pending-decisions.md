# 31 — Pending Decisions

Decisions discussed in design sessions but not yet committed to a design file. Each has a proposed answer based on the constraints already in place. Confirm or override before code starts.

---

## 1. Compliance / BAA posture

**Decision needed.** Which subprocessors require Business Associate Agreements (BAAs) given that the platform processes PHI (patient profiles, modifiers, audio, transcripts, summaries)?

**Proposed.**

- **Required BAAs:** Supabase (Postgres + Auth + Storage — primary PHI store), Anthropic (Claude API — processes transcripts and clinical notes), OpenAI (Whisper API — processes audio if used; alternative: self-hosted Whisper to avoid the BAA), Vercel (frontend hosting — receives no raw PHI in v1 but session data may include identifiers), Sentry (error monitoring — must be configured to scrub PHI before any BAA decision).
- **Not required:** GitHub (no PHI in code or repos), Fly.io/Railway (only if backend processes PHI — likely yes), domain registrar, analytics (if used, must not receive any PHI).
- **Posture statement to add to `14-master-design.md`:** "All subprocessors that may receive PHI hold a Business Associate Agreement with the operating entity. A subprocessor register is maintained and audited annually."

**Open question.** Self-hosted Whisper vs. OpenAI Whisper with BAA. Self-hosted avoids one BAA and one outbound PHI flow but adds infra burden. Recommend OpenAI Whisper with BAA in v1; revisit if cost or latency becomes a problem.

---

## 2. Patient consent for AI scribe

**Decision needed.** How is patient consent for audio recording captured?

**Proposed.**

- **Default to two-party consent.** Several US states (CA, FL, IL, MA, MD, MT, NV, NH, PA, WA) require all-party consent for recording. Default to two-party everywhere to simplify legal posture.
- **Written consent at intake.** A short consent form delivered with the first questionnaire link or signed on the in-office tablet. Plain language. Covers: what's recorded, what's transcribed, who has access, how long it's retained, right to refuse.
- **Verbal confirmation at visit start.** Provider says, before tapping "Start visit": "I'm going to record this visit for note-taking. Anything you'd like to say off-record, just let me know." A "Patient consents to recording" checkbox in the visit-start dialog.
- **Right to refuse without penalty.** If a patient declines, the scribe is disabled for that visit. Provider can still take manual notes and modifiers. Audit log captures the refusal.
- **Retention policy.** Audio retained for X days (recommend 30 — long enough for re-extraction if needed, short enough to limit blast radius). Transcripts retained per record-retention policy (typically 7+ years for clinical records).

**Open question.** Some states have additional consent requirements around AI processing of health data; the consent form should be reviewed by counsel before launch.

---

## 3. Brand name

**Decision.** **Auria.** Latin gold-rooted. Warm, editorial, clinical-warm. Pairs naturally with the Fraunces typeface and the visual direction in `21-ux-north-star.md`.

- Auria is used consistently everywhere — patient-facing, clinician-facing, UI, marketing, methodology bundle, brain-type narrative.
- The folder name (`receptor-intelligence/`) is a filesystem artifact preserved for repo history only.

**Verification still required before launch.**

- `.com` domain availability (auria.com is likely taken; consider auriahealth.com, getauria.com, useauria.com, hello.auria, or a variant)
- USPTO trademark search in healthcare / mental-health classes (IC 044 medical services; IC 042 software services)
- General internet name conflict check (no existing major Auria-branded product in the mental-health or clinical-software space)

**Naming conventions.**

- Product: Auria
- Company entity: separate decision (Auria, Inc. is a candidate)
- Underlying methodology bundle and registry data: Auria. The folder name (`receptor-intelligence/`) is preserved for repo history only.

---

## 4. Pilot exit criteria

**Decision needed.** What evidence from the friendly pilot tells the team that v1 is ready to scale beyond the original 1–3 psychiatrists?

**Proposed.**

- **Volume:** 3 providers each see at least 5 unique patients on the platform. (Total: 15+ patients.)
- **Provider net promoter:** ≥ 80% of pilot providers say they would recommend to a peer.
- **Provider time:** modifier review queue averages < 60 seconds per visit.
- **Prediction accuracy:** at 6-week retest, predicted-vs-observed deltas within ±1.0 standardized units on ≥ 70% of cells across all patients.
- **Safety:** zero critical incidents (defined as: a clinical decision the provider attributes to the platform that caused or could have caused patient harm).
- **Auto-approve trust:** at the chosen threshold, < 5% of auto-approved modifiers are subsequently reversed by the provider on post-hoc review.
- **Patient understanding:** in a post-visit survey, ≥ 70% of patients say the brain type identity helped them understand their condition.

**Kill criteria (if any of these hit, pause and rethink):**

- Any critical safety incident.
- Provider net promoter < 50%.
- Prediction accuracy < 40% within ±1.0.
- Auto-approve reversal rate > 15%.

---

## 5. Content authoring review order

**Decision needed.** In what order does a clinical advisor review and finalize the 8 AI-drafted disorder templates?

**Proposed order (by clinical impact and pilot relevance):**

1. **MDD** — highest prevalence in PCP setting; anchors Type 2 and Type 5.
2. **GAD** — second-highest prevalence; anchors Type 1.
3. **Panic Disorder** — high-acuity, common ER overflow into PCP; anchors Type 1.
4. **PTSD** — high prevalence, frequently missed; uses subtype_overrides for the dissociative branch (extra review attention).
5. **ADHD** — booming adult population, drug coverage implications complex (stimulant vs. non-stimulant); anchors Type 5.
6. **Social Anxiety** — fits Type 1 well, lower acuity, can defer.
7. **Insomnia** — depends on the v1.1 vocabulary expansion (`30-v1.1-schema-extensions.md`) being implemented; review after the new regions/systems are in the registry.
8. **Adjustment Disorder** — thinnest evidence base; lowest stakes; review last.

**Gate for pilot launch:** OCD (already reviewed), MDD, GAD finalized. The remaining 5 can launch in batches during the pilot. The platform handles "draft" status gracefully (provider sees a "Draft template — under clinical review" badge on the disorder chip).

---

## 6. Auto-approve threshold tuning protocol

**Decision needed.** Who can tune the 0.85 auto-approve threshold? How? With what audit trail?

**Proposed.**

- **Per-provider tuning.** Each provider has their own threshold value, defaulting to 0.85 at account creation. Threshold range is bounded: minimum 0.70, maximum 0.95. Below 0.70 the audit posture weakens too much; above 0.95 almost nothing auto-approves and the feature loses utility.
- **Provider self-service.** Threshold lives in provider settings, not in admin-only space. Each provider should be able to adjust their own threshold without admin involvement.
- **Audit trail.** Every threshold change writes a row to `audit.threshold_changes` with: provider_id, old_value, new_value, timestamp, optional free-text reason. No silent changes.
- **Default protection.** New provider accounts cannot ship with a threshold below 0.85. Lowering it requires an explicit acknowledgment dialog ("You are reducing the auto-approve threshold. More modifiers will commit without your pre-review. Confirm.").
- **High-stakes overrides.** Regardless of the threshold, prescribing decisions, type assignment, dose changes, and any cell modifier flagged as high-impact in its disorder template always queue for explicit commit. The threshold does not apply to these actions.
- **No global override.** Admins cannot raise or lower a provider's threshold remotely. This is a per-provider clinical judgment.

---

## Status of decisions

| # | Decision | Status |
|---|----------|--------|
| 1 | Compliance / BAA posture | Proposed; needs counsel review before launch |
| 2 | Patient consent for AI scribe | Proposed; needs counsel review before launch |
| 3 | Brand name | **Decided — Auria** (pending domain + trademark verification) |
| 4 | Pilot exit criteria | Proposed; numbers can be tuned |
| 5 | Content authoring review order | Proposed |
| 6 | Threshold tuning protocol | Proposed |

---

Last updated: 2026-05-13
