# Activity / donor legacy guardrails inventory

**Scope:** `GPT UI/instructions/*`  
**Target runtime:** DOGEstonia Module 1 (Issue-first ingest)

This file tracks where legacy donor wording can still appear and defines edit discipline so Issue ingest changes do not accidentally reintroduce Activity routes or payload names into the active path.

---

## Issue-first edit checklist

Before merging edits in instruction files, verify:

1. Runtime ingest path points to `issue-lifecycle-instructions.md` and not donor lifecycle text.
2. Strict output artifact is `normalized_issue_payload` (not `normalized_activity_payload`).
3. HTTP references for Issue track come from `issue-api-methods-reference.md`.
4. If `/activities` is mentioned, it is explicitly marked as donor/historical and not executable for DOGEstonia ingest.
5. `issue-policy-gate.md` remains in strict chain before `issue-normalizer.md`.
6. `api-orchestrator.md` is still the only module allowed to execute HTTP.
7. `instruction-modules-index.md` keeps the explicit pointer to Issue-first track.
8. New examples use Issue domain terms (`issue`, `labels`, `title`, `description`) and avoid donor aliases.
9. Any donor compatibility note includes a clear "not for Issue ingest" label.

Canonical chain references:

- [`issue-lifecycle-instructions.md`](./issue-lifecycle-instructions.md)
- [`issue-normalizer.md`](./issue-normalizer.md)
- [`issue-policy-gate.md`](./issue-policy-gate.md)
- [`issue-api-methods-reference.md`](./issue-api-methods-reference.md)

---

## High-risk files (review first)

| File | Risk | Guardrail |
|------|------|-----------|
| `base.md` | Historical examples can reintroduce donor artifact names. | Keep Issue-first routing and strict-chain pointers in §1 (INGEST). |
| `ingest-validation.md` | Mixed guidance can blur interview flow vs structural checks. | Keep overlay precedence + Issue canonical references at top. |
| `api-orchestrator.md` | HTTP examples are drift-prone when Actions/OpenAPI changes. | Keep lockstep with `issue-api-methods-reference.md`; donor mentions only as historical notes. |
| `instruction-modules-index.md` | Editors may start from wrong module set. | Keep explicit "use Issue track" pointer near top. |

