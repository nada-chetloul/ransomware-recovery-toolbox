# Integration Documentation (Bonus)

## Compiling the Binaries

Before using the restoration scripts, the C programs must be compiled:
```bash
make -C src
mv src/decipher src/findkey .
```

---

## restore-archive.sh Script

### Overview
This script uses the variables exported by `check-archive.sh` to restore the encrypted files of an archive.

### Integration with check-archive.sh
The script sources `check-archive.sh` to automatically retrieve:
- `$ARCHIVE_SELECTED`: name of the selected archive
- `$ATTACK_TS`: attack timestamp (in seconds)
- `$DATA_DIR`: path to the decompressed data folder
- `$MODIFIED_FILES`: temporary file listing the modified files
- `$TMP_DIR`: temporary extraction folder

### Calls to Existing Scripts
```bash
./init-toolbox.sh      # Checks/compiles the binaries
./restore-toolbox.sh   # Checks archive consistency
```

### Restoration Process

#### 1. Detection and Key Search
For each encrypted file (listed in `$MODIFIED_FILES`):
- Look for the matching plaintext file (same name, modified before the attack)
- Call `./findkey` with the base64 versions of both files
- Temporarily store the key in `/tmp/key_restore`

#### 2. Key Type Handling
The key is analyzed to determine its type:

**Printable key** (displayable characters only):
- Test: `[[ $key =~ ^[[:print:]]+$ ]]`
- Storage: directly in the `archives` file (column 3)
- Format: `archive.tar.gz:date:KEY:s`

**Non-printable key** (contains binary characters):
- Creation of a folder: `.sh-toolbox/archive_name/` (without .tar.gz)
- Storage: in the `KEY` file of that folder
- Format: `archive.tar.gz:date::f`

#### 3. Updating the archives File
Uses `sed` with regular expressions to modify the corresponding line:
```bash
safe_archive=$(echo "$archive_select" | sed 's/[][\.*^$/]/\\&/g')
sed -Ei "2,\$ s|^($safe_archive:[^:]*:).*|\1$key:s|" "$wd/archives"
```

#### 4. Decrypting the Files
- Retrieve the key according to its type (s or f)
- For each file in `$MODIFIED_FILES`:
  - Compute the relative path from `$TMP_DIR`
  - Ask for confirmation if the file already exists
  - Call `./decipher KEY base64_file`
  - Extract the decrypted content via `awk`
  - Write it to the destination folder

### Return Codes
- `0`: restoration successful
- `1`: .sh-toolbox folder does not exist
- `2`: failed to create destination folder
- `3`: failed to update the archives file
- `4`: failed to restore a file

---

## init-toolbox.sh Improvements

### Automatic Binary Compilation

Added a check-and-compile loop for `decipher` and `findkey`.

### New Return Codes
- `10`: source file (.c) missing
- `11`: GCC compiler not available
- `12`: compilation failed

### How It Works
1. Checks whether the binary exists and is executable (`-x`)
2. If missing, looks for the source file in `./src/`
3. Checks that `gcc` is available
4. Automatically compiles the program
5. Displays a confirmation message

---

## archives File Structure (Extended Format)
```
number_of_archives
archive1.tar.gz:import_date:key_or_empty:type
archive2.tar.gz:import_date:key_or_empty:type
```

**Column 4 (type)**:
- `s`: key stored in the archives file (string)
- `f`: key stored in an external file (file)

### Examples
```
2
client1-20250411-1311.tar.gz:20251104-131504:CleSAE2025:s
client2-20250411-1341.tar.gz:20251104-134407::f
```

---

## Full Project Tree
```
.
├── .sh-toolbox/
│   ├── archives                           # Index file
│   ├── client1-20250411-1311.tar.gz      # Archive 1
│   ├── client2-20250411-1341.tar.gz      # Archive 2
│   └── client2-20250411-1341/            # Key folder
│       └── KEY                            # Non-printable key
├── src/
│   ├── Makefile                           # Makefile
│   ├── base64_lib.c
│   ├── decipher.c
│   ├── findkey.c
│   ├── key_lib.c
│   ├── key_lib.h
│   ├── vignere_lib.c
│   └── vignere_lib.h
├── check-archive.sh
├── import-archive.sh
├── init-toolbox.sh
├── ls-toolbox.sh
├── restore-toolbox.sh
├── restore-archive.sh
├── decipher                               # Compiled binary
└── findkey                                # Compiled binary
```

---

## Data Flow Between Scripts
```
restore-archive.sh
    ↓ sources
check-archive.sh
    → exports: ARCHIVE_SELECTED, ATTACK_TS, DATA_DIR, MODIFIED_FILES, TMP_DIR
    ↓
restore-archive.sh
    ↓ calls
init-toolbox.sh (compiles decipher + findkey)
    ↓ calls
restore-toolbox.sh (checks consistency)
    ↓
Processing of encrypted files
    → ./findkey → key type detection → archives update
    → ./decipher → file restoration
```
