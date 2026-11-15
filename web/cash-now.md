# 💸 Cash Now

| | |
|---|---|
| **Category** | Web |
| **Points** | 200 |
| **Flag** | `InIt{ssrf_pNgcrush_3numt00l0ng}` |

## 📝 Challenge Analysis

A multi-stage web challenge with source code provided (`handout.zip`). The flag is stored at `/data/internal/verification_status.txt` on the server.

## 🛠️ Exploit Chain

### Stage 1: SSRF via HTTP 307 redirect (`ssrf`)
The `/profile/avatar` endpoint fetches images from user-supplied URLs. It validates IPs against private ranges and explicitly blocks **301** and **302** redirects, but **307 was left out**.

Point the avatar fetcher at a public server that returns a `307` redirect to:

```text
http://127.0.0.1:39417/archive/config
```

The server follows the redirect internally, and the JSON error response leaks the private upload path: `/data/uploads/enc-.../`.

### Stage 2: ImageMagick arbitrary file read (`pNgcrush`, CVE-2022-44268)
The backend uses **ImageMagick 7.1.0-48** to generate thumbnails. Craft a PNG with a `tEXt` chunk whose keyword is `profile` and whose value is:

```text
/data/internal/verification_status.txt
```

ImageMagick embeds the **hex-encoded contents** of that file into the generated thumbnail.

### Stage 3: Temp cleanup bypass (`3numt00l0ng`)
Thumbnails are normally scrubbed after processing. Supplying an excessively long filename (**250+ characters**) triggers an `ENAMETOOLONG` error that halts the final move/cleanup routine, leaving the thumbnail in `tmp/<username>.png`.

### Retrieving the flag
Request:

```text
/documents/preview?path=/data/uploads/enc-.../tmp/<username>.png
```

Extract the `Raw profile type` chunk from the PNG and hex-decode it:

```text
flag: InIt{ssrf_pNgcrush_3numt00l0ng}
```

## 🏁 Flag

```text
InIt{ssrf_pNgcrush_3numt00l0ng}
```

## 🧰 Tools
Python (Flask redirect server) · pngcrush-style PNG crafting · exiftool / pngcheck · CyberChef
