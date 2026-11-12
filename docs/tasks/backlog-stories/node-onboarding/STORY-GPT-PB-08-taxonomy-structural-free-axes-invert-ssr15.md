# STORY-GPT-PB-08 — Taxonomy structural (free axes) + invert GPT-SSR-15

## Meta
- **Key:** `STORY-GPT-PB-08-taxonomy-structural-free-axes-invert-ssr15`
- **Пакет:** [`node-onboarding/`](./INDEX.md)
- **Status:** Done (Awaiting Commits) · pkg-000057 GIM-394…400 🟢 P3 2026-09-04T19:37:37Z
- **Приоритет:** P1 — Part A alignment (post D-SSR-11)
- **Тип:** product / Pack Builder taxonomy meta + backlog hygiene
- **Parent analysis:** [handoff 2026-09-04 Part A](../../../../../doge-complaints-gateway/docs/analysis/uus-veerenni-civic-v1-gateway-load-blockers-handoff-2026-09-04.md) — taxonomy / dispositions / internal_axes · repair rec 2
- **Parent REQ:** [`REQ-46`](../../../requirements/REQ-46-pack-builder-prompt-wrapper-and-repair-debug.md)
- **Related wrong-direction story:** [`STORY-GPT-SSR-15-taxonomy-meta-axis-lockstep.md`](../semantic-schema-runtime/STORY-GPT-SSR-15-taxonomy-meta-axis-lockstep.md) — **invert/cancel** in this story
- **Skill declared:** openai-custom-gpt-builder
- **Depends:** GPT-PB-03 Done; gateway SSR-31/32 Done (Contour2/1 open)
- **Blocks:** —
- **Method:** [`.cursor/rules/analysis.mdc`](../../../../../.cursor/rules/analysis.mdc)

---

# Layer 1 — Постановка задачи

## Зачем простыми словами

После D-SSR-11 gateway **принимает** pack-defined Contour2 axes (не lockstep 13). Meta уже free-axes — **aligned**. Остаётся усилить **structural** checks (`dispositions` enum, `internal_axes ⊆ axes`) и **отменить** GPT-SSR-15 (lock meta to 13), иначе будущий P3 сломает alignment с uus 24-axis и SSR-31.

## Scope

- Keep taxonomy meta **`axes` free** (`minItems: 1`, unique strings — no set-eq to 13).
- Enum-lock `dispositions` to `LABEL_DISPOSITION_VALUES` when present.
- Enforce / document `internal_axes ⊆ axes` (each ∈ `axes[]`).
- **Cancel/invert** GPT-SSR-15 story intent (Status + Layer1 note: free axes + D-SSR-11 SSOT).

## Out of scope

- Remapping uus vocabulary; Part B pack.json.
- Locking axes to tallinn 13.
- pack.json exact/civic/geo (PB-06/07).

## Verified current state (high-level)

| Fact | Evidence |
|------|----------|
| Meta axes free | `pack-builder-taxonomy.schema.json` axes minItems 1 |
| Gateway Contour2 open | handoff Flexible taxonomy; SSR-31 Done |
| GPT-SSR-15 still lock-to-13 | story file Status Todo · lock intent |
| dispositions loose in meta | taxonomy meta dispositions branch |

## Target / Acceptance Criteria (stubs)

- [x] **AC-GPT-PB08-01:** Taxonomy meta keeps free axes; **MUST NOT** require exactly 13.
- [x] **AC-GPT-PB08-02:** dispositions ∈ gateway disposition set; internal_axes ⊆ axes documented/enforced.
- [x] **AC-GPT-PB08-03:** GPT-SSR-15 marked Cancelled/inverted with pointer to this story + D-SSR-11.

---

# Layer 2 — Реализация (P3; не в этой materialize-волне)

## Gateway / product SSOT (post D-SSR-11)

| Rule | Behaviour | Evidence |
|------|------------|----------|
| Contour2 axes | Free non-empty unique strings; **no** set-eq to 13 | handoff Flexible taxonomy; SSR-31 Done |
| Contour1 wire | strip+lower only | `axes.py` `normalize_axis` |
| Dispositions | ∈ `LABEL_DISPOSITION_VALUES` | [`disposition.py`](../../../../../doge-complaints-gateway/src/core/taxonomy/disposition.py) L6–14: `canonical`, `metadata_only`, `needs_clarification`, `rejected`, `internal` |
| internal_axes | ⊆ axes | handoff Part A; resolver taxonomy structural |

## Wrong-direction sibling

| File | Current | This story P3 |
|------|---------|---------------|
| [`STORY-GPT-SSR-15-taxonomy-meta-axis-lockstep.md`](../semantic-schema-runtime/STORY-GPT-SSR-15-taxonomy-meta-axis-lockstep.md) | Todo · lock axes to 13 | **Cancelled / inverted** — Status + Layer1 banner: do **not** lock to 13; SSOT = D-SSR-11 + GPT-PB-08 |

## Meta today

| Branch | Now |
|--------|-----|
| `axes` | free `minItems: 1` — **keep** |
| `dispositions` | `items: { type: string }` — loosen |
| `internal_axes` | free strings — no ⊆ axes check |

## Subtasks (Txx)

| ID | Subtask | Detail |
|----|---------|--------|
| T01 | Keep free axes | Explicit `$comment`: MUST NOT enum/const to 13; uniqueItems true recommended. |
| T02 | dispositions enum | `items.enum` = five `LABEL_DISPOSITION_VALUES`. |
| T03 | internal_axes ⊆ axes | Document MUST in description; if JSON Schema alone insufficient, add checklist row in PB-09 **or** `$comment` + operator note that Pack Builder primary/repair must enforce ⊆. Prefer schema `uniqueItems` + description; note gateway rejects violation at load. |
| T04 | Exemplar | Keep short tallinn-minimal axes list (2+) — valid under free axes. |
| T05 | Invert GPT-SSR-15 | Edit SSR-15: Status `Cancelled (inverted by GPT-PB-08)`; strike lock-to-13 Scope; link D-SSR-11 + this story. Update SSR package INDEX row if present. |
| T06 | Smoke (P3) | 24-axis-shaped taxonomy fixture still meta-VALID; disposition `nope` → INVALID. |

## Точки в docs (P3)

| Action | Path |
|--------|------|
| UPDATE | `GPT UI/instructions/node-onboarding/pack-builder-taxonomy.schema.json` |
| UPDATE | `GPT UI/docs/tasks/backlog-stories/semantic-schema-runtime/STORY-GPT-SSR-15-taxonomy-meta-axis-lockstep.md` (+ INDEX if needed) |

## Target / Acceptance Criteria (refined)

- [x] **AC-GPT-PB08-01:** Taxonomy meta keeps free axes; **MUST NOT** require exactly 13.
- [x] **AC-GPT-PB08-02:** dispositions ∈ `LABEL_DISPOSITION_VALUES`; internal_axes ⊆ axes documented (and enforced where JSON Schema allows).
- [x] **AC-GPT-PB08-03:** GPT-SSR-15 Status Cancelled/inverted with pointer to GPT-PB-08 + D-SSR-11.

## Dependencies

- Soft: PB-09 checklist must not reintroduce «must equal 13 axes».
