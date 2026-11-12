# STORY-GPT-SSR-01 — Core vs two swappable pack artifacts

## Meta
- **Key:** `STORY-GPT-SSR-01-core-vs-swappable-packs`
- **Пакет:** [`semantic-schema-runtime/`](./INDEX.md)
- **Status:** Done
- **Приоритет:** P1 — foundation for SSR-02/03
- **Тип:** product / architecture (instruction core + pack projections)
- **Parent REQ:** [`REQ-45-semantic-schema-intake-envelope-compatibility.md`](../../../requirements/REQ-45-semantic-schema-intake-envelope-compatibility.md)
- **Siblings:** [GPT-SSR-02](./STORY-GPT-SSR-02-openapi-schema-binding-geo-detail.md), [GPT-SSR-03](./STORY-GPT-SSR-03-instructions-emit-binding-and-geo.md)
- **Skill declared:** openai-custom-gpt-builder
- **Parent tokens:** REQ3-GPT-001/002, REQ3-AC-016; §30 out of scope
- **Sibling gateway:** [`51-semantic-schema-runtime.md`](../../../../../doge-complaints-gateway/docs/requirements/51-semantic-schema-runtime.md)
- **Locked pack layout (closes REQ-45 §6.1):**
  - `GPT UI/instructions/schema-packs/<schema_id>/<schema_version>/data-model.md`
  - `GPT UI/instructions/schema-packs/<schema_id>/<schema_version>/inbound-validation.md`
  - `GPT UI/instructions/schema-packs/README.md` — active pair
- **First instance:** `tallinn_civic` / `v1` ← gateway [`schema-packs/tallinn_civic/v1/`](../../../../../doge-complaints-gateway/schema-packs/tallinn_civic/v1/)
- **Re-import policy:** pack `.md` only → upload Instructions (no Actions re-import required). OpenAPI change → SSR-02 / Actions re-import.

## Зачем простыми словами

Один Custom GPT на все ноды: процесс интервью и handoff не должны знать имена полей конкретной civic-модели. Модель узла и inbound-validation живут в **двух сменных pack-файлах**; civic — один экземпляр пака. Сейчас taxonomy, ISSUE_TYPE, Estonia-геоканон и ecosystem cues **размазаны** по core — это ломает взаимозаменяемость нод.

## Архитектура решения

```text
┌─────────────────────────────────────────────────────────────┐
│ Instruction CORE (node-agnostic)                            │
│  interview phases · handoff/stash · PII STOP · envelope id  │
│  emit plumbing hooks: «read active pack», «apply geo_intake»│
└────────────┬───────────────────────────────┬────────────────┘
             │ overlay                       │ overlay
             ▼                               ▼
┌────────────────────────┐    ┌──────────────────────────────┐
│ Pack (1) data-model.md │    │ Pack (2) inbound-validation  │
│ payload / field_policy │    │ mission/policy domain checks │
│ geo_intake + geo canon │    │ label enum / evidence lists  │
│ interview clustering   │    │ completeness vs pack fields  │
└────────────────────────┘    └──────────────────────────────┘
             │
             ▼  (operator swap = replace two files + binding pair)
   Active pair in schema-packs/README.md = NODE_SCHEMA_* mirror
```

| Layer | Responsibility | Must NOT contain |
|-------|----------------|------------------|
| **Core** | Process, privacy, envelope `m2.story_intake_envelope.v2`, generic gates | Civic axis names, Estonia city tables, ISSUE_TYPE weld as sole model |
| **data-model pack** | Projection of gateway pack: `field_policy`, `geo_intake`, payload field shapes, geo formation rules, interview notes for exact/civic clustering | Executable validate; invent OpenAPI names |
| **inbound-validation pack** | Domain admission / label checklists / multi-axis evidence (REQ3-GPT-002) | HTTP Actions schemas |

**Gateway remains authoritative** (SEC-001): GPT packs are **non-executable projections**. Validate = gateway Runtime.

## Sprawl / gap matrix (verified)

| Concern | Path (evidence) | Today | Target |
|---------|-----------------|-------|--------|
| 13-axis civic vocabulary | [`story-label-taxonomy.md`](../../../../instructions/story-label-taxonomy.md) | Pack-like but treated as core | **data-model** (or keep file, pointer from pack) |
| ISSUE_TYPE / labels / institution card | [`story-data-model.md`](../../../../instructions/story-data-model.md) §4–§5 | Welded in core | **data-model** |
| `canonical_payload` type/taxonomy weld | [`story-normalizer.md`](../../../../instructions/story-normalizer.md) §4.1 / §4.1a | Civic hints in core | **data-model** |
| Estonia city canon + `location_query` | normalizer §4.6 L414–433 | Hardcoded in core | **data-model** (`geo_intake` + formation) |
| Phase→axes + ecosystem cues | [`story-interview-flow.md`](../../../../instructions/story-interview-flow.md) L170–207 | Civic in interview core | **data-model** notes; phases stay core |
| Label enum / multi-axis evidence | [`ingest-validation.md`](../../../../instructions/ingest-validation.md) | Domain lists in core | **inbound-validation** |
| Mission `IRRELEVANT_NON_CIVIC` | [`story-policy-gate.md`](../../../../instructions/story-policy-gate.md) L111–120 | Civic mission in core | **inbound-validation** |
| Envelope + stash plumbing | [`api-orchestrator.md`](../../../../instructions/api-orchestrator.md) L294, L350 | OK | **core** |
| PII scrub | normalizer §4.4 | OK | **core** |
| GPT `schema_binding` / pack dirs | GPT UI | **Absent** | SSR-01 creates layout; SSR-02/03 wire/emit |
| Gateway civic SSOT | `schema-packs/tallinn_civic/v1/pack.json` L1–2, L207–211 `geo_intake.mode=optional` | Exists | Projection source |

## Scope

- Зафиксировать core vs два pack overlay (пути Locked выше).
- Civic instance = проекция `tallinn_civic/v1` (`pack.json` + `payload.schema.json`).
- Operator swap runbook + AC-016 checklist.
- Content outlines для двух pack files (materialize files = P3 DoD, не эта backlog-правка).
- Закрыть REQ-45 §6.1 (paths больше не Unknown).

## Out of scope

- Schema Runtime / validate в GPT (gateway).
- OpenAPI wire → GPT-SSR-02; emit behavior → GPT-SSR-03.
- Materialize pack `.md` на диске в этой story-write (spec only until P3).
- Marketplace / §30; WASM; SPA; identity; REQ-37/44.
- Loosen OpenAPI `NarrativeTaxonomy` 13 axes (Residual → SSR-02).

## Verified current state

| Fact | Evidence |
|------|----------|
| Envelope v2 in OpenAPI | [`custom-gpt-story-intake-actions.openapi.yaml`](../../../custom-gpt-story-intake-actions.openapi.yaml) L253–255 |
| Orchestrator hard-codes v2 | [`api-orchestrator.md`](../../../../instructions/api-orchestrator.md) L294, L350 |
| Civic taxonomy file exists | [`story-label-taxonomy.md`](../../../../instructions/story-label-taxonomy.md) |
| GPT `schema-packs/` directory | **Not found** (create in P3 per T02) |
| `schema_binding` / `geo_detail` in GPT OpenAPI | **Not found** |
| Gateway pack identity | `tallinn_civic` / `v1` — [`pack.json`](../../../../../doge-complaints-gateway/schema-packs/tallinn_civic/v1/pack.json) L1–2 |
| Gateway `geo_intake` civic | same file L207–211: `optional`, `merge: true`, `mirror_to_payload: true` |
| Binding required on node | [`contracts.py`](../../../../../doge-complaints-gateway/src/core/intake/contracts.py) L393+ |

## Подзадачи

| ID | Задача | DoD |
|----|--------|-----|
| **T01** | Inventory matrix: concern \| path:line \| core \| pack1 \| pack2 | Таблица в этой story (Sprawl) полная; каждый claim с path |
| **T02** | Layout + README active pair | На P3: создать `schema-packs/README.md` с active `tallinn_civic`/`v1`; dirs `<id>/<ver>/`; stub или full pack files |
| **T03** | Civic `data-model.md` content | Outline/P3 file mirrors gateway `field_policy`, `geo_intake`, `payload.schema.json` signals/geo; §4.6 Estonia canon **переезжает сюда** |
| **T04** | Civic `inbound-validation.md` content | Outline/P3: policy-gate civic rules, ingest label checks, completeness vs pack required fields |
| **T05** | Core seams table | Orchestrator/interview/normalizer hooks без civic field names: «read active pack», «apply geo_intake.mode» |
| **T06** | Operator swap runbook | Replace two files under new `<id>/<ver>/` + align emitted binding pair; AC-016 checklist rows |
| **T07** | Story acceptance | Paths closed; civic = instance; §30 out; SEC-001 non-executable; handoff SSR-02/03 |

### T03 content outline (civic data-model)

1. Binding pair: `schema_id=tallinn_civic`, `schema_version=v1`.
2. `structured_payload` shape from [`payload.schema.json`](../../../../../doge-complaints-gateway/schema-packs/tallinn_civic/v1/payload.schema.json) — `signals.*` (required `civic_domain`, `failure_pattern` per `field_policy`).
3. `geo_intake`: mode/merge/mirror from pack.json L207–211.
4. Geo formation: move normalizer §4.6 city canon + Latin preference here.
5. Interview notes: phase→axes map + ecosystem recognition (from interview-flow L170–207) as pack overlay text.
6. Taxonomy pointer: SSOT labels — pack-owned or explicit `@story-label-taxonomy` include until migrated.

### T04 content outline (civic inbound-validation)

1. Mission / `IRRELEVANT_NON_CIVIC` / standing rules from policy-gate (domain).
2. Label enum validation checklists from ingest-validation.
3. Completeness: required `signals.*` before stash (pack field_policy).
4. No leading taxonomy questions (privacy baseline) — keep as validation guidance.

### T05 core seams (must remain)

| Hook | Module | Behavior |
|------|--------|----------|
| Envelope id | orchestrator / OpenAPI | Always `m2.story_intake_envelope.v2` |
| Active pair | README | Read `schema_id`/`schema_version` for binding |
| Geo gate shape | orchestrator pre-flight | Parameterized by pack `geo_intake.mode` (SSR-03) |
| PII / admission process | normalizer / orchestrator | Unchanged process |
| Phase 7 affirmation process | interview-flow | Process stays; field lists from pack |

## Target / Acceptance Criteria

- [x] **AC-GPT-REQ3-01:** core vs two pack artifacts documented; civic = pack instance at locked paths.
- [x] **AC-GPT-REQ3-16:** swap pack does not require core domain-field edits (T05+T06 checklist).
- [x] Paths recorded (Meta + T02) — REQ-45 §6.1 closed.
- [x] Packs non-executable; gateway authoritative (SEC-001).
- [x] §30 marketplace extraction not required.
- [x] T01–T07 DoD met (files on disk at P3 for T02–T04).

## Точки в коде / docs

| Зона | Path |
|------|------|
| Pack layout (target) | `GPT UI/instructions/schema-packs/` |
| Sprawl sources | taxonomy, data-model, normalizer, interview-flow, ingest-validation, policy-gate |
| Gateway SSOT | `doge-complaints-gateway/schema-packs/tallinn_civic/v1/` |
| Manual | [`schema-packs-node-data-model-ru.md`](../../../../../doge-complaints-gateway/docs/runtime-docs/manuals/schema-packs-node-data-model-ru.md) |

## Dependencies

- Sibling gateway 51 Done (SSR wire + packs).
- Blocks GPT-SSR-02 / GPT-SSR-03 until T02–T03 paths/outlines accepted.
- Related: REQ-20 taxonomy pattern; REQ-18/22 envelope.

## Residual

- Physical migration of entire `story-label-taxonomy.md` into pack vs symlink/include — decide at P3 T03.
- Multi-node OpenAPI taxonomy soften — SSR-02 Residual / follow-up.
