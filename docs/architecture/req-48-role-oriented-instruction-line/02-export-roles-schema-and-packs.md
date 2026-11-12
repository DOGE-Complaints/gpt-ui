# REQ-48 — Export roles-schema & schema packs

**Parent:** [00-overview.md](./00-overview.md)  
**As-of:** 2026-11-02T12:21:17Z  
**Contract SSOT:** [REQ-48 §3b](../../requirements/REQ-48-req15-role-oriented-instruction-line-and-export.md) = copy of [aibridge REQ-13 §3b](../../../../doge-ai-bridge/docs/requirements/REQ-13-REQ15-SEMANTIC-STATE-ROLES-INSTRUCTION-ASSEMBLY.md)

## Roles export surface (sibling root)

After implement, `GPT UI/instructions-roles/` is intended to hold **flat root children** suitable for direct use as `DOGESTONIA_INSTRUCTIONS_DIR`:

| Contract field | Value |
|----------------|-------|
| Manifest env | `DOGESTONIA_INSTRUCTIONS_MANIFEST` → roles-schema JSON |
| Schema | `bundle_version` (string) · `core` (filename) · `roles` (object) |
| Role ids | `communication_setup` · `interviewer` · `extractor` · `normalizer` · `checker` |
| Per role | `{ "file": "<root-child.md>", "version": "<pin>" }` |
| Path rule | No `/`, `\`, `.`, `..`, absolute paths |

**Consumer enforcement (as-is):** `doge-ai-bridge/src/aibridge/roles/assembly.py` `_reject_root_child` (approx. L38–44) · `load_roles_manifest` / `assemble_role_prompt`.

**Reference shape (fixture, not GPT UI SSOT):**  
[`doge-ai-bridge/tests/fixtures/content/roles.manifest.json`](../../../../doge-ai-bridge/tests/fixtures/content/roles.manifest.json) + `roles-instructions/*.md`.  
Authoring remains in GPT UI; fixtures are consume/harness reference only (**E3 rejected** as AC-06 oracle).

## Forbidden in new texts

Gateway stash URL choice, `POST /story-drafts`, SPA redirect procedure — peer **REQ-11**. Checker may keep **prose** about readiness; submit stays code (REQ-48 §3b).

## Schema packs (decision P1)

- Active schema pack **must** appear in the **mapping table** (AC-01) — no silent drop.  
- Pack files are **not** part of roles-schema export inventory (AC-05/06).  
- Typical classifications: archive-only / Custom GPT continuity / future bridge — chosen at mapping authoring, not invented here.  
- Do **not** extend roles.manifest with pack entries in this target arch.  
- As-built pack layer remains documented in [`../02-schema-packs-and-binding.md`](../02-schema-packs-and-binding.md) for Module-1/as-was.

## Phase-A `files[]`

Remains **legacy/transitional** readiness on aibridge until ASGI role wire. **Not** SSOT of the new export line (REQ-48 §3b). Concurrent dual schema on one env = peer wire residual, not authoring ambiguity if export follows this table.
