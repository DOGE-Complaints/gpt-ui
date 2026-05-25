# REQ-28: `institution` demo-constraint enforcement — normalizer + orchestrator gate

> **Назначение:** добавить явный код-гейт в GPT-инструкции (normalizer и/или orchestrator pre-flight), запрещающий передачу `institution` в `StoryIntakeRequest` в demo-scope. Сейчас ограничение существует только в `story-data-model.md §4.2` как текстовое предупреждение, но нигде не реализовано как проверяемое правило в pipeline.  
> **Источник:** `GPT UI/docs/audit-field-provenance-2026-05-24.md` (FINDING-03)  
> **Технический контекст:** Pipeline (`story-normalizer.md`, `api-orchestrator.md`, `contracts.py`, DB) полностью готов принять и сохранить `institution` — ограничение существует только в `story-data-model.md §4.2` («Demo scope: do NOT populate from dialogue»).

**Версия:** 1.0 · 2026-05-25  
**Статус:** requirements — ready for tasking  
**Приоритет:** P2 (demo-integrity risk; нет HTTP-breakage, но сохранение поля нарушит demo-scope контракт)  
**Тип:** GPT instruction update — `story-normalizer.md` + `api-orchestrator.md`; сервер не затрагивается  
**Серверная сторона:** не требует изменений  
**Парные REQ:** [REQ-22](./REQ-22-story-intake-wire-contract-v2-alignment.md) (маппинг institution), [REQ-43](../../../doge-complaints-gateway/docs/requirements/43-institution-json-story-column.md) (серверная колонка)

---

## 1. Текущее состояние (проблема)

### 1.1 Где зафиксирован demo-constraint

**`GPT UI/instructions/story-data-model.md §4.2`** (строка 66):
```markdown
| `institution` | `{ et, ru, en }` | Agency / institution **if** product scope allows inferring it 
  from interview. **Demo scope:** do **not** populate from dialogue; keep field **absent** / 
  null in `canonical_payload` until integration matures. |
```

### 1.2 Где отсутствует enforcement

`story-normalizer.md §4.3` — нет правила, запрещающего выводить `institution` в demo-scope.

`api-orchestrator.md §5.2` pre-flight — нет проверки на presence of `institution`.

`api-orchestrator.md §5.2.1` field mapping table — строка `narrative.institution` (строка 333):
```
| narrative.institution | canonical_payload.institution | No | If present: omit if any slot empty; demo scope: do not populate |
```

Текст "demo scope: do not populate" присутствует как hint в mapping-таблице orchestrator, но:
1. Это hint на отправку, не правило на генерацию
2. Нет явного pre-flight шага, который отбрасывает поле если оно попало в `canonical_payload`
3. Normalizer не документирует demo-constraint явно

### 1.3 Риск

Если GPT-агент обновится или изменится промпт, `institution` может:
- Попасть в `canonical_payload.institution` через normalizer
- Пройти через orchestrator field mapping (hint проигнорирован)
- Сохраниться в `stories.institution_json` (REQ-43 колонка)
- Войти в `doge_issues.payload_json` через promotion pipeline

Без code-gate это silent violation — сервер не вернёт ошибку.

---

## 2. Целевое состояние

### 2.1 `story-normalizer.md §4.2` — добавить явный demo-constraint block

В секцию §4.2 (optional fields) добавить явное правило для `institution`:

```markdown
**`institution` — demo-scope constraint (M1 demo):**  
Do **not** populate `canonical_payload.institution` in current demo scope, regardless of 
what GPT infers from the interview. If GPT reasoning suggests an institution:
- Record it in `non_wire_metadata.institution_candidate` (informational only, not wired)
- Do not write to `canonical_payload.institution`

This constraint is lifted when backend integration for institution-routing matures 
(tracked as REQ-43 / `43-institution-json-story-column.md`).
```

### 2.2 `api-orchestrator.md §5.2` pre-flight check — добавить institution guard

В pre-flight validation block (`api-orchestrator.md §5.2`, перед §5.2.1 field mapping):

Добавить check item:
```markdown
**#[N] institution demo-gate:** If `canonical_payload.institution` is present → **omit** 
it from `narrative.institution` wire field. Log: "demo scope: institution omitted 
(REQ-28)". Do not raise an error — silently drop the field. This gate is active until 
REQ-43 integration is complete.
```

### 2.3 `api-orchestrator.md §5.2.1` — обновить hint строки `institution`

**Было:**
```
| narrative.institution | canonical_payload.institution | No | If present: omit if any slot empty; demo scope: do not populate |
```

**Стало:**
```
| narrative.institution | canonical_payload.institution | No | **Always omit in current demo scope** — pre-flight check #[N] drops this field regardless of canonical_payload content (REQ-28). Activated post-demo: omit only if any i18n slot empty. |
```

---

## 3. Файлы для изменения

| Файл | Секция | Что изменяется |
|------|--------|----------------|
| `GPT UI/instructions/story-normalizer.md` | §4.2 optional fields | Добавить explicit demo-constraint block для `institution` |
| `GPT UI/instructions/api-orchestrator.md` | §5.2 pre-flight | Добавить institution guard item |
| `GPT UI/instructions/api-orchestrator.md` | §5.2.1 field mapping table | Обновить hint строки `narrative.institution` |
| `GPT UI/instructions/story-normalizer.md` | Version header + history | Bump → 0.2.2 · 2026-05-25 |

---

## 4. Acceptance Criteria

- [ ] `story-normalizer.md §4.2` содержит явное правило "do not populate `canonical_payload.institution` in demo scope"
- [ ] `story-normalizer.md §4.2` содержит fallback: сохранить кандидата в `non_wire_metadata.institution_candidate` если GPT вывел учреждение
- [ ] `api-orchestrator.md §5.2` pre-flight содержит явный guard: если `canonical_payload.institution` присутствует → omit from wire
- [ ] `api-orchestrator.md §5.2.1` строка `narrative.institution` явно указывает "Always omit in current demo scope"
- [ ] `story-normalizer.md` версия ≥ `0.2.2` с changelog-записью REQ-28

---

## 5. Не в scope этого REQ

- Изменения на серверной стороне (`contracts.py`, `services.py`, DB schema) — сервер корректно принимает institution; REQ-43 управляет серверной стороной
- Реализация полноценной institution-routing интеграции (post-demo scope)
- Изменения в `story-data-model.md §4.2` — текст уже корректен; enforcement добавляется в normalizer и orchestrator
- Изменения в OpenAPI schema
- Изменения в interview flow (institution не спрашивается в demo)
