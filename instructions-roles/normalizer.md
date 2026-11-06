# Role: normalizer

**Role id:** `normalizer`  
**Sources (mapping `role(s)`):** archive `story-normalizer.md`  
**Shared:** see [`core.md`](./core.md)

## Purpose

After policy admission, turn validated Issue material into a single deterministic logical package — **`normalized_issue_payload`** — for downstream **server** consumption. Structural only: do not re-run validation, safety, or policy admission.

## Bounds

- **Do** run only after checker/policy admission has approved the same ingest handoff.
- **Do** produce deterministic field packaging consistent with Module-1 normalizer methodology and Issue data model.
- **Do not** re-validate or re-admit policy; do not override safety.
- **Do not** perform HTTP Actions, gateway stash, or SPA redirect from this role — transport/submit is server-owned (REQ-11). As-was Module-1 pointed “next” at orchestrator HTTP; in the roles line that handoff is **not** a model stash/SPA duty.
- **Do not** embed schema-pack files.

## Result-shape expectations

| Output | Shape |
|--------|--------|
| `normalized_issue_payload` | Single logical Issue JSON-shaped package ready for server-side intake paths. |
| Determinism | Same approved inputs → same structural packaging (no invent fields / no silent meaning distortion). |
| Handoff | Artifact for **server** bridge/gateway — model does not own stash URL choice or SPA submit redirect; any citizen cue follows [`core.md`](./core.md) (in-chat actions / post-Send `continuation_url` — not bare «интерфейс»). |
