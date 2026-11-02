# STORY-GPT-ROL-02 — Module-1 → role / archive / server mapping

## Meta
- **Key:** `STORY-GPT-ROL-02-module1-role-mapping`
- **Пакет:** [`req-48-role-oriented-instruction-line/`](./INDEX.md)
- **Status:** Done (Awaiting Commits)
- **Приоритет:** P1
- **Тип:** mapping deliverable (methodology preservation)
- **Parent REQ:** [`REQ-48`](../../../requirements/REQ-48-req15-role-oriented-instruction-line-and-export.md) **AC-GPT-48-01** · parent AC-11 intent
- **Tech-arch:** [`00-overview`](../../../architecture/req-48-role-oriented-instruction-line/00-overview.md) · [`02-export-roles-schema-and-packs`](../../../architecture/req-48-role-oriented-instruction-line/02-export-roles-schema-and-packs.md) (**P1** packs)
- **Siblings:** [GPT-ROL-01](./STORY-GPT-ROL-01-archive-module1-snapshot.md) · [GPT-ROL-03](./STORY-GPT-ROL-03-author-roles-line-strip-stash.md) · [GPT-ROL-04](./STORY-GPT-ROL-04-export-roles-schema-parity-ops.md)
- **Skill declared:** `openai-custom-gpt-builder`
- **Depends:** GPT-ROL-01
- **Pipeline (pkg-000065):** [`STORY-GPT-ROL-02 pipeline`](../../../analysis/tasks/epics/EPIC-M1-12-req48-role-oriented-instruction-line/stories/STORY-GPT-ROL-02-module1-role-mapping/STORY-GPT-ROL-02-module1-role-mapping.md) · GIM-447…451 🟢 · GIM-452 🟢 · P3 `2026-11-02T13:25:46Z`
- **Method:** [`.cursor/rules/analysis.mdc`](../../../../../.cursor/rules/analysis.mdc)

## Зачем простыми словами

Нужна явная карта: какой кусок старой книги уходит в shared core, какую роль, что остаётся только в архиве, а что (stash/SPA) уходит в серверный код bridge — без тихой потери методики.

## Scope

- Mapping table: every active interview Module-1 module + **active schema pack** → `shared` | `role(s)` (ids §3b) | `archive-only` | `server-only`.
- Explicit pack row(s) under arch **P1** (packs outside roles volume — no silent drop).
- Classify `api-orchestrator` stash/SPA duties as **server-only** (peer REQ-11).
- Deliverable path (default): [`GPT UI/docs/analysis/req-48-module1-role-mapping.md`](../../../analysis/req-48-module1-role-mapping.md) — **Present** (P3).

## Out of scope

- Writing full role file bodies (ROL-03).
- Implementing aibridge handlers / dual-confirm.
- Inventing joint consume hash wire.

## Verified current state

- One-personality chain: REQ-48 §3 · as-built [`01-module1-instruction-layers.md`](../../../architecture/01-module1-instruction-layers.md).
- Model-owned stash+SPA still in [`api-orchestrator.md`](../../../../instructions/api-orchestrator.md) §5.2 / §5.2.B (REQ-48 §3); duties classified **server-only** in mapping (AC-02).
- Mapping table **Present** — [`GPT UI/docs/analysis/req-48-module1-role-mapping.md`](../../../analysis/req-48-module1-role-mapping.md) (P3 · GIM-447…451 · 24/24 Module-1 + pack P1 rows).

## Target / Acceptance Criteria

- [x] **AC-GPT-ROL02-01** (= **AC-GPT-48-01**): Mapping table Present; covers Module-1 + active pack; no silent methodology drop.
- [x] **AC-GPT-ROL02-02:** Stash/SPA redirect classified server-only; pack classification explicit (P1).

## Подзадачи (технические)

> **Delivery class этой волны story-deepen:** только постановка T01–T05. Mapping file **name ≠ create** until P1.3/P3.

| ID | Task | AC | Done when |
|----|------|-----|-----------|
| **T01** | For **every** file in ROL-01 T01 inventory: one mapping row → `shared` \| `role(s)` (ids: `communication_setup` · `interviewer` · `extractor` · `normalizer` · `checker`) \| `archive-only` \| `server-only`. Baseline = snapshot tree (arch: prefer snapshot over drifting working). | AC-01 | Table rows complete; no inventory file without a class |
| **T02** | **P1 pack rows:** active six-file + `schema-packs.README.md` classified explicitly (typical: archive-only / Custom GPT continuity / future bridge). **MUST NOT** place packs in roles export volume. No silent drop of pack methodology. | AC-01, AC-02 | Pack rows Present; volume note = outside roles-schema |
| **T03** | Classify [`api-orchestrator.md`](../../../../instructions/api-orchestrator.md) §5.2 / §5.2.B stash + SPA redirect duties as **server-only** (peer REQ-11). Under **C2**, rewriting as-was orchestrator is **not** required for this story’s DoD. | AC-02 | Rows/mark server-only; C2 note in mapping doc |
| **T04** | Write deliverable at locked path [`GPT UI/docs/analysis/req-48-module1-role-mapping.md`](../../../analysis/req-48-module1-role-mapping.md): table + legend + links to snapshot / REQ-48 / arch 02. | AC-01 | File Present on implement |
| **T05** | **Gate:** every interview-relevant module + active pack classified; methodology preservation intent (parent AC-11) — no silent drop without archive-only or server-only decision. | AC-01, AC-02 | Checklist Pass |

### Classification vocabulary (normative for T01)

| Class | Meaning |
|-------|---------|
| `shared` | Goes into sibling `core.md` (or shared prose reused by roles) |
| `role(s)` | One or more §3b role ids |
| `archive-only` | Kept in snapshot / as-was; not authored into new line |
| `server-only` | Duty moves to aibridge/gateway code (not instruction prose) |

## Locked paths (tech-arch)

| Role | Path |
|------|------|
| Mapping deliverable | `GPT UI/docs/analysis/req-48-module1-role-mapping.md` |
| Baseline input | `GPT UI/instructions-archive-module1-req48/` (after ROL-01) |

## Точки в репо

- Inputs: snapshot (ROL-01) + `instruction-modules-index.md` + `schema-packs.README.md` active pair.
- Output: `GPT UI/docs/analysis/req-48-module1-role-mapping.md`.

## Dependencies

- Hard: GPT-ROL-01. Soft: peer REQ-11 owns submit runtime.

## Related

| Artifact | Role |
|----------|------|
| REQ-48 §2 / AC-01 | Mapping deliverable |
| aibridge REQ-11 | server-only stash/SPA |
| Arch 02 P1 | Packs outside roles volume |
