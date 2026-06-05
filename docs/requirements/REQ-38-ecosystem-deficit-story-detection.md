# REQ-38: Ecosystem-deficit story detection — классификация историй об отсутствии среды

> **Назначение:** научить GPT распознавать и корректно классифицировать истории, где проблема — **отсутствие среды / дефицит экосистемы / институциональный упадок / слабые сообщества**, а не один сломанный сервис. Сейчас такие истории сжимаются в один topic-domain label (`education`), теряя civic-смысл «город как среда развития». REQ-38 — **поведенческий слой** поверх vocabulary из [REQ-36](./REQ-36-civic-taxonomy-expansion-multi-axis.md).
> **Источник:** Gap Analysis 2026-06-04 (черновик REQ-36 §5 «Story Ecosystem Detection» + §2.2 multi-dimensional goal); тест-история культурно-научный центр.
> **Технический контекст:** ось `ecosystem_signal` (`ecosystem_gap`, `institutional_decline`, …) определяется в REQ-36 в [`story-label-taxonomy.md`](../../instructions/story-label-taxonomy.md); этот REQ добавляет правила её **применения** в [`story-interview-flow.md`](../../instructions/story-interview-flow.md) и [`story-normalizer.md`](../../instructions/story-normalizer.md). REQ-33 уже требует multi-axis extraction, REQ-34 разводит observation/complaint — оба смежны.

**Версия:** 1.0 · 2026-06-04
**Статус:** Done — instruction P3 (pkg-000018 · GIM-163…165) + audit follow-up P6 (GIM-166 · normalizer v0.2.9); manual replay Advisory pending
**Приоритет:** P3 (classification quality для ecosystem-историй; зависит от foundational REQ-36)
**Тип:** GPT instruction update — `story-interview-flow.md` (detection в интервью), `story-normalizer.md` §4.1/§4.1a (classification preference)
**Серверная сторона:** не требует изменений
**Парные REQ:** [REQ-36](./REQ-36-civic-taxonomy-expansion-multi-axis.md) (**dependency** — ось/vocabulary), [REQ-33](./REQ-33-label-extraction-multi-axis-improvement.md) (multi-axis), [REQ-34](./REQ-34-summary-generation-and-canonical-type-clarity.md) (observation/complaint boundary)

---

## 1. Текущее состояние (verified по коду 2026-06-04)

### 1.1 Нет понятия «история про отсутствие среды»

[`story-normalizer.md`](../../instructions/story-normalizer.md) §4.1a (REQ-33) даёт multi-axis hints, но все ориентированы на наличие конкретного объекта/сбоя (`public_space`, broken-road regression guard L160). **Нет** правила: «если история про отсутствие среды/экосистемы — классифицируй по `ecosystem_signal`, не сворачивай в один topic-domain».

### 1.2 REQ-34 boundary observation/complaint частично смежен, но не покрывает ecosystem

[`story-normalizer.md`](../../instructions/story-normalizer.md) §4.1 (REQ-34, L149–159): `complaint` когда «absent or malfunctioning condition». Ecosystem-deficit история (отсутствие культурно-научной среды) попадает под «absence → complaint», но **тип** не равен **классификации по осям** — даже верный `complaint` сейчас получит `["education"]` без `ecosystem_signal`/`governance_signal`, т.к. этих осей нет (REQ-36) и нет detection-правила (этот REQ).

### 1.3 Зависимость от REQ-36

Ось `ecosystem_signal` и её ключи (`ecosystem_gap`, `institutional_decline`, `mentor_shortage`, `loss_of_continuity`, `replicable_model_needed`) добавлены REQ-36 в [`story-label-taxonomy.md`](../../instructions/story-label-taxonomy.md) v0.2.1 §4.9. **REQ-38 разблокирован** — P3 добавляет detection/classification rules (не vocabulary).

---

## 2. Целевое состояние

### 2.1 Detection в интервью (`story-interview-flow.md`)

Добавить признаки ecosystem-истории, распознаваемые из уже сказанного резидентом (без наводящих вопросов, как REQ-35 §4.3 privacy-baseline):

- отсутствие среды/пространства как проблема («негде», «нет места для», «раньше было, теперь нет»);
- упадок институтов / потеря преемственности / нехватка наставников;
- фрагментация сообщества / недоиспользованные ресурсы;
- желание воспроизводимой модели («чтобы и в других районах»).

### 2.2 Classification preference (`story-normalizer.md` §4.1/§4.1a)

Правило: когда нарратив описывает **отсутствие среды**, а не единичный сбой — нормализатор preferentially применяет `ecosystem_signal` метки (`ecosystem_gap`, `institutional_decline`, при evidence `city_for_people`) **в дополнение** к topic-domain, и НЕ сворачивает историю в один domain-label.

### 2.3 Анти-коллапс гарантия

Ни одна ecosystem-история не сводится к single-domain label, когда evidence поддерживает multi-axis классификацию (усиление REQ-33 для ecosystem-класса).

### 2.4 Согласование с REQ-34

Ecosystem-deficit с implied harm/absence → `type = complaint` (REQ-34); чистое улучшение-без-вреда → `observation`. Detection классификации (оси) ортогонально выбору `type` — оба применяются.

---

## 3. Файлы для изменения

| Файл | Секция | Что изменяется |
|------|--------|----------------|
| `GPT UI/instructions/story-interview-flow.md` | §8 anti-patterns / интервью-фазы | Признаки ecosystem-истории (из сказанного, без наводящих) |
| `GPT UI/instructions/story-normalizer.md` | §4.1a extraction hints | Hint-блок `ecosystem_signal`: absence-of-environment → preferential labels; анти-коллапс |
| `GPT UI/instructions/story-normalizer.md` | §4.1 type | Cross-link ecosystem ↔ REQ-34 observation/complaint |
| `GPT UI/instructions/story-normalizer.md` | Version + history | Bump + changelog REQ-38 |

---

## 4. Acceptance Criteria

**Static (verifiable по инструкциям, после REQ-36):**
- [ ] `story-normalizer.md` §4.1a содержит hint-блок `ecosystem_signal` с триггером «absence of environment / missing ecosystem»
- [ ] Правило preferential `ecosystem_signal` + анти-коллапс (не single-domain) присутствует
- [ ] `story-interview-flow.md` содержит признаки ecosystem-истории без наводящих вопросов
- [ ] Cross-link на REQ-34 observation/complaint (type ортогонален классификации)
- [ ] Регрессия: REQ-33 broken-road guard (`city_for_people` не для infra-failure) не ослаблен

**Advisory (live replay):**
- [ ] История «нет культурной среды для детей» → `ecosystem_signal` + `culture`/`youth_development`, не только `education` (AC-36.1/36.4 источника)
- [ ] История про научные школы → multi-axis с `mentor_shortage`/`academic_mentorship` когда evidence

---

## 5. Не в scope

- **Определение оси `ecosystem_signal` и vocabulary** → [REQ-36](./REQ-36-civic-taxonomy-expansion-multi-axis.md) §2.7 (это REQ только применяет).
- Geographic intelligence → [REQ-39](./REQ-39-geographic-intelligence-confidence-canonicalization.md).
- Изменения backend / wire.
- Общая multi-axis extraction (уже REQ-33) — здесь только ecosystem-специфика.
