# Issue — i18n policy (Module 1, instruction layer)

**Product:** DOGEstonia — Module 1 (GPT Interview → Issue)  
**Purpose:** Normative **i18n** behavior for **FR-M1-028…031** on top of [`story-data-model.md`](story-data-model.md) §4.1 (`{ et, ru, en }` objects). If wire contracts diverge from published OpenAPI, **OpenAPI wins after publication** and gaps go to the product backlog.

| Field | Value |
|-------|--------|
| **Version** | 0.4 |
| **Date** | 2026-04-27 |
| **Traceability** | FR-M1-028…031 (session + trilingual fields); SPA field-shape alignment via [`story-data-model.md`](story-data-model.md) §4 |

---

## 1. Session / interview surface language (FR-M1-028)

1. **Primary user-facing language** for questions, reflections, and Phase 7 summary text MUST follow the session language inferred from [`bootstrap.md`](bootstrap.md) **`comm_context.ui_lang`** (or explicit `/lang` user command), defaulting to the language of the user’s substantive content when `ui_lang` is still `unknown` after first turn.
2. Do **not** switch the interview surface language mid-session **without** an explicit user request or `/lang` update (avoid surprising bilingual flips).
3. **Linked modules:** [`story-interview-flow.md`](story-interview-flow.md) for dialogue phrasing; [`communication-presets-reference.md`](communication-presets-reference.md) for tone (tone is orthogonal to language — see §5).

---

## 2. Trilingual card fields (FR-M1-029)

1. Draft **`title`**, **`description`**, optional **`summary`**, optional **`institution`** MUST use the logical shape **`{ et: string, ru: string, en: string }`** as in [`story-data-model.md`](story-data-model.md) §4.1 — SPA mocks and UI expect all three keys.
2. **Fill strategy (MVP instruction layer):**
   - **Primary slot:** the language matching **`ui_lang`** (map `et` / `ru` / `en`) receives the **richest** text copied or lightly edited from the agreed narrative / Phase 7 framing.
   - **Other slots:** **translation** or **explicit placeholder** (e.g. same string with trailing `" [TBC]"` only if product policy allows — prefer short honest stub over fabrication). Never invent facts in a secondary language.
3. **Normalizer / API (M1-05 / M1-06):** may tighten canonical JSON; until then, validation reports MAY flag `needs_translation` style notes in metadata (wording not frozen).

---

## 3. Fidelity vs polish (FR-M1-030)

1. **Semantic fidelity** is mandatory for the **primary** language (see §1–§2).
2. **Secondary** languages may be: machine translation, human-style translation, or **controlled fallback** (e.g. English-only stub when Estonian/Russian unavailable) **only** when the downgrade is **explicit** in artifacts or user-facing notes — not silent degradation.
3. If translation confidence is low, prefer **shorter** secondary text that stays true over **fluent** secondary text that adds details.

---

## 4. No meaning distortion for translation (FR-M1-031)

1. Do **not** change facts, blame allocation, institutional claims, or emotional emphasis to make a translation read smoother.
2. If a literal translation reads awkwardly, keep the awkwardness or ask a **clarifying** question — do not “fix” the story.
3. Conflicts between languages after user correction in Phase 7 (**§7.2**): **user-affirmed primary language** wins; update secondaries to match, not the reverse.

---

## 5. Relation to presets and safety

- **Tone / verbosity presets** (`communication-presets-reference.md`) MUST NOT override §1–§4 (e.g. do not force English output just because `tone_preset` is `technical`).
- **Safety & compliance** overrides i18n when redaction or block is required; redacted text must remain consistent across `{ et, ru, en }` slots that remain visible.

---

## 6. Artifact: `session_language` in `normalized_issue_payload`

1. [`story-normalizer.md`](story-normalizer.md) **MUST** emit **`normalization_metadata.session_language`** with value `et` \| `ru` \| `en`, matching the **primary** user-facing language from §1 (`comm_context.ui_lang` / substantive content default).
2. Downstream modules and backends **MUST NOT** infer primary slot only by longest string — use **`session_language`** as the explicit marker alongside trilingual `{ et, ru, en }` objects.
3. This does **not** change OpenAPI request shapes until the node publishes a matching field; until then the key is **instruction-layer + logical handoff** truth.

---

## 6.1 Story Intake wire v2 (REQ-22)

For demo M2 **`POST /intake/stories`** (`m2.story_intake_envelope.v2`), the runtime contract requires:

1. `narrative.title` and `narrative.description` as **`{ et, ru, en }` objects** — map directly from `canonical_payload.title` / `.description` ([`api-orchestrator.md`](api-orchestrator.md) §5.2.1).
2. `narrative.original_text` — primary-slot string from `canonical_payload.description[session_language]`.
3. `narrative.session_language` — from `normalization_metadata.session_language`.
4. `narrative.language` — from `normalization_metadata.detected_input_language` (not a substitute for `session_language`).

Legacy `title_hint*` wire fields are **superseded** — do not emit them in Story Intake requests.

Issue draft routes (`IssueCreateRequest.title` as a single string) remain out of scope for this REQ; see Module 1 vs Module 2 boundary in [`story-lifecycle-instructions.md`](story-lifecycle-instructions.md) §2.2.

---

## 6.2 Demo language allowlist (D-11)

For demo M2 Story Intake submission, `session_language` MUST be one of: `et`, `ru`, `en`.

- If `ui_lang` resolves to a language outside this set, the interview may proceed normally,
  but the orchestrator MUST block `POST /intake/stories` and explain the limitation to the user.
- This restriction applies only to the Story Intake submission step — the interview itself
  may be held in any language supported by the model.
- Post-demo: expand allowlist per product decision; do not hardcode additional languages here
  without a lockstep update to the gateway allowlist.

---

## 7. Version history

| Version | Date | Change |
|---------|------|--------|
| 0.1 | 2026-04-10 | Initial policy (FR-M1-028…031); links to bootstrap, issue-data-model, interview flow, presets. |
| 0.2 | 2026-04-20 | Added §6 `session_language` in `normalization_metadata`. |
| 0.3 | 2026-04-25 | Added §6.1 runtime single-string bridge for GIM-74. |
| 0.4 | 2026-04-27 | Added §6.2 demo language allowlist for M2 Story Intake (`et` \| `ru` \| `en`); aligns with [`api-orchestrator.md`](api-orchestrator.md) §5.2.2 pre-flight (GIM-97). |
