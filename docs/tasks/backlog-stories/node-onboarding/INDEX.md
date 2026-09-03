# Node onboarding — Pack Builder backlog package

**Зона:** `GPT UI/docs/tasks/backlog-stories/node-onboarding/`  
**Кодовое имя пакета:** `node-onboarding`  
**Тип:** backlog (постановка).  
**Метод:** [`.cursor/rules/analysis.mdc`](../../../../../.cursor/rules/analysis.mdc) — факты с paths.  
**Parent REQ:** [`REQ-46-pack-builder-prompt-wrapper-and-repair-debug.md`](../../../requirements/REQ-46-pack-builder-prompt-wrapper-and-repair-debug.md) (Draft) · product parent [`REQ4` §5.1](../../../../../docs/requirements%20backlog/REQ4-Node-Onboarding-Schema-Pack-Federation-Gate.md)  
**Related runtime SSOT:** [`REQ-45a`](../../../requirements/REQ-45a-pack-prose-identities-and-flat-layout.md) · Wave 4 [`GPT-SSR-11…13`](../semantic-schema-runtime/INDEX.md) (operator executes **before** Pack Builder P3 — stories written against **Done** flat+locale target)  
**Related app:** [`REQ-APP-04`](../../../../../dogestonia-app/docs/requirements/REQ-APP-04-node-onboarding-wave1.md) F2 (validate/repair gate — not this package)

## Architecture (one-liner)

Pack Builder tooling under `GPT UI/instructions/node-onboarding/` (primary + repair + validation standards + wrapper/README). Emits **flat** `schema-packs.*` per REQ-45a. **Not** Module-1 interview Instructions. App/gateway gates outside.

## Контекст одной фразой

Оператор ноды через NL получает candidate schema pack → validate → repair loop → REQ4 F2 app gate. Runtime interview GPT ортогонален (REQ-45 / 45a).

## Stories

| Order | Key | Story | Status | Depends |
|------|-----|-------|--------|---------|
| 1 | GPT-PB-01 | [primary prompt emit flat pack](./STORY-GPT-PB-01-primary-prompt-emit-flat-pack.md) | Done | pkg-000051 GIM-357…361 · primary on disk |
| 2 | GPT-PB-02 | [repair-debug prompt loop](./STORY-GPT-PB-02-repair-debug-prompt-loop.md) | Done | pkg-000052 GIM-362…365 · repair on disk · P8 `d6a8cdb` |
| 3 | GPT-PB-03 | [validation meta-schemas + overlays checklist](./STORY-GPT-PB-03-validation-meta-schemas-and-overlays-checklist.md) | Done | pkg-000053 GIM-366…371 · standards on disk · P8 `5345f25`+`cef7b32` |
| 4 | GPT-PB-04 | [wrapper pin + standards hash (+ README)](./STORY-GPT-PB-04-wrapper-pin-and-standards-hash.md) | Done | pkg-000054 GIM-372…376 · wrapper+README on disk |
| 5 | GPT-PB-05 | [remote git URL pins](./STORY-GPT-PB-05-remote-git-url-pins.md) | Done | raw URLs in wrapper §0–§1 + primary §1 + repair §2.3 |

## Progress

| Metric | Value |
|--------|-------|
| Stories total | 5 |
| Done | 5 |
| Todo | 0 |
| **Progress** | **5/5 (100%)** |

## Recommended sequence

```text
PB-01…05 Done
```

## Граница

- **In:** Pack Builder prompts + validation standards under `instructions/node-onboarding/`; emit = REQ-45a flat set.  
- **Out:** Module-1 process edits; runtime flat migrate (SSR-11…13); app/gateway engines; empty stubs «чтобы path существовал».
