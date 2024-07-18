cmake_minimum_required(VERSION 3.5)
project(game.libretro.mupen64plus-nx)

list(APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR})

find_package(Kodi REQUIRED)

set(CMAKE_CXX_STANDARD 17)

# CMake on windows only searches for .lib libraries (static library or import library).
# The libretro game library is dynamically loaded so even if an import library would exists it's of no use.
if(CMAKE_SYSTEM_NAME STREQUAL Windows)
  set(_CMAKE_FIND_LIBRARY_SUFFIXES ${CMAKE_FIND_LIBRARY_SUFFIXES})
  set(CMAKE_FIND_LIBRARY_SUFFIXES ${CMAKE_SHARED_LIBRARY_SUFFIX})
endif()
find_library(MUPEN64PLUS-NX_LIB NAMES mupen64plus_next_libretro${CMAKE_SHARED_LIBRARY_SUFFIX} PATH_SUFFIXES libretro)
if(CMAKE_SYSTEM_NAME STREQUAL Windows)
  set(CMAKE_FIND_LIBRARY_SUFFIXES ${_CMAKE_FIND_LIBRARY_SUFFIXES})
endif()

set(MUPEN64PLUS-NX_CUSTOM_BINARY ${MUPEN64PLUS-NX_LIB}
    ${PROJECT_NAME}${CMAKE_SHARED_LIBRARY_SUFFIX})

build_addon(${PROJECT_NAME} MUPEN64PLUS-NX "")

include(CPack)
