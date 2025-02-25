#
# SPDX-License-Identifier: BSD-3-Clause
#
# Copyright © 2022 Keith Packard
#
# Redistribution and use in source and binary forms, with or without
# modification, are permitted provided that the following conditions
# are met:
#
# 1. Redistributions of source code must retain the above copyright
#    notice, this list of conditions and the following disclaimer.
#
# 2. Redistributions in binary form must reproduce the above
#    copyright notice, this list of conditions and the following
#    disclaimer in the documentation and/or other materials provided
#    with the distribution.
#
# 3. Neither the name of the copyright holder nor the names of its
#    contributors may be used to endorse or promote products derived
#    from this software without specific prior written permission.
#
# THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS
# "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT
# LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS
# FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE
# COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT,
# INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES
# (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
# SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION)
# HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT,
# STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE)
# ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED
# OF THE POSSIBILITY OF SUCH DAMAGE.
#

cmake_minimum_required(VERSION 3.20.0)

project(Picolibc VERSION 1.7.9 LANGUAGES C ASM)

if(${CMAKE_SYSTEM_PROCESSOR} STREQUAL "arm64")
  set(CMAKE_SYSTEM_PROCESSOR "aarch64")
endif()

set(CMAKE_SYSTEM_SUB_PROCESSOR ${CMAKE_SYSTEM_PROCESSOR})

if(${CMAKE_SYSTEM_PROCESSOR} STREQUAL "i686" OR
    ${CMAKE_SYSTEM_PROCESSOR} STREQUAL "x86_64") 
  set(CMAKE_SYSTEM_PROCESSOR "x86")
endif()

if(ZEPHYR_BASE)
  if(NOT CONFIG_PICOLIBC_USE_MODULE)
    return()
  endif()
  include(zephyr/zephyr.cmake)
endif()

include(cmake/picolibc.cmake)

enable_testing()

set(CMAKE_TRY_COMPILE_TARGET_TYPE STATIC_LIBRARY)

# Set all configure values to defaults for now

# Use atomics for fgetc/ungetc for re-entrancy
set(ATOMIC_UNGETC 1)

# Always optimize strcmp for performance
if(NOT DEFINED FAST_STRCMP)
  option(FAST_STRCMP "Always optimize strcmp for performance" ON)
endif()

# Obsoleted. Use regular syscalls
set(MISSING_SYSCALL_NAMES 0)

# use global errno variable
if(NOT DEFINED NEWLIB_GLOBAL_ERRNO)
  option(NEWLIB_GLOBAL_ERRNO "use global errno variable" OFF)
endif()

# use thread local storage
if(NOT DEFINED PICOLIBC_TLS)
  option(PICOLIBC_TLS "use thread local storage for static data" ON)
endif()

# use thread local storage
set(NEWLIB_TLS ${PICOLIBC_TLS})

# Use open/close/read/write in tinystdio
option(POSIX_IO "Provide fopen/fdopen using POSIX I/O (open, close, read, write, lseek)" ON)

option(POSIX_CONSOLE "Use POSIX I/O for stdin/stdout/stderr" OFF)

# Optimize for space over speed

if(NOT DEFINED PREFER_SIZE_OVER_SPEED)
  if(CMAKE_BUILD_TYPE STREQUAL "MinSizeRel")
    set(default_prefer_size 1)
  else()
    set(default_prefer_size 0)
  endif()
  option(PREFER_SIZE_OVER_SPEED ${default_prefer_size})
endif()

# Obsoleted. Reentrant syscalls provided for us
set(REENTRANT_SYSCALLS_PROVIDED 0)

# Use tiny stdio from gcc avr
option(TINY_STDIO "Use tiny stdio from avr libc" ON)

if(NOT TLS_MODEL)
  set(TLS_MODEL "local-exec")
endif()

set(_ATEXIT_DYNAMIC_ALLOC 0)

set(_FSEEK_OPTIMIZATION 0)

set(_FVWRITE_IN_STREAMIO 0)

try_compile(_HAVE_ALIAS_ATTRIBUTE ${PROJECT_BINARY_DIR} ${CMAKE_CURRENT_SOURCE_DIR}/cmake/have-alias-attribute.c)

# The compiler REALLY has the attribute __alloc_size__
try_compile(_HAVE_ALLOC_SIZE ${PROJECT_BINARY_DIR} ${CMAKE_CURRENT_SOURCE_DIR}/cmake/have-alloc-size.c)

# The compiler supports the always_inline function attribute
try_compile(_HAVE_ATTRIBUTE_ALWAYS_INLINE ${PROJECT_BINARY_DIR} ${CMAKE_CURRENT_SOURCE_DIR}/cmake/have-attribute-always-inline.c)

# The compiler supports the gnu_inline function attribute
try_compile(_HAVE_ATTRIBUTE_GNU_INLINE  ${PROJECT_BINARY_DIR} ${CMAKE_CURRENT_SOURCE_DIR}/cmake/have-attribute-gnu-inline.c)

# Use bitfields in packed structs
set(_HAVE_BITFIELDS_IN_PACKED_STRUCTS 1)

# The compiler supports __builtin_alloca
set(_HAVE_BUILTIN_ALLOCA 1)

# The compiler supports __builtin_copysign
set(_HAVE_BUILTIN_COPYSIGN 1)

# The compiler supports __builtin_copysignl
set(_HAVE_BUILTIN_COPYSIGNL 1)

# The compiler supports __builtin_ctz
set(_HAVE_BUILTIN_CTZ 1)

# The compiler supports __builtin_ctzl
set(_HAVE_BUILTIN_CTZL 1)

# The compiler supports __builtin_ctzll
set(_HAVE_BUILTIN_CTZLL 1)

# Compiler has __builtin_expect
set(_HAVE_BUILTIN_EXPECT 1)

# The compiler supports __builtin_ffs
set(_HAVE_BUILTIN_FFS 1)

# The compiler supports __builtin_ffsl
set(_HAVE_BUILTIN_FFSL 1)

# The compiler supports __builtin_ffsll
set(_HAVE_BUILTIN_FFSLL 1)

# The compiler supports __builtin_finitel
set(_HAVE_BUILTIN_FINITEL 1)

# The compiler supports __builtin_isfinite
set(_HAVE_BUILTIN_ISFINITE 1)

# The compiler supports __builtin_isinf
set(_HAVE_BUILTIN_ISINF 1)

# The compiler supports __builtin_isinfl
set(_HAVE_BUILTIN_ISINFL 1)

# The compiler supports __builtin_isnan
set(_HAVE_BUILTIN_ISNAN 1)

# The compiler supports __builtin_isnanl
set(_HAVE_BUILTIN_ISNANL 1)

# Compiler has __builtin_mul_overflow
set(_HAVE_BUILTIN_MUL_OVERFLOW 1)

# Compiler flag to prevent detecting memcpy/memset patterns
set(_HAVE_CC_INHIBIT_LOOP_TO_LIBCALL 1)

# Compiler supports _Complex
set(_HAVE_COMPLEX 1)

set(_HAVE_FCNTL 0)

set(_HAVE_FORMAT_ATTRIBUTE 1)

# IEEE fp funcs available
set(_HAVE_IEEEFP_FUNCS 0)

# compiler supports INIT_ARRAY sections
set(_HAVE_INITFINI_ARRAY 1)

# Support _init() and _fini() functions
set(_HAVE_INIT_FINI 1)

# Compiler has long double type
try_compile(_HAVE_LONG_DOUBLE ${PROJECT_BINARY_DIR} ${CMAKE_CURRENT_SOURCE_DIR}/cmake/have-long-double.c)

# Compiler attribute to prevent the optimizer from adding new builtin calls
set(_HAVE_NO_BUILTIN_ATTRIBUTE 0)

# _set_tls and _init_tls functions available
if(NOT DEFINED _HAVE_PICOLIBC_TLS_API)
  set(_HAVE_PICOLIBC_TLS_API 0)
endif()

set(_HAVE_SEMIHOST 1)
set(_HAVE_WEAK_ATTRIBUTE 1)
set(_ICONV_ENABLE_EXTERNAL_CCS 0)
set(_ICONV_FROM_ENCODING_ 1)
set(_ICONV_FROM_ENCODING_BIG5 1)
set(_ICONV_FROM_ENCODING_CP775 1)
set(_ICONV_FROM_ENCODING_CP850 1)
set(_ICONV_FROM_ENCODING_CP852 1)
set(_ICONV_FROM_ENCODING_CP855 1)
set(_ICONV_FROM_ENCODING_CP866 1)
set(_ICONV_FROM_ENCODING_EUC_JP 1)
set(_ICONV_FROM_ENCODING_EUC_KR 1)
set(_ICONV_FROM_ENCODING_EUC_TW 1)
set(_ICONV_FROM_ENCODING_ISO_8859_1 1)
set(_ICONV_FROM_ENCODING_ISO_8859_10 1)
set(_ICONV_FROM_ENCODING_ISO_8859_11 1)
set(_ICONV_FROM_ENCODING_ISO_8859_13 1)
set(_ICONV_FROM_ENCODING_ISO_8859_14 1)
set(_ICONV_FROM_ENCODING_ISO_8859_15 1)
set(_ICONV_FROM_ENCODING_ISO_8859_2 1)
set(_ICONV_FROM_ENCODING_ISO_8859_3 1)
set(_ICONV_FROM_ENCODING_ISO_8859_4 1)
set(_ICONV_FROM_ENCODING_ISO_8859_5 1)
set(_ICONV_FROM_ENCODING_ISO_8859_6 1)
set(_ICONV_FROM_ENCODING_ISO_8859_7 1)
set(_ICONV_FROM_ENCODING_ISO_8859_8 1)
set(_ICONV_FROM_ENCODING_ISO_8859_9 1)
set(_ICONV_FROM_ENCODING_ISO_IR_111 1)
set(_ICONV_FROM_ENCODING_KOI8_R 1)
set(_ICONV_FROM_ENCODING_KOI8_RU 1)
set(_ICONV_FROM_ENCODING_KOI8_U 1)
set(_ICONV_FROM_ENCODING_KOI8_UNI 1)
set(_ICONV_FROM_ENCODING_UCS_2 1)
set(_ICONV_FROM_ENCODING_UCS_2BE 1)
set(_ICONV_FROM_ENCODING_UCS_2LE 1)
set(_ICONV_FROM_ENCODING_UCS_2_INTERNAL 1)
set(_ICONV_FROM_ENCODING_UCS_4 1)
set(_ICONV_FROM_ENCODING_UCS_4BE 1)
set(_ICONV_FROM_ENCODING_UCS_4LE 1)
set(_ICONV_FROM_ENCODING_UCS_4_INTERNAL 1)
set(_ICONV_FROM_ENCODING_US_ASCII 1)
set(_ICONV_FROM_ENCODING_UTF_16 1)
set(_ICONV_FROM_ENCODING_UTF_16BE 1)
set(_ICONV_FROM_ENCODING_UTF_16LE 1)
set(_ICONV_FROM_ENCODING_UTF_8 1)
set(_ICONV_FROM_ENCODING_WIN_1250 1)
set(_ICONV_FROM_ENCODING_WIN_1251 1)
set(_ICONV_FROM_ENCODING_WIN_1252 1)
set(_ICONV_FROM_ENCODING_WIN_1253 1)
set(_ICONV_FROM_ENCODING_WIN_1254 1)
set(_ICONV_FROM_ENCODING_WIN_1255 1)
set(_ICONV_FROM_ENCODING_WIN_1256 1)
set(_ICONV_FROM_ENCODING_WIN_1257 1)
set(_ICONV_FROM_ENCODING_WIN_1258 1)
set(_ICONV_TO_ENCODING_ 1)
set(_ICONV_TO_ENCODING_BIG5 1)
set(_ICONV_TO_ENCODING_CP775 1)
set(_ICONV_TO_ENCODING_CP850 1)
set(_ICONV_TO_ENCODING_CP852 1)
set(_ICONV_TO_ENCODING_CP855 1)
set(_ICONV_TO_ENCODING_CP866 1)
set(_ICONV_TO_ENCODING_EUC_JP 1)
set(_ICONV_TO_ENCODING_EUC_KR 1)
set(_ICONV_TO_ENCODING_EUC_TW 1)
set(_ICONV_TO_ENCODING_ISO_8859_1 1)
set(_ICONV_TO_ENCODING_ISO_8859_10 1)
set(_ICONV_TO_ENCODING_ISO_8859_11 1)
set(_ICONV_TO_ENCODING_ISO_8859_13 1)
set(_ICONV_TO_ENCODING_ISO_8859_14 1)
set(_ICONV_TO_ENCODING_ISO_8859_15 1)
set(_ICONV_TO_ENCODING_ISO_8859_2 1)
set(_ICONV_TO_ENCODING_ISO_8859_3 1)
set(_ICONV_TO_ENCODING_ISO_8859_4 1)
set(_ICONV_TO_ENCODING_ISO_8859_5 1)
set(_ICONV_TO_ENCODING_ISO_8859_6 1)
set(_ICONV_TO_ENCODING_ISO_8859_7 1)
set(_ICONV_TO_ENCODING_ISO_8859_8 1)
set(_ICONV_TO_ENCODING_ISO_8859_9 1)
set(_ICONV_TO_ENCODING_ISO_IR_111 1)
set(_ICONV_TO_ENCODING_KOI8_R 1)
set(_ICONV_TO_ENCODING_KOI8_RU 1)
set(_ICONV_TO_ENCODING_KOI8_U 1)
set(_ICONV_TO_ENCODING_KOI8_UNI 1)
set(_ICONV_TO_ENCODING_UCS_2 1)
set(_ICONV_TO_ENCODING_UCS_2BE 1)
set(_ICONV_TO_ENCODING_UCS_2LE 1)
set(_ICONV_TO_ENCODING_UCS_2_INTERNAL 1)
set(_ICONV_TO_ENCODING_UCS_4 1)
set(_ICONV_TO_ENCODING_UCS_4BE 1)
set(_ICONV_TO_ENCODING_UCS_4LE 1)
set(_ICONV_TO_ENCODING_UCS_4_INTERNAL 1)
set(_ICONV_TO_ENCODING_US_ASCII 1)
set(_ICONV_TO_ENCODING_UTF_16 1)
set(_ICONV_TO_ENCODING_UTF_16BE 1)
set(_ICONV_TO_ENCODING_UTF_16LE 1)
set(_ICONV_TO_ENCODING_UTF_8 1)
set(_ICONV_TO_ENCODING_WIN_1250 1)
set(_ICONV_TO_ENCODING_WIN_1251 1)
set(_ICONV_TO_ENCODING_WIN_1252 1)
set(_ICONV_TO_ENCODING_WIN_1253 1)
set(_ICONV_TO_ENCODING_WIN_1254 1)
set(_ICONV_TO_ENCODING_WIN_1255 1)
set(_ICONV_TO_ENCODING_WIN_1256 1)
set(_ICONV_TO_ENCODING_WIN_1257 1)
set(_ICONV_TO_ENCODING_WIN_1258 1)

# math library does not set errno (offering only ieee semantics)
set(_IEEE_LIBM 1)

if(NOT DEFINED _IO_FLOAT_EXACT)
  option(_IO_FLOAT_EXACT "Provide exact binary/decimal conversion for printf/scanf" 1)
endif()

# is long double the same as double
try_compile(_LDBL_EQ_DBL ${PROJECT_BINARY_DIR} ${CMAKE_CURRENT_SOURCE_DIR}/cmake/ldbl-eq-dbl.c)

set(_LITE_EXIT 1)

if(NOT DEFINED _MB_CAPABLE)
  option(_MB_CAPABLE "Enable multi-byte support" 0)
endif()

set(_MB_LEN_MAX 1 1)

set(_NANO_FORMATTED_IO 0)

option(_NANO_MALLOC "Use smaller malloc implementation" 1)

set(_REENT_GLOBAL_ATEXIT 0)

set(_UNBUF_STREAM_OPT 0)

if(NOT DEFINED _WANT_IO_C99_FORMATS)
  option(_WANT_IO_C99_FORMATS "Support C99 formats in printf/scanf" ON)
endif()

if(NOT DEFINED _WANT_IO_LONG_LONG)
  option(_WANT_IO_LONG_LONG "Support long long in integer printf/scanf" OFF)
endif()

if(NOT DEFINED _WANT_IO_POS_ARGS)
  option(_WANT_IO_POS_ARGS "Support positional args in integer printf/scanf" OFF)
endif()

option(__IO_FLOAT "Support floating point in printf/scanf by default" OFF)

if(NOT DEFINED FORMAT_DEFAULT_DOUBLE)
  set(FORMAT_DEFAULT_DOUBLE ${__IO_FLOAT})
endif()

if(NOT DEFINED FORMAT_DEFAULT_FLOAT)
  set(FORMAT_DEFAULT_FLOAT 0)
endif()

if(NOT DEFINED FORMAT_DEFAULT_INTEGER)
  set(FORMAT_DEFAULT_INTEGER NOT ${__IO_FLOAT})
endif()

# math library sets errno
set(_WANT_MATH_ERRNO 0)

set(_WANT_REENT_SMALL 0)

set(_WANT_REGISTER_FINI 0)

# Obsoleted. Define time_t to long instead of using a 64-bit type
set(_WANT_USE_LONG_TIME_T 0)

set(_WIDE_ORIENT 0)

# locale support
if(NOT DEFINED __HAVE_LOCALE_INFO__)
  option(__HAVE_LOCALE_INFO__ "Provide locale support" 0)
endif()

# extended locale support
if(NOT DEFINED __HAVE_LOCALE_INFO_EXTENDED__)
  option(__HAVE_LOCALE_INFO_EXTENDED__ "Provide even more locale support" 0)
endif()

# Use old math code for double funcs (0 no, 1 yes)
set(__OBSOLETE_MATH_FLOAT 1)

# Use old math code for double funcs (0 no, 1 yes)
set(__OBSOLETE_MATH_DOUBLE 1)

# Compute static memory area sizes at runtime instead of link time
set(__PICOLIBC_CRT_RUNTIME_SIZE 0)

if(NOT DEFINED __SINGLE_THREAD__)
  option(__SINGLE_THREAD__ "Disable multithreading support" 0)
endif()

set(_RETARGETABLE_LOCKING NOT ${__SINGLE_THREAD__})

set(NEWLIB_VERSION 4.3.0)
set(NEWLIB_MAJOR 4)
set(NEWLIB_MINOR 3)
set(NEWLIB_PATCH 0)

set(PICOLIBC_INCLUDE ${PROJECT_BINARY_DIR}/picolibc/include)

set(PICOLIBC_SCRIPTS "${CMAKE_CURRENT_SOURCE_DIR}/scripts")

include(CheckIncludeFile)

CHECK_INCLUDE_FILE(xtensa/config/core-isa.h _XTENSA_HAVE_CONFIG_CORE_ISA_H)

configure_file(picolibc.h.in "${PICOLIBC_INCLUDE}/picolibc.h")

set(INCLUDEDIR include)
set(LIBDIR .)
set(TLSMODEL "-ftls-model=${TLS_MODEL}")
set(LINK_SPEC "")
set(CC1_SPEC "")
set(CC1PLUS_SPEC "")
set(ADDITIONAL_LIBS "")
set(SPECS_EXTRA "")
set(SPECS_ISYSTEM "-isystem ${PROJECT_BINARY_DIR}/${include}")
set(SPECS_LIBPATH "-L${PROJECT_BINARY_DIR}")
set(SPECS_STARTFILE "${PROJECT_BINARY_DIR}/crt0.o")
string(APPEND SPECS_PRINTF "%{DPICOLIBC_FLOAT_PRINTF_SCANF:--defsym=vfprintf=__f_vfprintf}"
  " %{DPICOLIBC_FLOAT_PRINTF_SCANF:--defsym=vfscanf=__f_vfscanf}"
  " %{DPICOLIBC_DOUBLE_PRINTF_SCANF:--defsym=vfprintf=__d_vfprintf}"
  " %{DPICOLIBC_DOUBLE_PRINTF_SCANF:--defsym=vfscanf=__d_vfscanf}"
  " %{DPICOLIBC_INTEGER_PRINTF_SCANF:--defsym=vfprintf=__i_vfprintf}")
set(PREFIX "${PROJECT_BINARY_DIR}")

configure_file(picolibc.specs.in "${PROJECT_BINARY_DIR}/picolibc.specs" @ONLY)

set(PICOLIBC_COMPILE_OPTIONS
  "--include" "${PICOLIBC_INCLUDE}/picolibc.h"
  "-nostdlib"
  "-fno-common"
  "-frounding-math"
  "-fsignaling-nans"
  "-fno-stack-protector"
  "-ffunction-sections"
  "-fdata-sections"
  "-Wall"
  "-Wextra"
  "-Werror=implicit-function-declaration"
  "-Werror=vla"
  "-Warray-bounds"
  "-Wold-style-definition"
  "-D_LIBC"
  ${TLSMODEL}
  ${TARGET_COMPILE_OPTIONS}
  ${PICOLIBC_EXTRA_COMPILE_OPTIONS}
  )

add_library(c STATIC)

target_compile_options(c PRIVATE ${PICOLIBC_COMPILE_OPTIONS})

set(PICOLIBC_INCLUDE_DIRECTORIES
  "${PICOLIBC_INCLUDE}")

target_include_directories(c SYSTEM PUBLIC ${PICOLIBC_INCLUDE_DIRECTORIES})

define_property(GLOBAL PROPERTY PICOLIBC_HEADERS
  BRIEF_DOCS "Installed header files"
  FULL_DOCS "These are names of header files which are to be installed.")

add_subdirectory(newlib)

install(TARGETS c
  LIBRARY DESTINATION lib
  PUBLIC_HEADER DESTINATION include)

option(TESTS "Enable tests" OFF)
if(TESTS)

  # This could use some generalization, but it's good enough to do
  # semihosting-based tests

  add_subdirectory(semihost)
  add_subdirectory(picocrt)

  set(PICOCRT_OBJ $<TARGET_OBJECTS:picocrt>)
  set(PICOCRT_SEMIHOST_OBJ $<TARGET_OBJECTS:picocrt-semihost>)

  # semihost and libc have mutual-dependencies, so place them in a
  # linker group

  set(PICOLIBC_TEST_LINK_LIBRARIES
    ${PICOLIBC_COMPILE_OPTIONS}
    ${PICOCRT_SEMIHOST_OBJ}
    -Wl,--start-group c semihost -Wl,--end-group
    ${PICOLIBC_LINK_FLAGS}
    )

  add_subdirectory(test)

endif()

install(FILES ${CMAKE_BINARY_DIR}/picolibc.specs DESTINATION lib)
install(FILES ${CMAKE_BINARY_DIR}/picolibc/include/picolibc.h DESTINATION include)
