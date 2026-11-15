# CTF Writeup: Indian Railways - Golden Diamond Tour

**Category:** Forensics / Steganography / Cryptography  
**Difficulty:** Medium - Hard  
**Flag:** `InIt{j0urney_4cr0ss_1ndia}`

---

## 1. Challenge Overview

We are provided with a directory containing the following three initial files:
- `IRCTC_Tatkal_Hack.pdf` (2,384,151 bytes)
- `outer_wordlist.txt` (88,413 entries)
- `README.txt`

The initial `README.txt` presents the scenario:

> **INDIAN RAILWAYS — GOLDEN DIAMOND TOUR**  
> Congratulations! You have won an exclusive Golden Diamond Tour across India.  
> Unfortunately, someone has tampered with your ticket.  
> Your journey begins in Delhi and continues through Mumbai, Chennai, and Kolkata. The files left behind may not be what they appear to be.  
> 
> **Route:** Delhi → Mumbai → Chennai → Kolkata  
> **Objective:** Recover the original Golden Diamond Tour ticket and find the flag.

---

## 2. Step-by-Step Solution

### Step 0: The First Gate (`IRCTC_Tatkal_Hack.pdf`)

Inspecting the file signature of `IRCTC_Tatkal_Hack.pdf` reveals that despite the `.pdf` extension, the magic bytes begin with `PK\x03\x04`:
```python
with open('IRCTC_Tatkal_Hack.pdf', 'rb') as f:
    print(f.read(4))  # b'PK\x03\x04' -> ZIP archive
```

Checking the zip header flags shows bit 0 is set (`flag_bits = 9`), indicating password protection.

We crack the archive against `outer_wordlist.txt` using Python's standard `zipfile` module:
```python
import zipfile

zf = zipfile.ZipFile('IRCTC_Tatkal_Hack.pdf')
with open('outer_wordlist.txt', 'r', encoding='latin-1') as f:
    for line in f:
        pwd = line.strip().encode('utf-8')
        try:
            with zf.open('README.txt', pwd=pwd) as test_f:
                print('Found password:', line.strip())
                break
        except Exception:
            pass
```
- **Password found:** `railwaystationroyaljourney`
- **Extracted files:**
  - `delhi.jpg`
  - `delhi_wordlist.txt`
  - `README.txt`

---

### Step 1: Delhi (`delhi.jpg`)

`delhi.jpg` is a JPEG image, but inspecting its structure reveals an embedded ZIP archive starting at offset **`89038`** (`0x15BCE`):
```python
with open('delhi.jpg', 'rb') as f:
    data = f.read()
    print('PK\\x03\\x04 offset:', data.find(b'PK\x03\x04'))  # 89038
```

Extracting the byte slice from `89038` to EOF yields a valid ZIP archive containing:
- `delhi.txt`
- `mumbai.jpg`
- `mumbai_wordlist.txt`

Testing words from `delhi_wordlist.txt`:
- **Password found:** `cholebhature`
- **`delhi.txt` message:**
  > *DELHI - JOURNEY BEGINS*  
  > *Your journey has only just begun. The photograph contains more than meets the eye. Investigate its contents to find the next stop.*

---

### Step 2: Mumbai (`mumbai.jpg`)

Inspecting `mumbai.jpg` reveals another ZIP archive embedded within the file starting at offset **`1958742`** (`0x1DE356`):
```python
with open('mumbai.jpg', 'rb') as f:
    f.seek(1958742)
    zip_bytes = f.read()
```

The embedded ZIP archive contains:
- `mumbai.txt`
- `chennai.jpg`
- `chennai_wordlist.txt`

Testing words from `mumbai_wordlist.txt`:
- **Password found:** `vada_pav`
- **`mumbai.txt` message:**
  > *MUMBAI - TRANSIT POINT*  
  > *The journey continues to Chennai. Do not trust an image just because it opens normally. Look for data beyond its visible contents and identify the archive hidden inside.*

---

### Step 3: Chennai (`chennai.jpg`)

Inspecting `chennai.jpg` shows a ZIP archive starting at byte offset **`167393`** (`0x28DE1`).

The embedded ZIP archive contains:
- `chennai.txt`
- `kolkata.jpg`

Testing words from `chennai_wordlist.txt`:
- **Password found:** `mylapore`
- **`chennai.txt` message:**
  > *CHENNAI - THE NEXT STOP*  
  > *The journey continues towards Kolkata. This photograph seems ordinary, but files can contain more than their visible contents suggest.*

---

### Step 4: Kolkata (`kolkata.jpg`)

Analyzing `kolkata.jpg`:
- The JPEG End of Image (EOI) marker `\xFF\xD9` occurs at byte offset **`157805`**.
- The file has a total size of `178287` bytes, leaving **20,480 bytes** appended after EOF.
- Inspecting the raw appended bytes shows 512-byte block alignments with filenames padded by null bytes—a standard **POSIX TAR archive**.

Extracting the TAR archive:
```python
import tarfile, io

with open('kolkata.jpg', 'rb') as f:
    f.seek(157807)
    with tarfile.open(fileobj=io.BytesIO(f.read())) as tar:
        tar.extractall()
```
- **Extracted files:**
  - `kolkata.txt`
  - `final_list.txt`
  - `golden_diamond_ticket_locked.pdf`
- **`kolkata.txt` message:**
  > *KOLKATA - FINAL STOP*  
  > *The Golden Diamond Tour ends here. This archive uses an old-fashioned format. Identify the data appended to this image and extract it. The ticket has a separate password. The accompanying wordlist may help you unlock it.*

---

### Step 5: Unlocking the Ticket (`golden_diamond_ticket_locked.pdf`)

`golden_diamond_ticket_locked.pdf` is an authentic PDF document. Checking its encryption dictionary (`/Encrypt 16 0 R`):
```text
16 0 obj
<< 
   /CF << /StdCF << /AuthEvent /DocOpen /CFM /AESV3 /Length 32 >> >> 
   /Filter /Standard 
   /Length 256 
   /R 6 
   /V 5 
   /U <f5eb3240d4747d7068bb1c9b02884c34a0787a0e247afb29d0aa1ae4a302eef850227c358066b4370f4ec82cdf5fdc78> 
   /UE <ea4b969c81d3874cf7e8cb7e1cf1298479b5f206429db2c9dba0a4628659ce71> 
   ...
>>
```

This specifies **Standard Security Handler Revision 6 (R=6, V=5, AES-256)** defined in **ISO 32000-2 (PDF 2.0)**.

#### ISO 32000-2 Algorithm 2.B Password Derivation
Under Revision 6:
1. User password verification involves an iterative multi-hash algorithm cycling across **SHA-256**, **SHA-384**, and **SHA-512** alongside **AES-128-CBC** encryption rounds until at least 64 rounds have completed and the termination condition is met.
2. The user validation salt is stored in bytes 32–39 of `/U`.
3. The user key salt is stored in bytes 40–47 of `/U`.

We implement Algorithm 2.B in Python and test all 20 words in `final_list.txt`:
- **Password found:** `peacock`

#### Decrypting the File Encryption Key
1. The intermediate key is derived via `hash2b(b"peacock", key_salt, b"")`.
2. The 32-byte `/UE` field is decrypted using AES-256-CBC with IV = `\x00` * 16.
3. This yields the master file encryption key:  
   `866eb17cc0c36edbe99114cccfcaf7ca53dc4c4b1df1e0bc8bf232d7f69a86bf`
4. The key is verified against the `/Perms` entry (bytes 9–11 decrypt to ASCII `'adb'`).

#### Decrypting the Ticket Stream
In PDF AESV3 (`/CFM /AESV3`), stream data is decrypted using AES-256-CBC where the first 16 bytes of the stream serve as the IV. Decompressing the decrypted payload with `zlib.decompress` reveals the original ticket text:

```text
============================================================
                    INDIAN RAILWAYS
                  GOLDEN DIAMOND TOUR
============================================================
                    E - TICKET
Passenger        : pir4cy
------------------------------------------------------------
ITINERARY
------------------------------------------------------------
        NEW DELHI
             |
             v
          MUMBAI
             |
             v
          CHENNAI
             |
             v
          KOLKATA
------------------------------------------------------------
Train            : 12302
Journey          : Golden Diamond Tour
Status           : CONFIRMED
Coach            : B2
Seat             : 37
------------------------------------------------------------
Congratulations! You won the giveaway!
Your flag is:
InIt{j0urney_4cr0ss_1ndia}
------------------------------------------------------------
                    HAPPY JOURNEY
============================================================
```


## 3. Summary Table

| Stage | Container / File | Technique | Password / Mechanism | Extracted Clue |
|---|---|---|---|---|
| **0** | `IRCTC_Tatkal_Hack.pdf` | Disguised ZIP archive | `railwaystationroyaljourney` | `delhi.jpg`, `delhi_wordlist.txt` |
| **1** | `delhi.jpg` | Embedded ZIP (offset 89038) | `cholebhature` | `mumbai.jpg`, `mumbai_wordlist.txt` |
| **2** | `mumbai.jpg` | Embedded ZIP (offset 1958742) | `vada_pav` | `chennai.jpg`, `chennai_wordlist.txt` |
| **3** | `chennai.jpg` | Embedded ZIP (offset 167393) | `mylapore` | `kolkata.jpg`, `chennai.txt` |
| **4** | `kolkata.jpg` | Appended TAR (after `\xFF\xD9`) | POSIX TAR parsing | `golden_diamond_ticket_locked.pdf`, `final_list.txt` |
| **5** | `golden_diamond_ticket_locked.pdf` | PDF 2.0 AES-256 (Revision 6) | `peacock` (Algorithm 2.B) | **`InIt{j0urney_4cr0ss_1ndia}`** |
