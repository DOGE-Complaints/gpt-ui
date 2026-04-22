# DOGEstonia: переиспользуемая архитектура GPT UI и мост к Issue / SPA

**Статус:** рабочий референс (верификация по репозиторию, без выдуманных API).  
**Область:** модуль **GPT UI** (`GPT UI/instructions/`, handoff JSON), связка с **`spa-app`** (дашборд Issues).  
**Метод:** мышление `@analysis.mdc` — факты из файлов; пробелы помечены явно.  
**Сквозной SoT продукта:** `docs/DOGEstonia.md`.

## TODO: Точки трансформации под DOGEstonia

- [x] Таблица §1.1–1.2 обновлена по мере очистки (**2026-04-20**): рабочие модули — `issue-*`, `issue-policy-gate`; `activity-*` — stub.
- [x] Канон полей Issue — `issue-data-model.md`; legacy `activity-data-model.md` удалён из active runtime surface.
- [ ] Когда появится operator rulebook — добавить `policy_ref`, версию документа и связь с будущим `issue-policy-gate` вместо отсылок к Kоны Рода как к SoT эталона.
- [ ] После фиксации OpenAPI ноды DOGEstonia — обновить §1.5 и индекс связанных файлов (в т.ч. отдельный YAML для Issues).

**Конвенция маркеров в корне `GPT UI/docs`:** см. `DOGEstonia-doc-root-mirror-and-TODO-convention.md`.

---

## Часть 0. Зеркало: что оператор хочет (упорядоченная формулировка)

Исходный запрос сведён к трём целевым направлениям и одной интеграционной цели.

| # | Направление | Суть |
|---|-------------|------|
| A | **Документ правил валидации / gate** | Вместо legacy контура operator должен поставить **свой** normative-документ (правила допуска, категории отказов, границы контента). Он станет SoT для слоя policy gate + для согласованности с серверной валидацией. |
| B | **Этап «собеседования» (ингест)** | Интервью — не анкета. Нужна **функция мягкого выявления** целей и образов желаемого **социального / общественного пространства** (район, город): то, что респондент не всегда формулирует с первого ответа. Технически это ложится на цепочку `ingest-validation` (+ при необходимости deep parsing), но **содержание вопросов и критерии «достаточности»** задаёт оператор. |
| C | **Выходной JSON под двойное потребление** | Одна структура данных уровня **Issue** (аналогия с JIRA: тип, статус, метки, артефакты) должна разделяться на: (1) поля/наратив для **гос-обработки**, (2) поля/метаданные для **публичной токенизации** (on-chain / Arweave и т.д., в рамках прототипа). Точный список полей — решение оператора + контракт ноды. |
| D | **Совместимость с SPA** | Промежуточный web2-слой (нода) при приёме ответа от GPT **нормализует** данные в ту же форму, что уже потребляет дашборд (`spa-app`), чтобы **не переписывать UI**, а подключить реальный backend. Контракт моков зафиксирован в `spa-app/docs/mock-layer-issues-guide.md` и `spa-app/src/router/mockIssues.js`; расширенная типизация — `spa-app/src/domain/types.js`. |

**Явные допущения документа (не из кода эталона GPT):** что доменная сущность DOGEstonia называется **Issue** и подчиняется контракту SPA; что оператор предоставит rulebook и сценарий интервью; что нода выполнит маппинг GPT JSON → `Issue` (это целевое состояние прототипа, не текущее состояние всех репозиториев).

---

## Часть 1. Экстракция технической reusable-архитектуры (фокус GPT UI)

### 1.1. Иерархия и разделение ответственности

Источник: `GPT UI/instructions/root.md`, `base.md`, `instruction-modules-index.md`, `safety-compliance.md`.

| Уровень | Файл(ы) | Переиспользуемая идея |
|---------|---------|------------------------|
| Обёртка | `root.md` | GPT — оркестратор; не SoT; не утверждать финальные статусы без ответа backend. |
| Конституция режимов | `base.md` | Ровно один активный режим (INGEST / SEARCH / HELP / POLICY); строгое делегирование downstream-модулям. |
| Коммуникация до маршрута | `bootstrap.md`, `communication-presets-reference.md` | Параметры тона/языка/детализации в начале сессии (опционально сокращать для грант-MVP). |
| Безопасность | `safety-compliance.md` | Контрольные точки: raw → extracted → validated → normalized; возможность HALT всего потока. |
| Парсинг | `ingest-deep-parsing.md` | Мультимодальное извлечение во внутренние артефакты без валидации и без API. |
| Валидация структуры | `ingest-validation.md` | Полнота полей, батч-уточнений, `stop_the_line`; без policy gate и без API. |
| Policy gate | `issue-policy-gate.md` | Допуск по внешнему operator rulebook; `policy_gate_result`; без API. |
| Нормализация | `issue-normalizer.md` | `normalized_issue_payload` под схему Issue / ноды. |
| Вызовы API | `api-orchestrator.md` | Единственный модуль HTTP; SSOT **`issue-api-methods-reference.md`** + Issues OpenAPI YAML. |
| Поиск | SEARCH-mode handoff (`base.md` + `api-orchestrator.md`) | Включается только после появления Issue search в OpenAPI. |

**Артефактный handoff (универсально):** между шагами передаются явные отчёты/ссылки на артефакты (как в эталоне: validation report, policy_gate_result, normalized payload ref). Имена полей в DOGEstonia могут отличаться; паттерн «один шаг — один контрактный выход» сохраняется.

### 1.2. Замена legacy-домена на **Issue** (факт состояния репозитория)

| Компонент | Статус |
|-----------|--------|
| `activity-data-model.md` / `activity-normalizer.md` | Legacy files removed from active runtime surface; канон → `issue-data-model.md`, `issue-normalizer.md`. |
| legacy gate module | Legacy file removed from active runtime surface; DOGEstonia использует `issue-policy-gate.md` как актуальный gate-модуль. |
| `api-methods-reference.md` | Legacy donor routes removed from active runtime surface; SSOT Issue → `issue-api-methods-reference.md` + Issues OpenAPI artifact. |

**Инфра-скелет** (режимы, safety, parse → validate → gate → normalize → API) сохранён; доменные тексты переведены на Issue.

### 1.3. Целевой контракт **Issue** для стыкования с SPA (факт из кода)

Ниже — то, что дашборд уже ожидает от in-memory репозитория (см. `mock-layer-issues-guide.md`).

**Обязательные поля объекта Issue (как в моках):**

| Поле | Тип / ограничения |
|------|-------------------|
| `id` | `string` |
| `status` | `ISSUE_STATUS`: `NEW`, `VERIFIED`, `IN_REVIEW`, `ARCHIVED` (`spa-app/src/domain/types.js`) |
| `type` | `ISSUE_TYPE`: `complaint`, `observation`, `absurdity`, `system_bug` |
| `labels` | `string[]` (ключи для фильтра UI; новые — в `AVAILABLE_LABELS` на доске) |
| `title` | `{ et, ru, en }` (гайд требует i18n-объект для моков) |
| `description` | `{ et, ru, en }` |
| `summary` | `{ et, ru, en }` опционально; карточка: `summary ?? title` |

**Опционально:** `institution`, `created_at`, `arweave_txid`, `image_txid`, `image_hash` (фиктивные txid в UI не подставлять — см. гайд).

**Несоходство JSDoc и моков (зафиксировать при маппинге):** в `types.js` для `Issue` допускаются `title`/`description` как `string` или i18n-объект (`isIssue`); мок-гайд для демо требует объект `{ et, ru, en }`. Целевой выход GPT→нода для SPA логично **нормализовать к трём языкам**, даже если один заполнен, а остальные — fallback/пустая строка (решение оператора/ноды).

### 1.4. `IssueIntakePayload` и токенизация (расширение домена в типах SPA)

В `spa-app/src/domain/types.js` описан **`IssueIntakePayload`**: пользователь (`first_name`, `last_name`), `problem_categories`, `description`, `location`, опционально `media_files`, `time`, `severity`, `impact_estimation`, `problem_status`, `related_events`, `metadata: { source, tokenized }`.

Это **кандидат** на:

- «сырой» или полусырой слой после интервью (до свёртки в публичный `Issue` для доски);
- поля, релевантные **гос-логике** (локализация, время, серьёзность, масштаб воздействия).

**Конфликт с privacy эталона GPT** (`root.md`: не собирать PII): использование `user.first_name/last_name` в GPT-потоке **не предполагается автоматически** — только если оператор явно разрешит и нода обезличит до публичного Issue.

### 1.5. Роль web2-слоя (ноды) в мосте GPT → SPA

Логическая схема (целевая, не обязательно уже реализована end-to-end):

```
GPT (инструкции Issue-домена)
  → JSON черновик / нормализованный Issue (+ опционально intake_payload, interview_transcript_ref)
  → нода: валидация схемы, rulebook, присвоение id/status, запрет фиктивных txid
  → тот же JSON shape, что потребляет issueService / репозиторий SPA
```

**Идентификаторы:** в моках — `DE-001` и т.д.; продакшен-формат id задаёт нода.

**Статус после создания:** для новой карточки с доски — как минимум `NEW` (как в примерах моков); переходы `VERIFIED` / `IN_REVIEW` — бизнес-логика вне GPT (или подтверждение API, без ложных claim из эталона `root.md`).

---

## Часть 2. Кастомизация бизнес-слоя: вопросы и задачи для оператора

### 2.1. Доставка артефактов (обязательные входы от оператора)

| ID | Задача | Результат |
|----|--------|-----------|
| **OP-DOC-01** | Подготовить **документ validation / gate rules** (замена Kоны Рода): допустимые типы Issue, запрещённый контент, когда нужен `needs_clarification`, коды причин отказа. | Версионируемый документ (PDF/MD) + `policy_ref` для трассировки в `policy_gate_result`. |
| **OP-DOC-02** | Описать **сценарий собеседования**: фазы диалога, целевые «открыты» (латы скрытых желаний по общественному пространству), признаки достаточности для приостановки вопросов. | Спецификация для правок `ingest-validation` / отдельного подмодуля «Issue intake dialogue». |
| **OP-DOC-03** | Утвердить **канонический JSON Issue** для выдачи в ноду: обязательные/опциональные поля, что уходит в публичный дашборд vs что только в гос-пакет vs что в on-chain метаданные. | JSON Schema или таблица полей с колонками: GPT | нода | SPA | chain. |
| **OP-DOC-04** | Согласовать **словари**: `ISSUE_TYPE`, `labels`, при необходимости регион/муниципалитет, соответствие ведомствам (`institution` i18n). | Список enum + правила маппинга интервью → `labels` / `type`. |
| **OP-DOC-05** | Политика **многоязычия**: обязательны ли все три (`et`, `ru`, `en`) с первого дня или допустим заполненный один язык + машинный/ручной дозаполнение на ноде. | Правило заполнения `title`/`summary`/`description`. |

### 2.2. Углублённые вопросы (требуют решения оператора)

| # | Вопрос |
|---|--------|
| Q1 | Нужен ли **отдельный тип Issue** для «идея улучшения общественного пространства» или достаточно `observation` / `complaint`? |
| Q2 | Должен ли GPT собирать **IssueIntakePayload** целиком, или часть полей (`severity`, `impact_estimation`, `problem_status`) выводится только нодой/модератором? |
| Q3 | Как соотнести **этап собеседования** с «strict protocol» эталона (батч недостающих полей): сохранить батч или сознательно **пошаговую** провокацию для психологической глубины? |
| Q4 | Разрешены ли в интервью **открытые переживания** (миноры, здоровье) — как пересекается с `safety-compliance` и эстонским правом? |
| Q5 | Для **токенизации**: какие поля Issue обязательны в публичном артефакте (хэш нарратива, labels, institution, geo), а какие строго off-chain? |
| Q6 | Нужен ли в MVP **SEARCH** по Issues из GPT или только ingest? |
| Q7 | Подтверждаете ли цель **zero-change SPA**: любые новые поля только как опциональные расширения, не ломающие `BoardPage` / `IssuePage`? |

### 2.3. Следующие шаги в репозитории (после ответов оператора)

1. Поддерживать `issue-data-model.md` и `issue-normalizer.md` как канон strict chain.  
2. Развивать **issue-policy-gate.md** (OP-DOC-01 / demo baseline profile).  
3. Поддерживать `issue-api-methods-reference.md` + OpenAPI artifact под create/update Issue.  
4. Документ маппинга **GPT JSON → `Issue`**, проверка против `isIssue()` / при необходимости расширение типов согласованно с `spa-app`.  
5. Синхронизация `docs/DOGEstonia.md` §2.2–2.3 с принятыми решениями.

---

## Связанные файлы (индекс)

| Назначение | Путь |
|------------|------|
| Зеркало корня docs + конвенция TODO | `GPT UI/docs/DOGEstonia-doc-root-mirror-and-TODO-convention.md` |
| FR модуля GPT Interview (PDF→MD) | `GPT UI/docs/requirements/README.md` |
| Экстракция эталона: JSON → orchestrator → web2 | `GPT UI/docs/extracted-technical-architecture-instructions-to-web2-integration.md` |
| Инструкции GPT (эталон) | `GPT UI/instructions/*.md` |
| Моки и контракт UI Issues | `spa-app/docs/mock-layer-issues-guide.md`, `spa-app/src/router/mockIssues.js` |
| Типы и валидация Issue | `spa-app/src/domain/types.js` |
| Продуктовый SoT | `docs/DOGEstonia.md` |
| Промпт: внешний диалог → MD-постановка интервью | `GPT UI/docs/prompt-external-dialogue-to-issue-interview-functional-spec.md` |

---

## Журнал документа

| Версия | Дата | Изменение |
|--------|------|-----------|
| 0.3 | 2026-04-10 | Шапка: TODO-трансформация; ссылка на зеркало-конвенцию; строка в индексе файлов |
| 0.2 | 2026-04-09 | Индекс: ссылка на промпт внешнего диалога для генерации `issue-interview-functional-spec` |
| 0.1 | 2026-04-09 | Первый выпуск: зеркало оператора, экстракция архитектуры, контракт SPA, бэклог OP-DOC / Q1–Q7 |
