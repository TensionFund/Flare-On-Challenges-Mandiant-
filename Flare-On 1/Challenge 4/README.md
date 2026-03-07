# Flare-On Challenge 4: APT9001

**Category:** Malicious PDF / Shellcode | **Difficulty:** Medium | **Platform:** Windows / JavaScript

## Challenge Description
We are provided with a PDF named `APT9001.pdf`. The document contains decoy text regarding a Chinese cyber espionage unit. The true challenge lies in analyzing the PDF structure to find embedded malicious code.

## Tools Used
* **pdfid & pdf-parser** - *PDF Triage & Extraction*
* **Python** - *Zlib / Hex Decoding & Shellcode Extraction*
* **x32dbg / blobrunner** - *Dynamic Execution Analysis*

---

## 1. Initial Triage (pdfid)
We began by analyzing the PDF structure using `pdfid`. 

```bash
pdfid APT9001.pdf
```
The scan revealed critical anomalies:

- /JS: 1
- /JavaScript: 1
- /OpenAction: 1

This confirmed the PDF contained a single embedded JavaScript payload designed to execute automatically as soon as the document is opened.

## 2. Surgical Extraction

Using pdf-parser, we traced the /OpenAction tag to Object 5, which pointed to Object 6 as the container for the actual JavaScript.

The stream in Object 6 was obfuscated. The filters were deliberately mangled (e.g., /Fla#74eDe...) to break automated analysis tools. We bypassed the broken PDF headers by writing a custom Python extraction script that:

1. Located the raw binary stream of Object 6.
2. Decompressed the payload using **Zlib** (FlateDecode).
3. Decoded the resulting text using **ASCII Hex Decoding**.

This successfully extracted the malicious script, malware.js.

## 3. Shellcode Analysis (Heap Spray)

Analyzing malware.js revealed a classic **Heap Spray** exploit. The script allocated massive arrays and filled them with a repetitive Unicode string (%u72f9%u4649...).

These Unicode values were not text, but **Little-Endian x86 Shellcode**. We wrote a secondary Python script to parse the %u values, convert them into raw bytes, and save the result as shellcode.bin.

## 4. Execution & The "Stack String" Defense

We attempted to execute the raw shellcode dynamically on a Windows VM using blobrunner.exe and x32dbg.

While the shellcode successfully executed its payload (popping a MessageBoxA window), the text inside the popup was gibberish (5$t%*k!u/k66.ut21...).

This occurred due to the fragility of shellcode. Running it outside of its native Adobe Reader environment corrupted the stack alignment. When the shellcode grabbed the stack pointer (ESP) to begin its XOR decryption loop, it was offset by several bytes, causing it to decrypt the wrong memory segment.

**The Static Bypass:**

We attempted to statically brute-force the 4-byte XOR key (BEEF) against the raw binary file. However, this also failed due to the malware's use of **Stack Strings**. The encrypted payload was not stored contiguously in the file; it was broken up every 4 bytes by a 0x68 (PUSH) opcode.

By manually extracting the PUSH operands and XORing them against the 0x46454542 (BEEF) key, the true string was revealed.

**Flag:**
```
should_have_gone_to_tashi_station@flare-on.com
```

---

## Lessons Learned

1. **Trust the X-Ray:** Tools like pdfid are essential for cutting through decoy content and instantly locating execution vectors like /OpenAction.

2. **Tools Break:** Malware authors intentionally mangle file headers to break standard tools like pdf-parser. Knowing how to manually decompress (Zlib) and decode (Hex) streams is crucial.

3. **Shellcode is Fragile:** Executing exploit code outside of its intended environment (e.g., using blobrunner) can lead to misaligned registers, causing internal decryption loops to yield garbage data.

4. **Stack Strings Defeat Static Scans:** Breaking strings up into PUSH instructions ensures that static decryption brute-forcers will fail, as the target string never exists contiguously on disk.
