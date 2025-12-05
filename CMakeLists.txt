cmake_minimum_required(VERSION 3.14)
project(LibRift 
    VERSION 1.0.0 
    DESCRIPTION "High-Performance Regular Expression Engine" 
    LANGUAGES C
)

# Configuration options
option(LIBRIFT_BUILD_TESTS "Build test suite" ON)
option(LIBRIFT_BUILD_EXAMPLES "Build examples" OFF)
option(LIBRIFT_BUILD_DOCS "Build documentation" OFF)
option(LIBRIFT_ENABLE_OPTIMIZATIONS "Enable performance optimizations" ON)
option(LIBRIFT_USE_MEMORY_POOL "Use custom memory pool" ON)
option(LIBRIFT_ENABLE_REGEX_SYNTAX "Enable r'' regex syntax" ON)
option(LIBRIFT_ENABLE_THREAD_SAFETY "Enable thread-safe operations" ON)

# C Standard settings
set(CMAKE_C_STANDARD 11)
set(CMAKE_C_STANDARD_REQUIRED ON)
set(CMAKE_C_EXTENSIONS OFF)

# Implementation constants
set(LIBRIFT_MAX_STATES 1024 CACHE STRING "Maximum number of automaton states")
set(LIBRIFT_MAX_TRANSITIONS 5000 CACHE STRING "Maximum number of automaton transitions")
set(LIBRIFT_MAX_BACKTRACKING_DEPTH 1000 CACHE STRING "Maximum backtracking depth")

# Compiler flags
if(CMAKE_C_COMPILER_ID MATCHES "GNU|Clang")
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wall -Wextra -Wpedantic")
    if(CMAKE_BUILD_TYPE STREQUAL "Debug")
        set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -g -O0")
        add_definitions(-DLIBRIFT_DEBUG_MODE)
    endif()
    if(LIBRIFT_ENABLE_OPTIMIZATIONS AND CMAKE_BUILD_TYPE STREQUAL "Release")
        set(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} -O3")
    endif()
elseif(MSVC)
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} /W4")
    if(CMAKE_BUILD_TYPE STREQUAL "Debug")
        set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} /Od /Zi")
        add_definitions(-DLIBRIFT_DEBUG_MODE)
    endif()
    if(LIBRIFT_ENABLE_OPTIMIZATIONS AND CMAKE_BUILD_TYPE STREQUAL "Release")
        set(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} /O2")
    endif()
endif()

# Platform detection and configuration
include(CheckIncludeFile)
check_include_file(stdint.h HAVE_STDINT_H)
check_include_file(stdbool.h HAVE_STDBOOL_H)
check_include_file(regex.h HAVE_REGEX_H)

# Define include paths
set(LIBRIFT_INCLUDE_DIRS 
    ${PROJECT_SOURCE_DIR}/include
    ${PROJECT_BINARY_DIR}/include
)

# Create necessary output directories
file(MAKE_DIRECTORY 
    ${PROJECT_BINARY_DIR}/include/librift
)

# Configure config.h from template if it exists
if(EXISTS "${PROJECT_SOURCE_DIR}/config.h.in")
    configure_file(
        "${PROJECT_SOURCE_DIR}/config.h.in"
        "${PROJECT_BINARY_DIR}/include/librift/config.h"
        @ONLY
    )
endif()

# Add source directory
add_subdirectory(src)

# Testing setup
if(LIBRIFT_BUILD_TESTS)
    enable_testing()
    
    # Create cmake module path for custom modules like FindCheck.cmake
    set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${PROJECT_SOURCE_DIR}/cmake/modules)
    
    # Check testing framework setup - use a custom finder module
    file(MAKE_DIRECTORY ${PROJECT_SOURCE_DIR}/cmake/modules)
    
    # Create a minimal FindCheck.cmake if it doesn't exist
    if(NOT EXISTS "${PROJECT_SOURCE_DIR}/cmake/modules/FindCheck.cmake")
        file(WRITE "${PROJECT_SOURCE_DIR}/cmake/modules/FindCheck.cmake"
"# FindCheck.cmake - Find the Check unit testing framework
#
# This module defines:
#  CHECK_INCLUDE_DIRS - where to find check.h
#  CHECK_LIBRARIES - the libraries to link against
#  CHECK_FOUND - if the library was found

find_path(CHECK_INCLUDE_DIR NAMES check.h)
find_library(CHECK_LIBRARY NAMES check)

include(FindPackageHandleStandardArgs)
find_package_handle_standard_args(Check DEFAULT_MSG
                                  CHECK_LIBRARY CHECK_INCLUDE_DIR)

if(CHECK_FOUND)
  set(CHECK_LIBRARIES ${CHECK_LIBRARY})
  set(CHECK_INCLUDE_DIRS ${CHECK_INCLUDE_DIR})
endif()

mark_as_advanced(CHECK_INCLUDE_DIR CHECK_LIBRARY)
"
        )
    endif()
    
    # Add test directory if it exists
    if(EXISTS "${PROJECT_SOURCE_DIR}/tests/CMakeLists.txt")
        add_subdirectory(tests)
    else()
        message(STATUS "Tests directory not found or CMakeLists.txt missing in tests directory")
    endif()
endif()

# Documentation setup
if(LIBRIFT_BUILD_DOCS AND EXISTS "${PROJECT_SOURCE_DIR}/docs/CMakeLists.txt")
    add_subdirectory(docs)
endif()

# Examples setup
if(LIBRIFT_BUILD_EXAMPLES AND EXISTS "${PROJECT_SOURCE_DIR}/examples/CMakeLists.txt")
    add_subdirectory(examples)
endif()

# Output configuration summary
message(STATUS "LibRift Configuration Summary:")
message(STATUS "  Version: ${PROJECT_VERSION}")
message(STATUS "  Build type: ${CMAKE_BUILD_TYPE}")
message(STATUS "  Memory pool: ${LIBRIFT_USE_MEMORY_POOL}")
message(STATUS "  R'' syntax support: ${LIBRIFT_ENABLE_REGEX_SYNTAX}")
message(STATUS "  Thread safety: ${LIBRIFT_ENABLE_THREAD_SAFETY}")
message(STATUS "  Building tests: ${LIBRIFT_BUILD_TESTS}")
message(STATUS "  Building examples: ${LIBRIFT_BUILD_EXAMPLES}")
message(STATUS "  Building documentation: ${LIBRIFT_BUILD_DOCS}")