# REQ-39: Geographic intelligence — location confidence model + city canonicalization

> **Назначение:** дополнить уже реализованный location_query-слой (REQ-26 wire, REQ-35 MUST+Latin) двумя элементами geo-intelligence: (1) **confidence-метаданными** извлечения локации (detected / source / confidence), (2) **канонизацией city-уровня** к форме `<City>, Estonia` для надёжного geocoding-handoff. Независимая geo-подсистема, не связана с таксономией (REQ-36/37).
> **Источник:** Gap Analysis 2026-06-04 (черновик REQ-36 §4 «Geographic Intelligence»).
> **Технический контекст (verified по коду):** backend `geo_service.resolve_for_story()` → `normalize_location_query()` ([`src/core/geo/normalize.py`](../../../doge-complaints-gateway/src/core/geo/normalize.py)) приводит к lowercase + collapse whitespace; stub-резолвер ([`src/core/geo/providers.py`](../../../doge-complaints-gateway/src/core/geo/providers.py) L316–323) матчит **подстрокой** (`if keyword in canonical_key`) по алиасам, включающим латиницу **и кириллицу** (`"tallinn"`, `"таллин"`, `"таллинн"`); city-labels в форме `"Tallinn, EE"`. Значит форма `Tallinn, Estonia` backend-совместима (подстрока `tallinn` находится), но для матчинга избыточна — её ценность в disambiguation для будущего реального geocoder.

**Версия:** 1.0 · 2026-06-04
**Статус:** requirements — ready for tasking
**Приоритет:** P3 (geo-полнота demo; частично перекрыто REQ-35 — остаётся узкая дельта)
**Тип:** GPT instruction update — `story-normalizer.md` §4.6 (canonicalization) + §4.2.1/новый sidecar (confidence metadata)
**Серверная сторона:** не требует изменений (substring-match уже принимает `<City>, Estonia`; реальный geocoder — product backlog)
**Парные REQ:** [REQ-26](./REQ-26-location-query-normalizer-to-wire.md) (location_query wire), [REQ-35](./REQ-35-demo-field-population-gpt-fixes.md) (location_query MUST + Latin) · backend [REQ-46](../../../doge-complaints-gateway/docs/requirements/46-demo-services-enablement-and-response-transparency.md) (geo expansion)

---

## 1. Текущее состояние (verified по коду 2026-06-04)

### 1.1 Уже сделано (вне scope — REQ-26 / REQ-35)

| Черновик §4 пункт | Закрыто | Доказательство |
|---|---|---|
| §4.1 city named → location candidate + normalizer emit | REQ-35 | [`story-normalizer.md:317`](../../instructions/story-normalizer.md#L317) «MUST form `location_query`» когда место подтверждено |
| формат latin для надёжности | REQ-35 | [`story-normalizer.md:318`](../../instructions/story-normalizer.md#L318) «prefer Latin script» + `normalize_location_query()` ref |
| wire-маппинг location_query | REQ-26 | [`api-orchestrator.md:325`](../../instructions/api-orchestrator.md#L325) |

### 1.2 Остаточная дельта (scope этого REQ)

| # | Дельта | Факт в коде | Severity |
|---|---|---|---|
| G1 | Нет **confidence-метаданных** локации | `story-normalizer.md §4.6` (L299–323) формирует только строку `location_query`; нет полей `location_detected` / `location_source` (explicit/inferred) / `confidence` (LOW/MEDIUM/HIGH) | MEDIUM |
| G2 | Нет **city-канонизации** к `<City>, Estonia` | §4.6 правило 2 (L318) задаёт «prefer Latin» и формат `<street/place>, <city>`, но не канонизирует варианты (`Tallinn` / `Tallinn, Estonia` / `Tallinna linn`) к единой форme | LOW–MEDIUM |

---

## 2. Целевое состояние

### 2.1 G1 — Geographic confidence model (`story-normalizer.md`)

Добавить внутренний (non-wire) sidecar метаданных извлечения локации:

| Поле | Значение |
|---|---|
| `location_detected` | bool |
| `location_source` | `explicit` (резидент назвал) / `inferred` (выведено из контекста) |
| `confidence` | `LOW` / `MEDIUM` / `HIGH` |

Размещение: `normalization_metadata` (как label_extraction_metadata) — **не wire** (server geo-резолв не потребляет; реальный потребитель — операторская диагностика / будущий geocoder). Согласовать с REQ-35 privacy-baseline (только из сказанного резидентом для `explicit`).

### 2.2 G2 — City canonicalization (`story-normalizer.md` §4.6)

Правило: city-уровневые упоминания канонизировать к `<City>, Estonia` (латиница) для geocoding-handoff:

```
Tallinn           → Tallinn, Estonia
Tallinn, Estonia  → Tallinn, Estonia
Tallinna linn     → Tallinn, Estonia
```

Verified: форма backend-совместима (substring-match найдёт `tallinn`); district/street-детализация (`Kalamaja, Tallinn`) сохраняется как есть (REQ-35) — канонизация применяется к **city-уровню**, не затирает более точные адреса.

### 2.3 Согласование с REQ-35

Не ослаблять REQ-35: MUST-when-confirmed (L317), omit-rules 3–5 (L319–321), prefer-Latin (L318). Канонизация G2 — уточнение формата city-уровня поверх существующего правила 2, не замена.

---

## 3. Файлы для изменения

| Файл | Секция | Что изменяется |
|------|--------|----------------|
| `GPT UI/instructions/story-normalizer.md` | §4.6 правило 2 | G2: city-канонизация к `<City>, Estonia` (city-уровень), сохранить district/street детализацию |
| `GPT UI/instructions/story-normalizer.md` | §4.2.1 / новый metadata-блок | G1: `location_detected` / `location_source` / `confidence` в `normalization_metadata` (non-wire) |
| `GPT UI/instructions/story-normalizer.md` | Version + history | Bump + changelog REQ-39 |

---

## 4. Acceptance Criteria

**Static (verifiable по `story-normalizer.md`):**
- [ ] §4.6 содержит правило city-канонизации `<City>, Estonia` с примерами (`Tallinn` / `Tallinna linn` → `Tallinn, Estonia`)
- [ ] Канонизация применяется к city-уровню и НЕ затирает district/street (`Kalamaja, Tallinn` сохраняется)
- [ ] `normalization_metadata` содержит confidence-поля (`location_detected`, `location_source`, `confidence`) как non-wire
- [ ] Регрессия: REQ-35 MUST-when-confirmed (L317), Latin (L318), omit-rules 3–5 (L319–321) не ослаблены; confidence-sidecar не попадает в `StoryIntakeRequest`

**Advisory (live replay):**
- [ ] История с «Tallinn» → `location_query = "Tallinn, Estonia"` без доп. вопросов (AC-36.3 источника)
- [ ] История с районом → city-канонизация не теряет район

---

## 5. Не в scope

- **location_query MUST + Latin + wire** — уже REQ-35 / REQ-26 (§1.1).
- **Taxonomy / ecosystem** → REQ-36 / REQ-38.
- Реальный geocoding API (вместо stub) — product backlog.
- Backend geo expansion (кириллица street-уровня, intake_notes) → backend [REQ-46](../../../doge-complaints-gateway/docs/requirements/46-demo-services-enablement-and-response-transparency.md).
- Wire-передача geo-confidence — не входит (sidecar internal-only; активировать только при lockstep-изменении схемы).
