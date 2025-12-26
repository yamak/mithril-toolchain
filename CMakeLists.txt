cmake_minimum_required(VERSION 3.20)
project(MithrilToolchain NONE)

# Set default install prefix if not specified (use user's home directory)
if(CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT)
    set(CMAKE_INSTALL_PREFIX "$ENV{HOME}/mithril_toolchain" CACHE PATH 
        "Installation prefix for the toolchain" FORCE)
endif()

###############################
# XPAC Configuration
###############################

set(MITHRIL_ENABLE_XPAC ON CACHE BOOL "Enable XPAC (Pointer Authentication) support" FORCE)

if(MITHRIL_ENABLE_XPAC)
    message(STATUS "XPAC support: ENABLED")
else()
    message(STATUS "XPAC support: DISABLED")
endif()

###############################
# Submodule management
###############################

find_package(Git REQUIRED)

# Function to check and initialize submodule
function(ensure_submodule SUBMODULE_PATH)
    set(FULL_PATH "${CMAKE_CURRENT_SOURCE_DIR}/${SUBMODULE_PATH}")
    
    # Check if submodule is initialized
    if(NOT EXISTS "${FULL_PATH}/.git")
        message(STATUS "Initializing submodule: ${SUBMODULE_PATH}")
        execute_process(
            COMMAND ${GIT_EXECUTABLE} submodule update --init --recursive ${SUBMODULE_PATH}
            WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
            RESULT_VARIABLE GIT_RESULT
        )
        
        if(NOT GIT_RESULT EQUAL "0")
            message(FATAL_ERROR "Failed to initialize submodule ${SUBMODULE_PATH}")
        endif()
    else()
        message(STATUS "Submodule already initialized: ${SUBMODULE_PATH}")
    endif()
endfunction()

# Initialize required submodules
ensure_submodule("llvm/llvm-project")
ensure_submodule("picolibc/picolibc-src")
ensure_submodule("gdb/binutils-gdb")

###############################
# Add subdirectories
###############################

message(STATUS "Adding LLVM to build")
add_subdirectory(llvm)

message(STATUS "Adding Picolibc to build")
add_subdirectory(picolibc)

message(STATUS "Adding GDB to build")
add_subdirectory(gdb)

add_custom_target(mithril
COMMENT "Building Mithril Toolchain (LLVM distribution + Picolibc + GDB, no tests)"
DEPENDS distribution picolibc-build gdb-build
)

add_custom_target(install-mithril
COMMENT "Installing Mithril Toolchain (LLVM distribution + Picolibc + GDB)"
DEPENDS install-distribution picolibc-install gdb-install
)

add_custom_target(install-mithril-stripped
COMMENT "Installing Mithril Toolchain - Stripped (LLVM distribution stripped + Picolibc + GDB stripped)"
DEPENDS install-distribution-stripped picolibc-install gdb-install-stripped
)