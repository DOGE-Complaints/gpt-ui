# Pack Builder — Repair / Debug Prompt (validation-fail loop)

**Product:** DOGEstonia — Pack Builder (node onboarding)  
**Role of this file:** Fill-in **repair / debug** template the operator pastes back into the Pack Builder dialogue after a **candidate validation fail**.  
**Not** the primary Pack Builder body. **Not** the wrapper. **Not** Module-1 Story Interview Instructions.  
**Traceability:** REQ-46 §5.0 repair · §5.1.6 · §6.3–4 · AC-GPT-REQ46-02 · GPT-PB-02 · STORY-GPT-PB-02  
**Version:** 1.0 · **Date:** 2026-09-03  
**prompt_version_hash:** `pb-repair-v1.0-20260903` (update when this body changes)

**Path:** `GPT UI/instructions/node-onboarding/pack-builder-repair-debug-prompt.md`

**Companion (read-only):** `pack-builder-primary-prompt.md` §5 Repair loop hook — primary **accepts** a filled copy of this template and **re-emits** the same flat six-file set.

---

## 0. Purpose (operator)

Use this template when:

1. Pack Builder (primary) emitted a candidate flat `schema-packs.<schema_id>.<schema_version>.*` set, **and**
2. Validation against `node-onboarding/` standards (and/or operator/app checklist) **failed**.

Fill the slots below → paste the completed block into the **same** Pack Builder dialogue → primary regenerates the **same flat file set** with corrections → **re-validate**.

This loop is a **candidate authoring** repair only (REQ-46 F2 authoring side). It does **not** replace federation admission gates (see §4).

---

## 1. How to use (fail → repair → regenerate → re-validate)

```text
emit (primary) → validate vs node-onboarding standards (when present)
  → FAIL → fill this repair template → paste into Pack Builder dialogue
  → primary re-emits flat schema-packs.* only (same id/ver)
  → re-validate → success OR iterate again
```

| Step | Actor | Action |
|------|-------|--------|
| 1 | Validator / operator | Capture fail output (errors, missing files, checklist misses) |
| 2 | Operator | Fill §2–§3 slots in this template |
| 3 | Operator | Paste filled template into Pack Builder (primary) dialogue |
| 4 | Primary | Re-elicit failing parts; **regenerate** flat `schema-packs.*` set only (REQ-45a) |
| 5 | Operator | Re-run validation; if fail — repeat from step 1 |

**Wrapper orchestration** (fetch/pin primary + standards hashes) is finalized in GPT-PB-04 — name only here; do not invent wrapper body in this file.

**MUST NOT** “fix” validation by editing Module-1 interview process files (`story-interview-flow`, `api-orchestrator`, OpenAPI Actions, etc.).

---

## 2. Fill-in slots (operator completes)

Copy from `---` to `---` and fill every `[[…]]` slot.

---

### REPAIR / DEBUG — Pack Builder candidate

**schema_id:** `[[SCHEMA_ID]]`  
**schema_version:** `[[SCHEMA_VERSION]]`  
**prompt_version_hash (primary used):** `[[PRIMARY_PROMPT_VERSION_HASH]]`  
**standards_set_hash (if known):** `[[STANDARDS_SET_HASH_OR_PENDING]]`  
**Validation run id / timestamp:** `[[VALIDATION_RUN_ID_OR_UTC]]`

#### 2.1 Validator / checklist errors (paste verbatim)

```text
[[PASTE_VALIDATOR_ERRORS_AND_CHECKLIST_FAILS_HERE]]
```

#### 2.2 Which flat artifacts failed (check all that apply)

- [ ] `schema-packs.<id>.<ver>.pack.json`
- [ ] `schema-packs.<id>.<ver>.payload.schema.json`
- [ ] `schema-packs.<id>.<ver>.taxonomy.json`
- [ ] `schema-packs.<id>.<ver>.inbound-validation.md` (content / admission)
- [ ] `schema-packs.<id>.<ver>.interview-overlay.md` (interview guidance)
- [ ] `schema-packs.<id>.<ver>.locale-jurisdiction.md` (language / jurisdiction)
- [ ] Missing / wrong **flat** naming (nested path emitted)
- [ ] Other: `[[OTHER]]`

#### 2.3 Standards paths checked (bind by path; bodies may arrive with GPT-PB-03)

When present under `GPT UI/instructions/node-onboarding/`, cite which failed:

| Standard path | Used? | Fail note |
|---------------|-------|-----------|
| `pack-builder-pack.schema.json` | `[[Y/N/PENDING]]` | `[[NOTE]]` |
| `pack-builder-payload-schema.schema.json` | `[[Y/N/PENDING]]` | `[[NOTE]]` |
| `pack-builder-taxonomy.schema.json` | `[[Y/N/PENDING]]` | `[[NOTE]]` |
| `pack-builder-overlays.checklist.md` | `[[Y/N/PENDING]]` | `[[NOTE]]` |

If standards are **absent (PB-03 pending):** still request a corrected flat six-file emit; mark candidate `standards_pending_PB-03`; re-validate when standards land.

#### 2.4 Operator intent for the fix (plain language)

```text
[[WHAT_SHOULD_CHANGE_IN_THE_PACK_IN_NL]]
```

#### 2.5 Explicit instructions to Pack Builder (primary)

1. Treat this message as a **filled repair / debug prompt** from `pack-builder-repair-debug-prompt.md`.
2. Do **not** start a new unrelated pack; keep `schema_id` / `schema_version` above unless the fail requires a deliberate rename (then state both old and new).
3. Re-elicit **only** the failing parts; keep passing parts stable where possible.
4. **Regenerate the same flat REQ-45a six-file set** (filenames only — no nested GPT upload paths):

   | Flat filename | Duty |
   |---------------|------|
   | `schema-packs.<id>.<ver>.pack.json` | Pair; field_policy; geo_intake; clustering/card |
   | `schema-packs.<id>.<ver>.payload.schema.json` | Payload / signals / geo schema |
   | `schema-packs.<id>.<ver>.taxonomy.json` | When taxonomy is required |
   | `schema-packs.<id>.<ver>.inbound-validation.md` | Content / admission |
   | `schema-packs.<id>.<ver>.interview-overlay.md` | Interview guidance |
   | `schema-packs.<id>.<ver>.locale-jurisdiction.md` | Language / jurisdiction |

5. Emit corrected file bodies + `prompt_version_hash` (and `standards_set_hash` if known).
6. **MUST NOT** rewrite Module-1 process Instructions / OpenAPI to “pass” validation.
7. **MUST NOT** place instance pack files under `instructions/node-onboarding/`.

---

## 3. Regenerated emit contract (for primary)

After accepting a filled repair prompt, primary **MUST**:

- Re-emit **flat** `schema-packs.*` only (REQ-45a / REQ-46 §5.1.4 / primary §3).
- Preserve three pack prose identities when GPT-facing (inbound + overlay + locale).
- Align with `node-onboarding` meta-schemas / overlays checklist **when present**.
- Keep **SEC-001:** pack JSON = data (read-only reference); GPT prose is not SchemaRuntime.

Primary **MUST NOT**:

- Switch to nested `schema-packs/<id>/<ver>/` for Custom GPT upload layout.
- Edit Module-1 interview process files as the repair.
- Claim that this repair loop **is** app or gateway admission.

---

## 4. Non-replacement of app / gateway gates (AC-GPT-PB02-03)

**MUST NOT replace** any of the following:

| Gate | What it is | This repair loop |
|------|------------|------------------|
| **dogestonia-app** REQ-APP-04 F2 (and related) | App-side validate / repair UX for node onboarding | Candidate Pack Builder authoring only |
| **REQ4 F2–F5** federation admission | Product federation gate after candidate readiness | Outside Pack Builder tooling |
| **Gateway SchemaRuntime** validate | Executable pack validate on gateway | Gateway wins on drift; Pack Builder does not pretend to be SchemaRuntime |

Operator next step after a **successful** re-validate against `node-onboarding` standards: proceed to **app/gateway admission** paths (REQ4 / REQ-APP-04) — those gates remain authoritative.

---

## 5. Cross-links (SSOT pointers)

| Concern | SSOT |
|---------|------|
| Primary accept / re-emit hook | `pack-builder-primary-prompt.md` §5 |
| Flat layout + three prose identities | REQ-45a |
| Pack Builder tooling / repair duty | REQ-46 §5.0 · §5.1.6 · §6.3–4 |
| App validate / repair (not this file) | REQ-APP-04 F2 |
| Federation admission (not this file) | REQ4 F2–F5 |
| Wrapper pin + hashes | GPT-PB-04 (future) |
| Validation meta-schema bodies | GPT-PB-03 (future; paths named in §2.3) |

---

*End of Pack Builder repair / debug prompt v1.0*
