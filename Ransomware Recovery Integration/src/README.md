# Documentation: Vigenère Base64 Encryption System (Separate Compilation)

## Project Organization

### File Structure
```
.
├── Makefile                  # Build orchestration
├── cipher.c                  # Encryption program
├── decipher.c                # Decryption program
├── findkey.c                 # Key extraction program
├── base64_lib.c / .h         # Base64 conversion library
├── vignere_lib.c / .h        # Vigenère encryption library
├── key_lib.c / .h            # Period detection library
└── exemple.txt               # Test file
```

### Architecture

**Base64 Library** (`base64_lib`)
- `base64_index(char)`: converts a Base64 character → index (0-63)
- `base64_char(int)`: converts an index (0-63) → Base64 character

**Vigenère Library** (`vignere_lib`)
- `vigenere_base64()`: Vigenère encryption on Base64
- `vigenere_base64_decode()`: Vigenère decryption on Base64
- `decode_vigenere()`: raw key extraction by comparison

**Key Library** (`key_lib`)
- `find_period()`: detects the minimal period of a repeated key

---

## Compilation

### Method 1: Compiling the executables
```bash
make all
```

**Generates:**
- `cipher`: encryption executable
- `decipher`: decryption executable
- `findkey`: key extraction executable

### Method 2: Fully automated process
```bash
make process INPUT=fichier.txt KEY=MyBase64Key
```

**Actions performed:**
1. Compiles all programs
2. Encodes `INPUT` in Base64 → `clair64`
3. Creates a copy → `exempleres`
4. Encrypts `exempleres` with `KEY`
5. Saves the result → `chiffre64`
6. Decrypts `exempleres`
7. Extracts the key from `clair64` and `chiffre64`

**Concrete example:**
```bash
make process INPUT=exemple.txt KEY=CleSAE2025A
```

### Cleaning
```bash
make clean
```

Removes all generated files (executables, `.o` files, temporary files).

---

## Using the Programs

### 1. cipher - Encryption

**Syntax:**
```bash
./cipher <base64_key> <base64_file>
```

**Behavior:**  
Applies the Vigenère cipher to the Base64 characters of the file (ignoring `=` and `\n`), then overwrites the file with the encrypted result.

**Example:**
```bash
# Preparation: encode a text file in Base64
base64 message.txt > message_b64.txt

# Compilation
make cipher

# Encryption
./cipher "ABC123==" message_b64.txt
```

**Output:** the encrypted text in Base64

---

### 2. decipher - Decryption

**Syntax:**
```bash
./decipher <base64_key> <encrypted_file>
```

**Behavior:**  
Reverses the Vigenère encryption to recover the original Base64, then overwrites the file with the result.

**Example:**
```bash
# Compilation
make decipher

# Decryption
./decipher "ABC123==" message_b64.txt
```

**Output:** the decrypted text in Base64

---

### 3. findkey - Key Extraction

**Syntax:**
```bash
./findkey <plaintext_b64_file> <encrypted_b64_file>
```

**Behavior:**  
Compares the two files to extract the key used, detects its minimal period, and prints it.

**Outputs:**
- `stdout` → minimal key
- `stderr` → key length

**Example:**
```bash
# Compilation
make findkey

# Extraction
./findkey clair64 chiffre64
```

**Output:**
```
ABC123        (on stdout)
6             (on stderr)
```
