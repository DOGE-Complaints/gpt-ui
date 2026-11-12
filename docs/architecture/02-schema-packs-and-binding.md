# 02 — Schema packs и `schema_binding`

**Зона:** [`architecture/`](./README.md)  
**As-of:** 2026-09-16  
**SSOT upload:** [`schema-packs.README.md`](../../instructions/schema-packs.README.md)  
**Operator deep-dive:** [`schema-pack-custom-model-operator-manual-ru.md`](../runtime-docs/schema-pack-custom-model-operator-manual-ru.md)

---

## 1. Зачем pack отдельно от core

Module-1 **process** (фазы, STOP, envelope mapping) — shared.  
**Node model** (signals, taxonomy enums, inbound lists, locale frames) — swappable pack (REQ-45 / AC-016).

GPT **не** исполняет pack JSON validate — gateway Schema Runtime авторитетен (**SEC-001**, README L12).

---

## 2. Active pair (as-built)

| Key | Value | Source |
|-----|--------|--------|
| `schema_id` | `uus_veerenni_civic` | `schema-packs.README.md` L18 |
| `schema_version` | `v3` | L19 |

**Locked flat paths** (L23–28):

- `schema-packs.uus_veerenni_civic.v3.pack.json`
- `schema-packs.uus_veerenni_civic.v3.payload.schema.json`
- `schema-packs.uus_veerenni_civic.v3.taxonomy.json`
- `schema-packs.uus_veerenni_civic.v3.inbound-validation.md`
- `schema-packs.uus_veerenni_civic.v3.interview-overlay.md`
- `schema-packs.uus_veerenni_civic.v3.locale-jurisdiction.md`

Gateway nested executable copy: `doge-complaints-gateway/schema-packs/uus_veerenni_civic/v3/` (README L45).

---

## 3. Swap node

Operator swap = обновить **README active pair** + заменить **six** flat files (REQ-45a).  
Не править domain field names в core (AC-016).  
`SPA_BASE` / Actions base URL — отдельно, если меняется хост.

**Verified discrepancy:** index L60 всё ещё описывает `tallinn_civic`/`v1` и «two files» — [01 §4](./01-module1-instruction-layers.md).

---

## 4. Wire: `schema_binding`

Перед stash orchestrator **MUST** читать active pair и эмитить root `schema_binding` (OpenAPI 0.8.0 required; GPT-SSR-03 / `api-orchestrator.md`).

- Envelope `schema_version` = `m2.story_intake_envelope.v2` — **не** путать с pack `schema_version` (`v3`).
- `structured_payload` shape = pack `payload.schema.json` / `field_policy`.
- Optional `geo_detail` / `location_query` — по pack `geo_intake.mode`.

OpenAPI `structured_payload` имеет `additionalProperties: true` на wire — для **strict** Responses tool bridge подменяет model-facing schema активным pack payload ([REQ-47.1 / REQ-01](../../../doge-ai-bridge/docs/requirements/REQ-01-doge-ai-bridge-runtime.md); [05](./05-python-bridge-instruction-delivery.md)).

---

## 5. Path drift

Многие core modules всё ещё ссылаются на nested `schema-packs/README.md`.  
**Live upload SSOT filename:** `schema-packs.README.md` (flat). Nested layout retired / archive (README L39–41).
