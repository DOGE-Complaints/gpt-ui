# Activity / Amanita legacy paths — inventory (`instructions/`)

**Product:** DOGEstonia — Module 1 (Custom GPT instructions)  
**Story:** GM1-06 (EPIC-M1-01) — analyze only; no mass text replacement in this story.  
**Canonical copy for `gpt-ui` repo:** this file lives under `instructions/` (English-only policy for instruction files).

---

## Reproducible scan (AC)

Run from repository root (parent of `GPT UI/`):

```bash
rg -n --glob '*.md' -i 'activity-|Activity|Activities|Amanita|Eventify|normalized_activity|activity_type|delivery_mode|GPT UI/docs/activity|search-data-model|search-dialogue|konyrody|Коны' "GPT UI/instructions"
```

Optional (broader Russian UI strings in legacy blocks):

```bash
rg -n 'delivery_mode|Какой delivery_mode' "GPT UI/instructions"
```

Outputs depend on branch state; this document was produced against the tree at **2026-04-10**.

---

## Summary table

| Instruction file | Legacy / risk signal | Recommended action | Owner epic |
|------------------|----------------------|--------------------|------------|
| `base.md` | Title and body are **Amanita / Activity** constitution; lifecycle, `normalized_activity_payload`, `/activities` examples; references `GPT UI/docs/activity-data-model.md` (path does not match `instructions/activity-data-model.md`). | Keep for Amanita; extend **Issue INGEST overlay** (already partially present) and fix doc paths when touching file. | **M1-03** (terminology), **M1-02** (overlay), **M1-06** (HTTP examples) |
| `ingest-validation.md` | Primary role text is **Activities**; Russian prompts in strict batch; `GPT UI/docs/activity-data-model.md`; Activity §1.1 gates vs Issue overlay at top. | Continue dual-track; migrate Activity-first wording per roadmap in `instruction-modules-index.md` §7.2. | **M1-03**, **M1-02** |
| `ingest-deep-parsing.md` | Activity extraction algorithms; `GPT UI/docs/activity-data-model.md` references. | Add Issue extraction appendix or parallel section when **M1-03** defines FR mapping. | **M1-03** |
| `activity-data-model.md` | Entire file is **Activity** schema; PDF path; links to `GPT UI/docs/tasks/...` (may be stale). | Keep as **reference**; fix broken relative links when **M1-05** defines `normalized_issue_payload`. | **M1-05** |
| `activity-normalizer.md` | **Activity** JSON / `normalized_activity_payload` pattern. | Replace with **`issue-normalizer.md`** pattern per index roadmap. | **M1-05** |
| `api-orchestrator.md` | `/activities/*` table, Activity Normalizer handoff; Issue overlay exists lower in file. | Complete **Issue OpenAPI** SSOT (`issue-api-methods-reference.md` + YAML); trim Activities table when product cut. | **M1-06** |
| `api-methods-reference.md` | Activities HTTP paths (legacy SSOT until YAML). | Superseded by **`issue-api-methods-reference.md`** per epic M1-06. | **M1-06** |
| `konyrody-gate.md` | Kоны Рода / **Activities** policy gate. | Replace with **`issue-policy-gate.md`** + operator rulebook. | **M1-04** |
| `search-dialogue.md`, `search-data-model.md` | **Search** for Activities stack. | Optional for Module 1 (REQ-15); align wording if SEARCH kept. | **M1-03** |
| `safety-compliance.md` | Pipeline references **Activity Normalizer** / `normalized_activity_payload`; generic “activity” wording in examples. | Align checkpoints with **Issue** chain + `issue-data-model` when **M1-04** / **M1-05** land. | **M1-04**, **M1-03** |
| `instruction-modules-index.md` | Explicit **dual track** (Amanita reference vs Issue); roadmap renames. | Maintain as SoT for migration; **this inventory** linked from index. | **M1-03** |
| `root.md` | Amanita product identity + **DOGEstonia** overlay; mapping Activities → Issues. | Keep; update branding when **M1-04** delivers `root.md` patch scope. | **M1-04** |
| `issue-data-model.md` | Single-line comparison to `activity-data-model.md`. | OK as explicit reference. | — |
| `issue-interview-flow.md`, `issue-lifecycle-instructions.md`, `issue-api-methods-reference.md`, `bootstrap.md`, `communication-presets-reference.md` | No substantive **Activity** schema coupling in body (Issue-first or neutral). | No inventory-driven change required. | — |

---

## Critical path references (broken or confusing paths)

Several files point to **`GPT UI/docs/activity-data-model.md`**. In this repository the Activity schema under instructions is **`instructions/activity-data-model.md`** (and Issue schema is **`issue-data-model.md`**). Path cleanup is a **documentation / consistency** task — bundle with **M1-03** or a small hygiene task.

---

## Version

| Date | Change |
|------|--------|
| 2026-04-10 | Initial inventory (GM1-06); ripgrep commands + owner epics. |
