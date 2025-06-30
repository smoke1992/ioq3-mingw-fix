# ioquake3 - 针对现代 MinGW-w64 环境的构建修复

[English Version](README.md)

这是一个原始 [ioquake/ioq3](https://github.com/ioquake/ioq3) 项目的分支（fork），旨在解决和记录在现代 Windows MSYS2/MinGW-w64 环境下编译时遇到的问题。

原始项目的 README 文件可以在这里找到：[README_original.md](README_original.md)。

## 构建环境

本项目在以下环境中成功构建和测试。这些修复方案应适用于大多数现代 MinGW-w64 配置。

- **操作系统**: Windows 10 专业版 (版本 22H2, 操作系统内部版本 19045.5965)
- **构建环境**: MSYS2 MINGW64
- **MSYS2 运行时**: 3.5.7-2
- **MinGW-w64 工具链**:
  - **GCC**: 15.1.0
  - **Binutils (ld)**: 2.44
  - **mingw-w64-crt / headers**: 13.0.0 (git 版本)

## 问题描述

当尝试在最新的 MSYS2/MinGW-w64 环境中使用 `make ARCH=x86_64` 命令构建 ioquake3 原始代码时，会遇到一系列的构建错误。这些问题源于项目的旧代码（特别是与 Windows API 交互的部分）与现代工具链的头文件和库之间存在不兼容性。

## 解决方案

这个分支包含了使项目能够成功构建的所有必要修改。这些修复是“外科手术式”的，旨在尽可能不影响原有代码结构。

### 修复摘要

1.  **编译错误: `too many arguments to function 'qSHGetFolderPath'` (函数 'qSHGetFolderPath' 的参数过多)**
    *   **症状**: 构建在编译 `code/sys/sys_win32.c` 文件时失败。编译器抱怨动态加载的函数指针 `qSHGetFolderPath` 被调用时传递了错误的参数数量。
    *   **分析**: 这是由于旧代码的函数签名预期与现代 Windows 头文件（`shlobj.h`）中的定义不匹配造成的。
    *   **修复**: 修改了 `code/sys/sys_win32.c` 中的 `Sys_DefaultHomePath` 和 `Sys_MicrosoftStorePath` 函数，使其优先使用现代的 `SHGetKnownFolderPath` API。同时，为了向后兼容旧系统（如 Windows XP），保留了回退到旧 `SHGetFolderPathA` 方法的逻辑。

2.  **链接错误: `undefined reference to '__imp_CoTaskMemFree'` (未定义的引用 `__imp_CoTaskMemFree`)**
    *   **症状**: 解决了编译错误后，构建在链接阶段、生成专用服务器可执行文件（`ioq3ded.x86_64.exe`）时失败。
    *   **分析**: 新的 `SHGetKnownFolderPath` API 需要调用 `CoTaskMemFree` 来释放内存。这个函数位于 `ole32.lib` 库中。原始的 `Makefile` 为了保持专用服务器的轻量化，没有为其链接该库。
    *   **修复**: 修改了根目录的 `Makefile` 文件（746 行），在为 MinGW 平台定义的基础 `LIBS` 变量中追加了 `-lole32`。这确保了客户端和专用服务器都能访问到所需的函数。

## 如何构建

应用这些修复后，你现在可以在现代 MinGW-w64 系统上成功构建本项目。

1.  **先决条件**:
    *   安装了 MSYS2 和 MinGW-w64 工具链 (`mingw-w64-x86_64-toolchain`)。
    *   `make` 和其他基础构建工具。

2.  **构建命令**:
    在项目根目录下，直接运行：
    ```bash
    make ARCH=x86_64
    ```

这个分支可以为所有面临类似“旧代码 vs 新工具链”挑战的开发者提供一份实用的参考。