# REQ-32: Origin traceability — enforce sending `origin.source` в каждом StoryIntakeRequest

> **Назначение:** устранить двусмысленность, которая приводит к тому, что `origin` блок не отправляется в API-запросе: GPT либо пропускает его, либо не заполняет `origin.source`. В результате `origin_source = null` в БД, хотя код backend корректно сохраняет поле при наличии. Проблема в инструкции и аннотации OpenAPI YAML.  
> **Источник:** `GPT UI/docs/analysis/1.06.26-testing-gap-report.md` §2 — `origin_source` null при ожидаемом `openai_gpt_action`  
> **Технический контекст:** `services.py:208` корректно маппит `request.origin.source → origin_source` в БД. Если поле null в БД — значит `origin.source` не было в запросе. Backend не виноват.

**Версия:** 1.0 · 2026-06-01  
**Статус:** requirements — ready for tasking  
**Приоритет:** P2 (traceability gap — нет runtime breakage, но диагностика инцидентов без origin_source существенно сложнее)  
**Тип:** GPT instruction update — `api-orchestrator.md` §5.2.1; `custom-gpt-story-intake-actions.openapi.yaml` annotation fix  
**Серверная сторона:** не требует изменений (backend код корректен)  
**Парные REQ:** [REQ-30](./REQ-30-admission-gate-story-intake-strict-chain.md) (admission gate + audit trace), [REQ-31](./REQ-31-God-mode-activation.md) (origin description changed in REQ-31)

---

## 1. Текущее состояние (проблема)

### 1.1 Аннотация в OpenAPI YAML создаёт двусмысленность

`custom-gpt-story-intake-actions.openapi.yaml` — текущая аннотация поля `origin` (добавлена в REQ-31 / GIM-137):

```yaml
origin:
  $ref: "#/components/schemas/Origin"
  description: >
    Optional traceability sidecar. Prefer omitting from citizen summaries (REQ-31 §2.5.3).
```

Фраза **"Prefer omitting from citizen summaries"** предназначалась для UI: «не показывай origin-детали пользователю в citizen preview». Но в контексте GPT, который читает этот YAML как описание поля запроса, фраза читается иначе: «в Citizen Mode — не включай в запрос». В результате `origin` блок может быть пропущен.

### 1.2 Инструкция не содержит явного MUST для origin.source

`api-orchestrator.md §5.2.1` field mapping table (строка ~326):
```
| origin.source | Fixed: openai_gpt_action | No | Traceability; closes GAP-04 when set |
```

- Поле помечено **No** (optional).
- Комментарий "when set" подразумевает, что GPT вправе его не установить.
- Нет явного требования всегда включать `origin` блок.

### 1.3 Последствие в тест-данных

`origin_source = null` в БД при отправке через GPT Action. Без `origin_source` невозможно:
- Отследить, через какой канал пришла история
- Связать историю с конкретным GPT conversation для диагностики
- Разделить intake-источники в аналитике (GPT Action vs SPA vs API)

---

## 2. Целевое состояние

### 2.1 Исправить аннотацию в OpenAPI YAML

**Было:**
```yaml
origin:
  description: >
    Optional traceability sidecar. Prefer omitting from citizen summaries (REQ-31 §2.5.3).
```

**Стало:**
```yaml
origin:
  description: >
    Optional traceability sidecar. Always include origin.source='openai_gpt_action' in
    every StoryIntakeRequest regardless of Citizen/God Mode. Do NOT omit from the API
    request body — "omit from citizen summaries" means do not display origin fields in
    the conversational preview, not that the block should be absent from the request.
```

### 2.2 Добавить явное правило в api-orchestrator.md §5.2.1

В field mapping table, строку `origin.source` изменить:

**Было:**
```
| origin.source | Fixed: openai_gpt_action | No | Traceability; closes GAP-04 when set |
```

**Стало:**
```
| origin.source | Fixed: openai_gpt_action | **de-facto required** | Always include: this is the only way to track submission source. Value is fixed = "openai_gpt_action" for GPT Action runtime. Do not omit — origin.source = null in DB means the story cannot be attributed to the GPT channel. |
```

### 2.3 Добавить правило в §5.2.0b (Citizen Mode / God Mode section)

В секции §5.2.0b явно указать, что транспортные поля для traceability отправляются в API независимо от режима:

```markdown
**Transport fields vs display fields:** Citizen Mode controls what is *shown* to the user,
not what is *sent* in the API request. `origin`, `schema_version`, `identity_issuer`,
`gpt_signals` — always included in the StoryIntakeRequest body; never described to the
citizen in conversational preview unless God Mode is active.
```

---

## 3. Файлы для изменения

| Файл | Секция | Что изменяется |
|------|--------|----------------|
| `GPT UI/docs/custom-gpt-story-intake-actions.openapi.yaml` | `components.schemas.StoryIntakeRequest.properties.origin.description` | Убрать двусмысленность "prefer omitting"; добавить "always include origin.source" |
| `GPT UI/instructions/api-orchestrator.md` | §5.2.1 field mapping table, строка origin.source | Изменить `No` → `de-facto required`; расширить описание |
| `GPT UI/instructions/api-orchestrator.md` | §5.2.0b Citizen Mode / God Mode | Добавить явное разграничение "transport fields vs display fields" |
| `GPT UI/instructions/api-orchestrator.md` | Version header + history | Bump версии + changelog REQ-32 |

---

## 4. Acceptance Criteria

- [ ] YAML annotation для `origin` не содержит фраз "omit" без явного уточнения контекста (display vs API request)
- [ ] `api-orchestrator.md §5.2.1` строка `origin.source` содержит явное требование включать поле в каждый запрос
- [ ] `api-orchestrator.md §5.2.0b` содержит правило: transport fields всегда в теле запроса, независимо от Citizen/God Mode
- [ ] После следующей отправки через GPT Action: `origin_source = "openai_gpt_action"` в БД

---

## 5. Не в scope этого REQ

- Изменения backend (код корректен, проблема на GPT стороне)
- `origin.conversation_id` — ChatGPT Actions не предоставляет conversation_id в контексте GPT; null ожидаем и корректен на данном этапе.
- `origin.tool_call_id` — **Статус под вопросом:** api-orchestrator.md говорит "Tool invocation id when submit runs inside a tool call". Тест-репорт §2 фиксирует "Не сохраняется трассировка". Технически ChatGPT Actions выполняет submit как tool call — tool_call_id существует на уровне платформы, но недоступен в conversational context GPT. Если в будущем платформа предоставит доступ к tool_call_id → добавить в REQ-32 scope. Пока: **known limitation**, не functional bug.
- Изменения в интервью или нормализаторе
