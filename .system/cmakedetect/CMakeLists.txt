cmake_minimum_required(VERSION 3.12.4)
cmake_policy(SET CMP0054 NEW)

if (DG_QUIET)
  #override normal message function and block anything except errors and warnings
  function(message)
    list(GET ARGV 0 MessageType)
    if(MessageType STREQUAL FATAL_ERROR OR
       MessageType STREQUAL SEND_ERROR OR
       MessageType STREQUAL WARNING OR
       MessageType STREQUAL AUTHOR_WARNING)
      list(REMOVE_AT ARGV 0)
      _message(${MessageType} "${ARGV}")
    endif()
  endfunction()
endif()

project(DETECT)

set(output_filename "cmakecfg.txt")

set(BUILD_SHARED_LIBS ON)
if(NOT CMAKE_BUILD_TYPE)
  #Note that asserts are still triggered when running RELWITHDEBINFO
  set(CMAKE_BUILD_TYPE "RELWITHDEBINFO")#other options are DEBUG, RELEASE, MINSIZEREL, RELWITHDEBINFO
endif()

#We compile with -pedantic and -Werror unless DG_RELAX is set:
set(FLAG_PEDANTIC "")
set(FLAG_WERROR "")
if (NOT DG_RELAX)
  set(FLAG_PEDANTIC "-pedantic ")
  set(FLAG_WERROR "-Werror ")
endif()

set(FLAGS_PLATFORM_C "")
set(FLAGS_PLATFORM_CXX "")
if (CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  if (CMAKE_CXX_COMPILER_VERSION VERSION_GREATER 6.99)
    #allow "define private public" for now
    set(FLAGS_PLATFORM_CXX " -Wno-reserved-id-macro -Wno-keyword-macro")
  endif()
endif()

#NB: Using -std=c++1z rather than -std=c++17 for now, due to clang++ from XCode 9.2 not recognising it yet.
set(DG_GLOBAL_COMPILE_FLAGS_CXX "${FLAG_PEDANTIC}-std=c++1z -Wextra ${FLAG_WERROR}-Wall -Woverloaded-virtual -Wno-non-virtual-dtor -Wno-long-long -Wwrite-strings -Wpointer-arith -Wno-variadic-macros -Wshadow${FLAGS_PLATFORM_CXX} ${DG_EXTRA_CFLAGS}")
set(DG_GLOBAL_COMPILE_FLAGS_C "${FLAG_PEDANTIC}-std=c99 -Wextra${FLAGS_PLATFORM_C} ${FLAG_WERROR}-Wall ${DG_EXTRA_CFLAGS}")


set(DG_GLOBAL_COMPILE_FLAGS_Fortran "")
set(DG_GLOBAL_LINK_FLAGS "${DG_EXTRA_LDFLAGS}")
set(DG_GLOBAL_LINK_FLAGS_PREPENDED "")
if ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
  #Ubuntu has --as-needed by default and GCC, so we overrule this (prepended, before specific libs appear on the link cmd):
  set(DG_GLOBAL_LINK_FLAGS_PREPENDED "-Wl,--no-as-needed")
  #With GCC we can (since v4.6) make sure the compiler stops after after a few
  #errors, which is a bit more friendly for most users:
  set(DG_GLOBAL_COMPILE_FLAGS_CXX "${DG_GLOBAL_COMPILE_FLAGS_CXX} -fmax-errors=3")
  set(DG_GLOBAL_COMPILE_FLAGS_C "${DG_GLOBAL_COMPILE_FLAGS_C} -fmax-errors=3")
endif()

#To be filled with versions of global ext deps (Python):
set(DG_GLOBAL_VERSION_DEPS_CXX "")
set(DG_GLOBAL_VERSION_DEPS_C "")
set(DG_GLOBAL_VERSION_DEPS_Fortran "")

include("Utils.cmake")
include("Detect.cmake")
include("WriteResults.cmake")
