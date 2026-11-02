# REQ-48 — Git homes & ownership

**Parent:** [00-overview.md](./00-overview.md)  
**As-of:** 2026-11-02T12:21:17Z

## Three homes (decision N2 + B + B2)

| Home | Path (logical target) | Role |
|------|----------------------|------|
| Working as-was | `GPT UI/instructions/` | Custom GPT Knowledge / as-built docs links; continuity (**C2**) |
| Snapshot archive | `GPT UI/instructions-archive-module1-req48/` | Immutable baseline copy of pre-cutover Module-1 + active pack (**B2**, AC-04) |
| Roles sibling | `GPT UI/instructions-roles-req15/` | New-line authoring SSOT + export-ready flat root children (**B**, AC-02/05) |

**Name ≠ create:** paths are target architecture. Do not scaffold empty trees until an implement wave explicitly orders creation.

## Archive mechanics (B2)

1. Copy interview-relevant Module-1 + active pack set into the snapshot home (inventory from `instruction-modules-index.md` + active pair — implement lists files; do not invent here).  
2. Keep working `instructions/` reachable until operator cutover policy says otherwise.  
3. Mapping (AC-01) should treat the **snapshot** as baseline; if working drifts, record discrepancy rather than silent drop.  
4. Existing `GPT UI/instructions/archive/` (nested packs / old taxonomy artifact) is **orthogonal** — do not overload it as Module-1 REQ-48 archive.

## Custom GPT policy (C2)

- As-was channel: **continuity / best-effort**, not a second feature roadmap inside REQ-48.  
- New authoring only in `instructions-roles-req15/`.  
- Long-term Actions sunset remains **Open** (REQ-48 §6) — not decided by this arch.  
- AC-03 (no model-owned stash/SPA) applies to the **new** line; rewriting as-was `api-orchestrator.md` is not required for REQ-48 DoD under C2.

## Ownership matrix

| Artifact class | Owner | Store |
|----------------|-------|-------|
| As-was Module-1 texts | GPT UI | Git `instructions/` + snapshot copy |
| Role-oriented texts + roles-schema manifest | GPT UI | Git sibling |
| Mapping table deliverable | GPT UI | Docs/analysis or sibling README — exact file at implement |
| Runtime mounted content | Ops / aibridge env | Volume or configured dir (peer) |
| Image-baked `doge-ai-bridge/src/instructions/` | Peer fixture/dev | Not prod SSOT after cutover (peer runtime-doc 11) |

## Relation to as-built map

Citations in [`../00-two-channels-overview.md`](../00-two-channels-overview.md) … [`../05-python-bridge-instruction-delivery.md`](../05-python-bridge-instruction-delivery.md) that point at `GPT UI/instructions/` remain correct for the as-was channel under C2.
