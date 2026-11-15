# The Troll's Ticket 

- **Category:** Misc 

---

## Challenge Description

> Congratulations on winning the Indian Railways Golden Diamond Tour!  
> Unfortunately, someone has tampered with your ticket. Your journey passes through Delhi, Mumbai, Chennai, and Kolkata, but the files may not be what they appear to be.  
> Follow the trail, recover your ticket, and claim your prize.

##Overview and Strategy

This challenge was a series of nested archives hidden inside image files.

The challenge description gave us the route:

**Delhi → Mumbai → Chennai → Kolkata → Final Ticket**

At each stop, a city image contained an embedded archive appended to its bytes. Each stage also provided a customized wordlist containing the password needed to extract the next archive.

The final stage contained a TAR archive, followed by an encrypted PDF ticket.

## Walkthrough

### Stop 1: Delhi — The Disguised File

We started with two files:

- `IRCTC_Tatkal_Hack.pdf`
- `outer_wordlist.txt`

Running the `file` command on `IRCTC_Tatkal_Hack.pdf` revealed that it was not actually a PDF. It was a password-protected ZIP archive.

Testing the words in `outer_wordlist.txt` revealed the password:

- **Password:** `railwaystationroyaljourney`

Unzipping the archive gave us:

- `delhi.jpg`
- `delhi_wordlist.txt`

### Stop 2: Mumbai

The file `delhi.jpg` opened as a normal image. However, inspecting its raw bytes revealed a ZIP signature (`PK\x03\x04`) at offset `89038`.

This indicated that another ZIP archive had been appended to the image.

Using `delhi_wordlist.txt`, we found the password:

- **Password:** `cholebhature`

The extracted files were:

- `mumbai.jpg`
- `mumbai_wordlist.txt`
- `delhi.txt`

### Stop 3: Chennai

Following the same method, we inspected `mumbai.jpg` and found another embedded ZIP archive.

Testing the words in `mumbai_wordlist.txt` revealed the password:

- **Password:** `vada_pav`

The extracted files were:

- `chennai.jpg`
- `chennai_wordlist.txt`
- `mumbai.txt`

### Stop 4: Kolkata

The file `chennai.jpg` also contained an embedded ZIP archive, located at offset `167393`.

Using `chennai_wordlist.txt`, we discovered the password:

- **Password:** `mylapore`

The extracted files were:

- `kolkata.jpg`
- `chennai.txt`

### Final Stop: Extracting the Golden Diamond Ticket

When inspecting `kolkata.jpg`, we could not find the usual ZIP signature.

Instead, examining the bytes immediately after the JPEG End-Of-Image marker (`\xff\xd9`) revealed an embedded **POSIX TAR archive**.

Extracting the TAR archive gave us:

1. `golden_diamond_ticket_locked.pdf`
2. `final_list.txt`
3. `kolkata.txt`

### 4. Decrypting the Ticket and Recovering the Flag

The file `golden_diamond_ticket_locked.pdf` was password protected.

We tested the words in `final_list.txt` and found the correct PDF password:

- **PDF Password:** `peacock`

After decrypting the PDF, we extracted the ticket contents.

The ticket displayed the following information:

<img width="636" height="829" alt="Screenshot 2026-11-15 at 21 06 09" src="https://github.com/user-attachments/assets/d1e0c356-ca47-431b-9f94-55eb9aa5ef33" />

## 5. Final Flag

`InIt{j0urney_4cr0ss_1ndia}`

## 6. Key Takeaways

- A file extension does not guarantee the actual file format. Always inspect suspicious files using tools such as `file` and a hex editor.
- ZIP archives can be appended to image files without preventing the images from opening normally.
- JPEG files can contain additional data after their End-Of-Image marker.
- Different archive formats can be hidden inside one another.
- Wordlists supplied with challenge files often contain passwords for the next extraction stage.
- File carving and metadata inspection are useful techniques in digital forensics.
