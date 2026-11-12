# GPT UI — architecture (Module-1 instructions / packs / channels)

> **Назначение:** навигационная система as-built — логика Module-1 инструкций, schema packs, инициализации диалога и двух каналов HTTP (Custom GPT Actions vs `doge-ai-bridge` → OpenAI Responses).  
> **Не путать с** [`runtime-docs/`](../runtime-docs/) (операторские as-built manuals), [`requirements/`](../requirements/) (REQ), [`analysis/`](../analysis/) (working notes).

**As-of:** 2026-09-16 · **Метод:** [`.cursor/rules/analysis.mdc`](../../../.cursor/rules/analysis.mdc)

## Правило достоверности

1. Утверждение о **текущем поведении** ссылается на один из источников:
   - [`instructions/`](../../instructions/) (Module-1 SSOT),
   - [`docs/custom-gpt-story-intake-actions.openapi.yaml`](../custom-gpt-story-intake-actions.openapi.yaml),
   - `doge-ai-bridge/src/**` (Python channel),
   - `doge-complaints-gateway/src/**` / `spa-app/src/**` где путь касается stash/submit.
2. REQ без кода / без нормы в instructions → **Planned**, не Current.
3. Где поведения нет — писать **Verified gap** / **Unknown**, не додумывать.
4. Операторские «как настроить GPT» — в [`runtime-docs/`](../runtime-docs/); здесь — **архитектурная карта**.

## Карта документов

| Документ | Вопрос |
|----------|--------|
| [00-two-channels-overview.md](./00-two-channels-overview.md) | Два канала: Actions vs `doge-ai-bridge`; один wire stash |
| [01-module1-instruction-layers.md](./01-module1-instruction-layers.md) | Слои root/bootstrap/base; strict chain; upload vs runtime order |
| [02-schema-packs-and-binding.md](./02-schema-packs-and-binding.md) | Active pair, six-file flat layout, `schema_binding` |
| [03-init-and-runtime-order.md](./03-init-and-runtime-order.md) | Инициализация сессии → интервью → stash → SPA |
| [04-actions-openapi-wire.md](./04-actions-openapi-wire.md) | Actions OpenAPI YAML, auth, `postStoryDraftStash` |
| [05-python-bridge-instruction-delivery.md](./05-python-bridge-instruction-delivery.md) | Manifest → assembly → Responses API (и verified wiring gap) |
| [req-48-role-oriented-instruction-line/00-overview.md](./req-48-role-oriented-instruction-line/00-overview.md) | **Target** REQ-48: archive / roles sibling / export·publish (не as-built Module-1) |
| [req15-tg-interviewer-roles-ux/00-overview.md](./req15-tg-interviewer-roles-ux/00-overview.md) | **Target** ROL-05/06/07: proactivity + TG readability + citizen no role theater (prose only) |

## Смежные SSOT (не дублировать)

| Зона | Path |
|------|------|
| Старший extract Issue-контура | [`technical-architecture.md`](../technical-architecture.md) |
| Переносимые принципы dialog→API | [`custom-gpt-architecture-principles-for-dialog-ingest-api.md`](../custom-gpt-architecture-principles-for-dialog-ingest-api.md) |
| Operator manuals | [`runtime-docs/README.md`](../runtime-docs/README.md) |
| Protocol (Responses channel) | [`REQ-47`](../requirements/REQ-47-responses-api-https-bearer-bridge.md) |
| Bridge runtime REQ | [`doge-ai-bridge` REQ-01](../../../doge-ai-bridge/docs/requirements/REQ-01-doge-ai-bridge-runtime.md) |
| Bridge topology | [`doge-ai-bridge/docs/architecture/runtime/`](../../../doge-ai-bridge/docs/architecture/runtime/00-overview.md) |
| Content packaging (bridge) | [`02-content-packaging.md`](../../../doge-ai-bridge/docs/architecture/runtime/02-content-packaging.md) |
| Portability core vs pack | [`STORY-GPT-SSR-18`](../tasks/backlog-stories/semantic-schema-runtime/STORY-GPT-SSR-18-core-pack-portability-strip-node-specific.md) |

## Одна фраза

Module-1 текст живёт в `GPT UI/instructions/`; wire stash одинаков для обоих каналов; **транспорт tool call** разный (ChatGPT Actions vs application-owned Responses в `doge-ai-bridge`).
