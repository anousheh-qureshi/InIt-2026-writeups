# 🎁 hidden giveaway

| | |
|---|---|
| **Category** | Misc |
| **Points** | 100 |
| **Flag** | `InIt{h1d1ng_1n_b1ue_l1gh7}` |

## 📝 Challenge Analysis

We get an infographic map (`india_tourist_map.png`) of Indian tourist destinations with the text *"Scan the QR to enter the giveaway!"*. Looking at the image normally, there is no QR code.

## 🛠️ Solution

### 1. Bit plane steganography
- Upload `india_tourist_map.png` to **StegOnline** or **Aperi'Solve**, or open it in **StegSolve**.
- Go through the individual color bit planes. In **Blue Plane 0** (the least significant bit of the blue channel), a crisp, full-sized QR code appears.

### 2. Following the URL
Scanning the QR leads to `https://ctf.pir4cy.com/giveaway`, which shows a congratulations message with a Base64 string:

```text
SW5JdHt7aDFkMW5nXzFuX2IxdWVfbDFnaDd9Cg==
```

> ⚠️ The page HTML also has a hidden `<div style="display:none">` honeypot with a fake flag to catch scrapers.

### 3. Decoding the flag

```bash
echo 'SW5JdHt7aDFkMW5nXzFuX2IxdWVfbDFnaDd9Cg==' | base64 -d
```

```text
InIt{h1d1ng_1n_b1ue_l1gh7}
```

The phrase "hiding in blue light" hints at **Blue Plane 0**.

## 🏁 Flag

```text
InIt{h1d1ng_1n_b1ue_l1gh7}
```

## 🧰 Tools
StegOnline · Aperi'Solve · StegSolve · CyberChef
