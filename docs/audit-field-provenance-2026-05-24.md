# Audit: Честность сбора данных — Провенанс полей StoryIntakeRequest

**Дата аудита:** 2026-05-24  
**Аудитор:** Claude Sonnet 4.6 (automated)  
**Объект:** Все 22 поля `StoryIntakeRequest` → для каждого задокументирован реальный источник данных  
**Метод анализа:** `analysis.mdc` — сквозная трассировка данных от пользователя до HTTP-тела  

---

## 1. Цель и объём

Цель: для каждого поля `StoryIntakeRequest` честно ответить на вопрос — **откуда реально берётся это значение?** Явно разграничить:
- что пользователь сказал сам
- что GPT перевёл (механически)
- что GPT переформулировал / civic-framed
- что GPT вывел из контекста
- что захардкожено
- что берётся из технического контекста сессии

**Источники анализа:**
- `story-interview-flow.md` (фазы 1–7 — что GPT спрашивает и когда)
- `story-normalizer.md §4.*` (как формируется `normalized_issue_payload`)
- `api-orchestrator.md §5.2.1` (трансформ → HTTP)
- `contracts.py` (как сервер принимает и хранит)
- REQ-22, REQ-23, REQ-26 semantics

---

## 2. Классификация источников

| Класс | Код | Определение |
|-------|-----|-------------|
| Прямой ввод пользователя | **USER_DIRECT** | Пользователь явно сказал это в интервью; GPT передаёт без переработки смысла |
| GPT перевод | **GPT_TRANSLATED** | GPT механически переводит user-заявленный контент на другие языки |
| GPT переформулировка | **GPT_REFRAMED** | GPT переработал нарратив пользователя — civic framing, структурирование, сжатие |
| GPT вывод из контекста | **GPT_INFERRED** | GPT выводит значение из косвенных признаков; пользователь это явно не формулировал |
| Захардкожено | **GPT_HARDCODED** | Всегда одно и то же фиксированное значение; не зависит от интервью |
| Технический контекст сессии | **SESSION_TECH** | Берётся из технических метаданных сессии/платформы GPT |
| Условное | **CONDITIONAL** | Зависит от логики выбора; источник меняется в зависимости от ветки |

---

## 3. Провенанс поля за полем

### 3.1 Блок `schema_version`

| Поле | Класс | Источник | Комментарий |
|------|-------|----------|-------------|
| `schema_version` | **GPT_HARDCODED** | Всегда `"m2.story_intake_envelope.v2"` | Технический контракт; не зависит от пользователя. Не содержит никаких пользовательских данных. |

---

### 3.2 Блок `submitter`

| Поле | Класс | Источник | Комментарий |
|------|-------|----------|-------------|
| `submitter.external_user_id` | **SESSION_TECH** (demo: mock) | Wallet address из session context | В production: реальный wallet пользователя из auth-сессии. В demo M2: mock-значение. Пользователь это значение не произносит и не видит. |
| `submitter.identity_issuer` | **GPT_HARDCODED** | Всегда `"dogestonia.gpt.v1"` | Маркер интеграционного канала. Не данные пользователя. |

**Флаг honesty:** В demo-режиме `external_user_id` — мок. Записи в БД будут атрибутированы mock-идентификатору, а не реальному пользователю. При переходе в production это поле должно быть реальным wallet.

---

### 3.3 Блок `narrative` — языковые метаданные

| Поле | Класс | Источник | Комментарий |
|------|-------|----------|-------------|
| `narrative.language` | **GPT_INFERRED** | Auto-detected by GPT из текста пользователя | GPT определяет язык ввода (`detected_input_language`) на основе распознавания языка в нарративе. Пользователь это явно не объявляет. Ошибки детекции возможны на коротких текстах или смешанных языках. |
| `narrative.session_language` | **SESSION_TECH** | `bootstrap.md` `comm_context.ui_lang` — язык сессии/UI | Устанавливается при запуске сессии, не в ходе интервью. Может отличаться от `narrative.language` (например, UI на английском, нарратив на эстонском). |

---

### 3.4 Блок `narrative` — текстовые поля

| Поле | Класс | Источник | Детали |
|------|-------|----------|--------|
| `narrative.title.{et,ru,en}` | **GPT_REFRAMED** + **GPT_TRANSLATED** | Фазы 2–6 интервью + Phase 7 synthesis | Версия на `session_language`: GPT генерирует civic-framed однострочный заголовок из всего интервью. Остальные 2 языка: GPT переводит. |
| `narrative.description.{et,ru,en}` | **GPT_REFRAMED** + **GPT_TRANSLATED** | Фазы 2–5 интервью | Версия на `session_language`: GPT формирует структурированное гражданское описание. Остальные 2 языка: GPT переводит. |
| `narrative.original_text` | **GPT_REFRAMED** | `canonical_payload.description[session_language]` | **⚠ КЛЮЧЕВОЙ ВЫВОД:** поле называется `original_text`, но содержит НЕ дословные слова пользователя. Это GPT-переформулированное описание на языке сессии (REQ-22 §2, intentional design). Операторы, видящие `original_text` в БД, могут ошибочно считать его verbatim-вводом. |
| `narrative.summary.{et,ru,en}` | **GPT_REFRAMED** + **GPT_TRANSLATED** | Синтез всего интервью | GPT генерирует короткий summary из интервью. Пользователь его не диктует. Версия на session_language — reframed, остальные — translated. |

---

### 3.5 Блок `narrative` — классификационные поля

| Поле | Класс | Источник | Детали |
|------|-------|----------|--------|
| `narrative.canonical_type` | **GPT_INFERRED** | Фазы 3, 4, 6 интервью → civic framing | GPT классифицирует в `complaint | observation | absurdity | system_bug` на основе характера нарратива. Пользователь тип не объявляет; GPT выводит из описания проблемы и гражданского контекста. |
| `narrative.canonical_labels` | **GPT_INFERRED** | `story-label-taxonomy.md` + весь нарратив | GPT извлекает теги из контролируемой таксономии. В `canonical_labels[]` попадают только `disposition=canonical` кандидаты. Само извлечение — GPT reasoning; пользователь теги не называет. |
| `narrative.institution` | **GPT_INFERRED** + **GPT_TRANSLATED** (но **только если ≠ demo-scope**) | Phase 6 (civic framing) | GPT может вывести учреждение из описания. `story-data-model.md §4.2` явно запрещает населять это поле в demo-scope: «Demo scope: do NOT populate from dialogue». Если поле присутствует в demo — это нарушение demo-constraint orchestratora. |
| `narrative.location_query` | **USER_DIRECT** | Phase 2 ("где") + §7.2 affirmation | **Единственное текстовое поле с USER_DIRECT источником.** Per §4.6: устанавливается только если пользователь явно назвал и подтвердил место. GPT не должен добавлять location из контекста. Если пользователь назвал несколько мест — GPT выбирает наиболее специфичное (D-02), что уже добавляет editorial judgment. |

---

### 3.6 Блок `origin`

| Поле | Класс | Источник | Комментарий |
|------|-------|----------|-------------|
| `origin.source` | **GPT_HARDCODED** | Всегда `"openai_gpt_action"` | Трассировка канала; не данные пользователя. |
| `origin.conversation_id` | **SESSION_TECH** | Thread/conversation ID из GPT-платформы | Техническое. Пользователь не видит и не контролирует. |
| `origin.tool_call_id` | **SESSION_TECH** | Tool invocation ID при GPT Actions вызове | Техническое. Пусто если не tool-call. |

---

### 3.7 Блок `privacy`

| Поле | Класс | Источник | Детали |
|------|-------|----------|--------|
| `privacy.contains_pii` | **GPT_INFERRED** | `story-normalizer.md §4.4` PII scan | GPT сканирует `canonical_payload.description.*` на имена, адреса, телефоны, email, ID-номера. Conservative policy: при low-confidence → тоже `true`. Пользователь напрямую это не заявляет. |
| `privacy.redaction_requested` | **USER_DIRECT** | §5.2.0 two-step PII flow — явный выбор пользователя | Устанавливается ТОЛЬКО если `contains_pii = true`. `true` = пользователь согласился редактировать. `false` = пользователь отказался. Активный выбор пользователя. |

**Флаг honesty:** `privacy.redaction_requested = false` означает, что пользователь **отказался** редактировать PII, не то что PII нет. Наличие `privacy` блока само по себе означает, что PII был обнаружен (`contains_pii = true` всегда когда блок присутствует — orchestrator §5.2.0 omits block when false).

---

### 3.8 Блок `gpt_signals`

| Поле | Класс | Источник | Детали |
|------|-------|----------|--------|
| `gpt_signals.severity` | **GPT_INFERRED** | Inference из всего нарратива | GPT оценивает серьёзность из civic framing, описания последствий, интенсивности нарратива. Пользователь severity не называет в явном виде. Высоко субъективно — ни один вопрос интервью не запрашивает "насколько это серьёзно по 4-уровневой шкале". |
| `gpt_signals.impact_estimation` | **GPT_INFERRED** | Inference из географического контекста и масштаба | GPT выводит из описания ("весь район", "только я"), упомянутых территорий. Пользователь scope не называет в административных единицах. |
| `gpt_signals.problem_status` | **CONDITIONAL** (USER_DIRECT + GPT_INFERRED) | Phase 2–3 (что происходит) + temporal markers | Пользователь может явно сказать "всё ещё продолжается" или "уже решили" — тогда USER_DIRECT. Если не сказал — GPT выводит из глагольных времён, контекста. Граница между прямым заявлением и выводом в runtime неотличима. |

---

### 3.9 Блок `live_story_context`

| Поле | Класс | Источник | Детали |
|------|-------|----------|--------|
| `live_story_context.consistency_notes` | **GPT_INFERRED** | GPT детектирует противоречия в ходе интервью | GPT фиксирует противоречия между фазами (дата, место, статус, ответственный орган). Пользователь сам противоречие не маркирует. Поле — editorial judgment GPT. |

---

## 4. Сводная таблица провенанса

| # | Поле | Класс | Пользователь видит? | Risk |
|---|------|-------|---------------------|------|
| 1 | `schema_version` | GPT_HARDCODED | Нет | Нет |
| 2 | `submitter.external_user_id` | SESSION_TECH (demo: mock) | Нет | Demo: не атрибутирован реальному пользователю |
| 3 | `submitter.identity_issuer` | GPT_HARDCODED | Нет | Нет |
| 4 | `narrative.language` | GPT_INFERRED | Нет | Детекция может ошибаться |
| 5 | `narrative.session_language` | SESSION_TECH | Нет | Нет |
| 6 | `narrative.title.*` | GPT_REFRAMED + GPT_TRANSLATED | Да (Phase 7 подтверждение) | Перевод на языки кроме session_language не подтверждается |
| 7 | `narrative.description.*` | GPT_REFRAMED + GPT_TRANSLATED | Да (Phase 7) | То же |
| 8 | `narrative.original_text` | GPT_REFRAMED | **Скрыт** | Имя вводит в заблуждение — не verbatim |
| 9 | `narrative.canonical_type` | GPT_INFERRED | Phase 7 частично | Классификация влияет на issue promotion gate |
| 10 | `narrative.canonical_labels` | GPT_INFERRED | Нет | Влияет на кластеризацию |
| 11 | `narrative.summary.*` | GPT_REFRAMED + GPT_TRANSLATED | Нет (до Phase 7) | Перевод без подтверждения |
| 12 | `narrative.location_query` | USER_DIRECT | Да (§7.2 affirmation) | Multi-location → editorial choice |
| 13 | `origin.source` | GPT_HARDCODED | Нет | Нет |
| 14 | `origin.conversation_id` | SESSION_TECH | Нет | Нет |
| 15 | `origin.tool_call_id` | SESSION_TECH | Нет | Нет |
| 16 | `narrative.institution` | GPT_INFERRED (+TRANSLATED) | Нет (demo: absent) | Demo: must be absent |
| 17 | `privacy.contains_pii` | GPT_INFERRED | Да (§5.2.0 disclosure) | Conservative → false positives нормальны |
| 18 | `privacy.redaction_requested` | USER_DIRECT | Да (явный выбор) | false ≠ "нет PII" |
| 19 | `gpt_signals.severity` | GPT_INFERRED | Нет | Субъективно; не подтверждается пользователем |
| 20 | `gpt_signals.impact_estimation` | GPT_INFERRED | Нет | Субъективно; пользователь не использует эти административные категории |
| 21 | `gpt_signals.problem_status` | CONDITIONAL | Нет | Граница USER vs INFERRED неотличима в runtime |
| 22 | `live_story_context.consistency_notes` | GPT_INFERRED | Нет | Editorial judgment без подтверждения |

---

## 5. Ключевые находки

### FINDING-01 — `original_text` не содержит оригинального текста пользователя

**Серьёзность:** HIGH (информационная честность)  
**Суть:** Поле `narrative.original_text` в `StoryIntakeRequest` (и в `StoryRecord` в БД) содержит `canonical_payload.description[session_language]` — то есть GPT-переформулированное описание на языке сессии (REQ-22 §2, intentional design).

Это сознательное архитектурное решение (не баг), но:
1. Операторы и аналитики, видящие колонку `original_text` в БД, ожидают verbatim-ввод пользователя
2. Если пользователь когда-либо видит `original_text` из ответа API — он увидит GPT-версию своих слов как "оригинал"
3. В правовых/жалобных контекстах это разрыв между тем, что пользователь сказал, и тем, что зафиксировано

**Соответствие документации:** REQ-22 §2 явно указывает этот дизайн. Orchestrator §5.2.1 mapping-таблица корректно указывает источник. **Это не gap в реализации — это gap в явности документирования для операторов.**

**Рекомендация:** Добавить в `API_REFERENCE.md §6.3` примечание к `narrative.original_text`:
> "Note: not the verbatim user input but the GPT-reframed description in session_language per REQ-22 §2. Verbatim capture is outside the current scope."

---

### FINDING-02 — `gpt_signals` — три поля без явного user-заявления

**Серьёзность:** MEDIUM (прозрачность данных)  
**Суть:** `severity`, `impact_estimation`, `problem_status` полностью определяются GPT. Ни один вопрос interview-flow не спрашивает у пользователя напрямую: "как бы вы оценили серьёзность по шкале LOW/MEDIUM/HIGH/CRITICAL?" или "это LOCAL/DISTRICT/CITY/NATIONAL проблема?".

Эти значения отправляются в `story_signals` (REQ-42) как `gpt.story_classifier.v1` — что технически верно (это GPT-классификация). Но разрыв важен: данные представляются как "signals" без явной маркировки "GPT-only inference, not user-stated".

**Риск:** Downstream аналитика или ML на этих полях будет смешивать GPT-инференс с реальными user-данными.

**Рекомендация:** В `story_signals` таблице source уже маркируется как `"gpt_intake_v1"` — это частично митигирует риск. Документировать явно в API_REFERENCE что `gpt_signals.*` = GPT classification, not user-stated values.

---

### FINDING-03 — `institution` запрещён в demo, но pipeline его поддерживает

**Серьёзность:** LOW  
**Суть:** `story-data-model.md §4.2` явно запрещает населять `institution` из диалога в demo-scope. Тем не менее весь pipeline (normalizer, orchestrator, contracts.py, db layers) готов принять и сохранить это поле.

Если future GPT-агент или обновлённый normalizer начнёт выводить institution и не соблюдёт demo-constraint — данные пройдут без ошибки. Ограничение существует только в инструкции, не в code-gate.

---

### FINDING-04 — Переводы на неосновные языки не подтверждаются пользователем

**Серьёзность:** LOW  
**Суть:** `title`, `description`, `summary` — для двух языков, отличных от `session_language`, являются GPT_TRANSLATED. Пользователь видит и подтверждает только версию на своём языке (Phase 7). Переводы на остальные языки попадают в публичный `DOGEIssue.payload_json` без просмотра пользователем.

**Риск:** Ошибки перевода в публичной записи. В civic context возможно искажение смысла жалобы при переводе на другой язык.

---

## 6. Статистика по классам источников

| Класс | Кол-во полей | % |
|-------|-------------|---|
| GPT_HARDCODED | 4 | 18% |
| SESSION_TECH | 4 | 18% |
| GPT_INFERRED | 7 | 32% |
| GPT_REFRAMED + GPT_TRANSLATED | 4 | 18% |
| USER_DIRECT | 2 | 9% |
| CONDITIONAL (mixed) | 1 | 5% |

**Вывод:** Из 22 полей только **2 поля** содержат прямой user input без трансформации GPT:
- `narrative.location_query` (USER_DIRECT — подтверждено Phase 7)
- `privacy.redaction_requested` (USER_DIRECT — явный выбор в PII flow)

Все текстовые нарративные поля — переформулированы или переведены GPT. Все классификационные поля — GPT inference. Это соответствует архитектурному дизайну продукта, но важно понимать при интерпретации данных.

---

## 7. Рекомендации

| # | Действие | Файл/место | Приоритет |
|---|----------|------------|-----------|
| R-01 | Добавить явное примечание к `original_text` в API_REFERENCE.md §6.3: "GPT-reframed, not verbatim" | `API_REFERENCE.md §6.3` | Medium |
| R-02 | Документировать в API_REFERENCE что `gpt_signals.*` = GPT classifier output, не user-stated | `API_REFERENCE.md §6` (новый §6.8 или примечание к §6.6) | Low |
| R-03 | Рассмотреть добавление raw verbatim capture как отдельного поля (вне scope текущего sprint) | Product backlog | Low |
| R-04 | Рассмотреть Phase 7 confirmation scope для переводов (хотя бы spot-check) | `story-interview-flow.md` | Low |

---

## 8. Регрессии

Аудит не затрагивает код — только документирование существующего behaviour. Тест-суит не затронут.
