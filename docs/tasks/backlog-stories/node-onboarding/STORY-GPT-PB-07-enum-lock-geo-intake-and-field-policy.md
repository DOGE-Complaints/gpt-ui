# STORY-GPT-PB-07 — Enum-lock `geo_intake.mode` + `field_policy`

## Meta
- **Key:** `STORY-GPT-PB-07-enum-lock-geo-intake-and-field-policy`
- **Пакет:** [`node-onboarding/`](./INDEX.md)
- **Status:** Done (Awaiting Commits) — P3 pkg-000056 · GIM-387…393 🟢 · 2026-09-04T19:28:14Z
- **Приоритет:** P1 — Part A blockers
- **Тип:** product / Pack Builder pack.json meta harden (doc+JSON Schema P3)
- **Parent analysis:** [handoff 2026-09-04 Part A](../../../../../doge-complaints-gateway/docs/analysis/uus-veerenni-civic-v1-gateway-load-blockers-handoff-2026-09-04.md) — `geo_intake.mode` · `field_policy` rows
- **Parent REQ:** [`REQ-46`](../../../requirements/REQ-46-pack-builder-prompt-wrapper-and-repair-debug.md) · GPT-PB-03
- **Skill declared:** openai-custom-gpt-builder
- **Depends:** GPT-PB-03 Done; soft GPT-PB-06 (same file)
- **Blocks:** soft GPT-PB-09 (hash)
- **Method:** [`.cursor/rules/analysis.mdc`](../../../../../.cursor/rules/analysis.mdc)

---

# Layer 1 — Постановка задачи

## Зачем простыми словами

Meta сейчас показывает примеры `geo_intake.mode` = `required` / `disabled` и принимает **любые** строки в `field_policy`. Gateway допускает только `GEO_INTAKE_MODES` и `FIELD_POLICY_STATES`. Нужно enum-lock в pack meta, чтобы Pack Builder не пропускал невалидные mode/policy.

## Scope

- Lock `geo_intake.mode` to gateway set; fix examples; constrain extra keys (merge/mirror only as gateway).
- Lock `field_policy` additionalProperties values to `FIELD_POLICY_STATES` (7).
- Update in-schema examples.

## Out of scope

- exact/civic clustering (PB-06); taxonomy (PB-08); optional required[] drift (PB-09).
- Gateway code; Part B packs.

## Verified current state (high-level)

| Fact | Evidence |
|------|----------|
| Meta mode examples include `required`, `disabled` | `pack-builder-pack.schema.json` geo_intake.mode examples |
| Gateway modes | `contracts.py` `GEO_INTAKE_MODES` |
| field_policy any string | meta `additionalProperties: { type: string }` |
| Gateway policy states | `FIELD_POLICY_STATES` (7) |

## Target / Acceptance Criteria (stubs)

- [x] **AC-GPT-PB07-01:** `geo_intake.mode` enum = gateway three modes only; invalid examples removed.
- [x] **AC-GPT-PB07-02:** `field_policy` values enum-locked to `FIELD_POLICY_STATES`.
- [x] **AC-GPT-PB07-03:** Document merge/mirror expectations aligned with `GeoIntakeBlock`.

---

# Layer 2 — Реализация (P3; не в этой materialize-волне)

## Gateway contracts (cite)

| Contract | Values | Evidence |
|----------|--------|----------|
| `GEO_INTAKE_MODES` | `optional`, `require_location_or_detail`, `require_detail` | [`contracts.py`](../../../../../doge-complaints-gateway/src/core/schema/contracts.py) L75–77 |
| `GeoIntakeBlock` keys only | `mode`, `merge`, `mirror_to_payload` | `resolver.py` `_parse_geo_intake` L420–447 — unknown keys **reject** |
| `FIELD_POLICY_STATES` (7) | `required`, `optional`, `disabled`, `forbidden`, `node_private`, `derived`, `computed_aggregate_only` | `contracts.py` L10–20; `resolver.py` L645–648 |

## Meta today

| Branch | Now | Gap |
|--------|-----|-----|
| `geo_intake.mode` examples | includes `required`, `disabled` | **Invalid** vs gateway |
| `geo_intake` | `additionalProperties: true` | Gateway forbids unknown keys |
| `field_policy` values | any string | Must ∈ FIELD_POLICY_STATES |

## Subtasks (Txx)

| ID | Subtask | Detail |
|----|---------|--------|
| T01 | Enum `geo_intake.mode` | `enum`: three `GEO_INTAKE_MODES` only; delete `required`/`disabled` from examples. |
| T02 | Close geo_intake props | `required`: `mode`, `merge`, `mirror_to_payload`; `additionalProperties: false`; types bool/bool. |
| T03 | Enum field_policy values | `additionalProperties: { type: string, enum: […7 FIELD_POLICY_STATES…] }`. |
| T04 | Refresh examples | tallinn-like: mode `optional`, merge/mirror true; field_policy sample with only enum values. |
| T05 | `$comment` | GPT-PB-07 · Part A · contracts.py. |
| T06 | Smoke (P3) | Candidate with mode=`required` → INVALID; mode=`optional` + valid policies → VALID. |

## Точки в docs (P3)

| Action | Path |
|--------|------|
| UPDATE | `GPT UI/instructions/node-onboarding/pack-builder-pack.schema.json` |

## Target / Acceptance Criteria (refined)

- [x] **AC-GPT-PB07-01:** `geo_intake.mode` enum = three gateway modes; invalid examples removed.
- [x] **AC-GPT-PB07-02:** `field_policy` values enum-locked to all seven `FIELD_POLICY_STATES`.
- [x] **AC-GPT-PB07-03:** `geo_intake` only `mode`/`merge`/`mirror_to_payload` (`additionalProperties: false`).

## Dependencies

- Soft merge with PB-06 on same JSON file.
