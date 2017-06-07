set(SODIUM_LIBRARY_VERSION_MAJOR 4)
set(SODIUM_LIBRARY_VERSION_MINOR 2)
set(DLL_VERSION 4)
set(VERSION 0.${SODIUM_LIBRARY_VERSION_MAJOR}.${SODIUM_LIBRARY_VERSION_MINOR})

set(SODIUM_LIBRARY_VERSION=7:0:3)
#                      | | |
#               +------+ | +---+
#               |        |     |
#            current:revision:age
#               |        |     |
#               |        |     +- increment if interfaces have been added
#               |        |        set to zero if interfaces have been removed
#               |        |        or changed
#               |        +- increment if source code has changed
#               |           set to zero if current is incremented
#               +- increment if interfaces have been added, removed or changed


if(CMAKE_BUILD_TYPE MATCHES debug)
    message(STATUS "Enabling DEBUG build")
    set(ENABLE_DEBUG 1)
endif()

# ******************************************************************************
# ************************** MACRO DEFINITION **********************************
# ******************************************************************************

include(CheckCCompilerFlag)

macro(PROJECT_CHECK_COMPILER_FLAGS)
    set(CF)

    foreach(flag ${ARGN})
        set(FLAG_NAME   "-${flag}")
        set(FLAG_OPTION "_${flag}")

        check_c_compiler_flag("${FLAG_NAME}" ${FLAG_OPTION})
        if(${FLAG_OPTION})
            set(CF "${CF} ${FLAG_NAME}")
        #    message(STATUS "Enabling compiler flag '${flag}'")
        #else()
        #    message(STATUS "Not Enabling compiler flag '${flag}'")
        endif()

    endforeach(flag)

    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${CF}")
endmacro(PROJECT_CHECK_COMPILER_FLAGS)

# ******************************************************************************
# **************************  ENDIANESS TESTS  *********************************
# ******************************************************************************

include(TestBigEndian)

if(CMAKE_TEST_ENDIANESS_STRINGS_LE)
    add_definitions("-DNATIVE_LITTLE_ENDIAN=1")
endif()

if(CMAKE_TEST_ENDIANESS_STRINGS_BE)
    add_definitions("-DNATIVE_BIG_ENDIAN=1")
endif()

# ******************************************************************************
# ************************ COMPILE CHECKS CHECKS *******************************
# ******************************************************************************

project_check_compiler_flags(
    std=c99
    fPIC fvisibility=hidden
    fwrapv
    fno-strict-aliasing
    fno-strict-overflow
)

set(__TMP ${CMAKE_REQUIRED_LIBRARIES})
set(CMAKE_REQUIRED_LIBRARIES "m")
check_c_source_compiles(
    "
    #include <fenv.h>
    #pragma STDC FENV_ACCESS ON

    int main(void) {
    const int previous_rounding_mode = fegetround();
    fesetround(FE_TONEAREST);
    fesetround(previous_rounding_mode);
    };
    "
    AVAIL_FENV_H
)
set(CMAKE_REQUIRED_LIBRARIES ${__TMP})

# test wether it is a x86 machine and as/gas is available
set(CAN_USE_ASSEMBLER FALSE)
set(CMAKE_ASM_SOURCE_FILE_EXTENSIONS "s;S")
enable_language(ASM)
if(CMAKE_ASM_COMPILER_WORKS)
    set(CAN_USE_ASSEMBLER TRUE)
    message(STATUS "ASM compiler found")
else()
    message(STATUS "ASM compiler not found")
endif()

if(CAN_USE_ASSEMBLER)
    check_c_source_compiles(
        "
        int main(void) {
        #if defined(__amd64) || defined(__amd64__) || defined(__x86_64__)
        /* neat */
        #else
        # error !amd64
        #endif
        __asm__(\"pxor %xmm12,%xmm6\");
        };
        "
        AVAIL_AMD64_ASM
    )
endif()

check_c_source_compiles(
    "
    int main(void) {
    #ifndef __GNUC__
    # error mode(TI) is a gcc extension
    #endif
    #if defined(__clang__) && !defined(__x86_64__)
    # error clang does not properly compile smult_curve25519_donna_c64.c
    #endif
    typedef unsigned uint128_t __attribute__((mode(TI)));
    uint128_t x;
    (void) x;
    };
    "
    AVAIL_TI_MODE
)

if(AVAIL_FENV_H)
    message(STATUS "* Floating-point rounding mode is accessible")
endif()

if(AVAIL_AMD64_ASM)
    message(STATUS "* Basic amd64 code can be assembled")
endif()

if(AVAIL_TI_MODE)
    message(STATUS "* GCC TI mode is available")
endif()

# ******************************************************************************
# *********************** COMPILER FLAGS CHECKS ********************************
# ******************************************************************************

add_definitions("-D_FORTIFY_SOURCE=2")

project_check_compiler_flags(Wall Wdiv-by-zero Wextra Winit-self
  Wsometimes-uninitialized Wstrict-prototypes Wunused Wwrite-strings)

# ******************************************************************************
# ********************* COMMAND LINE FLAGS CHECKS ******************************
# ******************************************************************************

if(ENABLE_DEBUG)
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -O0 -g3")
    add_definitions("-DDEBUG=1")
    project_check_compiler_flags(
        Wno-unknown-warning-option
        Wbad-function-cast
        Wcast-align
        Wcast-qual
        Wchar-subscripts
        Wcomment
        Wfloat-equal
        Wformat=2
        Wimplicit
        Wmissing-declarations
        Wmissing-prototypes
        Wnormalized=id
        Woverride-init
        Wparentheses
        Wpointer-arith
        Wredundant-decls
        Wstrict-prototypes
        Wswitch-enum
        Wvariable-decl
    )
else()
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -O3 -fomit-frame-pointer")
endif()

message(STATUS "---------------------------------------------------")

if(NOT DISABLE_SSP)
    check_c_compiler_flag("-fstack-protector" WITH_SSP)
    check_c_compiler_flag("-fstack-protector-all" WITH_SSP_ALL)
    if(WITH_SSP_ALL)
        message(STATUS "Enabling stack protector-all")
        set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fstack-protector-all")
    else()
        if(WITH_SSP)
            message(STATUS "Enabling stack protector")
            set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fstack-protector")
        endif()
    endif()
else()
    message(STATUS "Disabling SSP")
endif()

#if(NOT DISABLE_PIE)
#    check_c_compiler_flag("-fPIE" WITH_FPIE)
#    if(WITH_FPIE)
#        message(STATUS "Enabling -fPIE")
#        set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fPIE")
#        set(_LINKER_OPTIONS "-pie")
#    endif()
#else()
#    message(STATUS "Disabling PIE")
#endif()

if(NOT DISABLE_ASM AND AVAIL_AMD64_ASM)
    set(HAVE_AMD64_ASM_V 1)
    add_definitions("-DHAVE_AMD64_ASM=1 -DHAVE_AMD64_ASM_V=1")
    message(STATUS "Enabling ASM")
else()
    set(HAVE_AMD64_ASM_V 0)
    message(STATUS "Disabling ASM")
endif()

if(ENABLE_BLOCKING_RANDOM)
    add_definitions("-DUSE_BLOCKING_RANDOM=1")
    message(STATUS "Enabling random blocking")
endif()

if(NOT AVAIL_TI_MODE)
    message(STATUS "Disabling 128-bit arithmetic")
    set(HAVE_TI_MODE_V 0)
else()
    message(STATUS "Enabling 128-bit arithmetic")
    set(HAVE_TI_MODE_V 1)
    add_definitions("-DHAVE_TI_MODE=1")
endif()

# ******************************************************************************
# ******************************* BUILD PATHS **********************************
# ******************************************************************************

if(NOT DEFINED CMAKE_SIZEOF_VOID_P)
    message(AUTHOR_WARNING
        "Unable to determine default CMAKE_INSTALL_LIBDIR directory because no target architecture is known. "
        "Please enable at least one language before including GNUInstallDirs.")
endif()

if(NOT LIBSODIUM_DIR)
    set(LIBSODIUM_DIR /opt/libsodium)
endif()

if(NOT LIBSODIUM_INCLUDE_DIR)
    set(LIBSODIUM_INCLUDE_DIR ${LIBSODIUM_DIR}/include)
endif()

if(NOT LIBSODIUM_LIB_DIR)
    set(LIBSODIUM_LIB_DIR ${LIBSODIUM_DIR}/lib)

    if("${CMAKE_SIZEOF_VOID_P}" EQUAL "4")
        set(LIBSODIUM_LIB_DIR ${LIBSODIUM_DIR}/lib32)
    endif()

    if("${CMAKE_SIZEOF_VOID_P}" EQUAL "8")
        set(LIBSODIUM_LIB_DIR ${LIBSODIUM_DIR}/lib64)
    endif()
endif()

message(STATUS "---------------------------------------------------")
message(STATUS "LIBSODIUM PREFIX:      " ${LIBSODIUM_DIR})
message(STATUS "LIBSODIUM INCLUDE DIR: " ${LIBSODIUM_INCLUDE_DIR})
message(STATUS "LIBSODIUM LIB DIR:     " ${LIBSODIUM_LIB_DIR})

add_subdirectory(libsodium)
