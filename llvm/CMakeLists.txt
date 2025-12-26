cmake_minimum_required(VERSION 3.20)

###############################
#   RISC-V RV32 LLVM TOOLCHAIN
###############################

# Build type
set(CMAKE_BUILD_TYPE Release CACHE STRING "")

###############################
# LLVM / CLANG CONFIGURATION
###############################

# Only RISC-V backend
set(LLVM_TARGETS_TO_BUILD RISCV CACHE STRING "")

# Projects to build
set(LLVM_ENABLE_PROJECTS
    clang;lld
    CACHE STRING "")

# Only compiler-rt runtimes (libgcc replacement)
set(LLVM_ENABLE_RUNTIMES
    compiler-rt
    CACHE STRING "")

# Install only toolchain parts (no dev tools)
set(LLVM_INSTALL_TOOLCHAIN_ONLY ON CACHE BOOL "")

# Default linker is LLD (bare-metal)
set(CLANG_DEFAULT_LINKER lld CACHE STRING "")

# Default runtime library is compiler-rt
set(CLANG_DEFAULT_RTLIB compiler-rt CACHE STRING "")

# Default unwind library (none for baremetal RISC-V)
set(CLANG_DEFAULT_UNWINDLIB none CACHE STRING "")

# libc++ is NOT used
set(CLANG_DEFAULT_CXX_STDLIB "" CACHE STRING "")

# Do NOT build LLVM libc
set(LLVM_LIBC_ENABLE OFF CACHE BOOL "")
set(LLVM_LIBC_FULL_BUILD OFF CACHE BOOL "")
set(LLVM_DEFAULT_TARGET_TRIPLE riscv32-unknown-elf CACHE STRING "")

set(DEFAULT_SYSROOT "../${LLVM_DEFAULT_TARGET_TRIPLE}" CACHE STRING "")

set(LLVM_TOOLCHAIN_TOOLS
    llvm-ar
    llvm-ranlib
    llvm-objcopy
    llvm-objdump
    llvm-size
    llvm-readobj
    llvm-strings
    llvm-strip
    llvm-readelf
    llvm-nm
    CACHE STRING "")

set(LLVM_DISTRIBUTION_COMPONENTS
    builtins
    clang
    clang-resource-headers
    lld
    ${LLVM_TOOLCHAIN_TOOLS}
    CACHE STRING "")

###############################
# COMPILER-RT (BUILTINS ONLY)
###############################

# Only build builtins for riscv32-unknown-elf
set(LLVM_BUILTIN_TARGETS
    riscv32-unknown-elf
    CACHE STRING "")

# Target properties
set(COMPILER_RT_CMAKE_SYSTEM_NAME Generic CACHE STRING "")
set(COMPILER_RT_CMAKE_SYSTEM_PROCESSOR RISCV CACHE STRING "")

# Configure flags based on XPAC support
if(MITHRIL_ENABLE_XPAC)
    set(RV32_FLAGS "-march=rv32imac_zicsr_zifencei_xpac -mabi=ilp32 -mxpac-ret")
else()
    set(RV32_FLAGS "-march=rv32imac_zicsr_zifencei -mabi=ilp32")
endif()

set(BUILTINS_riscv32-unknown-elf_CMAKE_C_FLAGS
    "${RV32_FLAGS}" CACHE STRING "" FORCE)
set(BUILTINS_riscv32-unknown-elf_CMAKE_CXX_FLAGS
    "${RV32_FLAGS}" CACHE STRING "" FORCE)
set(BUILTINS_riscv32-unknown-elf_CMAKE_ASM_FLAGS
    "${RV32_FLAGS}" CACHE STRING "" FORCE)

# Bare-metal builtins only (no OS)
set(BUILTINS_riscv32-unknown-elf_COMPILER_RT_BAREMETAL_BUILD ON CACHE BOOL "")

# Disable unnecessary compiler-rt components for baremetal
set(COMPILER_RT_BUILD_PROFILE OFF CACHE BOOL "")
set(COMPILER_RT_BUILD_SANITIZERS OFF CACHE BOOL "")
set(COMPILER_RT_BUILD_XRAY OFF CACHE BOOL "")
set(COMPILER_RT_BUILD_LIBFUZZER OFF CACHE BOOL "")
set(COMPILER_RT_BUILD_MEMPROF OFF CACHE BOOL "")
set(COMPILER_RT_BUILD_ORC OFF CACHE BOOL "")
set(COMPILER_RT_BUILD_GWP_ASAN OFF CACHE BOOL "")


###############################
# Add LLVM project
###############################

set(LLVM_SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/llvm-project")

if(NOT EXISTS "${LLVM_SOURCE_DIR}/llvm/CMakeLists.txt")
    message(FATAL_ERROR "LLVM source not found. Please run: git submodule update --init --recursive")
endif()

message(STATUS "LLVM source directory: ${LLVM_SOURCE_DIR}")
message(STATUS "LLVM install directory: ${CMAKE_INSTALL_PREFIX}")

add_subdirectory(${LLVM_SOURCE_DIR}/llvm llvm-build)