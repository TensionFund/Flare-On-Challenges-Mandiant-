# Flare-On Challenge 4: APT9001

**Category:** Malicious PDF / Shellcode | **Difficulty:** Medium | **Platform:** Windows / JavaScript

## Challenge Description

We are provided with a PDF named `APT9001.pdf`. The document contains decoy text regarding a Chinese cyber espionage unit. The true challenge lies in analyzing the PDF structure to find embedded malicious code.

## Tools Used

- `pdfid` & `pdf-parser` — PDF Triage & Extraction
- `Python` — Zlib / Hex Decoding & Shellcode Extraction
- `x32dbg` / `blobrunner` — Dynamic Execution Analysis

---

## 1. Initial Triage (pdfid)

We began by analyzing the PDF structure using `pdfid`.
```
pdfid APT9001.pdf
```

The scan revealed critical anomalies:

- `/JS`: 1
- `/JavaScript`: 1
- `/OpenAction`: 1

This confirmed the PDF contained a single embedded JavaScript payload designed to execute automatically as soon as the document is opened.

## 2. Surgical Extraction

Using `pdf-parser`, we traced the `/OpenAction` tag to Object 5, which pointed to Object 6 as the container for the actual JavaScript.

The stream in Object 6 was obfuscated. The filters were deliberately mangled (e.g., `/Fla#74eDe...`) to break automated analysis tools. We bypassed the broken PDF headers by writing a custom Python extraction script that:

1. Located the raw binary stream of Object 6.
2. Decompressed the payload using **Zlib** (FlateDecode).
3. Decoded the resulting text using **ASCII Hex Decoding**.

This successfully extracted the malicious script, `malware.js`.

## 3. Shellcode Analysis (Heap Spray)

Analyzing `malware.js` revealed a classic **Heap Spray** exploit. The script allocated massive arrays and filled them with a repetitive Unicode string (`%u72f9%u4649...`).

These Unicode values were not text, but **Little-Endian x86 Shellcode**. We wrote a secondary Python script to parse the `%u` values, convert them into raw bytes, and save the result as `shellcode.bin`.

## 4. Execution & The Environmental Fragility Problem

We attempted to execute the raw shellcode dynamically on a Windows VM using `blobrunner.exe` and `x32dbg`.

While the shellcode successfully executed its payload (popping a `MessageBoxA` window), the text inside the popup was gibberish (`5$t%*k!u/k66.ut21...`).

This occurred due to the fragility of shellcode. Running it outside of its native Adobe Reader environment corrupted the stack alignment. When the shellcode grabbed the stack pointer (`ESP`) to begin its XOR decryption loop, it was offset by several bytes, causing it to decrypt the wrong memory segment.

## 5. Static Bypass: Self-Modifying Code & Stack String Reconstruction

Since dynamic execution failed, we pivoted to pure static analysis of the shellcode binary.

### Identifying the Decryption Routine

By hex-dumping `shellcode.bin`, we identified the following structure after a `CALL +0` (GET EIP) trick at offset 857:
```
E8 00 00 00 00 CALL +0 (get current address)
8B 14 24 MOV EDX, [ESP] (EDX = current EIP)
81 72 0B [key1] XOR DWORD PTR [EDX+0x0B], key1
68 [encrypted1] PUSH encrypted_dword_1
81 72 17 [key2] XOR DWORD PTR [EDX+0x17], key2
68 [encrypted2] PUSH encrypted_dword_2
... (8 pairs total)
81 36 42 45 45 46 XOR DWORD PTR [ESI], 0x46454542 ("BEEF")
```

The shellcode used **self-modifying code**: each `XOR DWORD PTR [EDX+offset]` instruction decrypted the adjacent `PUSH` operand in-place before execution. The `PUSH` instructions then built the flag string on the stack in **reverse order** (LIFO).

### Manual Decryption

We extracted the 8 PUSH operands and their corresponding per-block XOR keys directly from the hex dump:

| PUSH Operand | XOR Key | Decrypted (LE) |
|---|---|---|
| `0x32BECE79` | `0x32FBA316` | `omE\x00` |
| `0x2BE12BC1` | `0x48CF45AE` | `on.c` |
| `0xFFFA4471` | `0xD29F3610` | `are-` |
| `0x60CFE984` | `0x0CA9A9F7` | `s@fl` |
| `0x3798A3D2` | `0x43A993BE` | `l01t` |
| `0x4B11A4EF` | `0x3B628A82` | `m.sp` |
| `0xFFA469BE` | `0xCCC047D6` | `h.d3` |
| `0x5265ABD4` | `0x3154CAA3` | `wa1c` |

Reading bottom-to-top (stack LIFO order):
```
wa1c | h.d3 | m.sp | l01t | s@fl | are- | on.c | om
```
## Flag
```
wa1ch.d3m.spl01ts@flare-on.com
```

---

## Lessons Learned

1. **Trust the X-Ray:** Tools like `pdfid` are essential for cutting through decoy content and instantly locating execution vectors like `/OpenAction`.

2. **Tools Break:** Malware authors intentionally mangle file headers to break standard tools like `pdf-parser`. Knowing how to manually decompress (Zlib) and decode (Hex) streams is crucial.

3. **Shellcode is Fragile:** Executing exploit code outside of its intended environment (e.g., using `blobrunner`) can lead to misaligned registers, causing internal decryption loops to yield garbage data.

4. **Self-Modifying Code Defeats Static Scans:** Breaking encrypted strings across `PUSH` instructions with per-block XOR keys ensures that the flag never exists contiguously on disk. Understanding the decryption routine and manually replicating its logic is required to recover the payload.
