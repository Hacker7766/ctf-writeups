# Challenge Name: Tesla

## Description

**Category:** Forensics

> Flipper Zero, often referred to as a hacking device, is designed to capture frequencies and execute commands. It's considered a risky tool to have, as it is illegal in some countries.

**Flag format:** `0xfun{...}`

---

## Writeup

### Step 1: Inspecting the capture

First, I opened the provided `Tesla.sub` file and immediately noticed something weird:

- The header looks like a Sub-GHz capture (frequency/preset/protocol),
- but the file says **`Filetype: Bad Usb 0xfun`** and the `RAW_Data` field contains **tons of 8-bit binary chunks** (`01010101` style).

That’s a strong hint the “signal” is actually **hidden data**.

---

### Step 2: Converting RAW_Data bits into bytes

I extracted every `XXXXXXXX` (8-bit) chunk from the file, converted each one to a byte, and concatenated them.

Once I did that, the byte stream started with `0xFF 0xFE` and quickly turned into readable Windows batch content like:

- `cls`
- `set "Il...=...."`
- lots of `%VAR:~x,1%` expansions (classic batch obfuscation)

So the `.sub` file is basically a “carrier” for an **obfuscated batch payload**.

---

### Step 3: De-obfuscating the batch script

The script stores a long random string into a variable (in my case: `IlÃc`) and then prints commands using substring slicing:

- `%IlÃc:~29,1%` → character at index 29
- `%IlÃc:~1,1%` → character at index 1  
…etc.

I emulated those substitutions and removed junk `%UNKNOWNVAR%` expansions (they evaluate to empty in batch if undefined). The cleaned script became:

```bat
@echo off
powershell -NoProfile -Command "[Convert]::ToBase64String([System.Text.Encoding]::UTF8.GetBytes('i could be something to this'))"
:: 5958051a1b170013520746265a0e51435b36165752470b7f03591d1b364b501608616e ::
:: ive been encrypted many in ways::
pause
```

Now I have:
- a **key phrase**: `i could be something to this`
- a **ciphertext** hex string

---

### Step 4: XOR decrypting the ciphertext

The hint “encrypted many in ways” plus the simplicity strongly suggests XOR.

I treated the phrase as the XOR key (repeated) and XOR’d it against the hex-decoded ciphertext.

```python
import re

# 1) read Tesla.sub
text = open("Tesla.sub","rb").read().decode(errors="ignore")

# 2) extract 8-bit chunks
bits = re.findall(r"\b[01]{8}\b", text)
blob = bytes(int(b,2) for b in bits)

# 3) remove BOM-like bytes if present and keep it as latin1 so nothing breaks
s = blob.decode("latin1")
if s.startswith("ÿþ"):
    s = s[2:]

# 4) pull ciphertext + key (already visible after deobfuscation, but hardcode for clarity)
ct = bytes.fromhex("5958051a1b170013520746265a0e51435b36165752470b7f03591d1b364b501608616e")
key = b"i could be something to this"

pt = bytes(ct[i] ^ key[i % len(key)] for i in range(len(ct)))
print(pt.decode())
```

This prints the flag.

---

## Flag

```
0xfun{d30bfU5c473_x0r3d_w1th_k3y}
```
