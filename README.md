# Flare-On Reverse Engineering Portfolio

[![Category: Malware Analysis](https://img.shields.io/badge/Category-Malware%20Analysis-red.svg)](#)
[![Difficulty: Advanced](https://img.shields.io/badge/Difficulty-Advanced-orange.svg)](#)
[![Focus: Reverse Engineering](https://img.shields.io/badge/Focus-Reverse%20Engineering-blue.svg)](#)

## 📌 Executive Summary
This repository documents my methodological approach to solving challenges from **Flare-On**, the premier reverse engineering and malware analysis competition created by the Mandiant Advanced Practices team. 

Beyond simply finding "flags," the write-ups in this repository emphasize **practical malware analysis methodology**, detailing how to unpack obfuscated code, debug live execution environments, extract malicious shellcode, and bypass anti-analysis techniques used by real-world threat actors.

### 👤 About the Author
I hold a Master's degree in Cyber Security from the University of Birmingham. I am an engineer with a deep passion for low-level systems, telecommunications security, and protocol analysis. 

**Notable Real-World Engineering:**
Alongside reverse engineering software, I have extensive experience in hardware and signal-level security. I authored a Master's dissertation focused on **Diagnosing and Improving a 5G NR Sniffer for PRACH Detection** utilizing USRP B210 / NR-Scope hardware. This required deep protocol-level analysis, signal processing, and a rigorous engineering mindset—skills that directly translate into my approach to dismantling complex, obfuscated software vulnerabilities.

---

## 🛠️ Core Competencies & Toolchain
* **Static Analysis:** Ghidra, IDA, dnSpy, ILSpy, CFF Explorer.
* **Dynamic Analysis & Debugging:** x32dbg, OllyDbg, Windbg.
* **Malicious Document & Web Analysis:** pdfid, pdf-parser, PHP/HTML deobfuscation, custom Python extraction scripting.
* **Shellcode & Payload Emulation:** scdbg, blobrunner, manual memory extraction.
* **Languages & Architecture:** x86/x64 Assembly, C/C++, C#/.NET, Python, JavaScript, PHP.

---

## 📂 Challenge Highlights & Technical Write-Ups

### [Challenge 1: Bob Doge (Windows .NET) - View Write-up](./Flare-On%201/Challenge%201)
A reverse engineering challenge focused on .NET decompilation and de-obfuscation.
* **Techniques Demonstrated:** * Identifying and bypassing Self-Extracting Archive (SFX) wrappers.
  * .NET static analysis and live debugging using **dnSpy**.
  * Reversing custom cryptography (nibble swapping, character permutation, and XOR decryption).
* **Key Takeaway:** Recognizing when standard static analysis (e.g., Ghidra) is looking at an installer wrapper rather than the true payload, and pivoting to the correct decompilation tools.

### [Challenge 2: Web Polyglot (home.html) - View Write-up](./Flare-On%201/Challenge%202)
An analysis of a web-based challenge masquerading as a simple HTML countdown page.
* **Techniques Demonstrated:**
  * Identifying server-side execution vectors hidden within static HTML.
  * Discovering a hidden PHP include directive (`<?php include "img/flare-on.png" ?>`) embedded at the bottom of the `home.html` file.
  * Recognizing **Polyglot Files**: Deducing that the referenced `flare-on.png` is not merely an image, but a disguised PHP script intended for server-side execution.
* **Key Takeaway:** Threat actors frequently hide malicious code inside seemingly benign static assets (like PNGs) and rely on intentional includes or server misconfigurations to execute them.

### [Challenge 3: Such Evil (Native C++) - View Write-up](./Flare-On%201/Challenge%203)
An analysis of a native Windows executable that utilizes dynamic code generation to hide its true intent.
* **Techniques Demonstrated:**
  * Identifying **Stack Strings** and **Shellcode Execution** built at runtime.
  * Pivoting from static analysis in Ghidra to dynamic debugging in **x32dbg**.
  * Locating and stepping through a live XOR decryption loop in memory (`XOR BYTE PTR DS:[ESI], 66`) to catch the payload before execution.
* **Key Takeaway:** Static analysis tools are blind to code that does not exist until runtime; deploying a debugger to let the malware unpack itself is essential.

### [Challenge 4: APT9001 (Malicious PDF & Shellcode) - View Write-up](./Flare-On%201/Challenge%204)
A complex challenge mimicking an Advanced Persistent Threat (APT) spear-phishing document containing heavily obfuscated JavaScript and fragile shellcode.
* **Techniques Demonstrated:**
  * Performing PDF triage using `pdfid` to locate `/OpenAction` and `/JS` execution triggers.
  * Bypassing intentionally mangled PDF filters (which break standard tools like `pdf-parser`) by writing custom Python scripts to perform raw Zlib decompression and ASCII-Hex decoding.
  * Identifying **Heap Spray** mechanics and extracting Little-Endian x86 shellcode from Unicode (`%u`) strings.
  * Dealing with **Environmental Fragility**: Diagnosing why shellcode decrypts garbage data when run outside its native environment (stack pointer misalignment).
  * Defeating **Self-Modifying Code**: Manually parsing the shellcode's 
    decryption routine, which used per-block XOR keys embedded in 
    `XOR DWORD PTR [EDX+offset]` instructions to decrypt PUSH operands 
    that built the flag string on the stack in reverse (LIFO) order.
* **Key Takeaway:** Automated tools and dynamic execution often fail against highly tailored exploits. A deep understanding of data structures, raw byte manipulation, and stack mechanics is required to manually extract and decrypt the payload.

---

## 🧠 My Analytical Methodology
I approach malware analysis not as a puzzle to guess, but as an engineered system to dismantle. My methodology relies on:
1. **Behavioral Triage:** Establishing the execution chain and identifying initial indicators of compromise without executing the payload.
2. **Surgical Extraction:** Building custom tooling (often in Python) to bypass broken file headers, mangled filters, or anti-analysis tricks designed to crash standard tools.
3. **Environment Control:** Understanding the exact environmental dependencies of a payload (e.g., specific vulnerable software versions, stack alignments) before attempting dynamic execution.
4. **Algorithmic Reversal:** Locating the core decryption loops (`XOR`/`INC`/`DEC`) in assembly, extracting the keys directly from CPU registers, and applying them natively to recover the underlying logic.

---
## 📫 Contact

- **LinkedIn:** [www.linkedin.com/in/zain-bin-hasan]
- **Email:** [szainbh@gmail.com]
- **Location:** United Kingdom
- **Status:** Actively seeking roles in Malware Analysis, 
  Threat Intelligence, Detection Engineering, and SOC Analysis.
