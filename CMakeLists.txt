cmake_minimum_required(VERSION 3.1)
project(game.libretro.mupen64plus)

list(APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR})

find_package(Kodi REQUIRED)
find_library(MUPEN64PLUS_LIB NAMES mupen64plus_libretro${CMAKE_SHARED_LIBRARY_SUFFIX} PATH_SUFFIXES libretro)

set(MUPEN64PLUS_CUSTOM_BINARY ${MUPEN64PLUS_LIB}
    ${PROJECT_NAME}${CMAKE_SHARED_LIBRARY_SUFFIX})

build_addon(${PROJECT_NAME} MUPEN64PLUS "")

include(CPack)
