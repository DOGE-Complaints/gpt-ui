# REQ-24: GPT Orchestrator — исправление описания response-контракта

> **Назначение:** устранить устаревшие и неверные описания форматов ответов сервера в `api-orchestrator.md` — error envelope (legacy-формат), статус success-ответа, семантика HTTP 422, и неполная документация lifecycle-поведения.  
> **Источник gap-анализа:** `GPT UI/docs/analysis/gap-analysis-api-reference-vs-gpt-integration-2026-05-22.md` (GAP-1, GAP-2, GAP-5, GAP-7, GAP-8).  
> **Технический SoT (сервер):** `doge-complaints-gateway/docs/runtime-docs/api-reference/API_REFERENCE.md §4, §6.5, §6.6, §6.7`.

**Версия:** 1.0 · 2026-05-22  
**Статус:** requirements — ready for tasking  
**Приоритет:** P1  
**Тип:** документационные исправления — поведение GPT не меняется, только описание контракта  
**Парные REQ:** [REQ-22](./REQ-22-story-intake-wire-contract-v2-alignment.md), [REQ-23](./REQ-23-gpt-behavioral-extensions-pii-signals-consistency.md)

---

## 1. Текущее состояние (broken)

`GPT UI/instructions/api-orchestrator.md` содержит несколько устаревших или неверных описаний, которые остались с донорской эпохи.

### 1.1 Error envelope — legacy-формат (GAP-1)

**Файл:** `api-orchestrator.md §6.1` (~строка 408–416)

Текущее описание:
```json
{
  "success": false,
  "error": "error_code",
  "message": "Human-readable message",
  "details": [...],
  "timestamp": 1640995200,
  "request_id": "req_1234567890"
}
```

Реальный формат сервера (`API_REFERENCE.md §4`, `envelope.py:50-122`):
```json
{
  "error": {
    "code": "DOMAIN_ERROR",
    "type": "domain",
    "message": "Missing or invalid root.submitter.",
    "details": {}
  },
  "trace_id": "abc123"
}
```

Конкретные несоответствия:
- Поле `success` — не существует в API
- `error` — в GPT строка; в сервере объект `{code, type, message, details}`
- `timestamp` — не существует; реальный аналог `trace_id` в top-level
- `request_id` — не существует; реальный аналог `trace_id`

Дополнительно: intake validation errors (`IntakeValidationError`) маппятся в `DOMAIN_ERROR / domain` — не `VALIDATION_ERROR`. Это важно для диагностики ошибок.

---

### 1.2 Success response — неверное значение `status` (GAP-2)

**Файл:** `api-orchestrator.md §6.1` (~строка 422–427)

Текущий пример:
```json
{
  "data": {
    "schema_version": "m2.story_intake_response.v1",
    "story_id": "story_123",
    "status": "received"
  },
  "trace_id": "trace_1234567890"
}
```

Реальные значения `status` (`API_REFERENCE.md §6.6`, `services.py:233-241`):
- `"ready_for_profile"` — все обязательные narrative-поля заполнены
- `"partial_ready"` — история принята, но не все поля заполнены

`"received"` не является допустимым значением `status`. Это значение не отправляется сервером.

---

### 1.3 HTTP 422 — неверная семантика (GAP-5)

**Файл:** `api-orchestrator.md §5.2.3` (~строка 391)

Текущее описание:
```
HTTP 422: schema validation failure on gateway side; log detail field if present; do not fabricate correction.
```

Реальное поведение сервера (`API_REFERENCE.md §6.7`, `handlers.py:166-170`):
```
422: location_query resolved to geo outside CLUSTER_GEO_SCOPE (GeoScopeMismatchError)
error.code = GEO_SCOPE_MISMATCH / validation
```

HTTP 422 не является ошибкой схемы. Это ошибка geo scope — `location_query` разрешился в координаты за пределами настроенного региона (для demo: Эстония / Таллин). Ошибки схемы/валидации → HTTP 400 (`DOMAIN_ERROR`).

---

### 1.4 `gpt_signals` graceful degradation не задокументирована (GAP-7)

**Файл:** `api-orchestrator.md §5.2.3`

Сервер (`API_REFERENCE.md §6.3`):
> "A failure to persist signals is logged and does **not** change HTTP 202 for the intake."

Текущее описание HTTP 202 в §5.2.3 не упоминает, что ошибка записи `gpt_signals` не влияет на ответ — история всё равно принята с кодом 202.

---

### 1.5 Lifecycle `partial_ready` — не задокументировано последствие (GAP-8)

**Файл:** `api-orchestrator.md §5.2.3`

Текущий текст: _"Story is accepted; clustering is deferred. Do not retry on 202."_

Сервер (`API_REFERENCE.md §6.5`):
- `ready_for_profile` → история попадает в cron кластеризации → может стать публичным issue
- `partial_ready` → история **не кластеризуется** до тех пор, пока narrative не будет заполнен

Если GPT сообщает пользователю только "история принята", пользователь не узнает, что история в `partial_ready` не появится в SPA.

---

## 2. Целевое состояние

После выполнения REQ-24 `api-orchestrator.md` содержит корректные описания всех форматов ответов, соответствующие `API_REFERENCE.md`.

### 2.1 Error envelope (исправление GAP-1)

В `§6.1` заменить error envelope пример на:
```json
{
  "error": {
    "code": "DOMAIN_ERROR",
    "type": "domain",
    "message": "Missing or invalid root.submitter.",
    "details": {}
  },
  "trace_id": "abc123"
}
```

С примечанием:
- Поле `error` — всегда объект `{code, type, message, details}`
- `trace_id` — в top-level, всегда присутствует
- Intake validation errors: `code = "DOMAIN_ERROR"`, `type = "domain"` (не `VALIDATION_ERROR`)
- Полная таблица кодов — `API_REFERENCE.md §4.1`

---

### 2.2 Success response (исправление GAP-2)

В `§6.1` заменить success пример на:
```json
{
  "data": {
    "schema_version": "m2.story_intake_response.v1",
    "story_id": "550e8400-e29b-41d4-a716-446655440000",
    "status": "ready_for_profile"
  },
  "trace_id": "abc123"
}
```

С примечанием: `status` может быть `"ready_for_profile"` или `"partial_ready"` (см. §5.2.3).

---

### 2.3 HTTP 422 (исправление GAP-5)

В `§5.2.3` заменить строку про 422 на:
```
HTTP 422 (GEO_SCOPE_MISMATCH): location_query разрешился за пределами настроенного 
geo scope сервера (для demo: Эстония / Таллин). 
Рекомендация: уточнить у пользователя локацию и убедиться что она в пределах 
поддерживаемого региона. Не искать ошибки схемы — это 400 (DOMAIN_ERROR).
```

---

### 2.4 gpt_signals graceful degradation (исправление GAP-7)

В `§5.2.3` дополнить описание HTTP 202:
```
Если в запросе был gpt_signals: ошибка записи сигналов на сервере не меняет HTTP 202.
История принята; gpt_signals могут не попасть в story_signals при DB-ошибке — это логируется 
сервером, но не влияет на intake.
```

---

### 2.5 Lifecycle `partial_ready` (исправление GAP-8)

В `§5.2.3` расширить описание HTTP 202:
```
HTTP 202: история принята.
- status = "ready_for_profile": история пойдёт в кластеризацию; может стать публичным issue.
- status = "partial_ready": история сохранена, но НЕ кластеризуется — narrative неполный 
  (пустое поле title, description или языковой слот). Сообщить пользователю: история сохранена, 
  но потребуется дополнение для публикации.
```

---

## 3. Файлы для изменения

| Файл | Секция | Что изменяется |
|------|--------|----------------|
| `GPT UI/instructions/api-orchestrator.md` | §5.2.3 | HTTP 422 описание; HTTP 202 + partial_ready + gpt_signals graceful degradation |
| `GPT UI/instructions/api-orchestrator.md` | §6.1 | Error envelope пример; success response status значение |

---

## 4. Acceptance Criteria

- [ ] `api-orchestrator.md §6.1` error envelope пример совпадает с `API_REFERENCE.md §4`
- [ ] `api-orchestrator.md §6.1` success response содержит `status: "ready_for_profile"` (не `"received"`)
- [ ] `api-orchestrator.md §5.2.3` HTTP 422 описание ссылается на geo scope mismatch (не schema validation)
- [ ] `api-orchestrator.md §5.2.3` HTTP 202 содержит семантику `ready_for_profile` vs `partial_ready`
- [ ] `api-orchestrator.md §5.2.3` содержит примечание о graceful degradation gpt_signals
- [ ] Ни одно из изменений не затрагивает wire-контракт (только описания)

---

## 5. Не в scope этого REQ

- Изменение wire-полей запроса (covered by REQ-22/23)
- Активация `canonical_type`, `canonical_labels` (REQ-25)
- Добавление `location_query` в normalizer (REQ-26)
- Изменение поведения GPT при PII (REQ-23, уже реализовано)
