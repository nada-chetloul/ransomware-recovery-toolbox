# ransomware-recovery-toolbox
This project is a cybersecurity toolbox written in C that simulates ransomware behavior in order to analyze file encryption and provide decryption and data recovery features. It focuses on file protection and restoration mechanisms.


### ⚙️ Toolbox Initialization
![Initialization](screenshot/wsl_inittoolbox.png)
![Initialization](screenshot/wsl_executable.png)
The script sets up the working environment and generates the required executables if they are missing.

### 📂 Archive Import
![Import](screenshot/wsl_import.png)
Archives are added to the toolbox so they can be analyzed and processed.

### 🔓 Archive Restoration
![Restoration](screenshot/wsl_restore.png)
The system scans the archive, identifies encrypted files, and starts the restoration process.

### 🔑 Key Recovery
![Key found](screenshot/wsl_cletrouve.png)
The key is automatically deduced from the encrypted files and stored by the system.

### 📄 Decrypted File
![Decrypted file](screenshot/wsl_distination.png)
![Decrypted file](screenshot/wsl_resultat.png)
The file's content is successfully recovered and can be read normally.
