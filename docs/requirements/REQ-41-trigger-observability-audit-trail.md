# REQ-41: Trigger observability — audit trail срабатывания extraction-триггеров

> **Назначение:** сделать каждое крупное extraction-решение **объяснимым**: экспонировать, какие извлечения-триггеры (location / origin / summary / multi-axis labels / gpt_signals) сработали при подготовке `StoryIntakeRequest`. Диагностический слой над теми же триггерами, что валидирует [REQ-40](./REQ-40-pre-submission-instruction-compliance-validation.md) — но REQ-41 не блокирует, а **показывает** (operator transparency), естественно в God Mode.
> **Источник:** Gap Analysis 2026-06-04 (черновик «Trigger Reliability» §2.1 «Trigger Audit Trail»).
> **Технический контекст (verified по коду):** God Mode уже существует — [`api-orchestrator.md`](../../instructions/api-orchestrator.md) §5.2.0b (REQ-31, L416–481) показывает полный draft `StoryIntakeRequest` оператору, но **без** per-trigger activation-таблицы (видно поле в payload, но не «сработал ли триггер и почему»). `trace_notes` ([`story-normalizer.md`](../../instructions/story-normalizer.md) §4.2.1) — существующий internal-канал для диагностических заметок.

**Версия:** 1.0 · 2026-06-05
**Статус:** requirements — depends-on REQ-40 (общее определение набора триггеров)
**Приоритет:** P3 (operator diagnostics / explainability; не блокирует submit)
**Тип:** GPT instruction update — `api-orchestrator.md` §5.2.0b God Mode (диагностический вывод), `story-normalizer.md` §4.2 (trace источника триггеров)
**Серверная сторона:** не требует изменений (диагностика на instruction-слое; не wire)
**Парные REQ:** [REQ-40](./REQ-40-pre-submission-instruction-compliance-validation.md) (**dependency** — набор триггеров и evidence↔payload логика), [REQ-31](./REQ-31-God-mode-activation.md) (God Mode preview — дом для вывода), [REQ-35](./REQ-35-demo-field-population-gpt-fixes.md) (origin.conversation_id availability)

---

## 1. Текущее состояние (verified по коду 2026-06-05)

### 1.1 God Mode показывает payload, но не активацию триггеров

[`api-orchestrator.md`](../../instructions/api-orchestrator.md) §5.2.0b (REQ-31): God Mode выводит полный JSON `StoryIntakeRequest` с mapping-заметками. Оператор видит **результат** (какие поля присутствуют), но не **per-trigger explainability**: сработал ли `location_trigger`, `summary_generation_trigger`, `multi_axis_labels_trigger`, `gpt_signals_trigger`, `origin_trigger` и почему отсутствующее поле отсутствует (нет evidence vs runtime-unavailable vs пропуск).

### 1.2 trace_notes есть, но не структурирован под триггеры

[`story-normalizer.md`](../../instructions/story-normalizer.md) §4.2.1 `trace_notes` — свободные internal-заметки (REQ-28 institution, REQ-35 D3 conversation_id). Нет стандартизированной trigger-activation-структуры.

### 1.3 Зависимость от REQ-40

Набор триггеров и логика «expected когда / missing если» определяются в [REQ-40](./REQ-40-pre-submission-instruction-compliance-validation.md) §2.3. REQ-41 **переиспользует** их для отображения — без REQ-40 нет канонического списка триггеров. **REQ-41 depends-on REQ-40.**

---

## 2. Целевое состояние

### 2.1 Trigger activation table (God Mode)

В §5.2.0b God Mode добавить диагностическую таблицу активации (показывать только в God Mode, не citizen-facing, не wire):

| Trigger | Activated | Reason (если не activated) |
|---|---|---|
| `location_trigger` | true/false | no confirmed location / runtime |
| `origin_trigger` | true/false | — |
| `summary_generation_trigger` | true/false | content minimal |
| `multi_axis_labels_trigger` | true/false | single-domain narrative |
| `gpt_signals_trigger` | true/false | no subjective signal |

Значения берутся из той же evidence↔payload оценки, что REQ-40 §2.3 (single source — не дублировать оценку, переиспользовать результат).

### 2.2 Объяснимость отсутствия (tie-in с REQ-40 §2.4)

Для каждого не-activated триггера различать причину: **no evidence** / **runtime-unavailable** (origin.conversation_id) / **omitted-by-rule** (summary minimal, institution demo-gate). Отражать в Reason-колонке и/или `trace_notes`.

### 2.3 Границы

- Только God Mode (REQ-31): citizen-facing preview не показывает trigger-таблицу (forbidden-lexicon §5.2.0b L435 сохраняется).
- Не wire: trigger-таблица не входит в `StoryIntakeRequest` (internal diagnostics only).
- Не блокирует submit (это REQ-40).

---

## 3. Файлы для изменения

| Файл | Секция | Что изменяется |
|------|--------|----------------|
| `GPT UI/instructions/api-orchestrator.md` | §5.2.0b God Mode | Trigger activation table (диагностика, God Mode only) |
| `GPT UI/instructions/story-normalizer.md` | §4.2 trace_notes | Стандартизированная trigger-activation запись (источник для таблицы) |
| `GPT UI/instructions/api-orchestrator.md` | Version + history | Bump + changelog REQ-41 |

---

## 4. Acceptance Criteria

**Static (verifiable по `api-orchestrator.md` §5.2.0b):**
- [ ] God Mode содержит trigger activation table (5 триггеров: location/origin/summary/multi_axis_labels/gpt_signals)
- [ ] Каждый не-activated триггер имеет Reason (no-evidence / runtime-unavailable / omitted-by-rule)
- [ ] Таблица помечена God-Mode-only и non-wire (не попадает в `StoryIntakeRequest`, не показывается в Citizen Mode)
- [ ] Trigger-значения переиспользуют REQ-40 evidence↔payload оценку (не вторая независимая логика)
- [ ] Регрессия: §5.2.0b Citizen Mode forbidden-lexicon (L435) и dual-mode поведение не изменены

**Advisory (live replay):**
- [ ] God Mode submit → таблица показывает activated-статус по каждому триггеру
- [ ] Отсутствующий summary (minimal content) → Reason = «omitted-by-rule»

---

## 5. Не в scope

- **Enforcement / FAIL-warning** на missing-триггеры → [REQ-40](./REQ-40-pre-submission-instruction-compliance-validation.md) (это REQ только показывает).
- Изменение Citizen Mode preview / forbidden-lexicon (REQ-31).
- Wire-передача trigger-метаданных (internal diagnostics only; активировать только при lockstep-изменении схемы).
- Сами поля-правила (REQ-26/32/33/34/35).
