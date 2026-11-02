# REQ-48 Export Pass evidence (E1) — roles line

**As-of:** 2026-11-02T14:39:55Z  
**Story:** STORY-GPT-ROL-04 · AC-GPT-ROL04-02 (= AC-GPT-48-06)  
**Sibling:** `GPT UI/instructions-roles-req15/`  
**Manifest:** `GPT UI/instructions-roles-req15/roles.manifest.json`  
**bundle_version:** `req15-roles-v1`

## Labels (mandatory)

| Label | Value |
|-------|--------|
| Joint inventory/hash wire SSOT with REQ-13 | **No** — this file is GPT UI local Export Pass only; joint wire remains **Open** |
| Consume Pass (live volume) | **No** — not Railway / mounted-volume evidence |
| Runtime `session_pin` / assembled core+role hash | **Not** this inventory — do not conflate |

## Env cutover (authoring → consume config)

| Env | Intended value for new line |
|-----|-----------------------------|
| `DOGESTONIA_INSTRUCTIONS_DIR` | Sibling root `GPT UI/instructions-roles-req15/` (or published volume copy of that root) |
| `DOGESTONIA_INSTRUCTIONS_MANIFEST` | Path to `roles.manifest.json` under that root |
| Phase-A `files[]` manifest | **Not** new-line SSOT (legacy/transitional on aibridge) |

## Hash method

1. Read file bytes.  
2. `normalize_text` — UTF-8 decode + newlines → LF (`doge-ai-bridge/src/aibridge/content.py` L23–26).  
3. `sha256_hex` of UTF-8 encoding of normalized text (L19–20).  

## Intended inventory (no silent subset)

Root children required for Export Pass honesty: roles-schema manifest + `core.md` + five role files.  
`README.md` is authoring SSOT prose — **outside** roles-schema consume inventory (not listed below).

| File | sha256 (after normalize_text) | normalized_len |
|------|-------------------------------|----------------|
| `roles.manifest.json` | `b92039c373a8b0c6e99f674fd821c7ad6cdd472330c941074e14b2425067c89f` | 491 |
| `core.md` | `e905700d546aae003dc3b22e7d4652900415693b4e7232e77a1c6bea5cf97d32` | 2373 |
| `communication_setup.md` | `a2ed22e109eadbbfd176400d75d56112d2184dfbc3cffb41e72a85e0e1097bea` | 1488 |
| `interviewer.md` | `8acb15ec16005a9fc10aebfdc7da209c6e33d76c0d3e4c94c7c7606c4aa599fd` | 1516 |
| `extractor.md` | `a7c4352382c662e0ebddd610b33c188ebbdfb2fbf358df1f364484065e3dadcf` | 1478 |
| `normalizer.md` | `0af466594b964da72f9fd14c37922f422ff8ef5c602821b0f252cca550a77288` | 1390 |
| `checker.md` | `f139485bf5f234757eedd64870681e360f3826a12afe7c960b7bad81cca78b85` | 1559 |

**Count:** 7/7 Present under sibling · packing packs **excluded** (P1).

## Recompute (operator)

```bash
python3 - <<'PY'
from pathlib import Path
import hashlib, json
def sha256_hex(b): return hashlib.sha256(b).hexdigest()
def normalize_text(raw):
    t = raw.decode("utf-8")
    return t.replace("\r\n","\n").replace("\r","\n")
root = Path("GPT UI/instructions-roles-req15")
names = ["roles.manifest.json","core.md","communication_setup.md","interviewer.md","extractor.md","normalizer.md","checker.md"]
for n in names:
    t = normalize_text((root/n).read_bytes())
    print(n, sha256_hex(t.encode("utf-8")))
print("bundle_version", json.loads((root/"roles.manifest.json").read_text())["bundle_version"])
PY
```
