# Mithril Toolchain

RISC-V 32-bit (rv32imc) Bare-Metal Toolchain (LLVM + Picolibc + GDB)

## Overview

This project builds a minimal bare-metal toolchain for RISC-V 32-bit targets:

- **LLVM/Clang**: Compiler and linker (LLD)
- **GDB**: Debugger 
- **Picolibc**: Minimal C library (optimized for bare-metal)
- **Compiler-RT**: LLVM builtins (replacement for libgcc)

## Documentation

- [Mithril PAC Specification](docs/mithril_pac.md) — Hardware-accelerated CFI architecture

## Requirements

- CMake 3.20+
- Git
- Ninja
- Python 3.6+
- Meson (`pip install meson`)
- Texinfo (for GDB build)
- C/C++ compiler (for building LLVM itself)

For Ubuntu/Debian:
```bash
sudo apt install cmake ninja-build python3 python3-pip build-essential git texinfo
pip install meson
```

## Usage

### 1. Configuration

```bash
# Clean build directory
rm -rf build && mkdir build

# Configure project (uses default: $HOME/mithril_toolchain)
cmake -B build -G Ninja

# Or specify a custom installation directory
cmake -B build -G Ninja -DCMAKE_INSTALL_PREFIX=/path/to/install
```

**Options:**

- `CMAKE_INSTALL_PREFIX`: Installation directory for the toolchain (default: `$HOME/mithril_toolchain`)

### 2. Build

The build system handles dependencies automatically. Picolibc will use the Clang binary from the LLVM build directory without requiring an installation step first.

```bash
# Build everything (LLVM distribution + Picolibc + GDB)
cmake --build build --target mithril
```

### 3. Install

Install the complete toolchain:

```bash
# Install normal version
cmake --build build --target install-mithril

# Install stripped version (recommended for production)
# This strips both LLVM binaries and GDB
cmake --build build --target install-mithril-stripped
```

**Note:** The stripped version removes debug symbols to reduce installation size. For development, you may prefer the normal version.

### 4. Usage

After the toolchain is installed, you can use it from the installation directory.

**Example compilation:**

```bash
# If installed to default location ($HOME/mithril_toolchain)
export PATH=$HOME/mithril_toolchain/bin:$PATH
export SYSROOT=$HOME/mithril_toolchain/riscv32-unknown-elf

# Compile a program
clang --target=riscv32-unknown-elf -march=rv32imc -mabi=ilp32 \
    --sysroot=$SYSROOT \
    -o program program.c

# Or use from build directory (before installation)
export LLVM_BIN=/path/to/mithril-toolchain/build/llvm/llvm-build/bin
export SYSROOT=/path/to/mithril-toolchain/build/picolibc/picolibc-build/install

$LLVM_BIN/clang --target=riscv32-unknown-elf -march=rv32imc -mabi=ilp32 \
    --sysroot=$SYSROOT \
    -o program program.c
```

## Architecture Details

### LLVM Configuration

- **Target**: RISC-V 32-bit (`riscv32-unknown-elf`)
- **Architecture**: rv32imac + zicsr + zifencei
- **ABI**: ilp32
- **Backend**: RISC-V only
- **Projects**: Clang, LLD
- **Runtime**: Compiler-RT (builtins only)
- **Linker**: LLD (default)
- **Distribution Components**: Only essential toolchain components (clang, lld, builtins, toolchain tools)

### GDB Configuration

- **Target**: riscv32-unknown-elf
- **TUI**: Enabled
- **Expat**: Enabled
- **Python**: Disabled
- **Docs/NLS**: Disabled
- **Install**: Minimal (only `bin/riscv32-unknown-elf-gdb` is installed, no shared data)

### Picolibc Configuration

- **Target**: riscv32-unknown-elf
- **Tests**: Disabled
- **Picocrt**: Disabled
- **Multilib**: Disabled
- **TLS**: Disabled

## Directory Structure

```
mithril-toolchain/
├── CMakeLists.txt              # Main CMake file
├── llvm/
│   ├── CMakeLists.txt          # LLVM build configuration
│   └── llvm-project/           # LLVM submodule (git submodule)
├── gdb/
│   ├── CMakeLists.txt          # GDB build configuration
│   └── binutils-gdb/           # GDB submodule (git submodule)
└── picolibc/
    ├── CMakeLists.txt          # Picolibc build wrapper (CMake -> Meson)
    └── picolibc-src/           # Picolibc submodule (git submodule)
```

## Notes

- The build process configures Picolibc to use the LLVM toolchain from the build directory (`build/llvm/llvm-build/bin`).
- Submodules are automatically initialized.
- The build process can take quite a while (especially LLVM: 1-2 hours).
- The toolchain uses `LLVM_DISTRIBUTION_COMPONENTS` to build only essential components, reducing build time and installation size.
- Default installation directory is `$HOME/mithril_toolchain` (no root privileges required).

## License

This build script is under the MIT license. LLVM and Picolibc are subject to their own licenses.
