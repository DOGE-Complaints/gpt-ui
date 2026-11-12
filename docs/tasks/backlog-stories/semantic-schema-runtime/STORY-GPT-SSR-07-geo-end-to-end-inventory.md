# STORY-GPT-SSR-07 — Geo end-to-end inventory (форматы и стыки)

## Meta
- **Key:** `STORY-GPT-SSR-07-geo-end-to-end-inventory`
- **Пакет:** [`semantic-schema-runtime/`](./INDEX.md)
- **Status:** Done
- **Приоритет:** P1 — prerequisite для SSR-08
- **Тип:** analysis / architecture (read-only inventory → backlog spec)
- **Parent REQ:** [`REQ-45`](../../../requirements/REQ-45-semantic-schema-intake-envelope-compatibility.md) · gateway [`51-semantic-schema-runtime`](../../../../../doge-complaints-gateway/docs/requirements/51-semantic-schema-runtime.md)
- **Siblings:** upstream [GPT-SSR-03](./STORY-GPT-SSR-03-instructions-emit-binding-and-geo.md); downstream [GPT-SSR-08](./STORY-GPT-SSR-08-geo-precision-instance-territory.md)
- **Skill declared:** openai-custom-gpt-builder
- **Depends:** GPT-SSR-03
- **Method:** [`.cursor/rules/analysis.mdc`](../../../../../.cursor/rules/analysis.mdc) — code facts path:line
- **Parallel with:** GPT-SSR-04, GPT-SSR-06
- **Coherence analysis:** [`gpt-ssr-wave2-coherence-analysis-2026-09-01.md`](../../../analysis/gpt-ssr-wave2-coherence-analysis-2026-09-01.md) §9

## Coherence analysis (2026-09-01)

Verified against disk (P3 pkg-000044): geo channels with path:line in [`gpt-ssr-07-geo-e2e-evidence-2026-09-01.md`](../../../analysis/gpt-ssr-07-geo-e2e-evidence-2026-09-01.md). GW-SSR-27 contract landed: `detail_level`, `geo_model`, `gpt_instance_territory` in pack copy; GPT emit / instance STOP remain SSR-08 handoff.

## Зачем простыми словами

Geo в DOGEstonia идёт **несколькими параллельными каналами** одновременно. GPT, оператор и даже разработчик легко путают: «место в разговоре», «структурированный sidecar на wire», «поля в payload для кластеризации», «scope ноды на intake» и «geo на Issue для карты» — это **разные вещи** с разными форматами.

Перед тем как добавлять precision index и territory of responsibility (SSR-08), нужна **сквозная карта as-built**: что где живёт, что gateway реально исполняет, что только описано в GPT Markdown, где уже есть gaps для user use-cases (район vs дом vs регион).

## End-to-end flow (resident → spa)

```text
Resident mentions place (freeform)
        │
        ▼ GPT normalizer (pack geo formation rules — today data-model §4)
        ├── narrative.location_query  (fuzzy string)
        └── geo_detail?  (lat/lon + address object — SSR-03 emit)
        │
        ▼ POST /story-drafts
Gateway intake
        ├── resolve location_query → StoryGeoSnapshot (admin_*)
        ├── parse geo_detail → merge per pack geo_intake.merge
        ├── mirror_to_payload? → structured_payload.geo.*
        ├── geo_scope gate? → 422 GEO_SCOPE_MISMATCH (if location_query + scope)
        └── validate structured_payload (JSON Schema)
        │
        ▼ cluster + Issue projection
        ├── civic buckets: admin_* from snapshot (geo_filter)
        └── exact lenses: structured_payload.geo.* (e.g. geographic_district_micro)
        │
        ▼ GET /node/issues
SPA (civic fields today; geo/schema_card pending spa-16)
```

## Channel matrix (verified as-built)

| Channel | Format | Role | Owner | Evidence |
|---------|--------|------|-------|----------|
| `narrative.location_query` | string | Fuzzy geocode input | Wire OpenAPI | openapi.yaml narrative; gateway resolve |
| root `geo_detail` | `{ lat, lon, address?, detail_level?, … }` | Structured sidecar | Wire + gateway parse | `contracts.py` L153–162, L361–389 |
| `geo_detail.address` | admin + street + house XOR house_range XOR houses[] | Address depth | Wire | `contracts.py:320–341` |
| `structured_payload.geo.*` | strings (+ houses array) + optional `detail_level` | Pack clustering / exact lenses | Gateway pack | `payload.schema.json` L23–49 |
| `geo_model` | precision_levels[], inference | Pack contract (parse) | Gateway pack | `pack.json` L214–224; GW-SSR-27 |
| `gpt_instance_territory` | rules[] (admin_token/…) | Instance scope (GPT STOP) | Pack contract | `pack.json` L226–235; gateway parse-only |
| `pack.geo_intake` | mode, merge, mirror_to_payload | Intake policy | Gateway pack | `pack.json` L207–211 |
| `node_clustering.civic.geo_scope` | `"<level>:<value>"` or null | Intake rejection scope | Gateway pack | `pack.json` L201–202; `scope.py` |
| `node_clustering.civic.geo_filter` | district\|settlement\|region\|country | Cluster bucket level | Gateway pack | `pack.json` L201 |
| Issue `geo` | lat/lon + label + admin_* | Public read / map | Gateway projection | REQ 51 §projection |
| GPT city canon | prose rules | location_query formatting | GPT data-model §4 | **not** in gateway JSON |

## Field-by-field: geo_detail.address vs payload.geo

| Field | geo_detail.address | structured_payload.geo | Sync rule |
|-------|-------------------|------------------------|-----------|
| country | yes | yes | mirror when mirror_to_payload=true |
| region | yes | yes | same |
| settlement | yes | yes | same |
| district | yes | yes | same |
| street | yes | yes | same |
| house | yes (XOR) | yes | same |
| house_range | yes (XOR) | yes | same |
| houses[] | yes (XOR) | yes | same |
| latitude/longitude | geo_detail root only | **no** in payload schema | snapshot persist SSR-24 |
| detail_level | wire optional | payload enum optional | contract GW-SSR-27; GPT emit → SSR-08 |
| legal territory ID | admin_token normalized only | **no** EHAK/NUTS registry | G-GEO-INST-03 |

## Use-case vs as-built (mismatch indicators for SSR-08)

| User use-case | As-built support | Gap ID |
|---------------|------------------|--------|
| Resident: «Tallinn» only (city-level) | location_query + optional geo.settlement; city canon in data-model §4 | OK |
| Resident: «Lasnamäe, street X, house 5» | geo_detail.address depth + mirror | partial — detail_level contract; GPT emit gap | G-GEO-PREC-01 |
| Resident: «Harju maakond» (region-level story) | geo.region + detail_level enum | no GPT inference rules yet | G-GEO-PREC-01 |
| Story about whole district, not a point | geo.district in payload | detail_level enum exists; GPT emit gap | G-GEO-PREC-01 |
| Operator: GPT instance = Tallinn only | node geo_scope + pack gpt_instance_territory | partial OK | — |
| Operator: GPT instance = one district only | settlement rule only in pack | district-only not configured | G-GEO-INST-01 |
| Operator: territory = bounding box on map | bbox in GW contract README | not in tallinn pack rules | G-GEO-INST-02 |
| Operator: territory = EHAK / NUTS / legal ID | admin_token normalized match | no legal ID registry | G-GEO-INST-03 |
| Out-of-scope place on intake | 422 GEO_SCOPE_MISMATCH when location_query + geo_scope | copy in inbound §5 |
| spa map pin | Issue.geo lat/lon when resolved | spa-16 pending consume |

## Gap indicators (handoff to SSR-08)

| ID | Gap | Why it blocks product |
|----|-----|----------------------|
| G-GEO-PREC-01 | **partial** — wire/payload/pack + gateway persist; GPT emit/inference missing | SSR-08 instructions lockstep |
| G-GEO-INST-01 | **partial** — pack block exists; district-only instance not configured | SSR-08 GPT STOP rules |
| G-GEO-INST-02 | **partial** — bbox in GW contract; not in tallinn pack | SSR-08 if map-drawn zone needed |
| G-GEO-INST-03 | open — admin_token only, no EHAK/NUTS | SSR-08 / future registry |
| G-GEO-SYNC-01 | open — GPT data-model lacks sidecar↔payload sync table | SSR-06/09 interview-overlay |
| G-GEO-SPA-01 | spa does not consume Issue.geo | Board map gap (spa-16 parallel) |

## Scope

- T01: flow diagram resident→GPT→gateway→Issue→spa (above — refine at P3 with path:line audit)
- T02: field-by-field matrix geo_detail vs payload.geo vs snapshot vs Issue
- T03: use-case mismatch list vs operator/product scenarios
- T04: gap ID table published for SSR-08 input

## Out of scope

- Implement new geo fields
- spa-16 implementation
- Gateway SSR-22…25 code changes (already Done — document only)
- Map UI design

## Verified current state (As-of-Done P3)

| Fact | Evidence |
|------|----------|
| Evidence artifact | [`gpt-ssr-07-geo-e2e-evidence-2026-09-01.md`](../../../analysis/gpt-ssr-07-geo-e2e-evidence-2026-09-01.md) |
| Pipeline gate | GIM-305 PASS `2026-09-01T16:56:00Z` |
| GEO_SCOPE levels | 4 only — `scope.py` L5–8 |
| geo_intake tallinn | mode optional; mirror true — `pack.json` L209–212 |
| GW-SSR-27 contract on pack copy | geo_model + gpt_instance_territory + detail_level enum |
| Gateway `src/` | **not edited** this story |

## Подзадачи

| ID | Задача | DoD |
|----|--------|-----|
| **T01** | E2E flow diagram | ASCII/mermaid in story; each hop has path evidence |
| **T02** | Field matrix | geo_detail vs payload.geo vs Issue — all keys mapped |
| **T03** | Use-case mismatches | Table ≥8 rows with gap IDs |
| **T04** | Gap handoff | G-GEO-* IDs in SSR-08 Meta |

## Target / Acceptance Criteria

- [x] **AC-GPT-SSR07-01:** all geo channels documented with owner + evidence path.
- [x] **AC-GPT-SSR07-02:** field sync rules explicit (mirror_to_payload behavior).
- [x] **AC-GPT-SSR07-03:** use-case gap list feeds SSR-08 without re-discovery.
- [x] **AC-GPT-SSR07-04:** no invented fields — gaps marked; partial GW-SSR-27 noted.

## Dependencies

- GPT-SSR-03 Done (geo emit baseline).
- Blocks GPT-SSR-08.
- Parallel with GPT-SSR-04, GPT-SSR-06.

## Residual

- Persisted StoryGeoSnapshot column list — see gateway SSR-24 run report; not duplicated here unless audit finds drift.
- Provider-specific geocode behavior — out of pack model scope.
