# REQ-22: Story Intake Wire Contract v2 — выравнивание GPT-инструкций под сервер

> **Назначение:** зафиксировать требования к обновлению GPT-инструкций для устранения P0 wire-конфликтов между `StoryIntakeRequest`, отправляемым GPT-оркестратором, и intake-контрактом сервера (`m2.story_intake_envelope.v2`).  
> **Технический SoT (сервер):** `doge-complaints-gateway/src/core/intake/contracts.py`, `docs/runtime-docs/api-reference/API_REFERENCE.md §6`.  
> **Анализ исходных gap-ов:** `doge-complaints-gateway/docs/analysis/gap-analysis-gpt-to-server-story-model-2026-05-21.md` (GAP-W-01…05).

**Версия:** 1.0 · 2026-05-21  
**Статус:** requirements — ready for tasking  
**Приоритет:** P0 (блокирует intake: текущие GPT-отправки отвергаются сервером)  
**Источник:** Gap-интервью 2026-05-21, все P0 гапы в одном батче  
**Связанные серверные REQ:** `doge-complaints-gateway/docs/requirements/33-multilingual-story-intake-contract-v2.md`  
**Парные GPT REQ:** [REQ-23](./REQ-23-gpt-behavioral-extensions-pii-signals-consistency.md)

---

## 1. Текущее состояние (broken)

GPT-оркестратор (`GPT UI/instructions/api-orchestrator.md §5.2.1`) и полевой замок (`GPT UI/instructions/story-api-methods-reference.md`) отправляют:

```json
{
  "schema_version": "m2.story_intake_envelope.v1",
  "submitter": {
    "external_user_id": "...",
    // identity_issuer: ОТСУТСТВУЕТ
  },
  "narrative": {
    "original_text": "...",
    "language": "et",
    // session_language: ОТСУТСТВУЕТ
    "title_hint": "...",
    "title_hint_et": "...",
    "title_hint_ru": "...",
    "title_hint_en": "...",
    // title: ОТСУТСТВУЕТ (новый объектный формат)
    // description: ОТСУТСТВУЕТ
  }
}
```

Сервер **отклоняет каждый такой запрос** на уровне `parse_story_intake_request()`:
- `v1` → `IntakeValidationError("schema_version v1 is no longer accepted")` → HTTP 400
- Отсутствие `identity_issuer` → HTTP 400
- Отсутствие `session_language` → HTTP 400
- Отсутствие `title` (объект) → HTTP 400
- Отсутствие `description` → HTTP 400

---

## 2. Целевое состояние (wire contract v2)

После выполнения этого REQ GPT-оркестратор отправляет:

```json
{
  "schema_version": "m2.story_intake_envelope.v2",
  "submitter": {
    "external_user_id": "<conversation_id или иной уникальный ID>",
    "identity_issuer": "dogestonia.gpt.v1"
  },
  "narrative": {
    "original_text": "<description[session_language] из canonical_payload>",
    "language": "<auto-detected язык текста пользователя>",
    "session_language": "<normalization_metadata.session_language>",
    "title": {
      "et": "<canonical_payload.title.et>",
      "ru": "<canonical_payload.title.ru>",
      "en": "<canonical_payload.title.en>"
    },
    "description": {
      "et": "<canonical_payload.description.et>",
      "ru": "<canonical_payload.description.ru>",
      "en": "<canonical_payload.description.en>"
    }
  }
}
```

---

## 3. Изменения по gap-ам

### GAP-W-01: schema_version — константа

| Было | Стало |
|------|-------|
| `"m2.story_intake_envelope.v1"` | `"m2.story_intake_envelope.v2"` |

**Файлы для обновления:**
- `GPT UI/instructions/api-orchestrator.md §5.2.1` — строка со schema_version
- `GPT UI/instructions/story-api-methods-reference.md` — полевой замок, таблица §1.1

---

### GAP-W-02: submitter.identity_issuer — фиксированное значение

| Было | Стало |
|------|-------|
| Поле отсутствует | `"identity_issuer": "dogestonia.gpt.v1"` |

**Семантика:** строковый идентификатор namespace источника. В demo-режиме — фиксированная константа. В будущем может стать динамическим при введении eID/OAuth.

**Файлы для обновления:**
- `GPT UI/instructions/api-orchestrator.md §5.2.1` — добавить поле в `submitter`
- `GPT UI/instructions/story-api-methods-reference.md` — добавить строку в таблицу полей

---

### GAP-W-03: narrative.session_language — из normalization_metadata

| Было | Стало |
|------|-------|
| Поле отсутствует | `"session_language": <normalization_metadata.session_language>` |

**Семантика:** язык интерфейса/сессии GPT-диалога. Уже вычисляется story-normalizer и доступен в `normalized_issue_payload.normalization_metadata.session_language`.

**Маппинг:** `normalized_issue_payload.normalization_metadata.session_language → request.narrative.session_language`

**Файлы для обновления:**
- `GPT UI/instructions/api-orchestrator.md §5.2.1` — добавить маппинг
- `GPT UI/instructions/story-api-methods-reference.md` — добавить строку

---

### GAP-W-04: narrative.title / narrative.description — прямой маппинг объектов

| Было | Стало |
|------|-------|
| `"title_hint": "<строка>"` | Удалить |
| `"title_hint_et/ru/en": "<строка>"` | Удалить |
| (нет) | `"title": {"et": ..., "ru": ..., "en": ...}` |
| (нет) | `"description": {"et": ..., "ru": ..., "en": ...}` |

**Маппинг:**
```
canonical_payload.title.et  → request.narrative.title.et
canonical_payload.title.ru  → request.narrative.title.ru
canonical_payload.title.en  → request.narrative.title.en

canonical_payload.description.et  → request.narrative.description.et
canonical_payload.description.ru  → request.narrative.description.ru
canonical_payload.description.en  → request.narrative.description.en
```

Прямой маппинг: нет трансформаций, только перенос структуры.

**Файлы для обновления:**
- `GPT UI/instructions/api-orchestrator.md §5.2.1` — переписать `narrative` блок
- `GPT UI/instructions/story-api-methods-reference.md §1.1` — обновить таблицу полей

---

### GAP-W-05: narrative.language — язык текста пользователя

| Было | Стало |
|------|-------|
| Поле присутствовало, источник неопределён | Явно: автодетектированный язык оригинального ввода пользователя |

**Семантика:** язык текста, который пользователь набрал в сообщении (может отличаться от `session_language`). Уже определяется story-normalizer при языковом анализе нарратива.

**Маппинг:** `normalization_metadata.detected_input_language → request.narrative.language`

Если story-normalizer не сохраняет `detected_input_language` как отдельное поле — добавить его в `normalization_metadata` в `story-normalizer.md`.

**Файлы для обновления:**
- `GPT UI/instructions/story-normalizer.md` — убедиться что `normalization_metadata` содержит `detected_input_language`
- `GPT UI/instructions/api-orchestrator.md §5.2.1` — источник `narrative.language`

---

## 4. Файлы GPT для изменения

| Файл | Что изменяется |
|------|----------------|
| `GPT UI/instructions/api-orchestrator.md §5.2.1` | Полная перезапись `StoryIntakeRequest` transform: v2, identity_issuer, session_language, title/desc объекты, language source |
| `GPT UI/instructions/story-api-methods-reference.md §1.1` | Обновление field lock table: v2, новые поля, удаление title_hint* |
| `GPT UI/instructions/story-normalizer.md` | Добавить `detected_input_language` в `normalization_metadata` если отсутствует |

---

## 5. Acceptance Criteria

- [ ] GPT отправляет `"schema_version": "m2.story_intake_envelope.v2"` — не v1
- [ ] GPT отправляет `"submitter.identity_issuer": "dogestonia.gpt.v1"` в каждом запросе
- [ ] GPT отправляет `"narrative.session_language"` = язык сессии диалога
- [ ] GPT отправляет `"narrative.title"` как объект `{et, ru, en}` (не `title_hint` строку)
- [ ] GPT отправляет `"narrative.description"` как объект `{et, ru, en}`
- [ ] GPT отправляет `"narrative.language"` = автодетектированный язык текста пользователя
- [ ] `POST /intake/stories` с GPT-payload → HTTP 202 (не HTTP 400)
- [ ] Поля `title_hint`, `title_hint_et/ru/en` удалены из GPT-инструкций

---

## 6. Не в scope этого REQ

- Реальная eID-аутентификация (REQ-19 post-demo)
- PII-детекция и privacy block (REQ-23)
- GPT-сигналы (`gpt_signals` блок) — (REQ-23 + сервер REQ-42)
- institution поле в narrative — (REQ-23 + сервер REQ-43)
- Версионирование `identity_issuer` при multi-platform (решение после demo)
