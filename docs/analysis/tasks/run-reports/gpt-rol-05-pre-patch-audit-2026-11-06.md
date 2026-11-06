# ROL-05 pre-patch audit — Goals 1/2/4 gaps

**UTC:** `2026-11-06T11:58:46Z` · **GIM-468** · volume `GPT-UI/instructions-roles/`

| Goal / topic | As-is | Gap |
|--------------|-------|-----|
| **1 Auto-advance** | `core` chain order Present; `interviewer` has phase discipline but **no** rule to advance without user «дальше/продолжай/проверяй» | **Missing** proactive MUST |
| **2 Handoff / actions** | Stash/SPA invent bans Present; no align with server continue/edit/Send actions; no `continuation_url` post-confirm wording | **Missing** handoff contract |
| **4 Anti vague-UI** | No citizen-facing «интерфейс» sole-path found | **Weak** — not explicitly forbidden |
| Stash strip (ROL-03) | Present in core / interviewer / checker / normalizer / README | **Keep** |
| STORY-68 ownership | Absent | **Missing** sentence |

**T04 preview:** checker/normalizer handoffs already server-owned; no bare-UI sole-path → expect **N/A** unless inconsistency after core patch.
