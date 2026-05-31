# REQ-31: Dual-mode action confirmation UX — Citizen Mode default + God Mode debug toggle

> **Назначение:** добавить в GPT два режима отображения перед вызовом `postStoryIntake`: **Citizen Mode** (по умолчанию) — понятный человекоязычный preview без технических деталей; **God Mode** (активируется секретным токеном) — полная техническая видимость payload, схем и метаданных для операторов и разработчиков. Устранить утечку внутренних деталей реализации (`operationId`, `schema_version`, `gpt_signals`) в citizen-facing UX.  
> **Источник:** продуктовое требование — UX quality / operator observability (автор: Product Management)

**Версия:** 1.0 · 2026-05-29  
**Статус:** requirements — ready for tasking  
**Приоритет:** P2 (UX quality risk; нет runtime-breakage, но citizen UX некорректен без этого)  
**Тип:** GPT instruction update — `api-orchestrator.md` pre-submit preview; новый instruction block для debug-token activation; `custom-gpt-story-intake-actions.openapi.yaml` (action naming + description)  
**Серверная сторона:** не требует изменений  
**Парные REQ:** [REQ-30](./REQ-30-admission-gate-story-intake-strict-chain.md) (admission gate / confirmation flow), [REQ-22](./REQ-22-story-intake-wire-contract-v2-alignment.md) (OpenAPI contract lockstep)

---

## 1. Текущее состояние (проблема)

### 1.1 Утечка реализации в citizen-facing UX

Текущий Action confirmation UI раскрывает внутренние детали транспортного слоя:

- имена операций (`postStoryIntake`)
- идентификаторы схем (`m2.story_intake_envelope.v2`)
- транспортные структуры
- внутренние метаданные (`gpt_signals`)
- implementation-oriented форму payload

Это удобно для дебаггинга, но создаёт плохой citizen-facing experience.

### 1.2 Ожидание пользователя vs реальность

Обычный пользователь должен воспринимать:

> «Я отправляю историю»

а не

> «Я вызываю операцию postStoryIntake со schema_version m2.story_intake_envelope.v2»

Текущий UX создаёт ощущение незавершённого продукта и генерирует вопросы: «что такое schema_version?», «что такое gpt_signals?», «это безопасно?»

### 1.3 Серьёзность

**Medium** — нет технического сбоя, но UX качество не соответствует civic product. Операторы и разработчики теряют возможность инспекции payload если ввести упрощение без debug-режима.

---

## 2. Целевое состояние

### 2.1 Два режима

#### Mode A — Citizen Mode (по умолчанию)

`debug_mode = false`

Оптимизирован для:
- доверия
- простоты
- эмоционального комфорта
- низкой когнитивной нагрузки

Пользователь видит только значимую информацию.

#### Mode B — God Mode (debug)

`debug_mode = true`

Оптимизирован для:
- разработчиков
- операторов
- продуктового дебаггинга
- инспекции payload

Пользователь видит именно то, что видно сегодня.

Без упрощения. Без скрытия. Без трансформаций.

---

### 2.2 Механизм активации God Mode

#### 2.2.1 Секретный debug-токен

GPT поддерживает специальную activation phrase.

Пример:
```
XXX
```
(фактическое значение задаётся оператором)

При обнаружении фразы:
```
debug_mode = true
```
для текущего conversation.

#### 2.2.2 Требования безопасности

Токен:
- **MUST NOT** появляться в UI help
- **MUST NOT** появляться в onboarding
- **MUST NOT** быть предложен GPT самостоятельно
- **MUST NOT** быть раскрыт когда пользователи спрашивают:
  - «как включить debug»
  - «какие команды существуют»
  - «покажи скрытые режимы»

Токен — знание оператора.

#### 2.2.3 Session scope

После активации `debug_mode = true` сохраняется только для текущего conversation.

Не глобально. Не cross-session.

---

### 2.3 Требования Citizen Mode (`debug_mode = false`)

#### 2.3.1 Язык

Только человеческий язык.

Запрещены:
- `schema`
- `payload`
- `operationId`
- `JSON`
- `API`
- `envelope`
- `gpt_signals`
- `canonical_payload`
- `normalization_metadata`

#### 2.3.2 Submission Preview

Перед выполнением action GPT должен показать:

```
Ready to submit:

Title:
Night noise from neighbour

Summary:
Repeated night shouting while gaming.

Location:
Pille Street, Tallinn

Would you like to submit?
```

#### 2.3.3 Confirmation Copy

Предпочтительная формулировка:

```
Submit Story
```

Не:

```
postStoryIntake
```

#### 2.3.4 Transport Abstraction

Citizen mode **никогда** не описывает внутренние поля:

```
schema_version
identity_issuer
gpt_signals
origin
canonical_type
canonical_labels
```

Вместо этого — назначение поля. Пример:

```
Category: Complaint
```

вместо

```
canonical_type=complaint
```

---

### 2.4 Требования God Mode (`debug_mode = true`)

#### 2.4.1 Zero Simplification

Отобразить точно:
- payload
- schema version
- labels
- generated metadata
- severity
- transport fields
- all mappings

Без редакции. Без абстракций.

#### 2.4.2 Debug Banner

Рекомендуется:

```
DEBUG MODE ACTIVE
```

видимый в GPT responses.

#### 2.4.3 Payload Preview

GPT может отобразить:

```json
{
  "schema_version": "...",
  "narrative": {...},
  "gpt_signals": {...}
}
```

перед submission.

---

### 2.5 Улучшения Action Metadata (независимо от режима)

#### 2.5.1 Rename Action

Текущее:
```
postStoryIntake
```

Предпочтительное:
```
Submit Story
```
или
```
Send Story
```

#### 2.5.2 Action Description

Текущий стиль:
```
POST /intake/stories
```

Предпочтительный:
```
Send a citizen story to DOGEstonia for processing.
```

#### 2.5.3 Reduce Visible Technical Fields

Пересмотреть OpenAPI schema и вынести implementation-only fields из top-level request где возможно.

Цель: ChatGPT confirmation dialog должен раскрывать минимум transport noise.

---

## 3. Файлы для изменения

| Файл | Секция | Что изменяется |
|------|--------|----------------|
| `GPT UI/instructions/api-orchestrator.md` | §5.2 pre-submit (до HTTP call) | Добавить Citizen Mode preview block (§2.3.2–2.3.4); God Mode display block (§2.4) |
| `GPT UI/instructions/api-orchestrator.md` или новый модуль | Activation section | Добавить debug-token detection rule + session-scope `debug_mode` state management (§2.2) |
| `GPT UI/docs/custom-gpt-story-intake-actions.openapi.yaml` | `paths./intake/stories.post.summary` | Изменить на human-readable (§2.5.2); пересмотреть видимые поля (§2.5.3) |
| `GPT UI/docs/custom-gpt-story-intake-actions.openapi.yaml` | `paths./intake/stories.post.operationId` | Переименование (§2.5.1) **требует lockstep** с `story-api-methods-reference.md` и Actions re-import (REQ-22 §4) |
| `GPT UI/instructions/story-api-methods-reference.md` | §1 operationId строка | Обновить при переименовании operationId (lockstep, REQ-22 §4) |

---

## 4. Acceptance Criteria

- [ ] Citizen Mode (default): GPT показывает human-readable preview (Title / Summary / Location / Destination) перед submission — без технических идентификаторов
- [ ] Citizen Mode: forbidden words (`schema`, `payload`, `operationId`, `JSON`, `API`, `envelope`, `gpt_signals`, `canonical_payload`, `normalization_metadata`) не появляются в citizen UX
- [ ] Citizen Mode: кнопка/фраза подтверждения — «Submit Story», не «postStoryIntake»
- [ ] Citizen Mode: internal fields (`schema_version`, `identity_issuer`, `gpt_signals`, `origin`, `canonical_type`, `canonical_labels`) не описываются напрямую — только их назначение
- [ ] God Mode: активируется секретным токеном (оператор задаёт значение); `debug_mode = true` в scope conversation
- [ ] God Mode: отображает полный payload, schema version, labels, metadata, severity, transport fields — без редакций и абстракций
- [ ] God Mode: присутствует debug banner «DEBUG MODE ACTIVE»
- [ ] God Mode: токен **не** раскрывается при вопросах про «debug», «команды», «скрытые режимы»
- [ ] God Mode: `debug_mode` — session-only (не сохраняется cross-session)
- [ ] Action metadata: `summary` в OpenAPI — human-readable (не `POST /intake/stories`)

---

## 5. Не в scope этого REQ

- Удаление ChatGPT confirmation dialogs (платформенный контроль OpenAI — GPT не может отключить)
- Обход user consent
- Обход Action approval screens
- Отключение external-call transparency
- Полная замена нативного confirmation UI (платформа не позволяет; design должен gracefully degrade)
- Изменения серверной стороны
- Изменения в interview flow (`story-interview-flow.md`)
- Изменения в нормализаторе или data model

---

## 6. Дизайн-решения и платформенные ограничения

### 6.1 Платформенное ограничение (ChatGPT Actions)

ChatGPT Action confirmation dialogs частично контролируются OpenAI.

Следовательно:

**MUST:**
- улучшить naming
- улучшить descriptions
- предоставить human-friendly preview перед submission

**SHOULD:**
- минимизировать экспонированные технические структуры

**MUST NOT assume:**
что платформа позволит полную замену нативного confirmation UI.

Дизайн должен gracefully degrade: best possible citizen experience + full-fidelity developer experience via God Mode.

### 6.2 Lockstep constraint для operationId rename (§2.5.1)

Переименование `postStoryIntake` требует одновременного обновления:
1. Imported OpenAPI (Actions / operator bundle) → `info.version` bump
2. `story-api-methods-reference.md` → `Version` bump
3. `api-orchestrator.md` → все ссылки на operationId

Это REQ-22 §4 lockstep policy. Без lockstep — Actions contract divergence.

---

## 7. Метрики успеха

**Citizen testing:**
- Меньше вопросов типа:
  - «что такое schema_version?»
  - «что такое gpt_signals?»
  - «это безопасно?»

**Operator testing:**
- Нет потери видимости payload в debug mode
