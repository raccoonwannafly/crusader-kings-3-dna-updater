# crusader-kings-3-dna-updater
Updates CK3 old DNA
live at: https://raccoonwannafly.github.io/crusader-kings-3-dna-updater/

---
Allows you to import outdated *Crusader Kings 3* DNA strings (from versions prior to 1.14/Royal Court) into the current game version.

## The Problem
CK3's clipboard parser performs a "Sanity Check" on every DNA string you try to paste. It compares the **Byte Length** of your clipboard data against the expected length for the current game version.
* **Old DNA:** Short length (e.g., 200 bytes).
* **New DNA:** Long length (e.g., 400 bytes).

Because the lengths do not match, the game assumes the code is corrupted and rejects it immediately.

## The Solution: "The Template Graft"
This tool immitates the memory-injection method used by Cheat Engine but performs it safely in the browser using JavaScript. It uses a **"Template Grafting"** algorithm.

### 1. The Container (Template)
We take a valid DNA string from the *current* game version (the "Template"). We use this as a container to ensure the final output has the exact file structure and byte length the game expects.

### 2. The Graft (Byte Overwrite)
We decode both the **Old DNA** and the **Template DNA** from Base64 into raw Byte Arrays (`Uint8Array`). We then loop through the Old DNA and overwrite the Template's bytes one by one.

* **If Old DNA is shorter:** We stop writing when we run out of old data. The remaining bytes (which control new features added in recent DLCs) remain as they were in the Template (default/randomized).
* **If Old DNA is longer (e.g., EPE Mod):** We stop writing when we reach the end of the Template. The extra data is safely truncated (discarded) to prevent buffer overflow.

### 3. The Result
The final string is re-encoded into Base64.
* **The Game sees:** A string with the correct header and correct length. It accepts the paste.
* **The Player sees:** The old facial features (which are stored in the first ~100 bytes) applied successfully to the new character structure.

## Technical Implementation
The core logic relies on JavaScript's `Uint8Array` to manipulate the binary data directly.

```javascript
// 1. Decode Base64 to Binary
const oldBytes = new Uint8Array(atob(oldDNA));
const templateBytes = new Uint8Array(atob(templateDNA));

// 2. Create the Container (Clone the Template)
const finalBytes = new Uint8Array(templateBytes);

// 3. Graft Data (Overwrite Template with Old Data)
// Math.min ensures we never write past the end of the array
const limit = Math.min(finalBytes.length, oldBytes.length);

for (let i = 0; i < limit; i++) {
    finalBytes[i] = oldBytes[i];
}
```
```text
Visualizing the Graft:

[ OLD DNA (Short) ]      [ TEMPLATE DNA (Long) ]
| 0x1A | 0x4F | 0xB2 |   | 0x00 | 0x00 | 0x00 | 0x00 | 0x00 |
   |      |      |          |      |      |      |      |
   v      v      v          v      v      v      v      v
[      GRAFTING      ]   [ 0x1A | 0x4F | 0xB2 | 0x00 | 0x00 ]
                                              ^      ^
                                 (Old Face)   (New Features retained)
```
// 4. Encode back to Base64 for the clipboard
return btoa(String.fromCharCode(...finalBytes));
