# Schema packs — active pair and operator swap

**Product:** DOGEstonia — Module 1 (Custom GPT instruction overlays)  
**Purpose:** Name the **active** node schema pair and swappable pack artifacts. Core instructions stay node-agnostic; civic (or any other node model) is a **pack instance**.

| Field | Value |
|-------|--------|
| **Version** | 2.1 |
| **Date** | 2026-09-03 |
| **Traceability** | GPT-SSR-12 / GIM-345; REQ-45a §5–§6; GPT-SSR-11; GPT-SSR-09 capstone; REQ3-AC-016 |

This directory holds **navigation + operator runbook**. Live upload SSOT for the active instance is **flat** under `instructions/` (GPT-SSR-11). See also [`schema-packs.README.md`](../schema-packs.README.md).

**SEC-001:** Gateway Schema Runtime remains authoritative. Pack JSON is **non-executable read-only reference**. Do **not** run validate inside GPT.

## Active pair (NODE_SCHEMA_* mirror)

| Key | Value |
|-----|--------|
| `schema_id` | `tallinn_civic` |
| `schema_version` | `v1` |

## Locked paths (flat layout — wave 4)

Pattern: `schema-packs.<schema_id>.<schema_version>.<artifact>` (REQ-45a §5).

- [`schema-packs.tallinn_civic.v1.pack.json`](../schema-packs.tallinn_civic.v1.pack.json)
- [`schema-packs.tallinn_civic.v1.payload.schema.json`](../schema-packs.tallinn_civic.v1.payload.schema.json)
- [`schema-packs.tallinn_civic.v1.taxonomy.json`](../schema-packs.tallinn_civic.v1.taxonomy.json)
- [`schema-packs.tallinn_civic.v1.inbound-validation.md`](../schema-packs.tallinn_civic.v1.inbound-validation.md)
- [`schema-packs.tallinn_civic.v1.interview-overlay.md`](../schema-packs.tallinn_civic.v1.interview-overlay.md)
- [`schema-packs.tallinn_civic.v1.locale-jurisdiction.md`](../schema-packs.tallinn_civic.v1.locale-jurisdiction.md)

**Nested instance retired (SSR-11):** no nested upload SSOT under `schema-packs/`. Archive: [`archive/schema-packs-nested-tallinn_civic-v1/`](../archive/schema-packs-nested-tallinn_civic-v1/). Redirect stub: [`tallinn_civic/README.md`](tallinn_civic/README.md) (if present).

### Artifact roles (SEC-001)

| Artifact | Role | SEC-001 |
|----------|------|---------|
| `*.pack.json` | Gateway field_policy, geo_intake, clustering — **read-only reference** | Non-executable |
| `*.payload.schema.json` | Required signals / geo shape — **read-only reference** | Non-executable |
| `*.taxonomy.json` | 13 axes, canonical keys, axis_to_signal_map — **read-only reference** | Non-executable |
| `*.inbound-validation.md` | Content / admission rules (dual contour) | GPT process |
| `*.interview-overlay.md` | Geo formation pointers, phase→axes, ecosystem cues | GPT process |
| `*.locale-jurisdiction.md` | Working languages, script policy, jurisdiction framing, STOP copy language | GPT process |

Live flat JSON byte-identical to archived nested GPT copy (`cmp` ok at P3). Gateway nested layout unchanged (AC-GPT-REQ45a scope).

## Operator swap runbook

To point this Custom GPT at another node pack, follow **authoritative checklist** [`schema-pack-custom-model-operator-manual-ru.md` §5 v3](../../docs/runtime-docs/schema-pack-custom-model-operator-manual-ru.md) (byte-identical JSON copy-paste).

**Summary (v4 — flat layout after GPT-SSR-11):**

1. **Copy gateway JSON** — byte-identical `pack.json` + `payload.schema.json` + `taxonomy.json` from `doge-complaints-gateway/schema-packs/<new_schema_id>/<new_schema_version>/` into flat `schema-packs.<id>.<ver>.{pack.json,payload.schema.json,taxonomy.json}` under `instructions/` (optional `cmp` before upload).
2. **Inbound + overlay** — add or update flat `schema-packs.<id>.<ver>.{inbound-validation,interview-overlay}.md`.
3. **Active pair** — update **this README** + [`schema-packs.README.md`](../schema-packs.README.md) (`schema_id`, `schema_version`) and locked-path links.
4. **Emit alignment** — orchestrator MUST emit binding pair matching the new active pair (GPT-SSR-03).
5. **Upload Instructions** — core + flat pack artifacts + README index files. JSON/MD-only pack change → **no Actions re-import**.
6. **Actions re-import** — **only** when OpenAPI / Actions schema changes (GPT-SSR-02).

> **Historical:** nested `schema-packs/<id>/<ver>/` wave 2 layout — archived SSR-11. Wave 1 `data-model.md` — archived SSR-09.

### AC-016 checklist (swap must not require core domain-field edits)

| # | Check | Pass when |
|---|--------|-----------|
| 1 | Interview phases / Phase 7 affirmation process | Unchanged in `story-interview-flow.md`; field lists come from **flat interview-overlay**. |
| 2 | Envelope id | Still `m2.story_intake_envelope.v2` in orchestrator / OpenAPI. |
| 3 | Active pair | Read from this README — not hardcoded civic names in core. |
| 4 | Geo gate shape | Parameterized by **flat `pack.json`** `geo_intake.mode` (orchestrator emit + pre-flight = SSR-03). |
| 5 | PII / admission process | Unchanged in normalizer / orchestrator. |
| 6 | Domain field names (`signals.*`, city canon, civic axes) | Live in flat pack JSON + overlay, not as the sole core model. |

## Re-import policy

| Change | Operator action |
|--------|-----------------|
| Pack `.md` / JSON only | Upload Instructions. No Actions re-import. |
| OpenAPI / Actions schema | GPT-SSR-02 + Actions re-import. |

## Wave 4 handoff (SSR-13)

| Item | Story |
|------|-------|
| Core Module-1 links → flat paths + allowlist read pack locale | **SSR-13** — orchestrator / normalizer / ingest retarget |

## Out of this file

- OpenAPI `schema_binding` / `geo_detail` wire tokens — GPT-SSR-02.
- Stash emit of binding / geo — GPT-SSR-03 (**done**).
- Gateway validate / Schema Runtime (nested SSOT unchanged).
- Pack Builder / REQ-46.
