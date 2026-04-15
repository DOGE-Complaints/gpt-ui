# Instruction Modules Index

| Поле | Значение |
|------|----------|
| **Версия индекса** | 0.28 |
| **Дата** | 2026-04-10 |
| **STORY** | GM1-03 + REQ-11 A–B (GM1-05) + **GM1-06** (`activity-legacy-paths-inventory`) + **GM2-01…06** (`issue-interview-flow` §12 REQ-04.2, §7.1, REQ-12/13, overlays) + **GM3-01…06** (EPIC-M1-03: matrix + FR-M1-019 + REQ-16 Q3 + Phase 7 §7.2 + FR-M1-024…027 / FR-M1-025 + **`issue-i18n-policy.md`** FR-M1-028…031) + **GM4-01** (`issue-policy-gate.md` scaffold, **GIM-20**) + **GM4-02** (strict chain handoff lifecycle / `ingest-validation` / `base`, **GIM-21**) + **GM4-03** ([`operator-rulebook-template.md`](../docs/analysis/tasks/epics/EPIC-M1-04-policy-gate-operator-rulebook/artifacts/operator-rulebook-template.md) v0.2, **GIM-22**) + **GM4-04** (`safety-compliance` Issue overlay + REQ-14 checklist, **GIM-23**) + **GM5-01** ([`issue-normalizer.md`](./issue-normalizer.md) v0.1.2, **`normalized_issue_payload`**, **GIM-24**) + **GM5-02** (`base` §1.5 Issue artifact + `ingest-validation` post-gate + `safety-compliance` Point 4 Issue + `issue-lifecycle-instructions` v0.6, **GIM-25**) + **GM5-03** (`api-orchestrator` Issue input from `normalized_issue_payload`, `activity-normalizer` deprecation, FR-M1-035…037 map, **GIM-26**) + **M6-02** (`issue-api-methods-reference` v0.3 parity matrix + `api-orchestrator` lockstep section + Issues OpenAPI YAML `0.1.1`, **GIM-27**) + **M6-03** (`api-orchestrator` Issue execution contract: pre-flight stop-the-line + response-truth, **GIM-28**) + **M6-04** (`issue-api-methods-reference` v0.4 auth/version lock policy + Issues OpenAPI YAML `0.1.2` + operator checklist `operator-actions-checklist-M6-04.md`, **GIM-29**) |

This document lists instruction modules in the repository. **Two tracks:**

1. **Amanita reference (Activities)** — file names and much of the body text still describe the donor **Activity** domain; kept for pattern reuse.
2. **DOGEstonia / Issue (Module 1)** — target domain is **Issue** (`spa-app`), per `GPT UI/docs/requirements/REQ-11-instruction-blocks.md` (catalog level) and `GPT UI/docs/technical-architecture.md` §7.

For Custom GPT: **instructions** carry behavior; this file is a **navigation map** — use together with [`issue-data-model.md`](./issue-data-model.md), [`issue-lifecycle-instructions.md`](./issue-lifecycle-instructions.md), [`issue-interview-flow.md`](./issue-interview-flow.md), [`issue-i18n-policy.md`](./issue-i18n-policy.md) (FR-M1-028…031), [`issue-policy-gate.md`](./issue-policy-gate.md) (operator rulebook admission, **EPIC-M1-04**), and [`issue-normalizer.md`](./issue-normalizer.md) (`normalized_issue_payload`, **EPIC-M1-05** / **GM5-01**).

---

## DOGEstonia — Issue (Module 1, parallel track)

### Canonical Issue docs (in repo)

| File | Role |
|------|------|
| [`issue-data-model.md`](./issue-data-model.md) | Logical **Issue** fields (`spa-app`-aligned). Not Activity. |
| [`issue-lifecycle-instructions.md`](./issue-lifecycle-instructions.md) | Ingest pipeline and strict artifacts from the modules’ viewpoint; not `ISSUE_STATUS` semantics. **§2.1** mandatory Issue order: validation → safety → **`issue-policy-gate`** → normalization → API (**GM4-02**). |
| [`issue-interview-flow.md`](./issue-interview-flow.md) | **REQ-08** + **§7.2** FR-M1-032…034 + **§8** row 9 (FR-M1-025 / REQ-15.3) + **§12** REQ-04.2 latent + limited-depth + **`safety-compliance` / FR-M1-039…043** (**GM4-04**) + **§7.1** FR-M1-017 + REQ-05/06/07 + REQ-12/13 (**v0.12**); [`issue-i18n-policy.md`](./issue-i18n-policy.md) (**GM3-06**); REQ-16 Q3 ADR (**GM3-03**); overlays GM2-03. **EPIC-M1-02** + **EPIC-M1-03** + **EPIC-M1-04** (§12 gate alignment). |
| [`issue-api-methods-reference.md`](./issue-api-methods-reference.md) | **HTTP SSOT for Issue** (paths, operationIds, request/response parity matrix, explicit app-level bearer assumptions, lockstep version policy; link to Actions YAML `0.1.2`); lock source documented until live node `/openapi.json` is available (**EPIC-M1-06**). |
| [`issue-i18n-policy.md`](./issue-i18n-policy.md) | **FR-M1-028…031**: session language (`bootstrap` / `ui_lang`), trilingual `{ et, ru, en }` drafts, fidelity vs polish, no meaning distortion; v0.1 (**GM3-06** / **GIM-19**). |
| [`issue-policy-gate.md`](./issue-policy-gate.md) | **EPIC-M1-04**: admission gate for Issue ingest; external operator rulebook via `policy_ref` + `rulebook_version`; `policy_gate_result`; **no API**; degraded mode without OP-DOC (**GM4-01** / **GIM-20**; **GM4-02** chain **GIM-21**); template [`operator-rulebook-template.md`](../docs/analysis/tasks/epics/EPIC-M1-04-policy-gate-operator-rulebook/artifacts/operator-rulebook-template.md) **v0.2** (**GM4-03** / **GIM-22**). |
| [`issue-normalizer.md`](./issue-normalizer.md) | **EPIC-M1-05** / **GM5-01** (**GIM-24**): `normalized_issue_payload` (`canonical_payload` + `normalization_metadata` with refs to validation, safety, gate); **Plain GPT** — **no API**, **no user questions**; SoT [`issue-data-model.md`](./issue-data-model.md) §4.1 (**v0.1.2**). **GM5-02:** strict chain in `base` / `ingest-validation` / `safety-compliance` / lifecycle aligned on this artifact. **GM5-03:** REQ-10 projection table (§5.1) for card fields. |

Product refs: `GPT UI/docs/requirements/REQ-10-output-content-model.md`, `REQ-15-working-assumptions.md`, `GPT UI/docs/technical-architecture.md` §3.2, §7.

**REQ-11 blocks A–B → instructions (traceability):** [`../docs/analysis/REQ-11-blocks-AB-instruction-traceability.md`](../docs/analysis/REQ-11-blocks-AB-instruction-traceability.md) (STORY-GM1-05).

**FR-M1 (REQ-09) → instruction modules (EPIC-M1-03):** [`FR-M1-traceability-matrix.md`](../docs/analysis/tasks/epics/EPIC-M1-03-FR-M1-traceability-and-ingest-core/artifacts/FR-M1-traceability-matrix.md) (scaffold; **GM3-01** / **GIM-14**) · [`FR-M1-019-extraction-mapping.md`](../docs/analysis/tasks/epics/EPIC-M1-03-FR-M1-traceability-and-ingest-core/artifacts/FR-M1-019-extraction-mapping.md) (**GM3-02** / **GIM-15**) · [`REQ-16-Q3-interview-versus-strict-batch-issue.md`](../docs/analysis/tasks/epics/EPIC-M1-03-FR-M1-traceability-and-ingest-core/artifacts/REQ-16-Q3-interview-versus-strict-batch-issue.md) (**GM3-03** / **GIM-16**) · Phase 7 **§7.2** (**GM3-04** / **GIM-17**) · **FR-M1-024…027** + REQ-15.3 **observation** (**GM3-05** / **GIM-18**) · [`issue-i18n-policy.md`](./issue-i18n-policy.md) (**GM3-06** / **GIM-19**, FR-M1-028…031). Plain Custom GPT: §5 → **§7.2** → Issue §4.1 structural batch (`ingest-validation` overlay) → trilingual policy.

**Language (canonical Issue instruction modules):** `issue-data-model.md`, `issue-lifecycle-instructions.md`, `issue-interview-flow.md`, `issue-api-methods-reference.md`, `issue-i18n-policy.md`, `issue-policy-gate.md`, and `issue-normalizer.md` are **English-only** in `instructions/` (repository policy; see [openai-custom-gpt-builder skill](../../.cursor/skills/openai-custom-gpt-builder/SKILL.md) → *Language: English-only for instruction files*).

### Planned renames & new files (§7.2–7.3)

[`issue-policy-gate.md`](./issue-policy-gate.md) is **in repo** (scaffold **GM4-01**); [`issue-normalizer.md`](./issue-normalizer.md) is **in repo** (scaffold **GM5-01** / **GIM-24**). **GM5-02** (**GIM-25**): `base.md` §1.5 Issue normalization artifact, `ingest-validation` post-gate handoff, `safety-compliance` Point 4 (Issue), `issue-lifecycle-instructions` v0.6. The **operator rulebook** (OP-DOC) remains **outside** git — see gate module §3. Other rows are **roadmap** until implemented (**GM5-03** orchestrator wire, **M1-06**, grep pass §7.4).

| Current file (exists) | Target / addition (planned) | Owner (epic) |
|----------------------|-----------------------------|--------------|
| [`activity-normalizer.md`](./activity-normalizer.md) | For **Issue**: use [`issue-normalizer.md`](./issue-normalizer.md) + `normalized_issue_payload` (**GM5-01**); Activity file remains Amanita reference | EPIC-M1-05 |
| `activity-data-model.md` | Superseded for Issue by `issue-data-model.md` (keep for Amanita reference) | — |
| `konyrody-gate.md` | [`issue-policy-gate.md`](./issue-policy-gate.md) + **external** operator rulebook (`policy_ref`) | EPIC-M1-04 |
| `api-methods-reference.md` | **`issue-api-methods-reference.md`** + **`custom-gpt-actions-issues-reference.openapi.yaml`** (v0.1 stubs — reconcile with node TBD) | EPIC-M1-06 |
| [`issue-interview-flow.md`](./issue-interview-flow.md) | `issue-interview-extraction.md` (planned) | EPIC-M1-02 |

Internal content of `base.md`, `ingest-validation.md`, `ingest-deep-parsing.md`, `safety-compliance.md`, `api-orchestrator.md`, `search-dialogue.md` should gain **Issue** terminology over time without renaming files first (§7.2).

---

## Hierarchy & shared references (wrapper layer)

| File | Role |
|------|------|
| [`root.md`](./root.md) | Top-level hierarchy, **no false backend claims**; still Amanita-branded — DOGEstonia identity in **EPIC-M1-04** (`root.md` patch). |
| [`bootstrap.md`](./bootstrap.md) | `comm_context` (language, tone, verbosity). |
| [`communication-presets-reference.md`](./communication-presets-reference.md) | Preset definitions referenced from bootstrap / base. |

---

## Legacy Amanita domain references (Activities)

**Inventory (GM1-06):** [`activity-legacy-paths-inventory.md`](./activity-legacy-paths-inventory.md) — ripgrep commands, summary table, owner epics (M1-03 … M1-06).

| File | Note |
|------|------|
| [`activity-data-model.md`](./activity-data-model.md) | **Amanita / Activities** schema (Eventify). For DOGEstonia Issue fields use **`issue-data-model.md`**. |
| [`search-data-model.md`](./search-data-model.md) | Search shape for Activities stack; Module 1 ingest may stay SEARCH-optional per REQ-15. |

---

## Module List (functional pipeline)

Numbering follows the ingest-oriented order from `GPT UI/docs/technical-architecture.md` §2. **DOGEstonia / Issue** notes are additive; module files still use legacy wording until refactors land.

### 1. Base Instruction — Functional Constitution  
**File:** [`base.md`](./base.md)

**Role:**  
Defines the **core functional rules** of the system.

Covers:
- operational modes (ingest / search / help / policy),
- (legacy text) Activity status model and allowed transitions,
- global privacy & GDPR constraints,
- rules for handling non-dialogue input,
- clarification policy for missing data.

**DOGEstonia / Issue:** Ingest targets **Issue** creation/update semantics; align strict-artifact chain with [`issue-lifecycle-instructions.md`](./issue-lifecycle-instructions.md) (**§2.1** **GM4-02**; **GM5-02** §1.5 Issue `normalized_issue_payload` note). **INGEST mode** includes Issue overlay (stop-the-line + `issue-interview-flow` §5) — **GM2-03**. REQ-15: SEARCH not mandatory for Module 1.

**Amanita:** Indexed wording remains Activity-centric until `base.md` is edited.

All functional modules assume Base Instruction rules are already enforced.

---

### 2. Ingest Validation Instruction  
**File:** [`ingest-validation.md`](./ingest-validation.md)

**Role:**  
Handles all inputs intended to create or modify **Activities** (legacy).

Covers:
- parsing of free-form text, links, screenshots, PDFs,
- extraction of preliminary structured data,
- identification of missing required fields,
- differentiation between new Activities and updates.

**DOGEstonia / Issue:** **Issue** field completeness (`issue-data-model.md` §4.1) + **Issue track overlay** at top of [`ingest-validation.md`](./ingest-validation.md) (narrative §5; **Phase 7 §7.2** **GM3-04**; **REQ-15.3 / FR-M1-025** provisional `observation` **GM3-05**; **FR-M1-028…031** [`issue-i18n-policy.md`](./issue-i18n-policy.md) **GM3-06**; **REQ-16 Q3** ADR **GM3-03**; **GM4-02** — no skip from validation/safety **directly** to Issue normalization: hand off to [`issue-policy-gate.md`](./issue-policy-gate.md) per `issue-lifecycle-instructions` §2.1; **GM5-02** — after gate **approved**, hand off to [`issue-normalizer.md`](./issue-normalizer.md), **no** API before `normalized_issue_payload`); never calls APIs. Align with [`issue-interview-flow.md`](./issue-interview-flow.md) v0.11 and [`base.md`](./base.md) INGEST overlay + §1.5 Issue artifact (**GM2-03**, **GM3-03**, **GM3-04**, **GM3-05**, **GM3-06**, **GM5-02**).

This module never publishes data and never calls APIs directly.

---

### 3. Policy admission gate (Amanita reference vs Issue)

**Amanita (Activities):** [`konyrody-gate.md`](./konyrody-gate.md) — policy admission using Kоны Рода PDF.

**DOGEstonia / Issue:** [`issue-policy-gate.md`](./issue-policy-gate.md) — same **gate** pattern for **Issue** ingest; operator rulebook is **external** (see [`issue-policy-gate.md`](./issue-policy-gate.md) §3). `konyrody-gate.md` remains a **donor reference** only for Issue work.

This module defines *whether* a record may proceed, not *how* it is stored.

---

### 4. Normalization & Structuring Instruction  
**Amanita (Activities):** [`activity-normalizer.md`](./activity-normalizer.md) — `normalized_activity_payload` pattern (donor reference).

**DOGEstonia / Issue:** [`issue-normalizer.md`](./issue-normalizer.md) — **`normalized_issue_payload`** (**GM5-01** / **GIM-24**, **v0.1.1**): `canonical_payload` per [`issue-data-model.md`](./issue-data-model.md) §4.1 + `normalization_metadata` (refs to `ingest_validation_report`, `safety_compliance_report`, `policy_gate_result`); **Plain Custom GPT** — **no API**, **no user questions**. **GM5-02** — `base` §1.5, `ingest-validation`, `safety-compliance` Point 4, lifecycle §2.1 aligned on this handoff.

---

### 5. API Orchestrator Instruction  
**File:** [`api-orchestrator.md`](./api-orchestrator.md)

**Role:**  
All **HTTP** to backend (only module that may trigger Actions).

Covers:
- create/update draft, review/publish flows (Activities in reference),
- search requests,
- error handling; **backend response is authoritative**.

**DOGEstonia / Issue:** Same **single-orchestrator** pattern; wire to **Issue** OpenAPI on the node (**EPIC-M1-06**). SSOT: [`issue-api-methods-reference.md`](./issue-api-methods-reference.md) + Actions YAML. **M6-03:** explicit Issue stop-the-line gates and response-truth wording in [`api-orchestrator.md`](./api-orchestrator.md).

This module never invents state and always defers to backend responses.

---

### 6. Search Dialogue Instruction  
**File:** [`search-dialogue.md`](./search-dialogue.md)

**Role:**  
Search-oriented conversations; structured query → orchestrator.

**DOGEstonia / Issue:** Optional for Module 1 (REQ-15 ingest-first). If enabled, map search results to **Issue** list semantics when the node supports it.

This module is used only in SEARCH mode.

---

### 7. Ingest Deep Parsing Instruction  
**File:** [`ingest-deep-parsing.md`](./ingest-deep-parsing.md)

**Role:**  
Non-dialogue input → `deep_parsing_artifact`.

Covers:
- parsing of screenshots, PDFs, links,
- (legacy) extraction per **Activity** data model,
- ambiguous extraction handling.

**DOGEstonia / Issue:** Extraction should target **Issue** narrative/projection fields; reference [`issue-data-model.md`](./issue-data-model.md) + REQ-10. **Issue overlay** at top of [`ingest-deep-parsing.md`](./ingest-deep-parsing.md): FR-M1-024…027 hints, **REQ-15.3 / FR-M1-025** provisional **observation** (**GM3-05**).

This module is activated by Ingest Validation for non-dialogue input.

---

### 8. Safety & Compliance Instruction  
**File:** [`safety-compliance.md`](./safety-compliance.md)

**Role:**  
Global safety and compliance boundaries.

Covers:
- prohibited content,
- minors / sensitive contexts,
- redaction or rejection,
- privacy-first behavior.

**DOGEstonia / Issue:** **DOGEstonia / Issue track** overlay under Purpose — REQ-03, **FR-M1-039…043**, checkpoints **→** [`issue-policy-gate.md`](./issue-policy-gate.md) per [`issue-lifecycle-instructions.md`](./issue-lifecycle-instructions.md) §2.1; **Point 4** (`normalized`) references **`normalized_issue_payload`** / [`issue-normalizer.md`](./issue-normalizer.md) (**GM5-02** / **GIM-25**); **§12** alignment with [`issue-interview-flow.md`](./issue-interview-flow.md) v0.12 (**GM4-04** / **GIM-23**). REQ-14 §14.4 checklist: [`REQ14-safety-scenarios-checklist-GM4-04.md`](../docs/analysis/tasks/task-GM4-04-safety-compliance-req03-fr-m1-039-043/REQ14-safety-scenarios-checklist-GM4-04.md).

This module may interrupt any flow if safety rules are violated.

---

## Index changelog

| Версия | Дата | Изменение |
|--------|------|-----------|
| 0.2 | 2026-04-10 | GM1-03: DOGEstonia/Issue notes per module; roadmap §7.2–7.3; hierarchy + legacy tables; working links only. |
| 0.3 | 2026-04-10 | GM1-05: ссылка на [`REQ-11-blocks-AB-instruction-traceability.md`](../docs/analysis/REQ-11-blocks-AB-instruction-traceability.md). |
| 0.4 | 2026-04-10 | EPIC-M1-06 scaffold: `issue-api-methods-reference.md`, Issues OpenAPI YAML, оверлей в `api-orchestrator.md`. |
| 0.5 | 2026-04-10 | GM2-01: `issue-interview-flow.md` v0.1 в canonical Issue docs и roadmap §7.3. |
| 0.6 | 2026-04-10 | Canonical Issue `issue-*.md` set to **English-only**; language policy note; roadmap `issue-interview-flow` v0.2; skill reference. |
| 0.7 | 2026-04-10 | GM2-02: `issue-interview-flow` v0.3 — REQ-06/07; canonical row updated. |
| 0.8 | 2026-04-10 | GM2-03: `ingest-validation` + `base` Issue overlays; `issue-interview-flow` v0.4; module 2 DOGEstonia note. |
| 0.9 | 2026-04-10 | GM2-04: `issue-interview-flow` v0.5 — REQ-12/13 §8–9; canonical + roadmap. |
| 0.10 | 2026-04-10 | GM2-05: `issue-interview-flow` v0.6 — §7.1 FR-M1-017 reframe templates. |
| 0.11 | 2026-04-10 | GM2-06: `issue-interview-flow` v0.7 — §12 latent + limited-depth; EPIC-M1-04 pointer. |
| 0.12 | 2026-04-10 | GM1-06: `activity-legacy-paths-inventory.md` + link in Legacy Amanita section. |
| 0.13 | 2026-04-10 | GM3-01: link to EPIC-M1-03 `FR-M1-traceability-matrix.md` (FR-M1 scaffold, REQ-09). |
| 0.14 | 2026-04-10 | GM3-02: `FR-M1-019-extraction-mapping.md` + matrix row FR-M1-019; FR-M1 nav line in index. |
| 0.15 | 2026-04-10 | GM3-03: REQ-16 Q3 ADR + matrix FR-M1-007/013/018/022–023; `issue-interview-flow` v0.8; `base`/`ingest-validation` Issue overlay; FR-M1 nav line. |
| 0.16 | 2026-04-10 | GM3-04: `issue-interview-flow` v0.9 §7.2; `base`/`ingest-validation`/`bootstrap`; matrix FR-M1-032…034; FR-M1 nav line. |
| 0.17 | 2026-04-10 | GM3-05: `ingest-deep-parsing` + `ingest-validation` Issue overlays; `issue-interview-flow` v0.10 §8 row 9; matrix FR-M1-024…027; FR-M1 nav + module §7 note. |
| 0.18 | 2026-04-10 | GM3-06: `issue-i18n-policy.md` v0.1; hooks in `bootstrap`/`base`/`communication-presets-reference`; matrix FR-M1-028…031; roadmap row `issue-i18n-policy` removed from planned; canonical Issue table. |
| 0.19 | 2026-04-10 | **GM4-01:** `issue-policy-gate.md` v0.1 in canonical Issue table + roadmap §7.2–7.3 hyperlink; module §3 Issue vs Amanita; English-only list. |
| 0.20 | 2026-04-10 | **GM4-02:** `issue-lifecycle-instructions` v0.4 §2.1; `ingest-validation` / `base` Issue overlays + handoff; `issue-policy-gate` v0.2; STORY line **GIM-21**. |
| 0.21 | 2026-04-10 | **GM4-03:** `operator-rulebook-template.md` v0.2 (EN); `issue-policy-gate` v0.3; STORY line + canonical row **GIM-22**. |
| 0.22 | 2026-04-10 | **GM4-04:** `safety-compliance` Issue overlay; `issue-interview-flow` v0.12; REQ-14 checklist path; module §8 **GIM-23**. |
| 0.23 | 2026-04-10 | **GM5-01:** [`issue-normalizer.md`](./issue-normalizer.md) v0.1 — canonical table + roadmap + module §4 + English-only list + STORY line **GIM-24**. |
| 0.24 | 2026-04-10 | **GM5-02:** `base` §1.5 + `ingest-validation` + `safety-compliance` + `issue-lifecycle-instructions` v0.6; modules 1/2/4/8 DOGEstonia notes; STORY line **GIM-25**. |
| 0.25 | 2026-04-10 | **GM5-03:** `api-orchestrator` Issue input contract (`normalized_issue_payload`), `issue-normalizer` v0.1.2 REQ-10 table, `activity-normalizer` deprecation, FR-M1-035…037 mapping; STORY line **GIM-26**. |
| 0.26 | 2026-04-10 | **M6-02:** `issue-api-methods-reference` v0.3 parity matrix (operationId/method/path/request/response), `api-orchestrator` lockstep section, Issues OpenAPI YAML `0.1.1`; lock snapshot note for absent committed node `/openapi.json`. |
| 0.27 | 2026-04-10 | **M6-03:** `api-orchestrator` Issue execution contract — pre-HTTP stop-the-line gates, strict response-truth phrasing, no silent fallback on payload/schema mismatch. |
| 0.28 | 2026-04-10 | **M6-04:** Actions auth/identity/versioning lock — `issue-api-methods-reference` v0.4 (app-level bearer + lockstep rule), Issues OpenAPI YAML `0.1.2`, operator checklist `operator-actions-checklist-M6-04.md`. |
| 0.1 | *(prior)* | Amanita list + initial DOGEstonia Issue stub (GM1-01/02). |
