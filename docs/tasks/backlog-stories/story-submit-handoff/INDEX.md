# Story-submit handoff — backlog package

**Зона:** `GPT UI/docs/tasks/backlog-stories/story-submit-handoff/`  
**Метод:** [`analysis.mdc`](../../../../../.cursor/rules/analysis.mdc) — факты по фактическим инструкциям, пути указаны.  
**Решение (интервью 2026-07-02):** модель создания истории — **«браузер сабмитит»**. GPT больше не постит историю напрямую в gateway. См. [`mvp-integration-plan-2026-07-02.md` §2](../../../../../docs/analysis/mvp-integration-plan-2026-07-02.md).

| Order | Story | Status | Depends |
|-------|-------|--------|---------|
| 1 | [STORY-GPT-SUBMIT-01 — redirect handoff](./STORY-GPT-SUBMIT-01-redirect-handoff.md) | Done | — |
| 2 | [STORY-GPT-SUBMIT-02 — legacy intake/submitter cleanup](./STORY-GPT-SUBMIT-02-legacy-intake-submitter-cleanup.md) | Done | SUBMIT-01 |

**Progress:** 2/2 Done (100%)

### Pipeline mirror

| Key | Pipeline story | Audit |
|-----|----------------|-------|
| GPT-SUBMIT-01 | [STORY-M1-06-25](../../../analysis/tasks/epics/EPIC-M1-06-orchestrator-openapi-web2/stories/STORY-M1-06-25-submit-redirect-handoff/STORY-M1-06-25-submit-redirect-handoff.md) | [re-audit #2](../../../analysis/audit-gpt-submit-01-redirect-handoff-reaudit2-2026-07-06.md) |
| GPT-SUBMIT-02 | [STORY-M1-06-26](../../../analysis/tasks/epics/EPIC-M1-06-orchestrator-openapi-web2/stories/STORY-M1-06-26-legacy-intake-submitter-cleanup/STORY-M1-06-26-legacy-intake-submitter-cleanup.md) | [audit](../../../analysis/audit-gpt-submit-02-legacy-cleanup-code-2026-07-07.md) · [re-audit](../../../analysis/audit-gpt-submit-02-reaudit-gap-closure-2026-07-07.md) |

## Граница

- Приём стеша и браузер-сабмит — gateway ([story-draft-handoff](../../../../../doge-complaints-gateway/docs/tasks/backlog-stories/story-draft-handoff/INDEX.md)); логин/verify — identity+spa (ID-08). Здесь — только правка GPT-инструкций.
