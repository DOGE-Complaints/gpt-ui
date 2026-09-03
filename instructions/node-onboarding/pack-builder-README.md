# Pack Builder — README (node-onboarding index)

**Product:** DOGEstonia — Pack Builder tooling under `GPT UI/instructions/node-onboarding/`  
**Role of this file:** Short index of Pack Builder siblings + bans + emit SSOT.  
**Not** an instance schema pack. **Not** Module-1 Story Interview Instructions.  
**Traceability:** REQ-46 §5.0 README · AC-GPT-REQ46-01/05/07/12 · REQ-45a · GPT-PB-04 · GPT-PB-05  
**Version:** 1.1 · **Date:** 2026-09-03  

**Path:** `GPT UI/instructions/node-onboarding/pack-builder-README.md`

---

## 1. Pack Builder vs interview Custom GPT

| | Pack Builder (`node-onboarding/`) | Module-1 interview Instructions |
|--|-----------------------------------|----------------------------------|
| Audience | Node operator authoring a **candidate** pack | Resident interview / story intake |
| Output | Flat `schema-packs.<id>.<ver>.*` candidates | Dialogue + Actions against active pack |
| Process edits | **Forbidden** to rewrite Module-1 / OpenAPI | Owns interview **process** |
| Runtime layout SSOT | **Consumer** of [REQ-45a](../../docs/requirements/REQ-45a-pack-prose-identities-and-flat-layout.md) | Consumes flat pack set at runtime |

Pack Builder is **orthogonal** to Module-1. Do not merge these trees.

---

## 2. Sibling index (present on disk)

| File | Duty |
|------|------|
| [`pack-builder-wrapper.md`](./pack-builder-wrapper.md) | Pin primary + bind standards + hashes + operator flow |
| [`pack-builder-primary-prompt.md`](./pack-builder-primary-prompt.md) | Full NL → flat six-file emit Pack Builder |
| [`pack-builder-repair-debug-prompt.md`](./pack-builder-repair-debug-prompt.md) | Validation-fail repair template |
| [`pack-builder-pack.schema.json`](./pack-builder-pack.schema.json) | Meta-schema for candidate `pack.json` |
| [`pack-builder-payload-schema.schema.json`](./pack-builder-payload-schema.schema.json) | Meta-schema for candidate `payload.schema.json` |
| [`pack-builder-taxonomy.schema.json`](./pack-builder-taxonomy.schema.json) | Meta-schema for candidate `taxonomy.json` |
| [`pack-builder-overlays.checklist.md`](./pack-builder-overlays.checklist.md) | Three prose identities presence/shape checklist |

**Hashes:** see wrapper §0 — `prompt_version_hash` **required**; `standards_set_hash` **recommended**.  
**Remote pins (вне IDE):** SSOT = [`pack-builder-wrapper.md`](./pack-builder-wrapper.md) §0 (primary raw URL + commit) and §1 (standards raw URLs). Same URLs mirrored in primary §1 and repair §2.3.

---

## 3. Bans (AC-05 / AC-07 / AC-12)

| MUST | MUST NOT |
|------|----------|
| Keep Pack Builder prompts + validation standards **in** `instructions/node-onboarding/` | Place **instance** `schema-packs.*` files under `node-onboarding/` |
| Emit flat `schema-packs.<id>.<ver>.*` per **REQ-45a** (JSON + three prose identities) | Invent a parallel layout SSOT in Pack Builder docs |
| Treat REQ-45a as runtime layout/identity SSOT (Pack Builder = consumer) | Create empty stub siblings “so the path exists” |
| Stay orthogonal to Module-1 process | Rewrite interview core / OpenAPI from Pack Builder |

---

## 4. Quick start

1. Open [`pack-builder-wrapper.md`](./pack-builder-wrapper.md) §0 — primary raw URL is filled; pin commit/tag as needed.  
2. Load pinned **primary** into the Pack Builder GPT / dialogue (fetch from raw URL if вне IDE).  
3. Bind the four standards from wrapper §1 (local paths and/or raw URLs).  
4. Emit flat six-file set → validate → repair loop if needed → app/gateway admission.

**Gateway wins** on federation drift. App hosting of raw URLs = REQ-APP-04 F1 (out of this README).
