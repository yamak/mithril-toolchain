cmake_minimum_required(VERSION 3.20)
include(ProcessorCount)
ProcessorCount(N)
if(N EQUAL 0)
    set(N 1)
endif()

###############################
# GDB Build Configuration
###############################

message(STATUS "Configuring GDB build...")

set(GDB_SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/binutils-gdb")
set(GDB_BUILD_DIR "${CMAKE_CURRENT_BINARY_DIR}/gdb-build")
set(TOOLCHAIN_BIN_DIR "${CMAKE_BINARY_DIR}/llvm/llvm-build/bin")

find_program(MAKE_EXECUTABLE NAMES make gmake REQUIRED)

# Create build directory
file(MAKE_DIRECTORY ${GDB_BUILD_DIR})

###############################
# Build GDB with Autotools
###############################

# Custom command to configure GDB
add_custom_command(
    OUTPUT ${GDB_BUILD_DIR}/Makefile
    COMMAND ${GDB_SOURCE_DIR}/configure
        --target=riscv32-unknown-elf
        --prefix=${CMAKE_INSTALL_PREFIX}
        --disable-nls
        --disable-werror
        --with-expat
        --disable-source-highlight
        --enable-tui
        --disable-docs
        --with-python=no
        MAKEINFO=true
    WORKING_DIRECTORY ${GDB_BUILD_DIR}
    COMMENT "Configuring GDB"
    VERBATIM
)

# Make picolibc depend on LLVM
if(TARGET clang)
    add_dependencies(picolibc-setup clang lld llvm-ar llvm-nm llvm-ranlib llvm-strip llvm-objcopy llvm-objdump)
endif()

# Target to drive the configure step
add_custom_target(gdb-configure DEPENDS ${GDB_BUILD_DIR}/Makefile)

# Custom target to build GDB
add_custom_target(gdb-build
    COMMAND ${MAKE_EXECUTABLE} -j${N}
    DEPENDS ${GDB_BUILD_DIR}/Makefile
    WORKING_DIRECTORY ${GDB_BUILD_DIR}
    COMMENT "Building GDB"
    VERBATIM
)

# Custom target to install GDB (copy binary only)
add_custom_target(gdb-install
    COMMAND ${CMAKE_COMMAND} -E make_directory ${CMAKE_INSTALL_PREFIX}/bin
    COMMAND ${CMAKE_COMMAND} -E copy ${GDB_BUILD_DIR}/gdb/gdb ${CMAKE_INSTALL_PREFIX}/bin/riscv32-unknown-elf-gdb
    DEPENDS gdb-build
    COMMENT "Installing GDB binary to ${CMAKE_INSTALL_PREFIX}/bin"
    VERBATIM
)

# Custom target to install stripped GDB (copy and strip)
add_custom_target(gdb-install-stripped
    COMMAND ${CMAKE_COMMAND} -E make_directory ${CMAKE_INSTALL_PREFIX}/bin
    COMMAND ${CMAKE_COMMAND} -E copy ${GDB_BUILD_DIR}/gdb/gdb ${CMAKE_INSTALL_PREFIX}/bin/riscv32-unknown-elf-gdb
    COMMAND ${TOOLCHAIN_BIN_DIR}/llvm-strip ${CMAKE_INSTALL_PREFIX}/bin/riscv32-unknown-elf-gdb
    DEPENDS gdb-build
    COMMENT "Installing stripped GDB binary to ${CMAKE_INSTALL_PREFIX}/bin"
    VERBATIM
)
