# STORY-GPT-SSR-02 — OpenAPI: schema_binding + geo_detail (gateway-named only)

## Meta
- **Key:** `STORY-GPT-SSR-02-openapi-schema-binding-geo-detail`
- **Пакет:** [`semantic-schema-runtime/`](./INDEX.md)
- **Status:** Done
- **Приоритет:** P1 — contract surface for stash body
- **Тип:** contract (OpenAPI Action artifact)
- **Parent REQ:** [`REQ-45-semantic-schema-intake-envelope-compatibility.md`](../../../requirements/REQ-45-semantic-schema-intake-envelope-compatibility.md)
- **Siblings:** [GPT-SSR-01](./STORY-GPT-SSR-01-core-vs-swappable-packs.md) (upstream), [GPT-SSR-03](./STORY-GPT-SSR-03-instructions-emit-binding-and-geo.md) (downstream)
- **Skill declared:** openai-custom-gpt-builder
- **Depends:** GPT-SSR-01 (pack paths + civic field projection known)
- **Operator:** OpenAPI change → **Custom GPT Actions re-import** required (unlike pack-only Instruction upload).

## Зачем простыми словами

Gateway уже **MUST** root `schema_binding` и умеет `geo_detail`. GPT Actions OpenAPI их не описывает — стеш уходит в 4xx или в «серую» зону. Нужен транспортный контракт только с **gateway-named** токенами.

## Архитектура решения

```text
StoryDraftStashRequest (Actions artifact = transport SSOT)
├── schema_version: "m2.story_intake_envelope.v2"     ← envelope id (COMPAT-001)
├── narrative: … (existing; location_query stays)
├── schema_binding: SchemaBinding                   ← REQUIRED (mirror gateway)
│     ├── schema_id, schema_version                 ← semantic pair ≠ envelope
│     ├── structured_payload                        ← object (pack JSON Schema)
│     └── profile_id?, profile_version?
├── geo_detail?: GeoDetail                          ← OPTIONAL
│     ├── latitude?, longitude? (together)
│     ├── address?: { country, region, settlement, district, street,
│     │               house | house_range | houses[]  }  ← XOR house forms
│     ├── normalized_label?, confidence?, provider?
└── origin? / privacy? / gpt_signals? / …           ← existing sidecars
```

| Rule | Detail |
|------|--------|
| Name source | Only [`contracts.py`](../../../../../doge-complaints-gateway/src/core/intake/contracts.py) / sibling 51 — **no invent** |
| Envelope vs semantic | Root `schema_version` = envelope; binding.`schema_version` = pack version (`v1`) — never equal envelope string (gateway rejects) |
| Requiredness | `schema_binding` in OpenAPI `required[]` (MUST emit); `geo_detail` optional |
| Lockstep | Bump `info.version` + [`story-api-methods-reference.md`](../../../../instructions/story-api-methods-reference.md) |

## Sprawl / gap matrix (OpenAPI)

| Concern | Today | Target |
|---------|-------|--------|
| `StoryDraftStashRequest` properties | narrative, origin, privacy, live_story_context, gpt_signals ([openapi](../../../custom-gpt-story-intake-actions.openapi.yaml) L249–274) | + `schema_binding`, + `geo_detail` |
| `schema_binding` / `geo_detail` schemas | **Absent** | New `components.schemas` |
| Envelope enum | L253–255 OK | Keep |
| `NarrativeTaxonomy` 13 civic axes | L80–139 fixed | **MVP keep** (Residual: multi-node soften = follow-up) |
| `narrative.location_query` | L170–172 exists | Keep (fuzzy resolve) |

## Scope

- Add `SchemaBinding`, `GeoDetail`, `GeoDetailAddress` schemas 1:1 with gateway parser field names.
- Extend `StoryDraftStashRequest`: properties + `required` includes `schema_binding`.
- Lockstep methods-reference + version bump + re-import note.
- Document Residual: civic axes in NarrativeTaxonomy stay for MVP.

## Out of scope

- Instruction emit → GPT-SSR-03.
- Pack file creation → GPT-SSR-01 P3.
- Gateway parser; SPA; identity; invent fields.
- Redesign / soften `NarrativeTaxonomy` for multi-node (Residual).
- Marketplace / §30.

## Verified current state

| Fact | Evidence |
|------|----------|
| `required: [schema_version, narrative]` | openapi.yaml L249–251 |
| No binding/geo_detail properties | L259–274 |
| Envelope enum v2 | L253–255 |
| `info.version` | L13 `"0.7.0"` |
| Gateway SchemaBinding required fields | contracts.py L393–431: `schema_id`, `schema_version`, `structured_payload`; optional profiles |
| Gateway GeoDetail keys | contracts.py L356–389: lat/lon, address, normalized_label, confidence, provider |
| Address house XOR | contracts.py L337–341 |
| Absent binding → 4xx | contracts.py L393–396; services.py NODE_SCHEMA match |

## Подзадачи

| ID | Задача | DoD |
|----|--------|-----|
| **T01** | `components.schemas`: `SchemaBinding`, `GeoDetail`, `GeoDetailAddress` | Field names + descriptions match contracts.py; house XOR documented in description; binding.`schema_version` ≠ envelope note |
| **T02** | Wire into `StoryDraftStashRequest` | Properties added; `required` = `[schema_version, narrative, schema_binding]`; `geo_detail` optional `$ref` |
| **T03** | Lockstep [`story-api-methods-reference.md`](../../../../instructions/story-api-methods-reference.md) | Binding/geo rows + version parity with OpenAPI |
| **T04** | Examples / God Mode notes | Sample stash JSON includes binding; display-vs-transport note (REQ-31/32 precedent) |
| **T05** | Version bump + operator re-import | `info.version` increment; changelog line; operator: re-import Actions after publish |
| **T06** | Residual doc in story + OpenAPI comment | NarrativeTaxonomy 13 axes **remain**; multi-node soften = follow-up story (not this wave) |

### T01 field checklist (gateway-named only)

**SchemaBinding:** `schema_id`, `schema_version`, `structured_payload`, `profile_id`, `profile_version`  
**GeoDetail:** `latitude`, `longitude`, `address`, `normalized_label`, `confidence`, `provider`  
**GeoDetailAddress:** `country`, `region`, `settlement`, `district`, `street`, `house`, `house_range`, `houses`

## Target / Acceptance Criteria

- [x] **AC-GPT-REQ3-02** (OpenAPI half): only gateway-named tokens added.
- [x] **AC-GPT-REQ3-03:** v2 remains required envelope identifier.
- [x] **AC-GPT-BIND-01** (contract): `schema_binding` required on request schema.
- [x] Optional `geo_detail` shape matches REQ-45 §2 / gateway address rules.
- [x] Methods-reference lockstep + version bump (T03/T05).
- [x] Residual taxonomy axes documented (T06) — no silent invent of soft schema.

## Точки в коде

| Зона | Path |
|------|------|
| OpenAPI | `GPT UI/docs/custom-gpt-story-intake-actions.openapi.yaml` |
| Methods ref | `GPT UI/instructions/story-api-methods-reference.md` |
| Gateway lockstep | `doge-complaints-gateway/src/core/intake/contracts.py` |

## Dependencies

- GPT-SSR-01 (active pair + structured_payload outline for examples).
- Unblocks GPT-SSR-03.
- Sibling 51; REQ-22 envelope v2.

## Residual

- **NarrativeTaxonomy fixed 13 axes** — MVP keep for civic lockstep; legal_process / mobility_observation nodes may need follow-up OpenAPI soften (not invent here).
- Pack `geo_intake` / `merge` / `mirror_to_payload` are **not** wire properties — node config only.
