# Instruction Modules Index

| Поле | Значение |
|------|----------|
| **Версия индекса** | 0.39 |
| **Дата** | 2026-04-27 |
| **Scope** | Runtime navigation map for DOGEstonia Issue instructions; no task/epic dependency in operational sections. |

This file is a navigation map for runtime instruction modules. The active production path is **DOGEstonia / Issue (Module 1)**. Legacy donor content is removed from the active runtime surface.

**For Custom GPT DOGEstonia ingest use Story track modules only** (`story-*`, `ingest-*`, `api-orchestrator.md` in Story mode). Legacy donor guardrails and review checklist: [`activity-legacy-paths-inventory.md`](./activity-legacy-paths-inventory.md).

Use this index together with [`story-data-model.md`](./story-data-model.md), [`story-label-taxonomy.md`](./story-label-taxonomy.md), [`story-lifecycle-instructions.md`](./story-lifecycle-instructions.md), [`story-policy-gate.md`](./story-policy-gate.md), [`story-normalizer.md`](./story-normalizer.md), [`story-api-methods-reference.md`](./story-api-methods-reference.md), and label regression matrix [`label-extraction-e2e-acceptance-matrix.md`](../docs/analysis/label-extraction-e2e-acceptance-matrix.md).

---

## Canonical Issue references

| File | Role |
|------|------|
| [`story-data-model.md`](./story-data-model.md) | Canonical logical Issue fields and structural contracts. |
| [`story-label-taxonomy.md`](./story-label-taxonomy.md) | Controlled label axes, allowed canonical keys, metadata-only candidates, and internal label disposition. |
| [`story-lifecycle-instructions.md`](./story-lifecycle-instructions.md) | Strict chain and handoff order for Module 1. |
| [`story-interview-flow.md`](./story-interview-flow.md) | Dialogue intake flow and clarification boundaries. |
| [`story-i18n-policy.md`](./story-i18n-policy.md) | Session language and `{ et, ru, en }` output policy. |
| [`story-policy-gate.md`](./story-policy-gate.md) | Policy admission gate for Issue ingest (no HTTP calls). |
| [`story-normalizer.md`](./story-normalizer.md) | Builds `normalized_issue_payload` after gate approval. |
| [`story-api-methods-reference.md`](./story-api-methods-reference.md) | HTTP/OpenAPI reference for Issue actions and lockstep policy. |
| [`openapi-ssot-governance-playbook-GIM-46.md`](../docs/analysis/tasks/task-implement-m1-issues-api-openapi-ssot-governance/openapi-ssot-governance-playbook-GIM-46.md) | Executable governance checklist for one-changeset OpenAPI/SSOT updates (GIM-46). |
| [`label-extraction-e2e-acceptance-matrix.md`](../docs/analysis/label-extraction-e2e-acceptance-matrix.md) | E2E acceptance matrix for label vocabulary, metadata vs canonical labels, orchestrator boundary (GIM-91). |

Core requirement references: `GPT UI/docs/requirements/REQ-09-functional-requirements.md`, `REQ-10-output-content-model.md`, `REQ-11-instruction-blocks.md`, `REQ-14-acceptance-criteria.md`, `REQ-15-working-assumptions.md`, `REQ-18-api-inbound-story-intake-and-gpt-handoff.md`.

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

`ingest-validation -> safety-compliance -> story-policy-gate -> story-normalizer -> api-orchestrator`

with `base.md` as global constitution and `story-lifecycle-instructions.md` as chain reference.

| Module | File | Primary output / constraint |
|--------|------|-----------------------------|
| Base | [`base.md`](./base.md) | Global behavior rules and mode contracts. |
| Ingest validation | [`ingest-validation.md`](./ingest-validation.md) | `ingest_validation_report`; no API calls. |
| Deep parsing | [`ingest-deep-parsing.md`](./ingest-deep-parsing.md) | `deep_parsing_artifact` for non-dialogue inputs. |
| Safety | [`safety-compliance.md`](./safety-compliance.md) | `safety_compliance_report`; may BLOCK/REQUEST. |
| Policy gate | [`story-policy-gate.md`](./story-policy-gate.md) | `policy_gate_result`; admission decision only. |
| Normalization | [`story-normalizer.md`](./story-normalizer.md) | `normalized_issue_payload`; no user questions/API. |
| API orchestration | [`api-orchestrator.md`](./api-orchestrator.md) | The only module allowed to execute HTTP actions. |
| Search flow | SEARCH-mode handoff via base/orchestrator contracts | Optional; activate only when Issue search operations exist in OpenAPI. |

---

## Minimal version log

| Версия | Дата | Изменение |
|--------|------|-----------|
| 0.39 | 2026-04-27 | REQ-09 core reference filename corrected (`REQ-09-functional-requirements.md`); aligned with demo M2 gap closure hygiene (GIM-99). |
| 0.38 | 2026-04-27 | Removed `instructions/issue-*.md` symlink aliases; runtime modules live only as `story-*.md`. Historical task docs: [`README-issue-to-story-rename-aliases.md`](../docs/analysis/tasks/README-issue-to-story-rename-aliases.md). |
| 0.37 | 2026-04-27 | Linked `docs/analysis/label-extraction-e2e-acceptance-matrix.md` for label pipeline regression cases (GIM-91 artifact). |
| 0.36 | 2026-04-26 | Added `story-label-taxonomy.md` as the controlled label axes and vocabulary reference (GIM-85). |
| 0.35 | 2026-04-23 | Added explicit Issue-first ingest pointer with link to legacy guardrails inventory. |
| 0.34 | 2026-04-23 | Added GIM-46 OpenAPI SSOT governance playbook to canonical references. |
| 0.33 | 2026-04-22 | Operator readiness table: link OP-DOC pilot checklist (GIM-58) + pilot/demo criticality rubric; no runtime chain changes. |
| 0.32 | 2026-04-21 | Removed legacy stub file references and search-dialogue file dependency from runtime index. |
| 0.31 | 2026-04-21 | Final cleanup: operational-only index, removed task/epic and roadmap noise from runtime navigation. |
| 0.30 | 2026-04-20 | Legacy donor cleanup synchronized with Issue-first module set. |
