cmake_minimum_required(VERSION 3.2)

project("cmake-project-template"
	VERSION 0.0.1
	LANGUAGES C CXX
)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -O3")

# HELLO_IS_SUBPROJECT is 1 if added via add_subdirectory() from another project.
get_directory_property(HELLO_IS_SUBPROJECT PARENT_DIRECTORY)
if(HELLO_IS_SUBPROJECT)
    set(HELLO_IS_SUBPROJECT 1)
endif()

###########################################################
# Options
###########################################################
function(option_if_not_defined name description default)
    if(NOT DEFINED ${name})
        option(${name} ${description} ${default})
    endif()
endfunction()

option_if_not_defined(HELLO_WARNINGS_AS_ERRORS "Treat warnings as errors" OFF)
option_if_not_defined(HELLO_BUILD_EXAMPLES "Build example applications" OFF)
option_if_not_defined(HELLO_BUILD_TESTS "Build tests" ON)
option_if_not_defined(HELLO_BUILD_BENCHMARKS "Build benchmarks" OFF)
option_if_not_defined(HELLO_BUILD_SHARED "Build HELLO as a shared / dynamic library (default static)" OFF)
option_if_not_defined(HELLO_INSTALL "Install header and library" OFF)
option_if_not_defined(HELLO_FULL_BENCHMARK "Run benchmarks for [0 .. numLogicalCPUs] with no stepping" OFF)
option_if_not_defined(HELLO_DEBUG_ENABLED "Enable debug checks even in release builds" OFF)


###########################################################
# Directories
###########################################################
function(set_if_not_defined name value)
    if(NOT DEFINED ${name})
        set(${name} ${value} PARENT_SCOPE)
    endif()
endfunction()

set(CMAKE_INSTALL_PREFIX ${PROJECT_SOURCE_DIR})

set(DIVISIBLE_INSTALL_INCLUDE_DIR ${PROJECT_SOURCE_DIR}/include)
set(DIVISIBLE_INSTALL_BIN_DIR ${PROJECT_SOURCE_DIR}/bin)
set(DIVISIBLE_INSTALL_LIB_DIR ${PROJECT_SOURCE_DIR}/lib)

set(DIVISION_HEADERS_DIR ${PROJECT_SOURCE_DIR}/src/division)


set(HELLO_SRC_DIR ${CMAKE_CURRENT_SOURCE_DIR}/src)
set(HELLO_INCLUDE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/include)
set_if_not_defined(HELLO_THIRD_PARTY_DIR ${CMAKE_CURRENT_SOURCE_DIR}/third_party)
set_if_not_defined(HELLO_GOOGLETEST_DIR ${HELLO_THIRD_PARTY_DIR}/googletest)
set_if_not_defined(HELLO_BENCHMARK_DIR ${HELLO_THIRD_PARTY_DIR}/benchmark)


###########################################################
# Submodules
###########################################################
if(HELLO_BUILD_TESTS)
    if(NOT EXISTS ${HELLO_GOOGLETEST_DIR}/.git)
        message(WARNING "third_party/googletest submodule missing.")
        message(WARNING "Run: `git submodule update --init` to build tests.")
        set(HELLO_BUILD_TESTS OFF)
	else()
		add_subdirectory(${HELLO_GOOGLETEST_DIR})
    endif()
endif(HELLO_BUILD_TESTS)

if(HELLO_BUILD_BENCHMARKS)
    if(NOT EXISTS ${HELLO_BENCHMARK_DIR}/.git)
        message(WARNING "third_party/benchmark submodule missing.")
        message(WARNING "Run: `git submodule update --init` to build benchmarks.")
        set(HELLO_BUILD_BENCHMARKS OFF)
    endif()
endif(HELLO_BUILD_BENCHMARKS)

if(HELLO_BUILD_BENCHMARKS)
    set(BENCHMARK_ENABLE_TESTING FALSE CACHE BOOL FALSE FORCE)
    add_subdirectory(${HELLO_BENCHMARK_DIR})
endif(HELLO_BUILD_BENCHMARKS)


###########################################################
# Functions
###########################################################
function(hello_set_target_options target)
    # Enable all warnings
    if(MSVC)
        target_compile_options(${target} PRIVATE "-W4")
    else()
        target_compile_options(${target} PRIVATE "-Wall")
    endif()

    # Disable specific, pedantic warnings
    if(MSVC)
        target_compile_options(${target} PRIVATE
            "-D_CRT_SECURE_NO_WARNINGS"
            "/wd4127" # conditional expression is constant
            "/wd4324" # structure was padded due to alignment specifier
        )
    endif()

    # Treat all warnings as errors
    if(HELLO_WARNINGS_AS_ERRORS)
        if(MSVC)
            target_compile_options(${target} PRIVATE "/WX")
        else()
            target_compile_options(${target} PRIVATE "-Werror")
        endif()
    endif(HELLO_WARNINGS_AS_ERRORS)

    if(HELLO_DEBUG_ENABLED)
        target_compile_definitions(${target} PRIVATE "HELLO_DEBUG_ENABLED=1")
    endif()

    target_include_directories(${target} PUBLIC $<BUILD_INTERFACE:${HELLO_INCLUDE_DIR}>)
endfunction(hello_set_target_options)


###########################################################
# Install
###########################################################
if(HELLO_INSTALL)

endif(HELLO_INSTALL)


###########################################################
# Tests
###########################################################
if(HELLO_BUILD_TESTS)
	add_subdirectory(test)
endif(HELLO_BUILD_TESTS)


###########################################################
# Benchmarks
###########################################################
if(HELLO_BUILD_BENCHMARKS)

endif(HELLO_BUILD_BENCHMARKS)


###########################################################
# Examples
###########################################################
if(HELLO_BUILD_EXAMPLES)

endif(HELLO_BUILD_EXAMPLES)

include_directories(${DIVISIBLE_INSTALL_INCLUDE_DIR})
#include_directories(${DIVISION_HEADERS_DIR})

add_subdirectory(src)

