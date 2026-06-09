# DOGEstonia — Module 1: GPT Interview Layer (requirements)

**Источник истины (SoT):** PDF `GPT UI/docs/DOGEstonia — GPT Requirements.pdf` (перенос в Markdown). При расхождении приоритет у PDF.

**Связка GPT → HTTP API (Story Intake, полный конверт):** продуктовые требования — **[REQ-18](./REQ-18-api-inbound-story-intake-and-gpt-handoff.md)**; детальный контракт полей и примеры — `doge-complaints-gateway/docs/requirements/19-inbound-api-gpt-preprocessing-and-spa-issue-contracts.md` (**§19**).

**Связанные эпики реализации (Scrum):** `GPT UI/docs/analysis/tasks/README-epics-module1-gpt-interview.md`  
**Индекс эпиков и сторис (S / Key / Task):** `GPT UI/docs/analysis/tasks/gpt-interview-module1-tasks-index.md`

## Оглавление REQ (42 файла; №37 вакантен)

| № | Файл |
|---|------|
| 1 | [REQ-01-mission.md](./REQ-01-mission.md) |
| 2 | [REQ-02-business-task.md](./REQ-02-business-task.md) |
| 3 | [REQ-03-scope.md](./REQ-03-scope.md) |
| 4 | [REQ-04-personas.md](./REQ-04-personas.md) |
| 5 | [REQ-05-interview-principles.md](./REQ-05-interview-principles.md) |
| 6 | [REQ-06-psychological-model.md](./REQ-06-psychological-model.md) |
| 7 | [REQ-07-target-outcome.md](./REQ-07-target-outcome.md) |
| 8 | [REQ-08-dialogue-flow.md](./REQ-08-dialogue-flow.md) |
| 9 | [REQ-09-functional-requirements.md](./REQ-09-functional-requirements.md) |
| 10 | [REQ-10-output-content-model.md](./REQ-10-output-content-model.md) |
| 11 | [REQ-11-instruction-blocks.md](./REQ-11-instruction-blocks.md) |
| 12 | [REQ-12-anti-patterns.md](./REQ-12-anti-patterns.md) |
| 13 | [REQ-13-quality-criteria.md](./REQ-13-quality-criteria.md) |
| 14 | [REQ-14-acceptance-criteria.md](./REQ-14-acceptance-criteria.md) |
| 15 | [REQ-15-working-assumptions.md](./REQ-15-working-assumptions.md) |
| 16 | [REQ-16-open-decisions.md](./REQ-16-open-decisions.md) |
| 17 | [REQ-17-module-formula.md](./REQ-17-module-formula.md) |
| 18 | [REQ-18-api-inbound-story-intake-and-gpt-handoff.md](./REQ-18-api-inbound-story-intake-and-gpt-handoff.md) |
| 19 | [REQ-19-security-auth-boundaries-and-user-identity-flow.md](./REQ-19-security-auth-boundaries-and-user-identity-flow.md) |
| 20 | [REQ-20-label-taxonomy-and-extraction-axes.md](./REQ-20-label-taxonomy-and-extraction-axes.md) |
| 21 | [REQ-21-post-demo-privacy-pii-and-web3-vault-policy.md](./REQ-21-post-demo-privacy-pii-and-web3-vault-policy.md) |
| 22 | [REQ-22-story-intake-wire-contract-v2-alignment.md](./REQ-22-story-intake-wire-contract-v2-alignment.md) |
| 23 | [REQ-23-gpt-behavioral-extensions-pii-signals-consistency.md](./REQ-23-gpt-behavioral-extensions-pii-signals-consistency.md) |
| 24 | [REQ-24-gpt-orchestrator-response-contract-corrections.md](./REQ-24-gpt-orchestrator-response-contract-corrections.md) |
| 25 | [REQ-25-canonical-type-labels-and-summary-wire-activation.md](./REQ-25-canonical-type-labels-and-summary-wire-activation.md) |
| 26 | [REQ-26-location-query-normalizer-to-wire.md](./REQ-26-location-query-normalizer-to-wire.md) |
| 27 | [REQ-27-gpt-signals-enum-sync-and-data-model-update.md](./REQ-27-gpt-signals-enum-sync-and-data-model-update.md) |
| 28 | [REQ-28-institution-demo-constraint-gate.md](./REQ-28-institution-demo-constraint-gate.md) |
| 29 | [REQ-29-phase7-translation-review.md](./REQ-29-phase7-translation-review.md) |
| 30 | [REQ-30-admission-gate-story-intake-strict-chain.md](./REQ-30-admission-gate-story-intake-strict-chain.md) |
| 31 | [REQ-31-God-mode-activation.md](./REQ-31-God-mode-activation.md) |
| 32 | [REQ-32-origin-traceability-sending-enforcement.md](./REQ-32-origin-traceability-sending-enforcement.md) |
| 33 | [REQ-33-label-extraction-multi-axis-improvement.md](./REQ-33-label-extraction-multi-axis-improvement.md) |
| 34 | [REQ-34-summary-generation-and-canonical-type-clarity.md](./REQ-34-summary-generation-and-canonical-type-clarity.md) |
| 35 | [REQ-35-demo-field-population-gpt-fixes.md](./REQ-35-demo-field-population-gpt-fixes.md) |
| 36 | [REQ-36-civic-taxonomy-expansion-multi-axis.md](./REQ-36-civic-taxonomy-expansion-multi-axis.md) |
| 37 | _(вакантен — черновик Trigger reliability перенесён в REQ-40/41)_ |
| 38 | [REQ-38-ecosystem-deficit-story-detection.md](./REQ-38-ecosystem-deficit-story-detection.md) |
| 39 | [REQ-39-geographic-intelligence-confidence-canonicalization.md](./REQ-39-geographic-intelligence-confidence-canonicalization.md) |
| 40 | [REQ-40-pre-submission-instruction-compliance-validation.md](./REQ-40-pre-submission-instruction-compliance-validation.md) |
| 41 | [REQ-41-trigger-observability-audit-trail.md](./REQ-41-trigger-observability-audit-trail.md) |
| 42 | [REQ-42-adaptive-post-submission-confirmation-message.md](./REQ-42-adaptive-post-submission-confirmation-message.md) |

**Версия:** 1.8 · 2026-06-08 (REQ-42 Done — adaptive post-submit message; `comm_context.cognitive_style` (systemic/narrative/mixed, passive, default mixed), `api-orchestrator.md` §5.2.4, оба 202-статуса, Citizen forbidden-lexicon; GAP-42-01 namespace disambiguation gateway vs GPT-UI REQ-42; P3 demo+post-demo).

Предыдущее (1.7 · 2026-06-05): REQ-38 Done — ecosystem-deficit detection in `story-interview-flow.md` v0.19 + `story-normalizer.md` v0.2.9; pkg-000018 GIM-163…166.

Предыдущее (1.6 · 2026-06-05): REQ-40 черновик «Trigger Reliability & Execution Determinism» разбит по механизму на 2 дока: REQ-40 pre-submission instruction-compliance validation (enforcement, расширяет §5.2.2 pre-flight, Critical), REQ-41 trigger observability/audit trail (диагностика God Mode, зависит от REQ-40). Номер REQ-37 освобождён — исходный draft перемещён пользователем в REQ-40 и стандартизирован.

Предыдущее (1.5 · 2026-06-04): REQ-36 черновик «Story Intelligence Coverage» разбит на REQ-36 civic taxonomy expansion (foundational SSOT) / REQ-38 ecosystem-deficit detection (зависит от REQ-36) / REQ-39 geographic intelligence (расширяет REQ-26/35).
