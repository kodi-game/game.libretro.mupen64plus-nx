project(game.libretro.mupen64plus)

cmake_minimum_required(VERSION 2.6)

list(APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR})

enable_language(CXX)

find_package(kodi REQUIRED)
find_package(mupen64plus REQUIRED)

set(DEPLIBS mupen64plus)
set(MUPEN64PLUS_CUSTOM_BINARY ${MUPEN64PLUS_LIB}
                              game.libretro.mupen64plus${CMAKE_SHARED_LIBRARY_SUFFIX} mupen64plus)

build_addon(game.libretro.mupen64plus MUPEN64PLUS DEPLIBS)
