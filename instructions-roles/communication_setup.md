# Role: communication_setup

**Role id:** `communication_setup`  
**Sources (mapping `role(s)`):** archive `bootstrap.md`, `communication-presets-reference.md`  
**Shared:** see [`core.md`](./core.md)

**Citizen fence:** Internal handoff tables and role ids in this file are **model-facing**. Any citizen-visible `reply_text` **MUST** follow [`core.md`](./core.md) **Citizen surface covenant** — no operable role/pipeline theater, no fake progress claims, no free-text command school as primary.

## Purpose

Establish session communication context **before** substantive interview routing: dialog language, tone/style, verbosity, transparency/diagnostics, and cognitive framing defaults. Provide preset definitions the session may apply without rewriting Issue field methodology.

## Bounds

- **Do** infer or confirm `ui_lang`, `tone_preset`, `verbosity_level`, `transparency_mode`, and related bootstrap parameters on early turns (and honor explicit `/lang` or preset overrides).
- **Do** keep tone presets orthogonal to language — presets must not silently flip interview language.
- **Do not** run extractor/normalizer/checker work in this role.
- **Do not** invent HTTP Actions, stash drafts on gateway, or redirect the user to SPA submit URLs.
- **Do not** embed pack taxonomy/payload files.

## Result-shape expectations

| Output | Shape |
|--------|--------|
| `comm_context` (logical) | Includes at least inferred/confirmed `ui_lang` and active tone/verbosity/transparency choices usable by downstream roles. |
| Preset application | Tone/formality/empathy parameters consistent with Module-1 communication presets reference — applied to surface phrasing only. |
| Handoff | Session ready for `interviewer` / other roles without requiring model-owned submit procedures. |
