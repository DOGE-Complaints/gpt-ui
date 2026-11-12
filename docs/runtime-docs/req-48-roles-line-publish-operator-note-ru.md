# REQ-48 — Публикация role-oriented линии (операторская заметка V2)

**As-of:** 2026-11-02T14:39:55Z  
**AC:** AC-GPT-ROL04-03 (= AC-GPT-48-07)  
**Решение publish:** **V2 manual** (нет CI sync GPT UI → aibridge image; V3 rejected).

## Три Git homes (не путать)

| Home | Путь | Роль |
|------|------|------|
| **Working / C2** | `GPT UI/instructions/` | Текущая Custom GPT continuity (не удалять ради REQ-48) |
| **Archive Module-1** | `GPT UI/instructions-archive-module1-req48/` | As-was snapshot (ROL-01) |
| **New roles line** | `GPT UI/instructions-roles/` | Authoring + export SSOT новой линии (ROL-03/04) |

Archive ≠ new line: архив — снимок; publish volume для новой линии берёт **sibling** `instructions-roles/`, не `instructions/archive/` и не весь Module-1 working tree.

## Env cutover

| Переменная | Значение |
|------------|----------|
| `DOGESTONIA_INSTRUCTIONS_DIR` | Корень sibling (или копия на volume) — `…/instructions-roles` |
| `DOGESTONIA_INSTRUCTIONS_MANIFEST` | Файл `roles.manifest.json` внутри этого корня |

**Phase-A `files[]`** — не SSOT новой линии. Не переключайте consume на Phase-A list как «новую» схему ролей.

## Ручная публикация на volume (V2)

1. Убедиться, что в Git sibling есть **полный** intended set: `roles.manifest.json` + `core.md` + пять role-файлов (см. Export Pass evidence).  
2. Скопировать **весь** корень sibling на смонтированный volume / путь, который читает aibridge (`DOGESTONIA_INSTRUCTIONS_DIR`).  
3. Выставить `DOGESTONIA_INSTRUCTIONS_MANIFEST` на `roles.manifest.json` в этом корне.  
4. Не публиковать **подмножество** файлов без обновления intended inventory / evidence — это ломает честность Export Pass (AC-06).  
5. **Не** ожидать CI/image sync script — его нет (REQ-48 §3); ручной copy = норма v1.

## Связанные артефакты

- Export Pass evidence (E1): [`../analysis/req-48-export-parity-evidence.md`](../analysis/req-48-export-parity-evidence.md) — **не** Consume Pass, **не** joint REQ-13 wire SSOT.  
- Arch: [`../architecture/req-48-role-oriented-instruction-line/03-publish-verify-parity.md`](../architecture/req-48-role-oriented-instruction-line/03-publish-verify-parity.md)  
- Peer honesty (Consume Partial): `doge-ai-bridge/docs/runtime-docs/11-content-volume-parity-stub-live-evidence.md`

## Out of scope этой заметки

- Live Railway Consume Pass.  
- Совместный inventory/hash SSOT с REQ-13 (остаётся Open).  
- Написание CI sync pipeline.
