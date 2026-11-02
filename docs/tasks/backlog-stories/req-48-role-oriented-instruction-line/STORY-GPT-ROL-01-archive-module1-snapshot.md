# STORY-GPT-ROL-01 — Archive Module-1 snapshot (REQ-48)

## Meta
- **Key:** `STORY-GPT-ROL-01-archive-module1-snapshot`
- **Пакет:** [`req-48-role-oriented-instruction-line/`](./INDEX.md)
- **Status:** Done (Awaiting Commits)
- **Приоритет:** P1 (REQ-48 prerequisite — archive first)
- **Тип:** content archive (instruction tree snapshot)
- **Parent REQ:** [`REQ-48`](../../../requirements/REQ-48-req15-role-oriented-instruction-line-and-export.md) **AC-GPT-48-04**
- **Tech-arch:** [`../../../architecture/req-48-role-oriented-instruction-line/00-overview.md`](../../../architecture/req-48-role-oriented-instruction-line/00-overview.md) · [`01-git-homes-and-ownership.md`](../../../architecture/req-48-role-oriented-instruction-line/01-git-homes-and-ownership.md) (B2, N2, C2)
- **Siblings:** [GPT-ROL-02](./STORY-GPT-ROL-02-module1-role-mapping.md) · [GPT-ROL-03](./STORY-GPT-ROL-03-author-roles-line-strip-stash.md) · [GPT-ROL-04](./STORY-GPT-ROL-04-export-roles-schema-parity-ops.md)
- **Skill declared:** `openai-custom-gpt-builder` (`.cursor/skills/openai-custom-gpt-builder/SKILL.md`)
- **Depends:** —
- **Pipeline (pkg-000064):** [`STORY-GPT-ROL-01 pipeline`](../../../analysis/tasks/epics/EPIC-M1-12-req48-role-oriented-instruction-line/stories/STORY-GPT-ROL-01-archive-module1-snapshot/STORY-GPT-ROL-01-archive-module1-snapshot.md) · GIM-442…445 🟢 · P3 `2026-11-02T13:02:47Z`
- **Method:** [`.cursor/rules/analysis.mdc`](../../../../../.cursor/rules/analysis.mdc)

## Зачем простыми словами

Пока пишется новая ролевая линия, старая «книга» Module-1 должна остаться доступной для сверки и Custom GPT (continuity). Делаем **снимок-копию** as-was в отдельный дом — ничего ценного не затираем.

## Scope

- Snapshot-copy interview-relevant as-was Module-1 (+ active pack used in runtime) into `GPT UI/instructions-archive-module1-req48/` (arch **N2** / **B2**).
- Keep `GPT UI/instructions/` readable (**C2** continuity); do not delete as-was.
- Record inventory of what was copied (gate note / short README in archive home).

## Out of scope

- New role texts; mapping table body; Python/aibridge; Actions OpenAPI archive; Telegram.
- Empty stub trees without real copy content.
- Using existing `instructions/archive/` (nested packs artifact) as the Module-1 REQ-48 snapshot.

## Verified current state

- As-was SSOT: `GPT UI/instructions/` · [`instruction-modules-index.md`](../../../../instructions/instruction-modules-index.md) (REQ-48 §3).
- `GPT UI/instructions/archive/` exists but is **not** a full Module-1 snapshot (arch `01`).
- Target path `instructions-archive-module1-req48/` **Absent** until implement (REQ-48 §3; name ≠ create).

## Target / Acceptance Criteria

- [x] **AC-GPT-ROL01-01** (= **AC-GPT-48-04**): Archive tree Present at `GPT UI/instructions-archive-module1-req48/` and reachable.
- [x] **AC-GPT-ROL01-02:** Working `GPT UI/instructions/` not deleted; C2 continuity preserved.
- [x] **AC-GPT-ROL01-03:** Inventory of copied set recorded (modules + active pack files).

## Подзадачи (технические)

> **Delivery class этой волны story-deepen:** только постановка T01–T04 ниже. **Не** создавать archive tree / README bodies, пока оператор не закажет P1.3/P3 execute (name ≠ create).

| ID | Task | AC | Done when |
|----|------|-----|-----------|
| **T01** | **Inventory set:** из [`instruction-modules-index.md`](../../../../instructions/instruction-modules-index.md) — wrappers (`root.md`, `bootstrap.md`, `base.md`, `communication-presets-reference.md`) + Story/Issue modules (`story-*`, `ingest-*`, `safety-compliance.md`, `api-orchestrator.md`, `story-api-methods-reference.md`, index itself) + flat active pack six-file + `schema-packs.README.md` для active pair **`uus_veerenni_civic` / `v3`** ([`schema-packs.README.md`](../../../../instructions/schema-packs.README.md)). **Exclude:** `node-onboarding/`. **Orthogonal (do not treat as Module-1 snapshot root):** existing `instructions/archive/`. | AC-03 inventory | Written inventory list (in gate / later README) matches these rules |
| **T02** | **B2 snapshot copy** of T01 set → `GPT UI/instructions-archive-module1-req48/` (arch N2). **MUST NOT** delete or empty working `GPT UI/instructions/` (C2). | AC-01, AC-02 | Dest tree Present; working tree still Present |
| **T03** | **Archive `README.md`** at `instructions-archive-module1-req48/README.md`: UTC date of snapshot, source note, full file inventory from T01, C2 continuity note (Custom GPT may still use working `instructions/`). | AC-03 | README Present with inventory |
| **T04** | **Gate:** dest Present + reachable; working intact; inventory in README ⊆/== T01 set; no reliance on `instructions/archive/` as Module-1 snapshot. | AC-01…03 | Checklist Pass |

## Locked paths (tech-arch)

| Role | Path |
|------|------|
| Snapshot home | `GPT UI/instructions-archive-module1-req48/` |
| Snapshot README | `GPT UI/instructions-archive-module1-req48/README.md` |
| Working as-was | `GPT UI/instructions/` (unchanged ownership) |

## Точки в репо

- Source: `GPT UI/instructions/**` (list from index + active pair in `schema-packs.README.md`).
- Dest: `GPT UI/instructions-archive-module1-req48/` (create on implement / P3).

## Dependencies

- Soft: tech-arch Present. Hard: none.

## Related

| Artifact | Role |
|----------|------|
| REQ-48 | Parent AC-04 |
| Arch 00 / 01 | Path + B2/C2 locks |
