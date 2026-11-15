# CTF Writeup: 2026 India Trip (Digital Forensics)

## Challenge Summary

- **Category:** Forensics / Steganography
- **Description:** A digital camera was found abandoned with photos from a trip. We are tasked with investigating the images to recover what was hidden.
- **Archive Password:** `camera-found`
- **Flag:** `InIt{h1d1ng_1n_m3tad4t4}`

---

## Walkthrough

1. **Extracting the Archive:**
   I unpacked `2026-india-trip.7z` using the provided password `camera-found`, which yielded a folder containing multiple trip photos (`IMG_01.jpg` through `IMG_50.jpg`).

2. **Manually Checking the Metadata:**
   While inspecting the photos, I manually checked the EXIF metadata across the image files. Most pictures had standard, innocuous captions in their `ImageDescription` tags (e.g., *"The light was perfect that day"*, *"A moment from the journey"*).

3. **Locating the Hidden Clue:**
   Upon checking **`IMG_38.jpg`**, I noticed an unusual value stored in its description field:
   ```text
   https://ctf.pir4cy.com/exif-flag
   ```

4. **Retrieving and Decoding the Flag:**
   I opened the URL, which presented the challenge page with a Base64-encoded string:
   ```text
   SW5JdHtoMWQxbmdfMW5fbTN0YWQ0dDR9Cg==
   ```
   *(Note: The page also contained a hidden `<div>` tag with a fake decoy flag designed to confuse automated bots).*

   Decoding the Base64 string revealed the final flag:
   ```bash
   echo "SW5JdHtoMWQxbmdfMW5fbTN0YWQ0dDR9Cg==" | base64 -d
   ```

   # **Flag:**
   ```text
   InIt{h1d1ng_1n_m3tad4t4}
   ```
