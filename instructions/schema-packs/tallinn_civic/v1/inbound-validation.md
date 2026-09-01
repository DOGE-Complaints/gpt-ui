# Pack inbound-validation — `tallinn_civic` / `v1`

**Product:** DOGEstonia — Module 1 (GPT instruction overlay)  
**Purpose:** Domain admission checklists, label-enum guidance, and completeness vs pack `field_policy` for the civic instance. **Not** HTTP Actions schemas. **Not** executable validate.

| Field | Value |
|-------|--------|
| **Version** | 1.3 |
| **Date** | 2026-09-01 |
| **Traceability** | GPT-SSR-06 / GIM-295; GPT-SSR-05 / GIM-288; GPT-SSR-04 / GIM-278; GPT-SSR-03 / GIM-270; GPT-SSR-01 / GIM-250; REQ3-GPT-002; SEC-001 |
| **Binding pair** | `schema_id=tallinn_civic`, `schema_version=v1` |

**SEC-001:** Gateway Schema Runtime remains authoritative. This pack is a **non-executable projection**. Do not call APIs from this file. Do not duplicate OpenAPI request bodies.

Admission **process** (when to BLOCK / clarify / approve) stays in [`story-policy-gate.md`](../../../story-policy-gate.md). Label **process** stays in [`ingest-validation.md`](../../../ingest-validation.md). Domain **lists** for this civic instance live here.

## 1. Mission / standing / `IRRELEVANT_NON_CIVIC` (domain lists)

When operator profile is `demo_baseline`, apply these **domain** classes (process in policy-gate §7.1):

- **BLOCK `IRRELEVANT_NON_CIVIC`:** clear off-topic / non-civic content (not a personal story with standing).
- **BLOCK `SCAM_OR_SPAM`:** scam, phishing, spam, promo bait.
- **BLOCK `OBSCENE_OR_TROLL`:** obscene / sexualized / trolling unrelated to civic intake.
- **BLOCK `NEIGHBOR_GOSSIP`:** others' affairs without standing.
- **needs_clarification `RELEVANCE_UNCLEAR`:** weakly relevant but noisy content where civic intent is not explicit.
- **ACCEPT:** personal stories with standing (self or personally affected) — must **not** map to `IRRELEVANT_NON_CIVIC`.
- **ACCEPT:** recognizably civic content that can continue the strict chain.
- Do **not** require a formal civic-complaint frame. Personal ≠ out of mission.

## 2. Label enum / multi-axis evidence checklists

Validate candidate labels against same-pack [`taxonomy.json`](taxonomy.json) (`canonical_keys` + axis disposition). [`story-label-taxonomy.md`](../../../story-label-taxonomy.md) remains narrative glossary / process until SSR-09 — **not** the pack swap SSOT for enum checks (GPT-SSR-05 / GPT-SSR-06).

Every value destined for `canonical_payload.labels[]` must be a canonical allowed key with story evidence.

- Unknown free-text, metadata-only, internal-only safety/privacy, and low-confidence hypotheses must **not** enter canonical labels.
- Keep them in validation notes as `metadata_only`, `needs_clarification`, or `rejected`.
- Multi-axis: do not collapse a civic narrative onto a single axis when evidence supports more than one (REQ-33 / REQ-36 / REQ-38 rules remain in core normalizer; this pack supplies the civic domain lists).

## 3. Completeness vs pack required fields

Before stash, this civic instance requires structured signals per same-pack [`pack.json`](pack.json) `field_policy` (literal):

- `signals.civic_domain` → **required** (`pack.json` → `field_policy["signals.civic_domain"]`)
- `signals.failure_pattern` → **required** (`pack.json` → `field_policy["signals.failure_pattern"]`)

Shape reference: same-pack [`payload.schema.json`](payload.schema.json) — **when** the pack structured payload is being filled.

Core still requires logical Issue §4.1 `type`, `labels`, `title`, `description` for interview completeness. Pack required `signals.*` are **additional** for this instance — not a replacement for narrative-first intake (REQ3-PR-004).

Do **not** encode HTTP Action schemas here.

## 4. No leading taxonomy questions (privacy baseline)

Do not ask the resident to pick labels or confirm taxonomy keys mid-interview. Capture evidence in phases; map to pack axes after Phase 7 affirmation. Same privacy baseline as REQ-35 §4.3 / interview-flow §7.3–§7.4.

## 5. GEO_SCOPE_MISMATCH copy (this civic instance)

When orchestrator handles HTTP 422 `GEO_SCOPE_MISMATCH`, use this **pack copy**: the demo geo scope is Estonia / Tallinn (`pack.json` → `node_clustering.civic.geo_scope` = `settlement:tallinn`). Ask the resident to clarify a place inside that region. Core orchestrator keeps HTTP handling only — it must not weld this city list.

## 6. Instance territory (outline only — GPT-SSR-08)

**Reserved:** resident-facing STOP/copy for instance territory precision. Body lands when SSR-08 / GW-SSR-27 deliver the territory model. Do **not** invent territory rules in this file until that story.
