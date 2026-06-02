# REQ-33: Улучшение извлечения меток — multi-axis extraction и civic signal labels

> **Назначение:** в текущей реализации нормализатор систематически недоизвлекает метки: применяет только одну очевидную topic-domain метку и пропускает применимые civic-signal (`city_for_people`) и archetype-support (`improvement_wish`) метки, хотя они явно описаны в таксономии. Тестовая история получила только `["education"]` при наличии в таксономии `public_space`, `city_for_people`, `improvement_wish`, которые должны были сработать.  
> **Источник:** `GPT UI/docs/analysis/1.06.26-testing-gap-report.md` §4–5 — узкая классификация `["education"]` при явной теме публичных пространств и желаемого города  
> **Технический контекст:** `story-normalizer.md §2.1` явно содержит слово "**conservative** typing" — это вероятная корень проблемы. Таксономия (`story-label-taxonomy.md`) определяет 4 оси: topic_domain, failure_mode, civic_signal, issue_archetype_support. Нормализатор не имеет явного правила применять метки с нескольких осей одновременно.

**Версия:** 1.0 · 2026-06-01  
**Статус:** implemented (GPT instructions v0.2.3 · GIM-145…146 Done Awaiting Commits)  
**Приоритет:** P2 (classification quality — истории попадают в неполный кластер; влияет на видимость в SPA и civic analytics)  
**Тип:** GPT instruction update — `story-normalizer.md` §4.1 label extraction guidance  
**Серверная сторона:** не требует изменений  
**Парные REQ:** [REQ-20](./REQ-20-label-taxonomy-and-extraction-axes.md) (taxonomy SSOT), [REQ-25](./REQ-25-canonical-type-labels-and-summary-wire-activation.md) (label wire activation), [REQ-34](./REQ-34-summary-generation-and-canonical-type-clarity.md) (canonical_type boundary + summary generation — смежный gap из того же тест-репорта)

---

## 1. Текущее состояние (проблема)

### 1.1 Что пошло не так в тестовой истории

История про культурную/молодёжную инфраструктуру в Таллине — желание создать лучшее публичное пространство для развития детей и молодёжи.

Результат классификации:
```json
"canonical_labels": ["education"]
```

Что должно было быть (верификация по таксономии):

| Метка | Ось | Определение в таксономии | Применимость к истории |
|---|---|---|---|
| `education` | topic_domain | School, kindergarten, youth education context | ✅ Присвоена |
| `public_space` | topic_domain | Streets, squares, parks, playgrounds, shared outdoor space | ✅ Должна быть — история про публичное пространство для детей |
| `city_for_people` | civic_signal | The desired state is a more human, usable, respectful city environment | ✅ Должна быть — история формулирует желаемый облик города |
| `improvement_wish` | issue_archetype | The story is framed as a desired improvement | ✅ Должна быть — история про желаемое улучшение |

### 1.2 Корневая причина — отсутствие multi-axis guidance

`story-normalizer.md §2.1` (строка ~29):
```
Preserve **conservative** typing: enums and label keys must match Issue SoT
```

Слово "conservative" интерпретируется GPT как: "возьми только самую очевидную метку, не рискуй". 

Нормализатор не содержит:
- Явного правила: "применяй метки с нескольких осей, если они применимы"
- Примера multi-axis extraction
- Пояснения разницы между topic_domain (что) и civic_signal (намерение/паттерн) + archetype (форма истории)

### 1.3 Что означает "conservative" в правильном понимании

`conservative` должно означать: "не придумывай метки, которых нет; не используй метки с низкой уверенностью". Это не означает: "используй только одну метку". Обе трактовки логичны в отсутствие примера.

---

## 2. Целевое состояние

### 2.1 Изменить формулировку §2.1 + добавить multi-axis guidance

В `story-normalizer.md §2.1` изменить описание label extraction:

**Было:**
```
Preserve conservative typing: enums and label keys must match Issue SoT
```

**Стало:**
```
Label extraction: apply ALL taxonomy keys that genuinely apply to the story — one key per
applicable axis (topic_domain, failure_mode, civic_signal, issue_archetype_support) where
evidence exists in the validated narrative. "Conservative" means: do not invent labels or
apply low-confidence keys — it does NOT mean use only one label total.
```

### 2.2 Добавить в §4.1 пример multi-axis extraction

В секцию `canonical_payload.labels` добавить пример с несколькими осями:

```markdown
**Multi-axis label rule:** A single story commonly maps to labels from multiple taxonomy axes:
- **topic_domain** (what civic area): `education`, `public_space`, `transport`, etc.
- **civic_signal** (what pattern): `city_for_people`, `equity_access`, `systemic_pattern`, etc.
- **issue_archetype_support** (story form): `improvement_wish`, `harm_reported`, `positive_observation`, etc.

Extract from each axis independently. A story about wanting better public spaces for children
should produce: `["education", "public_space", "city_for_people", "improvement_wish"]` — not
just `["education"]`.

Do NOT omit civic_signal labels because they seem "less obvious" — these are among the most
valuable for clustering and analytics.
```

### 2.3 Добавить конкретные extraction hints для проблемных меток

В §4.1 или отдельном подразделе указать сигналы для меток, которые системно пропускаются:

| Метка | Сигналы в тексте истории |
|---|---|
| `public_space` | площадь, парк, двор, детская площадка, пешеходная зона, набережная, общественное пространство, улица, сквер; если история о физическом месте общего пользования — применить |
| `city_for_people` | "хотелось бы", "нужен", "должен быть", "пространство для", "удобный", "место для людей", "развитие"; если история описывает желаемый образ города — применить |
| `improvement_wish` | история не фиксирует сломанное, а желает улучшенное; если `type = observation` и контекст позитивного желания — применить |
| `equity_access` | дети, пожилые, люди с ограниченными возможностями, социальные группы, неравный доступ — применить если тема справедливого доступа |

---

## 3. Файлы для изменения

| Файл | Секция | Что изменяется |
|------|--------|----------------|
| `GPT UI/instructions/story-normalizer.md` | §2.1 conservative typing note | Переформулировать "conservative" — добавить multi-axis clarification |
| `GPT UI/instructions/story-normalizer.md` | §4.1 `canonical_payload.labels` | Добавить multi-axis extraction rule + пример |
| `GPT UI/instructions/story-normalizer.md` | §4.1 или новый §4.1a | Добавить extraction hints для public_space / city_for_people / improvement_wish / equity_access |
| `GPT UI/instructions/story-normalizer.md` | Version header + history | Bump версии + changelog REQ-33 |

---

## 4. Acceptance Criteria

- [x] `story-normalizer.md §2.1` — слово "conservative" содержит явное уточнение: multi-axis = норма, conservative = не изобретай, не пропускай существующие (L29–30)
- [x] `story-normalizer.md §4.1` — содержит явный пример multi-axis extraction с минимум 3 метками из разных осей (L139–145)
- [x] `story-normalizer.md §4.1a` — extraction hints для `public_space`, `city_for_people`, `improvement_wish` присутствуют (L149–158)
- [ ] Валидация на тестовой истории (культура/молодёжь/публичное пространство): GPT извлекает минимум `["education", "public_space", "city_for_people"]` — ⚠️ Advisory manual (REQ33-DOC-04)
- [ ] Regression: история про поломанную дорогу НЕ получает `city_for_people` — ⚠️ Advisory manual + instruction guard L160 (REQ33-DOC-05)

---

## 5. Не в scope этого REQ

- Добавление новых меток в таксономию (`culture`, `youth` как отдельные ключи) — отдельный backlog item
- Изменения в orchestrator, OpenAPI, backend
- Изменения в interview flow — проблема на стадии нормализации, не сбора данных
- Изменения в `ingest-validation.md` — label validation там не меняется, меняется только extraction guidance

---

## 6. Дизайн-решения

**Почему "conservative" осталось как инструкция?**  
Слово важно: оно предотвращает галлюцинации и присвоение неприменимых меток. Проблема не в самом слове, а в отсутствии уточнения. Решение — не убирать, а разъяснить.

**Почему extraction hints, а не строгие правила?**  
GPT лучше работает с иллюстративными примерами и контекстными сигналами, чем с жёсткими if/then условиями. Hints позволяют обобщение на новые истории.
