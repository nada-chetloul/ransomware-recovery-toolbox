# Bash Scripts Documentation – SAE Security

This documentation explains how each script developed for the SAE works, how to use it, and what role it plays. It also describes the return codes, the messages displayed, and the required prerequisites.

## 1. init-toolbox.sh

### Purpose of the script
This script initializes the tool's working environment by automatically creating:
- a `.sh-toolbox` folder (if it does not exist),
- an `archives` file containing the number of imported archives.

### How it works
1. Checks whether the `.sh-toolbox` folder exists.
2. Creates it if it is missing.
3. Checks whether the `archives` file exists.
4. Creates it if it is missing and writes 0 in it.
5. Checks that no other file or folder is present in `.sh-toolbox`.

### Return codes
- **0**: Correct initialization when the folder and file already exist.
- **1**: Unable to create the folder or file.
- **2**: Unauthorized items present in `.sh-toolbox`.

### Example
```bash
./init-toolbox.sh
```

### Possible output:
```text
creation du dossier
le dossier a etais cree
le fichier a etais cree
le dossier et le fichier existe
ERREUR:il ya un ou  plusieur (fichier/dossier) en plus de archives dans votre
dossier
```

## 2. import-archive.sh

### Purpose
Import a `.tar.gz` archive into the working environment. This script:
- copies the archive into `.sh-toolbox`,
- asks for confirmation if it already exists (unless in `-f` mode),
- updates the `archives` file.

### How it works
1. Checks that `.sh-toolbox` exists.
2. Checks that the archive passed as a parameter exists.
3. Copies the archive into `.sh-toolbox`.
4. If it already exists: asks the user for confirmation unless `-f` is used.
5. Updates:
    1. the first line of the `archives` file (total count),
    2. a new descriptive line: `archive_name:date_added:key`

### Available modes
- **Normal**: confirmation required to overwrite.
- **Forced (-f)**: automatic overwrite.
- **Multi-import**: several archives can be added in a single call.

### Example
```bash
./import-archive.sh client1.tar.gz
./import-archive.sh -f client1.tar.gz
./import-archive.sh client1.tar.gz client2.tar.gz
```

### Return codes
- **0**: Copy completed or copy cancelled.
- **1**: `.sh-toolbox` does not exist.
- **2**: Archive not found.

> **Note (bonus)**: In the bonus version, the script can receive several parameters. If one of the given files does not exist, the script displays an error message, then uses `continue` instead of `exit 2` in order to keep checking the other parameters.

- **3**: Error during copy.
- **4**: Error updating the `archives` file.

## 3. ls-toolbox.sh

### Purpose
Display the list of archives recorded in the "archives" file.

### How it works
1. Checks that `.sh-toolbox` and the `archives` file are present.
2. Reads all lines (except the first) of the `archives` file.
3. Displays for each archive: `Name – date added – key known or unknown`

### Example
```bash
./ls-toolbox.sh
```

### Possible output:
```text
Nom : client1.tar.gz date : 20251104-131504 clé : inconnue
```

### Return codes
- **0**: Display successful.
- **1**: `.sh-toolbox` missing.
- **2**: `archives` file missing.
- **3 (bonus)**: inconsistencies found.

## 4. restore-toolbox.sh (bonus)

### Purpose
Automatically repair the `.sh-toolbox` environment in case of error or inconsistency.

### Detected problems
- `.sh-toolbox` folder missing.
- `archives` file missing.
- Archive listed but not physically present.
- Archive physically present but not listed in `archives`.

### How it works
For each anomaly found, the script asks for confirmation before acting:
- creating the folder,
- creating the file,
- deleting an invalid line,
- adding a missing archive.

### Example
```bash
./restore-toolbox.sh
```

## 5. check-archive.sh

### Purpose
Analyze an archive in order to identify:
- the admin user's last SSH login,
- the list of files modified after that date,
- (bonus) files that were not modified but have the same name and size.

### How it works
1. Checks `.sh-toolbox` and `archives`.
2. Displays the list of archives and offers a choice.
3. Decompresses into a temporary folder.
4. Looks for the last admin login in: `var/log/auth.log`
5. Scans `data/` to identify:
    - files modified after the attack,
    - unmodified files with identical name and size (bonus).
    - The file must be read-only and must not belong to the user (bonus)

### Example
```bash
./check-archive.sh
```

### Possible output:
```text
la liste des fichier est :
arch1
arch1
client1-20250901-1503.tar.gz
quel archives voulez vous utiliser?( de 1 a n)
3
archive selectionnée : client1-20250901-1503.tar.gz
......
Dernière connexion admin : Jul 15 15:26:24
Fichiers modifiés après la dernière connexion admin :
.......
```

### Return codes
- **0**: everything went well.
- **1**: `.sh-toolbox` missing.
- **2**: `archives` file missing.
- **3**: decompression failed.
- **4**: log file missing.
- **5**: `data/` folder empty.
