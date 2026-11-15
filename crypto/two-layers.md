# Two Layers (Crypto - 100 pts)

### Description

> We intercepted this string from a careless admin. He swore he'd "double-encoded it for safety." Prove him wrong.

`VmFWZ3tlMGdfZ3UzYV9vNjRfMWZfYTBnX2YzcGhlMWdsfQ==`



### Walkthrough

The challenge title says "Two Layers", and the admin claims to have double-encoded the string.

The string ends with `==`, which suggests Base64 encoding. I used Python to decode it:

```python
import base64
s = 'VmFWZ3tlMGdfZ3UzYV9vNjRfMWZfYTBnX2YzcGhlMWdsfQ=='
print(base64.b64decode(s).decode())
```

Output:

```text
VaVg{e0g_gu3a_o64_1f_a0g_f3phe1gl}
```

The result is still scrambled. However, notice the flag wrapper: `VaVg{...}`. The actual flag format is `InIt{...}`.

Comparing the two reveals a shift of 13 positions, which indicates ROT13.

Decoding the entire string using ROT13 gives:

- `VaVg` → `InIt`
- `e0g_gu3a_o64_1f_a0g_f3phe1gl` → `r0t_th3n_b64_1s_n0t_s3cur1ty`

### Flag

`InIt{r0t_th3n_b64_1s_n0t_s3cur1ty}`

---

