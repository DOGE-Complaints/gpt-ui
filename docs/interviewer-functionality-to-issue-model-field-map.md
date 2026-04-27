# Interviewer functionality → GPT Issue model field map

**Date:** 2026-04-25  
**Scope:** DOGEstonia Module 1, Custom GPT interviewer / Issue ingest path.  
**Purpose:** one table that links interviewer functionality, implementation instructions, and the target GPT logical Issue model / handoff artifacts.

This document maps **data actually collected or produced by the current instruction chain**. It does not introduce new product requirements. Future expansion ideas belong to `docs/analysis/issue-model-layer-completeness-growth-recommendations.md`.

**Canonical location:** this mapping lives in `GPT UI/docs/interviewer-functionality-to-issue-model-field-map.md`. Analysis docs may link here, but should not create a parallel mapping SSOT in `GPT UI/docs/analysis/`.

---

## Legend

| Marker | Layer | Meaning |
|---|---|---|
| 🔵 | technical | Session, routing, artifacts, API/handoff metadata, validation mechanics. |
| 🟣 | emotional | User meaning, emotional signal, value/deep-need layer. |
| 🟢 | practical | Facts, logistics, location, time, desired change, concrete issue content. |
| 🟡 | social | Affected people/groups, civic relevance, systemic/collective signal. |
| 🟠 | administrative | Classification, policy, safety, privacy, identity/provenance, governance. |

Decision codes:

- `map` — can map into a current logical Issue/handoff field.
- `non_wire_metadata` — retained only in GPT/session/report artifacts.
- `drop` — intentionally not retained.
- `requires_api_change` — needs a separate backlog/API schema decision.

---

## Target model boundary

The target **GPT Issue model** is not only `canonical_payload`. It includes:

| Envelope | Purpose |
|---|---|
| `normalized_issue_payload.canonical_payload` | Logical Issue card fields from `story-data-model.md` §4.1–§4.3 (after GIM-72, no minors legacy fields). |
| `normalized_issue_payload.normalization_metadata` | Session language and references to upstream artifacts. |
| `ingest_validation_report` | Missing data, ambiguity, conflict, readiness, and stop-the-line state. |
| `safety_compliance_report` | Safety/PII decisions and redactions. |
| `deep_parsing_artifact` | Non-dialogue extraction hints before validation. |
| `comm_context` | Bootstrap communication context used by interview and i18n policy. |

Runtime `StoryIntakeRequest` mapping is a downstream bridge, not the same thing as the GPT logical Issue model.

---

## 1) 🔵 Technical / session layer

| layer_code | interviewer_function | collected_data | instruction_source | target_gpt_issue_model_field | runtime_story_intake_target | decision | transformation_rule | verification |
|---|---|---|---|---|---|---|---|---|
| 🔵 technical | Detect or accept interface language | `ui_lang` (`ru` / `et` / `en` / `unknown` until resolved) | [`bootstrap.md`](../instructions/bootstrap.md) §Bootstrap Algorithm; [`story-i18n-policy.md`](../instructions/story-i18n-policy.md) §1 | `comm_context.ui_lang`; `normalization_metadata.session_language` | `narrative.language` | `map` | Use explicit `/lang` first; otherwise language detection; map to session language and primary i18n slot. | Check `bootstrap.md`, `story-i18n-policy.md`, and OpenAPI `Narrative.language`. |
| 🔵 technical | Choose answer style | `tone_preset` | [`bootstrap.md`](../instructions/bootstrap.md) §Step 2 | `comm_context.tone_preset` | none | `non_wire_metadata` | Affects conversation tone only; do not publish into Issue card. | Confirm no OpenAPI target and only `comm_context`/conversation behavior uses it. |
| 🔵 technical | Choose answer length | `verbosity_level` | [`bootstrap.md`](../instructions/bootstrap.md) §Step 3; [`story-interview-flow.md`](../instructions/story-interview-flow.md) §7.2 | `comm_context.verbosity_level` | none | `non_wire_metadata` | Controls Phase 7 summary length; not content payload. | Confirm Phase 7 wording only; no `StoryIntakeRequest` field. |
| 🔵 technical | Choose diagnostic visibility | `transparency_mode` | [`bootstrap.md`](../instructions/bootstrap.md) §Step 4 | `comm_context.transparency_mode` | none | `non_wire_metadata` | Controls artifact visibility; not Issue content. | Confirm no mapping to `canonical_payload` or OpenAPI. |
| 🔵 technical | Mark bootstrap completion | `bootstrap_completed` | [`bootstrap.md`](../instructions/bootstrap.md) §Finalize | `comm_context.bootstrap_completed` | none | `non_wire_metadata` | Prevents repeated bootstrap questions. | Confirm bootstrap state stays session-local. |
| 🔵 technical | Preserve source type for non-dialogue input | `source_type` (`screenshot` / `pdf` / `link` / `text` / `mixed`) | [`ingest-deep-parsing.md`](../instructions/ingest-deep-parsing.md) §7; [`ingest-validation.md`](../instructions/ingest-validation.md) §2.1 | `deep_parsing_artifact.metadata.source_type` | `origin.source` (if product maps it) | `map` | For non-dialogue ingest, source type is artifact metadata; may also feed origin source when API handoff supports it. | Check `deep_parsing_artifact.metadata.source_type` and OpenAPI `Origin.source`. |
| 🔵 technical | Track input sources | `sources[]` | [`ingest-deep-parsing.md`](../instructions/ingest-deep-parsing.md) §7 | `deep_parsing_artifact.metadata.sources` | none in current OpenAPI | `non_wire_metadata` | Keep source references for validation/debug; do not invent runtime field. | Confirm `sources[]` exists only in deep parsing artifact. |
| 🔵 technical | Track parsing confidence | `confidence_scores` | [`ingest-deep-parsing.md`](../instructions/ingest-deep-parsing.md) §4, §7 | `deep_parsing_artifact.metadata.confidence_scores`; `ingest_validation_report` | `live_story_context.consistency_notes` (summary only) | `map` | Low confidence drives clarification; summarize only stable notes into `consistency_notes`. | Check confidence rules and OpenAPI `LiveStoryContext.consistency_notes`. |
| 🔵 technical | Track missing required fields | `missing_required_fields` | [`ingest-validation.md`](../instructions/ingest-validation.md) Issue overlay; [`ingest-deep-parsing.md`](../instructions/ingest-deep-parsing.md) §7 | `ingest_validation_report.missing_required_fields[]` | `live_story_context.consistency_notes` (summary only) | `map` | Use for stop-the-line / clarification; do not publish as card text. | Check validation report fields and `consistency_notes` boundary. |
| 🔵 technical | Track update-vs-new intent | `update_vs_new` (`new` / `update` / `ambiguous`) | [`ingest-validation.md`](../instructions/ingest-validation.md) §8; `interview-data-capture-catalog.md` §7 | `ingest_validation_report.update_vs_new` | none in current OpenAPI | `non_wire_metadata` | Affects routing; current runtime intake target is absent. | Confirm OpenAPI has no `update_vs_new` field. |
| 🔵 technical | Carry schema version | Constant `m2.story_intake_envelope.v1` | Runtime OpenAPI `StoryIntakeRequest`; [`REQ-18`](requirements/REQ-18-api-inbound-story-intake-and-gpt-handoff.md) §5 | not interview-collected; handoff constant | `schema_version` | `map` | Constant set by orchestrator/bridge, not asked from user. | Check OpenAPI enum `m2.story_intake_envelope.v1`. |

---

## 2) 🟢 Practical / factual layer

| layer_code | interviewer_function | collected_data | instruction_source | target_gpt_issue_model_field | runtime_story_intake_target | decision | transformation_rule | verification |
|---|---|---|---|---|---|---|---|---|
| 🟢 practical | Open topic gently | soft entry / opening context | [`story-interview-flow.md`](../instructions/story-interview-flow.md) §6–§7 | Draft material for `canonical_payload.description.{primary}` | `narrative.original_text` | `map` | Preserve as part of the user story if substantive; otherwise use only as conversation context. | Check interview phases and `Narrative.original_text` required field. |
| 🟢 practical | Capture event or pattern | what happened / keeps happening | [`story-interview-flow.md`](../instructions/story-interview-flow.md) §5 Q1, §7 Phase 2; [`ingest-validation.md`](../instructions/ingest-validation.md) Issue overlay | `canonical_payload.description.{et,ru,en}`; may influence `type` | `narrative.original_text` | `map` | User-confirmed facts become primary narrative text and card description content. | Check §5 Q1, §7 Phase 2, and OpenAPI `original_text`. |
| 🟢 practical | Capture location | where it happens | [`story-interview-flow.md`](../instructions/story-interview-flow.md) §5 Q2; [`ingest-validation.md`](../instructions/ingest-validation.md) readiness | `canonical_payload.description.{primary}`; validation note `issue_narrative:location` | `narrative.location_query` | `map` | Exact location → string; vague/missing/conflicting location → clarification or null/consistency note per GIM-76. | Check `location-query-mapping-policy.md` and OpenAPI `location_query`. |
| 🟢 practical | Capture temporal context | when / approximate period when relevant | [`story-interview-flow.md`](../instructions/story-interview-flow.md) §7 Phase 2 criteria; `interview-data-capture-catalog.md` §3 | `canonical_payload.description.{primary}`; `ingest_validation_report` note | no current `StoryIntakeRequest` time field | `non_wire_metadata` | Include in narrative text; if future structured time is required, backlog/API decision needed. | Confirm OpenAPI has no structured time field. |
| 🟢 practical | Capture concrete affected object/service | service, place, digital process, object, condition | [`story-interview-flow.md`](../instructions/story-interview-flow.md) §5 Q1–Q3; [`ingest-deep-parsing.md`](../instructions/ingest-deep-parsing.md) §3 | `canonical_payload.description`; label candidates | `narrative.original_text`; `live_story_context.consistency_notes` | `map` | Facts remain in original text; stable shorthand can support labels/type. | Check interview completeness Q1-Q3 and `consistency_notes`. |
| 🟢 practical | Capture desired change | desired state / expected improvement | [`story-interview-flow.md`](../instructions/story-interview-flow.md) §5 Q6, §7 Phase 5 | `canonical_payload.description`; optional `summary`; may shape `title` | `narrative.original_text`; `narrative.title_hint` if concise | `map` | Include after user confirmation; do not turn into promise of government action. | Check §5 Q6 and trilingual/runtime bridge policy. |
| 🟢 practical | Capture one-off vs recurring signal | recurring/systemic vs isolated | [`story-interview-flow.md`](../instructions/story-interview-flow.md) §5 Q7, §7 Phase 6 | `canonical_payload.description`; label/type rationale | `live_story_context.consistency_notes` | `map` | Store as civic/systemicity note; only confirmed evidence should influence classification. | Check §5 Q7 and `LiveStoryContext.consistency_notes`. |
| 🟢 practical | Extract non-dialogue title hint | parsed title from text/PDF/link/screenshot | [`ingest-deep-parsing.md`](../instructions/ingest-deep-parsing.md) §7 | `deep_parsing_artifact.extracted_data.title`; later `canonical_payload.title` after validation | `narrative.title_hint` after validation | `map` | Parsed title is a hint; final title follows interview/validation and i18n policy. | Check deep parsing output and OpenAPI `title_hint`. |
| 🟢 practical | Extract non-dialogue description hint | parsed description/body blocks | [`ingest-deep-parsing.md`](../instructions/ingest-deep-parsing.md) §3, §7 | `deep_parsing_artifact.extracted_data.description`; later `canonical_payload.description` | `narrative.original_text` after validation | `map` | Preserve user/source facts; unresolved ambiguity stays in artifact metadata. | Check deep parsing output and required `original_text`. |
| 🟢 practical | Extract non-dialogue summary hint | parsed summary | [`ingest-deep-parsing.md`](../instructions/ingest-deep-parsing.md) §7 | `deep_parsing_artifact.extracted_data.summary`; optional `canonical_payload.summary` | none direct | `map` | Optional card summary only after validation; not a runtime intake field. | Confirm `summary` is logical-only and not in OpenAPI intake. |

---

## 3) 🟣 Emotional / meaning layer

| layer_code | interviewer_function | collected_data | instruction_source | target_gpt_issue_model_field | runtime_story_intake_target | decision | transformation_rule | verification |
|---|---|---|---|---|---|---|---|---|
| 🟣 emotional | Identify user importance | why it matters to the user | [`story-interview-flow.md`](../instructions/story-interview-flow.md) §5 Q4, §7 Phase 3 | `canonical_payload.description`; optional `summary` | `narrative.original_text`; `live_story_context.consistency_notes` | `map` | Include only user-confirmed meaning; no therapy framing. | Check §5 Q4 and Phase 7 confirmation rules. |
| 🟣 emotional | Capture emotional signal | emotional framing / “what stung” | [`story-interview-flow.md`](../instructions/story-interview-flow.md) §4, §7.1, §9 | `canonical_payload.description`; validation notes | `narrative.original_text`; `live_story_context.consistency_notes` | `map` | Useful for fidelity; do not exaggerate or invent affect. | Check quality bar and no-fabrication/anti-pattern rules. |
| 🟣 emotional | Surface deep unmet need | deep need / value / dignity / predictability / safety | [`story-interview-flow.md`](../instructions/story-interview-flow.md) §4–§5, §7.1 | `canonical_payload.description`; label rationale; `normalization_metadata.trace_notes` | `live_story_context.consistency_notes` | `map` | Must be offered as hypothesis and confirmed or marked uncertain. | Check reframe templates and `trace_notes` boundary. |
| 🟣 emotional | Confirm or correct interpretation | user correction of meaning / gist / emphasis | [`story-interview-flow.md`](../instructions/story-interview-flow.md) §7.2 | Final `canonical_payload.title`, `description`, `summary`; `ingest_validation_report` | `narrative.original_text`; `live_story_context.consistency_notes` | `map` | User correction overrides GPT framing before handoff. | Check Phase 7 correction loop and runtime bridge policy. |
| 🟣 emotional | Stop deep probing when unsafe | limited-depth switch | [`story-interview-flow.md`](../instructions/story-interview-flow.md) §12; [`safety-compliance.md`](../instructions/safety-compliance.md) Issue overlay | `safety_compliance_report`; `ingest_validation_report.stop_the_line` | `privacy.*`; `live_story_context.consistency_notes` | `map` | Safety may leave narrative incomplete; record block/limited-depth reason. | Check safety checkpoints and privacy/live context fields. |

---

## 4) 🟡 Social / civic layer

| layer_code | interviewer_function | collected_data | instruction_source | target_gpt_issue_model_field | runtime_story_intake_target | decision | transformation_rule | verification |
|---|---|---|---|---|---|---|---|---|
| 🟡 social | Capture affected parties | who/what is affected | [`story-interview-flow.md`](../instructions/story-interview-flow.md) §5 Q3 | `canonical_payload.description`; labels if stable | `narrative.original_text`; `live_story_context.consistency_notes` | `map` | Preserve user wording; avoid unverified identity claims. | Check §5 Q3 and safety/PII constraints. |
| 🟡 social | Capture civic relevance | public cost / not-only-me signal | [`story-interview-flow.md`](../instructions/story-interview-flow.md) §4, §7 Phase 6, §9 | `canonical_payload.description`; `labels` rationale | `live_story_context.consistency_notes` | `map` | Use only evidence from user story; no promise of official action. | Check civic generalization phase and anti-false-promise rule. |
| 🟡 social | Capture systemicity | pattern, recurrence, structural process issue | [`story-interview-flow.md`](../instructions/story-interview-flow.md) §5 Q7; [`ingest-deep-parsing.md`](../instructions/ingest-deep-parsing.md) archetype hints | `canonical_payload.type`; `labels`; `description` | `live_story_context.consistency_notes` | `map` | May support `system_bug` / `absurdity` / `complaint` classification after confirmation. | Check `ISSUE_TYPE` enum and Phase 7 confirmation. |
| 🟡 social | Handle latent request | hidden wish / vague dissatisfaction / “would be nice if…” | [`story-interview-flow.md`](../instructions/story-interview-flow.md) §12 | `canonical_payload.description`; possible `type=observation` after validation | `narrative.original_text`; `title_hint` if stable | `map` | Do not demand complaint framing; classify only after enough substance. | Check latent request guidance and runtime string bridge. |
| 🟡 social | Surface topic | plain-language subject theme | [`ingest-deep-parsing.md`](../instructions/ingest-deep-parsing.md) §7; `interview-data-capture-catalog.md` §6 | `normalization_metadata.trace_notes`; possible `labels` | `live_story_context.consistency_notes` | `map` | A hint, not a final taxonomy. | Check deep parsing notes and label validation. |

---

## 5) 🟠 Administrative / classification / safety layer

| layer_code | interviewer_function | collected_data | instruction_source | target_gpt_issue_model_field | runtime_story_intake_target | decision | transformation_rule | verification |
|---|---|---|---|---|---|---|---|---|
| 🟠 administrative | Classify issue archetype | provisional/final `type` | [`story-interview-flow.md`](../instructions/story-interview-flow.md) §8; [`ingest-deep-parsing.md`](../instructions/ingest-deep-parsing.md) overlay; [`story-data-model.md`](../instructions/story-data-model.md) §4.1 | `canonical_payload.type` | none direct | `map` | Final enum only after maturity and confirmation. Allowed values: `complaint`, `observation`, `absurdity`, `system_bug`. | Check `story-data-model.md` enum and no OpenAPI intake field. |
| 🟠 administrative | Assign labels | `labels_hints` / final labels | [`ingest-deep-parsing.md`](../instructions/ingest-deep-parsing.md) §7; [`ingest-validation.md`](../instructions/ingest-validation.md) §6; [`story-data-model.md`](../instructions/story-data-model.md) §4.1 | `canonical_payload.labels` | none direct | `map` | Hints become final labels only after validation; no invented vocabulary. | Check label validation and vocabulary rules. |
| 🟠 administrative | Keep institution hypothesis separate | `institution_hypothesis` | [`ingest-deep-parsing.md`](../instructions/ingest-deep-parsing.md) overlay; [`story-interview-flow.md`](../instructions/story-interview-flow.md) §8 / REQ-16 Q5; [`story-data-model.md`](../instructions/story-data-model.md) §4.2 | `normalization_metadata.trace_notes`; **not** `canonical_payload.institution` in demo | none | `non_wire_metadata` | Demo default: omit institution from canonical payload unless product lifts restriction. | Check REQ-16 Q5 demo rule and `story-data-model.md` §4.2. |
| 🟠 administrative | Validate policy/safety admission | `policy_gate_result`, `safety_compliance_report` refs | [`safety-compliance.md`](../instructions/safety-compliance.md) checkpoints; [`story-policy-gate.md`](../instructions/story-policy-gate.md) | `normalization_metadata.safety_compliance_report_ref`; `policy_gate_ref` | `privacy.*`; `live_story_context.consistency_notes` | `map` | Gate/safety are required artifacts before normalization/API. | Check normalizer metadata refs and OpenAPI privacy/live context. |
| 🟠 administrative | Detect PII | `pii_detected[]`, redactions | [`safety-compliance.md`](../instructions/safety-compliance.md) §1.1; [`ingest-deep-parsing.md`](../instructions/ingest-deep-parsing.md) §5, §7 | `safety_compliance_report.pii_detected[]`; `redactions[]` | `privacy.contains_pii`; `privacy.redaction_requested` | `map` | Redact/remove before proceeding; do not store raw PII beyond user-provided source. | Check PII rules and OpenAPI `Privacy` schema. |
| 🟠 administrative | Track safety block/redact/allow | `decision`, `stop_the_line` | [`safety-compliance.md`](../instructions/safety-compliance.md) activation points | `safety_compliance_report.decision`; `stop_the_line` | `live_story_context.consistency_notes` if allowed | `map` | Blocks halt the chain; allow proceeds with report reference. | Check safety decision paths and `consistency_notes` boundary. |
| 🟠 administrative | Capture user identity linkage | `external_user_id`, `identity_issuer` | [`REQ-19`](requirements/REQ-19-security-auth-boundaries-and-user-identity-flow.md) §3; API orchestrator boundary | not interview content; handoff/auth context | `submitter.external_user_id`; `submitter.identity_issuer` | `map` | Comes from auth/session/demo redirect; never inferred from message text or bearer secret. | Check REQ-19 and OpenAPI `Submitter.required`. |
| 🟠 administrative | Capture origin/provenance | `origin.source`, `conversation_id`, `tool_call_id` | Runtime OpenAPI; `REQ-18` handoff docs | `normalization_metadata.trace_notes` / orchestrator context | `origin.source`; `origin.conversation_id`; `origin.tool_call_id` | `map` | Platform context only; user should not be asked to provide these. | Check OpenAPI `Origin` schema and no interview question requirement. |
| 🟠 administrative | Track privacy choices | user redaction request / PII handling | [`safety-compliance.md`](../instructions/safety-compliance.md); `REQ-18` | `safety_compliance_report`; validation notes | `privacy.contains_pii`; `privacy.redaction_requested` | `map` | Based on safety detection or explicit user request. | Check safety PII rules and `Privacy` defaults. |
| 🟠 administrative | Track ambiguity/conflict | `ambiguities[]`, `conflicts[]` | [`ingest-validation.md`](../instructions/ingest-validation.md); [`ingest-deep-parsing.md`](../instructions/ingest-deep-parsing.md) §7 | `ingest_validation_report.ambiguities[]`; `conflicts[]` | `live_story_context.consistency_notes` | `map` | Must be resolved or accepted as residual uncertainty before final handoff. | Check validation/deep parsing artifacts and `LiveStoryContext`. |

---

## 6) I18n and runtime string bridge

| layer_code | interviewer_function | collected_data | instruction_source | target_gpt_issue_model_field | runtime_story_intake_target | decision | transformation_rule | verification |
|---|---|---|---|---|---|---|---|---|
| 🔵 technical | Select primary slot | primary language / `session_language` | [`story-i18n-policy.md`](../instructions/story-i18n-policy.md) §1, §6; [`bootstrap.md`](../instructions/bootstrap.md) | `normalization_metadata.session_language`; primary slot in `title/description/summary` | `narrative.language` | `map` | Use `ui_lang` or detected substantive language; do not infer from longest string downstream. | Check `session_language` policy and OpenAPI `Narrative.language`. |
| 🟢 practical | Store user-approved text | primary-language title/description/summary | [`story-interview-flow.md`](../instructions/story-interview-flow.md) §7.2; [`story-i18n-policy.md`](../instructions/story-i18n-policy.md) §2–§4 | `canonical_payload.title/description/summary.{primary}` | `narrative.original_text`; `narrative.title_hint` | `map` | Primary slot is richest and semantically faithful; runtime gets single strings. | Check Phase 7 confirmation and `trilingual-to-runtime-string-policy.md`. |
| 🔵 technical | Track secondary language variants | translations or explicit fallback placeholders | [`story-i18n-policy.md`](../instructions/story-i18n-policy.md) §2–§4 | `canonical_payload.title/description/summary.{secondary}`; validation notes | none direct in current OpenAPI | `non_wire_metadata` | Do not promise `{et,ru,en}` objects in runtime string fields. | Confirm OpenAPI string fields and secondary non-wire policy. |

---

## 7) Fields currently excluded or decision-only

| layer_code | data / field | source | target_gpt_issue_model_field | runtime_story_intake_target | decision | reason | verification |
|---|---|---|---|---|---|---|---|
| 🟠 administrative | `age_group` | Historical GIM-65 decision, superseded by GIM-72 | none after GIM-72 | none | `drop` | Legacy minors field; not part of DOGEstonia active Issue model. | `rg "age_group" "GPT UI/instructions"` returns no active hits. |
| 🟠 administrative | `parental_accompaniment` | Historical GIM-65 decision, superseded by GIM-72 | none after GIM-72 | none | `drop` | Legacy minors field; safety may discuss minors as risk topic without this canonical key. | `rg "parental_accompaniment" "GPT UI/instructions"` returns no active hits. |
| 🟣 emotional | `severity` | `story-data-model.md` §4.3; `spa-app/src/domain/types.js`; `subjective-fields-wire-disposition.md` | non-wire artifact / logical sidecar | none | `non_wire_metadata` | GIM-77 decision: keep as resident-perceived metadata only for current runtime. | Confirm no OpenAPI target and non-wire decision doc. |
| 🟡 social | `impact_estimation` | `story-data-model.md` §4.3; `spa-app/src/domain/types.js`; `subjective-fields-wire-disposition.md` | non-wire artifact / logical sidecar | none | `non_wire_metadata` | GIM-77 decision: keep as resident-perceived metadata only for current runtime. | Confirm no OpenAPI target and non-wire decision doc. |
| 🟢 practical | `problem_status` | `story-data-model.md` §4.3; `spa-app/src/domain/types.js`; `subjective-fields-wire-disposition.md` | non-wire artifact / logical sidecar | none | `non_wire_metadata` | GIM-77 decision: keep as resident-perceived metadata only for current runtime. | Confirm no OpenAPI target and non-wire decision doc. |
| 🔵 technical | `id`, `status`, `created_at`, `arweave_txid`, `image_txid`, `image_hash` | `story-data-model.md` §4.4 | none from GPT facts | API response/backend only | `drop` | GPT must not fabricate backend-issued values. | Check `story-data-model.md` §4.4 system-only block. |

---

## 8) Verification checklist

Use this file when executing GIM-73 and related tasks:

- [ ] Every row above has an explicit `decision`.
- [ ] Every row above has a per-row `verification` cell.
- [ ] Every `map` row points to a current GPT issue-model field, report artifact, or current OpenAPI field.
- [ ] Every `requires_api_change` row has a backlog pointer before closure.
- [ ] No row maps removed minors fields into `canonical_payload`.
- [ ] `GIM-73` acceptance references this file and the analysis baseline `docs/analysis/gim72-77-issue-model-target-state-and-acceptance.md`.
