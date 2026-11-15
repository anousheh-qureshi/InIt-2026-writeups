# deleted evidence

| | |
|---|---|
| **Category** | Forensics |
| **Points** | 100 |
| **Author** | Kshitij Raj |
| **Artifact** | `evidence.img` (12 MB disk image) |
| **Flag** | `InIt{d3l3t3d_but_n0t_g0n3}` |

## Challenge

> A drive was seized before it could be wiped. The suspect is certain the evidence is gone for good. But on a filesystem, *Delete doesn't mean erased, it just means forgotten.* Find what they tried to bury.

## TL;DR

A deleted NTFS file (`recovery_code.gz`) was still intact in the MFT. Gunzip it, then ROT13-decode the result.

## Solution

### 1. Identify the image

`file` reports a DOS/MBR boot sector with one partition of type `0x07` (NTFS/exFAT) starting at sector 2048 (22528 sectors).

```bash
file evidence.img
dd if=evidence.img of=part.img bs=512 skip=2048 count=22528
```

The boot sector of the carved partition shows the `NTFS` OEM ID:

- 512 bytes/sector, 8 sectors/cluster, so 4096-byte clusters
- MFT starts at cluster 4
- 1024-byte MFT records

### 2. Walk the MFT

Every NTFS file has a record in the Master File Table. Deleting a file only clears the "in use" flag (bit 0) of its record; the filename and data attributes remain.

No Sleuth Kit was available, so I wrote a small parser that:

1. Applies the update-sequence fixups to each 1024-byte record
2. Reads the `$FILE_NAME` (0x30) and `$DATA` (0x80) attributes
3. Prints every record with an in-use / deleted marker and dumps the data

```text
75 DEL   ('recovery_code.gz', parent=66)  ('res', 46 bytes)
```

Record 75 is the only deleted user file, `recovery_code.gz` in `Users\j.morris\Documents`. Its `$DATA` attribute is **resident** (46 bytes stored inside the MFT record itself), so the content survived deletion.

### 3. Decode

```console
$ zcat 75_DEL_recovery_code.gz
VaVg{q3y3g3q_ohg_a0g_t0a3}

$ echo 'VaVg{q3y3g3q_ohg_a0g_t0a3}' | tr 'A-Za-z' 'N-ZA-Mn-za-m'
InIt{d3l3t3d_but_n0t_g0n3}
```

`VaVg` is ROT13 for `InIt`, which confirms the cipher.

> Decoy: `wifi_backup.txt` is only base64 for `my-wifi-password`.

## Flag

```
InIt{d3l3t3d_but_n0t_g0n3}
```

## Takeaways

- Deleting a file on NTFS only marks its MFT record as free.
- Small files are stored resident inside the record, so they stay recoverable until the record is reused.
