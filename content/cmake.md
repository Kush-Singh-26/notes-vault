---
title: "Modern CMake 4+ Complete Reference"
description: "cmake reference"
---

## The Definitive Beginner-to-Intermediate Guide

---

# PART 1: FOUNDATIONS

---

## 1. What Is CMake and Why Modern CMake

CMake is a cross-platform build system generator. It does not build your code — it generates the files that your actual build system (Ninja, Make, Visual Studio, Xcode) uses to build your code. You write CMakeLists.txt files, CMake reads them, and outputs a build system for your target platform.

"Modern CMake" refers to the paradigm shift that happened around CMake 3.0 and has been refined through every release since. The old way was about variables and directory-level settings. The modern way is about targets and their properties, and crucially, about how properties propagate between targets.

If you ever find yourself writing something like:

    include_directories(...)
    link_directories(...)
    add_definitions(...)

Stop. You are writing old CMake. These commands exist for legacy reasons. Modern CMake almost never uses them.

CMake 4.0 (released 2025) raised the minimum required CMake version floor and deprecated several legacy interfaces more aggressively. It also removed support for policies older than a certain threshold. This document targets CMake 4.x idioms throughout.

---

## 2. Installation

On Fedora/RHEL:

    sudo dnf install cmake

On Ubuntu/Debian:

    sudo apt install cmake

Via pip (gets you a very recent version):

    pip install cmake

From source or via the official installer at cmake.org. Always verify with:

    cmake --version

---

## 3. The Minimum Viable CMakeLists.txt

Every CMake project starts with a CMakeLists.txt in the root of the source tree. The bare minimum for a C++ executable:

    cmake_minimum_required(VERSION 3.25...4.0)

    project(MyProject
        VERSION 1.0.0
        DESCRIPTION "A sample project"
        LANGUAGES CXX
    )

    add_executable(my_app main.cpp)

Line by line:

`cmake_minimum_required(VERSION 3.25...4.0)` — the version range syntax (min...max) tells CMake: require at least 3.25, but if a newer CMake up to 4.0 is running, behave like 4.0. This is the recommended modern form. It activates all policies up to the max version, giving you the latest behavior without deprecation warnings.

`project(...)` — declares the project. Sets variables like PROJECT_NAME, PROJECT_VERSION, PROJECT_SOURCE_DIR, PROJECT_BINARY_DIR. Always declare LANGUAGES explicitly; omitting it causes CMake to probe for C and CXX by default which is fine but implicit.

`add_executable(my_app main.cpp)` — creates a target named my_app from main.cpp.

---

## 4. Configuring and Building

CMake uses an out-of-source build pattern. Never build inside your source tree.

    mkdir build
    cd build
    cmake ..
    cmake --build .

Or, more idiomatically with a preset (covered later):

    cmake -S . -B build
    cmake --build build

`-S` specifies the source directory (where CMakeLists.txt lives). `-B` specifies the build directory. This is the preferred modern invocation — you never have to cd into the build directory.

To choose a generator explicitly:

    cmake -S . -B build -G Ninja
    cmake -S . -B build -G "Unix Makefiles"
    cmake -S . -B build -G "Visual Studio 17 2022"

To set the build type:

    cmake -S . -B build -DCMAKE_BUILD_TYPE=Release

Build types: Debug, Release, RelWithDebInfo, MinSizeRel.

To build with parallel jobs:

    cmake --build build --parallel 8
    cmake --build build -j8

To install:

    cmake --install build --prefix /usr/local

---

# PART 2: TARGETS — THE CORE CONCEPT

---

## 5. Everything Is a Target

A target is the fundamental unit of modern CMake. A target represents something: an executable, a library, or a logical grouping. Targets have properties. Properties flow between targets via usage requirements.

The three target-creating commands:

    add_executable(my_app main.cpp other.cpp)

    add_library(my_lib STATIC lib.cpp)
    add_library(my_lib SHARED lib.cpp)
    add_library(my_lib OBJECT lib.cpp)
    add_library(my_lib INTERFACE)

    add_library(my_alias ALIAS other_target)

Library types:
- STATIC — archive (.a / .lib), linked at compile time
- SHARED — dynamic library (.so / .dll), linked at load time
- OBJECT — compiles sources into object files, not linked into any library, useful for sharing object files between multiple targets
- INTERFACE — has no build artifact; exists only to carry usage requirements (headers, compile flags, link libraries) — extremely important in modern CMake
- ALIAS — a read-only alias for another target; used heavily by library authors to provide namespaced targets

---

## 6. Target Properties and the Three Commands

Every property on a target has a visibility:
- PRIVATE — applies only to this target during compilation/linking
- PUBLIC — applies to this target AND propagates to any target that links against this target
- INTERFACE — does NOT apply to this target, but DOES propagate to consumers

The three most important property-setting commands:

**target_include_directories**

    target_include_directories(my_lib
        PUBLIC
            $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
            $<INSTALL_INTERFACE:include>
        PRIVATE
            ${CMAKE_CURRENT_SOURCE_DIR}/src
    )

PUBLIC include directories propagate to consumers. PRIVATE ones are only for building the library itself.

**target_compile_options**

    target_compile_options(my_lib
        PRIVATE
            -Wall
            -Wextra
            -Wpedantic
    )

Warning flags are almost always PRIVATE. You don't want to force your consumers to compile with your warning flags.

**target_link_libraries**

    target_link_libraries(my_app
        PRIVATE
            my_lib
            Threads::Threads
    )

This is the most important command in modern CMake. When you link my_lib into my_app, all PUBLIC and INTERFACE properties of my_lib automatically propagate to my_app. This is the entire point of modern CMake's target model.

**The rule of thumb:**
- Use PRIVATE when the dependency is an implementation detail not visible in your headers
- Use PUBLIC when the dependency is part of your public API (you include its headers in your own public headers)
- Use INTERFACE for header-only libraries or to propagate requirements without building anything

---

## 7. target_compile_features — C++ Standard

Never set CMAKE_CXX_STANDARD as a global variable if you can avoid it. Instead, per target:

    target_compile_features(my_lib PUBLIC cxx_std_20)
    target_compile_features(my_app PRIVATE cxx_std_23)

This sets the minimum C++ standard required and propagates correctly via PUBLIC/PRIVATE/INTERFACE. CMake will pass the right -std=c++20 or equivalent flag.

If you must set it globally (acceptable for application projects):

    set(CMAKE_CXX_STANDARD 20)
    set(CMAKE_CXX_STANDARD_REQUIRED ON)
    set(CMAKE_CXX_EXTENSIONS OFF)  # disables -std=gnu++20, forces -std=c++20

CMAKE_CXX_EXTENSIONS OFF is important if you want strict ISO C++ without GNU extensions.

---

## 8. Compile Definitions

    target_compile_definitions(my_lib
        PRIVATE
            MY_LIB_BUILD          # defined during build
        PUBLIC
            MY_LIB_VERSION=2
        INTERFACE
            MY_LIB_CONSUMER       # only defined for consumers
    )

Equivalent to -DMY_LIB_BUILD etc. Do not use add_definitions() globally.

---

# PART 3: PROJECT STRUCTURE

---

## 9. Directory Layout — The Recommended Pattern

A well-structured CMake project:

    MyProject/
    ├── CMakeLists.txt              (root)
    ├── CMakePresets.json
    ├── cmake/
    │   ├── FindSomeLib.cmake
    │   └── SomeLibConfig.cmake.in
    ├── include/
    │   └── myproject/
    │       └── core.hpp
    ├── src/
    │   ├── CMakeLists.txt
    │   ├── core.cpp
    │   └── utils.cpp
    ├── tests/
    │   ├── CMakeLists.txt
    │   └── test_core.cpp
    └── extern/
        └── somelib/               (submodule or FetchContent)

---

## 10. Subdirectories

    add_subdirectory(src)
    add_subdirectory(tests)

Each subdirectory has its own CMakeLists.txt. Variables in subdirectories are local unless set with CACHE or PARENT_SCOPE. Targets, however, are global — a target defined in src/CMakeLists.txt is visible to tests/CMakeLists.txt if tests is added after src.

    # src/CMakeLists.txt
    add_library(core STATIC core.cpp)
    target_include_directories(core PUBLIC
        $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include>
    )

    # tests/CMakeLists.txt
    add_executable(test_core test_core.cpp)
    target_link_libraries(test_core PRIVATE core GTest::GTest)

---

## 11. Variables

CMake variables are strings. All variable values are strings or lists of strings (a list is just a string with semicolons).

    set(MY_VAR "hello")
    set(MY_LIST "a;b;c")        # equivalent:
    set(MY_LIST "a" "b" "c")

Access with ${MY_VAR}. Variables are scoped to the current CMakeLists.txt and its subdirectories (function scope works differently — covered later).

Important built-in variables:

    CMAKE_SOURCE_DIR          # top-level source directory (project root)
    CMAKE_BINARY_DIR          # top-level build directory
    CMAKE_CURRENT_SOURCE_DIR  # source dir of currently-processing CMakeLists.txt
    CMAKE_CURRENT_BINARY_DIR  # build dir of currently-processing CMakeLists.txt
    PROJECT_SOURCE_DIR        # source dir of most recent project() call
    PROJECT_BINARY_DIR        # build dir of most recent project() call
    PROJECT_NAME              # name from project()
    PROJECT_VERSION           # version from project()
    CMAKE_BUILD_TYPE          # Debug/Release/etc (single-config generators)
    CMAKE_INSTALL_PREFIX      # installation root
    BUILD_SHARED_LIBS         # if ON, add_library without type builds SHARED

Prefer PROJECT_SOURCE_DIR over CMAKE_SOURCE_DIR in library code — it works correctly when your project is consumed as a subdirectory of another project.

---

## 12. Cache Variables and Options

Cache variables persist between runs and can be set from the command line.

    set(MY_OPTION "default_value" CACHE STRING "Description of this variable")

    option(MYPROJECT_BUILD_TESTS "Build the test suite" OFF)
    option(MYPROJECT_ENABLE_ASAN "Enable AddressSanitizer" OFF)

From command line:

    cmake -S . -B build -DMYPROJECT_BUILD_TESTS=ON

In CMakeLists.txt, check with:

    if(MYPROJECT_BUILD_TESTS)
        add_subdirectory(tests)
    endif()

Cache variable types: STRING, BOOL, PATH, FILEPATH, INTERNAL.
INTERNAL variables are not shown in cmake-gui.

---

# PART 4: CONTROL FLOW

---

## 13. Conditionals

    if(CONDITION)
        # ...
    elseif(OTHER)
        # ...
    else()
        # ...
    endif()

Condition forms:

    if(MY_VAR)                  # true if MY_VAR is not empty, 0, OFF, NO, FALSE, N, IGNORE, NOTFOUND
    if(NOT MY_VAR)
    if(A AND B)
    if(A OR B)

    if(DEFINED MY_VAR)          # true if variable exists (even if empty)
    if(DEFINED CACHE{MY_VAR})   # true if cache variable exists

    if(MY_VAR STREQUAL "hello")
    if(MY_VAR MATCHES "^hel.*")  # regex
    if(MY_NUM GREATER 5)
    if(MY_NUM LESS_EQUAL 10)
    if(MY_NUM EQUAL 7)
    if(VERSION_VAR VERSION_GREATER_EQUAL "3.20")

    if(TARGET my_lib)            # true if a target with that name exists
    if(EXISTS /path/to/file)
    if(IS_DIRECTORY /path)
    if(file1 IS_NEWER_THAN file2)

---

## 14. Loops

foreach:

    set(ITEMS alpha beta gamma)
    foreach(item IN LISTS ITEMS)
        message(STATUS "Item: ${item}")
    endforeach()

    foreach(i RANGE 0 10 2)     # 0, 2, 4, 6, 8, 10
        message(STATUS "${i}")
    endforeach()

    foreach(item IN ITEMS alpha beta gamma)  # iterate literals directly
    endforeach()

while:

    set(counter 0)
    while(counter LESS 5)
        math(EXPR counter "${counter} + 1")
    endwhile()

break() and continue() work as expected inside loops.

---

## 15. Functions and Macros

Functions create a new scope. Variables set inside a function do not affect the caller unless you use PARENT_SCOPE.

    function(my_function arg1 arg2)
        message(STATUS "arg1=${arg1}, arg2=${arg2}")
        set(RESULT "computed" PARENT_SCOPE)
    endfunction()

    my_function("hello" "world")
    message(STATUS "RESULT=${RESULT}")  # prints "computed"

Macros paste code without a new scope — like C macros. Arguments are string-replaced, not variables.

    macro(my_macro arg)
        message(STATUS "macro arg: ${arg}")
    endmacro()

Prefer functions over macros. Macros have subtle gotchas with return() and variable scoping.

ARGC, ARGV, ARGN — inside functions/macros:

    function(variadic_func)
        message(STATUS "Got ${ARGC} arguments")
        foreach(arg IN LISTS ARGN)  # ARGN = arguments beyond named ones
            message(STATUS "  ${arg}")
        endforeach()
    endfunction()

    variadic_func(one two three)

cmake_parse_arguments — for keyword-style arguments:

    function(create_library)
        cmake_parse_arguments(
            PARSE_ARGV 0          # start parsing from arg 0
            ARG                   # prefix for result variables
            "STATIC;SHARED"       # options (boolean flags)
            "NAME;OUTPUT_DIR"     # one-value keywords
            "SOURCES;HEADERS"     # multi-value keywords
        )
        # ARG_NAME, ARG_SOURCES, ARG_HEADERS, ARG_STATIC, etc. are now set
        add_library(${ARG_NAME} ${ARG_SOURCES})
    endfunction()

    create_library(
        NAME mylib
        SOURCES a.cpp b.cpp
        HEADERS a.hpp b.hpp
        STATIC
    )

---

# PART 5: FINDING AND USING DEPENDENCIES

---

## 16. find_package — The Standard Way

    find_package(OpenSSL REQUIRED)
    target_link_libraries(my_app PRIVATE OpenSSL::SSL OpenSSL::Crypto)

find_package searches for a package in two modes:

**Module mode**: looks for FindXxx.cmake in CMAKE_MODULE_PATH and CMake's built-in module path (cmake --help-module-list shows all built-in modules).

**Config mode**: looks for XxxConfig.cmake or xxx-config.cmake installed by the package itself, usually in its installation prefix.

Modern packages (LLVM, Boost, OpenCV, etc.) provide their own config files. CMake's built-in Find modules cover common system libraries (Threads, OpenGL, ZLIB, etc.).

Common patterns:

    find_package(Threads REQUIRED)
    target_link_libraries(my_app PRIVATE Threads::Threads)

    find_package(OpenGL REQUIRED)
    target_link_libraries(my_app PRIVATE OpenGL::GL)

    find_package(ZLIB REQUIRED)
    target_link_libraries(my_app PRIVATE ZLIB::ZLIB)

    find_package(Boost 1.80 REQUIRED COMPONENTS filesystem program_options)
    target_link_libraries(my_app PRIVATE Boost::filesystem Boost::program_options)

The REQUIRED keyword causes a fatal error if the package is not found. Without it, you get a <PackageName>_FOUND variable to check manually.

Version specification:

    find_package(OpenSSL 3.0 REQUIRED)       # at least 3.0
    find_package(OpenSSL 3.0...3.9 REQUIRED) # version range

After find_package, use the imported targets (Namespace::Target form). Never use the raw variables like OPENSSL_INCLUDE_DIRS directly on modern code.

---

## 17. FetchContent — Downloading Dependencies at Configure Time

FetchContent is the modern CMake-native way to pull in dependencies from the internet or local paths, making them available as if they were subdirectories of your project.

    include(FetchContent)

    FetchContent_Declare(
        googletest
        GIT_REPOSITORY https://github.com/google/googletest.git
        GIT_TAG        v1.14.0
        GIT_SHALLOW    TRUE
    )

    FetchContent_MakeAvailable(googletest)

    target_link_libraries(my_tests PRIVATE GTest::GTest GTest::Main)

FetchContent_MakeAvailable downloads (on first configure) and adds the content as a subdirectory. The targets defined by the fetched project become available just like local targets.

You can declare multiple dependencies and make them available in one shot:

    FetchContent_Declare(fmt ...)
    FetchContent_Declare(spdlog ...)
    FetchContent_MakeAvailable(fmt spdlog)

To avoid re-downloading every configure, FetchContent caches downloads in the build tree. If you set FETCHCONTENT_BASE_DIR globally or per-declaration you can control where.

Suppressing verbose output from fetched subprojects:

    set(FETCHCONTENT_QUIET OFF)    # show progress
    set(FETCHCONTENT_QUIET ON)     # suppress (default)

Overriding to use a local copy (useful for offline dev):

    set(FETCHCONTENT_SOURCE_DIR_GOOGLETEST /path/to/local/googletest)
    FetchContent_MakeAvailable(googletest)

Other source types for FetchContent_Declare:

    FetchContent_Declare(mylib
        URL      https://example.com/mylib-1.0.tar.gz
        URL_HASH SHA256=abcdef1234...
    )

    FetchContent_Declare(mylib
        SVN_REPOSITORY https://...
    )

---

## 18. Writing a Find Module

If a package doesn't provide a config file and CMake has no built-in module for it, write your own FindMyLib.cmake in your cmake/ directory and add that to CMAKE_MODULE_PATH:

    list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")
    find_package(MyLib REQUIRED)

A Find module template:

    # FindMyLib.cmake
    find_path(MYLIB_INCLUDE_DIR mylib/mylib.h
        HINTS /usr/local/include /usr/include
    )

    find_library(MYLIB_LIBRARY
        NAMES mylib libmylib
        HINTS /usr/local/lib /usr/lib
    )

    include(FindPackageHandleStandardArgs)
    find_package_handle_standard_args(MyLib
        REQUIRED_VARS MYLIB_INCLUDE_DIR MYLIB_LIBRARY
    )

    if(MyLib_FOUND AND NOT TARGET MyLib::MyLib)
        add_library(MyLib::MyLib UNKNOWN IMPORTED)
        set_target_properties(MyLib::MyLib PROPERTIES
            IMPORTED_LOCATION "${MYLIB_LIBRARY}"
            INTERFACE_INCLUDE_DIRECTORIES "${MYLIB_INCLUDE_DIR}"
        )
    endif()

    mark_as_advanced(MYLIB_INCLUDE_DIR MYLIB_LIBRARY)

---

# PART 6: GENERATOR EXPRESSIONS

---

## 19. Generator Expressions (genexes)

Generator expressions are evaluated at build system generation time (after configure), not during the CMake configure phase. They look like $<...> and are used wherever you need build-system-time logic.

They appear primarily in:
- target_include_directories
- target_compile_options
- target_link_libraries
- target_compile_definitions
- install(TARGETS ... DESTINATION ...)

**Conditional expressions:**

    $<condition:value>          # expands to value if condition is 1, empty if 0

    $<CONFIG:Release>           # 1 if current config is Release
    $<PLATFORM_ID:Linux>        # 1 if platform is Linux
    $<CXX_COMPILER_ID:GNU>      # 1 if compiler is GCC
    $<TARGET_EXISTS:mytarget>   # 1 if target exists

Example:

    target_compile_options(my_app PRIVATE
        $<$<CONFIG:Debug>:-g3 -O0>
        $<$<CONFIG:Release>:-O3 -DNDEBUG>
        $<$<CXX_COMPILER_ID:GNU>:-fstack-protector-strong>
        $<$<CXX_COMPILER_ID:MSVC>:/W4>
    )

**Logical expressions:**

    $<AND:cond1,cond2>
    $<OR:cond1,cond2>
    $<NOT:cond>
    $<BOOL:string>           # 1 if string is not empty/0/FALSE/etc.

**Build vs Install interface expressions:**

    $<BUILD_INTERFACE:path>    # only used during build
    $<INSTALL_INTERFACE:path>  # only used after install

Critical for include paths:

    target_include_directories(my_lib PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:include>
    )

During build: consumers get the absolute source path. After install: consumers get the relative install path.

**Target property expressions:**

    $<TARGET_FILE:my_app>          # full path to executable
    $<TARGET_FILE_NAME:my_app>     # just the filename
    $<TARGET_FILE_DIR:my_app>      # directory of the executable

**String expressions:**

    $<LOWER_CASE:STRING>
    $<UPPER_CASE:string>
    $<JOIN:list,separator>

---

# PART 7: INSTALLATION

---

## 20. Installing Your Project

A well-written CMake project supports installation so other projects (or package managers) can consume it.

**Basic install:**

    install(TARGETS my_app my_lib
        RUNTIME DESTINATION bin           # executables
        LIBRARY DESTINATION lib           # shared libraries
        ARCHIVE DESTINATION lib           # static libraries
        INCLUDES DESTINATION include      # not a real install, sets INTERFACE_INCLUDE_DIRECTORIES
    )

    install(DIRECTORY include/
        DESTINATION include
    )

    install(FILES cmake/MyProjectConfig.cmake
        DESTINATION lib/cmake/MyProject
    )

**GNUInstallDirs** — always use this module for standard paths:

    include(GNUInstallDirs)

    install(TARGETS my_lib
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
        INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
    )

GNUInstallDirs provides: CMAKE_INSTALL_BINDIR (bin), CMAKE_INSTALL_LIBDIR (lib or lib64), CMAKE_INSTALL_INCLUDEDIR (include), CMAKE_INSTALL_DATADIR, etc. These follow platform conventions automatically.

**Installing headers:**

    install(DIRECTORY include/
        DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
        FILES_MATCHING PATTERN "*.hpp" PATTERN "*.h"
    )

---

## 21. Packaging for Consumers — Config Files

For your library to be findable by other CMake projects via find_package, you need to install a config file.

This is a two-step process.

**Step 1 — Export targets:**

    install(TARGETS my_lib
        EXPORT MyProjectTargets
        ...
    )

    install(EXPORT MyProjectTargets
        FILE MyProjectTargets.cmake
        NAMESPACE MyProject::
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/MyProject
    )

**Step 2 — Write and install a config file:**

Create cmake/MyProjectConfig.cmake.in:

    @PACKAGE_INIT@

    include("${CMAKE_CURRENT_LIST_DIR}/MyProjectTargets.cmake")

    check_required_components(MyProject)

In CMakeLists.txt:

    include(CMakePackageConfigHelpers)

    configure_package_config_file(
        cmake/MyProjectConfig.cmake.in
        ${CMAKE_CURRENT_BINARY_DIR}/MyProjectConfig.cmake
        INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/MyProject
    )

    write_basic_package_version_file(
        ${CMAKE_CURRENT_BINARY_DIR}/MyProjectVersionConfig.cmake
        VERSION ${PROJECT_VERSION}
        COMPATIBILITY SameMajorVersion
    )

    install(FILES
        ${CMAKE_CURRENT_BINARY_DIR}/MyProjectConfig.cmake
        ${CMAKE_CURRENT_BINARY_DIR}/MyProjectVersionConfig.cmake
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/MyProject
    )

After installing, consumers can do:

    find_package(MyProject 1.0 REQUIRED)
    target_link_libraries(their_app PRIVATE MyProject::my_lib)

---

# PART 8: TESTING

---

## 22. CTest — Built-in Test Runner

    enable_testing()

    add_executable(test_core tests/test_core.cpp)
    target_link_libraries(test_core PRIVATE core GTest::GTest GTest::Main)

    add_test(NAME CoreTests COMMAND test_core)

To run tests:

    cmake --build build
    ctest --test-dir build
    ctest --test-dir build --output-on-failure
    ctest --test-dir build -R "^Core"       # run tests matching regex
    ctest --test-dir build -j8              # parallel

Test properties:

    set_tests_properties(CoreTests PROPERTIES
        TIMEOUT 30
        ENVIRONMENT "MY_ENV_VAR=value"
        LABELS "unit;fast"
    )

---

## 23. GoogleTest Integration

    include(FetchContent)
    FetchContent_Declare(
        googletest
        GIT_REPOSITORY https://github.com/google/googletest.git
        GIT_TAG        v1.14.0
        GIT_SHALLOW    TRUE
    )
    set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)  # Windows
    FetchContent_MakeAvailable(googletest)

    include(GoogleTest)

    add_executable(my_tests test_foo.cpp test_bar.cpp)
    target_link_libraries(my_tests PRIVATE my_lib GTest::gtest_main)

    gtest_discover_tests(my_tests)

gtest_discover_tests automatically discovers all TEST() and TEST_F() cases and registers them as individual CTest tests. Much better than a single add_test for the whole executable.

---

# PART 9: CMAKE PRESETS

---

## 24. CMakePresets.json — Modern Project Configuration

CMake presets (CMake 3.19+, mature in 3.21+) let you define named configurations that bundle together generator, build directory, CMake variables, environment variables, and more. Commit CMakePresets.json to version control; CMakeUserPresets.json is for personal overrides and should be gitignored.

A complete CMakePresets.json:

    {
      "version": 6,
      "configurePresets": [
        {
          "name": "base",
          "hidden": true,
          "generator": "Ninja",
          "binaryDir": "${sourceDir}/build/${presetName}",
          "cacheVariables": {
            "CMAKE_EXPORT_COMPILE_COMMANDS": "ON"
          }
        },
        {
          "name": "debug",
          "inherits": "base",
          "displayName": "Debug",
          "cacheVariables": {
            "CMAKE_BUILD_TYPE": "Debug",
            "CMAKE_CXX_FLAGS": "-fsanitize=address,undefined -fno-omit-frame-pointer"
          }
        },
        {
          "name": "release",
          "inherits": "base",
          "displayName": "Release",
          "cacheVariables": {
            "CMAKE_BUILD_TYPE": "Release"
          }
        },
        {
          "name": "asan",
          "inherits": "debug",
          "displayName": "Debug + ASan",
          "cacheVariables": {
            "CMAKE_CXX_FLAGS": "-fsanitize=address -fno-omit-frame-pointer"
          }
        }
      ],
      "buildPresets": [
        {
          "name": "debug",
          "configurePreset": "debug"
        },
        {
          "name": "release",
          "configurePreset": "release"
        }
      ],
      "testPresets": [
        {
          "name": "default",
          "configurePreset": "debug",
          "output": { "outputOnFailure": true },
          "execution": { "jobs": 4 }
        }
      ]
    }

Usage:

    cmake --preset debug
    cmake --build --preset debug
    ctest --preset default
    cmake --list-presets

---

# PART 10: ADVANCED TARGETS AND PROPERTIES

---

## 25. Object Libraries

Object libraries compile sources without linking. Useful when multiple targets need the same compiled objects without recompiling:

    add_library(common_objects OBJECT
        common/logging.cpp
        common/utils.cpp
    )
    target_include_directories(common_objects PUBLIC include/)

    add_executable(my_app main.cpp)
    target_link_libraries(my_app PRIVATE common_objects)

    add_executable(other_app other_main.cpp)
    target_link_libraries(other_app PRIVATE common_objects)

Both executables reuse the compiled objects without recompiling.

---

## 26. Interface Libraries — Header-Only Libraries

    add_library(my_headers INTERFACE)

    target_include_directories(my_headers INTERFACE
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:include>
    )

    target_compile_features(my_headers INTERFACE cxx_std_20)

    target_link_libraries(my_app PRIVATE my_headers)

The app gets the include directories and C++20 requirement automatically. No sources are compiled for my_headers itself.

---

## 27. Imported Targets

Imported targets represent pre-built libraries or executables. find_package typically creates these for you, but you can create them manually:

    add_library(SomeLib::SomeLib SHARED IMPORTED)
    set_target_properties(SomeLib::SomeLib PROPERTIES
        IMPORTED_LOCATION /path/to/libsomelib.so
        INTERFACE_INCLUDE_DIRECTORIES /path/to/somelib/include
    )

UNKNOWN type works when you don't know if it's shared or static:

    add_library(SomeLib::SomeLib UNKNOWN IMPORTED)

---

## 28. Alias Targets

    add_library(MyProject::Core ALIAS core)
    add_executable(MyProject::App ALIAS app)

Aliases provide a namespaced name for a target. When find_package loads your installed config, it exposes MyProject::Core. Using aliases internally means your internal add_subdirectory consumers use the same name as external find_package consumers — no code differences.

---

## 29. Custom Commands and Custom Targets

**add_custom_command** — adds a rule to generate a file during the build:

    add_custom_command(
        OUTPUT generated_code.cpp
        COMMAND python3 ${CMAKE_CURRENT_SOURCE_DIR}/scripts/codegen.py
                        --input schema.proto
                        --output generated_code.cpp
        DEPENDS schema.proto
                ${CMAKE_CURRENT_SOURCE_DIR}/scripts/codegen.py
        COMMENT "Generating code from schema.proto"
        VERBATIM
    )

    add_executable(my_app main.cpp generated_code.cpp)

**add_custom_target** — creates a target with no output file; always rebuilds when requested:

    add_custom_target(run_linter
        COMMAND clang-tidy ${SOURCE_FILES}
        WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
        COMMENT "Running clang-tidy"
        VERBATIM
    )

    add_custom_target(format
        COMMAND clang-format -i ${SOURCE_FILES}
        VERBATIM
    )

Build the custom target explicitly:

    cmake --build build --target run_linter

**PRE_BUILD / PRE_LINK / POST_BUILD** hooks:

    add_custom_command(TARGET my_app POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy $<TARGET_FILE:my_app> /deploy/
        COMMENT "Deploying app"
        VERBATIM
    )

---

# PART 11: PROPERTIES

---

## 30. Setting and Getting Properties

Properties can be set on targets, directories, source files, tests, and the global scope.

    set_target_properties(my_lib PROPERTIES
        OUTPUT_NAME "mylib"
        VERSION     ${PROJECT_VERSION}
        SOVERSION   1
        POSITION_INDEPENDENT_CODE ON
    )

    get_target_property(output_name my_lib OUTPUT_NAME)
    message(STATUS "Output name: ${output_name}")

    set_property(TARGET my_lib PROPERTY CXX_VISIBILITY_PRESET hidden)
    set_property(TARGET my_lib PROPERTY VISIBILITY_INLINES_HIDDEN ON)

**Source file properties:**

    set_source_files_properties(legacy.c PROPERTIES
        COMPILE_FLAGS "-w"              # suppress warnings for this file only
        LANGUAGE CXX                    # compile as C++ even though .c extension
    )

**Directory properties:**

    set_property(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR} PROPERTY
        VS_STARTUP_PROJECT my_app       # Visual Studio: set startup project
    )

**Global properties:**

    set_property(GLOBAL PROPERTY USE_FOLDERS ON)    # enables IDE solution folders
    set_target_properties(my_lib PROPERTIES FOLDER "Libraries")

---

## 31. Useful Target Properties

    COMPILE_OPTIONS           # compiler flags
    COMPILE_DEFINITIONS       # preprocessor defines
    INCLUDE_DIRECTORIES       # include paths
    LINK_LIBRARIES            # linked targets/libs
    LINK_OPTIONS              # linker flags
    SOURCES                   # source files
    OUTPUT_NAME               # output filename without extension
    RUNTIME_OUTPUT_DIRECTORY  # where executables go
    LIBRARY_OUTPUT_DIRECTORY  # where shared libs go
    ARCHIVE_OUTPUT_DIRECTORY  # where static libs go
    POSITION_INDEPENDENT_CODE # -fPIC
    CXX_STANDARD              # 17, 20, 23
    CXX_STANDARD_REQUIRED     # ON/OFF
    CXX_EXTENSIONS            # ON/OFF
    VERSION                   # shared library version
    SOVERSION                 # shared library soname version
    EXPORT_NAME               # name for export() / install(EXPORT)
    INTERPROCEDURAL_OPTIMIZATION  # link-time optimization (LTO)

---

# PART 12: TOOLCHAINS AND CROSS-COMPILATION

---

## 32. Toolchain Files

A toolchain file tells CMake which compiler and tools to use. Essential for cross-compilation.

    # toolchains/arm-linux-gnueabihf.cmake
    set(CMAKE_SYSTEM_NAME Linux)
    set(CMAKE_SYSTEM_PROCESSOR arm)

    set(CMAKE_C_COMPILER   arm-linux-gnueabihf-gcc)
    set(CMAKE_CXX_COMPILER arm-linux-gnueabihf-g++)

    set(CMAKE_FIND_ROOT_PATH /usr/arm-linux-gnueabihf)
    set(CMAKE_FIND_ROOT_PATH_MODE_PROGRAM NEVER)
    set(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)
    set(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)

    set(CMAKE_SYSROOT /path/to/sysroot)  # optional

Use it:

    cmake -S . -B build --toolchain toolchains/arm-linux-gnueabihf.cmake

Or in a preset:

    {
      "name": "arm-cross",
      "inherits": "base",
      "toolchainFile": "toolchains/arm-linux-gnueabihf.cmake"
    }

---

## 33. Compiler Detection and Platform Guards

    if(CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
        target_compile_options(my_lib PRIVATE -fno-rtti)
    elseif(CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
        target_compile_options(my_lib PRIVATE -fno-rtti)
    elseif(CMAKE_CXX_COMPILER_ID STREQUAL "MSVC")
        target_compile_options(my_lib PRIVATE /GR-)
    endif()

    if(CMAKE_SYSTEM_NAME STREQUAL "Linux")
        target_link_libraries(my_app PRIVATE dl pthread)
    elseif(CMAKE_SYSTEM_NAME STREQUAL "Darwin")
        target_link_libraries(my_app PRIVATE "-framework CoreFoundation")
    elseif(WIN32)
        target_link_libraries(my_app PRIVATE ws2_32)
    endif()

Generator expressions are cleaner for simple per-compiler flags:

    target_compile_options(my_lib PRIVATE
        $<$<CXX_COMPILER_ID:MSVC>:/W4 /WX>
        $<$<NOT:$<CXX_COMPILER_ID:MSVC>>:-Wall -Wextra -Werror>
    )

---

# PART 13: USEFUL MODULES

---

## 34. Built-in CMake Modules

Include modules with include(ModuleName).

**CheckCXXCompilerFlag** — test if a flag is supported:

    include(CheckCXXCompilerFlag)
    check_cxx_compiler_flag("-mavx2" HAVE_AVX2)
    if(HAVE_AVX2)
        target_compile_options(my_lib PRIVATE -mavx2)
    endif()

**CheckCXXSourceCompiles** — test if code compiles:

    include(CheckCXXSourceCompiles)
    check_cxx_source_compiles("
        #include <optional>
        int main() { std::optional<int> o; return 0; }
    " HAVE_STD_OPTIONAL)

**CMakePackageConfigHelpers** — already covered in installation section.

**GNUInstallDirs** — standard install paths.

**GenerateExportHeader** — generates a header with visibility macros for shared library symbol export:

    include(GenerateExportHeader)
    generate_export_header(my_lib
        EXPORT_FILE_NAME include/my_lib/export.hpp
    )

This generates MY_LIB_EXPORT, MY_LIB_NO_EXPORT, MY_LIB_DEPRECATED macros.

**CPack** — packaging:

    include(CPack)
    set(CPACK_PACKAGE_NAME "MyProject")
    set(CPACK_PACKAGE_VERSION ${PROJECT_VERSION})
    set(CPACK_GENERATOR "TGZ;DEB;RPM")

    cpack --config build/CPackConfig.cmake

**FetchContent** — already covered.

**GoogleTest** — already covered.

**ExternalProject** — older alternative to FetchContent; builds at build time rather than configure time. FetchContent is generally preferred now but ExternalProject is still useful when you need to build something with a different compiler or a non-CMake build system.

---

# PART 14: DEBUGGING AND DIAGNOSTICS

---

## 35. message() — Printing Information

    message("plain message")               # always prints
    message(STATUS "informational")        # prefix --
    message(WARNING "non-fatal warning")
    message(SEND_ERROR "non-fatal error")  # continue configuring but fail at end
    message(FATAL_ERROR "fatal error")     # stop immediately
    message(DEBUG "debug info")            # only with --log-level=DEBUG
    message(VERBOSE "verbose info")        # only with --log-level=VERBOSE
    message(TRACE "trace info")

Control log level:

    cmake -S . -B build --log-level=DEBUG

**cmake_print_variables** — quick variable dump:

    include(CMakePrintHelpers)
    cmake_print_variables(MY_VAR CMAKE_BUILD_TYPE PROJECT_VERSION)

**cmake_print_properties** — quick property dump:

    cmake_print_properties(TARGETS my_lib my_app
        PROPERTIES COMPILE_OPTIONS INCLUDE_DIRECTORIES LINK_LIBRARIES
    )

---

## 36. Common Debugging Techniques

Print all variables:

    get_cmake_property(all_vars VARIABLES)
    foreach(var IN LISTS all_vars)
        message(STATUS "${var}=${${var}}")
    endforeach()

Inspect a target's properties:

    get_target_property(incdirs my_lib INTERFACE_INCLUDE_DIRECTORIES)
    message(STATUS "Interface includes: ${incdirs}")

Check what generator is selected:

    message(STATUS "Generator: ${CMAKE_GENERATOR}")

Check compiler paths:

    message(STATUS "C++ compiler: ${CMAKE_CXX_COMPILER}")
    message(STATUS "C++ compiler ID: ${CMAKE_CXX_COMPILER_ID}")
    message(STATUS "C++ compiler version: ${CMAKE_CXX_COMPILER_VERSION}")

Verbose build output (see actual compiler commands):

    cmake --build build --verbose
    # or
    VERBOSE=1 cmake --build build

Dry run:

    cmake --build build --verbose -- -n   # for Make/Ninja

---

## 37. compile_commands.json

    cmake -S . -B build -DCMAKE_EXPORT_COMPILE_COMMANDS=ON

This generates build/compile_commands.json, used by clangd (LSP), clang-tidy, and other tooling. Essential for any C++ development workflow. In a preset, set this as a cacheVariable.

Symlink it into the source root for clangd to find automatically:

    ln -s build/debug/compile_commands.json compile_commands.json

---

# PART 15: POLICIES

---

## 38. CMake Policies

Policies are CMake's mechanism for introducing breaking changes while maintaining backward compatibility. Each policy has a name like CMP0XXXX. Policies can be either OLD (legacy behavior) or NEW (modern behavior).

When you set cmake_minimum_required(VERSION 3.25...4.0), all policies up to 4.0 are set to NEW automatically. You should almost never need to explicitly set policies.

But you'll encounter them when using old third-party code that doesn't specify a high enough minimum version:

    cmake_policy(SET CMP0077 NEW)  # option() honors normal variables
    cmake_policy(SET CMP0091 NEW)  # MSVC runtime library selection

To apply a policy only within a scope:

    cmake_policy(PUSH)
    cmake_policy(SET CMP0077 NEW)
    add_subdirectory(old_third_party)
    cmake_policy(POP)

Important policies to know:

CMP0048 — project() manages version variables. NEW: it does. Always set NEW.
CMP0069 — INTERPROCEDURAL_OPTIMIZATION is enforced per-target. 
CMP0076 — target_sources converts relative paths to absolute.
CMP0077 — option() does nothing if a normal variable is already set. NEW behavior means option() respects cache, which is what you want.
CMP0091 — MSVC runtime library is set via CMAKE_MSVC_RUNTIME_LIBRARY, not /MT flags.
CMP0135 — FetchContent sets download timestamp to extraction time (avoids spurious rebuilds).

---

# PART 16: MULTI-CONFIG GENERATORS

---

## 39. Single-Config vs Multi-Config Generators

Single-config generators (Ninja, Unix Makefiles) have one build type per build directory. CMAKE_BUILD_TYPE must be set at configure time.

    cmake -S . -B build/release -DCMAKE_BUILD_TYPE=Release
    cmake -S . -B build/debug   -DCMAKE_BUILD_TYPE=Debug

Multi-config generators (Ninja Multi-Config, Visual Studio, Xcode) support multiple configurations in a single build directory. CMAKE_BUILD_TYPE is ignored; configurations are selected at build time.

    cmake -S . -B build -G "Ninja Multi-Config"
    cmake --build build --config Release
    cmake --build build --config Debug

When writing portable CMake, use generator expressions for config-specific settings rather than if(CMAKE_BUILD_TYPE ...) since the latter doesn't work correctly with multi-config generators.

Check for multi-config programmatically:

    get_property(is_multi_config GLOBAL PROPERTY GENERATOR_IS_MULTI_CONFIG)
    if(is_multi_config)
        set(CMAKE_CONFIGURATION_TYPES "Debug;Release;RelWithDebInfo" CACHE STRING "" FORCE)
    endif()

---

# PART 17: REAL-WORLD PATTERNS

---

## 40. A Complete Example Project

    # CMakeLists.txt (root)
    cmake_minimum_required(VERSION 3.25...4.0)

    project(Ember
        VERSION     0.1.0
        DESCRIPTION "A small decoder-only LM"
        LANGUAGES   CXX
    )

    # Global settings
    set(CMAKE_CXX_STANDARD 20)
    set(CMAKE_CXX_STANDARD_REQUIRED ON)
    set(CMAKE_CXX_EXTENSIONS OFF)
    set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

    # Options
    option(EMBER_BUILD_TESTS   "Build tests" OFF)
    option(EMBER_BUILD_BENCH   "Build benchmarks" OFF)
    option(EMBER_ENABLE_ASAN   "Enable AddressSanitizer" OFF)
    option(EMBER_ENABLE_AVX2   "Enable AVX2 SIMD" ON)

    # Output directories
    set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
    set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
    set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)

    # Dependencies
    include(FetchContent)
    FetchContent_Declare(
        fmt
        GIT_REPOSITORY https://github.com/fmtlib/fmt.git
        GIT_TAG        10.2.1
        GIT_SHALLOW    TRUE
    )
    FetchContent_MakeAvailable(fmt)

    find_package(Threads REQUIRED)

    # Subdirectories
    add_subdirectory(src)
    if(EMBER_BUILD_TESTS)
        enable_testing()
        add_subdirectory(tests)
    endif()
    if(EMBER_BUILD_BENCH)
        add_subdirectory(benchmarks)
    endif()

---

    # src/CMakeLists.txt

    add_library(ember_core STATIC
        tensor.cpp
        attention.cpp
        ffn.cpp
        model.cpp
    )

    add_library(Ember::Core ALIAS ember_core)

    target_include_directories(ember_core
        PUBLIC
            $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include>
            $<INSTALL_INTERFACE:include>
        PRIVATE
            ${CMAKE_CURRENT_SOURCE_DIR}
    )

    target_compile_features(ember_core PUBLIC cxx_std_20)

    target_compile_options(ember_core PRIVATE
        $<$<NOT:$<CXX_COMPILER_ID:MSVC>>:
            -Wall -Wextra -Wpedantic
            $<$<CONFIG:Debug>:-g3 -O0>
            $<$<CONFIG:Release>:-O3 -march=native>
        >
    )

    if(EMBER_ENABLE_AVX2)
        include(CheckCXXCompilerFlag)
        check_cxx_compiler_flag("-mavx2" HAVE_AVX2)
        if(HAVE_AVX2)
            target_compile_options(ember_core PRIVATE -mavx2 -mfma)
            target_compile_definitions(ember_core PRIVATE EMBER_AVX2=1)
        endif()
    endif()

    if(EMBER_ENABLE_ASAN)
        target_compile_options(ember_core PUBLIC -fsanitize=address -fno-omit-frame-pointer)
        target_link_options(ember_core PUBLIC -fsanitize=address)
    endif()

    target_link_libraries(ember_core
        PUBLIC  Threads::Threads
        PRIVATE fmt::fmt
    )

    add_executable(ember_run main.cpp)
    target_link_libraries(ember_run PRIVATE Ember::Core)

---

## 41. Structuring for Reuse — The Library Author Pattern

When writing a library meant to be consumed both as a subproject (add_subdirectory) and as an installed package (find_package), follow this pattern:

1. Always create an ALIAS target with the namespace (MyLib::MyLib). Internal consumers use this; it matches the installed name.
2. Use $<BUILD_INTERFACE:...> and $<INSTALL_INTERFACE:...> for include paths.
3. Use GNUInstallDirs for install destinations.
4. Generate and install a config file + version file.
5. Never use absolute paths in INTERFACE properties without genexes.
6. Never use CMAKE_SOURCE_DIR; always use PROJECT_SOURCE_DIR or CMAKE_CURRENT_SOURCE_DIR.

---

## 42. Suppressing Third-Party Warnings

When including third-party code via add_subdirectory or FetchContent, suppress their warnings:

    FetchContent_Declare(somelib ...)
    FetchContent_MakeAvailable(somelib)

    # After MakeAvailable, patch the target if needed:
    if(TARGET somelib)
        target_compile_options(somelib PRIVATE
            $<$<NOT:$<CXX_COMPILER_ID:MSVC>>:-w>
            $<$<CXX_COMPILER_ID:MSVC>:/w>
        )
    endif()

Or set this before FetchContent:

    set(CMAKE_COMPILE_WARNING_AS_ERROR OFF)

Or use SYSTEM includes:

    target_include_directories(my_target SYSTEM PRIVATE
        ${SOMELIB_INCLUDE_DIR}
    )

SYSTEM tells the compiler to suppress warnings from those headers (-isystem on GCC/Clang).

---

## 43. Link-Time Optimization (LTO)

    include(CheckIPOSupported)
    check_ipo_supported(RESULT ipo_supported OUTPUT ipo_output)

    if(ipo_supported)
        set_property(TARGET my_lib PROPERTY INTERPROCEDURAL_OPTIMIZATION TRUE)
        set_property(TARGET my_app PROPERTY INTERPROCEDURAL_OPTIMIZATION TRUE)
    else()
        message(WARNING "LTO not supported: ${ipo_output}")
    endif()

Or globally:

    if(ipo_supported)
        set(CMAKE_INTERPROCEDURAL_OPTIMIZATION_RELEASE TRUE)
    endif()

---

## 44. Symbol Visibility

For shared libraries, hide symbols by default and export only intentional ones:

    set_target_properties(my_lib PROPERTIES
        CXX_VISIBILITY_PRESET     hidden
        VISIBILITY_INLINES_HIDDEN ON
    )

    include(GenerateExportHeader)
    generate_export_header(my_lib
        EXPORT_MACRO_NAME    MY_LIB_API
        EXPORT_FILE_NAME     include/my_lib/export.hpp
    )

In your headers:

    #include "my_lib/export.hpp"

    class MY_LIB_API MyClass { ... };
    MY_LIB_API void my_function();

---

# PART 18: QUICK REFERENCE

---

## 45. Command Cheat Sheet

**Configure:**

    cmake -S . -B build
    cmake -S . -B build -G Ninja
    cmake -S . -B build -DCMAKE_BUILD_TYPE=Release
    cmake --preset debug

**Build:**

    cmake --build build
    cmake --build build --config Release
    cmake --build build -j$(nproc)
    cmake --build build --target my_target
    cmake --build build --verbose

**Test:**

    ctest --test-dir build
    ctest --test-dir build --output-on-failure -j4
    ctest --test-dir build -R "pattern"

**Install:**

    cmake --install build
    cmake --install build --prefix ~/local
    cmake --install build --component devel

**Clean:**

    cmake --build build --target clean

**List targets:**

    cmake --build build --target help

**File operations:**

    cmake -E copy src dst
    cmake -E copy_directory src dst
    cmake -E make_directory path
    cmake -E remove file
    cmake -E rename old new
    cmake -E tar czf archive.tar.gz files...

---

## 46. Variable Quick Reference

    CMAKE_SOURCE_DIR              top-level source dir
    CMAKE_BINARY_DIR              top-level build dir
    CMAKE_CURRENT_SOURCE_DIR      current CMakeLists.txt source dir
    CMAKE_CURRENT_BINARY_DIR      current CMakeLists.txt build dir
    PROJECT_SOURCE_DIR            most-recent project() source dir
    PROJECT_BINARY_DIR            most-recent project() build dir
    PROJECT_NAME                  from project()
    PROJECT_VERSION               from project()
    CMAKE_BUILD_TYPE              Debug/Release/etc
    CMAKE_INSTALL_PREFIX          install root
    CMAKE_CXX_COMPILER            path to C++ compiler
    CMAKE_CXX_COMPILER_ID         GNU/Clang/MSVC/AppleClang
    CMAKE_CXX_COMPILER_VERSION    version string
    CMAKE_GENERATOR               Ninja/Unix Makefiles/etc
    BUILD_SHARED_LIBS             ON = default SHARED libraries
    CMAKE_EXPORT_COMPILE_COMMANDS ON = generate compile_commands.json
    CMAKE_MODULE_PATH             list of dirs to search for Find modules
    CMAKE_PREFIX_PATH             list of dirs for find_package/find_*

---

## 47. Anti-Patterns to Avoid

These are things you must stop doing if you see them in any code, including examples online. They are old CMake idioms that break the target model.

    # WRONG — global, leaks everywhere
    include_directories(include/)
    link_directories(/usr/local/lib)
    add_definitions(-DFOO=1)
    add_compile_options(-Wall)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall")

    # WRONG — sets properties globally, not per-target
    set(CMAKE_CXX_STANDARD 17)     # acceptable in app CMakeLists.txt, bad in library
    link_libraries(somelib)         # global — affects ALL subsequent targets

    # WRONG — should be find_package + imported targets
    find_library(SOMELIB somelib)
    include_directories(${SOMELIB_INCLUDE_DIRS})
    target_link_libraries(app ${SOMELIB_LIBRARIES})   # use imported target instead

    # WRONG — in-source builds
    cd myproject && cmake .        # pollutes source tree

    # WRONG — globbing source files
    file(GLOB SOURCES "src/*.cpp")
    add_executable(app ${SOURCES})

The globbing issue deserves elaboration. file(GLOB) captures the file list at configure time. If you add a new .cpp file, CMake does not know to re-run; the build system thinks nothing changed and your new file is silently excluded. List sources explicitly, or use file(GLOB) only in combination with a configure_file trick that forces reconfiguration. CMake 3.12+ has CONFIGURE_DEPENDS on GLOB which solves this, but it has performance costs. Explicit lists are still the cleanest approach.

    # BETTER (explicit)
    add_executable(app
        src/main.cpp
        src/core.cpp
        src/utils.cpp
    )

    # ACCEPTABLE (with CONFIGURE_DEPENDS)
    file(GLOB_RECURSE SOURCES CONFIGURE_DEPENDS "src/*.cpp")

---

## 48. The Ten Rules of Modern CMake

1. Think in targets, not in global state. Every setting belongs to a target.
2. Use PRIVATE/PUBLIC/INTERFACE correctly. It controls propagation.
3. Never use include_directories, add_definitions, or add_compile_options globally. Always use the target_ variants.
4. Always use $<BUILD_INTERFACE:...>/$<INSTALL_INTERFACE:...> for include paths in libraries.
5. Always use GNUInstallDirs for install destinations.
6. Use imported targets (Namespace::Name) from find_package, never raw variables.
7. Use ALIAS targets to make your targets look the same to both internal and external consumers.
8. Prefer FetchContent over manual add_subdirectory for third-party dependencies.
9. Set cmake_minimum_required with a version range. Activate modern policies.
10. Use CMakePresets.json for anything beyond trivial projects. Commit it.

---

This document covers every concept a beginner through intermediate CMake user needs to write clean, modern, portable, and reusable build systems. The target model — PRIVATE/PUBLIC/INTERFACE propagation, imported targets, usage requirements — is the conceptual heart of everything. Once that clicks, all of the rest is just API surface.