# 01 — Module-1: слои инструкций и strict chain

**Зона:** [`architecture/`](./README.md)  
**As-of:** 2026-09-16  
**SSOT:** [`instructions/`](../../instructions/) · карта [`instruction-modules-index.md`](../../instructions/instruction-modules-index.md) v0.44

---

## 1. Иерархия слоёв

| Слой | Файл | Роль (as-built) |
|------|------|-----------------|
| Root wrapper | [`root.md`](../../instructions/root.md) | Identity, приоритеты слоёв, bootstrap до роутинга |
| Bootstrap | [`bootstrap.md`](../../instructions/bootstrap.md) | `comm_context` (язык, tone, verbosity) **до** functional modules |
| Constitution | [`base.md`](../../instructions/base.md) | Глобальные mode/contracts Module 1 |
| Navigation | [`instruction-modules-index.md`](../../instructions/instruction-modules-index.md) | Runtime map; active path = DOGEstonia Issue |
| Interview / i18n / lifecycle | `story-interview-flow.md`, `story-i18n-policy.md`, `story-lifecycle-instructions.md`, … | Диалог и handoff артефактов |
| Ingest / safety / gate / normalizer | `ingest-*.md`, `safety-compliance.md`, `story-policy-gate.md`, `story-normalizer.md` | **Без** прямых HTTP |
| HTTP only | [`api-orchestrator.md`](../../instructions/api-orchestrator.md) | Единственный модуль, которому разрешены Actions/HTTP |
| Wire companion | [`story-api-methods-reference.md`](../../instructions/story-api-methods-reference.md) | Lockstep с OpenAPI |
| Node overlay | [`schema-packs.README.md`](../../instructions/schema-packs.README.md) + six flat files | Active pair — [02](./02-schema-packs-and-binding.md) |

**Не Module-1 Interview upload:** [`instructions/node-onboarding/`](../../instructions/node-onboarding/) — Pack Builder tooling ([SSR-18](../tasks/backlog-stories/semantic-schema-runtime/STORY-GPT-SSR-18-core-pack-portability-strip-node-specific.md)).

---

## 2. Mandatory execution chain

Из index (`instruction-modules-index.md` L45–48):

```text
ingest-validation → safety-compliance → story-policy-gate → story-normalizer → api-orchestrator
```

с `base.md` как constitution и `story-lifecycle-instructions.md` как reference chain.

Правило: validation / gate / normalizer **never call APIs directly** — только orchestrator (index L54–59; [`technical-architecture.md`](../technical-architecture.md) §1).

---

## 3. Upload set vs runtime order

| Понятие | Статус |
|---------|--------|
| **Runtime order** модулей | Documented (строка выше + lifecycle) |
| **Порядок файлов** в ChatGPT Knowledge | **Unknown** в репо — есть состав набора (operator manuals / SSR-18 contour), не sort-key upload |
| Flat pack filenames | Documented — Custom GPT отбрасывает субдиректории ([`schema-packs.README.md`](../../instructions/schema-packs.README.md) L4) |

Shared Module-1 contour (SSR-18): `root / base / story-* / ingest-* / orchestrator / index` + README + `schema-packs.<id>.<ver>.*` ×6.

---

## 4. Verified discrepancies (не игнорировать)

| Claim | Где | Correct as-built |
|-------|-----|------------------|
| Active pair `tallinn_civic`/`v1`; «swap two files» | `instruction-modules-index.md` L60 | Live: `uus_veerenni_civic`/`v3`; swap = README + **six** flat files ([`schema-packs.README.md`](../../instructions/schema-packs.README.md) L18–28) |
| Links `schema-packs/README.md` | index L13, L29, L60; многие core modules | Live file = flat `schema-packs.README.md` (каталог nested не upload SSOT) |

Постановка на strip: [STORY-GPT-SSR-18](../tasks/backlog-stories/semantic-schema-runtime/STORY-GPT-SSR-18-core-pack-portability-strip-node-specific.md).

---

## 5. Dual-channel note

Тот же Module-1 текст **читается** по-разному:

- Actions: Knowledge/Instructions в Custom GPT.  
- Bridge: subset через `instructions.manifest.json` — [05](./05-python-bridge-instruction-delivery.md).
