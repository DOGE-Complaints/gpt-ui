# REQ-16: Открытые решения, которые повлияют на инструкции

> **SoT:** `GPT UI/docs/DOGEstonia — GPT Requirements.pdf` — §16.  
> **Статус (как в PDF):** рабочий draft.

## TODO: интеграция

- [x] Переносить закрытые решения в `docs/DOGEstonia.md` §2.3 и в соответствующие FR/инструкции; здесь помечать дату закрытия.

---

## Закрыто в репозитории (не дублирует PDF дословно)

**Stakeholder clarifications (2026-04-20):** ответы продукта зафиксированы в [`../analysis/tasks/reference-m1-post-audit-req16-open-decisions-P3.md`](../analysis/tasks/reference-m1-post-audit-req16-open-decisions-P3.md) §1a; сквозная реализация — задачи **GIM-50…GIM-56** (см. [`../analysis/tasks/bullrun-launch-index.md`](../analysis/tasks/bullrun-launch-index.md) §REQ-16 product decisions). Статусы ниже отражают **текущее** согласование документации; строка **Deferred 2026-04-13 (M7-03)** для Q4 сохраняется как исторический baseline до смены статуса.

| Тема | Статус | Артефакт |
|------|--------|----------|
| **Q3** (см. также `docs/dogestonia-gpt-ui-reusable-architecture-and-issue-spa-bridge.md`, таблица решений, строка **Q3**): этап собеседования vs «strict protocol» (батч недостающих полей) для **Issue** | **Закрыто** 2026-04-10 (**GM3-03**, EPIC-M1-03) | ADR (EN): [`../analysis/tasks/epics/EPIC-M1-03-FR-M1-traceability-and-ingest-core/artifacts/REQ-16-Q3-interview-versus-strict-batch-issue.md`](../analysis/tasks/epics/EPIC-M1-03-FR-M1-traceability-and-ingest-core/artifacts/REQ-16-Q3-interview-versus-strict-batch-issue.md); строки FR-M1-007/013/018/022–023 в [`FR-M1-traceability-matrix.md`](../analysis/tasks/epics/EPIC-M1-03-FR-M1-traceability-and-ingest-core/artifacts/FR-M1-traceability-matrix.md); оверлеи `base.md` / `ingest-validation.md` / `story-interview-flow.md` v0.8. |
| **Q1** (idea/improvement отдельный тип vs `observation`) | **Deferred** 2026-04-13 (**M7-03**, EPIC-M1-07); **clarified** 2026-04-20 | Baseline без отдельного enum: `observation` (GM3-05/REQ-15.3). Продукт: максимально явные **человекочитаемые** пояснения — гид [`../analysis/tasks/artifacts/idea-improvement-vs-observation-human-guide.md`](../analysis/tasks/artifacts/idea-improvement-vs-observation-human-guide.md) (**GIM-51**). |
| **Q2** (обязательная генерация 3 языков внутри GPT vs fallback снаружи) | **Closed (MVP baseline)** 2026-04-13 (**M7-03**); **clarified** 2026-04-20 | [`story-i18n-policy.md`](../../instructions/story-i18n-policy.md): primary slot по `ui_lang`; явный маркер языка в `normalization_metadata.session_language` (**GIM-52**). |
| **Q4** (expanded intake-layer: severity/impact/problem_status) | **Deferred** 2026-04-13 (**M7-03**); **Accepted (resident-perceived)** 2026-04-20 (**GIM-50**, **GIM-54**) | Продукт подтвердил сбор **субъективного** intake (восприятие жителя). Реализация в инструкциях и логической модели — **GIM-54**; до merge GIM-54 narrative-first карточка остаётся baseline. |
| **Q5** (ранний institutional mapping в MVP) | **Deferred** 2026-04-13 (**M7-03**); **Scoped (demo)** 2026-04-20 (**GIM-55**) | Текущее демо: **не извлекать** institutional mapping / не заполнять `institution` из диалога; интеграция с ведомствами на ранней стадии. |
| **Q6** (режим collective story intake) | **Deferred** 2026-04-13 (**M7-03**); **Confirmed future (separate path)** 2026-04-20 (**GIM-56**) | Текущий M1 — single-story intake; коллективный режим — отдельный будущий трек, backlog: [`../analysis/tasks/artifacts/collective-story-intake-backlog.md`](../analysis/tasks/artifacts/collective-story-intake-backlog.md). |
| **PDF §16 п.3** (глубина на чувствительных / «личных» темах vs гражданский контекст) | **In instruction design** 2026-04-20 (**GIM-53**) | Не путать с **Q3** таблицы (interview vs strict batch). Правила глубины: `story-interview-flow.md` + `safety-compliance.md`. |

---

## Содержание (перенос из PDF)

### 16. Открытые решения, которые повлияют на инструкции

Эти пункты **не блокируют** старт разработки инструкций, но должны быть явно учтены как будущие **product switches**:

1. Нужен ли отдельный тип для «идеи улучшения», а не только **observation** / **complaint**.
2. Обязаны ли все три языка всегда генерироваться внутри GPT или часть translation/fallback лежит вне него.
3. Насколько глубоко модуль может работать с чувствительными темами.
4. Должен ли GPT собирать расширенный intake-layer (**severity**, **impact_estimation**, **problem_status**) или только narrative core.
5. Насколько рано в MVP нужен явный **institutional mapping**.
6. Нужен ли в будущем отдельный режим **«collective story intake»**, когда человек подаёт уже не только свою, а групповую историю.
