# REQ-45: Semantic Schema Runtime — GPT core vs swappable pack

Parent: docs/requirements backlog/REQ3-Semantic-Schema-Runtime-and-Node-Specific-Data-Processing.md REQ3-GPT-001/002, REQ3-FLOW-003, REQ3-PR-002/004, REQ3-COMPAT-001, REQ3-AC-016, §30 separately scoped T-wave (not marketplace LLM-for-every-schema)  
Parent-id: req3-semantic-schema-runtime  
Siblings: doge-complaints-gateway/docs/requirements/51-semantic-schema-runtime.md (contract: envelope `m2.story_intake_envelope.v2`; **MUST** root `schema_binding` matching node `NODE_SCHEMA_ID`/`NODE_SCHEMA_VERSION`; **MAY** root `geo_detail` + `location_query` per pack `geo_intake`; gateway authoritative validate) · spa-app/docs/requirements/16-semantic-schema-spa-compatibility.md (contract: none named GPT↔SPA HTTP; stories via GPT, board via gateway Issues) · doge-identity-service/docs/requirements/20-semantic-schema-identity-boundary.md (contract: none named for schema runtime; existing OAuth/intake auth outside this REQ)  
Continuation: [REQ-45a](./REQ-45a-pack-prose-identities-and-flat-layout.md) — three pack prose identities (`inbound-validation`, `interview-overlay`, `locale-jurisdiction`) + flat `schema-packs.*` Instructions layout  
Related (authoring consumer): [REQ-46](./REQ-46-pack-builder-prompt-wrapper-and-repair-debug.md) — Pack Builder emits REQ-45a layout; orthogonal tooling  
Status: Accepted — package delivered (GPT-SSR-01…03 Done); PA.2 closed by operator 2026-09-01T10:12:26Z  

Версия: 0.5 · 2026-09-03 (Continuation link REQ-45a + Related REQ-46; supersedes 0.4 header-only sync)  
Проект: GPT UI  
Фокус: universal instruction core + two swappable pack artifacts; emit gateway-named intake wire — **без** invented OpenAPI names beyond gateway contracts

---

## 1) Goal

Keep one Custom GPT runtime whose **instruction core** (interview process, handoff, privacy stop-lines, REQ3-PR-004) does not embed a node data model. Node model and inbound content validation SHALL be **two swappable pack artifacts** (REQ3-GPT-001) projected from the **active** gateway schema pack (`NODE_SCHEMA_*` on the node).

Required Actions envelope identifier remains `m2.story_intake_envelope.v2`. GPT **MUST** emit root sidecar **`schema_binding`** (gateway D-SSR-1 / D-SSR-8). GPT **MAY** emit root **`geo_detail`** and/or narrative `location_query` subject to pack **`geo_intake`**.

This is **not** LLM extraction for every schema / marketplace (§30). It is file-swap of pack projections for the node’s active model.

---

## 2) Scope / Out of scope

### In scope

- Instruction core stays node-agnostic: swapping pack MUST NOT require editing interview-flow / orchestrator for domain field names (REQ3-AC-016).
- Two pack artifacts: (1) data-model projection of the gateway schema pack (payload fields, `geo_intake`, exact/civic clustering notes for interview); (2) inbound-validation projection (REQ3-GPT-002). Paths are overlay implementation, not frozen here.
- Wire emit (names from sibling 51 / gateway parser only):
  - required `schema_version` = `m2.story_intake_envelope.v2`;
  - required root **`schema_binding`**: `schema_id`, `schema_version` (semantic model pair = active node pack), `structured_payload` per pack JSON Schema; optional `profile_id` / `profile_version`;
  - optional root **`geo_detail`**: optional lat/lon pair; `address` components (country/region/settlement/district/street/`house` \| `house_range` \| `houses[]`); `normalized_label`; optional `confidence` / `provider`;
  - optional `narrative.location_query` for fuzzy resolve when pack allows.
- Civic pack as one instance; do not weld civic taxonomy as the only core model.
- Narrative-first (REQ3-PR-004); structured + geo additional.

### Out of scope

- Implementing Schema Runtime inside GPT UI (gateway owns validate — RUNTIME-005 / SEC-001).
- Inventing OpenAPI fields not in gateway intake contracts.
- Filling vacant REQ-37; REQ-44.
- Second Custom GPT product; schema marketplace; WASM.
- Entire legal policy engine (SCHEMA-004).
- Identity claim invent; SPA board/map UI.
- Product implementation of OpenAPI/instruction files in this REQ edit (requirements only).

---

## 3) Verified current state

| Fact | Path |
|------|------|
| Stash request required `schema_version` | `GPT UI/docs/custom-gpt-story-intake-actions.openapi.yaml` enum `[m2.story_intake_envelope.v2]` |
| OpenAPI required `schema_binding` + optional `geo_detail` | same file `info.version` **0.8.0**; `StoryDraftStashRequest.required` includes `schema_binding` (GPT-SSR-02 Done) |
| Optional sidecar precedent | same OpenAPI: `gpt_signals` / `privacy` / `origin` |
| Orchestrator MUST emit binding / pack geo mode | `GPT UI/instructions/api-orchestrator.md` (GPT-SSR-03 Done) |
| Civic pack projections (active pair) | `GPT UI/instructions/schema-packs/README.md` + `tallinn_civic/v1/{data-model,inbound-validation}.md` |
| Civic taxonomy file (pack Residual include) | `GPT UI/instructions/story-label-taxonomy.md` |
| Gateway: binding **required**; omit → 4xx | `doge-complaints-gateway/src/core/application/services.py` + intake `contracts.py` |
| Gateway: root `geo_detail` + pack `geo_intake` | intake `contracts.py`; pack.json `geo_intake` |
| Parent tokens only as loose «FLOW-003 optional» | **Superseded:** binding is **required** on node intake |
| Backlog package | `GPT UI/docs/tasks/backlog-stories/semantic-schema-runtime/` — GPT-SSR-01…03 **Done** |

---

## 4) Target behavior

1. Required envelope `schema_version` remains `m2.story_intake_envelope.v2`.
2. Every stash/submit story body **MUST** include `schema_binding` whose `(schema_id, schema_version)` equals the node’s active pack (operator-configured; same pair gateway enforces via `NODE_SCHEMA_*`).
3. `structured_payload` MUST conform to the pack’s payload schema when the pack defines structured fields.
4. When the pack `geo_intake.mode` is `require_detail` or `require_location_or_detail`, GPT SHALL collect and emit `geo_detail` and/or `location_query` accordingly; `optional` mode MAY omit geo.
5. Operator/node change = replace the two pack artifacts (+ align emitted binding pair); core instructions unchanged.
6. Gateway remains authoritative; GPT pack is non-executable projection (SEC-001).
7. Do not require marketplace-wide extraction (§30).

---

## 5) Acceptance criteria

1. AC-GPT-REQ3-01: REQ3-GPT-001 — core vs two pack artifacts; civic is a pack instance.
2. AC-GPT-REQ3-02: OpenAPI/instructions add **only** gateway-named wire tokens (`schema_binding`, `structured_payload`, `geo_detail`, existing narrative/`location_query`) — no invented names.
3. AC-GPT-REQ3-03: v2 remains the required envelope identifier (COMPAT-001).
4. AC-GPT-REQ3-04: REQ3-PR-004 — narrative-first; structured + geo additional.
5. AC-GPT-REQ3-16: REQ3-AC-016 — swapping pack does not require core domain-field edits.
6. AC-GPT-BIND-01: emit **MUST** include `schema_binding` matching active node pair (omit → gateway 4xx).
7. AC-GPT-GEO-01: pack artifacts document `geo_intake`; emit respects mode (`geo_detail` / `location_query`).

Former 0.1 freeze ACs and 0.2 «optional omit binding» are **superseded**.

---

## 6) Open questions

1. Exact filenames/paths of the two pack artifacts in the GPT repo — **Closed (GPT-SSR-01 / GIM-248):** `GPT UI/instructions/schema-packs/<schema_id>/<schema_version>/{data-model,inbound-validation}.md` + `schema-packs/README.md` active pair (`tallinn_civic`/`v1`).
2. Whether OpenAI Custom GPT re-import is required when only pack files change — **Closed (GPT-SSR-01 Meta):** pack `.md` only → upload Instructions (no Actions re-import). OpenAPI change → SSR-02 / Actions re-import.
3. How multiple node packs are stored operationally — **Closed (GPT-SSR-01):** dirs `<schema_id>/<schema_version>/` under `GPT UI/instructions/schema-packs/` (not branches).

**Closed:** wire token names = gateway `schema_binding` / `geo_detail` (sibling 51).

---

## 7) Dependencies

- Parent GPT-001/002, FLOW-003, PR-002/004, COMPAT-001, AC-016, §30.
- Sibling 51 (parser + `NODE_SCHEMA_*` + `geo_intake`).
- Existing GPT REQ-18, REQ-22, REQ-20 (taxonomy file pattern), OpenAPI stash schema (0.8.0 includes `schema_binding` / `geo_detail`).
- **Continuation:** [REQ-45a](./REQ-45a-pack-prose-identities-and-flat-layout.md) — interview-overlay + locale-jurisdiction as first-class pack identities; flat `schema-packs.*` naming (Custom GPT upload). Wave 1 open Q1 nested paths remain historical; migrate outcome owned by REQ-45a.
