# STORY-GPT-SSR-11 — Flat `schema-packs.*` prefix layout (replace nested)

## Meta
- **Key:** `STORY-GPT-SSR-11-flat-schema-packs-prefix-layout`
- **Пакет:** [`semantic-schema-runtime/`](./INDEX.md)
- **Status:** Done
- **Приоритет:** P1 — Wave 4 foundation (REQ-45a §5–§6)
- **Тип:** product / instruction layout migration
- **Parent REQ:** [`REQ-45a-pack-prose-identities-and-flat-layout.md`](../../../requirements/REQ-45a-pack-prose-identities-and-flat-layout.md) · parent [`REQ-45`](../../../requirements/REQ-45-semantic-schema-intake-envelope-compatibility.md)
- **Siblings:** upstream GPT-SSR-09/10 Done; downstream [GPT-SSR-12](./STORY-GPT-SSR-12-locale-jurisdiction-pack-identity.md), [GPT-SSR-13](./STORY-GPT-SSR-13-core-retarget-flat-pack-paths.md)
- **Skill declared:** openai-custom-gpt-builder
- **Depends:** GPT-SSR-09 Done · GPT-SSR-10 Done
- **Blocks:** GPT-SSR-12, GPT-SSR-13

## Зачем простыми словами

Custom GPT upload **отбрасывает субдиректории**. Wave 2 положил instance pack в `instructions/schema-packs/<id>/<ver>/` — для git удобно, для upload — нет. Цель: **заменить** nested tree на **плоские** файлы с префиксом `schema-packs.` прямо под `GPT UI/instructions/`.

Эта story **только** layout migrate существующих артефактов (JSON×3 + inbound + interview-overlay). Новый `locale-jurisdiction` — [SSR-12](./STORY-GPT-SSR-12-locale-jurisdiction-pack-identity.md) **после** flat. Core pointer retarget — [SSR-13](./STORY-GPT-SSR-13-core-retarget-flat-pack-paths.md).

## Целевое состояние (locked paths)

```text
GPT UI/instructions/
  schema-packs.tallinn_civic.v1.pack.json
  schema-packs.tallinn_civic.v1.payload.schema.json
  schema-packs.tallinn_civic.v1.taxonomy.json
  schema-packs.tallinn_civic.v1.inbound-validation.md
  schema-packs.tallinn_civic.v1.interview-overlay.md
  schema-packs.README.md          ← thin active-pair index → flat names only (no nested segments)

REMOVED (replace, not mirror):
  schema-packs/tallinn_civic/v1/*   ← nested instance tree no longer upload SSOT
```

Naming pattern (REQ-45a §5): `schema-packs.<schema_id>.<schema_version>.<artifact>`.

Gateway `doge-complaints-gateway/schema-packs/<id>/<ver>/` **MAY** remain nested (executable SSOT).

## Scope

- T01: Copy/migrate five existing nested artifacts → flat filenames under `GPT UI/instructions/` (byte-identical for JSON).
- T02: Rewrite `schema-packs/README.md` as active-pair index listing **only** flat paths (pattern above).
- T03: **Replace** nested instance tree — remove or archive-out of live Instructions upload set; no parallel nested SSOT.
- T04: Operator note in story DoD: locale file **not** in this story (SSR-12).

## Out of scope

- Creating `locale-jurisdiction.md` (→ SSR-12).
- Retargeting core Module-1 process files beyond README index (→ SSR-13).
- Pack Builder / `node-onboarding/` (REQ-46).
- Gateway nested layout change.
- Touching instructions during **story authoring** — P3 only after go-ahead.

## Verified current state (as-built — start of P3)

| Fact | Evidence |
|------|----------|
| Nested five artifacts present | `GPT UI/instructions/schema-packs/tallinn_civic/v1/` — pack.json, payload.schema.json, taxonomy.json, inbound-validation.md, interview-overlay.md |
| README still nested locked paths | `schema-packs/README.md` L21–31 |
| Flat `schema-packs.*` filenames | **Absent** under `GPT UI/instructions/` |
| locale-jurisdiction | **Absent** (SSR-12) |

## Подзадачи

| ID | Задача | DoD |
|----|--------|-----|
| **T01** | Flat materialize ×5 | Files exist at target paths; JSON cmp vs nested/gateway source |
| **T02** | README active pair | Lists flat names only; no `tallinn_civic/v1/` segments for live set |
| **T03** | Replace nested | Nested instance dir not required for upload; remove or non-live archive |
| **T04** | Handoff SSR-12/13 | Story AC notes locale + core retarget deferred |

## Target / Acceptance Criteria

- [x] **AC-GPT-SSR11-01:** Five flat files present: `schema-packs.tallinn_civic.v1.{pack.json,payload.schema.json,taxonomy.json,inbound-validation.md,interview-overlay.md}` (AC-GPT-REQ45a-03 partial — locale = SSR-12).
- [x] **AC-GPT-SSR11-02:** Active-pair index uses flat names only — no nested path segments for instance pack (AC-GPT-REQ45a-03).
- [x] **AC-GPT-SSR11-03:** Nested `schema-packs/<id>/<ver>/` **replaced** as live upload SSOT (not kept as parallel mirror).
- [x] **AC-GPT-SSR11-04:** Gateway nested packs unchanged (AC-GPT-REQ45a scope).
- [x] **AC-GPT-SSR11-05:** SEC-001 — JSON still read-only reference; no validate-in-GPT wording.

## Точки в docs (P3)

| Action | Path |
|--------|------|
| CREATE | `GPT UI/instructions/schema-packs.tallinn_civic.v1.pack.json` (+ four siblings) |
| UPDATE | `GPT UI/instructions/schema-packs/README.md` |
| REMOVE/ARCHIVE | `GPT UI/instructions/schema-packs/tallinn_civic/v1/` instance files |

## Dependencies

- GPT-SSR-09/10 Done.
- **Blocks** GPT-SSR-12 (locale at flat path), GPT-SSR-13 (core retarget).

## Residual

- Sixth file `schema-packs.tallinn_civic.v1.locale-jurisdiction.md` — SSR-12.
- Core/orchestrator/normalizer link retarget — SSR-13.
