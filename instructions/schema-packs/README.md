# Schema packs — active pair and operator swap

**Product:** DOGEstonia — Module 1 (Custom GPT instruction overlays)  
**Purpose:** Name the **active** node schema pair and swappable pack artifacts. Core instructions stay node-agnostic; civic (or any other node model) is a **pack instance**.

| Field | Value |
|-------|--------|
| **Version** | 1.3 |
| **Date** | 2026-09-01 |
| **Traceability** | REQ-45 §6.1; GPT-SSR-01 / GIM-248; GPT-SSR-04 / GIM-278; GPT-SSR-05 / GIM-288; GPT-SSR-09 / GIM-309; REQ3-AC-016 |

This directory is **instruction overlay only**. Packs are **non-executable projections** of the gateway schema pack. Gateway Schema Runtime remains authoritative (SEC-001). Do **not** run validate inside GPT.

## Active pair (NODE_SCHEMA_* mirror)

| Key | Value |
|-----|--------|
| `schema_id` | `tallinn_civic` |
| `schema_version` | `v1` |

Locked paths for the active instance (wave 2 — GPT-SSR-09):

- [`tallinn_civic/v1/pack.json`](tallinn_civic/v1/pack.json)
- [`tallinn_civic/v1/payload.schema.json`](tallinn_civic/v1/payload.schema.json)
- [`tallinn_civic/v1/taxonomy.json`](tallinn_civic/v1/taxonomy.json)
- [`tallinn_civic/v1/inbound-validation.md`](tallinn_civic/v1/inbound-validation.md)
- [`tallinn_civic/v1/interview-overlay.md`](tallinn_civic/v1/interview-overlay.md)

Archived (do not use as SSOT): [`tallinn_civic/v1/archive/data-model.v1.3.md`](tallinn_civic/v1/archive/data-model.v1.3.md)

Layout for any node: `schema-packs/<schema_id>/<schema_version>/` — literal `{pack.json,payload.schema.json,taxonomy.json}` copy from gateway + `{inbound-validation,interview-overlay}.md`.

### Wave 2 layout (SSR-04 + SSR-05 + SSR-09)

| Artifact | Role | SEC-001 |
|----------|------|---------|
| `pack.json` | Gateway field_policy, geo_intake, clustering — **read-only reference** | Non-executable |
| `payload.schema.json` | Required signals / geo shape — **read-only reference** | Non-executable |
| `taxonomy.json` | 13 axes, canonical keys, axis_to_signal_map — **read-only reference** | Non-executable |
| `inbound-validation.md` | Content / admission rules (dual contour) | GPT process |
| `interview-overlay.md` | Geo canon, phase→axes, ecosystem cues | GPT process |

Live instance `tallinn_civic/v1/`: three JSON files byte-identical to gateway (`cmp` ok, SSR-04 + SSR-05 + P3 verify SSR-09).

## Operator swap runbook

To point this Custom GPT at another node pack, follow **authoritative checklist** [`schema-pack-custom-model-operator-manual-ru.md` §5 v3](../../docs/runtime-docs/schema-pack-custom-model-operator-manual-ru.md) (byte-identical JSON copy-paste).

**Summary (v3 — GPT-SSR-09 capstone):**

1. **Copy gateway JSON** — byte-identical `pack.json` + `payload.schema.json` + `taxonomy.json` from `doge-complaints-gateway/schema-packs/<new_schema_id>/<new_schema_version>/` into `schema-packs/<new_schema_id>/<new_schema_version>/` (optional `cmp` before upload).
2. **Inbound + overlay** — add or update `inbound-validation.md` + `interview-overlay.md` (content / interview guidance).
3. **Active pair** — update **this README** table (`schema_id`, `schema_version`) and locked-path links to match `NODE_SCHEMA_ID` / `NODE_SCHEMA_VERSION`.
4. **Emit alignment** — orchestrator MUST emit binding pair matching the new active pair (GPT-SSR-03).
5. **Upload Instructions** — core + README + pack artifacts (JSON×3 + inbound + interview-overlay). JSON/MD-only pack change → **no Actions re-import**.
6. **Actions re-import** — **only** when OpenAPI / Actions schema changes (GPT-SSR-02).

> **Historical (wave 1):** `data-model.md` MD projection — archived SSR-09; see `archive/data-model.v1.3.md`.

### AC-016 checklist (swap must not require core domain-field edits)

| # | Check | Pass when |
|---|--------|-----------|
| 1 | Interview phases / Phase 7 affirmation process | Unchanged in `story-interview-flow.md`; field lists come from **`interview-overlay.md`**. |
| 2 | Envelope id | Still `m2.story_intake_envelope.v2` in orchestrator / OpenAPI. |
| 3 | Active pair | Read from this README — not hardcoded civic names in core. |
| 4 | Geo gate shape | Parameterized by **`pack.json`** `geo_intake.mode` (orchestrator emit + pre-flight = SSR-03). |
| 5 | PII / admission process | Unchanged in normalizer / orchestrator. |
| 6 | Domain field names (`signals.*`, city canon, civic axes) | Live in pack JSON + overlay, not as the sole core model. |

If a swap requires editing orchestrator / interview-flow **for domain field names**, AC-016 fails.

## Re-import policy

| Change | Operator action |
|--------|-----------------|
| Pack `.md` / JSON only | Upload Instructions. No Actions re-import. |
| OpenAPI / Actions schema | GPT-SSR-02 + Actions re-import. |

## Out of this file

- OpenAPI `schema_binding` / `geo_detail` wire tokens — GPT-SSR-02.
- Stash emit of binding / geo — GPT-SSR-03 (**done**: orchestrator MUST emit matching pair).
- Gateway validate / Schema Runtime.
- Marketplace / §30 extraction.
