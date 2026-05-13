# 21 — UX North Star

The user experience is the product. Everything else (schema, backend, AI) exists to serve this flow. When in doubt about a design or build decision, the question is: does this make the flow below faster, clearer, or more trustworthy for the provider and the patient? If not, defer.

Read alongside `16-frontend-ux.md` (which specifies screens in implementation detail) and `14-master-design.md` (which specifies the product vision).

---

## Two surfaces

1. **Provider** — desktop web app. Six screens: login, dashboard, new patient, patient detail (main workspace), visit mode, follow-up retest.
2. **Patient** — mobile mini-app. Two surfaces: questionnaire link (pre-visit or in-office) and post-visit summary.

Everything else is supporting infrastructure.

---

## The full visit cycle, end to end

Eleven steps. Roughly two days of elapsed time for the active cycle, plus six weeks to the follow-up.

### 1. Pre-visit questionnaire

Patient receives a secure link on their phone, or hands a tablet in the waiting room. Takes Y-BOCS, PHQ-9, GAD-7, or MADRS (whichever the provider assigned at intake). One-time JWT, no account required.

*Surface:* patient mini-app.

### 2. Results land

System computes the patient's effective cell vector, renders the brain map, and suggests a brain type by closest-match to the six archetypes. Patient appears on the provider's dashboard under "Ready for review."

*Surface:* provider dashboard.

### 3. Provider opens the patient

The patient detail screen renders: brain map (effective state layer on by default), suggested type chip, subsystem heatmap, residual gap list, treatment fit table (drugs + supplements ranked by mechanism overlap), type-driven lifestyle and therapy-modality recommendations panel.

*Surface:* patient detail screen.

### 4. Provider clicks "Start visit"

The screen enters visit mode. A persistent timer appears in the corner. Audio recording begins via MediaRecorder with a red-dot indicator and elapsed time. An encounter record is created in the database.

*Surface:* visit mode (same screen, mode change).

### 5. Visit conversation

Provider talks to the patient normally. No special phrasing required. Audio uploads in chunks to the backend continuously. Provider may jot notes in a side panel or add a manual cell modifier if they want to capture something specific. Brain map remains visible for reference.

*Surface:* visit mode.

### 6. Provider clicks "End visit"

Audio is finalized, Whisper transcribes, Claude runs structured extraction over transcript + notes. Proposed cell modifiers are returned with confidence scores. High-confidence proposals (above a tunable threshold, default 0.85) auto-commit. The rest drop to a review queue.

*Surface:* proposal review panel.

### 7. Review queue

Each proposal is a card showing the verbatim transcript quote that triggered it, the cell or subsystem affected, the proposed magnitude, and the confidence rating. Provider approves, edits, or rejects each card. Approved modifiers write through and the brain map updates live. Failed transcript segments (if any) appear as timestamped grey blocks with a "dictate here" affordance.

*Surface:* proposal review panel.

### 8. Provider confirms brain type

If this is the first visit, the provider confirms or overrides the system-suggested brain type. Once committed, the type is stable — it does not change visit-to-visit. The type chip becomes the header of the patient detail screen.

*Surface:* type chip on patient detail.

### 9. Prescribe

Provider clicks a drug row in the treatment fit table. A prescribing dialog opens. Provider picks a dose band (low / medium / high — bands defined per drug). The predicted post-Rx brain map renders next to the current state map; both update as the dose band changes. Provider sets the expected re-evaluation date (defaults to +6 weeks) and optional notes. On confirm, a `TreatmentPrediction` snapshot is frozen — predicted cell-level deltas at the evaluation date.

*Surface:* prescribing dialog.

### 10. Patient summary

System drafts a plain-language summary. The spine is the brain type identity ("You are a Type 3 — Ruminative-Internal. Here's what that means about your brain"), followed by what the prescribed drug covers in plain language, followed by the type's supplement and lifestyle recommendations. Provider reviews and edits, then sends. The encounter is sealed, the audit log captures every modifier change and prescribing decision.

*Surface:* summary review screen.

### 11. Follow-up retest (~6 weeks later)

Patient retakes the questionnaire — same delivery channel as before. System detects the matching `TreatmentPrediction` and surfaces a predicted-vs-observed comparison view: each cell shows predicted delta, observed delta, and error. Provider sees how the prediction held up. Over time, these prediction/outcome pairs accumulate and calibrate the registry.

*Surface:* follow-up retest view.

---

## Design throughlines

These are the rules every screen and interaction must hold to.

**Provider stays with the patient, not the computer.** Visit mode is designed so the provider can talk to the patient face-to-face. Scribe runs silently in the background. Proposal review happens at the end, not during.

**AI proposes, clinician commits.** Every modifier from the scribe, every type assignment, every prescription requires an explicit clinician action. Nothing writes to the patient record without it.

**Evidence visible at the decision point.** Drugs show mechanism cells and citations on hover. Types show their research anchors. Supplements carry evidence-tier badges. The provider can always defend the recommendation back to a source.

**Two registers, one truth.** The provider sees cell-level mechanism. The patient sees brain-type identity. Both are projections of the same underlying patient vector — neither fabricates information.

**Determinism.** Same patient state + same registry version = same screens. Reproducible, debuggable, defensible.

**Visuals are the product.** Every screen should look like something a serious person would be proud to use. The brain map, the type chip, the treatment fit table, the patient summary — none of these are utilitarian. They are the trust contract with the provider and the dignity contract with the patient. Budget time and care for visual design proportional to clinical engineering. A clinically rigorous platform that looks like a 2010 EHR will not be trusted, used, or remembered. Stunning, intentional, restrained visual design is a non-negotiable v1 requirement — not a polish phase.

## Visual direction (v1)

The four anchoring decisions for v1 visual design. These constrain every screen, asset, and component decision downstream.

**Mood: clinical-warm.** Professional, calming, slightly soft. Think Headspace meets a high-end medical device. Whites and soft neutrals as the base, one or two warm accent colors (likely a saturated amber and a cool teal, see landing.html palette). Patients feel safe; providers feel trusted. Avoid: harsh contrasts, saturated dashboard-aesthetic colors, cold institutional whites.

**Brain map: stylized anatomical.** A simplified, illustrated brain — top-down and sagittal views available, region as colored zone, color encoding magnitude and direction of dysregulation. Recognizable as a brain at a glance; immediately interpretable for patients and providers alike. The hex-grid abstraction documented in earlier specs (`16-frontend-ux.md`) is retired as the v1 visual approach but retained as the underlying data model — the renderer can swap between anatomical and abstract views without changing data shape. Trades extensibility (full-body in v2 will need more design work per system) for clinical resonance now.

**Typography: Inter + Fraunces.** Inter for all UI (data tables, controls, dense provider screens, body copy) — clean, neutral, ergonomic. Fraunces for hero moments (patient summary headlines, brain type identity, marketing). Editorial weight where the patient lives, mechanical clarity where the provider works. Pair already in use in `landing.html` — brand continuity from marketing surface into the product.

**Hero strategy: all three surfaces, brain type as connective thread.** No single hero screen. The provider workspace, the brain map, and the patient summary each receive equal craft investment. The brain type — its color, its iconography, its descriptor — is the visual signature that ties them together. A patient identified as Type 3 sees a consistent color/icon for Type 3 on the provider's chip, on their own summary, and (eventually) on any marketing comparing types. This is the brand's most defensible visual asset.

### Component-level implications

- **Type chips:** distinctive shape (capsule), distinctive color per type, restrained icon. Visible on the patient detail screen header and at the top of the patient summary.
- **Brain map:** stylized anatomical illustration as the base layer, color overlay for delta. Multi-layer toggles (effective / disorder-only / treatment / residual / predicted) render as visual filters, not separate views.
- **Treatment fit table:** typographic hierarchy (Inter at varied weights) over heavy borders or shading. Evidence-tier badges as small, restrained chips. Drug vs. supplement distinguished by icon + subtle background tint, not by separate sections.
- **Patient summary:** magazine-grade typesetting. Fraunces for headings and the type identity statement, Inter for body and recommendations. Restrained color (mostly black on warm white with one accent), generous whitespace. The artifact the patient might print and bring to a therapist.
- **Color palette:** anchored to the existing `landing.html` palette (`--bg: #0a0e1a`, `--accent: #f5b86b`, `--teal: #6dd3c0`, `--rose: #ef6f7d`, `--azure: #7aa8ff`). Light-mode variants derived from this palette. One color per brain type (six total) drawn from a harmonious palette anchored to these accents.

---

Last updated: 2026-05-12
