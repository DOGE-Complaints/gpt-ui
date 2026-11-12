# REQ-48 — Publish, verify, export parity

**Parent:** [00-overview.md](./00-overview.md)  
**As-of:** 2026-11-02T12:21:17Z

## Packaging seam (Git ≠ volume)

| Layer | Meaning |
|-------|---------|
| Git sibling | Authoring SSOT + intended export set |
| Publish | Manual copy of sibling root → configured `DOGESTONIA_INSTRUCTIONS_DIR` |
| Volume / env | What aibridge reads at runtime (peer) |

**Decision V2:** v1 publish is **manual** (SSH / Railway volume UI / equivalent). Operator note documents steps (AC-07). Optional local hash checklist/helper — only when implement explicitly orders a script (**name ≠ create** in this arch wave). **V3 CI sync rejected** (script not found; invent pipeline out of scope).

## Export Pass vs Consume Pass

| Gate | Owner | Meaning |
|------|-------|---------|
| **Export Pass** (AC-GPT-48-06) | GPT UI | Released set matches **intended new-line inventory** |
| **Consume Pass** (REQ-13 AC-05) | aibridge / ops | Mounted dir matches export + fail-closed |
| Joint inventory/hash wire | **Open** | Do not invent SSOT format here |

## Local Export Pass evidence (decision E1)

Target evidence pack (logical; create at implement):

1. List of intended root children (`core` + five role files + roles-schema manifest filename).  
2. Per-file `sha256` of file bytes **after** the same text normalization consumer uses when hashing text: UTF-8 decode + newlines → LF (`normalize_text` in `doge-ai-bridge/src/aibridge/content.py` L23–26; `sha256_hex` L19–20).  
   - **Note:** Export inventory hashing of *files* vs runtime `session_pin` (assembly of core+role) are different concepts — do not conflate.  
3. Record `bundle_version` from manifest.  
4. Explicit label: **not** joint wire SSOT; **not** Consume Pass.

Suggested homes at implement (pick one; do not create now):

- `GPT UI/docs/analysis/req-48-export-parity-evidence-*.md`, or  
- a file beside `instructions-roles/` after that tree exists.

## Verify surfaces

| Surface | What it proves |
|---------|----------------|
| Local E1 checklist on Git sibling | Export Pass candidate |
| Peer pytest fixtures / stub parity tests | Harness only — not live volume Pass |
| Live volume + ops window | Consume Pass (peer); residual for GPT-48 |

Peer honesty note: [`doge-ai-bridge/docs/runtime-docs/11-content-volume-parity-stub-live-evidence.md`](../../../../doge-ai-bridge/docs/runtime-docs/11-content-volume-parity-stub-live-evidence.md).

## Fail-closed

Incomplete/mismatched **consume** readiness is aibridge REQ-13 duty. GPT UI operator note should still warn: publishing a subset without updating intended inventory fails Export Pass honesty (AC-06).

## Related (implement)

| Artifact | Path |
|----------|------|
| Operator note (V2) | [`../../runtime-docs/req-48-roles-line-publish-operator-note-ru.md`](../../runtime-docs/req-48-roles-line-publish-operator-note-ru.md) |
| Export Pass evidence (E1) | [`../../analysis/req-48-export-parity-evidence.md`](../../analysis/req-48-export-parity-evidence.md) |
| Roles-schema manifest | [`../../../instructions-roles/roles.manifest.json`](../../../instructions-roles/roles.manifest.json) |
