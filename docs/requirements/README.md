# DOGEstonia — Module 1: GPT Interview Layer (requirements)

**Источник истины (SoT):** PDF `GPT UI/docs/DOGEstonia — GPT Requirements.pdf` (перенос в Markdown). При расхождении приоритет у PDF.

**Связка GPT → HTTP API (Story Intake, полный конверт):** продуктовые требования — **[REQ-18](./REQ-18-api-inbound-story-intake-and-gpt-handoff.md)**; детальный контракт полей и примеры — `doge-complaints-gateway/docs/requirements/19-inbound-api-gpt-preprocessing-and-spa-issue-contracts.md` (**§19**).

**Связанные эпики реализации (Scrum):** `GPT UI/docs/analysis/tasks/README-epics-module1-gpt-interview.md`  
**Индекс эпиков и сторис (S / Key / Task):** `GPT UI/docs/analysis/tasks/gpt-interview-module1-tasks-index.md`

## Оглавление REQ (27 файлов)

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

**Версия:** 1.1 · 2026-05-25 (REQ-27: gpt_signals enum sync — `story-data-model.md` ↔ `contracts.py` frozensets; orchestrator `impact_estimation` hint fix; follow-up REQ-23 wire activation).
