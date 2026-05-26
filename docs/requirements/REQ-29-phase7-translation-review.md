# REQ-29: Phase 7 — translation quality gate для нелингвальных слотов

> **Назначение:** добавить в Phase 7 (confirmation step) явный индикатор пользователю о том, что перевод `title` / `description` на два языка кроме `session_language` выполнен GPT и не был подтверждён. В текущем состоянии Phase 7 показывает пользователю только version на `session_language`; переводы на другие языки попадают в публичную запись без просмотра.  
> **Источник:** `GPT UI/docs/audit-field-provenance-2026-05-24.md` (FINDING-04)

**Версия:** 1.0 · 2026-05-25  
**Статус:** requirements — ready for tasking  
**Приоритет:** P3 (civic quality / transparency; нет runtime-breakage)  
**Тип:** GPT instruction update — `story-interview-flow.md` §7.2 Phase 7; поведение не блокирующее  
**Серверная сторона:** не требует изменений  
**Парные REQ:** [REQ-08](./REQ-08-dialogue-flow.md) (dialogue flow), [REQ-22](./REQ-22-story-intake-wire-contract-v2-alignment.md) (i18n wire contract)

---

## 1. Текущее состояние (проблема)

### 1.1 Phase 7 confirmation scope

`story-interview-flow.md §7.2` Phase 7 выполняет 5 шагов:
1. Concise recap на `session_language`
2. Invite correction (факты, место, смысл, желаемый результат)
3. Framing update при correction
4. Completion rule
5. Title generation на `session_language`

**Что не делает Phase 7:**
- Не показывает `title`, `description` на двух других языках
- Не сигнализирует пользователю, что перевод выполнен GPT
- Не даёт возможности подтвердить / скорректировать переводы

### 1.2 Что попадает в публичную запись без подтверждения

Поля `narrative.title.{et,ru,en}`, `narrative.description.{et,ru,en}`, `narrative.summary.{et,ru,en}` — для двух языков кроме `session_language` являются GPT_TRANSLATED, не подтверждёнными пользователем.

В `DOGEIssue.payload_json` (после clustering/promotion) эти переводы отображаются публично на сайте. Ошибка перевода в гражданской жалобе может искажать смысл сообщения.

### 1.3 Серьёзность

LOW — GPT-переводы между эстонским, русским, английским на структурированных коротких текстах относительно надёжны. Однако в civic context точность атрибуции важна: published record должна честно отражать, что пользователь заявил, а не артефакт GPT.

---

## 2. Целевое состояние

### 2.1 Минимальное изменение: transparency disclosure в Phase 7

В конце Phase 7 (после step 5 — title generation) добавить disclosure-шаг:

```markdown
**Phase 7, Step 6 — Translation disclosure:**  
After generating the session_language title, GPT adds a brief transparency note:

> "Your issue has been prepared in [session_language]. The system will also generate 
> Estonian, Russian, and English versions automatically — these translations are 
> AI-generated and have not been individually reviewed."

This step is **not interactive** — it is informational only. GPT does not ask the user 
to review translations. The session proceeds to the handoff step.
```

### 2.2 Опциональное расширение: spot-check для session_language

Если пользователь изначально написал на языке, отличном от `session_language` (т.е. `narrative.language ≠ narrative.session_language`), добавить в Phase 7 дополнительный шаг:

```markdown
**Phase 7, Step 6b — Cross-language accuracy note (conditional):**  
Condition: `detected_input_language ≠ session_language`

GPT adds:
> "You wrote in [input_language]; I've prepared the issue in [session_language]. 
> If the meaning was changed in the process, please let me know before we submit."

This invites one correction round if user detects a translation error between their 
original language and session_language. It does NOT display the full translated text 
proactively — only triggers on explicit language switch.
```

### 2.3 `story-interview-flow.md §7.2` — конкретные изменения

В Phase 7 mandatory sequence добавить step 6 после step 5 (title generation):

**Добавить:**
```markdown
**Step 6 — Translation transparency note (mandatory):**
After the title is generated, inform the user briefly:
- That the issue will be published in 3 languages (et, ru, en)
- That translations are AI-generated
- That only the [session_language] version was reviewed together

Format: 1 short sentence in session_language. Non-interactive (no response needed).
Example (et): "Kaebus salvestatakse eesti, vene ja inglise keeles; tõlked on 
automaatsed."
Example (ru): "Жалоба будет сохранена на эстонском, русском и английском; переводы 
выполнены автоматически."
Example (en): "The issue will be saved in Estonian, Russian, and English; translations 
are AI-generated."
```

---

## 3. Файлы для изменения

| Файл | Секция | Что изменяется |
|------|--------|----------------|
| `GPT UI/instructions/story-interview-flow.md` | §7.2 Phase 7 mandatory sequence | Добавить Step 6 (translation disclosure) |
| `GPT UI/instructions/story-interview-flow.md` | Version header + history | Bump версии + changelog REQ-29 |

---

## 4. Acceptance Criteria

- [ ] `story-interview-flow.md §7.2` Phase 7 содержит Step 6 — translation disclosure
- [ ] Step 6 описан как non-interactive (information only, не ждёт ответа)
- [ ] Step 6 явно указывает, что только `session_language` version была совместно проверена
- [ ] Step 6 содержит примеры для всех трёх session_language (et, ru, en)
- [ ] Версия `story-interview-flow.md` обновлена с changelog-записью REQ-29
- [ ] Phase 7 Steps 1–5 не изменяются (нет регрессий в confirmation flow)

---

## 5. Не в scope этого REQ

- Полноценный review UI для переводов (out of scope для GPT chat interface)
- Изменения в серверной стороне
- Изменения в normalizer или orchestrator (translation quality — GPT behavior, не pipeline)
- Изменения в `story-data-model.md`
- Verbatim capture как отдельное поле (product backlog R-03 из аудита)

---

## 6. Связанные решения

**Почему не полный spot-check переводов?**  
Показ всех 3 языков версий в Phase 7 делает confirmation flow очень длинным для пользователя (особенно мобильного). Disclosure-note — минимально инвазивное решение: пользователь предупреждён, не перегружен.

**Почему disclosure, а не блокирующий вопрос?**  
Большинство пользователей не владеют всеми тремя языками; спрашивать их о точности перевода на неизвестном языке бесполезно. Disclosure честно сообщает факт без false choice.
