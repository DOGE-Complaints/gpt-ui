# STORY-GPT-SSR-13 — Core / manuals retarget to flat `schema-packs.*`

## Meta
- **Key:** `STORY-GPT-SSR-13-core-retarget-flat-pack-paths`
- **Пакет:** [`semantic-schema-runtime/`](./INDEX.md)
- **Status:** Done (Awaiting Commits)
- **Приоритет:** P1 — Wave 4 capstone (REQ-45a §5 root retarget · §7)
- **Тип:** product / instruction pointer migration
- **Parent REQ:** [`REQ-45a`](../../../requirements/REQ-45a-pack-prose-identities-and-flat-layout.md)
- **Siblings:** [GPT-SSR-11](./STORY-GPT-SSR-11-flat-schema-packs-prefix-layout.md), [GPT-SSR-12](./STORY-GPT-SSR-12-locale-jurisdiction-pack-identity.md)
- **Skill declared:** openai-custom-gpt-builder
- **Depends:** **GPT-SSR-11** Done · **GPT-SSR-12** Done

## Зачем простыми словами

После flat replace (SSR-11) и locale file (SSR-12) Module-1 core всё ещё может ссылаться на nested `schema-packs/<id>/<ver>/` или на README без `locale-jurisdiction`. Capstone: **все** live Instruction links на instance pack → **flat** `schema-packs.*` имена; active set = JSON×3 + три prose identities; SEC-001 / AC-016 сохранены.

## Целевое состояние (locked)

```text
Active set (read via schema-packs/README.md or direct flat names):
  schema-packs.tallinn_civic.v1.pack.json
  schema-packs.tallinn_civic.v1.payload.schema.json
  schema-packs.tallinn_civic.v1.taxonomy.json
  schema-packs.tallinn_civic.v1.inbound-validation.md
  schema-packs.tallinn_civic.v1.interview-overlay.md
  schema-packs.tallinn_civic.v1.locale-jurisdiction.md

Core process modules: path segments schema-packs/<id>/<ver>/  →  FORBIDDEN in live Instructions
Operator manual swap: copy flat six-file set (not nested dir)
```

## Scope

- T01: Grep inventory live Instructions + operator/runtime docs for nested pack paths / incomplete artifact lists.
- T02: Retarget orchestrator, normalizer, interview-flow, ingest-validation, instruction-modules-index, story-data-model, taxonomy stub, operator manual — flat names + include locale.
- T03: Confirm no civic domain lists welded into core (AC-016); JSON read-only (SEC-001).
- T04: Grep gate — zero live required nested instance paths.

## Out of scope

- Re-authoring locale/overlay body content (SSR-12/prior).
- Re-doing flat file copy (SSR-11).
- Pack Builder prompts (REQ-46).
- OpenAPI / new wire fields.
- spa / gateway code.

## Verified current state (pre-SSR-11 — nested; P3 starts after SSR-11/12)

| Module | Current pattern | Evidence (sample) |
|--------|-----------------|-------------------|
| `api-orchestrator.md` | README → nested artifact names | L353 |
| `story-normalizer.md` | README + overlay/pack.json | L32, L409–422 |
| `story-interview-flow.md` | README + overlay | L172–187 |
| `instruction-modules-index.md` | wave-2 bundle **without** locale | L29 |
| `story-label-taxonomy.md` stub | nested taxonomy path | L5 |
| Operator manual | nested paths | `schema-pack-custom-model-operator-manual-ru.md` |

After SSR-11/12: flat files + locale exist; this story only **retargets pointers**.

## Подзадачи

| ID | Задача | DoD |
|----|--------|-----|
| **T01** | Path inventory | Table file:line nested vs flat target |
| **T02** | Core + manuals retarget | All live links → flat `schema-packs.*`; locale in active list |
| **T03** | Invariants | SEC-001 + AC-016 checklist |
| **T04** | Grep gate | No live `schema-packs/<id>/<ver>/` for active upload set |

## Target / Acceptance Criteria

- [x] **AC-GPT-SSR13-01:** Root Instruction links use flat `schema-packs.*` only (AC-GPT-REQ45a-04).
- [x] **AC-GPT-SSR13-02:** Active set includes three prose identities + JSON trio (AC-GPT-REQ45a-02).
- [x] **AC-GPT-SSR13-03:** Operator manual documents flat six-file swap.
- [x] **AC-GPT-SSR13-04:** Grep: no live Instructions require nested instance tree (archive/historical OK).
- [x] **AC-GPT-SSR13-05:** No invented wire names; SEC-001 / narrative-first / AC-016 preserved (AC-GPT-REQ45a-06).

## Точки в docs (P3)

| Zone | Paths (indicators) |
|------|-------------------|
| Core | `api-orchestrator.md`, `story-normalizer.md`, `story-interview-flow.md`, `ingest-validation.md`, `instruction-modules-index.md`, `story-data-model.md`, `story-label-taxonomy.md` stub |
| Index | `schema-packs/README.md` |
| Manuals | `docs/runtime-docs/schema-pack-custom-model-operator-manual-ru.md` (+ related as-built docs if they cite nested paths) |

## Dependencies

- GPT-SSR-11 Done (flat files replace nested).
- GPT-SSR-12 Done (locale flat file present).

## Residual

- REQ-45a Status → Accepted only after Wave 4 Done + operator PA.
- REQ-46 must emit same flat set — separate package.
- Historical backlog/REQ prose citing nested paths — MAY leave with superseded note.
