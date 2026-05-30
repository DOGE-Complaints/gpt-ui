# REQ-30: Admission gate для `postStoryIntake` — запрет без strict-chain артефактов и sandbox-only тестовых отправок

> **Назначение:** добавить в GPT instruction pipeline явный admission gate, запрещающий вызов `postStoryIntake` без полного набора upstream-артефактов strict-chain; тестовые/мусорные submissions должны быть заблокированы или направлены в sandbox. В текущем состоянии GPT может вызвать `postStoryIntake` напрямую после минимального пользовательского подтверждения вроде «ага», не выполнив обязательную цепочку: `validation → safety → policy gate → normalization → API`.  
> **Технический контекст:** Action/tool `postStoryIntake` технически принимает валидный Story Intake envelope, даже если он был собран ad hoc — без доказанного наличия `ingest_validation_report`, `safety_compliance_report`, `policy_gate_result`, `normalized_issue_payload`. Pipeline guard отсутствует.

**Версия:** 1.0 · 2026-05-29  
**Статус:** requirements — ready for tasking  
**Приоритет:** P1 (data integrity / analytics pollution / operator trust risk)  
**Тип:** GPT instruction update — `api-orchestrator.md` §5.2 pre-flight; дополнительно server-side guardrail (SHOULD)  
**Серверная сторона:** рекомендованные изменения (SHOULD) — reject non-substantive production payloads; HTTP 422 `INTAKE_NOT_SUBSTANTIVE`  
**Парные REQ:** [REQ-18](./REQ-18-api-inbound-story-intake-and-gpt-handoff.md) (API handoff contract), [REQ-22](./REQ-22-story-intake-wire-contract-v2-alignment.md) (wire contract)

---

## 1. Текущее состояние (проблема)

### 1.1 Отсутствие admission gate

В текущем поведении GPT может вызвать `postStoryIntake` напрямую после минимального пользовательского подтверждения вроде «ага», даже если не выполнена обязательная цепочка:

`validation → safety → policy gate → normalization → API`

Это позволяет отправить в backend запись, которая не прошла полноценный DOGEstonia Issue intake и может загрязнять базу.

### 1.2 Root cause

Есть конфликт между:

1. Инструкциями, где API Orchestrator должен вызываться только после `normalized_issue_payload` и upstream-artifacts.
2. Доступностью прямого Action/tool `postStoryIntake`, который GPT может вызвать без доказанного наличия:

   * `ingest_validation_report`
   * `safety_compliance_report`
   * `policy_gate_result`
   * `normalized_issue_payload`

API-инструмент технически принимает валидный Story Intake envelope, даже если он был собран ad hoc.

### 1.3 Серьёзность

**High** — data integrity / analytics pollution / operator trust risk.

---

## 2. Целевое состояние

### 2.1 Обязательные preconditions перед вызовом `postStoryIntake`

GPT MUST NOT call `postStoryIntake` unless the current conversation contains a complete strict-chain handoff package for a real or explicitly allowed test submission.

Required preconditions before API call:

```json
{
  "required_before_postStoryIntake": {
    "ingest_validation_report": {
      "exists": true,
      "stop_the_line.blocked": false
    },
    "safety_compliance_report": {
      "exists": true,
      "decision": "allow",
      "check_point": "validated"
    },
    "policy_gate_result": {
      "exists": true,
      "status": "approved"
    },
    "normalized_issue_payload": {
      "exists": true,
      "source": "story-normalizer",
      "contains_required_fields": [
        "canonical_payload.type",
        "canonical_payload.labels",
        "canonical_payload.title",
        "canonical_payload.description",
        "normalization_metadata.session_language"
      ]
    },
    "explicit_user_confirmation": {
      "exists": true,
      "confirmation_text_must_reference": "submission to DOGEstonia backend"
    }
  }
}
```

### 2.2 Test-data exception — sandbox-only тестовые отправки

Test submissions MUST NOT use production Story Intake by default.

Allowed only if one of these is true:

```json
{
  "test_submission_allowed_if": [
    "environment == sandbox",
    "payload.test_mode == true AND backend stores separately",
    "operator_role == authorized_tester AND explicit_test_confirmation == true"
  ]
}
```

If none are true, GPT MUST refuse backend submission and offer a local preview only.

### 2.3 GPT behavior при попытке тестовой/мусорной отправки

When user says something like "давай отправим какую-нибудь хуйню" or "отправь тест", GPT MUST respond:

> Я не могу отправлять мусорные или тестовые записи в production DOGEstonia. Могу подготовить локальный тестовый payload для просмотра или отправить только в sandbox/test endpoint, если он доступен.

### 2.4 Server-side guardrail (SHOULD)

`postStoryIntake` SHOULD reject obvious test or placeholder payloads in production unless `test_mode=true` is supported and routed away from production analytics.

Suggested server-side validation:

```json
{
  "reject_in_production_if": [
    "title or description contains only test/check/placeholder semantics",
    "canonical_type is generic observation with no civic issue substance",
    "description explicitly says no real issue is described",
    "missing location/context/problem/affected/desired-state signals"
  ],
  "response": {
    "status": 422,
    "code": "INTAKE_NOT_SUBSTANTIVE",
    "message": "Story Intake must describe a substantive civic issue or be sent to a sandbox/test endpoint."
  }
}
```

---

## 3. Файлы для изменения

| Файл | Секция | Что изменяется |
|------|--------|----------------|
| `GPT UI/instructions/api-orchestrator.md` | §5.2 pre-flight checks | Добавить admission gate: проверка наличия всех 5 upstream-артефактов strict-chain перед вызовом `postStoryIntake` |
| `GPT UI/instructions/api-orchestrator.md` | §5.2 pre-flight checks | Добавить test-mode exception rule: reject non-sandbox test submissions с ответом из §2.3 |
| `GPT UI/instructions/api-orchestrator.md` | §5.2 / audit trace | Добавить требование логировать artifact IDs, которые обосновали API-вызов |
| `GPT UI/instructions/api-orchestrator.md` | Version header + history | Bump версии + changelog REQ-30 |
| `doge-complaints-gateway` | Intake endpoint validation | (SHOULD) Добавить rejection логику для non-substantive production payloads; HTTP 422 `INTAKE_NOT_SUBSTANTIVE` |

---

## 4. Acceptance Criteria

- [ ] GPT never calls `postStoryIntake` without `normalized_issue_payload`
- [ ] "ага", "ок", "отправь тест", "закинь любую хрень" are not sufficient confirmation for production API
- [ ] Test payloads cannot enter production analytics
- [ ] Backend rejects non-substantive production submissions even if GPT fails
- [ ] Audit log records which artifact IDs justified the API call
- [ ] If strict-chain artifacts are absent, GPT says: **"Локальная валидация FAIL: данных недостаточно для отправки в API."**

---

## 5. Не в scope этого REQ

- Изменения в interview flow (Phase 1–7) — gate только в orchestrator pre-flight, не в interview
- Изменения в `story-normalizer.md`, `story-interview-flow.md`, `story-data-model.md`
- Реализация `test_mode` flag в `StoryIntakeRequest` контракте и `contracts.py` (требует отдельного REQ)
- Реализация sandbox/test endpoint на сервере (product backlog)
- Изменения в OpenAPI schema (если `test_mode` параметр не добавляется в этом REQ)
- Изменения в `ingest-validation.md`, `safety-compliance.md`, `policy-gate.md` — upstream артефакты; их формат не меняется данным REQ
