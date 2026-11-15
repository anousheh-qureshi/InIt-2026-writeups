# ec2-dropper

| | |
|---|---|
| **Category** | Forensics |
| **Points** | 200 |
| **Author** | Kshitij Raj |
| **Artifact** | `incident_ec2.zip` |
| **Flag** | `InIt{imds_k3y3d_s3cr3t_v4ult}` |

## Challenge

> An EC2 instance in our web tier was found running an unauthorised scheduled script, `/opt/agent/vault.py`, installed right after an SSH login from an address nobody recognised. The script keeps its configuration in an encrypted vault, and the key isn't stored on disk. It's derived at runtime from the instance's own identity.
>
> We pulled the script, the vault, and the host logs. Review the code, work out how the vault is keyed, and recover the operator's configuration note.

## TL;DR

The vault key is `SHA256("s4lt_" + instance_id)`. The instance ID is printed in the cloud-init log, so the key can be rebuilt offline and the AES-256-CBC vault decrypted.

## Solution

### 1. Evidence

```text
incident_ec2/
  INCIDENT.txt
  agent/vault.py
  agent/vault.bin
  artifacts/syslog.txt
  artifacts/crontab.txt
  artifacts/cloud-init-output.log
```

### 2. Review the code

```python
SALT = "s4lt_"

def derive_key(instance_id):          # 256-bit key
    return hashlib.sha256((SALT + instance_id).encode()).digest()

def open_vault(path):
    blob = Path(path).read_bytes()
    iv, ct = blob[:16], blob[16:]     # IV is prepended
    plain = AES.new(key, AES.MODE_CBC, iv).decrypt(ct)
    plain = plain[:-plain[-1]]        # PKCS#7 unpad
    return json.loads(plain)
```

- Cipher: **AES-256-CBC**, PKCS#7 padding
- IV: first 16 bytes of `vault.bin`
- Key: `SHA256("s4lt_" + instance_id)`

On the live host the instance ID came from the metadata service (`169.254.169.254`). That is not reachable offline, so it has to be found elsewhere in the evidence.

### 3. Recover the instance ID from the logs

`cloud-init-output.log` prints it:

```text
instance-id: i-0a1b2c3d4e5f67890
```

The instance ID is not a secret, so anyone holding the logs can rebuild the key.

### 4. Decrypt

The Python crypto library was not installed, so I used the OpenSSL CLI with the derived key and the IV from the vault header:

```bash
KEY=$(printf 's4lt_i-0a1b2c3d4e5f67890' | openssl dgst -sha256 -binary | xxd -p -c 64)
IV=$(head -c 16 vault.bin | xxd -p)
tail -c +17 vault.bin | openssl enc -d -aes-256-cbc -K "$KEY" -iv "$IV"
```

Output (JSON):

```json
{"config_version": 2, "collected_region": "ap-south-1",
 "operator_note": "InIt{imds_k3y3d_s3cr3t_v4ult}"}
```

## Flag

```
InIt{imds_k3y3d_s3cr3t_v4ult}
```

## Takeaways

Deriving a key from an instance ID (readable from IMDS, logs and the console) is obfuscation, not secrecy. Real secrets belong in a secrets manager or KMS, not keyed from public identifiers.
