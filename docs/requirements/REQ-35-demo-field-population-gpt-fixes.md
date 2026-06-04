# REQ-35: Demo field-population — location_query obligation + latin, origin.conversation_id, severity

> **Назначение:** закрыть остаточную дельту GPT-инструктажного слоя по наполнению полей при подаче истории, выявленную тест-прогоном 2026-06-01. Бóльшая часть исходного списка уже закрыта парными REQ (см. §1.1 «Уже сделано»); этот REQ фиксирует **только проверенную по коду дельту**: (1) `location_query` сформулирован как разрешение, а не обязанность, и без правила латиницы; (2) `origin.conversation_id` без guidance на заполнение из контекста; (3) `gpt_signals.severity` без активного правила определения.
> **Источник:** перенос GPT-части из `doge-complaints-gateway/docs/requirements/47-demo-operational-railway-gpt-fixes.md` §2 (Railway-часть §3 — не-GPT ops, остаётся в REQ-47); первичный источник — [`1.06.26-testing-gap-report.md`](../analysis/1.06.26-testing-gap-report.md).
> **Технический контекст:** backend `normalize_location_query()` ([`src/core/geo/normalize.py`](../../doge-complaints-gateway/src/core/geo/normalize.py)) приводит запрос к lowercase; кириллица расширяется парным REQ-46 (backend, ready-for-tasking), латиница предпочтительна для надёжности geo-резолва.

**Версия:** 1.0 · 2026-06-03
**Статус:** Done — instruction P3 (pkg-000016 · GIM-152…155 🟢 2026-06-03); manual replay Advisory pending
**Приоритет:** P2 (качество данных demo: geo-поля, атрибуция канала, severity-сигнал)
**Тип:** GPT instruction update — `story-normalizer.md` §4.6 + §4.3; `api-orchestrator.md` §5.2.1
**Серверная сторона:** не требует изменений (backend принимает все поля; geo-расширение — отдельный REQ-46)
**Парные REQ:** [REQ-26](./REQ-26-location-query-normalizer-to-wire.md) (location_query wire), [REQ-32](./REQ-32-origin-traceability-sending-enforcement.md) (origin.source enforcement), [REQ-34](./REQ-34-summary-generation-and-canonical-type-clarity.md) (summary generation), [REQ-27](./REQ-27-gpt-signals-enum-sync-and-data-model-update.md) (gpt_signals enums) · backend [REQ-46](../../doge-complaints-gateway/docs/requirements/46-demo-backend-geo-intake-notes-logging.md), [REQ-47](../../doge-complaints-gateway/docs/requirements/47-demo-operational-railway-gpt-fixes.md)

---

## 1. Текущее состояние

### 1.1 Уже сделано (вне scope — verified по коду 2026-06-03)

| REQ-47 §2 пункт | Закрыто чем | Доказательство в коде | Статус |
|---|---|---|---|
| §2.2 `origin.source` всегда отправлять | REQ-32 | [`api-orchestrator.md:326`](../../instructions/api-orchestrator.md#L326) «de-facto required… do not omit»; [yaml L171–176](./../custom-gpt-story-intake-actions.openapi.yaml) «Always include origin.source='openai_gpt_action'» | ✅ DONE |
| §2.2 блок `origin` присутствует | REQ-32 | [`api-orchestrator.md:288-291`](../../instructions/api-orchestrator.md#L288-L291) JSON-шаблон | ✅ DONE |
| §2.3 summary формируется и отправляется | REQ-34 | `story-normalizer.md §4.1` L140–147 (MUST generate); `story-interview-flow.md` Step 5b L153; wire `api-orchestrator.md:324` | ✅ DONE |
| §2.1 `location_query` wire-маппинг | REQ-26 | [`api-orchestrator.md:325`](../../instructions/api-orchestrator.md#L325); `story-normalizer.md §4.6` L299–313 | ✅ DONE (маппинг) |
| §2.4 `gpt_signals` enum sync + wire | REQ-27 / REQ-42 / REQ-23 | `story-normalizer.md §4.3` L245–251; `api-orchestrator.md:332-334` | ✅ DONE (enums/wire) |

### 1.2 Остаточная дельта (scope этого REQ — verified по коду)

| # | Дельта | Факт в коде | Severity |
|---|---|---|---|
| D1 | `location_query` сформулирован как **разрешение**, не обязанность | `story-normalizer.md:307`: «Include **only** when the user explicitly named or confirmed a location» — задаёт необходимое условие, но не императив «MUST когда место упомянуто» | LOW |
| D2 | Нет правила **латиницы** + нет упоминания backend lowercase-нормализации | `story-normalizer.md:308`: «Format: freeform string, preferably `<street/place>, <city>`» — примеры латиницей, но **нет** явного правила script/транслитерации и ссылки на `normalize_location_query()` | MEDIUM |
| D3 | `origin.conversation_id` без guidance на заполнение | `api-orchestrator.md:327`: строка маппинга `origin.conversation_id` с **пустым** полем Notes; нет указания брать из GPT Actions контекста и не оставлять null | LOW–MEDIUM |
| D4 | `gpt_signals.severity` без активного правила определения | `story-normalizer.md:241` (§4.3): «If collected and confirmed upstream, place them… otherwise omit» — пассивно; нет правила «определи severity когда контекст достаточен» | LOW |

---

## 2. Целевое состояние

### 2.1 D1 — `location_query` как обязанность (`story-normalizer.md §4.6`)

Усилить формулировку правила 1: если резидент в любой фазе интервью назвал/подтвердил город, район, улицу или ориентир — `location_query` **обязателен** к формированию (а не «может быть включён»). Omit-условия (не подтверждено / неоднозначно / пусто) из правил 3–5 §4.6 сохраняются как единственные основания опустить поле.

### 2.2 D2 — латиница + backend-нормализация (`story-normalizer.md §4.6`)

Добавить в §4.6 правило формата: предпочтительна **латиница** (`Tallinn`, `Kalamaja, Tallinn`, `Tartu mnt 80, Tallinn`), т.к. backend нормализует запрос через `normalize_location_query()` (lowercase) перед geo-резолвом; кириллица допустима, но латиница надёжнее (кириллическое расширение — парный backend REQ-46). Не выдумывать адрес сверх сказанного резидентом.

### 2.3 D3 — `origin.conversation_id` guidance (`api-orchestrator.md §5.2.1`)

Заполнить пустое Notes строки маппинга `origin.conversation_id`: брать активный `conversation_id` из доступного GPT Actions контекста сессии; включать когда доступен, не отправлять null-строкой. (Доступность — runtime-свойство GPT Actions; инструкция не может гарантировать наличие, только предписать заполнение при наличии.)

### 2.4 D4 — активное определение `severity` (`story-normalizer.md §4.3`)

Уточнить §4.3: когда интервью содержит достаточный контекст серьёзности, заполнять все три субъективных поля (`severity`, `impact_estimation`, `problem_status`), а не только два. Сохранить ограничение privacy-слоя: собирать **только** из уже сказанного резидентом, без наводящих вопросов (`story-data-model.md §4.3`); при отсутствии сигнала — омит поля (для `impact_estimation` нет `UNKNOWN`-фоллбэка).

---

## 3. Файлы для изменения

| Файл | Секция | Что изменяется |
|------|--------|----------------|
| `GPT UI/instructions/story-normalizer.md` | §4.6 правило 1 | D1: разрешение → обязанность при упомянутом месте |
| `GPT UI/instructions/story-normalizer.md` | §4.6 правило 2 (format) | D2: латиница + ссылка на backend lowercase-нормализацию |
| `GPT UI/instructions/story-normalizer.md` | §4.3 | D4: активно заполнять severity при достаточном контексте (без наводящих вопросов) |
| `GPT UI/instructions/api-orchestrator.md` | §5.2.1 маппинг `origin.conversation_id` | D3: заполнить Notes — брать из GPT Actions контекста, не null |
| `GPT UI/instructions/story-normalizer.md` | Version header + history | Bump + changelog REQ-35 |
| `GPT UI/instructions/api-orchestrator.md` | Version header + history | Bump + changelog REQ-35 |

---

## 4. Acceptance Criteria

**Static (verifiable по инструкциям):**
- [x] `story-normalizer.md §4.6` содержит императив: место упомянуто → `location_query` обязателен (D1) — v0.2.6 L317
- [x] `story-normalizer.md §4.6` содержит правило латиницы + ссылку на backend lowercase-нормализацию (D2) — L318
- [x] `api-orchestrator.md §5.2.1` строка `origin.conversation_id` имеет непустой guidance (из контекста, не null) (D3) — v0.3.3 L327
- [x] `story-normalizer.md §4.3` содержит правило активного определения `severity` при достаточном контексте с сохранением no-leading-questions (D4) — L243–251
- [x] Регрессия: omit-условия §4.6 (правила 3–5) и privacy-ограничение §4.3 не ослаблены — L319–321, L245–246

**Advisory (live replay, из кода не верифицируются):**
- [ ] История с упомянутым Таллинном → `location_query` = «Tallinn» (латиница) в запросе (⚠️ Advisory)
- [ ] Каждый запрос → `origin.conversation_id` не null (при доступности контекста) (⚠️ Advisory)
- [ ] История с достаточным контекстом → `gpt_signals` содержит все три поля включая `severity` (⚠️ Advisory)

---

## 5. Не в scope этого REQ

- **Railway deployment / ops** (REQ-47 §3: env vars, `/ready`, миграции `story_signals`) — не-GPT инфраслой, остаётся в [REQ-47](../../doge-complaints-gateway/docs/requirements/47-demo-operational-railway-gpt-fixes.md).
- Изменения backend (geo-расширение, кириллица, `intake_notes`) — [REQ-46](../../doge-complaints-gateway/docs/requirements/46-demo-backend-geo-intake-notes-logging.md).
- `origin.source` / summary generation / gpt_signals enums / location wire-маппинг — уже закрыты (см. §1.1).
- Реальный geocoding API — product backlog.
