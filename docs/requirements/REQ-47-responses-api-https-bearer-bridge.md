# REQ-47: Responses API → HTTPS/Bearer Story Intake channel (protocol)

**Child (runtime service постановка):** [doge-ai-bridge REQ-01](../../../doge-ai-bridge/docs/requirements/REQ-01-doge-ai-bridge-runtime.md) — repo `doge-ai-bridge`  
**Parent / related (GPT UI):** [REQ-18](./REQ-18-api-inbound-story-intake-and-gpt-handoff.md) · [REQ-22](./REQ-22-story-intake-wire-contract-v2-alignment.md) · [REQ-32](./REQ-32-origin-traceability-sending-enforcement.md) · [REQ-45](./REQ-45-semantic-schema-intake-envelope-compatibility.md) / [REQ-45a](./REQ-45a-pack-prose-identities-and-flat-layout.md)  
**Soft dependency:** [STORY-GPT-SSR-18](../tasks/backlog-stories/semantic-schema-runtime/STORY-GPT-SSR-18-core-pack-portability-strip-node-specific.md) — clean shared Module-1 before **mass** multi-node Responses deploy (not a hard gate for single-node pilot)  
**Sibling (disambiguation):** gateway [`47-demo-operational-railway-gpt-fixes.md`](../../../doge-complaints-gateway/docs/requirements/47-demo-operational-railway-gpt-fixes.md) — **другой** документ. Этот файл = **GPT UI REQ-47** only.  
**Status:** Draft — protocol / channel REQ (runtime bodies → child **REQ-01** in `doge-ai-bridge`)  
**Версия:** 0.4 · 2026-09-10 (child relocated: was REQ-47.1 → `doge-ai-bridge` REQ-01; service repo = `doge-ai-bridge`)  
**Проект:** GPT UI (protocol framing only)  
**Фокус:** продуктовый канал Responses → тот же Story Intake wire, что Custom GPT Actions; **без** node-custom code; **без** materialization stubs

---

## 1. Purpose

Нужен **параллельный** (не заменяющий Custom GPT Actions) канал интервьюера: OpenAI **Responses API** function calling → HTTPS `POST /story-drafts` с Bearer — тот же stash/handoff, что уже закрыт для Actions ([REQ-18](./REQ-18-api-inbound-story-intake-and-gpt-handoff.md) / GPT-SUBMIT).

Поведенческий SSOT интервью остаётся Module-1 `GPT UI/instructions/` + active schema pack. Транспорт Actions сегодня = ChatGPT Actions + OpenAPI artifact. Этот REQ фиксирует **протокол канала** и владельца runtime; детальная постановка сервиса — в **[doge-ai-bridge REQ-01](../../../doge-ai-bridge/docs/requirements/REQ-01-doge-ai-bridge-runtime.md)**.

## 2. Named runtime: `doge-ai-bridge`

| | |
|--|--|
| **Service / repo** | `doge-ai-bridge` |
| **Role** | Core DOGEstonia **protocol-level** channel adapter (рядом по смыслу с `doge-complaints-gateway` / `doge-identity-service`) |
| **Not** | Кастомный код ноды, Pack Builder, Module-1 instruction bodies, GPL-форк Telegram Chat Completions kit |

**Locked shape (operator decision 2026-09-10):**

- **Новый тонкий сервис** в monorepo (отдельный процесс / Railway service) — репо **`doge-ai-bridge`**.
- **Не** форк [`n3d1117/chatgpt-telegram-bot`](../runtime-docs/DOGEstonia-Telegram-Railway-Deploy-Kit-RU.md) (kit = Chat Completions; GPL-2.0; не владеет DOGEstonia HTTPS/Bearer).
- **Не** встраивать в `doge-complaints-gateway` (stash API остаётся сервером; bridge — клиент канала).
- Default language for implement: **Python** (стык с gateway intake mental model).
- Child REQ: **`doge-ai-bridge/docs/requirements/REQ-01-…`** (relocated from former GPT UI REQ-47.1).

```text
Resident ↔ Telegram (or other thin UI)
        ↔ doge-ai-bridge  (Responses tool loop + confirm + HTTPS)
        ↔ doge-complaints-gateway POST /story-drafts (Bearer)
        ↔ SPA {SPA_BASE}/#/story/submit?draft_id=…
```

Custom GPT Actions channel остаётся валидным: тот же YAML, тот же stash; `origin.source` различает каналы (Actions vs Responses).

## 3. Core architectural statement (protocol)

Канал — **application/runtime layer**, не «модель сама ходит по URL».

1. приложение отдаёт модели function/tool definition;
2. модель эмитит `function_call`;
3. приложение валидирует;
4. приложение делает HTTPS;
5. приложение возвращает `function_call_output`;
6. модель формулирует ответ резиденту.

Следствия:

- модель не открывает произвольные URL и не видит Bearer;
- OpenAI **не** исполняет DOGEstonia endpoint за Responses function tool;
- приложение владеет HTTP, auth, validation, confirmation, retries, audit;
- allowlist операций; запрет «call any URL».

Детализация компонентов, env, CONF/HTTP/tests — **только** [REQ-01](../../../doge-ai-bridge/docs/requirements/REQ-01-doge-ai-bridge-runtime.md).

## 4. Canonical wire contract (preserve)

| Property | Canonical value/source |
|---|---|
| OpenAPI document | `GPT UI/docs/custom-gpt-story-intake-actions.openapi.yaml` |
| Contract version | `info.version = 0.8.0` (as-built) |
| Operation | `postStoryDraftStash` · `POST` · `/story-drafts` |
| Default server (YAML) | `https://dogestonia-tallinn.up.railway.app` (operator override via approved origin) |
| Auth | HTTP Bearer `GptActionsBearer` = host `GPT_ACTIONS_BEARER_SECRET` |
| Success | HTTP **201** + `data.draft_id` + `trace_id` |
| Errors | `400` / `401` / `422` (geo) / `500` |
| Consequential | `x-openai-isConsequential: true` (app policy outside Actions UI) |

- YAML — **единственный** первичный HTTP-контракт; не дублировать вручную как второй SSOT.
- Успешный 201 = draft stashed для browser handoff — **не** «Story создана».
- `structured_payload` в OpenAPI открыт (`additionalProperties: true`) — model-facing strict schema требует **active pack** projection (см. child).
- Pack identity / payload path — **config**, не hardcoded civic id (REQ-45 AC-016).

## 5. Verified state (2026-09-10) — facts

| Claim | Evidence |
|-------|----------|
| OpenAPI 0.8.0 + `postStoryDraftStash` | [`custom-gpt-story-intake-actions.openapi.yaml`](../custom-gpt-story-intake-actions.openapi.yaml) |
| Gateway stash 201 | `doge-complaints-gateway/src/core/api/handlers.py` `handle_story_draft_create` |
| Stash без idempotency header | same handler — каждый POST новый `draft_id` |
| Active pack pilot | `schema-packs.README.md` → `uus_veerenni_civic` / `v3` |
| OpenAPI example `tallinn_civic`/`v1` | ILLUSTRATIVE only |
| `origin.source` | Actions: `openai_gpt_action` (REQ-32); gateway принимает любой non-empty string |
| Bridge runtime product code | **Absent** (repo `doge-ai-bridge` bootstrap only) |
| Telegram kit | Chat Completions wrapper — **not** this channel |
| SPA continuation | `{SPA_BASE}/#/story/submit?draft_id=` — `api-orchestrator.md` §5.2.B |
| Child REQ home | `doge-ai-bridge/docs/requirements/REQ-01-doge-ai-bridge-runtime.md` |

## 6. Scope split

### This parent (REQ-47) owns

- Product need for Responses→Intake channel.
- Naming and placement of `doge-ai-bridge`.
- Non-goals vs Actions / gateway / node packs / GPL kit.
- Pointer to wire OpenAPI and success semantics.
- Soft link SSR-18 for mass multi-node instruction hygiene.

### Child (`doge-ai-bridge` REQ-01) owns

- Functional постановка сервиса: OAS→strict tool, assemble/validate, HTTPS/Bearer, Telegram confirm, tool loop, config, audit, tests, deliverables list (name ≠ create).

### Out of scope (both unless later REQ)

- Замена Custom GPT Actions.
- Browser submit / SPA rewrite.
- Pack content redesign; SSR-18 P3 instruction edits.
- Agents SDK / multi-agent.
- Создание product source tree «чтобы путь существовал» (без явной implement wave).

## 7. Parent acceptance (protocol)

- [ ] Documented parallel channel: Responses runtime client ↔ same OpenAPI stash as Actions.
- [ ] Service/repo named `doge-ai-bridge`; role = protocol adapter, not node custom.
- [ ] Child [REQ-01](../../../doge-ai-bridge/docs/requirements/REQ-01-doge-ai-bridge-runtime.md) holds runtime functional SSOT without duplicating this parent’s protocol framing.
- [ ] No claim that YAML may be forked as handwritten primary contract.
- [ ] No Story-created claim after stash 201.
- [x] Child relocated out of GPT UI into `doge-ai-bridge/docs/requirements/` (2026-09-10).

## 8. References

- Child: [REQ-01-doge-ai-bridge-runtime.md](../../../doge-ai-bridge/docs/requirements/REQ-01-doge-ai-bridge-runtime.md)
- Child index: [doge-ai-bridge/docs/requirements/README.md](../../../doge-ai-bridge/docs/requirements/README.md)
- OpenAPI: `GPT UI/docs/custom-gpt-story-intake-actions.openapi.yaml`
- Instructions: `GPT UI/instructions/`
- Telegram Completions kit (non-channel): [`DOGEstonia-Telegram-Railway-Deploy-Kit-RU.md`](../runtime-docs/DOGEstonia-Telegram-Railway-Deploy-Kit-RU.md)
- OpenAI: [migrate to Responses](https://developers.openai.com/api/docs/guides/migrate-to-responses) · [function calling](https://developers.openai.com/api/docs/guides/function-calling)

---

*End of REQ-47 v0.4*
