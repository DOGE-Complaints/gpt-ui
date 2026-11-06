# req15-tg-interviewer-roles-ux — INDEX

> **Пакет:** REQ15 Telegram interviewer — roles instruction UX residual (proactivity + surface)  
> **Parent:** peer [`REQ-48`](../../../requirements/REQ-48-req15-role-oriented-instruction-line-and-export.md) · package [`../req-48-role-oriented-instruction-line/INDEX.md`](../req-48-role-oriented-instruction-line/INDEX.md) ROL-01…04 **Done** (do not reopen)  
> **Product parent:** REQ15 channel interview  
> **Architecture:** [`../../../architecture/req15-tg-interviewer-roles-ux/00-overview.md`](../../../architecture/req15-tg-interviewer-roles-ux/00-overview.md)  
> **Evidence:** [`docs/analysis/req15-tg-interviewer-ux-gap-pille-2026-11-06.md`](../../../../../docs/analysis/req15-tg-interviewer-ux-gap-pille-2026-11-06.md) · [target](../../../../../docs/analysis/req15-tg-interviewer-target-ux-2026-11-06.md) · [triage](../../../../../docs/analysis/req15-tg-interviewer-ux-gap-triage-2026-11-06.md)  
> **Peer runtime:** aibridge [`STORY-68`](../../../../../doge-ai-bridge/docs/tasks/backlog-stories/req15-tg-channel-interviewer-ux/STORY-AIBRIDGE-68-channel-seam-send-actions-always.md) (soft-dep for ROL-05)  
> **Emitted:** 2026-11-06 · **Tech-arch + subtaks:** 2026-11-06T11:51:22Z · **Status:** Todo · implement deferred  
> **Method:** [`.cursor/rules/analysis.mdc`](../../../../../.cursor/rules/analysis.mdc)  
> **Honesty:** instruction **texts** only on Apply; no aibridge/telegram code this package; supersedes root accumulative prose batch

## Stories

| Order | Key | Title | Status | Depends |
|-------|-----|-------|--------|---------|
| 1 | STORY-GPT-ROL-05 | [Stage proactivity + handoff wording](STORY-GPT-ROL-05-channel-interview-stage-proactivity.md) | Done (Awaiting Commits) · pkg-000068 GIM-468…474 🟢 | Soft: aibridge 68 |
| 2 | STORY-GPT-ROL-06 | [Surface readability](STORY-GPT-ROL-06-channel-interview-surface-readability.md) | Todo · tech-arch + T01–T05 Present | Soft: after/parallel 05 |

## Build order

`05` (behavior) → `06` (surface). Soft-parallel OK if handoff paragraphs not co-edited.  
**Apply/P3** of `instructions-roles/**` = separate dialogue (name≠create in this deepen).

## Out of package

- Aibridge STORY-68/69 runtime actions  
- Telegram ops redact LF  
- Re-authoring ROL-01…04 / archive Module-1  
- Model-owned stash restore  
