# REQ-34: Summary generation в нормализаторе + canonical_type граница observation/complaint

> **Назначение:** два связанных gap в качестве нормализации. (1) GPT не генерирует `narrative.summary` — в результате `narrative_summary_json = null` в БД, хотя нарратив достаточно богат. (2) `canonical_type = "observation"` применяется слишком широко — история с признаками civic критики попадает в observation, хотя должна быть `complaint`. Оба gap происходят на этапе нормализации.  
> **Источник:** `GPT UI/docs/analysis/1.06.26-testing-gap-report.md` §1 (`narrative_summary_json = null`), §2 (summary не генерируется), §4 (`canonical_type = observation` — требует обсуждения)  
> **Технический контекст:** `story-normalizer.md L122`: "Omit `summary` if no validated short text exists" — нет обязательного шага генерации. `story-interview-flow.md §8 row 9` (anti-pattern): "Misusing observation — Relabel clear harm narratives as observation to skip depth. Use observation routing only for genuine improvement-without-harm (FR-M1-025)." — граница размыта.

**Версия:** 1.0 · 2026-06-01  
**Статус:** Done — instruction P3 (pkg-000015 · GIM-149…151 🟢 2026-06-02); manual replay Advisory pending  
**Приоритет:** P2 (качество данных — summary важна для SPA preview и кластеризации; неверный type влияет на issue promotion gate)  
**Тип:** GPT instruction update — `story-normalizer.md` §4.1 + §2.1; `story-interview-flow.md` §8  
**Серверная сторона:** не требует изменений  
**Парные REQ:** [REQ-25](./REQ-25-canonical-type-labels-and-summary-wire-activation.md) (summary wire), [REQ-33](./REQ-33-label-extraction-multi-axis-improvement.md) (label+type quality), [REQ-10](./REQ-10-output-content-model.md) (output model)

---

## 1. Текущее состояние (проблема)

### 1.1 summary не генерируется — `narrative_summary_json = null`

**`story-normalizer.md` L122:**
```
Omit summary if no validated short text exists.
```

Нет правила: "сгенерируй summary из нарратива если его нет". Нормализатор проецирует уже готовый контент — но **никто** не генерирует summary явно:
- `story-interview-flow.md §7.2` Phase 7 → генерирует только **title** (Step 5)
- `story-normalizer.md §4.1` → ожидает ready-made summary, omit если нет

В результате: у историй с богатым нарративом `narrative_summary_json = null`, хотя 1–2 предложения summary можно извлечь из материала интервью.

**Значимость для продукта:**
- SPA issue cards показывают summary как preview text
- Кластеризация использует summary как сигнал
- Без summary история выглядит неполной в публичном реестре

### 1.2 Граница observation / complaint размыта

**`story-interview-flow.md §8 row 9` (anti-pattern):**
```
Use observation routing only for genuine improvement-without-harm (FR-M1-025).
Keep §4–§5 when user expresses harm.
```

Тестовая история: желание создать культурную/молодёжную инфраструктуру — классифицирована как `observation`. Однако история содержит элементы civic критики: **отсутствие пространства как проблема**, а не просто желание.

Разница имеет прикладное значение: только `complaint` и `system_bug` проходят через issue promotion gate (API_REFERENCE §6.3). История с `observation` НЕ становится публичным DOGEIssue автоматически.

**Текущая проблема:** нет чёткого критерия в инструкции "вот когда observation становится complaint" — только anti-pattern "не злоупотребляй observation". GPT применяет observation по умолчанию при сомнении.

### 1.3 `narrative_consistency_notes = null` — поле не используется

`live_story_context.consistency_notes` omit'ится когда нет противоречий. Но поле мог бы нести value и при нормальном прохождении — "no contradictions detected" как positive confirmation для аудита. Это minor product feature, не functional bug. Фиксируем как known-acceptable.

---

## 2. Целевое состояние

### 2.1 Normalizer: добавить обязательный шаг генерации summary

В `story-normalizer.md §4.1` добавить явное правило:

```markdown
**`summary` generation rule:**

If `canonical_payload.description` and the interview narrative contain sufficient content
(more than a factual incident — i.e., there is meaning, desired state, or civic angle
beyond a bare description), the normalizer MUST generate a concise summary:

- 1–2 sentences in `session_language` capturing: what the problem is + what the desired
  state is (or civic angle).
- Translate to the other two languages per `story-i18n-policy.md`.
- Include as `canonical_payload.summary = { et, ru, en }`.

Omit only when: the story is so minimal that a meaningful summary would merely repeat
the title. Do NOT omit because summary was not produced upstream — generate it here.
```

### 2.2 Interview flow: добавить summary generation в Phase 7

В `story-interview-flow.md §7.2` Phase 7 mandatory sequence после Step 5 (title generation) и до Step 6 (translation note) добавить:

```markdown
**Step 5b — Summary draft (mandatory when content sufficient):**
After generating the session_language title, generate a 1–2 sentence summary in
session_language capturing: (a) what the concrete problem or wish is, (b) what the
desired state is. Pass this to the normalizer as `summary_draft[session_language]`.
The normalizer will translate to the other two languages.
```

*(Или передавать как часть нормализаторного handoff — технически проще.)*

### 2.3 Canonical type: добавить operational boundary в normalizer

В `story-normalizer.md §4.1` в секцию `type` добавить граничный критерий:

```markdown
**observation vs complaint decision rule:**

Use `complaint` when the story describes an **absent or malfunctioning condition** that
the resident experiences as a problem — even if framed gently or as a wish. Key signal:
the resident implies that something SHOULD exist or work, but DOESN'T.

Use `observation` only when: the story is a positive description of desired improvement
WITHOUT implied absence or harm — i.e., the resident explicitly frames it as a preference
or idea, not a problem.

Examples:
- "There's no cultural space for children in our district" → `complaint`
  (absence = problem; improvement wish + implied harm)
- "It would be nice to have more parks" → `observation`
  (pure improvement wish, no implied harm or absence)
- "The kindergarten is overcrowded and there's nowhere to go after school" → `complaint`
  (concrete harm + absence)

When in doubt: if the user would answer "yes" to "Does this bother you or cause a
problem?" → use `complaint`, not `observation`.
```

---

## 3. Файлы для изменения

| Файл | Секция | Что изменяется |
|------|--------|----------------|
| `GPT UI/instructions/story-normalizer.md` | §4.1 `canonical_payload.summary` | Добавить правило генерации summary (не только omit) |
| `GPT UI/instructions/story-interview-flow.md` | §7.2 Phase 7 mandatory sequence | Добавить Step 5b — summary draft generation |
| `GPT UI/instructions/story-normalizer.md` | §4.1 `canonical_payload.type` | Добавить operational boundary: observation vs complaint |
| `GPT UI/instructions/story-normalizer.md` | Version header + history | Bump + changelog REQ-34 |
| `GPT UI/instructions/story-interview-flow.md` | Version header + history | Bump + changelog REQ-34 |

---

## 4. Acceptance Criteria

**Summary generation:**
- [x] `story-normalizer.md §4.1` содержит явное правило: генерировать summary если контент достаточный (v0.2.5 L140–147)
- [x] `story-interview-flow.md §7.2` содержит явный шаг или правило генерации summary draft (v0.18 Step 5b L153)
- [ ] После следующего теста с богатым нарративом: `narrative_summary_json != null` (⚠️ Advisory manual)
- [ ] Регрессия: история с минимальным нарративом (только заголовок без деталей) → summary omit корректно (⚠️ Advisory)

**Canonical type:**
- [x] `story-normalizer.md §4.1 type` содержит operational boundary с примерами (v0.2.5 L149–159)
- [ ] История "нет культурного пространства для детей" → `complaint`, не `observation` (⚠️ Advisory manual)
- [ ] История "было бы хорошо иметь больше парков" (без implied harm) → `observation` корректно (⚠️ Advisory)
- [x] `story-interview-flow.md §8` anti-pattern row 9 не противоречит новому правилу (L202 cross-link)

---

## 5. Не в scope этого REQ

- Изменения в summary wire transport (уже есть REQ-25)
- Изменения в backend (summary сохраняется корректно при наличии)
- Изменения в SPA / issue display
- Полное переписывание canonical_type логики — только граничный критерий

---

## 6. Known-acceptable: `narrative_consistency_notes = null`

`live_story_context.consistency_notes` omit'ится при отсутствии противоречий — это корректное поведение по инструкции. Добавление "consistency check passed" как позитивного подтверждения — product backlog feature, не functional gap. Не входит в scope этого REQ.
