# REQ-42: Adaptive post-submission confirmation message — стиле-адаптивное сообщение после `POST /intake/stories`

> **Назначение:** после успешного ответа backend сделать post-submit сообщение не плоским техническим подтверждением, а **понятным объяснением** того, что произошло с историей, адаптированным под манеру мышления и эмоциональный фокус пользователя, без claims сверх backend-ответа.
> **Источник:** черновик `FR-M2-POSTSUBMIT-ADAPTIVE-MESSAGE` (2026-06; raw REQ-42); продуктовое требование к UX post-submit.
> **Технический контекст (verified по коду):** post-submit сейчас живёт в [`api-orchestrator.md`](../../instructions/api-orchestrator.md) §5.2.3 — плоский вывод `story_id` + `status` (`ready_for_profile` / `partial_ready`), без адаптации. `comm_context` ([`bootstrap.md`](../../instructions/bootstrap.md) L66–70) содержит **4 поля** (`ui_lang`, `tone_preset`, `verbosity_level`, `transparency_mode`) — поля «тип мышления» нет. Citizen Mode forbidden-lexicon ([`api-orchestrator.md`](../../instructions/api-orchestrator.md) §5.2.0b L435) запрещает `JSON`/`payload`/`gpt_signals` в citizen-facing copy. Запрет ложных claims о статусе — [`root.md`](../../instructions/root.md) (backend authority).

**Версия:** 1.0 · 2026-06-07
**Статус:** P3 Done (Awaiting Commits) · [`pkg-000022`](../docs/analysis/tasks/gpt-active-packages/pkg-000022-20260607-req42-adaptive-post-submit-message.yaml) (GIM-178…180 🟢) · [run-summary](../docs/analysis/tasks/run-reports/run-summary-20260608-req42-p3-execute.md)
**Приоритет:** P3 (UX-полировка post-submit; не блокирует submit) — demo + post-demo (общее правило)
**Тип:** GPT instruction update — `bootstrap.md` (новое поле `comm_context.cognitive_style`); `api-orchestrator.md` §5.2.4 (adaptive post-submit message)
**Серверная сторона:** не требует изменений (адаптация на instruction-слое; backend-ответ потребляется как есть)
**Парные REQ:** [REQ-31](./REQ-31-God-mode-activation.md) (Citizen/God Mode + forbidden-lexicon — дом и границы), [REQ-30](./REQ-30-admission-gate-story-intake-strict-chain.md) / [REQ-40](./REQ-40-pre-submission-instruction-compliance-validation.md) (pre-submit gates — предшествуют) · bootstrap `comm_context` baseline

### Namespace note (GAP-42-01 / GIM-181)

Идентификатор **«REQ-42»** в workspace используется **дважды**:

| Scope | Документ | Содержание |
|-------|----------|------------|
| **Gateway** | [`doge-complaints-gateway/docs/requirements/42-gpt-signals-story-intake-extension.md`](../../doge-complaints-gateway/docs/requirements/42-gpt-signals-story-intake-extension.md) | `gpt_signals` enum frozensets (`severity`, `impact_estimation`, `problem_status`) |
| **GPT-UI (этот файл)** | `REQ-42-adaptive-post-submission-confirmation-message.md` | `cognitive_style` + §5.2.4 adaptive post-submit UX |

В instruction-файлах GPT UI: **gateway REQ-42 (gpt_signals)** — wire enums; **GPT-UI REQ-42** — post-submit / `cognitive_style` (GIM-178…179). Не смешивать при traceability-правках.

---

## 1. Текущее состояние (verified по коду 2026-06-07)

### 1.1 Post-submit сообщение — плоское, без адаптации

[`api-orchestrator.md`](../../instructions/api-orchestrator.md) §5.2.3 (L551–562): на HTTP 202 — «Report `story_id` and `status` from response `data` envelope». Различает статусы:

| Статус (202) | Смысл (§5.2.3) |
|---|---|
| `ready_for_profile` | story войдёт в кластеризацию и **может** стать публичным issue |
| `partial_ready` | сохранено, но **не** кластеризовано — нарратив неполон (пустой title/description/языковой слот) |

Нет правил адаптации под стиль пользователя и нет явной структуры «что произошло / что не произошло / что дальше».

### 1.2 `comm_context` не содержит измерения «тип мышления»

[`bootstrap.md`](../../instructions/bootstrap.md) L66–70 — `comm_context` имеет **4 поля**: `ui_lang`, `tone_preset` (напр. `neutral_friendly`), `verbosity_level` (`brief`/`normal`/`detailed`), `transparency_mode` (`comfort`/`debug`). Измерения «системно vs история-ориентированно мыслит» нет — REQ-42 его вводит.

> `transparency_mode` (показывать ли диагностику) и `tone_preset` (эмоциональный тон) — **ортогональны** новому полю (как человек структурирует смысл). Эмоциональная теплота берётся из `tone_preset`; структурное обрамление (системное vs нарративное) — из нового `cognitive_style`.

### 1.3 Citizen forbidden-lexicon действует на citizen-facing copy

[`api-orchestrator.md`](../../instructions/api-orchestrator.md) §5.2.0b L435 (REQ-31): в Citizen Mode запрещены `schema`, `payload`, `JSON`, `API`, `envelope`, `gpt_signals`, `canonical_payload`, `normalization_metadata`, `postStoryIntake`. backend `status` (напр. `ready_for_profile`) при этом жителю показывается.

### 1.4 Backend authority (no false claims) — уже есть

[`root.md`](../../instructions/root.md): запрещено утверждать «принято / одобрено / стало Issue / опубликовано» без явного backend response body. REQ-42 §2.5 — cross-ref на этот инвариант, не новое правило.

---

## 2. Целевое состояние

### 2.1 Новое поле `comm_context.cognitive_style` (`bootstrap.md`)

Добавить 5-е измерение `comm_context`:

| Поле | Значения | Default |
|---|---|---|
| `cognitive_style` | `systemic` \| `narrative` \| `mixed` | `mixed` |

- `systemic` — пользователь мыслит системами/кластерами/статусами/пайплайнами (интерес к границе «частная история ↔ Issue», downstream-обработке).
- `narrative` — пользователь мыслит живой историей и эмоциями.
- `mixed` — явного перекоса нет (default).

**Определение — пассивное (без вопроса):** GPT выводит `cognitive_style` **только** из уже сказанного пользователем в диалоге; **не** задаёт наводящий вопрос и **не** имеет команды `/…` (в отличие от 4 базовых полей). Privacy-baseline согласован с [REQ-35](./REQ-35-demo-field-population-gpt-fixes.md) §4.3 / minimal-questions. При отсутствии сигнала → `mixed`.

### 2.2 Adaptive post-submit message (`api-orchestrator.md` §5.2.4 — новая подсекция)

Новая подсекция §5.2.4 сразу после §5.2.3 (response handling). **Только** после успешного backend response (HTTP 202). Сообщение строится с учётом `comm_context` (`ui_lang`, `tone_preset`, `verbosity_level`, `cognitive_style`) и обязательной структуры:

1. **Что произошло:** история сохранена как отдельный кейс.
2. **Backend-факт:** явно показать `story_id` и backend `status` без переинтерпретации.
3. **Что НЕ произошло:** отделить факт отправки от последующих решений системы — история ≠ Issue.
4. **Что дальше:** история — сигнал / точка данных; Issue может возникнуть **только позже** через кластеризацию / профиль / gate / downstream.
5. **No false claims:** не утверждать «принято / одобрено / стало Issue», если backend этого не вернул (§2.5).

### 2.3 Покрытие — оба 202-статуса

Адаптировать **оба** успешных исхода §5.2.3:
- `ready_for_profile` → «сохранено, пойдёт в кластеризацию, может позже стать Issue».
- `partial_ready` → «сохранено, но нарратив неполон — нужна доработка до публикации» (мягко, без обвинения).

Ошибочные исходы (400/401/422/5xx) — **вне scope** REQ-42; остаются как есть в §5.2.3 (§5).

### 2.4 Стиле-варианты в рамках Citizen forbidden-lexicon

Адаптация меняет **тон и структуру**, но **не нарушает** Citizen forbidden-lexicon (§1.3):

- **`cognitive_style = systemic` (Citizen Mode):** прямой, концептуальный стиль с явным «что произошло / что не произошло / что дальше»; backend `status` показываем; кластер / Issue / downstream объясняем **человеческими словами** (без `payload`/`JSON`/`gpt_signals`).
- **`cognitive_style = narrative` / эмоциональный `tone_preset`:** более поддерживающая формулировка, но **без ложных обещаний**.
- **`mixed`:** нейтрально-ясное сообщение со структурой «что произошло / что не произошло / что дальше».
- **God Mode (`debug_mode = true`):** полный системный вывод допустим без ограничений лексики (forbidden-lexicon — только Citizen).

### 2.5 Backend-authority alignment

Сообщение **не** делает claims сверх backend-ответа ([`root.md`](../../instructions/root.md)). `status` репортится дословно; «может стать Issue» формулируется как возможность, не факт.

### 2.6 Примеры

**Citizen Mode · `cognitive_style = systemic` · `status = ready_for_profile`:**

> История сохранена как отдельный кейс (`story_id: <id>`).
> Статус: `ready_for_profile`.
> Это **не** значит, что Issue уже создан или что систему признали проблему системной. Сейчас это одна точка данных. Дальше она может быть сопоставлена с похожими историями — если вокруг неё сложится устойчивый паттерн, из этого уже может родиться Issue.

**Citizen Mode · `cognitive_style = narrative` · `status = partial_ready`:**

> Спасибо — твоя история сохранена (`story_id: <id>`).
> Статус: `partial_ready`. Это значит, что она записана, но пока неполная: чтобы она могла пойти дальше, не хватает части деталей. Это нормально — её можно дополнить позже.

---

## 3. Файлы для изменения

| Файл | Секция | Что изменяется |
|------|--------|----------------|
| `GPT UI/instructions/bootstrap.md` | `comm_context` default block + новый Step | Добавить поле `cognitive_style` (`systemic`/`narrative`/`mixed`, default `mixed`, пассивное определение, без команды/вопроса) |
| `GPT UI/instructions/api-orchestrator.md` | §5.2.4 (новая подсекция) | Adaptive post-submit message: структура «что произошло / что не произошло / что дальше»; оба 202-статуса; Citizen-lexicon разграничение; no-false-claims |
| `GPT UI/instructions/api-orchestrator.md` | Version + history | Bump + changelog REQ-42 |
| `GPT UI/instructions/bootstrap.md` | Version + history (если ведётся) | Bump + changelog REQ-42 |

---

## 4. Acceptance Criteria

**Static (verifiable по инструкциям):**
- [ ] `bootstrap.md` `comm_context` содержит поле `cognitive_style` с values `systemic`/`narrative`/`mixed`, default `mixed`, явным правилом пассивного определения (без наводящего вопроса/команды)
- [ ] `api-orchestrator.md` §5.2.4 существует, активируется **только** после HTTP 202
- [ ] §5.2.4 обязывает явно показывать `story_id` и backend `status`
- [ ] §5.2.4 содержит структуру «что произошло / что НЕ произошло / что дальше» и no-false-claims (cross-ref root.md)
- [ ] §5.2.4 покрывает оба статуса: `ready_for_profile` и `partial_ready`
- [ ] §5.2.4 явно сохраняет Citizen forbidden-lexicon (системное объяснение — человеческими словами; God Mode без ограничений)
- [ ] Регрессия: §5.2.3 коды ответов (400/401/422/5xx) и Citizen forbidden-lexicon (§5.2.0b L435) не изменены

**Advisory (live replay, из кода не верифицируются):**
- [ ] Системно-мыслящий пользователь → сообщение с явным «история = точка данных, Issue только позже»
- [ ] partial_ready → корректное «сохранено, но неполно» без ложного «принято»
- [ ] Эмоциональный пользователь → поддерживающий тон без ложных обещаний

---

## 5. Не в scope

- **Адаптация error-сообщений** (400/401/422/5xx) — остаются в §5.2.3 как есть.
- **Изменение Citizen forbidden-lexicon** (REQ-31) — REQ-42 работает в его рамках, не меняет.
- **Команда `/…` или явный вопрос для `cognitive_style`** — поле пассивное (§2.1).
- **Backend / wire-изменения** — нет; backend-ответ потребляется как есть.
- **Downstream-механика** (реальная кластеризация / profile / gate) — серверная логика, вне GPT-слоя.

---

## 6. Принятые решения (интервью 2026-06-07)

| # | Развилка | Решение |
|---|---|---|
| D1 | Источник сигнала «тип мышления» | Новое поле `comm_context.cognitive_style` (не reuse `transparency_mode`) |
| D2 | Имя/значения поля | `cognitive_style: systemic \| narrative \| mixed` |
| D3 | Как определяется | Пассивно, из реплик; без команды и без наводящего вопроса |
| D4 | Default | `mixed` |
| D5 | Дом требования | Новая подсекция `api-orchestrator.md` §5.2.4 (рядом с §5.2.3) |
| D6 | Citizen-лексика | Адаптация в рамках forbidden-lexicon; полный системный вывод — только God Mode |
| D7 | Покрытие статусов | Оба 202: `ready_for_profile` + `partial_ready`; ошибки вне scope |
| D8 | Приоритет / scope | P3; demo + post-demo (общее правило) |
