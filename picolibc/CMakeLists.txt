cmake_minimum_required(VERSION 3.20)

###############################
# Picolibc Build Configuration
###############################

message(STATUS "Configuring Picolibc build...")

find_program(MESON_EXECUTABLE meson)
find_program(NINJA_EXECUTABLE ninja)

if(NOT MESON_EXECUTABLE)
    message(FATAL_ERROR "Meson not found. Please install meson: pip install meson")
endif()

set(PICOLIBC_SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/picolibc-src")
set(PICOLIBC_BUILD_DIR "${CMAKE_CURRENT_BINARY_DIR}/picolibc-build")
set(PICOLIBC_CROSS_FILE "${CMAKE_CURRENT_BINARY_DIR}/riscv32-clang.txt")


###############################
# Generate Meson cross-file
###############################

# Compiler paths from built LLVM toolchain
set(TOOLCHAIN_BIN_DIR "${CMAKE_BINARY_DIR}/llvm/llvm-build/bin")

# Create cross-file content
file(WRITE ${PICOLIBC_CROSS_FILE} "[binaries]\n")
file(APPEND ${PICOLIBC_CROSS_FILE} "c = ['${TOOLCHAIN_BIN_DIR}/clang', '--target=riscv32-unknown-elf', '-march=rv32imc_xpac', '-mabi=ilp32', '-fuse-ld=ld.lld', '-fno-pic', '-mxpac-ret']\n")
file(APPEND ${PICOLIBC_CROSS_FILE} "ar = '${TOOLCHAIN_BIN_DIR}/llvm-ar'\n")
file(APPEND ${PICOLIBC_CROSS_FILE} "nm = '${TOOLCHAIN_BIN_DIR}/llvm-nm'\n")
file(APPEND ${PICOLIBC_CROSS_FILE} "ranlib = '${TOOLCHAIN_BIN_DIR}/llvm-ranlib'\n")
file(APPEND ${PICOLIBC_CROSS_FILE} "strip = '${TOOLCHAIN_BIN_DIR}/llvm-strip'\n")
file(APPEND ${PICOLIBC_CROSS_FILE} "objcopy = '${TOOLCHAIN_BIN_DIR}/llvm-objcopy'\n")
file(APPEND ${PICOLIBC_CROSS_FILE} "objdump = '${TOOLCHAIN_BIN_DIR}/llvm-objdump'\n")
file(APPEND ${PICOLIBC_CROSS_FILE} "ld = '${TOOLCHAIN_BIN_DIR}/ld.lld'\n")
file(APPEND ${PICOLIBC_CROSS_FILE} "c_ld = '${TOOLCHAIN_BIN_DIR}/ld.lld'\n")
file(APPEND ${PICOLIBC_CROSS_FILE} "cpp_ld = '${TOOLCHAIN_BIN_DIR}/ld.lld'\n")
file(APPEND ${PICOLIBC_CROSS_FILE} "\n")
file(APPEND ${PICOLIBC_CROSS_FILE} "[host_machine]\n")
file(APPEND ${PICOLIBC_CROSS_FILE} "system = 'none'\n")
file(APPEND ${PICOLIBC_CROSS_FILE} "cpu_family = 'riscv32'\n")
file(APPEND ${PICOLIBC_CROSS_FILE} "cpu = 'rv32imc'\n")
file(APPEND ${PICOLIBC_CROSS_FILE} "endian = 'little'\n")
file(APPEND ${PICOLIBC_CROSS_FILE} "\n")
file(APPEND ${PICOLIBC_CROSS_FILE} "[properties]\n")
file(APPEND ${PICOLIBC_CROSS_FILE} "skip_sanity_check = true\n")
file(APPEND ${PICOLIBC_CROSS_FILE} "\n")
file(APPEND ${PICOLIBC_CROSS_FILE} "[built-in options]\n")
file(APPEND ${PICOLIBC_CROSS_FILE} "c_link_args = ['-fuse-ld=ld.lld']\n")

###############################
# Build Picolibc with Meson
###############################

# Custom target to setup meson
add_custom_target(picolibc-setup
    COMMAND ${MESON_EXECUTABLE} setup ${PICOLIBC_BUILD_DIR}
        --cross-file=${PICOLIBC_CROSS_FILE}
        --prefix=${CMAKE_INSTALL_PREFIX}/riscv32-unknown-elf
        -Dtests=false
        -Dpicocrt=false
        -Dmultilib=false
        -Dthread-local-storage=false
        -Dspecsdir=lib/clang
    WORKING_DIRECTORY ${PICOLIBC_SOURCE_DIR}
    COMMENT "Setting up Picolibc build with Meson"
    VERBATIM
)

# Make picolibc depend on LLVM
if(TARGET clang)
    add_dependencies(picolibc-setup clang lld llvm-ar llvm-nm llvm-ranlib llvm-strip llvm-objcopy llvm-objdump)
endif()

# Custom target to build picolibc
add_custom_target(picolibc-build
    COMMAND ${NINJA_EXECUTABLE} -C ${PICOLIBC_BUILD_DIR}
    DEPENDS picolibc-setup
    COMMENT "Building Picolibc with Ninja"
    VERBATIM
)

# Custom target to install picolibc
add_custom_target(picolibc-install
    COMMAND ${NINJA_EXECUTABLE} -C ${PICOLIBC_BUILD_DIR} install
    DEPENDS picolibc-build
    COMMENT "Installing Picolibc to ${CMAKE_INSTALL_PREFIX}/riscv32-unknown-elf"
    VERBATIM
)

