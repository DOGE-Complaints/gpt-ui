# Issue — i18n policy (Module 1, instruction layer)

**Product:** DOGEstonia — Module 1 (GPT Interview → Issue)  
**Purpose:** Normative **i18n** behavior for [REQ-09](../docs/requirements/REQ-09-functional-requirements.md) **FR-M1-028…031** on top of [`issue-data-model.md`](./issue-data-model.md) §4.1 (`{ et, ru, en }` objects). This file does **not** replace REQ PDFs or node OpenAPI; if wire contracts diverge, **OpenAPI wins after publication** and gaps go to `REQ-16` / product backlog.

| Field | Value |
|-------|--------|
| **Version** | 0.1 |
| **Date** | 2026-04-10 |
| **Traceability** | REQ-09 §9.7 (FR-M1-028…031), REQ-10 §10.5–10.6, REQ-15 item 4 (SPA field shape); **STORY GM3-06** (EPIC-M1-03) |

---

## 1. Session / interview surface language (FR-M1-028)

1. **Primary user-facing language** for questions, reflections, and Phase 7 summary text MUST follow the session language inferred from [`bootstrap.md`](./bootstrap.md) **`comm_context.ui_lang`** (or explicit `/lang` user command), defaulting to the language of the user’s substantive content when `ui_lang` is still `unknown` after first turn.
2. Do **not** switch the interview surface language mid-session **without** an explicit user request or `/lang` update (avoid surprising bilingual flips).
3. **Linked modules:** [`issue-interview-flow.md`](./issue-interview-flow.md) for dialogue phrasing; [`communication-presets-reference.md`](./communication-presets-reference.md) for tone (tone is orthogonal to language — see §5).

---

## 2. Trilingual card fields (FR-M1-029)

1. Draft **`title`**, **`description`**, optional **`summary`**, optional **`institution`** MUST use the logical shape **`{ et: string, ru: string, en: string }`** as in [`issue-data-model.md`](./issue-data-model.md) §4.1 — SPA mocks and UI expect all three keys.
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

## 6. Version history

| Version | Date | Change |
|---------|------|--------|
| 0.1 | 2026-04-10 | **GM3-06:** initial policy (FR-M1-028…031); links to bootstrap, issue-data-model, interview flow, presets. |
