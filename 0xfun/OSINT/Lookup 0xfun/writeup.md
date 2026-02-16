# Challenge Name: Lookup 0xfun

## Description

**Category:** OSINT

> This event takes place on **ctf.0xfun.org**, but you can easily find it by searching.

**Flag format:** `0xfun{...}`

---

## Writeup

### Step 1:

First, I used the Google search engine and searched for "0xfun" to find everything related to it, but I couldn't find the flag that way.

### Step 2:

Then, I read the description again and thought about searching the backend. I used the following command to search for DNS records:

```bash
nslookup -type=TXT ctf.0xfun.org
```

### Step 3: Reading the TXT Records

The response included TXT records. One of them contained the flag:

```
ctf.0xfun.org   text = "0xfun{4ny_1nfo_th4ts_pub1cly_4cc3ss1bl3_1s_0S1NT}"
```

---

## Flag

```
0xfun{4ny_1nfo_th4ts_pub1cly_4cc3ss1bl3_1s_0S1NT}
```