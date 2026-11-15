# Locked Notes

| | |
|---|---|
| **Category** | Forensics |
| **Points** | 100 |
| **Author** | Kshitij Raj |
| **Artifact** | `securenotes_data.tar.gz` (Android app data) |
| **Flag** | `InIt{s3cur3_n0t3s_n0t_s0_s3cur3}` |

## Challenge

> We pulled the app data from a suspect's Android phone during an investigation. The "Secure Notes" app looked locked on the device, but the data folder came off cleanly. We think they stashed something in there. Recover the hidden note.

## TL;DR

The "lock" is only a flag in the UI. The locked note is stored as a gzip blob in the app's SQLite database; gunzip it to get the flag.

## Solution

### 1. Survey the app data

```text
com.larkfield.securenotes/
  databases/notes.db
  shared_prefs/com.larkfield.securenotes_preferences.xml
  cache/image_cache.tmp
  files/onboarding_seen
logcat.txt
```

- `logcat.txt` shows the app unlocking via biometrics and then opening **note `_id=4` (locked)**, which points straight at that note.
- The preferences file holds a `last_backup_token`, but it base64-decodes to `larkfield-session-not-the-flag`, a decoy.

### 2. Read the database

```python
import sqlite3
c = sqlite3.connect('notes.db')
for r in c.execute('select * from notes'):
    print(r)
```

Notes 1, 2, 3 and 5 are ordinary plaintext. Note 4 is titled **Recovery Code**, has `is_locked = 1`, and its body is a BLOB starting with `1f 8b 08`, the gzip magic number. The data is compressed, not encrypted.

### 3. Decompress

```python
import gzip, sqlite3
body = sqlite3.connect('notes.db').execute(
    "select body from notes where _id=4").fetchone()[0]
print(gzip.decompress(body).decode())
# InIt{s3cur3_n0t3s_n0t_s0_s3cur3}
```

The output is already the flag in plaintext. A ROT13 pass produces gibberish, so no further decoding is needed.

## Flag

```
InIt{s3cur3_n0t3s_n0t_s0_s3cur3}
```

## Takeaways

An app-level lock that does not actually encrypt the stored data offers no protection once the data directory has been extracted.
