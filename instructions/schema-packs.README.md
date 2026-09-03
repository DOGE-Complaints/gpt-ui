# Schema packs — active pair index (flat upload layout)

**Product:** DOGEstonia — Module 1 (Custom GPT instruction overlays)  
**Purpose:** Name the **active** node pack artifacts using **flat** filenames (REQ-45a §5). Custom GPT upload drops subdirectories — live SSOT is `schema-packs.<schema_id>.<schema_version>.<artifact>` under `instructions/`.

| Field | Value |
|-------|--------|
| **Version** | 2.0 |
| **Date** | 2026-09-03 |
| **Traceability** | GPT-SSR-11 / REQ-45a §5–§6; GPT-SSR-09 capstone |

**SEC-001:** Gateway Schema Runtime remains authoritative. Pack JSON is **read-only reference** in GPT. Do **not** run validate inside GPT.

## Active pair

| Key | Value |
|-----|--------|
| `schema_id` | `tallinn_civic` |
| `schema_version` | `v1` |

## Locked paths (flat — live upload SSOT)

- [`schema-packs.tallinn_civic.v1.pack.json`](schema-packs.tallinn_civic.v1.pack.json)
- [`schema-packs.tallinn_civic.v1.payload.schema.json`](schema-packs.tallinn_civic.v1.payload.schema.json)
- [`schema-packs.tallinn_civic.v1.taxonomy.json`](schema-packs.tallinn_civic.v1.taxonomy.json)
- [`schema-packs.tallinn_civic.v1.inbound-validation.md`](schema-packs.tallinn_civic.v1.inbound-validation.md)
- [`schema-packs.tallinn_civic.v1.interview-overlay.md`](schema-packs.tallinn_civic.v1.interview-overlay.md)

**Not in this story (SSR-12):** `schema-packs.tallinn_civic.v1.locale-jurisdiction.md` — deferred.

**Core pointer retarget (SSR-13):** orchestrator / normalizer / ingest links still use [`schema-packs/README.md`](schema-packs/README.md) until SSR-13.

**Nested layout retired:** archive [`archive/schema-packs-nested-tallinn_civic-v1/`](archive/schema-packs-nested-tallinn_civic-v1/) · redirect under [`schema-packs/tallinn_civic/`](schema-packs/tallinn_civic/) (not upload SSOT).

Gateway executable SSOT remains nested: `doge-complaints-gateway/schema-packs/tallinn_civic/v1/`.
