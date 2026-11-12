# STORY-GPT-TAX-02 — Taxonomy axes lockstep + English label values (embeddings)

## Meta
- **Key:** `STORY-GPT-TAX-02-taxonomy-label-key-sync` _(key сохранён; смысл уточнён 2026-08-10)_
- **Пакет:** [`intake-hardening/`](./INDEX.md)
- **Status:** Done
- **Приоритет:** High — GAP-IH-02 (operator refine: **не** dictionary reject)
- **Тип:** product requirement (contract / embedding-friendly labels)
- **Gap:** [`gap-analysis-intake-mission-taxonomy-pii-2026-08-10.md`](../../../analysis/gap-analysis-intake-mission-taxonomy-pii-2026-08-10.md) §2
- **Основание:** [GPT-TAX-01](../taxonomy-contract/STORY-GPT-TAX-01-per-axis-taxonomy-contract.md) Done (per-axis + axes lockstep); [REQ-20](../../../requirements/REQ-20-label-taxonomy-and-extraction-axes.md); SSOT [`story-label-taxonomy.md`](../../../../instructions/story-label-taxonomy.md)
- **Pipeline / GIM:** GIM-226…230 (pkg-000033) 🟢

## Зачем простыми словами

Оси уже совпадают (TAX-01). Полный server-словарь canonical keys для MVP **не нужен**: кластеризация опирается на **векторы**, а не на exact tag equality. Важно другое: **значения labels на wire всегда на английском** (одна языковая норма), чтобы эмбеддинги и downstream были стабильнее.

## Текущее состояние (verified)

| Элемент | Реальность |
|---------|------------|
| Оси (13) | Lockstep GPT ↔ gateway `TAXONOMY_AXIS_VALUES` (GPT-TAX-01 Done) |
| Wire | `narrative.taxonomy` per-axis (OpenAPI 0.7.0) |
| Gateway parse | Axis + disposition + non-empty label string — **нет** vocabulary reject |
| Label language | SSOT keys в taxonomy — English-style; **нет** жёсткого product AC «wire labels always English / no et\|ru prose» как отдельного hardening outcome |
| GW-L10N-04 | Deferred public registry — **не** блокер этой стори |

## Требование / целевое состояние

1. **Оси** остаются lockstep с GPT-TAX-01 / gateway `TAXONOMY_AXIS_VALUES` (13 keys) — без регрессии.
2. Значения `label` на wire (canonical и иные disposition, которые уходят в stash) — **English** tokens (snake_case / SSOT style), **не** произвольная проза на et/ru как label value.
3. **Не** требовать server dictionary reject unknown keys и **не** активировать GW-L10N-04 для MVP.
4. Расхождение конкретных label strings между сессиями допустимо с точки зрения exact-match; компенсация — embeddings / clustering, не «обязательный общий enum на HTTP 422».

## Acceptance Criteria (requirements)

- [x] GPT normalizer / orchestrator / pre-flight: canonical (и wire-bound) labels нормализуются к **English** token form перед `postStoryDraftStash`.
- [x] Evidence: попытка отправить label-value на ru/et prose → исправление или STOP до HTTP; English SSOT-style labels → OK.
- [x] Regression: 13 axes lockstep с gateway не ломается (GPT-TAX-01).
- [x] Явно **нет** AC «unknown key → HTTP 422 / server vocabulary reject».
- [x] Документировано: exact key sync вторичен относительно vector similarity.

## Границы

- **In scope:** GPT instruction + contract guidance для English label values; axes lockstep hygiene.
- **Вне scope:** server full vocabulary registry; GW-L10N-04; redesign количества осей; gateway AI.
- **Не трогаем:** Done Status GPT-TAX-01 / REQ-20.

## Швы

- GPT: `story-label-taxonomy.md`, normalizer, orchestrator pre-flight.
- Gateway: axes already Done; **no** new vocabulary reject seam for this story.
- Related: GPT-GUARD-01 (не смешивать с junk), embeddings/clustering на gateway (consumer of English labels).
