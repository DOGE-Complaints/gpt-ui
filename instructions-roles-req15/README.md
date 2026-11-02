# instructions-roles-req15 — REQ15 authoring SSOT (GPT UI)

| Field | Value |
|-------|--------|
| **UTC (P3)** | `2026-11-02T14:25:00Z` |
| **Story** | STORY-GPT-ROL-03 · AC-GPT-ROL03-01/02 (= AC-GPT-48-02/03) |
| **Wave** | pkg-000066 · GIM-453…461 |
| **Skill** | `openai-custom-gpt-builder` |

## What this home is

GPT UI owns the **authoring SSOT** for the role-oriented instruction line: shared `core.md` + five §3b role root children. Export/parity/manifest wiring = later GPT-ROL-04. Runtime assembly consume = aibridge (OOS here).

## Root children (path rule)

Consumer rejects nested paths (`_reject_root_child`). Only these files:

| File | Role |
|------|------|
| [`core.md`](./core.md) | Shared constitution (`shared` mapping rows) |
| [`communication_setup.md`](./communication_setup.md) | Role id `communication_setup` |
| [`interviewer.md`](./interviewer.md) | Role id `interviewer` |
| [`extractor.md`](./extractor.md) | Role id `extractor` |
| [`normalizer.md`](./normalizer.md) | Role id `normalizer` |
| [`checker.md`](./checker.md) | Role id `checker` |
| [`README.md`](./README.md) | This index |

## Packs (P1)

Schema-pack six-file + `schema-packs.README.md` stay **outside** this volume (mapping pack table / arch 02). Do **not** add pack entries as roles.manifest children in this home.

## Stash / SPA (AC-03)

This line **must not** teach the model to perform model-owned gateway story-draft stash handoffs or browser SPA submit redirects. Those duties are **server-only** (peer REQ-11). See mapping § orchestrator stash/SPA.

## References (not Pass oracles)

| Artifact | Role |
|----------|------|
| [`../docs/analysis/req-48-module1-role-mapping.md`](../docs/analysis/req-48-module1-role-mapping.md) | Classification baseline |
| [`../instructions-archive-module1-req48/`](../instructions-archive-module1-req48/) | Module-1 snapshot methodology |
| [`../docs/architecture/req-48-role-oriented-instruction-line/`](../docs/architecture/req-48-role-oriented-instruction-line/) | Path locks / P1 |
| `doge-ai-bridge/tests/fixtures/content/roles*.md` | **Shape reference only** — E3 rejected as Export Pass oracle |

## Continuity (C2)

Working `GPT UI/instructions/` one-personality Module-1 remains readable for Custom GPT Knowledge continuity. This sibling does not delete or empty that tree.
