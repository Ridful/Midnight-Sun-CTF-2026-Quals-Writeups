### Writeup: smachine (Misc, 71pts)

**Challenge:** Blackbox register machine exposed over TCP. Goal: get any register to `0x1337`, then call `win`.

---

#### Recon

Connecting reveals a simple machine with 10 registers (`x0`–`x9`) and ops: `add`, `sub`, `mul`, `and`, `or`, `xor`. All start at 0. The `win` command tells you the objective immediately:

```
Set any register to 0x1337 in order to win.
```

---

#### Finding the Constraints

Trying `add x0 4919` (0x1337 in decimal) returns `Bad Result.` — the register doesn't change. Probing smaller values reveals:

- **Max immediate value: 255** — larger immediates are silently rejected
- **Any operation whose result equals 0x1337 returns `Bad Result.`** — the machine actively blocks you from landing on the target via arithmetic

This means you can't just add your way to 0x1337 directly; the last step is always blocked.

---

#### Solution

Use XOR for the final step to "jump" onto 0x1337 without an arithmetic operation producing it:

```
0x1300 XOR 0x37 = 0x1337
```

Build to `0x1300` (4864) safely:

- 19 × `add x0 255` → `0x12ED` (4845)
- `add x0 19` → `0x1300` (4864) ✓ not blocked
- `xor x0 55` → `0x1337` ✓ not blocked

Then call `win` and read until EOF (the server closes after printing the flag).

---

#### Solve Script

python

```python
from pwn import *

io = remote("smachine.play.ctf.se", 9189)
io.recvuntil(b"#> ")

def cmd(c):
    io.sendline(c.encode())
    return io.recvuntil(b"#> ", drop=True).decode().strip()

for _ in range(19):
    cmd("add x0 255")   # 19*255 = 4845 = 0x12ED

cmd("add x0 19")        # 0x12ED + 19 = 0x1300
cmd("xor x0 55")        # 0x1300 ^ 0x37 = 0x1337

io.sendline(b"win")
print(io.recvall().decode())
```

**Flag:** `midnight{s1MpL3_sT4cK_M4cH1N3}`
