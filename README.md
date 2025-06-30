# ioquake3 - Build Fix for Modern MinGW-w64

[中文说明](README_zh.md)

This is a fork of the original [ioquake/ioq3](https://github.com/ioquake/ioq3) project, specifically created to address and document compilation and linking issues encountered when building on a modern MSYS2/MinGW-w64 environment on Windows.

The original project's README can be found here: [README_original.md](README_original.md).

## Build Environment

This project was successfully built and tested in the following environment. The fixes should apply to most modern MinGW-w64 setups.

- **Operating System**: Windows 10 Pro (Version 22H2, OS Build 19045.5965)
- **Build Environment**: MSYS2 MINGW64
- **MSYS2 Runtime**: 3.5.7-2
- **MinGW-w64 Toolchain**:
  - **GCC**: 15.1.0
  - **Binutils (ld)**: 2.44
  - **mingw-w64-crt / headers**: 13.0.0 (git version)

## The Problem

When attempting to build the original ioquake3 source code using `make ARCH=x86_64` on a recent MSYS2/MinGW-w64 setup, a series of build errors occur. These issues stem from incompatibilities between the project's legacy code (particularly its interaction with Windows APIs) and the updated headers and libraries of the modern toolchain.

## The Solution

This fork contains the necessary modifications to enable a successful build. The fixes are surgical and aim to be as non-intrusive as possible.

### Summary of Fixes

1.  **Compiler Error: `too many arguments to function 'qSHGetFolderPath'`**
    *   **Symptom**: The build fails during the compilation of `code/sys/sys_win32.c`. The compiler complains that the dynamically loaded `qSHGetFolderPath` function pointer is being called with an incorrect number of arguments.
    *   **Analysis**: This is caused by a signature mismatch between the legacy code's expectations and the definitions in modern Windows headers (`shlobj.h`).
    *   **Fix**: The `Sys_DefaultHomePath` and `Sys_MicrosoftStorePath` functions in `code/sys/sys_win32.c` were modified to use the modern `SHGetKnownFolderPath` API. A fallback to the legacy `SHGetFolderPathA` method is preserved for backward compatibility with older systems like Windows XP.

2.  **Linker Error: `undefined reference to '__imp_CoTaskMemFree'`**
    *   **Symptom**: After fixing the compilation error, the build fails at the linking stage when creating the dedicated server executable (`ioq3ded.x86_64.exe`).
    *   **Analysis**: The new `SHGetKnownFolderPath` API requires `CoTaskMemFree` to release memory. This function resides in `ole32.lib`. The original `Makefile` did not link this library for the dedicated server build to keep it lightweight.
    *   **Fix**: The root `Makefile` was modified (line 746) to include `-lole32` in the base `LIBS` variable for all MinGW builds. This ensures that both the client and the dedicated server can access the necessary function.

## How to Build

With these fixes, you can now build the project successfully on a modern MinGW-w64 system.

1.  **Prerequisites**:
    *   MSYS2 with the MinGW-w64 toolchain installed (`mingw-w64-x86_64-toolchain`).
    *   `make` and other basic build utilities.

2.  **Build Command**:
    From the project root directory, simply run:
    ```bash
    make ARCH=x86_64
    ```

This fork serves as a practical guide for anyone facing similar legacy code vs. modern toolchain challenges.