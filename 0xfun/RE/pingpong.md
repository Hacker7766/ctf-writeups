# Challenge Name: pingpong

## Description

**Category:** Reverse Engineering

> ONLY attempt if you love table tennis... you have been warned.

**Flag format:** `0xfun{...}`

---

## Writeup

### Step 1: Quick recon on the binary

First, I ran `strings` on the `pingpong` binary to see if it leaked anything useful. Buried inside a long Rust panic/error string, I found:

- A long hex string (ciphertext):
```
0149545b5f4b5d1e5c545d1a55036c5700404b46505d426e02001b4909030957414a7b7a48
```

- A weird dotted number “key”:
```
112.105.110.103112.111.110.103
```

---

### Step 2: Understanding the key (the “table tennis” hint)

Those numbers match ASCII codes:

- `112.105.110.103` → `p i n g` → `"ping"`
- `112.111.110.103` → `p o n g` → `"pong"`

So the key string is literally `"pingpong"` expressed as ASCII decimals and concatenated.

The binary is basically screaming: “use this as the key”.

---

### Step 3: XOR decrypt

I hex-decoded the ciphertext, then XOR’d it with the key (repeating).

```python
import binascii

ct = binascii.unhexlify(
    "0149545b5f4b5d1e5c545d1a55036c5700404b46505d426e02001b4909030957414a7b7a48"
)
key = b"112.105.110.103112.111.110.103"

pt = bytes(ct[i] ^ key[i % len(key)] for i in range(len(ct)))
print(pt.decode())
```

That outputs the flag immediately.

---

## Flag

```
0xfun{h0mem4d3_f1rewall_305x908fsdJJ}
```
