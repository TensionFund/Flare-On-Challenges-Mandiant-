# CyberChef Verification

### Open CyberChef: 
```
https://gchq.github.io/CyberChef
```

We will do each DWORD one at a time to verify the decryption. Let me start with the last PUSH operand since that is the first part of the flag (LIFO).

## Quick Reference Table

| Block | Input Hex | XOR Key Hex | Expected Output |
|-------|-----------|-------------|-----------------|
| 8 | `d4ab6552` | `a3ca5431` | `wa1c` |
| 7 | `be69a4ff` | `d647c0cc` | `h.d3` |
| 6 | `efa4114b` | `828a623b` | `m.sp` |
| 5 | `d2a39837` | `be93a943` | `l01t` |
| 4 | `84e9cf60` | `f7a9a90c` | `s@fl` |
| 3 | `7144faff` | `10369fd2` | `are-` |
| 2 | `c12be12b` | `ae45cf48` | `on.c` |
| 1 | `79cebe32` | `16a3fb32` | `om` |

**Reading order (Stack LIFO — bottom to top):**
```
wa1c + h.d3 + m.sp + l01t + s@fl + are- + on.c + om
```
**Flag:**
```
wa1ch.d3m.spl01ts@flare-on.com
```
