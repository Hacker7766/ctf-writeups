# Challenge Name: Nanom-dinam???itee?

## Description

**Category:** Reverse Engineering

> Don't trust what you see, trust what happens when no one is looking.

**Flag format:** `0xfun{...}`

---

## Writeup

### Step 1: Running the program (the “don’t trust what you see” trick)

When I ran the binary, it asked for a password:

```
Pass:
```

If I typed *anything* with the wrong length, it happily printed a “flag”:

```
Oh sure, here is your flag: 0xfun{1_10v3_M1LF}
```

That’s obviously bait.

So I went into reversing mode.

---

### Step 2: Understanding the fork + ptrace design

Disassembling the binary showed:

- The program `fork()`s
- The **child** reads my input
- If my input length is not `0x28` (40), it prints the **fake flag** and exits
- If length is 40, the child computes a rolling hash **one byte at a time**
- After each byte, the child executes `ud2` (illegal instruction) to intentionally crash
- The **parent** is attached via `ptrace()` and, after every crash:
  - reads the child’s registers (`PTRACE_GETREGS`)
  - checks a table of expected values
  - if correct: it patches RIP to skip `ud2` and continues
  - if wrong: it detaches/exits, so I don’t get any helpful output

That matches the hint: **the real validation happens “when no one is looking”** (in the parent, not in the output).

---

### Step 3: Extracting the hash function and the expected table

The rolling hash function (called with length `1` per loop iteration) is:

- Start state:
```
h0 = 0xcbf29ce484222325
```

- Per input byte `b`:
```
t  = (h ^ b) * 0x100000001b3  (mod 2^64)
h' = t ^ (t >> 32)
```

The expected 40 intermediate states are stored in `.rodata` at `0x20a0` as 40 little-endian `uint64`.

---

### Step 4: Recovering the password (and flag) byte-by-byte

Because each step depends only on the previous hash and *one byte*, I didn’t need fancy inversion.

I just brute-forced each character:

- I already know `h_{i-1}`
- I know `expected_h_i` from the table
- so I try all `b in [0..255]` until `step(h_{i-1}, b) == expected_h_i`

That’s only `40 * 256 = 10240` checks — trivial.

```python
import struct

FNV_PRIME = 0x100000001b3
MASK = (1 << 64) - 1

def step(h, b):
    t = ((h ^ b) * FNV_PRIME) & MASK
    return (t ^ (t >> 32)) & MASK

bin_data = open("nanom-dinam-ite__iteee", "rb").read()

# .rodata is at file offset 0x2000, and the table starts at 0x20a0
table_off = 0x20a0
table = [struct.unpack_from("<Q", bin_data, table_off + i*8)[0] for i in range(40)]

h = 0xcbf29ce484222325
out = []
for expected in table:
    for b in range(256):
        if step(h, b) == expected:
            out.append(b)
            h = expected
            break

print(bytes(out).decode())
```

The recovered 40-byte password is the flag.

---

## Flag

```
0xfun{unr3adabl3_c0d3_is_s3cur3_c0d3_XD}
```
