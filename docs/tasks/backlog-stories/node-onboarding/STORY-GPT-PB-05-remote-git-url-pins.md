# STORY-GPT-PB-05 — Remote git URL pins (standards + primary)

## Meta
- **Key:** `STORY-GPT-PB-05-remote-git-url-pins`
- **Пакет:** [`node-onboarding/`](./INDEX.md)
- **Status:** Done
- **Приоритет:** P1 — follow-up after GPT-PB-01…04 Done
- **Тип:** product / remote pin (docs only in `instructions/node-onboarding/`)
- **Parent REQ:** [`REQ-46`](../../../requirements/REQ-46-pack-builder-prompt-wrapper-and-repair-debug.md) §5.0 wrapper pin · §6.1–6.2 · AC-08 (URL/tag fields)
- **Depends:** GPT-PB-01…04 Done (artifacts on disk + pushed)
- **Goal:** Операторы и Pack Builder **вне этого IDE** тянут primary + validation standards по HTTPS git URLs.
- **Evidence:** wrapper §0–§1 + primary §1 + repair §2.3 + README remote pins · REF=`dev` · pin commit `7f70ea64c2be6359986ec86e1cb1f0c309a9cff1` (re-pin after push)

## Verified (facts)

| Fact | Evidence |
|------|----------|
| Git root | `GPT UI/` → remote `https://github.com/DOGE-Complaints/gpt-ui.git` |
| Branch with artifacts | `dev` @ `7f70ea6` (wrapper/README) |
| Repo-relative paths | `instructions/node-onboarding/<file>` (**не** prefix `GPT UI/`) |
| Wrapper placeholder | `pack-builder-wrapper.md` §0 **Primary raw/tag URL** = `[[OPERATOR_FILLS …]]` |
| Primary/repair | Standards listed by **filename only** — no remote URLs yet |

## URL template (use these)

**Raw (fetch body outside IDE / Custom GPT knowledge fetch):**

```text
https://raw.githubusercontent.com/DOGE-Complaints/gpt-ui/<REF>/instructions/node-onboarding/<FILE>
```

**Browser blob (optional, human):**

```text
https://github.com/DOGE-Complaints/gpt-ui/blob/<REF>/instructions/node-onboarding/<FILE>
```

`<REF>` = pin target: prefer **git tag** or **commit SHA**; until tag exists use `dev` and record SHA in wrapper §0.

### Concrete targets

| Role | FILE |
|------|------|
| Primary (wrapper pin) | `pack-builder-primary-prompt.md` |
| Standards (primary + repair) | `pack-builder-pack.schema.json` |
| | `pack-builder-payload-schema.schema.json` |
| | `pack-builder-taxonomy.schema.json` |
| | `pack-builder-overlays.checklist.md` |

Example (branch `dev`, replace with tag when cut):

```text
https://raw.githubusercontent.com/DOGE-Complaints/gpt-ui/dev/instructions/node-onboarding/pack-builder-primary-prompt.md
https://raw.githubusercontent.com/DOGE-Complaints/gpt-ui/dev/instructions/node-onboarding/pack-builder-pack.schema.json
https://raw.githubusercontent.com/DOGE-Complaints/gpt-ui/dev/instructions/node-onboarding/pack-builder-payload-schema.schema.json
https://raw.githubusercontent.com/DOGE-Complaints/gpt-ui/dev/instructions/node-onboarding/pack-builder-taxonomy.schema.json
https://raw.githubusercontent.com/DOGE-Complaints/gpt-ui/dev/instructions/node-onboarding/pack-builder-overlays.checklist.md
```

## Scope (P3 — this story)

1. **`pack-builder-wrapper.md`**  
   - Fill §0 **Primary raw/tag URL** with raw URL to primary.  
   - Set **Primary pin tag / commit** to real tag or `7f70ea6`/`dev` SHA.  
   - Keep repo path column; add raw URLs for the four standards in §1 (or a small «Remote bind» table).

2. **`pack-builder-primary-prompt.md`**  
   - In §1 validation standards: keep local filenames **and** add the four (+ checklist) **raw git URLs** so a remote session can fetch without local checkout.

3. **`pack-builder-repair-debug-prompt.md`**  
   - Same: standards path refs + raw URLs (slots / companion table).

4. **Optional (same PR):** one «Remote URLs» row in `pack-builder-README.md` pointing at wrapper §0 as SSOT for pins.

## Out of scope

- Hosting URLs in dogestonia-app (REQ-APP-04 F1).  
- Changing meta-schema JSON bodies.  
- Cutting a release tag (recommend as operator step; story can use `dev` + SHA if no tag yet).  
- Making private-repo auth work inside ChatGPT (see Risk).

## Risk / note

If `DOGE-Complaints/gpt-ui` is **private**, anonymous `raw.githubusercontent.com` fails outside authenticated git. Then either: (a) public raw / release assets, or (b) app-hosted raw (REQ-APP-04 F1). Document which path operator uses in wrapper §0.

Проверено, все работает без авторизации, ссылки публичны.

## Acceptance Criteria

- [x] Wrapper §0 has real primary raw URL (no `[[OPERATOR_FILLS]]`).
- [x] Primary §1 lists raw URLs for pack / payload / taxonomy schemas + overlays checklist.
- [x] Repair template lists the same standards raw URLs.
- [x] Relative paths retained (IDE / checkout still work).
- [x] Pin REF documented (tag preferred, else `dev` + commit).

## Sequence

```text
1) Confirm REF (tag or commit on pushed `dev`)
2) Edit wrapper → primary → repair (+ README optional)
3) Commit/push; smoke: curl -I each raw URL from outside IDE
```
