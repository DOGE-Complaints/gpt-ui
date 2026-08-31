# Pack inbound-validation — `tallinn_civic` / `v1`

**Product:** DOGEstonia — Module 1 (GPT instruction overlay)  
**Purpose:** Domain admission checklists, label-enum guidance, and completeness vs pack `field_policy` for the civic instance. **Not** HTTP Actions schemas. **Not** executable validate.

| Field | Value |
|-------|--------|
| **Version** | 1.1 |
| **Date** | 2026-08-31 |
| **Traceability** | GPT-SSR-01 / GIM-250 / GIM-254; REQ3-GPT-002; SEC-001 |
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

Validate candidate labels against [`story-label-taxonomy.md`](../../../story-label-taxonomy.md) (include from the data-model pack). Every value destined for `canonical_payload.labels[]` must be a canonical allowed key with story evidence.

- Unknown free-text, metadata-only, internal-only safety/privacy, and low-confidence hypotheses must **not** enter canonical labels.
- Keep them in validation notes as `metadata_only`, `needs_clarification`, or `rejected`.
- Multi-axis: do not collapse a civic narrative onto a single axis when evidence supports more than one (REQ-33 / REQ-36 / REQ-38 rules remain in core normalizer; this pack supplies the civic domain lists).

## 3. Completeness vs pack required fields

Before stash, this civic instance requires structured `signals.civic_domain` and `signals.failure_pattern` per gateway `field_policy` / `payload.schema.json` **when** the pack structured payload is being filled.

Core still requires logical Issue §4.1 `type`, `labels`, `title`, `description` for interview completeness. Pack required `signals.*` are **additional** for this instance — not a replacement for narrative-first intake (REQ3-PR-004).

Do **not** encode HTTP Action schemas here.

## 4. No leading taxonomy questions (privacy baseline)

Do not ask the resident to pick labels or confirm taxonomy keys mid-interview. Capture evidence in phases; map to pack axes after Phase 7 affirmation. Same privacy baseline as REQ-35 §4.3 / interview-flow §7.3–§7.4.
