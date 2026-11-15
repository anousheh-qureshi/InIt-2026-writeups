# The Fractured Key 

**Category:** Crypto 

---

## Challenge Description 

We were provided with a downloadable archive containing two files:

- `key_gen.py`
- `intercepted.txt`

The challenge prompt stated:

> We intercepted a secure communication between two operatives. We managed to get the cipher text and a strange script they were using to generate their keys. It looks like they tried to be clever by hiding the true flag among fakes. The true flag follows the standard InIt{...} format, but don't trust everything you decrypt.

## Inspecting the Files

First, I opened `key_gen.py` to understand how the encryption and key generation worked.

### Key Generation Code

```python
import os
import random
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad

# We need a 32-byte key for AES-256
def generate_master_key():
    # To make it extra secure, we base it on a random seed
    seed = random.randint(0, 255)
    random.seed(seed)

    key = bytearray()
    for _ in range(32):
        key.append(random.randint(0, 255))

    return bytes(key)

def encrypt(data: bytes) -> bytes:
    key = generate_master_key()
    iv = os.urandom(16)
    cipher = AES.new(key, AES.MODE_CBC, iv=iv)
    return iv + cipher.encrypt(pad(data, AES.block_size))
```

The main vulnerability was immediately visible.

1. **Tiny Seed Space:** Although AES-256 uses a 256-bit key the key is generated using Python's standard `random` module, seeded with a value between 0 and 255.
2. **Only 256 Possible Seeds:** This means there are only 256 possible initial seeds, resulting in a very small key search space.
3. **Ciphertext Structure:** The encrypted data consists of a 16-byte IV followed by AES-256-CBC ciphertext using PKCS#7 padding.

Next, I inspected `intercepted.txt`.

It contained three lines of hexadecimal ciphertext.

Each line follows this structure:

- First 32 hexadecimal characters = 16-byte IV.
- Remaining characters = AES-CBC ciphertext.

The challenge prompt also warned us not to trust everything we decrypted, suggesting that some of the recovered flags might be decoys.

## Exploitation and Solution Script

Since there were only 256 possible seeds, brute-forcing them was practical.

Then i wrote a Python script to:

1. Read all three ciphertexts.
2. Generate a key for every seed from 0 to 255.
3. Decrypt each ciphertext using AES-CBC.
4. Remove the PKCS#7 padding.
5. Check whether the decrypted result was valid UTF-8.

### Python Solver

```python
import random
from Crypto.Cipher import AES
from Crypto.Util.Padding import unpad

# Load intercepted ciphertexts
with open("intercepted.txt", "r") as f:
    lines = [line.strip() for line in f if line.strip()]

def get_key_from_seed(seed: int) -> bytes:
    random.seed(seed)

    key = bytearray()

    for _ in range(32):
        key.append(random.randint(0, 255))

    return bytes(key)

for idx, line in enumerate(lines, 1):
    raw = bytes.fromhex(line)
    iv = raw[:16]
    ct = raw[16:]

    print(f"[*] Testing Line {idx}...")

    for seed in range(256):
        key = get_key_from_seed(seed)

        cipher = AES.new(key, AES.MODE_CBC, iv=iv)
        pt = cipher.decrypt(ct)

        try:
            plaintext = unpad(pt, AES.block_size).decode('utf-8')
            print(f"  [+] Match found with Seed {seed}: {plaintext}")

        except Exception:
            # Padding error or invalid UTF-8
            continue
```

## Results and Decoy Filtering

Running the script produced valid decryptions across all three lines.

**Line 1 — Seed 42**

```text
InIt{w0w_y0u_f0und_1t_but_1ts_f4k3}
```

This was a decoy flag, as it explicitly says that it is fake.

**Line 2 — Seed 201**

```text
InIt{ps3ud0_r4nd0m_1s_n0t_r4nd0m_k3y_sp4c3_t00_sm4ll}
```

This flag refers directly to the vulnerability in the challenge: using a pseudorandom generator with a very small seed space.

**Line 3 — Seed 137**

```text
InIt{h4h4_n1c3_try_k33p_l00k1ng}
```

This was another decoy flag, intended to distract from the actual solution.

After comparing the three decrypted messages, Line 2 was identified as the correct flag.

## Final Flag

```text
InIt{ps3ud0_r4nd0m_1s_n0t_r4nd0m_k3y_sp4c3_t00_sm4ll}
```

## Key Takeaway

AES-256 can use a 256-bit key, but that does not automatically make an encryption system secure.

In this challenge, the problem was the way the key was generated. Using Python's `random` module with only 256 possible initial seeds made brute-forcing the key practical.

The lesson is that a strong encryption algorithm cannot compensate for weak key generation. Cryptographic keys should be generated using a cryptographically secure random number generator.
