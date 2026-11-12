# Pack locale-jurisdiction — `uus_veerenni_civic` / `v1`

**Product:** DOGEstonia — Module 1 (GPT instruction overlay)  
**Purpose:** Working languages, place-string script policy and jurisdiction framing for the Uus Veerenni community node. Non-executable prose; Gateway Schema Runtime remains authoritative.

## 1. Working languages

Declared `working_languages`: `et`, `en`, `ru`.

- `et` — Estonian
- `en` — English
- `ru` — Russian

Reply in the resident's current supported language. Internal taxonomy keys remain English snake_case.

## 2. Place strings

For canonical location/address text:
- prefer official Estonian / Latin-script place and street names when known from confirmed resident input;
- preserve the resident's original wording when translation/transliteration would introduce uncertainty;
- do not invent street names, house numbers, EHAK/admin ids or bbox values;
- preserve street/district/landmark detail instead of collapsing it to city-only.

Examples of canonical locality framing:
- `Uus Veerenni, Tallinn`
- `Veerenni, Tallinn`
- a confirmed street/house string as supplied by the resident.

## 3. Jurisdiction and node standing

Jurisdiction frame: **Tallinn, Estonia**.

Community focus: **Uus Veerenni residents and neighbourhood life**.

This is not a hard “all coordinates must be inside Uus Veerenni” boundary. A Story is in scope when an adjacent place, route, public-transport service or other external system directly affects Uus Veerenni residents.

Machine GPT territory remains Tallinn for v1.

## 4. Resident-facing territory clarification

When locality is unclear, ask in the resident's language for the place or for the connection to Uus Veerenni.

When a place is confirmed outside Tallinn and no Uus Veerenni impact is stated:
- STOP before stash;
- ask the resident to clarify the Uus Veerenni connection;
- never invent a Tallinn/Uus Veerenni address.

For a gateway `GEO_SCOPE_MISMATCH`, explain the supported frame in the current session language: this node is for Uus Veerenni community life within the Tallinn jurisdiction context.

## 5. Language fallback

For short STOP/422 clarification:
1. use `session_language` when it is `et`, `en`, or `ru`;
2. otherwise use a supported UI language if known;
3. otherwise use English.

This file does not redefine core wire i18n shapes or translation algorithms.
