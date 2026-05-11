# CTF Writeup: sanity

**Category:** Misc / Sanity Check

**Points:** 91

**Author:** @quend

## 1. Challenge Overview

The challenge provides a single obfuscated string and a bash pipeline. The goal is to reverse the operations performed on the string to reveal the hidden flag.

**Provided String:** `K0nbxQzZ08FMn91M391MyNDafdjKoN3X3dHN7RHanlmbklWb`

---

## 2. Technical Analysis

The challenge utilizes two basic transformations via the Linux command line:

1. **`rev`**: A utility that reverses the order of characters in every line.
    
2. **`base64 -d`**: A tool to decode data that has been encoded using the Base64 alphabet.
    

In the provided command, the string is first **reversed**, and the resulting output is then **decoded**.

---

## 3. Solution

By executing the pipeline in a terminal, we can see the transformation in real-time:

### Step 1: Reversal

Reversing `K0nbxQzZ08FMn91M391MyNDafdjKoN3X3dHN7RHanlmbklWb` yields: `bWlknbmnalHR7NHd3X3NKo jdfaDNyM193M19nMF8Z0xbn0K`

### Step 2: Base64 Decoding

Decoding that reversed string results in the plain-text flag.

**Command execution:**

Bash

```
echo 'K0nbxQzZ08FMn91M391MyNDafdjKoN3X3dHN7RHanlmbklWb' | rev | base64 -d
```

---

## 4. Flag

The output of the command reveals the flag: `midnight{4ww_sh*7_h3r3_w3_g0_4g41n}`


