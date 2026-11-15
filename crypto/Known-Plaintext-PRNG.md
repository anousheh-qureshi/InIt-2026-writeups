# Known Plaintext PRNG 

### Description

A custom stream-like cipher encrypts a flag in four-byte blocks. Each block is XORed with a 32-bit key, and the key is mutated using a predictable Linear Congruential Generator (LCG).

The starting key was supposed to be secret, but the flag format reveals the first plaintext block.

Files: `encrypt.py`, `flag.enc`

* **Category:** Cryptography
* **Tools Used:** Python 3

### Walkthrough

After opening `encrypt.py`, I found that the encryption process works as follows:

1. An initial random 32-bit key is generated using `os.urandom(4)`.
2. The flag is divided into 4-byte blocks.
3. Each block is XORed with the current 32-bit key.
4. After each block, the key is updated using an LCG:

`key = (key * 1103515245 + 12345) mod 2^32`

XOR is reversible:

`C = P XOR K`

Therefore:

`K = C XOR P`

Since every flag starts with `InIt`, we already know the first four plaintext bytes. This allows us to recover the initial key:

```python
key0 = int.from_bytes(flag_enc[:4], "big") ^ int.from_bytes(b"InIt", "big")
```

Once the initial key is recovered, the LCG is deterministic. We can generate the remaining keys and decrypt every block.

The complete solver:

```python
def next_key(key: int) -> int:
    return (key * 1103515245 + 12345) & 0xFFFFFFFF

with open("flag.enc", "rb") as f:
    enc = f.read()

# Recover key0 from known prefix InIt
first_cipher_int = int.from_bytes(enc[:4], "big")
key = first_cipher_int ^ int.from_bytes(b"InIt", "big")

# Deecrypt blocks
flag = bytearray()

for offset in range(0, len(enc), 4):
    chunk = enc[offset:offset + 4]
    actual_len = len(chunk)
    padded = chunk + b"\x00" * (4 - actual_len)

    plain_int = int.from_bytes(padded, "big") ^ key
    flag.extend(plain_int.to_bytes(4, "big")[:actual_len])

    key = next_key(key)

print(flag.decode())
```

Running the solver produced the decrypted flag.

### Flag

`InIt{kn0wn_pl41nt3xt_4tt4cks_4r3_c00l}`

---
