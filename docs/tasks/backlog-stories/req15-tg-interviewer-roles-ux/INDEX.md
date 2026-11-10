# req15-tg-interviewer-roles-ux — INDEX

> **Пакет:** REQ15 Telegram interviewer — roles instruction UX residual (proactivity + surface + citizen no-theater)  
> **Parent:** peer [`REQ-48`](../../../requirements/REQ-48-req15-role-oriented-instruction-line-and-export.md) · package [`../req-48-role-oriented-instruction-line/INDEX.md`](../req-48-role-oriented-instruction-line/INDEX.md) ROL-01…04 **Done** (do not reopen)  
> **Product parent:** REQ15 channel interview  
> **Architecture:** [`../../../architecture/req15-tg-interviewer-roles-ux/00-overview.md`](../../../architecture/req15-tg-interviewer-roles-ux/00-overview.md) · [`03-citizen-surface-no-role-theater.md`](../../../architecture/req15-tg-interviewer-roles-ux/03-citizen-surface-no-role-theater.md)  
> **Evidence:** [citizen gap](../../../../../docs/analysis/req15-tg-interviewer-citizen-conversation-gap-2026-11-10.md) · [north star](../../../../../docs/analysis/req15-tg-interviewer-citizen-conversation-north-star-2026-11-10.md) · prior [Pille gap](../../../../../docs/analysis/req15-tg-interviewer-ux-gap-pille-2026-11-06.md) · [triage](../../../../../docs/analysis/req15-tg-interviewer-ux-gap-triage-2026-11-06.md)  
> **Peer runtime:** aibridge [`STORY-71`](../../../../../doge-ai-bridge/docs/tasks/backlog-stories/req15-tg-channel-interviewer-ux/STORY-AIBRIDGE-71-awaiting-interpretation-actions-always.md) (soft-dep ROL-07) · prior soft [`STORY-68`](../../../../../doge-ai-bridge/docs/tasks/backlog-stories/req15-tg-channel-interviewer-ux/STORY-AIBRIDGE-68-channel-seam-send-actions-always.md)  
> **Emitted:** 2026-11-06 · **Updated:** 2026-11-10T12:24:53Z — ROL-07 tech-arch + T01–T07 Present (implement deferred)  
> **Method:** [`.cursor/rules/analysis.mdc`](../../../../../.cursor/rules/analysis.mdc)  
> **Honesty:** instruction **texts** only on Apply; no aibridge/telegram code this package

## Stories

| Order | Key | Title | Status | Depends |
|-------|-----|-------|--------|---------|
| 1 | STORY-GPT-ROL-05 | [Stage proactivity + handoff wording](STORY-GPT-ROL-05-channel-interview-stage-proactivity.md) | Done (Awaiting Commits) · pkg-000068 GIM-468…474 🟢 | Soft: aibridge 68 |
| 2 | STORY-GPT-ROL-06 | [Surface readability](STORY-GPT-ROL-06-channel-interview-surface-readability.md) | Done (Awaiting Commits) · pkg-000069 GIM-475…479 🟢 | Soft: after/parallel 05 |
| 3 | STORY-GPT-ROL-07 | [Citizen surface no role theater](STORY-GPT-ROL-07-citizen-surface-no-role-theater.md) | Done (Awaiting Commits) · pkg-000070 GIM-480…486 🟢 | Soft: aibridge **71** |

## Build order

`05` → `06` → `07` Done (Awaiting Commits) · soft-parallel aibridge 71/72 remains Open peer.

## Out of package

- Aibridge STORY-68…72 runtime actions  
- Telegram ops redact LF  
- Re-authoring ROL-01…04 / archive Module-1  
- Model-owned stash restore  
