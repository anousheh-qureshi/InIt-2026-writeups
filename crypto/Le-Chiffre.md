# Le Chiffre

### Description

> A message was left on an old forum, along with a note:
>
> "The key is the hour when everything goes quiet."

`UvLg{d1m3u3k3_i4a_q0g_bn3_4uli3z}`
* **Category:** Cryptography
* **Tools Used:** Python 3 

### Walkthrough

"Le Chiffre" refers to the historical name associated with the Vigenère cipher.

We know the plaintext flag starts with `InIt`, while the ciphertext starts with `UvLg`.

Using the known plaintext, we can calculate the first four characters of the key:

- U - I = 12 → m
- v - n = 8 → i
- L - I = 3 → d
- g - t = 13 → n

The key therefore starts with `midn`.

The challenge hint says: "The key is the hour when everything goes quiet." This suggests the full key is `midnight`.

then used the following Python code to decrypt the ciphertext:

```python
ct = "UvLg{d1m3u3k3_i4a_q0g_bn3_4uli3z}"
key = "midnight"

pt = []
ki = 0

for c in ct:
    if c.isalpha():
        base = ord('A') if c.isupper() else ord('a')
        k = ord(key[ki % len(key)].lower()) - ord('a')
        pt.append(chr((ord(c) - base - k) % 26 + base))
        ki += 1
    else:
        pt.append(c)

print("".join(pt))
```

### Flag

`InIt{v1g3n3r3_w4s_n0t_th3_4nsw3r}`

---
