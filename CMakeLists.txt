cmake_minimum_required(VERSION 3.20.0)
project("QUICKJS" VERSION 24.3.5)
set(QJS_VERSION 2024-02-14)

##### * Configuration You could modify * #####
set(CONFIG_WIN32   ON)                  # Build on Windows
set(CONFIG_SHARED  ON)                  # Build dynamic shared library
set(CONFIG_LTO     OFF)                 # LTO Optimision
set(CONFIG_CLANG   OFF)                 # Use LLVM-Clang instead of GNU-C
set(CONFIG_WERROR  OFF)                 # Warning is Error   
set(CONFIG_FIX_DEPS ON)                 # Auto build missing libs on Windows
set(CONFIG_QJSC_CC         "")          # Options: host target ""
set(CONFIG_TARGET_PLATFORM "x86_64")    # Options: x86_64 i686 aarch64 armv7
##### * Configuration You could modify * #####

# Set Build Output Dir
if(CMAKE_BUILD_TYPE STREQUAL "Debug")
    message(STATUS "Current: Build type is Debug.")
    set(CMAKE_BINARY_DIR "debug/${CONFIG_TARGET_PLATFORM}")
else()
    set(CMAKE_BINARY_DIR "dist/${CONFIG_TARGET_PLATFORM}")
endif()
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

# Compiler Configuration
if(CONFIG_CLANG)
set(CC clang)
else()
set(CC gcc)
endif()
set(CMAKE_C_STANDARD "17")

# Preset for the Windows system platform
if(CONFIG_WIN32)
    if(CMAKE_HOST_WIN32 AND NOT "${MINGW}")
        message(FATAL_ERROR "**MinGW toolkit is required**")
    elseif(CONFIG_FIX_DEPS)
        message(STATUS "Fix-deps: Use dlfcn for lib \"ld\"")
        add_subdirectory("fix-deps")
    endif()
    # Config the cross platform information
    set(DEF_MINGW -D__USE_MINGW_ANSI_STDIO)
    set(CMAKE_C_COMPILER "${CONFIG_TARGET_PLATFORM}-w64-mingw32-${CC}")
endif()

# C flags
set(CMAKE_C_FLAGS "-fPIC -fwrapv -Wall")
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wno-array-bounds -Wno-format-truncation")
if(CONFIG_WERROR)
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Werror")
endif()
if(CONFIG_LTO)
    set(QJSC_LTO "-DCONFIG_LTO")
    set(CMAKE_C_FLAGS "-flto ${CMAKE_C_FLAGS}")
endif()

# Definions
add_definitions(${DEF_MINGW} -D_GNU_SOURCE -DCONFIG_VERSION="${QJS_VERSION}" -DCONFIG_BIGNUM)


# Link options
add_link_options("-Wl,--no-undefined")
if(NOT CMAKE_BUILD_TYPE STREQUAL "Debug")
    add_link_options("-s") # Use strip if not in debug
endif()

# C include dir
include_directories("./include")

# quickjs lib build
add_library(quickjs SHARED 
    "src/rt/cutils.c"
    "src/rt/libbf.c"
    "src/rt/libregexp.c"
    "src/rt/libunicode.c"
    "src/rt/quickjs-libc.c"
    "src/rt/quickjs.c"
)
target_link_libraries(quickjs PUBLIC m pthread dl "-Llib")

# qjsc bin build
if(CONFIG_QJSC_CC STREQUAL "host")
add_compile_definitions(qjsc ${QJSC_LTO} CONFIG_CC=${CMAKE_C_COMPILER})
elseif(CONFIG_QJSC_CC STREQUAL "target")
add_compile_definitions(qjsc ${QJSC_LTO} CONFIG_CC=${CC})
endif()
add_executable(qjsc "src/cli/qjsc.c")
target_link_libraries(qjsc PRIVATE quickjs)

# prebuild repl.c & qjscalc.c
set(REPL_JS_FILE "${CMAKE_SOURCE_DIR}/src/cli/repl.js")
set(REPL_C_FILE "${CMAKE_CACHEFILE_DIR}/gen/repl.c")  # This file requires qjsc to generate
set(CALC_JS_FILE "${CMAKE_SOURCE_DIR}/src/cli/qjscalc.js")
set(CALC_C_FILE "${CMAKE_CACHEFILE_DIR}/gen/qjscalc.c")
add_custom_command(
    OUTPUT ${REPL_C_FILE}
    COMMAND qjsc -c -o ${REPL_C_FILE} -m ${REPL_JS_FILE}
    COMMENT "Generating repl.c file for the qjs binary"
    VERBATIM
)
add_custom_command(
    OUTPUT ${CALC_C_FILE}
    COMMAND qjsc -fbignum -c -o ${CALC_C_FILE} ${CALC_JS_FILE}
    COMMENT "Generating qjcalc.c file for the qjs binary"
    VERBATIM
)

# qjs bin build
add_executable(qjs "src/cli/qjs.c" ${REPL_C_FILE} ${CALC_C_FILE})
target_link_libraries(qjs PRIVATE quickjs)

# run-262test bin build
add_executable(run-test262 "src/cli/run-test262.c")
target_link_libraries(run-test262 PRIVATE quickjs)