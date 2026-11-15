# The Careless Twins

**Category:** Cryptography  

---

## Challenge Description

> We captured communications between two different servers belonging to the same syndicate. They were bragging about their new "military-grade" 2048-bit RSA encryption. We managed to grab both of their public keys and an encrypted message sent to one of them. We suspect their custom prime generator isn't as random as they think.

We were given a ZIP file containing two files:

- `encryption.py`
- `output.txt`

## Source Code Analysis

After opening `encryption.py`, I found the following code:

```python
from Crypto.Util.number import getPrime, bytes_to_long
import random

# Server A's Key Generation
p = getPrime(1024)
q1 = getPrime(1024)
n1 = p * q1
e = 65537

# Server B's Key Generation
q2 = getPrime(1024)
n2 = p * q2

m = bytes_to_long(flag)
c1 = pow(m, e, n1)

with open("output.txt", "w") as f:
    f.write(f"n1 = {n1}\n")
    f.write(f"n2 = {n2}\n")
    f.write(f"e = {e}\n")
    f.write(f"c1 = {c1}\n")
```

The issue isn't that `getPrime()` generates weak or predictable primes. The problem is how the generated primes are being used.

Looking closely, `p` is generated only once:

`p = getPrime(1024)`

Both servers then reuse this exact same prime when generating their RSA moduli:

- Server A: `n1 = p * q1`
- Server B: `n2 = p * q2`

Normally, each RSA key should have its own independently generated pair of prime factors.

However, these two moduli share the same prime factor `p`. This is the actual vulnerability.

## Exploitation Strategy

Normally, factoring a properly generated 2048-bit RSA modulus is computationally infeasible.

However, when two different RSA moduli share a prime factor, we can recover that factor using the **Greatest Common Divisor (GCD)** algorithm.

Since:

`n1 = p * q1`

`n2 = p * q2`

We can calculate:

`gcd(n1, n2) = p`

Once we recover `p`, we can calculate the remaining values needed to decrypt the ciphertext.

The steps are:

1. Recover the shared prime: `p = gcd(n1, n2)`.
2. Calculate the other prime: `q1 = n1 // p`.
3. Calculate Euler's totient:

   `phi(n1) = (p - 1) * (q1 - 1)`

4. Calculate the private exponent:

   `d1 = e^(-1) mod phi(n1)`

5. Decrypt the ciphertext:

   `m = pow(c1, d1, n1)`

## Solve Script

```python
import math

# Load values from output.txt
data = {}

with open("output.txt") as f:
    for line in f:
        if "=" in line:
            k, v = line.split("=")
            data[k.strip()] = int(v.strip())

n1 = data["n1"]
n2 = data["n2"]
e = data["e"]
c1 = data["c1"]

# 1. Recover the shared prime factor
p = math.gcd(n1, n2)
assert p > 1 and p < n1

# 2. Factor n1 and compute phi
q1 = n1 // p
phi1 = (p - 1) * (q1 - 1)

# 3. Derive private key d and decrypt ciphertext
d1 = pow(e, -1, phi1)
m = pow(c1, d1, n1)

# 4. Convert plaintext integer back to bytes
flag = m.to_bytes((m.bit_length() + 7) // 8, "big")

print("[+] Decrypted Flag:", flag.decode())
```

## Output

```text
[+] Decrypted Flag: InIt{n3v3r_r3us3_pr1m3s_1n_rs4}
```

## Flag

`InIt{n3v3r_r3us3_pr1m3s_1n_rs4}`

## Key Takeaway

RSA relies on the difficulty of factoring large numbers into their prime factors.

However, using the same prime factor in multiple RSA moduli completely compromises their security.

