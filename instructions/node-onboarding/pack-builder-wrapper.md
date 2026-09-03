# Pack Builder — Synthetic Wrapper (pin primary + bind standards)

**Product:** DOGEstonia — Pack Builder (node onboarding)  
**Role of this file:** Thin **wrapper** that tells an operator / hosting surface how to **fetch/pin** the remote primary prompt and **bind** validation standards — then start the Pack Builder dialogue.  
**Not** the primary Pack Builder body. **Not** the repair/debug template. **Not** Module-1 Story Interview Instructions.  
**MUST NOT embed** the full primary instruction text here — pin by path/URL/tag only.  
**Traceability:** REQ-46 §5.0 wrapper · §6 · AC-GPT-REQ46-02/08 · GPT-PB-04 · GPT-PB-05 · STORY-GPT-PB-05  
**Version:** 1.1 · **Date:** 2026-09-03  

**Path:** `GPT UI/instructions/node-onboarding/pack-builder-wrapper.md`

**Index:** [`pack-builder-README.md`](./pack-builder-README.md)

---

## 0. Pin record (fill / publish with this wrapper)

| Field | Value |
|-------|--------|
| **Primary path (repo)** | `instructions/node-onboarding/pack-builder-primary-prompt.md` (git root = `gpt-ui`) |
| **Primary raw/tag URL** | `https://raw.githubusercontent.com/DOGE-Complaints/gpt-ui/dev/instructions/node-onboarding/pack-builder-primary-prompt.md` |
| **Primary pin tag / commit** | branch `dev` · commit `a8544cb0d85393b170f8ab966e9d598ffb583d0f` (GPT-PB-05 pin) |
| **`prompt_version_hash` (REQUIRED)** | `pb-primary-v1.1-20260903` — **MUST** match primary header; update when primary body changes |
| **Standards directory** | `instructions/node-onboarding/` |
| **`standards_set_hash` (RECOMMENDED)** | `sha256:258e13bfdd50e48ce980d4e4e12863e21c4c2896bc9bc675df562c5e7dd2f948` — sha256 of concat sha256 of: `pack-builder-overlays.checklist.md` + `pack-builder-pack.schema.json` + `pack-builder-payload-schema.schema.json` + `pack-builder-taxonomy.schema.json` (sorted by filename; concat per-file sha256 **hex** digests, then sha256); recompute when any standard changes |
| **Repair template path** | `pack-builder-repair-debug-prompt.md` |
| **Wrapper version** | `1.1` · `pb-wrapper-v1.1-20260903` |

Raw URLs are **public** (no auth). To freeze a release, pin a **tag** or immutable commit SHA instead of floating `dev`.

---

## 1. Bind validation standards (PB-03)

Before dialogue, bind these files (paths relative to `node-onboarding/` **and** remote raw URLs for вне-IDE):

| File (local) | Remote raw URL |
|--------------|----------------|
| `pack-builder-pack.schema.json` | `https://raw.githubusercontent.com/DOGE-Complaints/gpt-ui/dev/instructions/node-onboarding/pack-builder-pack.schema.json` |
| `pack-builder-payload-schema.schema.json` | `https://raw.githubusercontent.com/DOGE-Complaints/gpt-ui/dev/instructions/node-onboarding/pack-builder-payload-schema.schema.json` |
| `pack-builder-taxonomy.schema.json` | `https://raw.githubusercontent.com/DOGE-Complaints/gpt-ui/dev/instructions/node-onboarding/pack-builder-taxonomy.schema.json` |
| `pack-builder-overlays.checklist.md` | `https://raw.githubusercontent.com/DOGE-Complaints/gpt-ui/dev/instructions/node-onboarding/pack-builder-overlays.checklist.md` |

Primary **MUST** generate only candidates that pass these checks (when bound). Record `standards_set_hash` alongside `prompt_version_hash` on emit.

---

## 2. Operator flow (REQ-46 §6.1–6.4)

```text
1) OBTAIN  — this wrapper + pinned primary + standards (+ README index)
2) PIN     — lock primary URL/tag + prompt_version_hash; bind standards + optional standards_set_hash
3) DIALOGUE — run Pack Builder using **primary** body (not this wrapper)
4) EMIT    — flat schema-packs.<id>.<ver>.* six-file set (REQ-45a) + hashes
5) VALIDATE — against node-onboarding meta-schemas + overlays checklist
6) REPAIR  — on fail: fill pack-builder-repair-debug-prompt.md → paste into same dialogue → regenerate → re-validate
7) ADMIT   — success → REQ4 / REQ-APP-04 app·gateway gates (authoritative; wrapper does not replace them)
```

**Gateway wins** on federation drift. **MUST NOT** rewrite Module-1 process / OpenAPI.

---

## 3. Duties — what this wrapper does / does not

| DOES | DOES NOT |
|------|----------|
| Name pin target for primary | Embed full primary Pack Builder prose |
| Bind standards paths + recommend `standards_set_hash` | Replace repair template or meta-schema bodies |
| Require `prompt_version_hash` on session/emit | Host raw URLs for the operator (app F1 = REQ-APP-04) |
| Point to §2 flow | Merge Pack Builder into Module-1 interview Instructions |

---

## 4. Sibling pointers (no stubs)

| Artifact | Path |
|----------|------|
| README index | `pack-builder-README.md` |
| Primary | `pack-builder-primary-prompt.md` |
| Repair | `pack-builder-repair-debug-prompt.md` |
| Standards | three `*.schema.json` + `pack-builder-overlays.checklist.md` |

If a sibling is missing, **do not** invent empty placeholders — stop and obtain the real file (PB-01…03).
