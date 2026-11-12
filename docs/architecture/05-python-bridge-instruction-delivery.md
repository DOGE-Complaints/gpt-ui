# 05 — Python bridge: доставка инструкций в OpenAI Responses

**Зона:** [`architecture/`](./README.md)  
**As-of:** 2026-09-16  
**Code:** `doge-ai-bridge/src/aibridge/`  
**Deep packaging:** [aibridge 02-content-packaging.md](../../../doge-ai-bridge/docs/architecture/runtime/02-content-packaging.md) · [REQ-01 AIB-INS](../../../doge-ai-bridge/docs/requirements/REQ-01-doge-ai-bridge-runtime.md)

---

## 1. Идея

Канал B загружает **локальный** instruction bundle (manifest-listed files), регистрирует hashes, вызывает OpenAI **`POST {base}/v1/responses`** с tools; stash на gateway делает bridge, не модель ([REQ-47](../requirements/REQ-47-responses-api-https-bearer-bridge.md)).

Claimed SSOT текстов: `GPT UI/instructions/`.  
Physical copy в репо: `doge-ai-bridge/src/instructions/` (`Dockerfile` `COPY src`).  
**Скрипт sync GPT UI → aibridge в репо aibridge не найден** → **Unknown** (CI pins — см. packaging docs).

---

## 2. Env pins (content)

Из [`config.py`](../../../doge-ai-bridge/src/aibridge/config.py) L91–114:

| Variable | Role |
|----------|------|
| `DOGESTONIA_INSTRUCTIONS_DIR` | Root файлов из manifest |
| `DOGESTONIA_INSTRUCTIONS_MANIFEST` | Path к `instructions.manifest.json` |
| `DOGESTONIA_CONTENT_SOURCE_COMMIT` | Source commit pin |
| `DOGESTONIA_OPENAPI_PATH` | Story Intake wire YAML |
| `DOGESTONIA_PAYLOAD_SCHEMA_PATH` | Active pack `payload.schema.json` |
| `DOGESTONIA_TOOL_SCHEMA_PATH` | Optional recorded strict-tool JSON |
| `DOGESTONIA_SCHEMA_ID` / `DOGESTONIA_SCHEMA_VERSION` | Pack identity |
| `OPENAI_API_KEY` / `OPENAI_MODEL` | Responses client |
| `OPENAI_STORE_RESPONSES` | Must be false (readiness) |

`content_configured()` требует непустые commit, dir, manifest, openapi, payload schema.

---

## 3. Manifest → assembly

Пример локального manifest ([`src/instructions/instructions.manifest.json`](../../../doge-ai-bridge/src/instructions/instructions.manifest.json)):

```json
{
  "bundle_version": "uus-veerenni-v3-local-1",
  "files": [
    "bootstrap.md",
    "root.md",
    "base.md",
    "story-interview-flow.md",
    "safety-compliance.md",
    "schema-packs.uus_veerenni_civic.v3.interview-overlay.md"
  ]
}
```

Это **subset**, не полный Custom GPT Knowledge upload set ([01](./01-module1-instruction-layers.md)).

Loader ([`content.py`](../../../doge-ai-bridge/src/aibridge/content.py)):

1. Parse manifest (`bundle_version`, non-empty unique `files`; root children only).  
2. Read each file UTF-8/LF from `instructions_dir`.  
3. Join with `STABLE_SEPARATOR` (`\n\n`) → `assembled_instructions`.  
4. Compute `instructions_hash`, `wire_oas_hash`, `pack_hash`, `bundle_hash`.  
5. Startup `verify_and_register_deployment` → Postgres registry; `/readyz` fail-closed on hashes.

**No GitHub fetch at request time** (content.py module docstring / AIB-INS-04).

---

## 4. Responses turn

[`InterviewEngine`](../../../doge-ai-bridge/src/aibridge/interview.py):

- Default `instructions: str = ""` (L44).  
- `prompt_channel`: `prefix` | `instructions` (XOR — `assert_prompt_xor`).  
- Stable prefix parts: instructions text + tool JSON + pack id (`request_assembly.build_stable_prefix`).  
- Client: `ProductionResponsesClient` → `{base_url}/responses` when `OPENAI_API_KEY` set ([`app.py`](../../../doge-ai-bridge/src/aibridge/app.py) L286–298).

Tools: allowlisted `postStoryDraftStash` strict schema (generator / recorded path) — не open Actions schema as-is.

---

## 5. Verified gap — assembled instructions ≠ engine

| Step | Fact | Evidence |
|------|------|----------|
| Bundle loads `assembled_instructions` | Yes | `content.py` `LoadedContentBundle` L43, L154 |
| Registry / readiness use hashes | Yes | deployment / readiness |
| `create_app` passes `instructions=` into `InterviewEngine` | **No** | `app.py` L290–298 — only `client` + `model` |
| Engine default instructions | `""` | `interview.py` L44 |

**Current:** content bundle verify/register работает; **поведение «полный Module-1 текст в каждом Responses turn» не следует автоматически из loader** без отдельной wiring, которая подставляет `assembled_instructions` (или files) в `InterviewEngine.instructions` / tools setup.

Документировать как **Verified gap**, не как Planned fantasy и не как «уже работает end-to-end prompt».

---

## 6. Stash после tool call

После accept + confirm bridge исполняет HTTPS Bearer к gateway (тот же wire, [04](./04-actions-openapi-wire.md)) и возвращает `function_call_output`; SPA link из `DOGESTONIA_DRAFT_REDIRECT_BASE_URL` — см. aibridge confirm/HTTP stories и [03](./03-init-and-runtime-order.md) semantics.

---

## 7. Чеклист оператора (передача инструкций)

1. Зафиксировать SSOT в `GPT UI/instructions/` (+ active pack).  
2. Собрать/скопировать файлы в `DOGESTONIA_INSTRUCTIONS_DIR`.  
3. Обновить `instructions.manifest.json` (`files` = только нужные root names).  
4. Pin `DOGESTONIA_CONTENT_SOURCE_COMMIT`, OpenAPI, payload schema, schema id/version.  
5. Redeploy; проверить `/readyz`.  
6. **Проверить wiring:** реально ли `InterviewEngine.instructions` / prefix получает assembled text (gap §5).  
7. Не путать channel OAS (n8n) и story-intake OAS (gateway).
