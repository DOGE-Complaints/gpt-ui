# P4 Code Audit — STORY-GPT-ROL-04 (export roles-schema + parity + ops)

**Дата:** 2026-11-02  
**UTC verify:** 2026-11-02T14:43:09Z  
**Метод:** [`.cursor/rules/analysis.mdc`](../../../../../.cursor/rules/analysis.mdc) — claims с путями; findings only; RO (no product edits)  
**builder_project:** gpt  
**Объект (backlog):** [`STORY-GPT-ROL-04-export-roles-schema-parity-ops.md`](../../../tasks/backlog-stories/req-48-role-oriented-instruction-line/STORY-GPT-ROL-04-export-roles-schema-parity-ops.md)  
**Pipeline:** [`…/STORY-GPT-ROL-04-….md`](../epics/EPIC-M1-12-req48-role-oriented-instruction-line/stories/STORY-GPT-ROL-04-export-roles-schema-parity-ops/STORY-GPT-ROL-04-export-roles-schema-parity-ops.md)  
**P3 claim:** GIM-462…467 🟢 · UTC `2026-11-02T14:39:55Z`  
**Pkg:** [`pkg-000067-…yaml`](../gpt-active-packages/pkg-000067-20261102-gpt-rol-04-export-roles-schema-parity-ops.yaml) · pointer [`gpt-active-package.current.yaml`](../gpt-active-package.current.yaml)  
**Scope:** Findings only. Write = this audit path. Phases not advanced. Do not invent AC.

**Вердикт:** vs-AC **3/3 PASS** (AC-GPT-ROL04-01…03).  
**OPEN blocking:** **0**.  
**OPEN residual (story-accepted):** joint REQ-13 inventory/hash wire — **Open** (does **not** block local Export Pass).  
**WAVE COMPLETE:** **yes** (blocking OPEN=0 · product ACs PASS · residual documented per T06).

---

## 1. vs-AC (story ACs only)

| AC | Итог | Verified path / fact |
|----|------|----------------------|
| **AC-GPT-ROL04-01** (= AC-GPT-48-05): Export layout documented/produceable as roles-schema; Phase-A `files[]` ≠ new-line SSOT | **PASS** | [`roles.manifest.json`](../../../instructions-roles-req15/roles.manifest.json) Present · `bundle_version` + `core` + 5×`{file,version}` · root-child filenames · `load_roles_manifest` OK · env `DOGESTONIA_INSTRUCTIONS_DIR` / `_MANIFEST` in evidence + operator note · Phase-A `files[]` **not** SSOT wording Present |
| **AC-GPT-ROL04-02** (= AC-GPT-48-06): Export parity Pass evidence (E1); no silent subset | **PASS** | [`req-48-export-parity-evidence.md`](../../req-48-export-parity-evidence.md) Present · intended inventory 7/7 · live recompute sha256 after `normalize_text` **7/7 MATCH** · labels ≠ joint wire SSOT / ≠ Consume Pass / ≠ `session_pin` |
| **AC-GPT-ROL04-03** (= AC-GPT-48-07): Operator note Present — archive vs new line; manual publish | **PASS** | [`req-48-roles-line-publish-operator-note-ru.md`](../../runtime-docs/req-48-roles-line-publish-operator-note-ru.md) Present · three Git homes · V2 manual steps · no CI sync · cross-link in [`runtime-docs/README.md`](../../runtime-docs/README.md) · arch [`03-publish-verify-parity.md`](../../architecture/req-48-role-oriented-instruction-line/03-publish-verify-parity.md) Related |

Gate: [`story-acceptance-gate.md`](../epics/EPIC-M1-12-req48-role-oriented-instruction-line/stories/STORY-GPT-ROL-04-export-roles-schema-parity-ops/story-acceptance-gate.md) PASS · GIM-462…467 AV **6/6 Present**.

---

## 2. Fact verify (product)

| Check | Result |
|-------|--------|
| Manifest Present + §3b shape | **Y** — `req15-roles-v1` · 5 role ids · files exist · no `/` in paths |
| aibridge `load_roles_manifest` | **OK** on sibling manifest |
| E1 hashes vs disk | **7/7 MATCH** (UTF-8 + LF → sha256) |
| Packs in roles-schema inventory | **excluded** (P1) |
| Env cutover wording | **Present** (evidence + operator note) |
| Operator three homes + V2 steps | **Present** |
| Cross-links T05 | **Present** (runtime README + arch 03) |

---

## 3. OPEN — blocking vs residual

### Blocking GAP (fail AC / incomplete TASKED)

| ID | Severity | Finding |
|----|----------|---------|
| — | — | **none** |

### Story-accepted residual Open (not blocking WAVE)

| Item | Severity | Fate |
|------|----------|------|
| Joint inventory/hash wire with REQ-13 | Open (soft) | Explicit T06 / Dependencies · evidence + note label **No** joint SSOT · do **not** invent close |
| Live Railway Consume Pass | OOS | Peer aibridge; not AC-01…03 |

**Not OPEN (Info residual):**

| Item | Fate |
|------|------|
| `README.md` outside E1 consume inventory | Intentional (evidence states authoring prose outside) |
| CI sync GPT UI → aibridge image (V3) | Rejected / OOS |

---

## 4. Регрессии

| Инвариант | Состояние |
|-----------|-----------|
| Sibling role bodies (ROL-03) | **intact** (hashes match evidence) |
| Mapping / archive / working C2 | **not emptied** this wave |
| Joint wire invented? | **N** |
| Consume Pass claimed? | **N** |

---

## 5. Bullrun / pkg (read-only)

| Claim | Disk | P4 |
|-------|------|-----|
| active pkg-000067 | pointer + YAML | **ok** |
| GIM-462…467 🟢 | 6 AV + gate | **подтверждён** |
| Manifest / evidence / note Present | three locked paths | **подтверждён** |

Silent skip / incomplete TASKED: **none**.

---

## 6. Counts

| Metric | Value |
|--------|-------|
| vs-AC PASS | **3/3** |
| vs-AC FAIL | **0** |
| OPEN blocking | **0** |
| OPEN residual (story-accepted) | **1** (REQ-13 joint wire) |
| WAVE COMPLETE | **yes** |
| E1 hash match | **7/7** |

---

## 7. Handoff (OP fence)

VAL wrote audit. Phases **not** advanced. No product edits. Residual REQ-13 joint wire stays Open for peer epic — not a GPT-ROL-04 P5 blocker. Next: P8 commits / package INDEX as operator directs.
