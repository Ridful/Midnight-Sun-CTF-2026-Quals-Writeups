## Challenge Overview: **riscal**

- **Category:** Reverse Engineering
    
- **Architecture:** RISC-V 64-bit (Little-endian)
    
- **Difficulty:** Introductory / Speed
    
- **Objective:** Find the required input string to satisfy the binary's comparison logic.
    

---

## Technical Analysis

### 1. Identifying the File

The binary is a RISC-V ELF. Because it isn't an x86/x64 binary, you cannot execute it directly on a standard Linux machine without an emulator like `qemu-riscv64`.

Bash

```
$ file riscal
riscal: ELF 64-bit LSB pie executable, UCB RISC-V, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux-riscv64-lp64d.so.1, for GNU/Linux 4.15.0, stripped
```

### 2. Static Analysis (Strings)

A simple `strings` command reveals the most critical components of the challenge:

- **The Flag:** `midnight{RISCV_1S_4_34zy_1S4_70_unDeRst4Nd!!}`
    
- **Success Message:** `yeh!`
    
- **Failure Message:** `nah!`
    
- **The Passphrase:** `Reversing and pwning are both science and art`
    

### 3. Binary Logic

If you open the binary in a tool like **Ghidra** (with the RISC-V processor module) or **IDA Pro**, you will find a `main` function that performs the following steps:

1. **Buffer Allocation:** Sets aside space for user input via `fgets` or `read`.
    
2. **Comparison:** Uses `strcmp` or a manual loop to compare the user's input buffer against the hardcoded string:
    
    > `"Reversing and pwning are both science and art"`
    
3. **Branching:**
    
    - If the strings **match**, it executes the "Success" block which prints `yeh!` and the flag.
        
    - If the strings **differ**, it prints `nah!` and exits.
        

---

## Solution Execution

The server at `nc riscal.play.ctf.se 1338` is running this binary. To get the official flag (or confirm the one found in the binary), you must provide the passphrase.

### Step-by-Step Solve:

1. Connect to the remote instance: `nc riscal.play.ctf.se 1338`
    
2. When prompted with `flag:`, paste the following: `Reversing and pwning are both science and art`
    
3. The server will respond: `yeh! midnight{RISCV_1S_4_34zy_1S4_70_unDeRst4Nd!!}`
    

---

## Key Takeaways

- **Arch-Agnostic Strings:** Even if you can't run a binary because of its architecture, `strings` and static analysis often reveal 90% of the solution in introductory challenges.
    
- **RISC-V Simplicity:** As the flag suggests, RISC-V is a Reduced Instruction Set Computer. Its assembly is often much cleaner and easier to read than x86_64, making it a favorite for modern CTF "entry-level" reversing tasks.

