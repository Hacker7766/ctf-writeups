# Challenge Name: MultiVerse

## Description

**Category:** OSINT

> I have a friend named **Massive-Equipment393** who's obsessed with music. Try to figure out what his favorite genre is.

**Flag format:** `0xfun{...}`

---

## Writeup

### Step 1: Username Lookup

I used google dork for searching his username: `Massive-Equipment393`

### Step 2: Reddit and First Flag Part

On **Reddit**: [Massive-Equipment393](https://www.reddit.com/user/Massive-Equipment393/)

Their Reddit display name is **Ph0n8xV1me** ([lookup results](Resources/Ph0n8xV1me.txt)).

In their comments, I found out:

```
Playlist
all 49Rak48kGp7nJoUq9ofCX everyday.
```

The string `49Rak48kGp7nJoUq9ofCX` is **Base58**. I decoded it to get:

```
pl4yl1st_3xt3nd
```

This is the **second part** of the flag.

### Step 3: Spotify and First Part

The user's profile links to **Spotify**:

```
https://open.spotify.com/user/3164whos3zc5xss6lv7ejfdlmogi
```

In one playlist (name or description) find a **Base64** string:

```
MHhmdW57c3AwdDFmeV8=
```

Decode to get: `0xfun{sp0t1fy_` — the **first part** of the flag.

### Step 4: Third Part

I noticed a playlist called `My Playlist #2` on the Spotify account. I looked at the song names which I found out to be unusual, such as `_WORLD`, etc. Since underscores are commonly used in flags, this seemed important.

I took the first letter of each song name to extract the third part:

`_M0R3_TR4X}`

---

## Flag

`0xfun{sp0t1fy_pl4yl1st_3xt3nd__M0R3_TR4X}`