# REQ-48 — Target technical architecture: role-oriented instruction line

**Status:** Target (operator interview 2026-11-02) — **not** as-built Module-1  
**As-of:** 2026-11-02T12:21:17Z  
**Method:** [`.cursor/rules/analysis.mdc`](../../../../.cursor/rules/analysis.mdc) · Architect Studio target-tech-architecture  
**Interview report:** [`../../analysis/zeya888.target-tech-arch-interview-req-48-role-oriented-instruction-line-2026-11-02.md`](../../analysis/zeya888.target-tech-arch-interview-req-48-role-oriented-instruction-line-2026-11-02.md)

## Context & inputs

| Input | Role |
|-------|------|
| [`REQ-48`](../../requirements/REQ-48-req15-role-oriented-instruction-line-and-export.md) | Functional: archive → mapping → role line → export parity |
| Story scope | **none** (0 STORY files in this run) |
| Peer contract | [aibridge REQ-13 §3b](../../../../doge-ai-bridge/docs/requirements/REQ-13-REQ15-SEMANTIC-STATE-ROLES-INSTRUCTION-ASSEMBLY.md) (roles-schema; consume parity ≠ export parity) |
| Peer OOS | REQ-11 (server stash/SPA), REQ-12 (providers) — no instruction authorship |
| As-built dual-channel map | [`../README.md`](../README.md) (`00`–`05`) |

**Functional vs tech:** REQ остаётся product-facing (зачем/что). Этот пакет — **как** устроить Git-дома, export surface, publish/verify в стеке GPT UI без Python handlers.

**Use case:** оператор отдаёт Module-1 методологию в role-oriented export для `doge-ai-bridge` (`DOGESTONIA_INSTRUCTIONS_*`), сохранив as-was для continuity.

## Target technical architecture (focus)

```text
GPT UI/
  instructions/                          # working as-was (Custom GPT continuity)
  instructions-archive-module1-req48/    # B2 snapshot baseline (name ≠ create until implement)
  instructions-roles-req15/              # B sibling: authoring SSOT + export-ready flat root children
```

**Flow (logical):**

1. **Snapshot** as-was Module-1 (+ active pack used in interview) → archive home.  
2. **Mapping** as-was → shared / role(s) / archive-only / server-only (AC-01; content = later authoring).  
3. **Author** shared core + five roles in sibling (ids §3b). Strip gateway stash + SPA redirect from **new** prose (AC-03; peer REQ-11).  
4. **Export Pass (E1):** local inventory + per-file sha256 + `bundle_version` — not joint wire SSOT.  
5. **Publish (V2):** manual copy to volume; env → roles-schema manifest.  
6. **Consume Pass:** aibridge / ops (out of GPT-48 DoD).

**Components & trust boundaries**

| Component | Trust / duty |
|-----------|----------------|
| GPT UI Git (three homes) | Authoring + archive + export source |
| Roles-schema manifest | Declares `core` + five role files only |
| Railway/local volume | Runtime root for aibridge; not GPT UI process |
| Custom GPT Actions | Continuity-only (**C2**); not second roadmap in REQ-48 |
| Schema packs | **P1** — outside roles volume; must appear explicitly in mapping |

Details: [01](./01-git-homes-and-ownership.md) · [02](./02-export-roles-schema-and-packs.md) · [03](./03-publish-verify-parity.md)

## Ownership / contracts

- **SSOT texts (new line):** `GPT UI/instructions-roles-req15/` after implement.  
- **Consumer contract:** REQ-48 §3b / REQ-13 §3b — do not invent alternate layout.  
- **Path rule:** root children only (`assembly.py` `_reject_root_child`).  
- **Role ids:** `communication_setup` · `interviewer` · `extractor` · `normalizer` · `checker`.  
- **Packaging note:** Git ownership ≠ image/volume packaging — publish is a separate seam (V2).

## Runtime / packaging / verify / env

- GPT UI does **not** run the interview server; packaging theme = **export + publish layout**.  
- Env (consumer): `DOGESTONIA_INSTRUCTIONS_DIR`, `DOGESTONIA_INSTRUCTIONS_MANIFEST` → roles-schema after cutover.  
- Verify: Export Pass = local E1 evidence; live volume = residual/ops.  
- Sync script GPT UI → aibridge: **not found** (REQ-48 §3) — no invent CI (rejected **V3**).

## Coexistence with as-is

- As-built [`00`–`05`](../README.md) describe Module-1 / Actions / Phase-A bridge delivery — remain valid for as-was channel.  
- Working `GPT UI/instructions/` stays readable; existing `instructions/archive/` = nested packs artifact only — **not** the REQ-48 Module-1 snapshot.  
- Phase-A `files[]` loader remains transitional on aibridge until ASGI role wire (peer residual).  
- Verified gap in as-built bridge wiring may still apply to Phase-A path — cite [`05-python-bridge-instruction-delivery.md`](../05-python-bridge-instruction-delivery.md); roles assembly harness Present in aibridge (`roles/assembly.py`).

## Open questions

- Joint inventory/hash wire format (GPT-48 Export ↔ REQ-13 Consume) — Open.  
- Custom GPT Actions long-term sunset — Open (outside REQ-48 DoD; **C2**).  
- Exact on-disk evidence filename for E1 — propose at implement; name ≠ create here.  
- Full Module-1 file list for snapshot — from index/implement inventory, not invented here.  
- Live Railway volume proof path — ops residual / Unknown in this doc.

## Not in this doc

- NEW STORY keys, pkg, task README, P3.  
- Creating `instructions-archive-*` / `instructions-roles-*` trees or instruction bodies.  
- Python role coordinator / dual-confirm / gateway clients.  
- Invented APIs, joint hash SSOT, or CI sync pipelines.

## Operator decision lock (2026-11-02)

| Theme | Decision |
|-------|----------|
| Git layout | **B** parallel sibling |
| Archive mechanics | **B2** snapshot copy (overrode B1) |
| Path names | **N2** (paths above) |
| Schema packs | **P1** outside roles volume |
| Publish | **V2** manual + checklist/hash |
| Export Pass evidence | **E1** local inventory+sha256 |
| Custom GPT | **C2** continuity-only |
