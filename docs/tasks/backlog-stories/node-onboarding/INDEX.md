# Node onboarding — Pack Builder backlog package

**Зона:** `GPT UI/docs/tasks/backlog-stories/node-onboarding/`  
**Кодовое имя пакета:** `node-onboarding`  
**Тип:** backlog (постановка).  
**Метод:** [`.cursor/rules/analysis.mdc`](../../../../../.cursor/rules/analysis.mdc) — факты с paths.  
**Parent REQ:** [`REQ-46-pack-builder-prompt-wrapper-and-repair-debug.md`](../../../requirements/REQ-46-pack-builder-prompt-wrapper-and-repair-debug.md) (Draft) · product parent [`REQ4` §5.1](../../../../../docs/requirements%20backlog/REQ4-Node-Onboarding-Schema-Pack-Federation-Gate.md)  
**Related runtime SSOT:** [`REQ-45a`](../../../requirements/REQ-45a-pack-prose-identities-and-flat-layout.md)  
**Related Part A meta gaps:** [`uus-veerenni-civic-v1-gateway-load-blockers-handoff-2026-09-04.md`](../../../../../doge-complaints-gateway/docs/analysis/uus-veerenni-civic-v1-gateway-load-blockers-handoff-2026-09-04.md) **Part A only** → GPT-PB-06…09  
**Related audit:** [`uus-veerenni-civic-v1-pack-validation-audit-2026-09-05.md`](../../../../../docs/analysis/uus-veerenni-civic-v1-pack-validation-audit-2026-09-05.md) → GPT-PB-10 (primary elicit)  
**Related app:** [`REQ-APP-04`](../../../../../dogestonia-app/docs/requirements/REQ-APP-04-node-onboarding-wave1.md) F2 (validate/repair gate — not this package)  
**Related scripts tooling:** [`STORY-SCR-NODEONB-01`](../../../../../scripts/docs/tasks/backlog-stories/node-onboarding/STORY-SCR-NODEONB-01-node-pack-validation-script.md) · [`STORY-SCR-NODEONB-02`](../../../../../scripts/docs/tasks/backlog-stories/node-onboarding/STORY-SCR-NODEONB-02-validator-geo-load-gate-hard-fail.md) (geo hard-fail)  
**Related smoke (uus v2 geo):** [`uus-veerenni-civic-v2-active-volume-smoke-2026-09-06.md`](../../../../../doge-complaints-gateway/docs/analysis/uus-veerenni-civic-v2-active-volume-smoke-2026-09-06.md) → GPT-PB-12  
**Related standards align:** GPT-PB-13 (primary/repair/checklist/wrapper vs NODEONB-02)  
**Related instance ship:** [`.cursor/plans/uus-veerenni-civic-v3-pack-ship.plan.md`](../../../../../.cursor/plans/uus-veerenni-civic-v3-pack-ship.plan.md) (uus v2→v3)

## Architecture (one-liner)

Pack Builder tooling under `GPT UI/instructions/node-onboarding/` (primary + repair + validation standards + wrapper/README). Emits **flat** `schema-packs.*` per REQ-45a. **Not** Module-1 interview Instructions. App/gateway admission gates stay outside.

## Контекст одной фразой

Оператор ноды через NL получает candidate schema pack → validate → repair loop → REQ4 F2 app gate. Runtime interview GPT ортогонален (REQ-45 / 45a). Wave GPT-PB-06…09 + SCR-NODEONB-02 закрывают **false-admission** meta/CLI↔gateway. GPT-PB-10…12 — intent geo/load gates (bodies pending). **GPT-PB-13** — догнать instruction set под hard geo gates. Instance uus → **v3 plan** (не эта стори).

## Stories

| Order | Key | Story | Status | Depends |
|------|-----|-------|--------|---------|
| 1 | GPT-PB-01 | [primary prompt emit flat pack](./STORY-GPT-PB-01-primary-prompt-emit-flat-pack.md) | Done | pkg-000051 GIM-357…361 · primary on disk |
| 2 | GPT-PB-02 | [repair-debug prompt loop](./STORY-GPT-PB-02-repair-debug-prompt-loop.md) | Done | pkg-000052 GIM-362…365 · repair on disk · P8 `d6a8cdb` |
| 3 | GPT-PB-03 | [validation meta-schemas + overlays checklist](./STORY-GPT-PB-03-validation-meta-schemas-and-overlays-checklist.md) | Done | pkg-000053 GIM-366…371 · standards on disk · P8 `5345f25`+`cef7b32` |
| 4 | GPT-PB-04 | [wrapper pin + standards hash (+ README)](./STORY-GPT-PB-04-wrapper-pin-and-standards-hash.md) | Done | pkg-000054 GIM-372…376 · wrapper+README on disk |
| 5 | GPT-PB-05 | [remote git URL pins](./STORY-GPT-PB-05-remote-git-url-pins.md) | Done | raw URLs in wrapper §0–§1 + primary §1 + repair §2.3 |
| 6 | GPT-PB-06 | [harden exact_lenses + civic clustering meta](./STORY-GPT-PB-06-harden-exact-lenses-and-civic-clustering-meta.md) | Done (Awaiting Commits) | pkg-000055 GIM-378…385 🟢 P3 2026-09-04T19:15:31Z |
| 7 | GPT-PB-07 | [enum-lock geo_intake + field_policy](./STORY-GPT-PB-07-enum-lock-geo-intake-and-field-policy.md) | Done (Awaiting Commits) | pkg-000056 GIM-387…393 🟢 P3 2026-09-04T19:28:14Z |
| 8 | GPT-PB-08 | [taxonomy structural + invert SSR-15](./STORY-GPT-PB-08-taxonomy-structural-free-axes-invert-ssr15.md) | Done (Awaiting Commits) | pkg-000057 GIM-394…400 🟢 P3 2026-09-04T19:37:37Z |
| 9 | GPT-PB-09 | [optional-keys drift + checklist load gates](./STORY-GPT-PB-09-optional-keys-drift-and-checklist-load-gates.md) | Done (Awaiting Commits) | pkg-000058 GIM-401…407 🟢 P3 2026-09-04T19:44:48Z |
| 10 | GPT-PB-10 | [primary elicit load gates before first validation](./STORY-GPT-PB-10-primary-prompt-elicit-load-gates.md) | Done (Awaiting Commits) | pkg-000059 GIM-408…413 🟢 P3 story-only 2026-09-05T12:27:59Z |
| 11 | GPT-PB-11 | [primary enforce load-gate knobs before emit](./STORY-GPT-PB-11-primary-prompt-enforce-load-gate-emit.md) | Done (Awaiting Commits) | pkg-000060 GIM-414…419 🟢 P3 story-only 2026-09-06T11:37:37Z |
| 12 | GPT-PB-12 | [primary geo load-gate vocabulary (uus v2)](./STORY-GPT-PB-12-primary-prompt-geo-load-gate-vocabulary.md) | Done (Awaiting Commits) | pkg-000061 GIM-421…426 🟢 P3 Layer 1+2 analysis 2026-09-06T13:43:19Z |
| 13 | GPT-PB-13 | [align node-onboarding instructions with geo hard gates](./STORY-GPT-PB-13-node-onboarding-standards-geo-gate-alignment.md) | Done (Awaiting Commits) | pkg-000062 GIM-428…433 🟢 P3 story постановка 2026-09-07T08:03:42Z · instruction bodies deferred |

## Progress

| Metric | Value |
|--------|-------|
| Stories total | 13 |
| Done | 13 |
| In Progress | 0 |
| Todo | 0 |
| **Progress** | **13/13 (100%)** · PB-13 постановка Done · instruction bodies future P3 |

## Recommended sequence

```text
PB-01…05 Done
  → PB-06…09 Done (meta + checklist Part A)
  → PB-10…12 Done story/Layer2 (primary body pending)
  → SCR-NODEONB-02 Done (CLI+meta geo hard)
  → PB-13 постановка Done; instruction-body P3 deferred (primary/repair/checklist/wrapper)
Parallel: uus v2→v3 instance plan (.cursor/plans/uus-veerenni-civic-v3-pack-ship.plan.md)
```

## Граница

- **In:** Pack Builder prompts + validation standards under `instructions/node-onboarding/`; emit = REQ-45a flat set; Part A meta harden; PB-13 story постановка Done (instruction bodies → future P3).  
- **Out:** Module-1 process edits; instance uus v3 ship (→ plan); gateway SchemaRuntime code; empty stubs; scripts CLI (→ SCR-NODEONB-*).
