# Schema packs — active pair and operator swap

**Product:** DOGEstonia — Module 1 (Custom GPT instruction overlays)  
**Purpose:** Name the **active** node schema pair and the two swappable pack files. Core instructions stay node-agnostic; civic (or any other node model) is a **pack instance**.

| Field | Value |
|-------|--------|
| **Version** | 1.2 |
| **Date** | 2026-09-01 |
| **Traceability** | REQ-45 §6.1; GPT-SSR-01 / GIM-248; GPT-SSR-04 / GIM-278; GPT-SSR-05 / GIM-288; REQ3-AC-016 |

This directory is **instruction overlay only**. Packs are **non-executable projections** of the gateway schema pack. Gateway Schema Runtime remains authoritative (SEC-001). Do **not** run validate inside GPT.

## Active pair (NODE_SCHEMA_* mirror)

| Key | Value |
|-----|--------|
| `schema_id` | `tallinn_civic` |
| `schema_version` | `v1` |

Locked paths for the active instance:

- [`tallinn_civic/v1/data-model.md`](tallinn_civic/v1/data-model.md)
- [`tallinn_civic/v1/inbound-validation.md`](tallinn_civic/v1/inbound-validation.md)

Layout for any node: `schema-packs/<schema_id>/<schema_version>/` — wave 1 locked `{data-model,inbound-validation}.md`; **wave 2 target (SSR-04 + SSR-05 locked):** `{pack.json,payload.schema.json,taxonomy.json}` literal copy from gateway + `inbound-validation.md` (+ interim `data-model.md` until SSR-09 archive). README active-path rename to JSON → SSR-09.

### Wave 2 target layout (SSR-04 + SSR-05 locked spec)

| Artifact | Role | SEC-001 |
|----------|------|---------|
| `pack.json` | Gateway field_policy, geo_intake, clustering — **read-only reference** | Non-executable |
| `payload.schema.json` | Required signals / geo shape — **read-only reference** | Non-executable |
| `taxonomy.json` | 13 axes, canonical keys, axis_to_signal_map — **read-only reference** (GW-SSR-26/28) | Non-executable |
| `inbound-validation.md` | Content / admission rules (dual contour) | GPT process |
| `data-model.md` | Interim projection — **deprecate SSR-09** | Non-executable |

Live instance `tallinn_civic/v1/`: `pack.json` + `payload.schema.json` + `taxonomy.json` byte-identical to gateway (`cmp` ok, P3 SSR-04 + SSR-05).

## Operator swap runbook

To point this Custom GPT at another node pack, follow **authoritative checklist** [`schema-pack-custom-model-operator-manual-ru.md` §5 v2](../../docs/runtime-docs/schema-pack-custom-model-operator-manual-ru.md) (byte-identical JSON copy-paste; wave-1 MD projection is Historical only).

**Summary (v2 — GPT-SSR-04 + SSR-05):**

1. **Copy gateway JSON** — byte-identical `pack.json` + `payload.schema.json` + `taxonomy.json` from `doge-complaints-gateway/schema-packs/<new_schema_id>/<new_schema_version>/` into `schema-packs/<new_schema_id>/<new_schema_version>/` (optional `cmp` before upload). Requires gateway SSOT on disk ([GW-SSR-26/28/29](../../../../doge-complaints-gateway/docs/tasks/backlog-stories/semantic-schema-runtime/INDEX.md)).
2. **Inbound rules** — add or update `inbound-validation.md` (content / admission; dual contour — SSR-06).
3. **Active pair** — update **this README** table (`schema_id`, `schema_version`) and locked-path links to match `NODE_SCHEMA_ID` / `NODE_SCHEMA_VERSION` (README rename to JSON filenames → SSR-09).
4. **Emit alignment** — orchestrator MUST emit binding pair matching the new active pair (GPT-SSR-03).
5. **Upload Instructions** — core + README + pack artifacts (`pack.json`, `payload.schema.json`, `taxonomy.json`, `inbound-validation.md`, interim `data-model.md` if still present). JSON/MD-only pack change → **no Actions re-import**.
6. **Actions re-import** — **only** when OpenAPI / Actions schema changes (GPT-SSR-02).

> **Historical (wave 1):** manual JSON→MD projection (`data-model.md` only) — deprecated for new swaps; see manual §5 Historical block.

### AC-016 checklist (swap must not require core domain-field edits)

| # | Check | Pass when |
|---|--------|-----------|
| 1 | Interview phases / Phase 7 affirmation process | Unchanged in `story-interview-flow.md`; field lists come from the data-model pack. |
| 2 | Envelope id | Still `m2.story_intake_envelope.v2` in orchestrator / OpenAPI. |
| 3 | Active pair | Read from this README — not hardcoded civic names in core. |
| 4 | Geo gate shape | Parameterized by pack `geo_intake.mode` (orchestrator emit + pre-flight = SSR-03). |
| 5 | PII / admission process | Unchanged in normalizer / orchestrator. |
| 6 | Domain field names (`signals.*`, city canon, civic axes) | Live in the two pack files, not as the sole core model. |

If a swap requires editing orchestrator / interview-flow **for domain field names**, AC-016 fails.

## Re-import policy

| Change | Operator action |
|--------|-----------------|
| Pack `.md` only | Upload Instructions. No Actions re-import. |
| OpenAPI / Actions schema | GPT-SSR-02 + Actions re-import. |

## Out of this file

- OpenAPI `schema_binding` / `geo_detail` wire tokens — GPT-SSR-02.
- Stash emit of binding / geo — GPT-SSR-03 (**done**: orchestrator MUST emit matching pair).
- Gateway validate / Schema Runtime.
- Marketplace / §30 extraction.
