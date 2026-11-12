# STORY-GPT-SSR-03 — Instructions: emit schema_binding + geo per pack geo_intake

## Meta
- **Key:** `STORY-GPT-SSR-03-instructions-emit-binding-and-geo`
- **Пакет:** [`semantic-schema-runtime/`](./INDEX.md)
- **Status:** Done
- **Приоритет:** P1 — behavioral emit + geo hardening
- **Тип:** instruction update (orchestrator / normalizer / pack overlays)
- **Parent REQ:** [`REQ-45-semantic-schema-intake-envelope-compatibility.md`](../../../requirements/REQ-45-semantic-schema-intake-envelope-compatibility.md)
- **Siblings:** [GPT-SSR-01](./STORY-GPT-SSR-01-core-vs-swappable-packs.md), [GPT-SSR-02](./STORY-GPT-SSR-02-openapi-schema-binding-geo-detail.md)
- **Skill declared:** openai-custom-gpt-builder
- **Depends:** GPT-SSR-01, GPT-SSR-02
- **Active pair (civic instance):** `tallinn_civic` / `v1` from `schema-packs/README.md` (SSR-01)

## Зачем простыми словами

Даже с OpenAPI gateway отвергнет стеш без `schema_binding`. Сейчас geo = только размазанный `location_query` + Estonia-канон в normalizer; **`geo_detail` не эмитится**. Инструкции должны MUST-emit binding, собирать structured_payload из pack, и уважать `geo_intake.mode` — narrative-first, structured/geo дополнительно.

## Архитектура решения (целевой geo + binding flow)

```text
Interview CORE (phases / §7.2 affirm)
        │
        ▼ read pack data-model
Pack geo_intake.mode ──► collect freeform and/or address/coords parts
        │
        ▼
Normalizer CORE
  • apply pack geo formation (§4.6 becomes «delegate to pack»)
  • emit location_query when mode/rules say so
  • build structured_payload signals.* from pack field_policy
        │
        ▼
Orchestrator CORE
  • MUST schema_binding { schema_id, schema_version, structured_payload }
  • MAY geo_detail per mode
  • envelope schema_version = m2.story_intake_envelope.v2
  • pre-flight: missing binding → STOP; mode-aware geo gate
        │
        ▼
Gateway (authoritative validate / merge / mirror_to_payload)
```

### Geo mode matrix (from gateway `intake_merge` / pack)

| `geo_intake.mode` | GPT SHALL | Civic `tallinn_civic/v1` |
|------------------|-----------|---------------------------|
| `optional` | MAY omit `geo_detail` and `location_query` | **Current pack value** |
| `require_location_or_detail` | non-empty `location_query` **or** `geo_detail` | — |
| `require_detail` | non-empty `geo_detail` | — |

Pack also has `merge` / `mirror_to_payload` (gateway-side); GPT documents them in data-model pack but does **not** invent wire keys for them.

## Sprawl / gap matrix (emit + geo)

| Concern | Today | Target |
|---------|-------|--------|
| Emit `schema_binding` | **0 hits** in instructions | MUST from active pair |
| Emit `structured_payload` | Absent | From pack / normalizer handoff |
| Emit `geo_detail` | Absent | MAY/SHALL per mode |
| `location_query` §4.6 Estonia | [`story-normalizer.md`](../../../../instructions/story-normalizer.md) L414–433 | Rules live in **data-model pack**; core applies pack |
| Pre-flight confirmed ∧ missing `location_query` | [`api-orchestrator.md`](../../../../instructions/api-orchestrator.md) L633 | Mode-aware gate (pack `geo_intake`) |
| GEO_SCOPE_MISMATCH Tallinn copy | orchestrator L659 | Scope copy → pack; HTTP handling → core |
| Civic taxonomy examples in stash JSON | orchestrator ~L310–314 | Move examples to pack; core generic stub |
| Narrative-first PR-004 | Partial | Explicit: structured + geo additional |

## Scope

- Orchestrator MUST emit matching `schema_binding`; fill `structured_payload` per pack (no invent keys outside gateway `payload.schema.json`).
- Geo: mode matrix; migrate §4.6 canon into pack; mode-aware pre-flight.
- Strip civic-specific examples from core orchestrator into pack.
- Evidence + no regression PII/GUARD.
- Narrative-first (AC-GPT-REQ3-04).

## Out of scope

- OpenAPI authoring → SSR-02.
- Creating pack layout files → SSR-01 P3 (this story **consumes** them).
- Gateway validate / merge implementation.
- Soften OpenAPI taxonomy axes.
- Marketplace / §30; invent field names.

## Verified current state

| Fact | Evidence |
|------|----------|
| Orchestrator example has `schema_version`, not binding | api-orchestrator.md ~L294 |
| Field table hard-codes envelope | L350 |
| `narrative.location_query` mapped | L360; normalizer §4.6 |
| Optional omit list — no binding | L344 |
| Location coverage STOP | L633 |
| OpenAPI lacks binding/geo_detail | openapi.yaml L249–274 |
| Gateway missing binding → error | contracts.py L393–396 |
| Civic `geo_intake.mode` | pack.json L207–211 `optional` |

## Подзадачи

| ID | Задача | DoD |
|----|--------|-----|
| **T01** | Orchestrator MUST `schema_binding` | Pair from `schema-packs/README.md` active; pre-flight STOP if missing/empty; God Mode shows transport binding |
| **T02** | Map `structured_payload` | From pack data-model / normalizer; keys ⊆ civic `payload.schema.json` (`signals.*`, optional `geo.*`); no invent |
| **T03** | Geo mode matrix in instructions | Document three modes; civic instance = `optional`; collect `geo_detail` and/or `location_query` per mode |
| **T04** | Migrate §4.6 Estonia rules → pack | Pack data-model owns canon; normalizer §4.6 = «apply pack geo formation» + keep omit/blank core rules |
| **T05** | Mode-aware pre-flight | Replace hardcoded L633 gate with pack `geo_intake.mode` logic; `optional` does not force location_query |
| **T06** | Strip civic examples from orchestrator | Examples/labels → pack; core keeps generic placeholders |
| **T07** | Evidence matrix | Binding present on stash body; geo matrix rows; narrative-first; PII V1–V8 + GUARD H5–H7 not regressed |
| **T08** | Story acceptance gate | Checklist AC below; acceptance-verification artifact at P3 |

### T03/T05 decision table (implement in instructions)

| Mode | Confirmed place, no geo_detail, no location_query | Action |
|------|---------------------------------------------------|--------|
| `optional` | OK | Proceed (MAY still emit if resident gave place) |
| `require_location_or_detail` | FAIL | STOP — collect one of the two |
| `require_detail` | FAIL if no geo_detail | STOP — need structured geo_detail |

## Target / Acceptance Criteria

- [x] **AC-GPT-REQ3-02** (instructions half): emit only gateway-named tokens.
- [x] **AC-GPT-REQ3-04:** narrative-first; structured + geo additional.
- [x] **AC-GPT-BIND-01:** MUST emit matching `schema_binding`; pre-flight enforces.
- [x] **AC-GPT-GEO-01:** pack documents `geo_intake`; emit follows mode.
- [x] **AC-GPT-REQ3-16** spot-check: no new civic field names welded into core (use pack).
- [x] Evidence (T07) + acceptance (T08).
- [x] No regression: GPT-PII-01 stop-lines; GPT-GUARD-01 H5–H7.

## Точки в коде

| Зона | Path |
|------|------|
| Orchestrator | `GPT UI/instructions/api-orchestrator.md` §5.2.x |
| Normalizer | `GPT UI/instructions/story-normalizer.md` §4.6 → pack delegate |
| Pack data-model | `GPT UI/instructions/schema-packs/tallinn_civic/v1/data-model.md` (SSR-01) |
| Pack inbound-validation | `…/inbound-validation.md` |
| Methods ref | `GPT UI/instructions/story-api-methods-reference.md` |
| Gateway modes | `doge-complaints-gateway/src/core/schema/intake_merge.py` (enforce modes) |

## Dependencies

- GPT-SSR-01, GPT-SSR-02.
- REQ-18/22/26; sibling 51.
- Do not regress GPT-PII-01 / GPT-GUARD-01.

## Residual

- Live Custom GPT session evidence — optional advisory (info), not blocking if static path evidence exists.
- When pack mode upgrades from `optional` to `require_*`, operator updates pack file only (AC-016) — core gate already mode-aware after T05.
