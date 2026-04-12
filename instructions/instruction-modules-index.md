# Instruction Modules Index

| Поле | Значение |
|------|----------|
| **Версия индекса** | 0.11 |
| **Дата** | 2026-04-10 |
| **STORY** | GM1-03 + REQ-11 A–B (GM1-05) + **GM2-01…06** (`issue-interview-flow` §12 REQ-04.2, §7.1, REQ-12/13, overlays) |

This document lists instruction modules in the repository. **Two tracks:**

1. **Amanita reference (Activities)** — file names and much of the body text still describe the donor **Activity** domain; kept for pattern reuse.
2. **DOGEstonia / Issue (Module 1)** — target domain is **Issue** (`spa-app`), per `GPT UI/docs/requirements/REQ-11-instruction-blocks.md` (catalog level) and `GPT UI/docs/technical-architecture.md` §7.

For Custom GPT: **instructions** carry behavior; this file is a **navigation map** — use together with [`issue-data-model.md`](./issue-data-model.md), [`issue-lifecycle-instructions.md`](./issue-lifecycle-instructions.md), and [`issue-interview-flow.md`](./issue-interview-flow.md).

---

## DOGEstonia — Issue (Module 1, parallel track)

### Canonical Issue docs (in repo)

| File | Role |
|------|------|
| [`issue-data-model.md`](./issue-data-model.md) | Logical **Issue** fields (`spa-app`-aligned). Not Activity. |
| [`issue-lifecycle-instructions.md`](./issue-lifecycle-instructions.md) | Ingest pipeline and strict artifacts from the modules’ viewpoint; not `ISSUE_STATUS` semantics. |
| [`issue-interview-flow.md`](./issue-interview-flow.md) | **REQ-08** + **§12** REQ-04.2 latent + limited-depth + **EPIC-M1-04** pointer + **§7.1** FR-M1-017 + REQ-05/06/07 + REQ-12/13 (**v0.7**); overlays GM2-03. **EPIC-M1-02** (GM2-01…06). |
| [`issue-api-methods-reference.md`](./issue-api-methods-reference.md) | **HTTP SSOT for Issue** (paths, operationIds, link to Actions YAML v0.1); lock with node OpenAPI when ready (**EPIC-M1-06**). |

Product refs: `GPT UI/docs/requirements/REQ-10-output-content-model.md`, `REQ-15-working-assumptions.md`, `GPT UI/docs/technical-architecture.md` §3.2, §7.

**REQ-11 blocks A–B → instructions (traceability):** [`../docs/analysis/REQ-11-blocks-AB-instruction-traceability.md`](../docs/analysis/REQ-11-blocks-AB-instruction-traceability.md) (STORY-GM1-05).

**Language (canonical Issue instruction modules):** `issue-data-model.md`, `issue-lifecycle-instructions.md`, `issue-interview-flow.md`, and `issue-api-methods-reference.md` are **English-only** in `instructions/` (repository policy; see [openai-custom-gpt-builder skill](../../.cursor/skills/openai-custom-gpt-builder/SKILL.md) → *Language: English-only for instruction files*).

### Planned renames & new files (§7.2–7.3) — not hyperlinked until files exist

Do **not** treat the left column as missing files — these are **roadmap** rows. Implement in epics **M1-04…M1-06** (and grep pass §7.4).

| Current file (exists) | Target / addition (planned) | Owner (epic) |
|----------------------|-----------------------------|--------------|
| `activity-normalizer.md` | `issue-normalizer.md`, `normalized_issue_payload` | EPIC-M1-05 |
| `activity-data-model.md` | Superseded for Issue by `issue-data-model.md` (keep for Amanita reference) | — |
| `konyrody-gate.md` | `issue-policy-gate.md` + operator rulebook | EPIC-M1-04 |
| `api-methods-reference.md` | **`issue-api-methods-reference.md`** + **`custom-gpt-actions-issues-reference.openapi.yaml`** (v0.1 stubs — reconcile with node TBD) | EPIC-M1-06 |
| [`issue-interview-flow.md`](./issue-interview-flow.md) v0.7 | `issue-interview-extraction.md`, `issue-i18n-policy.md` (planned) | EPIC-M1-02, M1-03 |

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

**DOGEstonia / Issue:** Ingest targets **Issue** creation/update semantics; align strict-artifact chain with [`issue-lifecycle-instructions.md`](./issue-lifecycle-instructions.md). **INGEST mode** includes Issue overlay (stop-the-line + `issue-interview-flow` §5) — **GM2-03**. REQ-15: SEARCH not mandatory for Module 1.

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

**DOGEstonia / Issue:** **Issue** field completeness (`issue-data-model.md` §4.1) + **Issue track overlay** at top of [`ingest-validation.md`](./ingest-validation.md) (narrative §5 / layers §4 vs Activity §1.1 first gate); never calls APIs. Align with [`issue-interview-flow.md`](./issue-interview-flow.md) v0.4 and [`base.md`](./base.md) INGEST overlay (**GM2-03**).

This module never publishes data and never calls APIs directly.

---

### 3. KоныРода Admission Gate  
**File:** [`konyrody-gate.md`](./konyrody-gate.md)

**Role (Amanita):**  
Policy admission for **Activities** using Kоны Рода PDF.

**DOGEstonia / Issue:** **Amanita-only product** for this filename. Replace with **`issue-policy-gate.md`** + operator rulebook (**EPIC-M1-04**). Until then, treat as **placeholder path** in migration plans only.

This module defines *whether* a record may proceed, not *how* it is stored.

---

### 4. Normalization & Structuring Instruction  
**File:** [`activity-normalizer.md`](./activity-normalizer.md)

**Role (legacy):**  
Canonical **Activity** JSON and `normalized_activity_payload` pattern.

**DOGEstonia / Issue:** Target file name **`issue-normalizer.md`** and payload **`normalized_issue_payload`** (**EPIC-M1-05**). Current file on disk remains the reference implementation to fork.

This module produces structured output only (no direct user interrogation — via Validation).

---

### 5. API Orchestrator Instruction  
**File:** [`api-orchestrator.md`](./api-orchestrator.md)

**Role:**  
All **HTTP** to backend (only module that may trigger Actions).

Covers:
- create/update draft, review/publish flows (Activities in reference),
- search requests,
- error handling; **backend response is authoritative**.

**DOGEstonia / Issue:** Same **single-orchestrator** pattern; wire to **Issue** OpenAPI on the node (**EPIC-M1-06**). SSOT today: [`api-methods-reference.md`](./api-methods-reference.md) (Activities paths until YAML migration).

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

**DOGEstonia / Issue:** Extraction should target **Issue** narrative/projection fields; reference [`issue-data-model.md`](./issue-data-model.md) + REQ-10.

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

**DOGEstonia / Issue:** Same checkpoints along strict chain ([`issue-lifecycle-instructions.md`](./issue-lifecycle-instructions.md) §2); align with REQ-03 / FR-M1-039–043 via **EPIC-M1-04** gate + safety text updates.

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
| 0.1 | *(prior)* | Amanita list + initial DOGEstonia Issue stub (GM1-01/02). |
