# Schema packs — active pair and operator swap

**Product:** DOGEstonia — Module 1 (Custom GPT instruction overlays)  
**Purpose:** Name the **active** node schema pair and the two swappable pack files. Core instructions stay node-agnostic; civic (or any other node model) is a **pack instance**.

| Field | Value |
|-------|--------|
| **Version** | 1.0 |
| **Date** | 2026-08-31 |
| **Traceability** | REQ-45 §6.1; GPT-SSR-01 / GIM-248; REQ3-AC-016 |

This directory is **instruction overlay only**. Packs are **non-executable projections** of the gateway schema pack. Gateway Schema Runtime remains authoritative (SEC-001). Do **not** run validate inside GPT.

## Active pair (NODE_SCHEMA_* mirror)

| Key | Value |
|-----|--------|
| `schema_id` | `tallinn_civic` |
| `schema_version` | `v1` |

Locked paths for the active instance:

- [`tallinn_civic/v1/data-model.md`](tallinn_civic/v1/data-model.md)
- [`tallinn_civic/v1/inbound-validation.md`](tallinn_civic/v1/inbound-validation.md)

Layout for any node: `schema-packs/<schema_id>/<schema_version>/{data-model,inbound-validation}.md`.

Envelope identifier on the wire stays `m2.story_intake_envelope.v2` (core). Pack `schema_version` (`v1`) is the **semantic model** pair, not the envelope id.

## Operator swap runbook

To point this Custom GPT at another node pack:

1. Add or replace **two files** under `schema-packs/<new_schema_id>/<new_schema_version>/data-model.md` and `inbound-validation.md` (projections of that node’s gateway pack).
2. Update **this README** active pair (`schema_id`, `schema_version`) to match the node `NODE_SCHEMA_ID` / `NODE_SCHEMA_VERSION`.
3. Align the **emitted binding pair** with the new active pair (orchestrator MUST emit — GPT-SSR-03).
4. Upload **Instructions** (core + new pack files). Pack `.md` only → **no Actions re-import**.
5. OpenAPI / Actions re-import **only** when the Actions schema changes (GPT-SSR-02).

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
