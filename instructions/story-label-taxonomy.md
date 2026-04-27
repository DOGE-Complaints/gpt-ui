# Issue Label Taxonomy — controlled label axes

**Product:** DOGEstonia — Module 1 (GPT Interview / ingest → Issue)  
**Purpose:** Controlled GPT-side source of truth for deriving `canonical_payload.labels[]` and label reasoning metadata from resident stories.

| Document field | Value |
|----------------|--------|
| **Version** | 0.1 |
| **Date** | 2026-04-26 |
| **Traceability** | FR-M1-026/027/044…051; [`story-data-model.md`](story-data-model.md), [`story-interview-flow.md`](story-interview-flow.md), [`ingest-validation.md`](ingest-validation.md), [`story-normalizer.md`](story-normalizer.md) |

---

## 1. Role

This file defines how GPT may derive Issue labels from the story. It is an instruction-layer taxonomy, not a backend schema and not a public UI dictionary contract.

`canonical_payload.labels[]` must contain only controlled keys from the canonical allowed sections of this file and candidates with disposition `canonical`. All other label-like signals must stay in metadata, validation notes, or clarification state.

---

## 2. Non-negotiable rules

1. Do not ask the resident to choose labels upfront.
2. Do not lock `type` or `labels` before narrative maturity and Phase 7 confirmation.
3. Do not invent label keys.
4. Do not place internal/safety/privacy labels into public/card labels.
5. Do not use labels to imply official routing, institution responsibility, publication status, or backend acceptance.
6. Unknown or low-confidence labels must be omitted from `canonical_payload.labels[]` and recorded as `metadata_only`, `needs_clarification`, or `rejected`.

---

## 3. Axis model

| Axis | Role | Canonical disposition |
|---|---|---|
| `topic_domain` | High-level civic domain. | May produce canonical labels when supported. |
| `service_object` | Concrete service, object, or process. | May produce canonical labels when supported. |
| `location_context` | Place type or service context. | Usually metadata; canonical only if key is listed below. |
| `failure_mode` | What kind of friction or failure happened. | May produce canonical labels when supported. |
| `issue_archetype_support` | Evidence supporting `type`. | Metadata-only unless key is listed below. |
| `affected_scope` | Who or what is affected. | Usually metadata; canonical only if product-approved. |
| `civic_signal` | Recurrence, systemic relevance, public cost. | May produce canonical labels when supported. |
| `deep_need` | Human/civic need beneath the story. | Metadata-only by default. |
| `desired_outcome` | Better state requested by the resident. | Metadata-only by default. |
| `risk_privacy_safety` | Safety, PII, minors, sensitive topics. | Internal metadata only. |
| `confidence_state` | Label certainty and correction status. | Internal metadata only. |

---

## 4. Canonical allowed labels

Use these keys only when evidence is present in the story, validation report, or confirmed Phase 7 framing.

### 4.1 Topic domain

| Key | Meaning |
|---|---|
| `transport` | Public transport, stops, routes, mobility services. |
| `roads` | Roads, crossings, traffic flow, driving conditions. |
| `parking` | Parking availability, rules, or local parking friction. |
| `public_space` | Streets, squares, parks, playgrounds, shared outdoor space. |
| `waste` | Waste collection, bins, litter, recycling friction. |
| `environment` | Noise, air, greenery, water, or environmental quality. |
| `housing` | Residential buildings, yards, housing-adjacent services. |
| `education` | School, kindergarten, youth education context. |
| `healthcare` | Healthcare access or healthcare service friction. |
| `digital_service` | Online portal, login, form, or digital public-service flow. |
| `safety` | Non-sensitive public safety concern; do not use for private sensitive disclosures without policy review. |
| `accessibility` | Physical or digital accessibility barrier. |

### 4.2 Failure mode

| Key | Meaning |
|---|---|
| `delay` | Waiting, slow response, missed timing, or service lag. |
| `access_blocked` | Resident cannot access a place, service, route, or digital flow. |
| `information_gap` | Rules, status, instructions, or responsibilities are unclear. |
| `broken_infrastructure` | Physical infrastructure is damaged, absent, or not functioning. |
| `unsafe_condition` | A stated condition creates practical risk in public space or service use. |
| `bureaucratic_loop` | Resident is sent in circles or cannot complete a process. |
| `unclear_rules` | Requirements or rules are contradictory or hard to understand. |
| `service_unavailable` | Expected service is unavailable or repeatedly fails. |
| `maintenance_gap` | Upkeep, cleaning, repair, or routine maintenance is missing. |

### 4.3 Civic signal

| Key | Meaning |
|---|---|
| `recurring_issue` | The story indicates repetition over time. |
| `systemic_pattern` | The issue appears to reflect a broader system pattern. |
| `public_cost` | The problem creates cost beyond the individual episode. |
| `not_only_me` | The resident indicates others are affected too. |
| `equity_access` | Fair access or unequal burden is central to the story. |
| `trust_in_services` | The story concerns loss of trust in public-service functioning. |
| `city_for_people` | The desired state is a more human, usable, respectful city environment. |

### 4.4 Issue archetype support

| Key | Meaning |
|---|---|
| `process_absurdity` | The case supports `type = absurdity` because the process is visibly irrational. |
| `system_loop` | The case supports `type = system_bug` or `absurdity` because the resident is trapped in a loop. |
| `harm_reported` | The resident reports concrete harm or burden; do not downgrade to observation. |
| `improvement_wish` | The story is framed as a desired improvement. |
| `positive_observation` | Improvement-without-harm baseline; usually maps to `type = observation`. |
| `bug_like_flow` | Digital or procedural behavior resembles a system bug. |

---

## 5. Metadata-only label candidates

These may appear in `label_extraction_metadata.candidates[]` or validation notes, not in `canonical_payload.labels[]` unless a later product decision promotes them.

| Axis | Candidate examples |
|---|---|
| `location_context` | `school_area`, `residential_area`, `city_center`, `online_portal`, `public_transport_node`, `park`, `crossing`, `neighborhood` |
| `affected_scope` | `pedestrians`, `parents`, `children_context`, `elderly_context`, `drivers`, `residents`, `visitors`, `small_business`, `public_users` |
| `deep_need` | `predictability`, `being_heard`, `respect`, `safety_need`, `dignity`, `time_not_wasted`, `agency`, `fairness` |
| `desired_outcome` | `clear_rules`, `faster_response`, `safer_space`, `better_maintenance`, `accessible_service`, `transparent_process`, `human_contact`, `digital_fix` |

---

## 6. Internal-only labels

These are not public/card labels and must not be copied into `canonical_payload.labels[]`.

| Key | Use |
|---|---|
| `pii_present` | Internal privacy handling. |
| `redaction_needed` | Internal privacy handling. |
| `limited_depth` | Interview depth was limited by safety/trust. |
| `minor_context` | Minors are contextually involved; not a structured Issue field. |
| `health_context` | Health context requires careful handling. |
| `violence_context` | Violence/safety context requires gate/safety handling. |
| `confirmed_by_user` | Confidence state for metadata. |
| `gpt_hypothesis` | Candidate is inferred, not confirmed. |
| `needs_clarification` | Candidate requires clarification before canonical use. |
| `low_confidence` | Candidate is too weak for canonical use. |
| `conflict_unresolved` | User correction or evidence conflict is unresolved. |

---

## 7. Candidate metadata shape

When retaining label reasoning, use this conceptual shape in validation/normalization metadata:

```json
{
  "label": "transport",
  "axis": "topic_domain",
  "source": "Phase 2 resident story",
  "confidence": "high",
  "disposition": "canonical"
}
```

Allowed `disposition` values:

| Value | Meaning |
|---|---|
| `canonical` | Approved key and evidence supports inclusion in `canonical_payload.labels[]`. |
| `metadata_only` | Useful reasoning signal, not a public/card label. |
| `needs_clarification` | Plausible but not confirmed enough for canonical use. |
| `rejected` | Removed due to user correction, conflict, or vocabulary mismatch. |

---

## 8. Version history

| Version | Date | Change |
|---------|------|--------|
| 0.1 | 2026-04-26 | Initial controlled label axes and canonical/internal disposition rules (GIM-85). |
