# Stage proactivity + handoff wording (ROL-05)

**Parent:** [00-overview.md](./00-overview.md)  
**Story:** [STORY-GPT-ROL-05](../../tasks/backlog-stories/req15-tg-interviewer-roles-ux/STORY-GPT-ROL-05-channel-interview-stage-proactivity.md)  
**As-of:** 2026-11-06T11:51:22Z

## Stage chain (cite roles core)

From shared core chain order (as-built): interview / extract / validate → safety → policy admission → normalize → **server** submit (not model-owned HTTP stash).

Target policy for **channel** interview:

| Rule | Meaning |
|------|---------|
| Auto-advance | When enough citizen data is Present, instructions **MUST** require moving to the next stage without waiting for user prompts «дальше», «продолжай», «проверяй». |
| Boundary turns | Prose **MAY** acknowledge progress (optional one-line crumb). Advance **MUST NOT** depend on free-text coaching alone. |
| Server actions | Wording **MUST** align with server-offered continue / edit / Send **actions** when those are Present ([STORY-AIBRIDGE-68](../../../../doge-ai-bridge/docs/tasks/backlog-stories/req15-tg-channel-interviewer-ux/STORY-AIBRIDGE-68-channel-seam-send-actions-always.md)). |
| No bare «интерфейс» | Submit/handoff wording **MUST NOT** tell the user to use a vague «интерфейс» without naming in-chat actions **or** stating that after server Send confirm a SPA `continuation_url` may appear. |
| Strip preserved | **MUST** keep ROL-03 / `core.md` bans: no model-owned gateway stash; no inventing SPA redirect URLs or Bearer/HTTP submit. |

## Ownership vs peer STORY-68

| Concern | GPT UI (this arch) | aibridge 68 |
|---------|--------------------|-------------|
| Instruct model to auto-advance | Yes | No (runtime) |
| Mint Telegram actions | No | Yes |
| Execute stash | No | Yes (REQ-11) |
| Prohibit model POST gateway | Yes (prose ban) | Yes (code path) |

Soft-dep: ROL-05 wording assumes 68 will expose controls; do not invent action schemas in roles files.

## Touch set (implement)

1. `instructions-roles/core.md` — proactive constitution + handoff wording contract; retain stash bans.  
2. `instructions-roles/interviewer.md` — dialogue-phase auto-advance.  
3. `checker.md` / `normalizer.md` — only if handoff sentences contradict AC-ROL-05-02/04.

## Verify (implement gate)

- Grep volume: no model duty for `POST /story-drafts`, `postStoryDraftStash`, invent `#/story/submit` URL procedure.  
- Grep: no sole-path «интерфейс» submit guidance without actions/URL acknowledgment.  
- AC-ROL-05-01…04 checklist Pass.
