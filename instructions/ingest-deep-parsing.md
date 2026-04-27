# Ingest Deep Parsing Instruction
## Detailed Parsing Algorithms for Non-Dialogue Input (Issue track)

| Field | Value |
|-------|--------|
| **Version** | 1.0.0 |
| **Date** | 2026-04-20 |
| **Product** | DOGEstonia — Module 1 (Issue ingest) |
| **Traceability** | FR-M1-024…031; [`story-data-model.md`](story-data-model.md); [`ingest-validation.md`](ingest-validation.md) |

### Purpose

This instruction provides **format-specific extraction algorithms** for **non-dialogue** user input (screenshots, PDFs, links, pasted text, mixed content) into a structured **`deep_parsing_artifact`** for **Issue** civic intake. Legacy donor extraction has been **removed** from this repository version.

This instruction is activated by **Ingest Validation** when non-dialogue input is detected. It **never validates** final publishable Issue state, **never calls APIs**, and **never assigns** final `ISSUE_STATUS` or backend-only fields.

---

### DOGEstonia / Issue track — deep parsing overlay

When `root.md` routes to **Issue** ingest, this module prepares **`deep_parsing_artifact`** hints for **Ingest Validation** and downstream dialogue (`story-interview-flow.md`). It does **not** assign final published `type` / `labels`; those are confirmed in interview + validation.

**Canonical references:** [`story-data-model.md`](story-data-model.md) §4.1 · [`story-interview-flow.md`](story-interview-flow.md) · [`ingest-validation.md`](ingest-validation.md) (Issue overlay) · [`story-i18n-policy.md`](story-i18n-policy.md) (**FR-M1-028…031**). FR-M1-024…027 and FR-M1-028…031 cover intake + i18n behavior referenced by this overlay.

**Issue archetype signals (FR-M1-024 — hints only, not interview labels)**

| Archetype | Non-exhaustive signals in raw / multimodal input |
|-----------|---------------------------------------------------|
| **complaint** | Harm, unfair treatment, wrongdoing toward a person or group; acute negative affect tied to a specific actor or omission. |
| **observation** | Civic “what if…”, improvement wish, environmental pattern; **or**, per **FR-M1-025**, **positive improvement ideas without stated clear harm or a named victim** — record as **provisional `observation` intent** until product approves a separate Issue `type` in the backlog. Do **not** invent a fifth enum in strict payloads. |
| **absurdity** | Process ridiculousness, Kafkaesque loops, contradictory rules — often ironic without a single villain. |
| **system_bug** | Technical malfunction, broken digital service, reproducible defect language. |

**Boundaries (vs complaint / absurdity):** If the user text clearly centers **harm to people**, do **not** downgrade the hint to **observation** to avoid depth; pass through **complaint** signals for downstream dialogue (`story-interview-flow.md` **§4–§5**, **§8–§9**).

**Pseudo-therapy guard:** Using **observation** as a **routing bucket** is **not** minimizing distress and **not** clinical reframing. If content matches **§12** sensitive / limited-depth rules, prefer safety and interview constraints over tightening classification.

**FR-M1-026:** Put **label** candidates only in artifact metadata / `ambiguities[]`-style notes — not as a forced user-facing checklist (interview timing stays with `story-interview-flow.md` **§8** row 2).

**FR-M1-027:** In extracted notes, keep **surface topic**, **deep-need signals**, and possible **institutional address** hypotheses **separate** — do not assert institution IDs. **Demo scope:** do **not** promote institutional hypotheses into Issue **`institution`** card fields.

---

## 1. Scope of Responsibility

Activated **only** when Ingest Validation detects **non-dialogue** input. Handles:

- Deep parsing of screenshots, PDFs, external links, free-form text, mixed content.
- Producing **`deep_parsing_artifact`** with `extracted_data` (Issue hints) and `metadata` (confidence, ambiguities, conflicts, PII flags, missing fields).

Does **not**:

- Determine mode (Base Instruction does this).
- Validate for final handoff (Ingest Validation does).
- Normalize to canonical Issue JSON ([`story-normalizer.md`](story-normalizer.md) after policy gate).
- Call backend APIs.

---

## 2. Input Contract

Receives: raw user input; input type from Base Instruction; optional session context.

**Issue Data Model reference:** extraction hints MUST align with logical fields in [`story-data-model.md`](story-data-model.md) **§4.1–§4.3** (trilingual text, `type`, `labels`, optional fields). Do **not** populate §4.4 system-only fields from parsing, and do not introduce donor-era structured minors fields into Issue hints.

---

## 3. Format-specific extraction (algorithms)

Apply **best-effort** extraction per format. Output **only** hints and quotes; narrative completion stays with `issue-interview-flow` if the session moves to dialogue.

### 3.1 Free-form text

- Segment into: stated facts, emotional tone, actors, place, time, desired change.
- Map hints to Issue archetype table (§ overlay) without locking `type` in the artifact as final.

### 3.2 External links

- Prefer Open Graph / visible title and description; note uncertainty in `ambiguities[]`.
- If link unusable (404, login wall), still extract user-visible URL and flag in `metadata`.

### 3.3 PDF

- Extract headings and body blocks; preserve language detection for `issue-i18n-policy` handoff.

### 3.4 Screenshots / images

- OCR-style extraction of visible text; low confidence → `confidence_scores` & `ambiguities[]`.

### 3.5 Mixed content

- Merge sources; on conflict, list under `metadata.conflicts[]` with `resolved_value` = null unless one source clearly dominates.

---

## 4. Confidence scoring

For each populated hint field, attach **0.0–1.0** confidence in `metadata.confidence_scores`. Critical fields (`title`/`description` trilingual slots) below **0.5** should trigger clarification questions downstream, not silent defaults.

---

## 5. Ambiguity and PII

- **`ambiguities[]`:** field paths or synthetic keys (e.g. `issue_narrative:location`) when multiple interpretations exist.
- **`pii_detected[]`:** categories of potential PII for Safety & Compliance (no raw PII duplication in logs beyond what user already provided).

---

## 6. Error handling

- Never return **empty** artifact without `artifact_id` — partial extraction with explicit `missing_required_fields[]` is required.
- **Multiple distinct civic issues** in one upload: set `metadata.stop_reason = "multiple_issues_detected"` and do **not** merge into one Issue narrative; Ingest Validation may block until user splits input.

---

## 7. Output Contract (`deep_parsing_artifact`)

**Version:** `v1`  
**Artifact ID:** `deep_parsing_<ISO-8601-timestamp>`

SSOT field name for parsed payload: **`extracted_data`** (must match [`ingest-validation.md`](ingest-validation.md) §11.1).

```json
{
  "artifact_id": "deep_parsing_<ISO_timestamp>",
  "version": "v1",
  "extracted_data": {
    "type_hint": "complaint" | "observation" | "absurdity" | "system_bug" | null,
    "labels_hints": [],
    "title": { "et": "", "ru": "", "en": "" },
    "description": { "et": "", "ru": "", "en": "" },
    "summary": { "et": "", "ru": "", "en": "" },
    "notes": {
      "surface_topic": "",
      "deep_need_signals": "",
      "institution_hypothesis": null
    }
  },
  "metadata": {
    "source_type": "screenshot" | "pdf" | "link" | "text" | "mixed",
    "sources": [],
    "confidence_scores": {},
    "ambiguities": [],
    "conflicts": [],
    "pii_detected": [],
    "missing_required_fields": [],
    "stop_reason": null
  }
}
```

**Rules:**

- `extracted_data` uses **hints**; final `type` / `labels` follow interview + validation.
- Omit optional keys if absent; do not fabricate `institution` beyond current demo scope (keep hypotheses as notes only).
- Subjective intake fields (§4.3 `issue-data-model`) may appear as hints only when clearly present in source.

---

## 8. Integration with other instructions

- **Next:** [`ingest-validation.md`](ingest-validation.md) consumes `deep_parsing_artifact` before user questions for non-dialogue.
- **Safety:** [`safety-compliance.md`](safety-compliance.md) Issue overlay if content triggers checkpoints.
- **Normalizer:** [`story-normalizer.md`](story-normalizer.md) only **after** [`story-policy-gate.md`](story-policy-gate.md) on strict chains.

---

## 9. Version history

| Version | Date | Change |
|---------|------|--------|
| 1.0.0 | 2026-04-20 | Legacy donor bulk removal; Issue-only non-dialogue pipeline; `extracted_data` SSOT with `ingest-validation` §11.1; FR-M1-024…027 overlay preserved. |
