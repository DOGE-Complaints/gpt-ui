# STORY-GPT-ROL-07 — Citizen surface: no role theater

## Meta
- **Key:** `STORY-GPT-ROL-07-citizen-surface-no-role-theater`
- **Status:** Done (Awaiting Commits) · pkg-000070 GIM-480…486 🟢 · P3 `2026-11-10T12:35:03Z`
- **Package:** [`INDEX.md`](INDEX.md) · `req15-tg-interviewer-roles-ux` (residual reopen)
- **Parent:** peer REQ-48 roles line (ROL-01…04 Done) · residual after ROL-05/06
- **Architecture:** [`../../../architecture/req15-tg-interviewer-roles-ux/00-overview.md`](../../../architecture/req15-tg-interviewer-roles-ux/00-overview.md) · [`03-citizen-surface-no-role-theater.md`](../../../architecture/req15-tg-interviewer-roles-ux/03-citizen-surface-no-role-theater.md)
- **Soft-depends:** aibridge [`STORY-AIBRIDGE-71`](../../../../../doge-ai-bridge/docs/tasks/backlog-stories/req15-tg-channel-interviewer-ux/STORY-AIBRIDGE-71-awaiting-interpretation-actions-always.md) (do not promise CTAs server will not mint) · cite [`STORY-AIBRIDGE-72`](../../../../../doge-ai-bridge/docs/tasks/backlog-stories/req15-tg-channel-interviewer-ux/STORY-AIBRIDGE-72-send-after-draft-confirm-and-validation.md) for Send sequencing
- **Evidence:** [north star](../../../../../docs/analysis/req15-tg-interviewer-citizen-conversation-north-star-2026-11-10.md) §1–4 · [citizen gap](../../../../../docs/analysis/req15-tg-interviewer-citizen-conversation-gap-2026-11-10.md) §4.1–4.4 · transcript [`2026-11-10-pille-handoff-chain`](../../../../../docs/analysis/tg-interview-transcripts/req15-tg-interview-dialogue-2026-11-10-pille-handoff-chain.md)
- **Volume (Apply later):** `GPT-UI/instructions-roles/` (primarily `core.md`, `interviewer.md`; fence lines elsewhere)
- **Method:** [`.cursor/rules/analysis.mdc`](../../../../../.cursor/rules/analysis.mdc)
- **Emitted:** 2026-11-10 · **Tech deepen:** 2026-11-10T12:24:53Z
- **Delivery class:** tech-arch + subtaks Present — no Apply / instruction body patch this wave

## Scope (requirements)

1. Citizen-facing replies **MUST NOT** present internal roles or pipeline stages (**extractor**, **checker**, **normalizer**, **HANDOFF**, «следующий процесс», and equivalents) as operable steps the user should invoke.
2. Roles **MUST NOT** claim that handoff, validation, or send **completed** or that material was «передан» to a next process when server CTAs / confirm FSM did not actually advance that step.
3. Roles **MUST NOT** coach free-text commands («дальше», «валидируй», «отправляй», «запускай», «передай экстрактору») as the **primary** product path when in-chat server actions are the intended control surface.
4. Roles **MAY** honestly say the assistant cannot invent status, number, or SPA URL, and **MAY** point to in-chat actions **when** those actions are the server path (peer 71/72).
5. Roles **MUST** preserve ROL-03 / `core.md` bans: no model-owned gateway stash; no inventing SPA redirect URLs or Bearer/HTTP submit.

## Out of scope

- Minting channel actions / aibridge code — STORY-71/72.  
- Surface formatting (air, emoji) — ROL-06 Keep.  
- Re-authoring ROL-01…04 / archive Module-1.  
- Applying `instructions-roles/**` patches in this deepen wave (name≠create until Apply dialogue).

## Target technical architecture

SSOT: [`architecture/req15-tg-interviewer-roles-ux/`](../../../architecture/req15-tg-interviewer-roles-ux/00-overview.md) · [`03-citizen-surface-no-role-theater.md`](../../../architecture/req15-tg-interviewer-roles-ux/03-citizen-surface-no-role-theater.md).

- **Dual-surface:** role files **MAY** name role ids for the model; citizen `reply_text` **MUST NOT** leak operable theater.
- **Ownership:** follow Present CTAs (peer 71); empty actions → honest wait/refuse — never theater substitute.
- **Touch:** `core.md` covenant + `interviewer.md` utterances + fence lines on other role md.
- Keep ROL-03 stash bans; do not invent action schemas.

## Acceptance (product)

| ID | Criterion |
|----|-----------|
| AC-ROL-07-01 | Instructions forbid citizen-facing operable role/pipeline jargon (extractor/checker/HANDOFF/«следующий процесс» as user steps) |
| AC-ROL-07-02 | Instructions forbid claiming handoff/validation/send progress without real server advance |
| AC-ROL-07-03 | Instructions forbid free-text command coaching as the primary path when CTAs are the product path |
| AC-ROL-07-04 | Model-owned stash / invent SPA-URL bans remain Present (ROL-03 Keep) |

## Подзадачи (технические)

> **Delivery class этой волны:** постановка T01–T07 + tech-arch. **Не** патчить `instructions-roles/**` здесь — Apply/P3 в другом диалоге.

| ID | Task | AC | Done when |
|----|------|-----|-----------|
| **T01** | **Audit** volume vs north star §4 / gap §4.1–4.4: citizen-leaking role jargon, fake progress, free-text coaching as primary | — | Short audit note listing gaps |
| **T02** | Patch **`core.md`**: **Citizen surface covenant** — forbid operable role/pipeline jargon; forbid fake handoff/send claims; forbid command school as primary; MAY point to Present CTAs / honest limits; **keep** stash bans | 01–04 | Covenant Present; bans intact |
| **T03** | Patch **`interviewer.md`**: utterance rules for calm draft talk; no HANDOFF/extractor/checker as user steps; no «передал» without server advance | 01–03 | interviewer.md updated |
| **T04** | Fence lines in **`extractor.md` / `checker.md` / `normalizer.md` / `communication_setup.md`**: internal handoff tables OK; citizen-visible text MUST follow core covenant | 01,02 | Fence Present on each |
| **T05** | Soft-dep note (prose or README): align with **STORY-71** (and **72** sequencing) — do not promise CTAs server will not mint; empty actions → no theater substitute | 02,03 | Explicit ownership sentence Present |
| **T06** | **Grep gate** (citizen-facing patterns): `HANDOFF`, «экстрактор/чекер/нормализатор» as user steps, «передал», «следующий процесс», coaching «валидируй/запускай/передай» as primary; stash invent still banned | 01–04 | Grep clean or findings fixed |
| **T07** | AC checklist AC-ROL-07-01…04 Pass | all | Gate recorded |

### Locked touch paths

| Path | Role |
|------|------|
| `GPT-UI/instructions-roles/core.md` | T02 |
| `GPT-UI/instructions-roles/interviewer.md` | T03 |
| `…/extractor.md`, `checker.md`, `normalizer.md`, `communication_setup.md` | T04 |

## Verified current state (honesty)

| Fact | Status |
|------|--------|
| ROL-05/06 Done (Awaiting Commits) | Proactivity / readability Present in instructions |
| Live 2026-11-10 handoff-chain | Bot emitted HANDOFF / extractor / checker theater; user commanded those names |
| Server `actions_count=0` same turns | Theater ≠ real progress |
| Citizen covenant in core | **Present** (P3 `2026-11-10T12:35:03Z`) |

## Desired

Citizen hears a calm helper about their draft — never an internal ops console; no fake pipeline progress; no command school.

## Related

- Sibling: [STORY-GPT-ROL-05](STORY-GPT-ROL-05-channel-interview-stage-proactivity.md) · [STORY-GPT-ROL-06](STORY-GPT-ROL-06-channel-interview-surface-readability.md)  
- Peer runtime: [STORY-AIBRIDGE-71](../../../../../doge-ai-bridge/docs/tasks/backlog-stories/req15-tg-channel-interviewer-ux/STORY-AIBRIDGE-71-awaiting-interpretation-actions-always.md) · [STORY-AIBRIDGE-72](../../../../../doge-ai-bridge/docs/tasks/backlog-stories/req15-tg-channel-interviewer-ux/STORY-AIBRIDGE-72-send-after-draft-confirm-and-validation.md)  
- Arch: [00-overview](../../../architecture/req15-tg-interviewer-roles-ux/00-overview.md) · [03](../../../architecture/req15-tg-interviewer-roles-ux/03-citizen-surface-no-role-theater.md)  
