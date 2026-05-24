# REQ-23: GPT Behavioral Extensions — PII-детекция, gpt_signals, consistency_notes

> **Назначение:** зафиксировать требования к новым поведенческим функциям GPT на стыке моделей данных — дополнительно к wire-contract исправлениям (REQ-22).  
> **Охват:** три независимых расширения: (1) PII-детекция и запрос на редактуру, (2) передача классификационных сигналов в intake, (3) заполнение `consistency_notes` при противоречиях.

**Версия:** 1.0 · 2026-05-21  
**Статус:** requirements — ready for tasking  
**Приоритет:** P1  
**Источник:** Gap-интервью 2026-05-21 (Q6, Q7, Q8, Q9)  
**Связанные серверные REQ:**  
- `doge-complaints-gateway/docs/requirements/42-gpt-signals-story-intake-extension.md` (gpt_signals server side)  
- `doge-complaints-gateway/docs/requirements/43-institution-json-story-column.md` (institution server side)  
**Парный REQ:** [REQ-22](./REQ-22-story-intake-wire-contract-v2-alignment.md) (wire contract fixes)

---

## 1. Расширение A: PII-детекция и запрос на редактуру

### 1.1 Контекст

Сервер имеет поля `privacy.contains_pii` и `privacy.redaction_requested` (оба boolean, default `false`). GPT сейчас не заполняет блок `privacy` — всегда отправляется дефолт `false`.

Решение: GPT детектирует PII в пользовательском тексте и активирует двухшаговый флаг.

### 1.2 Детекция PII

GPT (story-normalizer + api-orchestrator) должен распознавать PII в `narrative.original_text` и `canonical_payload.description.*`:

| Тип PII | Примеры |
|---------|---------|
| Имя человека | «Иван Петров», «Mari Tamm» |
| Адрес | «Liivalaia 10-5», «ул. Садовая 5» |
| Телефон | «+372 5...», «8 (800)...» |
| Email | «ivan@...» |
| Личный идентификатор | ИК, паспорт, ID-kaart номер |

**Правило:** если GPT с высокой уверенностью определяет хотя бы один из этих типов — помечает нарратив как `contains_pii = true`.

Если **уверенность низкая** — предпочтительнее `contains_pii = true` (консервативная позиция), а не `false`.

### 1.3 Двухшаговый запрос редактуры (interaction flow)

Если `contains_pii = true`, GPT **до отправки** intake выполняет:

```
Шаг 1: GPT сообщает пользователю:
  «Я заметил, что ваш рассказ содержит личные данные [X].
   Хотите ли вы убрать их перед отправкой?»

Шаг 2a: Пользователь соглашается →
  GPT помогает с редактурой, затем:
  - privacy.contains_pii = true
  - privacy.redaction_requested = true

Шаг 2b: Пользователь отказывается →
  - privacy.contains_pii = true
  - privacy.redaction_requested = false
```

### 1.4 Intake payload

```json
{
  "privacy": {
    "contains_pii": true,
    "redaction_requested": true
  }
}
```

Если PII не обнаружен — блок `privacy` можно опустить (сервер применяет default `false`).

### 1.5 Файлы для изменения

| Файл | Что изменяется |
|------|----------------|
| `GPT UI/instructions/story-normalizer.md` | Добавить §: PII detection logic, когда ставить `contains_pii` |
| `GPT UI/instructions/api-orchestrator.md` | Добавить §: pre-send PII check, interaction flow, `privacy` блок в request |
| `GPT UI/instructions/story-api-methods-reference.md` | Добавить строку `privacy` в field lock table |

### 1.6 Acceptance Criteria (A)

- [ ] Если narrative содержит имя/адрес/телефон → GPT задаёт вопрос о редактуре перед отправкой
- [ ] Если пользователь соглашается → `privacy.contains_pii: true, redaction_requested: true` в запросе
- [ ] Если пользователь отказывается → `privacy.contains_pii: true, redaction_requested: false`
- [ ] Если PII не обнаружен → `privacy` блок отсутствует или `contains_pii: false`
- [ ] Сервер принимает запрос с `privacy` блоком → HTTP 202

---

## 2. Расширение B: gpt_signals — передача классификационных сигналов

### 2.1 Контекст

GPT генерирует `non_wire_metadata` с полями:
- `severity`: `"LOW"` / `"MEDIUM"` / `"HIGH"` / `"CRITICAL"`
- `impact_estimation`: `"LOCAL"` / `"DISTRICT"` / `"CITY"` / `"NATIONAL"`
- `problem_status`: `"ONGOING"` / `"RESOLVED"` / `"RECURRING"` / `"UNKNOWN"`

Принято решение: передавать их на сервер через новый блок `gpt_signals`. Сервер персистирует в `story_signals` (REQ-42, `extraction_policy = "gpt.story_classifier.v1"`).

### 2.2 Маппинг

```
non_wire_metadata.severity            → gpt_signals.severity
non_wire_metadata.impact_estimation   → gpt_signals.impact_estimation
non_wire_metadata.problem_status      → gpt_signals.problem_status
```

### 2.3 Intake payload

```json
{
  "gpt_signals": {
    "severity": "HIGH",
    "impact_estimation": "DISTRICT",
    "problem_status": "ONGOING"
  }
}
```

Блок опциональный. Если все три значения вычислены → передавать. Если GPT не уверен хотя бы в одном → использовать `"UNKNOWN"` / пропустить поле.

### 2.4 Переименование в инструкциях

`non_wire_metadata` остаётся внутренним промежуточным структурой story-normalizer (для backwards compat и для отображения пользователю при необходимости). `gpt_signals` — это wire-name для сервера.

В `api-orchestrator.md` явно задокументировать: «`gpt_signals` в запросе формируется из `non_wire_metadata`».

### 2.5 institution — дополнительное поле narrative

Параллельно с `gpt_signals` — добавить маппинг `institution`:

```
canonical_payload.institution.{et,ru,en} → narrative.institution.{et,ru,en}
```

Условие отправки: если GPT сгенерировал `institution` с не-пустыми значениями для всех трёх языков → включить в `narrative.institution`. Если хотя бы один ключ пустой — не включать.

Это устраняет текущий «demo omit» и обеспечивает передачу на сервер (REQ-43).

### 2.6 Файлы для изменения

| Файл | Что изменяется |
|------|----------------|
| `GPT UI/instructions/api-orchestrator.md §5.2.1` | Добавить `gpt_signals` блок в StoryIntakeRequest; маппинг из non_wire_metadata |
| `GPT UI/instructions/api-orchestrator.md` | Добавить `narrative.institution` маппинг |
| `GPT UI/instructions/story-api-methods-reference.md` | Добавить строки `gpt_signals.*` и `narrative.institution` |

### 2.7 Acceptance Criteria (B)

- [ ] GPT отправляет `gpt_signals` блок с severity/impact_estimation/problem_status
- [ ] После intake: `story_signals` на сервере содержит запись с policy=`gpt.story_classifier.v1`
- [ ] GPT отправляет `narrative.institution` если canonical_payload.institution заполнен
- [ ] После intake: `stories.institution_json` = переданный объект `{et,ru,en}`

---

## 3. Расширение C: consistency_notes при противоречиях

### 3.1 Контекст

Сервер имеет поле `live_story_context.consistency_notes` (текстовое, уже персистируется в `stories.narrative_consistency_notes`). GPT сейчас это поле не заполняет.

Решение: GPT заполняет `consistency_notes` когда в нарративе пользователя обнаружены противоречия или неопределённости, требующие документирования.

### 3.2 Критерии заполнения

GPT устанавливает `consistency_notes` (свободный текст, язык — `session_language` или `en`) если:

| Ситуация | Пример |
|---------|--------|
| Временное противоречие | «Сказал что проблема началась в 2020, но затем упомянул что видел это «несколько недель назад»» |
| Географическое противоречие | «Упомянул два разных района» |
| Субъект-ответственность неясна | «Не ясно, городская или государственная ответственность» |
| Статус проблемы неоднозначен | «Одновременно сказал что решено и что всё ещё происходит» |

**Формат:** свободный текст, одно-два предложения, фиксирует именно противоречие (не пересказывает суть жалобы).

### 3.3 Intake payload

```json
{
  "live_story_context": {
    "consistency_notes": "Пользователь упомянул два разных адреса проблемы — Sadama 5 и Liivalaia 7."
  }
}
```

Если противоречий нет — блок `live_story_context` можно опустить или передать без `consistency_notes`.

### 3.4 Файлы для изменения

| Файл | Что изменяется |
|------|----------------|
| `GPT UI/instructions/story-normalizer.md` | Добавить §: когда и как формировать consistency_notes |
| `GPT UI/instructions/api-orchestrator.md §5.2.1` | Добавить `live_story_context.consistency_notes` в StoryIntakeRequest |

### 3.5 Acceptance Criteria (C)

- [ ] Если пользователь рассказывает с явными противоречиями → GPT включает `consistency_notes` в запрос
- [ ] После intake: `stories.narrative_consistency_notes` содержит переданный текст
- [ ] Если противоречий нет → `consistency_notes` отсутствует в запросе (не передаётся как пустая строка)

---

## 4. Общие Acceptance Criteria

- [ ] Все три расширения (A, B, C) реализованы в GPT-инструкциях независимо
- [ ] `POST /intake/stories` с полным payload (privacy + gpt_signals + institution + consistency_notes) → HTTP 202
- [ ] Существующее поведение (без расширений A/B/C) остаётся совместимым — все три блока опциональны

---

## 5. Не в scope этого REQ

- Серверная обработка `redaction_requested = true` (контент-редактура, REQ-31)
- Аналитика на основе gpt_signals (отдельная задача)
- Нормализация/классификация institution (canonical institution registry)
- UI отображение consistency_notes в SPA (SA-07)
