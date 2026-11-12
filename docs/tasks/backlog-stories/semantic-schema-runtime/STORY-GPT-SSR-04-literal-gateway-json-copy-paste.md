# STORY-GPT-SSR-04 — Literal gateway JSON в GPT pack (copy-paste модель)

## Meta
- **Key:** `STORY-GPT-SSR-04-literal-gateway-json-copy-paste`
- **Пакет:** [`semantic-schema-runtime/`](./INDEX.md)
- **Status:** Done
- **Приоритет:** P1 — foundation wave 2 (устранение G-FMT-01)
- **Тип:** product / architecture (pack layout + migration spec)
- **Parent REQ:** [`REQ-45-semantic-schema-intake-envelope-compatibility.md`](../../../requirements/REQ-45-semantic-schema-intake-envelope-compatibility.md)
- **Siblings:** [GPT-SSR-01…03](./INDEX.md) (Done); downstream [GPT-SSR-05](./STORY-GPT-SSR-05-taxonomy-json-pack-copy-paste.md), [GPT-SSR-09](./STORY-GPT-SSR-09-use-case-audit-core-consume-json.md)
- **Skill declared:** openai-custom-gpt-builder
- **Depends:** GPT-SSR-01, GPT-SSR-02, GPT-SSR-03
- **Gateway sibling (doc only):** loader contract [`schema-packs/README.md`](../../../../../doge-complaints-gateway/schema-packs/README.md) — SCHEMA-005
- **Analysis input:** [`cross-project-node-model-portability-gap-2026-09-01.md`](../../../../../docs/analysis/cross-project-node-model-portability-gap-2026-09-01.md) G-FMT-01
- **Coherence analysis:** [`gpt-ssr-wave2-coherence-analysis-2026-09-01.md`](../../../analysis/gpt-ssr-wave2-coherence-analysis-2026-09-01.md) §6

## Coherence analysis (2026-09-01)

Verified against disk: GPT `schema-packs/tallinn_civic/v1/` has **no** `pack.json` / `payload.schema.json`; gateway files exist. Migration map §6 of analysis doc closes T01 with path:line evidence. G-JSON-01…06 handoff to SSR-05/08/09 confirmed. Bundle clarification: 2 schema JSON + 1 conditional taxonomy JSON + 2 MD.

## Зачем простыми словами

Wave 1 решил проблему «civic размазан по core», но модель ноды в GPT всё ещё живёт в **Markdown-проекции** (`data-model.md`). Оператор при swap ноды **переписывает** gateway JSON руками — это не copy-paste, а ручной перевод. Любая правка `field_policy` или `payload.schema.json` на gateway легко **не доезжает** до GPT, и модель расходится (drift).

Цель wave 2: положить в GPT instructions **те же два JSON-файла**, что на gateway — `pack.json` и `payload.schema.json` — **байт-в-байт** (или с явным diff-check перед upload). GPT читает их как read-only reference (SEC-001: validate по-прежнему только на gateway). Markdown `data-model.md` перестаёт быть SSOT формы полей.

## Архитектура решения (целевой layout)

```text
GPT UI/instructions/schema-packs/<schema_id>/<schema_version>/
  pack.json              ← COPY from gateway (literal)
  payload.schema.json    ← COPY from gateway (literal)
  inbound-validation.md  ← stays MD (GPT-SSR-06)
  [taxonomy artifact]    ← GPT-SSR-05 (follow-up)
  data-model.md          ← DEPRECATE at P3 capstone (GPT-SSR-09); interim: slim overlay or archive
```

```text
Gateway SSOT                    GPT instruction pack
─────────────────               ────────────────────
schema-packs/<id>/<ver>/        schema-packs/<id>/<ver>/
  pack.json          ──COPY──►    pack.json
  payload.schema.json ──COPY──►  payload.schema.json
```

**SEC-001 сохраняется:** JSON в Instructions = справочник для GPT; gateway Schema Runtime = единственный исполняемый validate.

## Migration map: что сейчас только в data-model.md

Построчный inventory [`data-model.md`](../../../../instructions/schema-packs/tallinn_civic/v1/data-model.md) vs gateway JSON (indicators для T01 deep analysis):

| data-model § | Содержание | В gateway JSON? | Куда переезжает при JSON-модели |
|--------------|------------|-----------------|----------------------------------|
| §1 Binding pair | schema_id / schema_version | yes — `pack.json` L1–2 | **удалить дубль** — читать `pack.json` |
| §2.1–2.2 signals/geo shape | field list + policy | yes — `field_policy` + `payload.schema.json` | **удалить дубль** — читать JSON |
| §2.3 axis→signal map | topic_domain→civic_domain, failure_mode→failure_pattern | **no** | → **GPT-SSR-05** taxonomy JSON |
| §3 geo_intake | mode/merge/mirror + mode matrix | yes — `pack.json` L207–211 | **удалить дубль** — читать `pack.json`; decision table → core hook (already SSR-03) |
| §3.3 GEO_SCOPE copy | resident-facing text | partial — `node_clustering.civic.geo_scope` value only | copy text → **inbound-validation.md** §5 (already there) |
| §4 Geo formation / city canon | Tallinn Latin rules, location_query format | **no** | → `interview-overlay.md` or inbound § (SSR-06 decision) |
| §5 Interview phase→axes | phase evidence map + ecosystem cues | **no** | → `interview-overlay.md` (not schema shape) |
| §6 Taxonomy pointer | include story-label-taxonomy.md | **no** | → **GPT-SSR-05** taxonomy JSON |
| §7 Stash examples | demo JSON | **no** | → pack examples file or inbound appendix (SSR-09) |

## Gap indicators: что JSON **не** выразит без расширения gateway (input для SSR-05, SSR-08, SSR-09)

| Gap ID | Описание | Evidence |
|--------|----------|----------|
| G-JSON-01 | `signals.*` — string minLength:1, **нет enum** | `payload.schema.json` L12–20 |
| G-JSON-02 | axis→signal mapping отсутствует в pack | data-model §2.3 only |
| G-JSON-03 | geo **precision / detail_level** отсутствует | payload.geo — flat strings |
| G-JSON-04 | GPT **instance territory** (district-only instance) отсутствует | only `node_clustering.civic.geo_scope` |
| G-JSON-05 | taxonomy axes/keys отсутствуют в pack JSON | grep schema-packs: 0 taxonomy matches |
| G-JSON-06 | interview overlay / geo canon — prose-only | data-model §4–§5 |

## Sprawl / gap matrix (wave 1 → wave 2)

| Concern | Wave 1 (Done) | Wave 2 target |
|---------|---------------|---------------|
| Pack model artifact | `data-model.md` projection | `pack.json` + `payload.schema.json` literal copy |
| Operator swap | manual project JSON→MD ([operator manual §5](../../../runtime-docs/schema-pack-custom-model-operator-manual-ru.md)) | copy-paste files + README active pair |
| README locked paths | names `data-model.md` | names JSON files (SSR-09) |
| Core reads | «read active pack data-model» | «read pack.json + payload.schema.json» (SSR-09) |
| Drift risk G-FMT-01 | high (dual-edit) | low (single SSOT copy) |

## Scope

- T01: полная diff matrix `data-model.md` vs gateway JSON (duplicate / prose-only / missing in JSON) — таблица в story + link to analysis artifact at P3
- T02: target layout spec + SEC-001 wording для JSON в Instructions
- T03: operator swap v2 draft — copy files from gateway dir, update README, **no** Actions re-import
- T04: gap list полей/правил, не выражаемых в текущем JSON (G-JSON-01…06) — handoff SSR-05, SSR-08, SSR-09

## Out of scope

- P3 physical file moves on disk (→ capstone GPT-SSR-09)
- Core orchestrator/normalizer edits (→ GPT-SSR-09)
- Taxonomy JSON artifact (→ GPT-SSR-05)
- Geo precision / instance territory (→ GPT-SSR-08)
- Gateway loader code changes
- Marketplace / §30

## Verified current state (As-of-Done P3 — mirrors pipeline)

| Fact | Evidence |
|------|----------|
| Gateway pack exists | `doge-complaints-gateway/schema-packs/tallinn_civic/v1/pack.json` L2 `schema_id`; L6 `field_policy`; L209 `geo_intake` |
| Payload schema exists | `…/payload.schema.json` L10–13 required `civic_domain` / `failure_pattern` |
| GPT projection exists | `GPT UI/instructions/schema-packs/tallinn_civic/v1/data-model.md` v1.3 §1–§7 (interim until SSR-09) |
| GPT `pack.json` | **Present** — `cmp` vs gateway = **IDENTICAL** (P3 sync) |
| GPT `payload.schema.json` | **Present** — `cmp` vs gateway = **IDENTICAL** |
| GPT `taxonomy.json` | Present (SSR-05 territory — out of this story) |
| Operator manual | §5 v2 copy-paste checklist drafted; wave-1 projection marked Historical ([`schema-pack-custom-model-operator-manual-ru.md`](../../../runtime-docs/schema-pack-custom-model-operator-manual-ru.md) L120–132) |
| Loader contract | gateway `schema-packs/README.md` — SCHEMA-005 gateway-only |
| README locked paths | `schema-packs/README.md` L23–26 — still MD pair (SSR-09 rename) |
| Migration evidence artifact | [`gpt-ssr-04-migration-map-evidence-2026-09-01.md`](../../../analysis/gpt-ssr-04-migration-map-evidence-2026-09-01.md) |

## Подзадачи

| ID | Задача | DoD |
|----|--------|-----|
| **T01** | Diff matrix data-model vs JSON | Таблица migration map (§ выше расширена); каждый row с path evidence |
| **T02** | Target layout + SEC-001 | Locked paths documented; JSON = read-only in Instructions |
| **T03** | Operator swap v2 draft | Checklist: copy 2 JSON + README; no Actions re-import; diff-check optional step |
| **T04** | JSON expressibility gaps | G-JSON-01…06 table; explicit handoff IDs to SSR-05/08/09 |

## Target / Acceptance Criteria

- [x] **AC-GPT-SSR04-01:** migration map complete — every data-model § has fate (copy / move / deprecate).
- [x] **AC-GPT-SSR04-02:** target layout locked — `pack.json` + `payload.schema.json` in GPT pack dir.
- [x] **AC-GPT-SSR04-03:** operator swap v2 checklist drafted (copy-paste, not project).
- [x] **AC-GPT-SSR04-04:** G-JSON gap list published for downstream stories.
- [x] SEC-001 preserved — no «validate in GPT» wording.

## Точки в docs

| Зона | Path |
|------|------|
| Current projection | `GPT UI/instructions/schema-packs/tallinn_civic/v1/data-model.md` |
| Gateway SSOT | `doge-complaints-gateway/schema-packs/tallinn_civic/v1/` |
| Operator manual | `GPT UI/docs/runtime-docs/schema-pack-custom-model-operator-manual-ru.md` §3–§5 |
| Gap analysis | `docs/analysis/cross-project-node-model-portability-gap-2026-09-01.md` |

## Dependencies

- GPT-SSR-01…03 Done.
- Blocks GPT-SSR-05 (taxonomy needs layout decision from T02).
- Parallel safe with GPT-SSR-06, GPT-SSR-07.

## Residual

- Fate of `data-model.md`: deprecate vs slim `interview-overlay.md` — decide at SSR-06 + capstone SSR-09.
- Byte-identical copy vs «semantic equal» diff-check tool — operator choice at P3; default = byte-identical.
