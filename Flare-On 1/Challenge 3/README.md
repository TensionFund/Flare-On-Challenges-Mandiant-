# Flare-On Challenge 3: Such Evil

**Category:** Malware Analysis / Shellcode | **Difficulty:** Medium | **Platform:** Windows (Native C++)

## Challenge Description
The challenge consists of a Windows executable named `such_evil.exe`. Unlike the previous challenges, this is a native C++ application (indicated by the import of `msvcrt.dll`).

When executed, the program appears to do nothing visible, but analysis reveals it constructs and executes malicious code (shellcode) dynamically in memory.

## Tools Used
* **Ghidra** - *Static Analysis (C Entry Point)*
* **x32dbg** - *Dynamic Analysis & Shellcode Debugging*
* **VMware Workstation** (Windows 11 Guest) - *Execution Environment*

---

## 1. Initial Triage & Static Analysis

We began by analyzing `such_evil.exe` in **Ghidra**. The entry point was a standard Visual C++ wrapper (`entry`), which set up the environment and called the main function at `FUN_00401000`.

### The "Stack String" Discovery
Upon inspecting `FUN_00401000`, we observed a large number of local byte variables being assigned hexadecimal values (e.g., `local_205 = 0xe8`, `local_200 = 0x8b`).

Crucially, the function ended with a suspicious instruction:
```c
(*(code *)&local_205)();
```
### Interpretation
This line casts the address of the local variable array (`local_205`) to a function pointer and calls it[cite: 24]. [cite_start]This is a technique known as **Stack Strings** or **Shellcode Execution**, where the program manually builds machine code on the stack and then jumps into it[cite: 25]. [cite_start]This confirmed that the actual challenge logic was not visible in Ghidra because it did not exist until runtime[cite: 26].

## 2. Dynamic Analysis (x32dbg)
To analyze the hidden code, we switched to **x32dbg** on a Windows 11 VM[cite: 27, 28].

### Catching the Shellcode
1.  **Breakpoint:** We set a breakpoint at the end of the main function (`00401000`), specifically on the `CALL` instruction that triggered the shellcode[cite: 30].
2.  **Execution:** We ran the program (`F9`), allowing it to build the shellcode on the stack [cite: 31].
3.  **Step Into:** When the breakpoint hit, we stepped into (`F7`) the call, landing at the shellcode's entry point on the stack (Address `0019FD33`)[cite: 32].

### Decrypting the Logic
Inside the shellcode, we identified a decryption loop[cite: 34].

```assembly
0019FD48 | 8036 66      | XOR BYTE PTR DS:[ESI], 66  ; XOR key = 0x66 ('f')
0019FD4B | 46           | INC ESI                    ; Next byte
0019FD4C | 49           | DEC ECX                    ; Decrease counter
0019FD4D | EB F4        | JMP 19FD43                 ; Loop
```
This loop iterated over the subsequent memory block, XORing every byte with `0x66` to reveal the true instructions.

#### Action: 
We set a breakpoint immediately after the loop (0019FD4F) and ran the program to instantly decrypt the code.

## 3. The Solution

Once the code was decrypted, we stepped through the new instructions. The malware used a series of `MOV` instructions to construct the flag string byte-by-byte onto the stack.

By inspecting the **Stack Window** and the **Dump** at the target memory address (`ESI`), we reconstructed the flag from the "garbage" data.

### Flag
```
such.evil.object.oriented.programming@flare-on.com
```
### Lessons Learned

1.  **Recognize Shellcode:** If a C function builds a massive array of hex values and then calls it as a function, you are dealing with shellcode. Static analysis stops here.
2.  **Dynamic is Essential:** You cannot reverse engineer code that doesn't exist yet. Debuggers like x32dbg are required to let the program build itself before you can analyze it.
3.  **XOR Decryption:** Simple loops (`XOR`/`INC`/`DEC`/`JMP`) are the most common way malware hides its secrets. Identify the loop, break after it, and let the computer do the work for you.
