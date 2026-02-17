# Shadow Maintenance — Write-up

## Goal
Derive the ZIP password from `modbus.json` telemetry metadata and recover `flag.txt` from `maintenance.zip`.

---

## Files
- `modbus.json`
- `maintenance.zip`

---

## Step 1 — Filter the telemetry (unknown-34 REQ only)

**Filter criteria**
Select only records where:
- `func == "unknown-34"`
- `pdu_type == "REQ"`

Then sort by `ts` ascending.

### Command (quick sanity check)
```bash
python3 - << 'PY'
import json
n=0
with open("modbus.json","r",encoding="utf-8") as f:
  for line in f:
    j=json.loads(line)
    if j.get("func")=="unknown-34" and j.get("pdu_type")=="REQ":
      n+=1
print(n)
PY
```

---

## Step 2 — Build the per-event “pair” value

**Transformation rule** (per selected event)
- `unit` -> zero-padded 2 digits  
- last octet of `id.resp_h` -> zero-padded 2 digits  
- concatenate as `UNIT+OCTET` (no separator)

Example:
- `unit=32`, `id.resp_h=10.47.8.51` -> `3251`

---

## Step 3 — Ordered values (full)

Format: `timestamp | unit | id.resp_h | pair`

```
2018-03-24T17:15:58.290250Z | unit=32 | resp=10.47.8.51 | pair=3251
2018-03-24T17:15:59.524209Z | unit=32 | resp=10.47.8.52 | pair=3252
2018-03-24T17:16:00.684137Z | unit=32 | resp=10.47.8.53 | pair=3253
2018-03-24T17:16:27.061563Z | unit=32 | resp=10.47.3.51 | pair=3251
2018-03-24T17:16:28.214083Z | unit=32 | resp=10.47.3.52 | pair=3252
2018-03-24T17:16:29.453374Z | unit=32 | resp=10.47.3.53 | pair=3253
2018-03-24T17:17:19.912424Z | unit=32 | resp=10.47.8.51 | pair=3251
2018-03-24T17:17:21.221104Z | unit=32 | resp=10.47.8.52 | pair=3252
2018-03-24T17:17:22.524547Z | unit=32 | resp=10.47.8.53 | pair=3253
2018-03-24T17:17:49.106218Z | unit=32 | resp=10.47.3.51 | pair=3251
2018-03-24T17:17:50.259683Z | unit=32 | resp=10.47.3.52 | pair=3252
2018-03-24T17:17:51.414048Z | unit=32 | resp=10.47.3.53 | pair=3253
2018-03-24T17:18:41.841061Z | unit=32 | resp=10.47.8.51 | pair=3251
2018-03-24T17:18:42.998499Z | unit=32 | resp=10.47.8.52 | pair=3252
2018-03-24T17:18:44.149643Z | unit=32 | resp=10.47.8.53 | pair=3253
2018-03-24T17:19:10.778798Z | unit=32 | resp=10.47.3.51 | pair=3251
2018-03-24T17:19:11.943163Z | unit=32 | resp=10.47.3.52 | pair=3252
2018-03-24T17:19:13.101656Z | unit=32 | resp=10.47.3.53 | pair=3253
2018-03-24T17:20:03.518224Z | unit=32 | resp=10.47.8.51 | pair=3251
2018-03-24T17:20:04.690356Z | unit=32 | resp=10.47.8.52 | pair=3252
2018-03-24T17:20:06.237997Z | unit=32 | resp=10.47.8.53 | pair=3253
2018-03-24T17:20:32.799773Z | unit=32 | resp=10.47.3.51 | pair=3251
2018-03-24T17:20:33.951338Z | unit=32 | resp=10.47.3.52 | pair=3252
2018-03-24T17:20:35.110810Z | unit=32 | resp=10.47.3.53 | pair=3253
2018-03-24T17:21:25.541168Z | unit=32 | resp=10.47.8.51 | pair=3251
2018-03-24T17:21:26.707159Z | unit=32 | resp=10.47.8.52 | pair=3252
2018-03-24T17:21:27.864006Z | unit=32 | resp=10.47.8.53 | pair=3253
2018-03-24T17:21:54.233983Z | unit=32 | resp=10.47.3.51 | pair=3251
2018-03-24T17:21:55.389630Z | unit=32 | resp=10.47.3.52 | pair=3252
2018-03-24T17:21:56.551735Z | unit=32 | resp=10.47.3.53 | pair=3253
2018-03-24T17:22:46.974778Z | unit=32 | resp=10.47.8.51 | pair=3251
2018-03-24T17:22:48.825243Z | unit=32 | resp=10.47.8.52 | pair=3252
2018-03-24T17:22:49.979760Z | unit=32 | resp=10.47.8.53 | pair=3253
2018-03-24T17:23:16.354360Z | unit=32 | resp=10.47.3.51 | pair=3251
2018-03-24T17:23:17.507084Z | unit=32 | resp=10.47.3.52 | pair=3252
2018-03-24T17:23:18.821720Z | unit=32 | resp=10.47.3.53 | pair=3253
2018-03-24T17:24:09.261694Z | unit=32 | resp=10.47.8.51 | pair=3251
2018-03-24T17:24:10.414290Z | unit=32 | resp=10.47.8.52 | pair=3252
2018-03-24T17:24:11.578562Z | unit=32 | resp=10.47.8.53 | pair=3253
2018-03-24T17:24:37.953288Z | unit=32 | resp=10.47.3.51 | pair=3251
2018-03-24T17:24:39.111237Z | unit=32 | resp=10.47.3.52 | pair=3252
2018-03-24T17:24:40.302265Z | unit=32 | resp=10.47.3.53 | pair=3253
2018-03-24T17:25:30.950141Z | unit=32 | resp=10.47.8.51 | pair=3251
2018-03-24T17:25:32.106722Z | unit=32 | resp=10.47.8.52 | pair=3252
2018-03-24T17:25:33.422690Z | unit=32 | resp=10.47.8.53 | pair=3253
```

---

## Step 4 — Concatenate all pairs into the payload string `S`

### Full `S`
```
325132523253325132523253325132523253325132523253325132523253325132523253325132523253325132523253325132523253325132523253325132523253325132523253325132523253325132523253325132523253
```

---

## Step 5 — Hash and encode to get the ZIP password

**Algorithm:** `SHA-1 -> Base32 -> strip '=' -> first 20 characters`

### Full values
**SHA-1 hex digest**
```
3c0db3660cd0f125e8adb3aa845c4e2a43ce5153
```

**Base32 (unpadded)**
```
HQG3GZQM2DYSL2FNWOVIIXCOFJB44UKT
```

**Derived ZIP password**
```
HQG3GZQM2DYSL2FNWOVI
```

---

## Step 6 — Extract the archive and read the flag

### Commands
```bash
unzip maintenance.zip
# when prompted for password:
# HQG3GZQM2DYSL2FNWOVI

cat flag.txt
```

### Flag
```
MetaCTF{b50Ke_iN70_Mo4Bu5}
```

---

## Reference solver script (copy/paste)

```python
import json
import hashlib
import base64

events = []

# 1) Load telemetry and filter suspicious requests
with open("modbus.json", "r", encoding="utf-8") as f:
    for line in f:
        j = json.loads(line)

        if (
            j.get("func") == "unknown-34"
            and j.get("pdu_type") == "REQ"
        ):
            events.append(j)

# 2) Preserve ordering (timestamp)
events.sort(key=lambda x: x["ts"])

# 3) Build concatenated string
parts = []

for j in events:
    unit = int(j["unit"])
    last_octet = int(j["id.resp_h"].split(".")[-1])

    # fixed-width formatting is important
    parts.append(f"{unit:02d}{last_octet:02d}")

S = "".join(parts)

# 4) Hash + encode
digest = hashlib.sha1(S.encode("utf-8")).digest()
token = base64.b32encode(digest).decode("ascii").rstrip("=")

# 5) Final ZIP password
password = token[:20]

print("Payload string S:")
print(S)
print("\nZIP password:")
print(password)
```

---
