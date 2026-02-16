# Challenge Name: Hawk_II

## Description

**Category:** Crypto

> A detective found some hawks feathers, could he determines which hawk of the pack is the feathers?

**Flag format:** `0xfun{...}`

---

## Writeup

### Step 1: Unzipping and inspecting files

After extracting `Hawk_II.zip`, I got:

- `Hawk_II.sage` / `hawk.sage` (Sage scripts)
- `output.txt`

The important part was `output.txt`, because it contained:

- `iv = "..."`
- `enc = "..."` (AES-CBC ciphertext)
- `pk = (...)` (public key polynomial)
- `sk = (...)` (secret key polynomial!)

So the “mystery feathers” are basically solved for me: the secret key is leaked.

---

### Step 2: How the AES key is derived

Reading the Sage code showed the AES key was derived like this:

```python
key = sha256(str(sk).encode()).digest()
```

So if I can reproduce the exact `str(sk)` string used during encryption, I can rebuild the AES key.

Luckily, `output.txt` prints `sk = (...)` in the exact same format I need.

---

### Step 3: Decrypting AES-CBC

I extracted:

- the IV hex string
- the ciphertext hex string
- the exact `sk` polynomial string (including parentheses)

Then I computed:

- `key = sha256(sk_string.encode()).digest()`
- `AES-CBC decrypt + PKCS7 unpad`

```python
import re
from hashlib import sha256
from Crypto.Cipher import AES
from Crypto.Util.Padding import unpad

data = open("output.txt", "r", encoding="utf-8", errors="ignore").read()

iv_hex  = re.search(r'iv\s*=\s*"([0-9a-f]+)"', data).group(1)
enc_hex = re.search(r'enc\s*="([0-9a-f]+)"', data).group(1)
sk_str  = re.search(r"sk\s*=\s*(.*)\r?\nleak_data", data, re.S).group(1).strip()

key = sha256(sk_str.encode()).digest()
iv  = bytes.fromhex(iv_hex)
ct  = bytes.fromhex(enc_hex)

pt = unpad(AES.new(key, AES.MODE_CBC, iv).decrypt(ct), 16)
print(pt.decode())
```

This printed the flag.

---

## Flag

```
0xfun{tOO_LLL_256_B_kkkkKZ_t4e_f14g_F14g}
```
