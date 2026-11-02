# STORY-GPT-ROL-04 — Roles-schema export, Export Pass, operator publish note

## Meta
- **Key:** `STORY-GPT-ROL-04-export-roles-schema-parity-ops`
- **Пакет:** [`req-48-role-oriented-instruction-line/`](./INDEX.md)
- **Status:** Todo
- **Приоритет:** P1
- **Тип:** export layout + parity evidence + operator docs
- **Parent REQ:** [`REQ-48`](../../../requirements/REQ-48-req15-role-oriented-instruction-line-and-export.md) **AC-GPT-48-05**, **AC-GPT-48-06**, **AC-GPT-48-07** · §3b
- **Tech-arch:** [`02-export-roles-schema-and-packs`](../../../architecture/req-48-role-oriented-instruction-line/02-export-roles-schema-and-packs.md) · [`03-publish-verify-parity`](../../../architecture/req-48-role-oriented-instruction-line/03-publish-verify-parity.md) (**V2**, **E1**)
- **Siblings:** [GPT-ROL-01](./STORY-GPT-ROL-01-archive-module1-snapshot.md) · [GPT-ROL-02](./STORY-GPT-ROL-02-module1-role-mapping.md) · [GPT-ROL-03](./STORY-GPT-ROL-03-author-roles-line-strip-stash.md)
- **Skill declared:** `openai-custom-gpt-builder`
- **Depends:** GPT-ROL-03
- **Method:** [`.cursor/rules/analysis.mdc`](../../../../../.cursor/rules/analysis.mdc)

## Зачем простыми словами

Новая линия должна лежать так, как ждёт aibridge (roles-schema + flat root children), с доказательством что выпустили **весь** задуманный набор, и короткой инструкцией оператору: куда копировать на volume руками.

## Scope

- Roles-schema manifest in sibling (`bundle_version`, `core`, `roles.*` with `file` + `version`); document env: `DOGESTONIA_INSTRUCTIONS_DIR` + `DOGESTONIA_INSTRUCTIONS_MANIFEST`.
- **Export Pass (E1):** intended inventory + per-file sha256 after text normalization aligned with aibridge `normalize_text` (UTF-8, newlines→LF — `doge-ai-bridge/src/aibridge/content.py`); label evidence as **not** joint wire SSOT and **not** Consume Pass.
- Operator note: three Git homes; archive vs new line; **manual** publish-to-volume (V2); optional hash checklist (script body only if later ordered — name ≠ create here).

## Out of scope

- CI sync GPT UI → aibridge image (arch **V3** rejected; sync script not found — REQ-48 §3).
- Inventing joint inventory/hash SSOT with REQ-13.
- Live Railway Consume Pass (peer AC-05).
- Phase-A `files[]` as new-line SSOT.

## Verified current state

- §3b locked in REQ-48 / REQ-13.
- Sync script **not found** (REQ-48 §3).
- Export bodies / parity evidence **Absent**.

## Target / Acceptance Criteria

- [ ] **AC-GPT-ROL04-01** (= **AC-GPT-48-05**): Export layout documented and produceable as roles-schema; Phase-A `files[]` is not the new-line SSOT.
- [ ] **AC-GPT-ROL04-02** (= **AC-GPT-48-06**): Export parity Pass evidence Present (E1); no silent subset vs intended inventory.
- [ ] **AC-GPT-ROL04-03** (= **AC-GPT-48-07**): Operator note Present — archive vs new line; publish-to-volume steps (manual v1).

## Подзадачи (технические)

> **Delivery class этой волны story-deepen:** только постановка T01–T06. Manifest / evidence / operator-note bodies **name ≠ create** until P1.3/P3. **No** CI sync script (V3 rejected).

| ID | Task | AC | Done when |
|----|------|-----|-----------|
| **T01** | Add `GPT UI/instructions-roles-req15/roles.manifest.json` — roles-schema §3b: `bundle_version` (string), `core` → `core.md`, `roles` for five ids each `{ "file", "version" }` with root-child filenames from ROL-03; start pins **`v1`** unless mapping says otherwise. Shape ref: fixture (not oracle). | AC-05 | Manifest Present + valid shape |
| **T02** | Document env cutover in operator note / evidence header: `DOGESTONIA_INSTRUCTIONS_DIR` → sibling root; `DOGESTONIA_INSTRUCTIONS_MANIFEST` → roles-schema path; Phase-A `files[]` **not** new-line SSOT. | AC-05, AC-07 | Wording Present |
| **T03** | **E1 Export Pass evidence** at `GPT UI/docs/analysis/req-48-export-parity-evidence.md`: intended inventory (manifest + six md); per-file sha256 after UTF-8 + LF (`normalize_text` in `doge-ai-bridge/src/aibridge/content.py` L23–26) then `sha256_hex`; record `bundle_version`; labels **≠ joint wire SSOT**, **≠ Consume Pass**. Do not conflate with runtime `session_pin`. | AC-06 | Evidence file Present; hashes match files |
| **T04** | **Operator note (V2)** at `GPT UI/docs/runtime-docs/req-48-roles-line-publish-operator-note-ru.md`: three Git homes; archive vs new line; manual publish-to-volume steps; warn against subset publish without updating inventory; **no** CI/image sync script. | AC-07 | Note Present |
| **T05** | Cross-link: row in [`runtime-docs/README.md`](../../../runtime-docs/README.md); pointer from arch [`03-publish-verify-parity.md`](../../../architecture/req-48-role-oriented-instruction-line/03-publish-verify-parity.md) if thin Related needed. | AC-07 | Links Present |
| **T06** | **Gate:** AC-05/06/07; joint inventory/hash wire with REQ-13 remains **Open** (do not invent). | AC-05…07 | Checklist Pass |

## Locked paths (tech-arch)

| Role | Path |
|------|------|
| Manifest | `GPT UI/instructions-roles-req15/roles.manifest.json` |
| Export evidence | `GPT UI/docs/analysis/req-48-export-parity-evidence.md` |
| Operator note | `GPT UI/docs/runtime-docs/req-48-roles-line-publish-operator-note-ru.md` |

## Точки в репо

- Manifest + role files under `GPT UI/instructions-roles-req15/` (from ROL-03).
- Evidence / operator note: locked paths above.
- Peer shape reference: fixtures only.

## Dependencies

- Hard: GPT-ROL-03. Soft: joint wire Open with REQ-13 (does not block local Export Pass).

## Related

| Artifact | Role |
|----------|------|
| REQ-48 AC-05…07 | Export + parity + ops |
| aibridge runtime-doc 11 | Consume Partial honesty |
| Arch 03 | V2 / E1 |
