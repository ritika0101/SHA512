# SHA512
## What This Does

A from-scratch, pure-Python implementation of **SHA-512**.  
It takes an input string, applies the SHA-512 hash algorithm (80 rounds per 1024-bit block, 64-bit arithmetic),  
and prints the **128-hex-character digest**.

---

## How It Works (High Level)

### Bit Helpers

- **ROTR**, **SHR** → 64-bit rotate right and logical right shift (masked to 64 bits).  
- **maj**, **Ch** → SHA majority and choose functions.  
- **sigmaA**, **sigmaE** → The “big sigmas” used in the compression round (`ROTR 28/34/39` and `14/18/41`).

---

### Constants & IV

- **K** → The 80 round constants defined by SHA-512 (64-bit).  
- **H** → The initial 8 hash state words (IV) for SHA-512.

---

### Preprocessing (Padding)

1. Convert the input string to a bitstring (binary).  
2. Append a single `1` bit, then `0`s so that the length ≡ 896 (mod 1024).  
3. Append the **128-bit big-endian length** of the original message (in bits).  
4. Result is split into **1024-bit chunks**.

---

### Message Schedule `w[0..79]`

- For each 1024-bit chunk, parse the first **16 × 64-bit words**.  
- Extend to 80 words using the SHA-512 schedule:

```python
s0 = ROTR(w[j-15], 1) ^ ROTR(w[j-15], 8) ^ SHR(w[j-15], 7)
s1 = ROTR(w[j-2], 19) ^ ROTR(w[j-2], 61) ^ SHR(w[j-2], 6)
w[j] = (w[j-16] + s0 + w[j-7] + s1) % (2**64)
```

### Compression (80 Rounds)

Initialize working variables a..h from H.

Each round computes:
```python
S1    = sigmaE(e)
ch    = Ch(e, f, g)
temp1 = h + S1 + ch + K[j] + w[j]
S0    = sigmaA(a)
maj3  = maj(a, b, c)
temp2 = S0 + maj3
```

---

Rotate the registers and update using (mod 2^64) arithmetic.

### State Update & Digest

Add the working variables back into H.  
After all chunks, format H as 8 × 16-hex-digit words → 128-hex-character hash.
