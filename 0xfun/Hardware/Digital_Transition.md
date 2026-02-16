# Challenge Name: Digital Transition

## Description

**Category:** Hardware

> We intercepted a raw signal capture from an HDMI display adapter. The data appears to be a single digitized frame from a 640x480 HDMI output.

**Flag format:** (as shown on-screen)

---

## Writeup

### Step 1: Checking the binary size (timing clue)

The file was `signal.bin`.

When I checked the file size, it was:

- `1,680,216` bytes total

That number is extremely suspicious for a “single frame” capture:

- If I remove a small header (I found it was **216 bytes**),
- the remainder is **1,680,000 bytes**.

If the capture is 32-bit samples:

```
1,680,000 / 4 = 420,000 samples
```

And:

```
800 * 525 = 420,000
```

That matches classic VGA/HDMI 640×480 timing totals:
- 640×480 active video
- **800×525 total** including blanking/sync

So I assumed the data represents **one pixel clock per sample** for a full 800×525 timing frame.

---

### Step 2: Interpreting each sample as 3×10-bit TMDS symbols

HDMI uses TMDS (Transition-Minimized Differential Signaling), which is a **10-bit** symbol per color channel.

Each 32-bit sample in this capture packs:

- 10 bits for Blue (channel 0)
- 10 bits for Green (channel 1)
- 10 bits for Red (channel 2)

So for each little-endian `uint32 w`:

```python
ch0 =  w        & 0x3FF
ch1 = (w >> 10) & 0x3FF
ch2 = (w >> 20) & 0x3FF
```

---

### Step 3: TMDS decoding back to 8-bit RGB

For control periods (blanking), the symbols use fixed “control codes”. On the blue channel these are commonly:

- `0x354`, `0x0AB`, `0x154`, `0x2AB`

For everything else, I used the standard TMDS inverse:

- Bit 9 is the “invert” flag for the 8-bit payload portion
- Bit 8 tells me if stage-1 used XOR or XNOR
- Then I reconstruct the original 8 data bits using the XOR/XNOR recurrence

Here’s the exact Python I used to decode the frame:

```python
import numpy as np
from PIL import Image

CONTROL_CODES = {0x354, 0x0AB, 0x154, 0x2AB}

def tmds_decode_array(codes: np.ndarray):
    codes = codes.astype(np.uint16)

    ctrl_mask = np.isin(codes, np.array(list(CONTROL_CODES), dtype=np.uint16))

    bit9 = (codes >> 9) & 1
    bit8 = (codes >> 8) & 1

    q = (codes & 0xFF).astype(np.uint8)
    q = q ^ (bit9.astype(np.uint8) * 0xFF)  # invert when bit9=1

    # stage-1 inverse to recover D[7:0]
    D = np.zeros_like(q, dtype=np.uint8)
    prev = (q & 1).astype(np.uint8)
    D |= prev

    for i in range(1, 8):
        qi = ((q >> i) & 1).astype(np.uint8)
        di = (qi ^ prev)
        di = np.where(bit8 == 1, di, di ^ 1)  # XNOR path
        D |= (di.astype(np.uint8) << i)
        prev = qi

    D = np.where(ctrl_mask, 0, D)  # blank controls to black
    return D

# --- load + parse ---
raw = open("signal.bin", "rb").read()

HEADER = 216
payload = raw[HEADER:]

words = np.frombuffer(payload, dtype="<u4")

ch0 =  words        & 0x3FF
ch1 = (words >> 10) & 0x3FF
ch2 = (words >> 20) & 0x3FF

b = tmds_decode_array(ch0)
g = tmds_decode_array(ch1)
r = tmds_decode_array(ch2)

W, H = 800, 525
img = np.stack([r.reshape(H, W), g.reshape(H, W), b.reshape(H, W)], axis=2)

# The capture starts mid-line, so the image wraps.
# I fixed it by rolling the image horizontally until the seam disappeared.
SHIFT = 750
img = np.roll(img, -SHIFT, axis=1)

Image.fromarray(img.astype(np.uint8), "RGB").save("decoded.png")
```

---

### Step 4: Reading the flag from the decoded frame

After decoding, the output image clearly showed the flag text overlaid on the meme:

```
0XFUN{TMDS_D3C0DED_LIKE_A_PRO}
```

---

## Flag

```
0XFUN{TMDS_D3C0DED_LIKE_A_PRO}
```
