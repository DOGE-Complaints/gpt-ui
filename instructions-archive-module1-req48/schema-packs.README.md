# Schema packs — active pair index (flat upload layout)

**Product:** DOGEstonia — Module 1 (Custom GPT instruction overlays)  
**Purpose:** Name the **active** node pack artifacts using **flat** filenames (REQ-45a §5). Custom GPT upload drops subdirectories — live SSOT is `schema-packs.<schema_id>.<schema_version>.<artifact>` under `instructions/`.

| Field | Value |
|-------|--------|
| **Version** | 2.2 |
| **Date** | 2026-09-07 |
| **Traceability** | uus v3 ship · GPT-SSR-12 / GIM-345; GPT-SSR-11 / REQ-45a §5–§6; GPT-SSR-09 capstone |

**SEC-001:** Gateway Schema Runtime remains authoritative. Pack JSON is **read-only reference** in GPT. Do **not** run validate inside GPT.

## Active pair

| Key | Value |
|-----|--------|
| `schema_id` | `uus_veerenni_civic` |
| `schema_version` | `v3` |

## Locked paths (flat — live upload SSOT)

- [`schema-packs.uus_veerenni_civic.v3.pack.json`](schema-packs.uus_veerenni_civic.v3.pack.json)
- [`schema-packs.uus_veerenni_civic.v3.payload.schema.json`](schema-packs.uus_veerenni_civic.v3.payload.schema.json)
- [`schema-packs.uus_veerenni_civic.v3.taxonomy.json`](schema-packs.uus_veerenni_civic.v3.taxonomy.json)
- [`schema-packs.uus_veerenni_civic.v3.inbound-validation.md`](schema-packs.uus_veerenni_civic.v3.inbound-validation.md)
- [`schema-packs.uus_veerenni_civic.v3.interview-overlay.md`](schema-packs.uus_veerenni_civic.v3.interview-overlay.md)
- [`schema-packs.uus_veerenni_civic.v3.locale-jurisdiction.md`](schema-packs.uus_veerenni_civic.v3.locale-jurisdiction.md)

## Emit / validate hashes (documented; not pack.json keys)

| Hash | Value |
|------|--------|
| `prompt_version_hash` | `pending-pb13` (Pack Builder primary body still pending GPT-PB-13) |
| `standards_set_hash` | `sha256:c3f11ad8c55c28096e0b9b9f8142c0f3fb519c93096c48fc1414ece76cdafa01` (node-onboarding four standards as of 2026-09-07) |

**Note:** Pack meta (`additionalProperties: false`) does **not** accept these as `pack.json` fields — keep them here / in prose headers.

**Core pointer retarget (SSR-13):** orchestrator / normalizer / ingest links still use [`schema-packs/README.md`](schema-packs/README.md) until SSR-13.

**Nested layout retired:** archive under [`archive/`](archive/) · redirects under [`schema-packs/`](schema-packs/) (not upload SSOT).

**Historical candidate retained:** flat `schema-packs.uus_veerenni_civic.v2.*` (geo_load_gate FAIL — superseded by v3).

Gateway executable SSOT (nested short names): `doge-complaints-gateway/schema-packs/uus_veerenni_civic/v3/`.
