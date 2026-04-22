# REQ-10: Рабочая content-модель выхода GPT

> **SoT:** `GPT UI/docs/DOGEstonia — GPT Requirements.pdf` — §10.  
> **Статус:** рабочий draft, расширен связкой с **REQ-18** и контрактом Story Intake (**§19** gateway).

**Версия:** 0.2 · 2026-04-19

---

## Интеграция с инструкциями и API (актуально)

- Канон логической карточки Issue в инструкциях: `GPT UI/instructions/issue-data-model.md`.
- Строгий артефакт перед HTTP (Module 1 Issues): `normalized_issue_payload` → см. `issue-normalizer.md`, `api-orchestrator.md`.
- **Полный handoff к API приёма историй (Module 2)** — поля, принципы, транспорт: **REQ-18** и авторитетный **`doge-complaints-gateway/.../19-inbound-api-gpt-preprocessing-and-spa-issue-contracts.md`**.
- Логическая content-модель ниже остаётся **не transport schema**; transport для intake — **`StoryIntakeEnvelope`** (REQ-18 / §19).

---

## Содержание (перенос из PDF + проекция на §19)

### 10. Рабочая content-модель выхода GPT

Ниже — **логическая структура содержания**, которую должны поддерживать инструкции. Она проецируется на блоки **`StoryIntakeEnvelope`** так:

| Слой REQ-10 | Где в §19 / REQ-18 |
|-------------|---------------------|
| 10.1–10.4 (narrative, meaning, civic, desired state) | Содержание **`narrative.original_text`** (дословная правда) + интерпретации в **`structured_signals`** + уточнения/факты в **`live_story_context`** |
| 10.5 Issue Projection | Блок **`spa_issue_draft`** (`type`, `labels`, `title`, `summary`, `description`, опционально `institution`, расширения) |
| 10.6 Confidence / Completeness | `live_story_context.key_facts[].confidence`, `spa_issue_draft.confidence`, явная маркировка пробелов; не смешивать с финальными статусами API |

#### 10.1. Narrative Core

- Краткое изложение эпизода.
- Основная проблема или наблюдение.
- Где это происходит.
- Кто затронут.

*Проекция:* база для **`narrative`**, детализация в **`live_story_context`**.

#### 10.2. Meaning Layer

- Что в этом самое неприятное.
- Какая эмоция или переживание доминирует.
- Какая глубинная потребность нарушена.

*Проекция:* **`structured_signals`** (`emotional_meaning`, `deep_need`, …).

#### 10.3. Civic Framing

- Что в этом общественно значимо.
- Признаки повторяемости.
- Это частный случай или устойчивый паттерн.

*Проекция:* **`structured_signals`** + классификация в **`spa_issue_draft.type` / `labels`**.

#### 10.4. Desired State

- Какой среды человек хочет.
- Какой улучшенный сценарий он считает нормальным.

*Проекция:* **`structured_signals.desired_state`** и содержание **`spa_issue_draft.description`**.

#### 10.5. Issue Projection

- Вероятный **type**
- Предварительные **labels**
- Draft **title**
- Draft **summary**
- **Full description**
- При наличии — предполагаемая **institution**

*Проекция:* блок **`spa_issue_draft`**; словарь **`type`** — как **`ISSUE_TYPE`** в SPA (не использовать устаревшие доменные enum вне согласования).

#### 10.6. Confidence / Completeness

- Что подтверждено пользователем
- Что является интерпретацией GPT
- Что осталось неполным

*Проекция:* уровни уверенности в **`live_story_context`**, **`spa_issue_draft.confidence`**, политика маркировки в **`privacy`**; см. REQ-18 §7–8.

---

## Устаревшие допущения (не использовать как единственную модель выхода)

- Считать достаточным **только** плоский набор полей карточки без **immutable `narrative.original_text`** и без разделения интерпретаций — **недостаточно** для контракта Story Intake (**REQ-18**).
- Описывать целевой handoff терминами legacy-домена донора — **не** соответствует продуктовому направлению (**FR-M1-035**, REQ-18 §1).
