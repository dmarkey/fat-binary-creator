# 📦 Fat Binary Creator

`fat_binary_creator` is a utility for packaging a dynamically linked Linux executable and its dependencies into a single, self-extracting, portable binary. This facilitates running binaries compiled for one architecture (e.g., amd64) on a different architecture (e.g., arm64) using an emulator such as QEMU.

## 📜 Project Background

This project originated from a practical need: to run amd64 binary printer filters on an arm64 Raspberry Pi. Conventional methods, such as manual dependency management, chroot environments, or containers, proved cumbersome for this specific use case. `fat_binary_creator` was developed to simplify this process.

The tool automates the identification and bundling of all necessary shared libraries, including the dynamic linker/loader. The resulting "fat binary" can then be transferred and executed on a target system, provided QEMU (or a similar emulator) is available for cross-architecture execution.

## ⚙️ Operational Overview

The script executes the following sequence of operations:

1.  **Temporary Directory Creation:** Establishes a staging area for the binary and its dependencies.
2.  **Binary Copying:** Copies the target executable to the staging area.
3.  **Dependency Analysis:** Employs `ldd` to identify all shared libraries required by the binary.
4.  **Library Copying:** Recursively copies all identified libraries into the package structure.
5.  **Loader/Interpreter Copying:** Identifies and includes the appropriate dynamic linker (e.g., `ld-linux-x86-64.so.2`).
6.  **Archive Creation:** Consolidates the binary, libraries, and loader into a tar archive.
7.  **Self-Extracting Executable Generation:** Prepends a shell script to the archive. This script manages:
    *   Extraction of contents to a temporary location (cached for subsequent executions to improve performance 🚀).
    *   Execution of the packaged binary using the bundled loader and libraries, leveraging QEMU for cross-architecture scenarios.
8.  **Cleanup:** Removes the initial temporary packaging directory.

The self-extracting executable is designed for efficient subsequent runs by utilizing cached extracted files.

##  উদাহরণ (Example): Packaging `psql` 🐘

The following demonstrates packaging the `psql` (PostgreSQL client) utility on an x86_64 system:

```bash
$ ./fat_binary_creator /usr/bin/psql
Creating temporary directory structure...
Copying binary /usr/bin/psql to package...
Analyzing dependencies for /usr/bin/psql...
Copying library /usr/lib/libc.so.6
Copying library /usr/lib/libcom_err.so.2
# ... (additional libraries copied) ...
Copying library /usr/lib64/ld-linux-x86-64.so.2
Found loader/interpreter: /lib64/ld-linux-x86-64.so.2
Copying loader /lib64/ld-linux-x86-64.so.2 to package...
Bundled loader basename will be: ld-linux-x86-64.so.2
Creating archive of binaries and libraries...
Archive hash for caching: 329d3964c818e3bac3a87bb416e160045b5f93b3176bc7f3f022bf8e31ab1a5c
Creating self-extracting executable...
Cleaning up temporary packaging directory: /tmp/tmp.r6W5IvyFAw...

Done! Self-contained executable has been created as psql_standalone
It will cache extracted files and run silently (errors will still be shown).
To use, simply run: ./psql_standalone
```

### Execution on a Raspberry Pi (arm64) 🍓

After transferring `psql_standalone` to an arm64 Raspberry Pi (with QEMU user-mode emulation installed, e.g., `qemu-user-static` and `binfmt-support`):

```bash
dmarkey@raspberrypi:~$ ./psql_standalone --version
psql (PostgreSQL) 17.4
```

## 🔧 Target System Prerequisites (for cross-architecture execution)

*   **QEMU user-mode emulation:** (e.g., `qemu-user-static`).
*   **binfmt_misc kernel support:** To enable automatic invocation of QEMU for foreign architecture binaries.

Installation of `qemu-user-static` and `binfmt-support` (or distribution-specific equivalents) is generally sufficient.

## 💡 Potential Enhancements

*   Option to specify the QEMU emulator path.
*   Improved error handling and reporting.
*   Configurable cache directory location.
*   Exploration of alternative packaging formats.

## ⚠️ Disclaimer

This tool interacts with system-level executables and libraries. Users should exercise caution and understand the potential implications of its use. It is primarily intended for specific use cases as described.