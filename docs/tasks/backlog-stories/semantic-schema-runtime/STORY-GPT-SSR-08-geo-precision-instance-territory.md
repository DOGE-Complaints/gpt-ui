# STORY-GPT-SSR-08 — Geo precision index + territory of responsibility (модель GPT instance)

## Meta
- **Key:** `STORY-GPT-SSR-08-geo-precision-instance-territory`
- **Пакет:** [`semantic-schema-runtime/`](./INDEX.md)
- **Status:** Done
- **Приоритет:** P1 — product gaps G-GEO-PREC-01, G-GEO-INST-01…03
- **Тип:** product / architecture (model spec + cross-project dependency)
- **Parent REQ:** [`REQ-45`](../../../requirements/REQ-45-semantic-schema-intake-envelope-compatibility.md) · gateway REQ-35 geo scope
- **Siblings:** upstream [GPT-SSR-07](./STORY-GPT-SSR-07-geo-end-to-end-inventory.md); downstream [GPT-SSR-09](./STORY-GPT-SSR-09-use-case-audit-core-consume-json.md), [GPT-SSR-06](./STORY-GPT-SSR-06-inbound-validation-post-ssr-delta.md) §6
- **Skill declared:** openai-custom-gpt-builder
- **Depends:** GPT-SSR-07
- **Gateway sibling (blocker indicator):** [GW-SSR-27](../../../../../doge-complaints-gateway/docs/tasks/backlog-stories/semantic-schema-runtime/STORY-GW-SSR-27-geo-precision-instance-territory.md) — pack parser + optional persist/projection for `detail_level` / `gpt_instance_territory` (**Done** gateway; GPT STOP this story)
- **Coherence analysis:** [`gpt-ssr-wave2-coherence-analysis-2026-09-01.md`](../../../analysis/gpt-ssr-wave2-coherence-analysis-2026-09-01.md) §10
- **Pipeline / pkg:** pkg-000046 · GIM-312…316 · evidence [`gpt-ssr-08-geo-precision-evidence-2026-09-01.md`](../../../analysis/gpt-ssr-08-geo-precision-evidence-2026-09-01.md)

## Coherence analysis (2026-09-01)

**P3 verified:** pack `geo_model` + `gpt_instance_territory` present (GW-SSR-27); GPT inference + inbound §6 body landed. Early «no detail_level» claim superseded. Residual: district-only config, bbox in tallinn rules, EHAK admin_id, spa-16.

## Зачем простыми словами

Житель описывает место **с разной точностью**: иногда это весь регион или город, иногда район, иногда улица с домом или диапазон домов. Сегодня gateway хранит **плоские строки** в `geo_detail.address` и `structured_payload.geo.*`, но **не говорит**, насколько точно мы поняли место — «это район» или «это конкретный дом». Для карты, кластеризации и UX это создаёт двусмысленность.

Отдельная продуктовая потребность: оператор разворачивает **конкретный Custom GPT instance** (например, только для одного района или одной организации) и хочет **ограничить зону ответственности** этого инстанса. Способы задания territory должны поддерживать:

1. **Юридический / административный идентификатор** — район, город, уезд, регион (EHAK, NUTS, ISO-subdivision — стандарт выбирается позже).
2. **Геометрическая зона** — bounding box (квадрант) по координатам; polygon — indicator for later.

Сегодня есть только **node-level** `node_clustering.civic.geo_scope` (`settlement:tallinn`) — это scope **ноды gateway**, не **инстанса GPT**. District-only GPT instance **не поддерживается**.

## Два уровня scope (conceptual separation)

```text
NODE scope (gateway pack — as-built)
  node_clustering.civic.geo_scope = "settlement:tallinn"
  → intake 422 when location_query resolves outside scope

GPT INSTANCE scope (as-built GPT pre-flight — SSR-08)
  gpt_instance_territory in pack.json
  → GPT STOP / clarify BEFORE stash when resident place outside instance zone
  → may be STRICTER than node scope (district ⊂ city) — tallinn demo = same breadth as node
```

| Level | Who configures | When enforced | As-built (2026-09-01) |
|-------|----------------|---------------|------------------------|
| **Node** | gateway pack `geo_scope` | gateway intake 422 | yes — SSR-22 · `settlement:tallinn` |
| **GPT instance** | pack `gpt_instance_territory` (operator upload) | GPT pre-flight STOP · inbound §6 | **yes** — admin_token settlement tallinn; gateway parse-only |
| **Story precision** | GPT emit `detail_level` from evidence | clustering + map UX | **yes (GPT emit)** — pack `geo_model` + payload enum; spa halo = spa-16 residual |

## Geo precision index (as-built pack + GPT inference)

**Problem:** same address fields used for city-only and house-level stories.

**Pack fields (live tallinn — GW-SSR-27 + GPT-SSR-08):**

```json
{
  "geo_model": {
    "precision_levels": [
      "region",
      "settlement",
      "district",
      "street",
      "house",
      "house_range",
      "coordinates"
    ],
    "default_precision_inference": "from_address_depth"
  }
}
```

**On wire / payload:**

| Location | Field | Meaning |
|----------|-------|---------|
| `geo_detail` or `structured_payload.geo` | `detail_level` | one of precision_levels — semantic precision of this story's place |
| Issue projection | pass-through detail_level | spa map: pin vs district halo vs no pin (spa-16 residual) |

**GPT behavior indicators:**

- When resident gives city only → `detail_level=settlement`; do not invent street/house.
- When resident gives district name only → `detail_level=district`; may omit coordinates.
- When house confirmed → `detail_level=house` or `coordinates` if lat/lon present.

## GPT instance territory (as-built + indicators)

**Live tallinn pack:** `admin_token` / settlement / tallinn (same breadth as node). Indicators below for narrower / bbox / admin_id configs (residual INST-01…03).

**Example blocks (not all configured in tallinn):**

```json
{
  "gpt_instance_territory": {
    "enabled": true,
    "rules": [
      {
        "type": "admin_id",
        "level": "district",
        "scheme": "EHAK",
        "value": "784"
      }
    ]
  }
}
```

```json
{
  "gpt_instance_territory": {
    "enabled": true,
    "rules": [
      {
        "type": "bbox",
        "west": 24.6,
        "south": 59.4,
        "east": 24.8,
        "north": 59.5
      }
    ]
  }
}
```

| type | Use case | Gateway gate today | GPT gate (SSR-08) |
|------|----------|-------------------|-------------------|
| `admin_id` | district-only GPT | no (parse-only) | pre-flight STOP + inbound §6 |
| `bbox` | Map-drawn quad | no (parse-only) | STOP if coords outside box |
| `admin_token` | Normalized settlement/district | no (parse-only) | STOP — tallinn as-built |
| `none` / absent | Full node scope | node geo_scope only | §6 inactive; §5 still applies |

## Use-case matrix (from SSR-07 — expanded)

| Use case | As-built | Target indicator |
|----------|----------|------------------|
| Resident: city only | OK + city canon | + detail_level=settlement |
| Resident: district + street + house | partial address depth | + detail_level=house |
| Region-level civic story (not a point) | geo.region string only | + detail_level=region; map: no pin |
| GPT instance = Tallinn | node geo_scope | OK |
| GPT instance = one district | **gap** | gpt_instance_territory admin_id or token |
| Territory by bbox | **gap** | gpt_instance_territory bbox rule |
| Territory by legal ID (EHAK/NUTS) | **gap** | admin_id + scheme registry in pack |
| Out-of-territory story | 422 at node level | + GPT STOP at instance level first |
| spa map for district-level story | pin if coords exist | respect detail_level — halo vs pin (spa-16) |

## GPT instruction behaviors (per territory type)

| Event | Core | Pack inbound §6 (new) |
|-------|------|-------------------------|
| Place outside instance admin_id | — | STOP — ask to clarify inside zone; do not stash |
| Place outside instance bbox | — | STOP — same |
| Place outside node geo_scope | orchestrator 422 handler | existing inbound §5 copy |
| detail_level below house but coords emitted | normalizer | warn in God Mode; do not invent house |

## Scope

- T01: pack JSON shape indicators — geo_model.precision_levels, detail_level, gpt_instance_territory
- T02: node scope vs instance scope separation doc + config ownership table
- T03: GPT STOP/copy spec per territory type → feeds inbound-validation §6
- T04: gateway sibling dependency list (parser, gate, persist, Issue projection) — analysis only

## Out of scope

- Choosing national ID standard (EHAK vs NUTS) — product decision later
- Map UI / polygon editor
- Gateway `src/` / intake enforce instance territory — [GW-SSR-27](../../../../../doge-complaints-gateway/docs/tasks/backlog-stories/semantic-schema-runtime/STORY-GW-SSR-27-geo-precision-instance-territory.md) Residual (GPT-only MVP)
- spa-16 map toggle implementation
- District-only / bbox tallinn pack reconfiguration (G-GEO-INST-01…02 residual)
- Reopening SSR-09 capstone

**P3 fence (Done):** GPT MD only — inbound §6 + interview-overlay inference; **not** gateway code.

## Verified as-built limits (post GW-SSR-27 / GPT-SSR-08 P3)

| Fact | Evidence |
|------|----------|
| Admin levels | 4: district, settlement, region, country — gateway `scope.py` |
| Scope gate | normalized token match — node `geo_scope` → 422 |
| No bbox **intake gate** | gateway does not enforce instance bbox on intake (GPT STOP only) |
| `detail_level` in payload schema | **yes** — `payload.schema.json` geo.detail_level enum (7 levels) |
| `geo_model` + `gpt_instance_territory` | **yes** — `pack.json` (admin_token settlement tallinn) |
| `geo_intake.mode` | requiredness / merge / mirror — not territory |
| GPT inbound §6 + overlay | **yes** — inbound v1.4; overlay v1.1 |

## Подзадачи

| ID | Задача | DoD |
|----|--------|-----|
| **T01** | Precision + territory JSON indicators | Draft blocks with field names; not implemented |
| **T02** | Node vs instance scope table | Config ownership + enforcement point per level |
| **T03** | GPT behavior + inbound §6 outline | STOP/copy rows per territory type |
| **T04** | Gateway dependency list | Parser, gate, persist, projection rows for GW-SSR-27 |

## Target / Acceptance Criteria

- [x] **AC-GPT-SSR08-01:** precision_levels enum indicator documented with GPT inference rules.
- [x] **AC-GPT-SSR08-02:** gpt_instance_territory supports admin_id and bbox indicators.
- [x] **AC-GPT-SSR08-03:** node vs instance scope separation explicit — no conflation.
- [x] **AC-GPT-SSR08-04:** inbound §6 outline ready for SSR-06 delta.
- [x] **AC-GPT-SSR08-05:** gateway sibling GW-SSR-27 flagged with dependency rows.

## Cross-project dependency (gateway)

| Capability | Gateway story |
|------------|---------------|
| Parse `geo_model` + `gpt_instance_territory` from pack | [GW-SSR-27](../../../../../doge-complaints-gateway/docs/tasks/backlog-stories/semantic-schema-runtime/STORY-GW-SSR-27-geo-precision-instance-territory.md) |
| Optional: gateway enforce instance territory on intake | GW-SSR-27 Residual (may stay GPT-only) |
| Persist `detail_level` on StoryGeoSnapshot | GW-SSR-27 T03 |
| Issue projection `detail_level` for spa | GW-SSR-27 T04 + spa-16 |

## Dependencies

- GPT-SSR-07 inventory (G-GEO-* gap IDs).
- Feeds GPT-SSR-06 §6, GPT-SSR-09 use-case matrix.

## Residual

- Polygon territory — defer; bbox is MVP indicator.
- Multiple territory rules (OR vs AND) — default OR at spec time; confirm at deep analysis.
- Instance territory in README vs separate instance config file — prefer pack overlay block copy-pasted with JSON pack (same swap workflow).
