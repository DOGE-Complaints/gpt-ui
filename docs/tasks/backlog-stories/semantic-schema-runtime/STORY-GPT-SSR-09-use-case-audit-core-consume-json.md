# STORY-GPT-SSR-09 — Use-case coverage audit + core consume JSON (capstone)

## Meta
- **Key:** `STORY-GPT-SSR-09-use-case-audit-core-consume-json`
- **Пакет:** [`semantic-schema-runtime/`](./INDEX.md)
- **Status:** Done
- **Приоритет:** P1 — wave 2 capstone (P3 execution)
- **Тип:** product / implementation spec (audit + instruction updates + file materialization)
- **Parent REQ:** [`REQ-45`](../../../requirements/REQ-45-semantic-schema-intake-envelope-compatibility.md)
- **Siblings:** [GPT-SSR-04…08](./INDEX.md) — all upstream analysis stories
- **Skill declared:** openai-custom-gpt-builder
- **Depends:** GPT-SSR-04, GPT-SSR-05, GPT-SSR-06, GPT-SSR-07, GPT-SSR-08
- **Analysis input:** [`cross-project-node-model-portability-gap-2026-09-01.md`](../../../../../docs/analysis/cross-project-node-model-portability-gap-2026-09-01.md)
- **Coherence analysis:** [`gpt-ssr-wave2-coherence-analysis-2026-09-01.md`](../../../analysis/gpt-ssr-wave2-coherence-analysis-2026-09-01.md) §11

## Coherence analysis (2026-09-01)

Verified P3 pkg-000045: JSON cmp ok; interview-overlay v1.0; data-model archived; core reads pack JSON. Evidence [`gpt-ssr-09-capstone-evidence-2026-09-01.md`](../../../analysis/gpt-ssr-09-capstone-evidence-2026-09-01.md). Residual: mobility/legal overlays, G-CARD-01, SSR-08 territory body.

## Зачем простыми словами

Wave 2 начинается с анализа (SSR-04…08). Capstone story **собирает** результаты в одну матрицу «юзкейс → поле/правило → as-built → gap» и **закрывает** переход на целевое состояние:

- GPT pack dir содержит **literal copy** gateway JSON (+ taxonomy JSON когда готов gateway sibling).
- Core instructions (**orchestrator, normalizer, README, operator manual**) читают JSON, а не Markdown-проекцию формы полей.
- `data-model.md` **deprecate/archive**; prose-only контент в `interview-overlay.md`.
- `inbound-validation.md` — точечные pointer updates (SSR-06 delta).

Подозрение оператора: gateway pack **ещё имеет дыры** — не все продуктовые юзкейсы выразимы в текущем JSON. Эта story **не закрывает** gateway gaps кодом, но **фиксирует** их в матрице и решает, что блокирует GPT-side P3 vs что требует GW-SSR-26/27.

## Целевое состояние wave 2 (capstone)

```text
GPT UI/instructions/schema-packs/<id>/<ver>/
  pack.json                 ← COPY gateway
  payload.schema.json       ← COPY gateway
  taxonomy.json             ← COPY gateway (when GW-SSR-26 Done)
  inbound-validation.md     ← MD (updated pointers)
  interview-overlay.md      ← MD (geo canon, phase→axes, ecosystem)
  [examples.md]             ← optional demo JSON

REMOVED or archived:
  data-model.md             ← deprecated — no longer SSOT for field shape

Core hooks:
  orchestrator → read pack.json (geo_intake, binding pair, instance_territory if present)
  normalizer   → read payload.schema.json + taxonomy.json (axis_to_signal_map)
```

## Use-case catalog (T01 — indicators)

| UC-ID | Use case | Packs involved |
|-------|----------|----------------|
| UC-01 | Civic interview tallinn — full stash | tallinn_civic/v1 |
| UC-02 | Operator swap node — copy-paste pack | any pack |
| UC-03 | Mobility observation pack — no GPT overlay yet | mobility_observation/v1 |
| UC-04 | Legal process pack — no GPT overlay yet | legal_process/v1 |
| UC-05 | Geo city-only story | tallinn_civic |
| UC-06 | Geo street + house story | tallinn_civic |
| UC-07 | Multi-district operator — instance territory | tallinn_civic + SSR-08 |
| UC-08 | Region-level story (not point) | tallinn_civic + SSR-08 |
| UC-09 | Required signals from taxonomy map | tallinn_civic + SSR-05 |
| UC-10 | schema_card on public board | tallinn — **no card_fields in pack** |
| UC-11 | Out-of-scope geo — 422 + resident copy | tallinn_civic |
| UC-12 | Direct API client bypassing GPT content rules | gateway only — document dual contour |

## Coverage matrix template (T02 — fill at P3 audit)

| UC-ID | Required field/rule | As-built (path) | Gap | Owner |
|-------|---------------------|-----------------|-----|-------|
| UC-01 | schema_binding required | contracts.py:393+ | — | gateway OK |
| UC-01 | signals.civic_domain required | pack field_policy | — | OK |
| UC-09 | axis→signal map | data-model §2.3 only | G-JSON-02 | SSR-05 / GW-SSR-26 |
| UC-10 | card_fields → schema_card | tallinn pack: absent | G-CARD-01 | gateway optional |
| UC-07 | instance territory | not in pack | G-GEO-INST-01 | SSR-08 / GW-SSR-27 |
| UC-08 | detail_level | not in schema | G-GEO-PREC-01 | SSR-08 / GW-SSR-27 |
| UC-03 | GPT pack overlay | GPT: absent | G-PAYLOAD-02 | operator / future pack |

## Core instruction edits spec (T03)

| Module | Change | Replace |
|--------|--------|---------|
| [`api-orchestrator.md`](../../../../instructions/api-orchestrator.md) | Read `pack.json` for geo_intake, binding pair, gpt_instance_territory | «read data-model.md» |
| [`story-normalizer.md`](../../../../instructions/story-normalizer.md) | Read `payload.schema.json` + `taxonomy.json` for signals handoff | data-model §2.3 prose |
| [`schema-packs/README.md`](../../../../instructions/schema-packs/README.md) | Locked paths: JSON files + inbound + interview-overlay | data-model.md path |
| [`story-api-methods-reference.md`](../../../../instructions/story-api-methods-reference.md) | Note: pack JSON read-only SEC-001 | if references data-model |
| Core demo examples | Generic placeholders only | civic examples → examples.md |

**Pre-flight additions (indicators):**

- Instance territory check before stash (SSR-08 rules).
- Pack-required keys from `field_policy` + taxonomy axis_to_signal_map.

## File materialization (T04 — P3 DoD)

| Action | File |
|--------|------|
| COPY | `pack.json`, `payload.schema.json` from gateway tallinn_civic/v1 |
| CREATE | `interview-overlay.md` — content from data-model §4–§5 |
| UPDATE | `inbound-validation.md` — SSR-06 delta pointers |
| ARCHIVE | `data-model.md` → `archive/data-model.v1.3.md` or delete with git history |
| UPDATE | operator manual §3–§5 — copy-paste not project |
| WAIT | `taxonomy.json` until GW-SSR-26 |

## Operator manual updates (T05)

| Manual § | Change |
|----------|--------|
| §3 layout | Two JSON + taxonomy JSON + inbound + interview-overlay |
| §5 swap step 1 | **Copy** files from gateway dir — remove «спроецировать» |
| §5 diff-check | Optional: `diff gateway/.../pack.json GPT/.../pack.json` |
| §6 dual contour | Unchanged philosophy |

## Scope

- T01: use-case catalog (UC-01…12)
- T02: per use-case coverage matrix against tallinn + mobility + legal packs
- T03: core instruction edits spec (table above)
- T04: file materialization checklist + archive data-model.md
- T05: operator manual §3–§5 rewrite spec

## Out of scope

- Gateway GW-SSR-26/27 implementation
- spa-16 FE overlay
- mobility/legal GPT pack creation (document as gap only)
- Marketplace / §30
- OpenAPI version bump unless wire changes

## Verified current state (As-of-Done P3)

| Fact | Evidence |
|------|----------|
| GPT JSON pack files | **Present** — cmp ok vs gateway |
| taxonomy.json | **Present** tallinn_civic/v1 |
| data-model.md | **Archived** → `archive/data-model.v1.3.md` |
| interview-overlay.md | **v1.0** on disk |
| Core references | orchestrator/normalizer → pack JSON + overlay |
| Gateway gaps documented | evidence UC matrix G-PAYLOAD-02, G-CARD-01, G-GEO-* partial |

## Подзадачи

| ID | Задача | DoD |
|----|--------|-----|
| **T01** | Use-case catalog | UC table ≥10 rows |
| **T02** | Coverage matrix | Each UC mapped to as-built + gap + owner |
| **T03** | Core edits spec | Module-by-module replace list |
| **T04** | File materialization | Checklist; data-model fate = archive |
| **T05** | Operator manual spec | §3–§5 copy-paste workflow |
| **T06** | P3 execution gate | All SSR-04…08 AC satisfied; partial P3 if GW-SSR-26 pending |

## Target / Acceptance Criteria

- [x] **AC-GPT-SSR09-01:** use-case coverage matrix complete for tallinn + mobility + legal gaps documented.
- [x] **AC-GPT-SSR09-02:** core instructions spec replaces data-model with JSON read paths.
- [x] **AC-GPT-SSR09-03:** pack JSON files on disk for tallinn_civic/v1 (literal copy, cmp ok).
- [x] **AC-GPT-SSR09-04:** data-model.md archived; interview-overlay.md created.
- [x] **AC-GPT-SSR09-05:** operator manual §3–§5 updated — copy-paste workflow v3.
- [x] **AC-GPT-SSR09-06:** README locked paths updated v1.3.
- [x] **AC-GPT-SSR09-07:** remaining gateway gaps documented — not silently dropped.

## Partial P3 policy (when GW-SSR-26 pending)

| Can ship without GW-SSR-26 | Blocked until GW-SSR-26 |
|----------------------------|-------------------------|
| pack.json + payload.schema.json copy | taxonomy.json copy |
| interview-overlay.md | axis_to_signal_map from JSON |
| inbound pointer to pack.json field_policy | inbound §2 taxonomy JSON pointer |
| Core read pack.json geo_intake | Core read taxonomy.json for signals |

## Dependencies

- All GPT-SSR-04…08 analysis AC met.
- Soft dependency GW-SSR-26 for full taxonomy path.
- Soft dependency GW-SSR-27 for full geo precision/territory path.

## Residual

- mobility_observation / legal_process GPT overlays — separate operator stories after tallinn capstone proven.
- Automated diff-check script — optional tooling story; not in wave 2 scope.
- **REQ-45 wave-2 addendum (doc-only, post-P3):** update REQ-45 Verified paths from Wave 1 `{data-model,inbound-validation}.md` to Wave 2 `{pack.json,payload.schema.json}` (+ `taxonomy.json` when GW-SSR-26 Done) + `{inbound-validation,interview-overlay}.md`. See [`gpt-ssr-wave2-coherence-analysis-2026-09-01.md`](../../../analysis/gpt-ssr-wave2-coherence-analysis-2026-09-01.md) §14. No REQ Status regression until P3 materialization lands.
