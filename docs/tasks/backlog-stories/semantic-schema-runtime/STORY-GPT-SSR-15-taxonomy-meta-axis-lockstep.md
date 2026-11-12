# STORY-GPT-SSR-15 — Pack Builder taxonomy meta: 13-axis lockstep

## Meta
- **Key:** `STORY-GPT-SSR-15-taxonomy-meta-axis-lockstep`
- **Пакет:** [`semantic-schema-runtime/`](./INDEX.md) (Wave 5 draft — **not** in INDEX)
- **Status:** Cancelled (inverted by GPT-PB-08)
- **Приоритет:** P1 (historical)
- **Тип:** product / Pack Builder taxonomy meta harden (doc+JSON Schema P3) — **DO NOT EXECUTE**
- **Parent analysis:** [handoff 2026-09-04](../../../../../doge-complaints-gateway/docs/analysis/uus-veerenni-civic-v1-gateway-load-blockers-handoff-2026-09-04.md) **Part A** (taxonomy rows)
- **Parent REQ:** [`REQ-46`](../../../requirements/REQ-46-pack-builder-prompt-wrapper-and-repair-debug.md) · GPT-PB-03
- **Inverted by:** [`STORY-GPT-PB-08-taxonomy-structural-free-axes-invert-ssr15.md`](../node-onboarding/STORY-GPT-PB-08-taxonomy-structural-free-axes-invert-ssr15.md) · pkg-000057 · 2026-09-04T19:37:37Z
- **SSOT post-invert:** D-SSR-11 / SSR-31 Contour2 **free** axes + GPT-PB-08 AC-01…03
- **Gateway cite:** [`disposition.py`](../../../../../doge-complaints-gateway/src/core/taxonomy/disposition.py) · [`resolver.py`](../../../../../doge-complaints-gateway/src/core/schema/resolver.py) L297–312 (`internal_axes ⊆ axes`)
- **Skill declared:** openai-custom-gpt-builder
- **Depends:** — (cancelled)
- **Blocks:** —
- **Method:** [`.cursor/rules/analysis.mdc`](../../../../../.cursor/rules/analysis.mdc)

---

# Layer 1 — Постановка задачи — SUPERSEDED

> **CANCELLED / INVERTED (GPT-PB-08 · D-SSR-11 / SSR-31):** Contour2 taxonomy axes are **FREE** pack-defined unique strings (`minItems: 1`). Pack Builder meta **MUST NOT** lock `axes` to exactly 13 or enum `TAXONOMY_AXIS_VALUES`. Authoritative schema: `GPT UI/instructions/node-onboarding/pack-builder-taxonomy.schema.json` post GPT-PB-08.

~~Taxonomy meta сейчас принимает любые axis strings… Gateway при наличии taxonomy_schema требует set(axes) == TAXONOMY_AXIS_VALUES (ровно 13)… Нужно закрыть gap lock-to-13.~~ **Wrong direction after D-SSR-11.**

## Scope — SUPERSEDED / DO NOT EXECUTE

- ~~Lock `axes` to the full Contour2 13-set~~ **FORBIDDEN** after GPT-PB-08.
- Replacement (authoritative via GPT-PB-08): free axes; dispositions ∈ `LABEL_DISPOSITION_VALUES`; document/enforce `internal_axes ⊆ axes`; tallinn-minimal exemplar (few axes) remains VALID.

## Out of scope (unchanged)

- Remapping uus vocabulary into 13 axes (Part B pack repair).
- Inventing new gateway axes.
- Dual GPT-rich vs gateway taxonomy artifact product decision.

## Acceptance Criteria — N/A (cancelled)

| ID | Criterion | Status |
|----|-----------|--------|
| AC-GPT-SSR15-01…05 | lock-to-13 / invalidate 24-axis | **Cancelled** — inverted by GPT-PB-08 (24-axis-shaped fixture MUST remain meta-VALID) |

## Soft

- Do **not** promote SSR-15 lock-to-13 into Wave 5 INDEX.
- Soft PB-09: checklist must not reintroduce «must equal 13 axes».
