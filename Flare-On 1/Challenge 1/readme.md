# Flare-On Challenge 1: Bob Doge

**Category:** Reverse Engineering | **Difficulty:** Easy | **Platform:** Windows (.NET)

## Challenge Description
The first challenge of the Flare-On series is a Windows executable named `C1.exe`. Upon execution, it displays a modified image of Bob Ross (with a "Doge" face) and a button labeled "Decode". Clicking the button transforms the image and displays cryptic text.

The goal is to reverse engineer the application to find the flag.

## Tools Used
* **VMware Workstation** (Windows 11 Guest) - *Safe execution environment*
* **7-Zip** - *Archive extraction*
* **dnSpy** - *Main .NET debugger/decompiler*
* **Ghidra** - *Initial static analysis*

---

## 1. Initial Triage & Extraction

We started by analyzing the downloaded file, `C1.exe`.
Running the file in a secure VM environment revealed a simple GUI with a "Decode" button.

### The "Wrapper" Discovery
Initial analysis in **Ghidra** led to a large amount of boilerplate code (e.g., `FUN_140003010`), which included strings like `EXTRACTOPT`, `TITLE`, and `VERCHECK`.

This indicated that `C1.exe` was not the actual challenge logic, but a **Self-Extracting Archive (SFX)** or an installer wrapper.

**Action:**
Instead of reverse engineering the Microsoft installer code, we used an archive tool (7-Zip) to inspect `C1.exe`. Inside, we found the actual payload: **`Challenge1.exe`**.

---

## 2. Static Analysis (.NET)

We loaded the extracted `Challenge1.exe` into **dnSpy** since it was identified as a .NET executable.

### Locating the Logic
In the namespace view, we navigated to `Form1`, which represents the main window. Inside `Form1`, we identified the event handler for the "Decode" button: `btnDecode_Click`.

### The Decompiled Code
The complete logic for generating the flag was found in this function:

```csharp
private void btnDecode_Click(object sender, EventArgs e)
{
    this.pbRoge.Image = Resources.bob_roge;
    byte[] dat_secret = Resources.dat_secret;
    string text = "";

    // Stage 1: Nibble Swapping and XOR
    foreach (byte b in dat_secret)
    {
        text += (char)((b >> 4 | ((int)b << 4 & 240)) ^ 41);
    }
    text += "\0";

    // Stage 2: Character Swapping
    string text2 = "";
    for (int j = 0; j < text.Length; j += 2)
    {
        text2 += text[j + 1];
        text2 += text[j];
    }

    // Stage 3: Final XOR
    string text3 = "";
    for (int k = 0; k < text2.Length; k++)
    {
        char c = text2[k];
        text3 += (char)((byte)text2[k] ^ 102);
    }

    this.lbl_title.Text = text3;
}
```

---

## 3. The Solution

The function performs three distinct operations on a hidden resource byte array (`dat_secret`):
1.  **Nibble Swap & XOR:** Swaps the high/low 4 bits of each byte and XORs with `41` (0x29).
2.  **Permutation:** Swaps adjacent characters (indices 0 and 1, 2 and 3, etc.).
3.  **Final XOR:** XORs the result with `102` (0x66).

### Retrieving the Flag
Using the debugger features in **dnSpy**, we placed a breakpoint on the line `this.lbl_title.Text = text3;`.

Upon running the program and clicking "Decode", the execution paused, allowing us to inspect the variable `text3` in the **Locals** window.

**Flag:**
```
3rmahg3rd.b0b.d0ge@flare-on.com
```

---

## Lessons Learned
1.  **Always Check for Wrappers:** If the entry point looks like standard Microsoft installation code, check if the binary is an SFX before reversing it.
2.  **Choose the Right Tool:** While Ghidra is powerful, tools like **dnSpy** or **ILSpy** are vastly superior for .NET executables due to their ability to reconstruct near-perfect source code.
3.  **Dynamic Analysis is Fast:** Debugging the live application to read the variable from memory is often faster than rewriting the decryption algorithm manually.
