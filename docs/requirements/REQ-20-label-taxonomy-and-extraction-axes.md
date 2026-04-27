# REQ-20: GPT label taxonomy and extraction axes

> **Назначение:** зафиксировать requirements-level модель того, как Custom GPT должен извлекать `labels` из истории жителя, используя не одну плоскую тему, а несколько осей анализа: практическую, системную, социальную, эмоционально-смысловую, административную и safety/privacy.
>
> **Статус:** active draft. Это требования к GPT instruction layer и logical Issue draft, **не transport/API schema**.

**Версия:** 0.1 · 2026-04-26  
**Связанные документы:** [REQ-09](./REQ-09-functional-requirements.md), [REQ-10](./REQ-10-output-content-model.md), [REQ-12](./REQ-12-anti-patterns.md), [REQ-15](./REQ-15-working-assumptions.md), [REQ-16](./REQ-16-open-decisions.md), [REQ-18](./REQ-18-api-inbound-story-intake-and-gpt-handoff.md), [`story-data-model.md`](../../instructions/story-data-model.md), [`story-interview-flow.md`](../../instructions/story-interview-flow.md), [`story-normalizer.md`](../../instructions/story-normalizer.md).

---

## 1. Decision

GPT **should** use its reasoning capacity to derive rich label candidates from the resident story across multiple axes.

However:

- GPT must not ask the user to choose labels upfront (REQ-09 FR-M1-007).
- GPT must not lock `type` / `labels` too early (REQ-12 anti-pattern #2).
- GPT must not treat labels as backend truth, publication status, official routing, or institution acceptance.
- GPT must keep `labels` as logical Issue / draft metadata until the relevant transport/API schema explicitly accepts them.

For the current GPT logical model, `labels` remain a required logical Issue field after interview maturity and validation (`story-data-model.md` §4.1), but the label extraction process should be multi-axis rather than only "topic matching".

---

## 2. Why multi-axis labels are required

Existing requirements already imply multiple label sources:

| Source | Existing requirement | Label implication |
|---|---|---|
| REQ-09 §9.5 | GPT extracts `raw_story`, `location_hint`, `affected_subject`, `emotional_signal`, `deep_need`, `desired_state`, `public_relevance`, `repeatability_hint`. | Labels can come from facts, affected parties, meaning, desired state, civic relevance, and recurrence. |
| REQ-09 §9.6 | GPT distinguishes `complaint`, `observation`, `absurdity`, `system_bug`; proposes labels from agreed vocabulary. | Labels must support classification but not collapse into `type`. |
| REQ-10 §10.3 | Civic framing includes public significance, repeatability, private vs stable pattern. | Labels should capture systemic/civic signal, not only object/topic. |
| REQ-10 §10.5 | Issue Projection includes probable `type` and preliminary `labels`. | Labels are part of Issue projection, but should carry confidence/provenance. |
| `story-interview-flow.md` §4–§7 | Interview moves from episode to emotion, deep need, desired state, civic generalization. | Label extraction should happen after narrative maturity, using the full story. |

---

## 3. Label extraction axes

GPT should derive label candidates across the following axes. A final `canonical_payload.labels[]` may contain only approved label keys, but the reasoning/report layer should preserve which axes produced each candidate.

| Axis | Purpose | Source in story | Example candidate labels / families | Include when |
|---|---|---|---|---|
| `topic_domain` | What domain the issue is about. | Surface topic, object/service/place. | `transport`, `roads`, `waste`, `housing`, `education`, `healthcare`, `digital_service`, `public_space`, `environment`, `safety`. | The story clearly names a domain or object. |
| `service_object` | Which concrete thing/process is failing or desired. | Service, institution-facing process, physical object, digital flow. | `parking`, `street_lighting`, `sidewalk`, `bus_stop`, `queue`, `portal_login`, `application_form`, `maintenance`, `communication`. | The story contains a stable object/process, even if agency is unknown. |
| `location_context` | What kind of place/context is relevant. | Location answer, place type, physical vs digital context. | `school_area`, `residential_area`, `city_center`, `online_portal`, `public_transport_node`, `park`, `crossing`, `neighborhood`. | Place type matters for interpretation or clustering. |
| `failure_mode` | What kind of failure pattern is described. | What happened / keeps happening. | `delay`, `access_blocked`, `information_gap`, `broken_infrastructure`, `unsafe_condition`, `bureaucratic_loop`, `unclear_rules`, `service_unavailable`, `maintenance_gap`. | The resident describes a concrete malfunction, barrier, or recurring friction. |
| `issue_archetype_support` | Evidence supporting `type`, without duplicating `type`. | Absurdity, system loop, harm, improvement wish. | `process_absurdity`, `system_loop`, `harm_reported`, `improvement_wish`, `positive_observation`, `bug_like_flow`. | The label explains why a `type` was selected. |
| `affected_scope` | Who/what is affected. | Affected person/group/object. | `pedestrians`, `parents`, `children_context`, `elderly_context`, `drivers`, `residents`, `visitors`, `small_business`, `public_users`. | The affected group is stated or directly inferable from the story. |
| `civic_signal` | Why it matters beyond an isolated event. | Public relevance, recurrence, collective relevance. | `recurring_issue`, `systemic_pattern`, `public_cost`, `not_only_me`, `equity_access`, `trust_in_services`, `city_for_people`. | User gives evidence of recurrence, public impact, or civic meaning. |
| `deep_need` | Which human/civic need is visible. | Emotional signal, reframe confirmation, deep need. | `predictability`, `being_heard`, `respect`, `safety_need`, `dignity`, `time_not_wasted`, `agency`, `fairness`. | The need is user-confirmed or clearly marked as GPT hypothesis in metadata. |
| `desired_outcome` | What better state the resident wants. | Desired state / improvement image. | `clear_rules`, `faster_response`, `safer_space`, `better_maintenance`, `accessible_service`, `transparent_process`, `human_contact`, `digital_fix`. | Phase 5 desired state is articulated or accepted as a rough goal. |
| `risk_privacy_safety` | Internal routing / gate signals. | Safety, PII, minors/sensitive topics. | `pii_present`, `redaction_needed`, `limited_depth`, `minor_context`, `health_context`, `violence_context`. | Use as non-public/internal labels only if policy allows; never use to expose sensitive user data. |
| `confidence_state` | Shows label maturity, not topic. | Validation report, uncertainty, user correction. | `confirmed_by_user`, `gpt_hypothesis`, `needs_clarification`, `low_confidence`, `conflict_unresolved`. | Useful in metadata/report; not necessarily public Issue labels. |

---

## 4. Functional requirements

- **FR-M1-044.** When the interview reaches narrative maturity, GPT shall derive label candidates across multiple axes, not only from the surface topic.
- **FR-M1-045.** When proposing final `canonical_payload.labels[]`, GPT shall include only label keys that are supported by the user story, validation notes, or confirmed Phase 7 framing.
- **FR-M1-046.** Where a label is inferred from meaning, deep need, or civic relevance, GPT shall preserve the source/provenance in validation or normalization metadata.
- **FR-M1-047.** Where a label is uncertain, GPT shall mark it as `gpt_hypothesis`, `low_confidence`, or `needs_clarification` instead of presenting it as a confirmed fact.
- **FR-M1-048.** GPT shall not ask the resident to choose labels during early interview phases; label assignment is an internal projection after sufficient story understanding.
- **FR-M1-049.** GPT shall separate `type` from `labels`: `type` is the coarse issue archetype, while labels are multi-axis descriptors.
- **FR-M1-050.** GPT shall not use labels to imply official routing, institution responsibility, publication state, or backend acceptance.
- **FR-M1-051.** GPT shall keep safety/privacy labels internal unless product policy explicitly allows public exposure.

---

## 5. Output shape requirement

At requirements level, GPT should conceptually produce two layers:

```json
{
  "canonical_payload": {
    "type": "complaint",
    "labels": ["transport", "broken_infrastructure", "safety", "recurring_issue"]
  },
  "label_extraction_metadata": {
    "candidates": [
      {
        "label": "transport",
        "axis": "topic_domain",
        "source": "resident story / Phase 2",
        "confidence": "high",
        "disposition": "canonical"
      },
      {
        "label": "predictability",
        "axis": "deep_need",
        "source": "Phase 4 reframe accepted by user",
        "confidence": "medium",
        "disposition": "metadata_only"
      }
    ]
  }
}
```

`label_extraction_metadata` is a requirements-level concept. It may be implemented later as `normalization_metadata`, `ingest_validation_report`, a logical sidecar, or another artifact. It is not automatically a current API field.

---

## 6. Label selection rules

1. Prefer labels supported by explicit user wording.
2. Use GPT inference for deeper labels only when the inference is grounded in interview phases and marked with confidence.
3. Do not generate more labels just because the model can; prefer a small set of high-signal labels.
4. Keep public/card labels distinct from internal safety/privacy/routing labels.
5. Do not use `institution` as a label substitute in demo scope; institution inference remains restricted by existing requirements.
6. If the label vocabulary is not yet approved, store candidate families in metadata and keep `canonical_payload.labels[]` limited to agreed keys.

---

## 7. Acceptance criteria

- Given a mature interview with facts, meaning, desired state, and civic relevance, when GPT creates an Issue draft, then it produces `type` and `labels` from multiple axes with traceable reasoning.
- Given a story with only a surface topic and no deeper signal, when GPT creates labels, then it uses only supported topic/service labels and marks deeper labels as missing or uncertain.
- Given a user correction in Phase 7, when GPT updates the Issue framing, then labels derived from the rejected framing are removed or downgraded to `needs_clarification`.
- Given safety/privacy-sensitive content, when GPT assigns internal routing labels, then those labels do not appear as public/card labels unless policy explicitly permits.
- Given a backend/API transport that does not accept labels, when GPT performs handoff, then labels remain in logical Issue / draft artifacts and are not silently injected into unrelated runtime fields.

---

## 8. Open decisions

| Decision | Why it remains open |
|---|---|
| Final controlled vocabulary for `canonical_payload.labels[]`. | Current requirements require labels but do not define a complete approved dictionary inside GPT docs. |
| Whether label axes become namespaced keys (`topic:transport`) or flat keys (`transport`). | Depends on SPA/backend vocabulary and product UX. |
| Whether `label_extraction_metadata` becomes a formal normalizer artifact. | Current `story-normalizer.md` has `normalization_metadata` and optional sidecars, but no formal label metadata schema yet. |
| Which safety/privacy labels can ever be visible publicly. | Requires policy/operator decision. |

---

## 9. Traceability

| Requirement | Trace |
|---|---|
| Multi-axis labels | REQ-09 §9.5, §9.6; REQ-10 §10.1–10.5 |
| Do not ask user for labels upfront | REQ-09 FR-M1-007 |
| Do not classify too early | REQ-12 anti-pattern #2; `story-interview-flow.md` §8 |
| Type vocabulary | REQ-09 FR-M1-024; `story-data-model.md` §4.1 |
| Labels in logical Issue model | REQ-09 FR-M1-026, FR-M1-037; REQ-10 §10.5; `story-data-model.md` §4.1 |
| No false backend/status claims | REQ-09 FR-M1-038; REQ-15 item 6 |

---

## 10. Version history

| Version | Date | Change |
|---|---|---|
| 0.1 | 2026-04-26 | Initial requirements-level label extraction axes for GPT Issue projection. |
