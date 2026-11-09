# Documentation: cipher / decipher / findkey


## Overview
This project implements an encryption system based on:
1. Base64
2. The Vigenère cipher, applied to Base64 indices
3. Preservation of Base64 padding (`=`) and line breaks

The tools are:
* cipher: applies Vigenère (encryption) to Base64 data
* decipher: applies inverse Vigenère to Base64 data
* findkey: automatically recovers the key used, in Base64

---

## 1. cipher — Encryption
🔹 **Purpose**  
Takes a file that is already Base64-encoded and applies Vigenère to each Base64 character. The key passed as an argument is also assumed to be in Base64, but `=` characters are ignored.

### Key Preparation
Before using `cipher`, the plaintext key must be converted to Base64:
```bash
echo -n "my plain key" | base64
```

### How the Code Works

#### a. Reading the file
* The file is opened and its entire content is read into a dynamic buffer.
* A `\0` is appended to make it a C string.

#### b. Cleaning the key
```c
if (argv[1][i] != '=') key[j++] = argv[1][i];
```
`=` characters in the key are removed.

#### c. Base64 conversion
Two essential functions:
* `base64_index(c)` → converts a Base64 character → 0..63
* `base64_char(idx)` → converts 0..63 → Base64 character

#### d. Vigenère on Base64
For each character:
```c
idx_c = (idx_d + idx_k) % 64;
```

If the character is `=` or `\n`: no encryption, and the key is NOT advanced.

#### e. Writing
The original file is overwritten with the encrypted version.

### Using the Result
The output of `cipher` is encrypted Base64. To get the raw binary:
```bash
cat cipher_result | base64 -d
```

---

## 2. decipher — Decryption
🔹 **Purpose**  
Works like cipher, but performs the inverse operation:
```c
idx_p = (idx_d - idx_k + 64) % 64;
```

### How the Code Works

#### a. Reading the whole file (like cipher)

#### b. Cleaning the key (removing `=`)

#### c. Decryption
* For each valid Base64 character → apply inverse Vigenère
```c
int idx_p = (idx_d - idx_k + 64) % 64;
```
* `=` and `\n` are never modified, and the key does not advance on them.

The result (still Base64) is written to the same file.

### Recovering the Plaintext File
After `decipher`, the result is decrypted Base64. To get the original file:
```bash
cat decipher_result | base64 -d
```

For binary files (images, etc.), redirect to a new file:
```bash
base64 -d decipher_result.png > final_result.png
```

---

## 3. findkey — Recovering the Key
🔹 **Purpose**  
From:
* a plaintext Base64 file
* an encrypted Base64 file

recover:
* the raw key (a repetition of the real key)
* the minimal period of the key
* the real key

### How the Code Works

#### a. Reading both Base64 files
* Both files are read entirely.
* They are compared character by character.

#### b. Raw extraction of key characters
For each position i:

If clair64[i] and chiff64[i] are valid:
```c
idx_k = (idx_x - idx_c + 64) % 64;
key_raw[kr++] = base64_char(idx_k);
```

Result: `key_raw = KKKKKKKKKKKK...` (the repeated key)

#### c. Detecting the real period
`find_period()` looks for the smallest length p such that:
```
key_raw[i] == key_raw[i % p] for all i
```

Example:
```
key_raw = ABCABCABCABC → period = 3 → key = "ABC"
```

#### d. Outputs:
* stdout → the real key
* stderr → the key length

---

## Data Flow Summary

### 🔶 Full Encryption
```
[Plain key] → base64 → [Base64 key]
[Binary file] → base64 → [Plain Base64] → cipher → [Encrypted Base64] → base64 -d → [Encrypted binary]
```

### 🔶 Full Decryption
```
[Encrypted Base64] → decipher → [Plain Base64] → base64 -d → [Original file]
```

### 🔶 Key Search
```
[clair64] + [chiffre64] → findkey → [Base64 key] → period → [final key]
```
