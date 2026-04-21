# Instruction Modules Index

| Поле | Значение |
|------|----------|
| **Версия индекса** | 0.31 |
| **Дата** | 2026-04-21 |
| **Scope** | Runtime navigation map for DOGEstonia Issue instructions; no task/epic dependency in operational sections. |

This file is a navigation map for runtime instruction modules. The active production path is **DOGEstonia / Issue (Module 1)**. Legacy Activity content must remain only in explicit stub or migration-log files.

Use this index together with [`issue-data-model.md`](./issue-data-model.md), [`issue-lifecycle-instructions.md`](./issue-lifecycle-instructions.md), [`issue-policy-gate.md`](./issue-policy-gate.md), [`issue-normalizer.md`](./issue-normalizer.md), and [`issue-api-methods-reference.md`](./issue-api-methods-reference.md).

---

## Canonical Issue references

| File | Role |
|------|------|
| [`issue-data-model.md`](./issue-data-model.md) | Canonical logical Issue fields and structural contracts. |
| [`issue-lifecycle-instructions.md`](./issue-lifecycle-instructions.md) | Strict chain and handoff order for Module 1. |
| [`issue-interview-flow.md`](./issue-interview-flow.md) | Dialogue intake flow and clarification boundaries. |
| [`issue-i18n-policy.md`](./issue-i18n-policy.md) | Session language and `{ et, ru, en }` output policy. |
| [`issue-policy-gate.md`](./issue-policy-gate.md) | Policy admission gate for Issue ingest (no HTTP calls). |
| [`issue-normalizer.md`](./issue-normalizer.md) | Builds `normalized_issue_payload` after gate approval. |
| [`issue-api-methods-reference.md`](./issue-api-methods-reference.md) | HTTP/OpenAPI reference for Issue actions and lockstep policy. |

Core requirement references: `GPT UI/docs/requirements/REQ-09-functional-requirements-m1.md`, `REQ-10-output-content-model.md`, `REQ-11-instruction-blocks.md`, `REQ-14-acceptance-criteria.md`, `REQ-15-working-assumptions.md`, `REQ-18-api-inbound-story-intake-and-gpt-handoff.md`.

---

## Runtime hierarchy and wrappers

| File | Role |
|------|------|
| [`root.md`](./root.md) | Top-level module hierarchy and activation rules. |
| [`bootstrap.md`](./bootstrap.md) | Session context (`ui_lang`, tone, verbosity, presets). |
| [`communication-presets-reference.md`](./communication-presets-reference.md) | Shared communication preset definitions. |

---

## Functional module chain

The mandatory Issue execution chain for Module 1 is:

`ingest-validation -> safety-compliance -> issue-policy-gate -> issue-normalizer -> api-orchestrator`

with `base.md` as global constitution and `issue-lifecycle-instructions.md` as chain reference.

| Module | File | Primary output / constraint |
|--------|------|-----------------------------|
| Base | [`base.md`](./base.md) | Global behavior rules and mode contracts. |
| Ingest validation | [`ingest-validation.md`](./ingest-validation.md) | `ingest_validation_report`; no API calls. |
| Deep parsing | [`ingest-deep-parsing.md`](./ingest-deep-parsing.md) | `deep_parsing_artifact` for non-dialogue inputs. |
| Safety | [`safety-compliance.md`](./safety-compliance.md) | `safety_compliance_report`; may BLOCK/REQUEST. |
| Policy gate | [`issue-policy-gate.md`](./issue-policy-gate.md) | `policy_gate_result`; admission decision only. |
| Normalization | [`issue-normalizer.md`](./issue-normalizer.md) | `normalized_issue_payload`; no user questions/API. |
| API orchestration | [`api-orchestrator.md`](./api-orchestrator.md) | The only module allowed to execute HTTP actions. |
| Search dialogue | [`search-dialogue.md`](./search-dialogue.md) | Optional SEARCH-mode flow; not required for ingest-first path. |

---

## Legacy and migration contour

Legacy files are allowed only as explicit stubs and migration documentation:

| File | Status |
|------|--------|
| [`activity-data-model.md`](./activity-data-model.md) | Historical stub; not used in active Issue chain. |
| [`activity-normalizer.md`](./activity-normalizer.md) | Historical stub; replaced by Issue normalizer path. |
| [`api-methods-reference.md`](./api-methods-reference.md) | Historical donor reference; non-normative for Issue runtime. |
| [`activity-legacy-paths-inventory.md`](./activity-legacy-paths-inventory.md) | Migration log and verification checklist. |

---

## Minimal version log

| Версия | Дата | Изменение |
|--------|------|-----------|
| 0.31 | 2026-04-21 | Final cleanup: operational-only index, removed task/epic and roadmap noise from runtime navigation. |
| 0.30 | 2026-04-20 | Activity legacy cleanup synchronized with Issue-first module set. |
