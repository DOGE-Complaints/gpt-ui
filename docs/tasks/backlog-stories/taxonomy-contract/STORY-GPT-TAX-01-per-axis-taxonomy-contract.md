# STORY-GPT-TAX-01 — Per-axis taxonomy в контракте (OpenAPI + инструкции)

## Meta
- **Key:** `STORY-GPT-TAX-01-per-axis-taxonomy-contract`
- **Пакет:** [`taxonomy-contract/`](./INDEX.md)
- **Status:** 🟢 Done (Awaiting Commits) · P3 Execute 2026-07-23 · GAP-TAX-01-01 closed (GIM-215 P6)
- **Приоритет:** 🟠 MED-HIGH — предпосылка «точно замапленной» таксономии на бекенде
- **Тип:** contract (OpenAPI Action + GPT instruction modules)
- **Основание (решения):** [`interview-taxonomy-persistence-clustering-2026-07-13`](../../../../../doge-complaints-gateway/docs/analysis/interview-taxonomy-persistence-clustering-2026-07-13.md) — **D-TAX-1** (авторитет по оси = GPT, а не gateway-словари)
- **Разблокирует:** gateway [`GW-TAX-01`](../../../../../doge-complaints-gateway/docs/tasks/backlog-stories/taxonomy-fidelity/STORY-GW-TAX-01-taxonomy-persistence-fidelity.md) (consumer)
- **Pipeline:** [`STORY-GM8-19-gpt-tax-01-per-axis-taxonomy-contract`](../../../analysis/tasks/epics/EPIC-M1-08-label-extraction-axes-controlled-vocabulary/stories/STORY-GM8-19-gpt-tax-01-per-axis-taxonomy-contract/STORY-GM8-19-gpt-tax-01-per-axis-taxonomy-contract.md) (🟢 Done · pkg-000027 · GIM-209…214 · audit GIM-215)
- **Код-аудит:** [`audit-gpt-tax-01-per-axis-taxonomy-contract-2026-07-23.md`](../../../analysis/audit-gpt-tax-01-per-axis-taxonomy-contract-2026-07-23.md) — T01–T04 ✅

## Зачем простыми словами
GPT классифицирует историю по 13 осям (§12-24). До GPT-TAX-01 в HTTP-стеше метки уходили одной кучей — бекенд терял ось. Контракт должен отдавать таксономию **структурой с осью**.

## Что наблюдаю сейчас (контекст до реализации)

> Gap до GPT-TAX-01 / P3. Актуальное — **Текущее состояние (post-Done)**.

## Текущее состояние (post-Done, verified)

| Элемент | Реальность |
|---------|-----------|
| OpenAPI | `narrative.taxonomy` = `{ axis: [{ label, disposition }] }` (`NarrativeTaxonomy` + `TaxonomyAxisEntry`); `info.version` **0.7.0** ([`custom-gpt-story-intake-actions.openapi.yaml`](../../../custom-gpt-story-intake-actions.openapi.yaml)) |
| `canonical_labels` | retained, `deprecated: true` — derived flat of `disposition=canonical` only ([yaml:178](../../../custom-gpt-story-intake-actions.openapi.yaml)) |
| Осей в транспорте | 13 keys lockstep with gateway `TAXONOMY_AXIS_VALUES` |
| Инструкции GPT | normalizer **0.2.16** emits `canonical_payload.taxonomy`; orchestrator **0.5.0** maps + God Mode per-axis |

## Контекст до реализации (исторический)

| Элемент | Было (pre-P3) |
|---------|---------------|
| OpenAPI | `narrative.canonical_labels` — плоский массив строк (до `NarrativeTaxonomy`) |
| Осей в транспорте | нет — 13 осей §12 схлопнуты в один список |
| Инструкции GPT | таксономия v0.2.1 извлекает per-axis, нормализатор отдавал только flat `labels[]` |

## Требование / целевое состояние (D-TAX-1)
1. **OpenAPI:** `narrative.taxonomy` = `{ axis: [ { label, disposition } ] }` (ось из §12-24; disposition ∈ canonical/metadata_only/needs_clarification/rejected/internal §13).
2. **Обратная совместимость:** `canonical_labels` (flat) можно сохранить как deprecated/derived, но per-axis — источник истины.
3. **Инструкции/нормализатор GPT:** отдавать per-axis без потери оси; internal-метки помечать disposition (бекенд отфильтрует на публичном чтении).
4. **Lockstep:** согласовать версию контракта с gateway GW-TAX-01 (consumer) — общий SSOT осей/dispositions.

## Подзадачи (черновик)
| ID | Задача | Итог |
|----|--------|------|
| T01 | OpenAPI: schema `narrative.taxonomy` per-axis + disposition; версия контракта | ✅ GIM-209 |
| T02 | GPT-инструкции/нормализатор: эмитить per-axis (маппинг 13 осей §12-24 → transport) | ✅ GIM-211 |
| T03 | Lockstep-сверка осей/dispositions с gateway GW-TAX-01 | ✅ GIM-212 |
| T04 | Обновить примеры/тесты контракта; God-mode диагностика показывает per-axis | ✅ GIM-213 |

## Границы
- **In scope:** GPT UI контракт (OpenAPI) + инструкции/нормализатор.
- **Вне scope:** приём/хранение/кластеризация → gateway GW-TAX-01/02.
- **Не трогаем:** identity; submit-flow (GW-DRAFT).

## Швы
- [`custom-gpt-story-intake-actions.openapi.yaml`](../../../custom-gpt-story-intake-actions.openapi.yaml); GPT instruction modules (taxonomy/normalizer/orchestrator).
