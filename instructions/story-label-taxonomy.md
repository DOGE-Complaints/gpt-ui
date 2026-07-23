# Issue Label Taxonomy — controlled label axes

**Product:** DOGEstonia — Module 1 (GPT Interview / ingest → Issue)  
**Purpose:** Controlled GPT-side source of truth for deriving `canonical_payload.labels[]` and label reasoning metadata from resident stories.

| Document field | Value |
|----------------|--------|
| **Version** | 0.2.2 |
| **Date** | 2026-07-23 |
| **Traceability** | FR-M1-026/027/044…051; REQ-36; GPT-TAX-01; [`story-data-model.md`](story-data-model.md), [`story-interview-flow.md`](story-interview-flow.md), [`ingest-validation.md`](ingest-validation.md), [`story-normalizer.md`](story-normalizer.md) |

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
| `affected_scope` | Who or what is affected. | May produce canonical labels when supported. |
| `civic_signal` | Recurrence, systemic relevance, public cost. | May produce canonical labels when supported. |
| `deep_need` | Human/civic need beneath the story. | May produce canonical labels when supported. |
| `desired_outcome` | Better state requested by the resident. | May produce canonical labels when supported. |
| `ecosystem_signal` | Absence or decline of civic environment, not a single broken service. | May produce canonical labels when supported. |
| `governance_signal` | Governance model, transparency, or ownership as part of the desired solution. | May produce canonical labels when supported. |
| `risk_privacy_safety` | Safety, PII, minors, sensitive topics. | Internal metadata only. |
| `confidence_state` | Label certainty and correction status. | Internal metadata only. |

**Axis enum lockstep (GPT-TAX-01 / GW-TAX-01):** the 13 axis keys above **MUST** match gateway `TAXONOMY_AXIS_VALUES` in [`axes.py`](../../doge-complaints-gateway/src/core/taxonomy/axes.py). Do not invent alternate axis names on the wire.

**Open decision (REQ-36 OD-1):** axis name `affected_scope` retained (not renamed to `affected_population`) to limit propagation across instruction modules.

**Open decision (REQ-36 OD-2):** no separate `equity_dimension` axis — fair-access themes use `civic_signal.equity_access`.

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
| `culture` | Cultural life, arts, heritage events, cultural access and continuity. |
| `youth_development` | Youth programs, spaces, mentorship, after-school civic development. |
| `science_and_research` | Science education, research culture, STEM, intellectual environment. |
| `arts_and_creativity` | Creative arts, performing arts, maker and creative spaces. |
| `sports_and_recreation` | Sports facilities, recreation, active lifestyle infrastructure. |
| `community_life` | Community hubs, neighbourhood cohesion, shared civic life. |
| `integration` | Integration services, newcomer inclusion, cross-community belonging. |
| `language_access` | Language learning, multilingual access, interpretation barriers. |
| `migration_adaptation` | Migration-related adaptation, refugee support, settlement friction. |
| `social_services` | Social support, welfare access, community social programs. |
| `employment_and_skills` | Jobs, skills training, employability, labour-market access. |
| `local_economy` | Small business, local economic vitality, commercial civic friction. |
| `urban_planning` | City planning, zoning, development decisions affecting residents. |
| `public_participation` | Civic participation, consultation, co-design, resident voice. |
| `heritage` | Built or cultural heritage preservation and access. |
| `mental_wellbeing_civic` | Public mental wellbeing, safe and supportive civic environments. |
| `family_support` | Family services, parenting support, family-oriented civic programs. |
| `elderly_support` | Services and spaces for older residents. |
| `childcare` | Childcare availability, quality, and access. |
| `school_environment` | Physical and social school environment beyond curriculum. |
| `after_school_ecosystem` | After-school programs, clubs, and enrichment ecosystem. |
| `civil_society` | NGOs, volunteers, civic initiatives, third-sector role. |
| `trust_and_governance` | Trust in public institutions, transparency, accountability themes at domain level. |

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
| `information_gap_civic` | Civic information, rules, or opportunities are hard to discover or understand. |
| `participation_signal` | Resident wants or lacks meaningful participation in civic decisions. |
| `social_cohesion` | Story concerns community bonds, inclusion, or social fabric. |
| `brain_drain_signal` | **Civic significance:** resident frames loss of mentors, active neighbours, or community participants from public-facing civic life (clubs, volunteering, local mentorship)—not institutional staffing or program capacity. |

### 4.4 Issue archetype support

| Key | Meaning |
|---|---|
| `process_absurdity` | The case supports `type = absurdity` because the process is visibly irrational. |
| `system_loop` | The case supports `type = system_bug` or `absurdity` because the resident is trapped in a loop. |
| `harm_reported` | The resident reports concrete harm or burden; do not downgrade to observation. |
| `improvement_wish` | The story is framed as a desired improvement. |
| `positive_observation` | Improvement-without-harm baseline; usually maps to `type = observation`. |
| `bug_like_flow` | Digital or procedural behavior resembles a system bug. |

### 4.5 Service object

| Key | Meaning |
|---|---|
| `kindergarten` | Kindergarten or early-childhood facility or program. |
| `school` | General school as a concrete service object. |
| `youth_center` | Youth centre, youth house, or dedicated youth space. |
| `cultural_center` | Cultural centre, house of culture, or civic cultural hub. |
| `library` | Public library or library service point. |
| `sports_facility` | Sports hall, stadium, court, or recreation facility. |
| `community_hub` | Multi-purpose community centre or neighbourhood hub. |
| `after_school_program` | After-school club, circle, or enrichment program. |
| `science_club` | Science club, lab activity, or youth science program. |
| `research_lab` | Research or academic lab context in civic/education narrative. |
| `mentoring_program` | Mentorship, tutoring, or guided development program. |
| `performing_arts_venue` | Theatre, concert hall, or performing-arts venue. |
| `creative_space` | Studio, maker space, or creative workshop facility. |
| `museum` | Museum or exhibition venue. |
| `public_event` | Festival, fair, or recurring public cultural event. |
| `integration_program` | Integration course, adaptation program, or newcomer service. |
| `language_course` | Language class or multilingual access program. |
| `social_service_office` | Social welfare office or case-management touchpoint. |
| `municipal_service` | Generic municipal service counter or city service process. |
| `online_portal_service` | Specific digital portal or e-service as the object of the story. |

### 4.6 Affected scope

| Key | Meaning |
|---|---|
| `children` | Children as primary affected group. |
| `youth` | Teenagers and young people. |
| `students` | School or university students. |
| `families` | Families with children or multi-generational households. |
| `parents` | Parents or guardians as affected group. |
| `elderly` | Older residents. |
| `residents` | General local residents of the area. |
| `people_with_disabilities` | People with mobility, sensory, or other accessibility needs. |
| `pedestrians` | People walking in the area. |
| `cyclists` | People cycling in the area. |
| `drivers` | People driving in the area. |
| `public_transport_users` | Regular public-transport passengers. |
| `newcomers` | Recent arrivals adapting to the city. |
| `refugees` | Refugees or asylum seekers as affected group. |
| `ukrainian_refugees` | Ukrainian refugees specifically mentioned or implied. |
| `russian_speaking` | Russian-speaking community members. |
| `estonian_speaking` | Estonian-speaking community members. |
| `multilingual_communities` | Mixed-language or multilingual resident groups. |
| `low_income_residents` | Residents facing economic hardship or limited resources. |
| `small_businesses` | Local small-business operators; use for one or more affected businesses when the story centres on commercial operators (singular or plural narrative). |
| `cultural_workers` | Artists, cultural workers, or creative professionals. |
| `teachers` | Teachers or educators as affected group. |
| `mentors` | Mentors, tutors, or volunteer guides. |
| `volunteers` | Civic volunteers or community organisers. |

### 4.7 Deep need

| Key | Meaning |
|---|---|
| `belonging` | Need to belong in community or city life. |
| `dignity` | Need for respectful, dignified treatment. |
| `agency` | Need to influence outcomes or participate meaningfully. |
| `being_heard` | Need to be listened to by institutions or community. |
| `predictability` | Need for reliable, predictable rules and services. |
| `safety_need` | Need for physical or psychological safety in civic life. |
| `fairness` | Need for fair treatment and equal burden sharing. |
| `cultural_identity` | Need to express or preserve cultural identity. |
| `intellectual_growth` | Need for learning, curiosity, and intellectual development. |
| `creative_self_realization` | Need for creative expression and self-realisation. |
| `meaningful_community` | Need for substantive community connection. |
| `future_opportunity` | Need for prospects and opportunities for self or children. |
| `trust` | Need to trust institutions and civic processes. |
| `continuity` | Need for continuity of programs, places, or traditions. |
| `respect` | Need for respectful interaction with services and spaces. |
| `time_not_wasted` | Need to avoid wasted time in bureaucratic or broken flows. |

### 4.8 Desired outcome

| Key | Meaning |
|---|---|
| `new_public_service` | Resident wants a new public service to exist. |
| `new_community_hub` | Resident wants a new community hub or centre. |
| `better_access` | Resident wants easier access to existing services or spaces. |
| `transparent_process` | Resident wants clearer, more transparent processes. |
| `stronger_standards` | Resident wants higher quality or safety standards. |
| `more_mentors` | Resident wants more mentors, tutors, or guides. |
| `more_programs` | Resident wants more programs, clubs, or activities. |
| `better_coordination` | Resident wants better coordination between actors. |
| `replicable_solution` | Resident proposes a model others could copy. |
| `open_methodology` | Resident wants open, shareable methods or practices. |
| `cooperative_institution` | Resident wants cooperative or community-led institution model. |
| `multilingual_access` | Resident wants services accessible in multiple languages. |
| `safe_environment` | Resident wants a safer physical or social environment. |
| `human_centered_city` | Resident wants a more human-centred city experience. |
| `clear_rules` | Resident wants rules and responsibilities to be clear. |
| `faster_response` | Resident wants quicker institutional response. |
| `safer_space` | Resident wants a specific space to be safer. |
| `better_maintenance` | Resident wants better upkeep of infrastructure or services. |
| `accessible_service` | Resident wants physically or digitally accessible service. |
| `human_contact` | Resident wants human contact instead of only digital flows. |
| `digital_fix` | Resident wants a digital service defect fixed. |

### 4.9 Ecosystem signal

| Key | Meaning |
|---|---|
| `ecosystem_gap` | The civic environment or program ecosystem is missing or thin. |
| `institutional_decline` | Institutions, venues, or programs are closing or weakening. |
| `missing_infrastructure` | Needed civic infrastructure does not exist at scale. |
| `weak_coordination` | Actors do not coordinate; residents bear the fragmentation cost. |
| `capacity_shortage` | Insufficient capacity (staff, space, funding) across the ecosystem. |
| `mentor_shortage` | Lack of mentors, tutors, or experienced guides. |
| `community_fragmentation` | Community ties or participation are breaking down. |
| `loss_of_continuity` | Long-running programs or traditions are at risk of ending. |
| `brain_drain` | **Ecosystem capacity:** resident describes institutional or program ecosystem losing talent, leaders, or sustained operators so venues, programs, or civic infrastructure cannot retain people who keep the environment alive. |
| `underused_resources` | Existing assets or spaces are underused relative to need. |
| `replicable_model_needed` | Resident describes a gap that a replicable model could fill. |

### 4.10 Governance signal

| Key | Meaning |
|---|---|
| `transparent_governance` | Resident wants transparent decision-making and information. |
| `participatory_governance` | Resident wants participatory or co-governance models. |
| `cooperative_model` | Resident proposes or wants a cooperative governance model. |
| `power_concentration_risk` | Resident fears power concentration or capture. |
| `accountability_gap` | Accountability for decisions or outcomes is missing. |
| `open_source_model` | Resident wants open, documented, shareable governance or methods. |
| `replicability` | Governance or institutional model should be replicable elsewhere. |
| `community_ownership` | Resident wants community ownership or stewardship of the solution. |

---

## 5. Metadata-only label candidates

These may appear in `label_extraction_metadata.candidates[]` or validation notes, not in `canonical_payload.labels[]` unless a later product decision promotes them.

| Axis | Candidate examples |
|---|---|
| `location_context` | `school_area`, `residential_area`, `city_center`, `online_portal`, `public_transport_node`, `park`, `crossing`, `neighborhood` |
| `affected_scope` | `visitors`, `public_users` — metadata-only until explicit product promotion (REQ-36 §2.3 scope; REQ-20 legacy examples) |

Promoted axes (`affected_scope`, `deep_need`, `desired_outcome`) now have canonical blocks in §4.6–4.8 — do not duplicate promoted keys here except the demoted candidates above.

---

## 6. Internal-only labels

These are not public/card labels and must not be copied into `canonical_payload.labels[]`. On the wire (`canonical_payload.taxonomy` / `narrative.taxonomy`), place them under the matching axis (`risk_privacy_safety` or `confidence_state`) with **`disposition: "internal"`** (GPT-TAX-01 — gateway filters public surfaces).

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

Allowed `disposition` values (lockstep gateway `LabelDisposition` in [`disposition.py`](../../doge-complaints-gateway/src/core/taxonomy/disposition.py)):

| Value | Meaning |
|---|---|
| `canonical` | Approved key and evidence supports inclusion in `canonical_payload.labels[]` and public card surfaces. |
| `metadata_only` | Useful reasoning signal, not a public/card label. |
| `needs_clarification` | Plausible but not confirmed enough for canonical use. |
| `rejected` | Removed due to user correction, conflict, or vocabulary mismatch. |
| `internal` | §6 internal-only / safety-privacy keys; included in `taxonomy` wire; **not** in flat `labels[]`; gateway filters from public surfaces. |

---

## 8. Version history

| Version | Date | Change |
|---------|------|--------|
| 0.2.2 | 2026-07-23 | **GPT-TAX-01 / GIM-211+212:** disposition `internal`; axis/disposition lockstep notes vs GW-TAX-01. |
| 0.2.1 | 2026-06-05 | **REQ-36 audit / GIM-161…162:** GAP-36-01 — dedup `small_business` removed; `visitors`/`public_users` demoted to §5; GAP-36-02 — disambiguated `brain_drain_signal` (civic) vs `brain_drain` (ecosystem capacity). |
| 0.2 | 2026-06-05 | **REQ-36 / GIM-156…158:** expanded `topic_domain`, `civic_signal`, `service_object`; promoted `affected_scope`, `deep_need`, `desired_outcome`; new axes `ecosystem_signal`, `governance_signal`; OD-1/OD-2 documented in §3. |
| 0.1 | 2026-04-26 | Initial controlled label axes and canonical/internal disposition rules (GIM-85). |
