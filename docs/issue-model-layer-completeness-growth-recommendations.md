# Issue model layer completeness — growth recommendations

**Date:** 2026-04-25  
**Status:** recommendations only / not current acceptance requirements.  
**Base map:** [`../interviewer-functionality-to-issue-model-field-map.md`](../interviewer-functionality-to-issue-model-field-map.md).  
**Scope:** future-proofing the GPT Issue model after GIM-72...GIM-77 clarifies the current target state.

This document does **not** change current runtime OpenAPI or instruction requirements. It records expansion ideas for a more flexible universal model while keeping the current implementation honest.

---

## 1) Current baseline

Current instructions already collect or produce five broad layers:

| Marker | Layer | Current state |
|---|---|---|
| 🔵 | technical | Strong session/context and artifact discipline exists (`comm_context`, validation reports, deep parsing artifacts, safety reports), but not all metadata has a runtime target. |
| 🟣 | emotional | Interview flow intentionally captures meaning, emotional signal, and deep need, but most of it is embedded in prose rather than typed fields. |
| 🟢 | practical | Facts, location, event pattern, desired state, and concrete issue content are well covered in dialogue, but only `location_query` has a current dedicated runtime field. |
| 🟡 | social | Affected parties, civic relevance, recurrence/systemicity, and latent collective signal are captured, but mostly as narrative/notes rather than structured fields. |
| 🟠 | administrative | Type/labels, policy/safety, privacy, identity, and provenance boundaries are documented, but identity/origin/privacy are partially platform-owned rather than interview-owned. |

---

## 2) 🔵 Technical layer recommendations

### Current strengths

- `bootstrap.md` defines `comm_context` with `ui_lang`, `tone_preset`, `verbosity_level`, `transparency_mode`.
- `ingest-deep-parsing.md` defines structured non-dialogue artifacts with confidence, ambiguities, conflicts, PII, missing fields, and source type.
- Runtime `StoryIntakeRequest` has a clear envelope with `schema_version`, `submitter`, `narrative`, `origin`, `privacy`, `live_story_context`.

### Gaps to consider later

| Recommendation | Why it helps | Suggested future field / artifact |
|---|---|---|
| Add explicit `conversation_context` metadata envelope | Avoids overloading `live_story_context.consistency_notes` with everything. | `conversation_context.ui_lang`, `tone_preset`, `verbosity_level`, `source_mode` |
| Separate `extraction_confidence` from `interpretation_confidence` | OCR/link parsing uncertainty and GPT meaning uncertainty are different. | `quality.confidence.extraction`, `quality.confidence.interpretation` |
| Preserve `source_type` and `sources[]` consistently through handoff | Useful for audit and deduplication. | `origin.source_type`, `origin.sources[]` |
| Add structured `schema_trace` | Helps validate mapping after OpenAPI changes. | `schema_trace.input_schema_version`, `mapping_version`, `normalizer_version` |

### Caution

Do not promote all technical metadata into public Issue card fields. Most of it is internal traceability or API handoff support.

---

## 3) 🟣 Emotional layer recommendations

### Current strengths

- `story-interview-flow.md` requires movement beyond facts into meaning and deep need.
- Phase 7 forces user correction before final framing.
- Safety and limited-depth rules prevent pseudo-therapy.

### Gaps to consider later

| Recommendation | Why it helps | Suggested future field / artifact |
|---|---|---|
| Add typed `meaning_summary` | Keeps “why it matters” retrievable without parsing long description. | `structured_signals.meaning_summary` |
| Add `deep_need_signal` as controlled-but-flexible text | Helps clustering similar civic needs across different factual topics. | `structured_signals.deep_need_signal` |
| Add `emotional_tone` with conservative vocabulary | Helps quality review and routing, but must avoid diagnosis. | `structured_signals.emotional_tone` (`frustration`, `fear`, `exhaustion`, `hope`, etc.) |
| Add `user_confirmed_framing` flag | Makes Phase 7 readiness explicit. | `readiness.user_confirmed_framing: boolean` |

### Caution

Avoid clinical categories, psychological diagnosis, or over-specific affect labels. Emotional fields should support civic interpretation, not therapy.

---

## 4) 🟢 Practical layer recommendations

### Current strengths

- The seven completeness questions cover what happened, where, who is affected, why it matters, desired state, and recurrence.
- `narrative.location_query` exists in current runtime OpenAPI.
- `story-data-model.md` already supports title/description/summary as logical card fields.

### Gaps to consider later

| Recommendation | Why it helps | Suggested future field / artifact |
|---|---|---|
| Add structured `time_context` | Current OpenAPI lacks a dedicated time field; time is important for interpretation. | `structured_signals.time_context` (`exact`, `approximate`, `recurring`, text) |
| Split `location_query` into raw + interpreted | Keeps user wording while enabling later geocoding/search. | `location.raw_text`, `location.normalized_hint`, `location.confidence` |
| Add `desired_state` | Desired change is central to civic usefulness and currently lives in prose. | `structured_signals.desired_state` |
| Add `problem_pattern` | Distinguishes one-off, recurring, systemic, seasonal, process-loop. | `structured_signals.problem_pattern` |
| Add `practical_constraints` | Captures access, timing, cost, physical barriers, digital friction. | `structured_signals.practical_constraints[]` |

### Caution

Do not force structured practical fields during the early soft-entry phase. Populate them only after enough narrative maturity or explicit user confirmation.

---

## 5) 🟡 Social layer recommendations

### Current strengths

- Interview flow asks who/what is affected and whether the issue is one-off or collective.
- The model differentiates individual complaint from civic signal.
- Latent requests and improvement wishes are supported without forcing complaint framing.

### Gaps to consider later

| Recommendation | Why it helps | Suggested future field / artifact |
|---|---|---|
| Add `affected_scope` | Supports clustering and prioritization without requiring exact identities. | `structured_signals.affected_scope` (`self`, `household`, `neighbors`, `public`, `unknown`) |
| Add `collective_relevance_evidence` | Prevents vague claims that “many people are affected” without user evidence. | `structured_signals.collective_relevance_evidence` |
| Add `systemicity_signal` | Makes systemic vs isolated distinction explicit. | `structured_signals.systemicity_signal` (`one_off`, `recurring`, `systemic`, `unclear`) |
| Add `stakeholder_mentions` as non-authoritative | Useful for routing, but should not become institutional claims. | `non_wire_metadata.stakeholder_mentions[]` |

### Caution

Social fields must not create unverified claims about groups, institutions, or public impact. Use “user says / evidence in story” framing.

---

## 6) 🟠 Administrative layer recommendations

### Current strengths

- `ISSUE_TYPE` is constrained by SPA enum.
- Demo scope blocks `institution` inference.
- Safety and policy gate artifacts are separated from normalization.
- REQ-19 distinguishes user identity from bearer channel auth.

### Gaps to consider later

| Recommendation | Why it helps | Suggested future field / artifact |
|---|---|---|
| Add `classification_rationale` | Prevents opaque `type` / `labels` assignment. | `normalization_metadata.classification_rationale` |
| Add `label_confidence` | Makes labels auditable and reversible. | `normalization_metadata.label_confidence` |
| Add typed `policy_gate_summary` reference | Keeps policy decisions machine-readable without duplicating operator rulebook. | `policy_gate_ref.decision`, `policy_gate_ref.reasons[]` |
| Add `identity_state` | Clarifies anonymous vs logged-in vs demo identity. | `submitter.identity_state` or non-wire auth context |
| Add `institution_disposition` | Clarifies omitted vs unknown vs product-disabled institution. | `non_wire_metadata.institution_disposition` |

### Caution

Administrative fields are the highest risk for false authority. They must not imply official acceptance, government routing, or backend status without API response.

---

## 7) Universal model shape to consider later

A future flexible model could separate user truth, interpretation, system metadata, and publication/runtime state:

```json
{
  "story": {
    "user_testimony": {},
    "confirmed_interpretation": {},
    "structured_signals": {},
    "quality": {}
  },
  "issue_projection": {
    "type": "",
    "labels": [],
    "title": { "et": "", "ru": "", "en": "" },
    "summary": { "et": "", "ru": "", "en": "" },
    "description": { "et": "", "ru": "", "en": "" }
  },
  "governance": {
    "safety": {},
    "policy": {},
    "privacy": {},
    "provenance": {}
  },
  "runtime_handoff": {
    "story_intake": {},
    "issue_create": {}
  }
}
```

This is **not** the current target schema. It is a direction for later architecture if the product needs more reuse across UI, clustering, moderation, and API handoff.

---

## 8) Recommended next documentation steps

1. Keep `../interviewer-functionality-to-issue-model-field-map.md` as the current factual mapping.
2. Let GIM-73 create/own the concrete bridge matrix for current execution.
3. Let GIM-77 decide whether subjective fields remain non-wire or become API-change backlog.
4. If future schema expansion is approved, create a dedicated task/epic instead of silently adding fields to `canonical_payload`.
