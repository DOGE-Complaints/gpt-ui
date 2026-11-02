# STORY-GPT-ROL-03 — Author roles line + strip model stash/SPA

## Meta
- **Key:** `STORY-GPT-ROL-03-author-roles-line-strip-stash`
- **Пакет:** [`req-48-role-oriented-instruction-line/`](./INDEX.md)
- **Status:** Done (Awaiting Commits)
- **Pipeline (pkg-000066):** [`STORY-GPT-ROL-03 pipeline`](../../../analysis/tasks/epics/EPIC-M1-12-req48-role-oriented-instruction-line/stories/STORY-GPT-ROL-03-author-roles-line-strip-stash/STORY-GPT-ROL-03-author-roles-line-strip-stash.md) · GIM-453…461 🟢 · P3 `2026-11-02T14:25:00Z`
- **Приоритет:** P1
- **Тип:** instruction authoring (role-oriented line)
- **Parent REQ:** [`REQ-48`](../../../requirements/REQ-48-req15-role-oriented-instruction-line-and-export.md) **AC-GPT-48-02**, **AC-GPT-48-03** · §3b
- **Tech-arch:** [`00-overview`](../../../architecture/req-48-role-oriented-instruction-line/00-overview.md) · [`01`](../../../architecture/req-48-role-oriented-instruction-line/01-git-homes-and-ownership.md) · [`02`](../../../architecture/req-48-role-oriented-instruction-line/02-export-roles-schema-and-packs.md)
- **Siblings:** [GPT-ROL-01](./STORY-GPT-ROL-01-archive-module1-snapshot.md) · [GPT-ROL-02](./STORY-GPT-ROL-02-module1-role-mapping.md) · [GPT-ROL-04](./STORY-GPT-ROL-04-export-roles-schema-parity-ops.md)
- **Skill declared:** `openai-custom-gpt-builder`
- **Depends:** GPT-ROL-02
- **Method:** [`.cursor/rules/analysis.mdc`](../../../../../.cursor/rules/analysis.mdc)

## Зачем простыми словами

Вместо одной «личности» Custom GPT пишем тексты **под роли** REQ15 + общий core. В новой линии модель **не** обязана сама стешить на gateway и слать SPA-ссылку — это сервер (REQ-11).

## Scope

- Create authoring/export home `GPT UI/instructions-roles-req15/` with shared **core** + five role files for ids: `communication_setup` · `interviewer` · `extractor` · `normalizer` · `checker` (root children only — §3b).
- Prose: purpose, bounds, result-shape expectations; methodology from mapping (no silent drop).
- New line **MUST NOT** require the model to perform gateway stash or SPA redirect (**AC-03**).

## Out of scope

- Python registry/coordinator (REQ-13). Gateway/SPA contract changes.
- Schema-pack files inside roles volume (**P1**).
- Custom GPT Knowledge as second product track (**C2**).
- Consume Pass / ASGI role wire (aibridge residual).

## Verified current state

- Sibling tree **Present** — [`GPT UI/instructions-roles-req15/`](../../../../instructions-roles-req15/) (core + five roles + README; P3).
- Grep gate **CLEAN** — no model duty strings for gateway story-draft stash / SPA submit redirect (AC-02).
- Fixture reference (not GPT UI SSOT): [`doge-ai-bridge/tests/fixtures/content/roles.manifest.json`](../../../../../doge-ai-bridge/tests/fixtures/content/roles.manifest.json) + `roles-instructions/*.md`.
- Path rule: `doge-ai-bridge/src/aibridge/roles/assembly.py` `_reject_root_child`.

## Target / Acceptance Criteria

- [x] **AC-GPT-ROL03-01** (= **AC-GPT-48-02**): Five roles + shared core Present under `instructions-roles-req15/`; documented as REQ15 authoring SSOT.
- [x] **AC-GPT-ROL03-02** (= **AC-GPT-48-03**): Review/grep gate — new line has no model duty for gateway stash URL / `POST /story-drafts` / SPA redirect procedure.

## Подзадачи (технические)

> **Delivery class этой волны story-deepen:** только постановка T01–T09. Sibling tree / role bodies **name ≠ create** until P1.3/P3.

| ID | Task | AC | Done when |
|----|------|-----|-----------|
| **T01** | Create sibling dir `GPT UI/instructions-roles-req15/` with **root children only** (no `/` in manifest paths): `core.md`, `communication_setup.md`, `interviewer.md`, `extractor.md`, `normalizer.md`, `checker.md`. Enforce consumer path rule (`assembly.py` `_reject_root_child`). | AC-01 | Six files Present as root children |
| **T02** | Author **`core.md`** from mapping rows classed `shared` (bootstrap/comm + constitution; no node field catalogs / no pack six-file embed — P1). | AC-01 | core.md Present; maps to shared |
| **T03** | Author **`communication_setup.md`** — purpose, bounds, result-shape expectations for role id `communication_setup`. | AC-01 | File Present |
| **T04** | Author **`interviewer.md`** — same for `interviewer`. | AC-01 | File Present |
| **T05** | Author **`extractor.md`** — same for `extractor`. | AC-01 | File Present |
| **T06** | Author **`normalizer.md`** — same for `normalizer`. | AC-01 | File Present |
| **T07** | Author **`checker.md`** — readiness / bounds prose OK; **MUST NOT** instruct model to perform gateway stash or SPA redirect (submit = REQ-11 code). | AC-01, AC-02 | File Present; no submit procedure |
| **T08** | **Strip / grep gate** on entire sibling: no model duty strings for `POST /story-drafts`, `postStoryDraftStash`, `#/story/submit`, stash-URL choice procedure. | AC-02 | Grep clean (or findings fixed) |
| **T09** | Sibling **`README.md`**: GPT UI is authoring SSOT; aibridge fixtures = shape reference only (**E3 rejected** as Pass oracle); packs not in this volume (P1); pointer to mapping + arch. | AC-01 | README Present |

### Out of scope (restate for T*)

- Schema-pack files inside sibling; Python coordinator; Custom GPT Knowledge rewrite as product track (C2); Consume Pass / ASGI wire.

## Locked paths / filenames (tech-arch + §3b / fixture shape)

| Role | Path |
|------|------|
| Sibling home | `GPT UI/instructions-roles-req15/` |
| Core | `…/core.md` |
| Roles | `…/communication_setup.md` · `interviewer.md` · `extractor.md` · `normalizer.md` · `checker.md` |
| Sibling README | `…/README.md` |

## Точки в репо

- `GPT UI/instructions-roles-req15/` — create on implement / P3; filenames must match later `roles.manifest.json` (ROL-04).
- Do not treat aibridge fixtures as Export Pass oracle.

## Dependencies

- Hard: GPT-ROL-02. Soft: REQ-13 §3b consumer contract.

## Related

| Artifact | Role |
|----------|------|
| REQ-48 §3b | Role ids + path rule |
| aibridge REQ-11 | Server-owned submit |
| aibridge REQ-13 | Assembly consumer |
| Arch 01 / 02 | Sibling + P1 / forbidden stash prose |
