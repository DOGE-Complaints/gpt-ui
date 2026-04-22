# Экстракция технической архитектуры: инструкции GPT UI → JSON по диалогу → web2 API

**Назначение:** зафиксировать **переиспользуемый технический паттерн** текущего Issue-контура из `GPT UI/instructions/*.md`: как пошагово формируются структуры данных в процессе диалога, **кто имеет право** инициировать вызовы backend и **как** это стыкуется с web2 (REST + Custom GPT Actions).  
**SoT по коду инструкций:** файлы в `GPT UI/instructions/`; для **DOGEstonia Issue** контракт HTTP — **`GPT UI/instructions/issue-api-methods-reference.md`** и импортированный Actions contract (build/import artifact: `GPT UI/docs/custom-gpt-issues-reference.openapi.yaml`).  
**Метод:** `@analysis.mdc` — только проверяемые факты из актуальных секций; legacy donor тела вырезаны (**2026-04-20**, strategy **C**).

**Версия:** 0.3 · 2026-04-20

---

## 1. Два слоя: «логика в инструкциях» и «реальный HTTP»

| Слой | Что это | Как проявляется |
|------|---------|-----------------|
| **Инструкции (модель в чате)** | Правила поведения GPT: модули, очередность, JSON-артефакты, stop-the-line | Текст в system/developer instructions; модель **описывает** и **соблюдает** артефакты в ответах (и при необходимости показывает их пользователю в debug-режиме по `bootstrap.md` / presets). |
| **Web2 / FastAPI** | Фактические REST-вызовы | В продукте OpenAI это слой **GPT Actions**: платформа выполняет HTTP по **OpenAPI**-схеме, когда модель выбирает operation (function calling). Секрет и базовый URL задаёт оператор (Actions import + mapping doc). |

**Критическое правило из индекса модулей:** модули **Ingest Validation**, **Deep Parsing**, **Gate**, **Normalizer** **не** «звонят на сервер» сами по себе — это прямо сказано для validation (`instruction-modules-index.md`: *never calls APIs directly*). Единственный модуль, отвечающий за **все** обращения к backend API в эталоне — **`api-orchestrator.md`**.

**Root:** контекст между модулями не смешивается — только **явный handoff** (`root.md`).

---

## 2. Конвейер INGEST: модули и границы ответственности

Порядок и роли (сжато по `instruction-modules-index.md`):

1. **Base** — режим INGEST, маршрутизация, (в strict) протокол артефактов и stop-the-line (`base.md` §1.5).
2. **Safety** — может прервать любой поток; несколько контрольных точек (`safety-compliance.md`).
3. **Ingest Deep Parsing** — для немультимодального/сложного ввода: извлечение → `deep_parsing_artifact`; без валидации и без API.
4. **Ingest Validation** — полнота полей, отчёт `ingest_validation_report`, при необходимости пакет для Gate `gate_request_package`; **без API**.
5. **`issue-policy-gate`** (DOGEstonia) — допуск / отказ / needs_clarification; **без API**; `policy_gate_result`.
6. **`issue-normalizer.md`** — `normalized_issue_payload` (strict); **без API**.
7. **API Orchestrator** — HTTP по **`issue-api-methods-reference.md`** / Issues OpenAPI для Issue; legacy donor роуты не являются рабочим контуром DOGEstonia.

**Normalizer ↔ Validation:** нормализатор **не** опрашивает пользователя напрямую — только через Validation ([`issue-normalizer.md`](../instructions/issue-normalizer.md) границы).

---

## 3. Как в инструкциях задаётся «JSON по мере диалога»

### 3.1. Инкрементальность

Диалог наращивает структуру: пользователь отвечает → Validation пересчитывает недостающие поля → при non-dialogue input сначала обязателен **deep parsing**, затем вопросы **по артефакту** (`ingest-validation.md`: не задавать вопросы до `deep_parsing_artifact`).

### 3.2. Strict Protocol Mode — обязательная цепочка артефактов

В `base.md` §1.5 задано: каждый шаг workflow **обязан** выдать **версионированный JSON-артефакт** перед переходом дальше. Перечень (порядок):

| # | Артефакт | Кто создаёт | Назначение (кратко) |
|---|-----------|-------------|---------------------|
| 1 | `deep_parsing_artifact` | Ingest Deep Parsing | Условно, если вход не чистый диалог; поля извлечения, ambiguities, missing_required_fields и т.д. |
| 2 | `ingest_validation_report` | Ingest Validation | readiness, missing_required_fields, `stop_the_line.blocked`, ссылки на deep_parsing |
| 3 | `safety_compliance_report` | Safety | несколько **check_point**: raw / extracted / validated / normalized; decision allow/redact/block |
| 4 | `gate_request_package` | Ingest Validation (до Gate) | validated payload + refs на validation/safety; **без** решения Gate |
| 5 | `normalized_issue_payload` | Issue Normalizer | `canonical_payload` + `normalization_metadata` (refs на validation, safety, gate) |

**Stop-the-line** (фрагмент логики из `base.md`): при непустом `missing_required_fields`, при safety redact/block, при невыполненных условиях для SentToReview — **не** переходить к Normalizer / Gate / API и **не** вызывать API.

### 3.3. Что именно уходит «на сервер»

Не промежуточные отчёты целиком, а **операции**, определённые **API Orchestrator** по правилам из `api-orchestrator.md`:

- Вход orchestrator для INGEST (Issue): **`normalized_issue_payload`** и контракт из Issues OpenAPI.
- Перед вызовом orchestrator проверяет минимальный input contract и **не** подменяет ответ backend.

Соответствие **типа операции** ↔ **HTTP** задаётся в `issue-api-methods-reference.md` (например `POST /issues/draft`).

### 3.4. SEARCH-поток (кратко)

**Search Dialogue** (stub / продукт) — при появлении Issue search в OpenAPI строит JSON запроса и передаёт **API Orchestrator**. Создание/изменение сущностей в этом потоке не выполняется.

---

## 4. Интеграция с web2: как это зашито в репозитории

### 4.1. SSOT по HTTP

- **`issue-api-methods-reference.md`** + imported Actions contract (`custom-gpt-issues-reference.openapi.yaml`) — SSOT Issue для Actions.

### 4.2. Механизм вызова (Custom GPT)

Инструкции **требуют** от модели выбрать действие orchestrator; **фактический HTTP** выполняет платформа OpenAI по OpenAPI operation. Поэтому reuse «рабочего кода» на стороне сервера — это **реализация контрактов** в FastAPI + стабильная схема Actions; reuse на стороне GPT — это **те же** модули и те же правила артефактов, с заменой legacy доменной схемы на Issue.

### 4.3. Идентичность и заголовки

В контуре Actions сохраняется нюанс: OpenAI Actions **не** поддерживают произвольные заголовки → `X-User-Id` типично **не** передаётся из GPT; backend должен работать через поддерживаемые auth/params (детально в `gpt-actions-bot-api-auth-mapping.md`).

---

## 5. Что переносить в DOGEstonia без поломки паттерна

| Паттерн эталона | Что меняется в DOGEstonia |
|-----------------|---------------------------|
| Цепочка модулей ingest | Сохранить; заменить Gate rulebook (не Kоны Рода) и целевую схему **Issue**. |
| Имена артефактов (`*_payload`, `*_report`) | Можно сохранить **префиксы** и идею `artifact_id` / `version: v1`; для текущего контура использовать `normalized_issue_payload`. |
| Единственная точка HTTP | Сохранить: один **issue-api-orchestrator** + новый OpenAPI для ноды. |
| Strict / stop_the_line | Сохранить как дисциплину CI/CD в диалоге. |
| Safety check_point | Сохранить идею многоступенчатых проверок. |

---

## 6. Индекс исходных файлов (для правок)

| Тема | Файл |
|------|------|
| Иерархия, handoff | `GPT UI/instructions/root.md` |
| Режимы, strict, артефакты | `GPT UI/instructions/base.md` |
| Описание модулей | `GPT UI/instructions/instruction-modules-index.md` |
| Validation + отчёт | `GPT UI/instructions/ingest-validation.md` |
| Deep parsing | `GPT UI/instructions/ingest-deep-parsing.md` |
| Gate (DOGEstonia) | `GPT UI/instructions/issue-policy-gate.md` |
| Normalizer | `GPT UI/instructions/issue-normalizer.md` |
| HTTP / операции | `GPT UI/instructions/api-orchestrator.md` |
| Контракт API (Issue) | `GPT UI/instructions/issue-api-methods-reference.md` |
| Safety | `GPT UI/instructions/safety-compliance.md` |
| Search | SEARCH-mode handoff in `base.md` + `api-orchestrator.md` (when search endpoints exist) |
| OpenAPI для Actions (Issue) | `GPT UI/docs/custom-gpt-issues-reference.openapi.yaml` |

---

## 7. План адаптации эталонных инструкций под FR DOGEstonia

**SoT по продукту:** `GPT UI/docs/requirements/REQ-01`…`REQ-18` (перенос из PDF + **REQ-18** — связка GPT ↔ API Story Intake).  
**Принцип:** сохранить **технический каркас** (§1–6), заменить legacy-домен, тексты gate-донора, поля валидации и HTTP-контракт на **Issue**, ноду DOGEstonia и при необходимости контур **`StoryIntakeEnvelope`** (см. `doge-complaints-gateway/docs/requirements/19-inbound-api-gpt-preprocessing-and-spa-issue-contracts.md`).

### 7.1. Трассировка FR → модули инструкций (рабочая матрица)

| Фокус FR (PDF) | Где отражать в instruction set |
|----------------|--------------------------------|
| Миссия, философия интервью, фазы 1–7 (`REQ-01`, `REQ-05`, `REQ-08`) | `base.md` (маршрутизация), **новый** модуль «issue interview dialogue» или расширение `ingest-validation.md` (см. §7.3) |
| Психологическая модель, 7 вопросов completeness (`REQ-06`, `REQ-07`) | Тот же модуль + критерии в `ingest_validation_report` / readiness |
| FR-M1-xxx (`REQ-09`) | Разнести по base, validation, normalizer, gate, safety, orchestrator |
| Content-модель выхода (`REQ-10`, **REQ-18**) | Схема **Issue** + mapping в normalizer; полный intake — **`StoryIntakeEnvelope`** (§19 gateway) |
| Блоки A–H для набора инструкций (`REQ-11`) | Оглавление репозитория `instructions/` + `instruction-modules-index.md` |
| Anti-patterns, AC (`REQ-12`, `REQ-14`) | Тест-планы / чеклисты; частично `root.md` + safety |
| Допущения ingest-only (`REQ-15`) | `base.md`: режим SEARCH не обязателен для модуля 1 |

### 7.2. Переименование файлов (кандидаты)

Имеет смысл **переименовать** там, где имя файла вводит в заблуждение после смены домена. После переименования обязательно обновить **все** перекрёстные ссылки (см. §7.4).

| Эталон (legacy donor) | Кандидат (DOGEstonia) | Примечание |
|------------------|----------------------|------------|
| `activity-normalizer.md` | `issue-normalizer.md` | Канонизация под Issue + `normalized_issue_payload` в strict-цепочке |
| `activity-data-model.md` | `issue-data-model.md` | Поля SPA + narrative layers |
| legacy gate module | `issue-policy-gate.md` | SoT = operator rulebook / demo policy profile |
| `custom-gpt-actions-activities-reference.openapi.yaml` | `custom-gpt-issues-reference.openapi.yaml` | OpenAPI build/import artifact под ноду Issue |

Файлы **`root.md`**, **`base.md`**, **`ingest-validation.md`**, **`ingest-deep-parsing.md`**, **`safety-compliance.md`**, **`api-orchestrator.md`** можно **не** переименовывать на первом шаге, но внутри — массово заменить термины и ссылки на `issue-*` / `Issue`.

### 7.3. Новые файлы для углубления интервью (кандидаты)

Чтобы не раздувать `ingest-validation.md` продуктовой психологией из `REQ-05`–`REQ-08`, уместно вынести:

| Новый файл (кандидат) | Содержание |
|----------------------|------------|
| `issue-interview-flow.md` | Фазы 1–7, допустимые формулировки входа, reframe, запрет ранней классификации (синхрон с `REQ-08`, `REQ-12`) |
| `issue-interview-extraction.md` | Детализация блоков из FR-M1-019 (`raw_story`, `deep_need`, …) и связь с `REQ-10` |
| `issue-i18n-policy.md` | FR-M1-028–031: политика `{ et, ru, en }`, fallback, запрет искажения смысла |

Подключение: из **`base.md`** / **`ingest-validation.md`** — явная ссылка «при INGEST Interview активировать …» по правилам иерархии из `root.md`.

### 7.4. Порядок работ (минимальный, без поломки ссылок)

1. Зафиксировать **issue-data-model.md** и целевой OpenAPI ноды (или черновик).  
2. **Скопировать** эталонные файлы в ветку/префикс `dogestonia/` *или* править на месте — по выбору команды; при правке на месте — один коммит = один крупный блок (gate → normalizer → orchestrator).  
3. Переименовать файлы из §7.2 **одним** изменением + `grep` по `GPT UI/instructions` и `GPT UI/docs` на старые имена.  
4. Обновить **`instruction-modules-index.md`** и упоминания в **`root.md`**.  
5. Прогнать чеклист **`REQ-14`** против нового текста инструкций.

### 7.5. Что не трогать без необходимости

- Общую **иерархию** root → base → safety → модули.  
- Правило **один активный функциональный модуль** и **explicit handoff**.  
- Роль **единственного** HTTP-модуля (orchestrator), пока тот же паттерн устраивает.

---

## TODO (DOGEstonia-transform)

- [ ] Сопоставить каждый артефакт strict-цепочки с полями из `GPT UI/docs/requirements/REQ-10-output-content-model.md` (narrative → Issue projection).
- [ ] После появления OpenAPI ноды Issue — дублировать в этот документ таблицу «операция orchestrator ↔ HTTP» для новых путей.
- [ ] Явно описать, будет ли strict protocol **обязателен** для Interview Layer MVP (см. `REQ-15` допущения ingest-only).
- [ ] Выполнить §7.2–7.4 (переименования, новые модули, обновление индекса и grep ссылок).

---

## Журнал

| Версия | Дата | Изменение |
|--------|------|-----------|
| 0.2 | 2026-04-10 | §7: план адаптации под FR, переименования, новые файлы интервью, порядок работ |
| 0.1 | 2026-04-10 | Первый выпуск: экстракция из `GPT UI/instructions` + связка с web2/Actions |
