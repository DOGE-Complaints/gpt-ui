# REQ-43: Proactive story-intake offer — личная история валидна без доказательства коллективности

> **Назначение:** после завершённой **личной истории** GPT должен **сам** проактивно предложить подготовить её к подаче — wire = **stash** `POST /story-drafts` → redirect `{SPA_BASE}/#/story/submit?draft_id=` (browser submit via SPA; см. [STORY-GPT-SUBMIT-01](../tasks/backlog-stories/story-submit-handoff/STORY-GPT-SUBMIT-01-redirect-handoff.md)) — **не требуя** доказать, что проблема системна или затрагивает других. Коллективность возникает **downstream** через честную кластеризацию похожих историй, а не через давление на пользователя «обобщить» или «говорить за других».
> **Источник:** черновик `FR-M2-STORY-INTAKE-PROACTIVE-OFFER` (2026-06; raw REQ-43); продуктовый принцип «пользователь отвечает только за свой опыт».
> **Технический контекст (verified по коду, 2026-07-06):** [`story-interview-flow.md`](../../instructions/story-interview-flow.md) §5 Q7 — **non-blocking** (GIM-182); §7.5 — **проактивное предложение** stash+redirect при «эпизод+смысл» (GIM-183/194); §8 anti-pattern «не требовать коллективность» (GIM-184). Wire handoff: [`api-orchestrator.md`](../../instructions/api-orchestrator.md) §5.2 stash + §5.2.B redirect ([reconciliation](../docs/analysis/interview-gpt-submit-handoff-reconciliation-2026-07-06.md) D-SUBMIT-1…6). Архитектура story→cluster→Issue — [REQ-42](./REQ-42-adaptive-post-submission-confirmation-message.md). `observation` для improvement-without-harm — FR-M1-025 (§8 row 9).

**Версия:** 1.1 · 2026-07-06 (wire handoff sync — GAP-DOC-02)
**Статус:** P3 Done (Awaiting Commits) · [`pkg-000023`](../docs/analysis/tasks/gpt-active-packages/pkg-000023-20260608-req43-proactive-story-intake-offer.yaml) (GIM-182…184 🟢) · [run-summary](../docs/analysis/tasks/run-reports/run-summary-20260609-req43-p3-execute.md)
**Приоритет:** P2 (conversion / качество входа — снимает барьер подачи личных историй) — demo + post-demo (общее правило)
**Тип:** GPT instruction update — `story-interview-flow.md` §5 (Q7 → non-blocking), §7 (proactive offer rule), §8 (новый anti-pattern)
**Серверная сторона:** не требует изменений (поведение интервью; коллективность — downstream кластеризация)
**Парные REQ:** [REQ-42](./REQ-42-adaptive-post-submission-confirmation-message.md) (story→cluster→Issue downstream — общий принцип), [REQ-34](./REQ-34-summary-generation-and-canonical-type-clarity.md) (observation/complaint — тип решает нормализатор, не интервью), [REQ-30](./REQ-30-admission-gate-story-intake-strict-chain.md) / [REQ-40](./REQ-40-pre-submission-instruction-compliance-validation.md) (pre-submit gates — не должны требовать коллективность) · FR-M1-024…027, FR-M1-032…034

### Namespace note (P1 — GPT-UI vs gateway REQ-43)

Идентификатор **«REQ-43»** в workspace используется **дважды**:

| Scope | Документ | Содержание |
|-------|----------|------------|
| **Gateway** | [`doge-complaints-gateway/docs/requirements/43-institution-json-story-column.md`](../../doge-complaints-gateway/docs/requirements/43-institution-json-story-column.md) | `institution` JSON column / story wire |
| **GPT-UI (этот файл)** | `REQ-43-proactive-story-intake-offer.md` | Proactive intake offer; Q7 non-blocking; §7/§8 narrative-bar |

В instruction-файлах GPT UI: **gateway REQ-43 (institution)** — wire/column; **GPT-UI REQ-43** — interview dialogue / proactive offer (GIM-182…184). См. также [`api-orchestrator.md`](../instructions/api-orchestrator.md) «gateway REQ-43 (institution)» qualifier (§5.2.1 row + §5.2.2 items 7–8).

---

## 1. Текущее состояние (verified по коду 2026-07-06)

> **Doc-sync note:** секция 1.x описывала pre-REQ-43 baseline (2026-06-08). Реализация GIM-182…184 + GPT-SUBMIT-01 propagation (GIM-194) закрыла целевое состояние §2. Ниже — исходный baseline для истории; актуальный wire см. header «Технический контекст».

### 1.1 Q7 «not a one-off» — обязательный гейт полноты (baseline 2026-06-08; fixed GIM-182)

[`story-interview-flow.md`](../../instructions/story-interview-flow.md) §5 (L66–80): «An interview is **content-complete** for handoff to **Phase 7** only if the model can answer **all seven** questions … If any answer is missing or only guessed, treat the interview as **incomplete** … do not imply readiness for backend». Седьмой вопрос ([L78](../../instructions/story-interview-flow.md#L78)):

> **Q7 — Is there evidence this is not a one-off?** (Phase 6)

→ Сейчас отсутствие доказательства коллективности делает интервью «неполным» и **не готовым** к подаче. Это прямо конфликтует с REQ-43.

> Смягчающий контекст (уже есть): §5 допускает «explicitly name the gap» и [L286](../../instructions/story-interview-flow.md#L286): «the narrative may remain **incomplete** for Issue handoff … that is **acceptable**» — но **дефолтная рамка** трактует отсутствие Q7 как неполноту.

### 1.2 Phase 6 «Civic generalization» исследует коллективность

Phase 6 ([L105](../../instructions/story-interview-flow.md#L105)): «Collective signal potential — Clear: one-off vs recurring». §3 (L58): «signal of **collective relevance** (for clusters … not 'flat tickets')». Сам по себе сбор сигнала корректен; проблема — когда он становится **требованием** доказать.

### 1.3 Нет anti-pattern «не требовать коллективность» и нет проактивного предложения (baseline 2026-06-08; fixed GIM-183/184 + §7.5 GIM-194)

§8 anti-patterns ([L211–221](../../instructions/story-interview-flow.md#L211-L221)): row 2 (early classification), row 9 (observation misuse) — смежны, но **нет** правила «не требовать доказать коллективность / не заставлять говорить за других». Проактивного «хочешь, подготовлю к отправке?» в инструкциях **нет** — поток идёт phases → §5 gate → Phase 7 → handoff.

### 1.4 Архитектура и тип — уже поддерживают личную историю

- story→cluster→Issue downstream — [REQ-42](./REQ-42-adaptive-post-submission-confirmation-message.md) §2.2 (история = точка данных; Issue только позже).
- `observation` для improvement-without-harm (FR-M1-025); §8 row 2 уже запрещает преждевременный `type`/`labels`.

---

## 2. Целевое состояние

### 2.1 Q7 → non-blocking сигнал (`story-interview-flow.md` §5)

Переформулировать Q7: отметить «единично / повторяемо» **если пользователь сам это сказал**, но **отсутствие** доказательства коллективности **НЕ** делает интервью неполным и **НЕ** блокирует handoff/подачу. Явно: для **личной истории** Q7 — опциональный сигнал, не гейт. Коллективность выявляется downstream (кластеризация, REQ-42).

### 2.2 Новый anti-pattern в §8 (`story-interview-flow.md`)

Добавить row: **DON'T требовать доказать коллективность / общественную значимость; DON'T просить «это не только у меня»; DON'T подталкивать говорить за других; DON'T блокировать intake потому что системность не доказана.** Do instead: принять частный кейс как валидный вход; коллективность — downstream. Cross-ref FR-M1-025 (row 9), §8 row 7 (false promises) и [REQ-42](./REQ-42-adaptive-post-submission-confirmation-message.md).

### 2.3 Проактивное предложение подать (`story-interview-flow.md` §7)

Правило в переходе Phase 6→7 (или компактный §7.5): когда личная история **«достаточно понятна»** — есть **эпизод** (что/где, Phase 2) и **смысл/важность** (Phase 3) — GPT **сам** предлагает подготовить историю к подаче, **не дожидаясь** явной команды и **не требуя** deep-need/desired/коллективности. Deep-probes (Phase 4–5) и Q7 остаются **опциональными** (можно углубить, если пользователь хочет).

Формулировка (пример, в `session_language`):

> Это можно подать как личную историю. Тебе не нужно доказывать, что это уже системная проблема — ты описываешь свой опыт, а повторяемость и возможная коллективность выявляются дальше, через сопоставление с другими историями.
> Хочешь, я подготовлю эту историю к отправке?

### 2.4 Без преждевременной типизации (alignment, §8 row 2)

Личная история **не** типизируется (`type`/`labels`) на этапе предложения/интервью — тип решает нормализатор ([`story-normalizer.md`](../../instructions/story-normalizer.md) §4.1, REQ-34) на этапе нормализации. Согласовано с §8 row 2.

### 2.5 Границы (не давить, не обещать)

- Предложение — **приглашение**, не давление: пользователь свободен отказаться/продолжить рассказ.
- Не обещать государственный/институциональный результат (§8 row 7 / [`root.md`](../../instructions/root.md)).
- Не превращать intake в преждевременную Issue-классификацию.

---

## 3. Файлы для изменения

| Файл | Секция | Что изменяется |
|------|--------|----------------|
| `GPT UI/instructions/story-interview-flow.md` | §5 Q7 | Q7 → non-blocking сигнал (личная история валидна без коллективности) |
| `GPT UI/instructions/story-interview-flow.md` | §7 (Phase 6→7 / §7.5) | Правило проактивного предложения подать при «эпизод+смысл» |
| `GPT UI/instructions/story-interview-flow.md` | §8 | Новый anti-pattern row: «не требовать коллективность / не говорить за других / не блокировать intake» |
| `GPT UI/instructions/story-interview-flow.md` | Version + history | Bump + changelog REQ-43 |

---

## 4. Acceptance Criteria

**Static (verifiable по `story-interview-flow.md`):**
- [ ] §5 Q7 переформулирован как **non-blocking**: отсутствие доказательства коллективности **не** делает интервью неполным и **не** блокирует handoff (личная история валидна)
- [ ] §7 содержит правило: при «эпизод (Phase 2) + смысл (Phase 3)» GPT **проактивно** предлагает подготовить историю к подаче (без явной команды, без требования Q4–Q7)
- [ ] §8 содержит anti-pattern: DON'T требовать коллективность / просить «не только у меня» / говорить за других / блокировать intake из-за недоказанной системности
- [ ] Предложение оформлено как приглашение, не давление; без false promises (cross-ref §8 row 7 / root.md)
- [ ] Тип/labels **не** присваиваются на этапе предложения (alignment §8 row 2; тип — нормализатор/REQ-34)
- [ ] Регрессия: §5 Q1–Q6 и Phase 7 §7.2 (summary→correction→confirm) не ослаблены; row 9 (observation) и safety §12 не затронуты

**Advisory (live replay, из кода не верифицируются):**
- [ ] Завершённая личная история → GPT сам предлагает подачу, не требуя «докажи, что это у многих»
- [ ] Пользователь, не доказавший коллективность → intake не заблокирован
- [ ] Нет фраз, заставляющих пользователя «говорить за других»

---

## 5. Не в scope

- **Pre-submit enforcement-гейты** (admission gate REQ-30 / compliance REQ-40) — не должны требовать коллективность, но их правка вне scope (REQ-43 меняет **narrative-bar**, не transport-gate).
- **Downstream кластеризация / profile / Issue formation** — серверная логика, вне GPT-слоя.
- **Изменение типизации** (`observation`/`complaint`) — REQ-34, нормализатор; здесь только «не типизировать преждевременно».
- **Post-submit сообщение** — [REQ-42](./REQ-42-adaptive-post-submission-confirmation-message.md).
- **Safety/sensitive-case routing** (§12) — не затрагивается; safety-бар сохраняется.

---

## 6. Принятые решения (интервью 2026-06-08)

| # | Развилка | Решение |
|---|---|---|
| D1 | Конфликт с обязательным Q7 «not a one-off» | Q7 → **non-blocking** сигнал; коллективность downstream, не блокирует подачу |
| D2 | Дом нового поведения | Новый anti-pattern в §8 + правило проактивного предложения в §7 (Phase 6→7 / §7.5) |
| D3 | Триггер проактивного предложения | «Эпизод (Phase 2) + смысл (Phase 3)» — без требования deep-need/desired/Q7 |
| D4 | Типизация личной истории | Не типизировать на этапе предложения (alignment §8 row 2; тип — нормализатор/REQ-34) |
| D5 | Приоритет / scope | P2; demo + post-demo (общее правило) |
