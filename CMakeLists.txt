project(game.mupen64plus)

cmake_minimum_required(VERSION 2.6)

list(APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR})

enable_language(CXX)

find_package(kodi REQUIRED)

include(ExternalProject)

externalproject_add(mupen64plus
                    GIT_REPOSITORY https://github.com/libretro/mupen64plus-libretro.git
                    GIT_TAG 429186a
                    PREFIX mupen64plus
                    CONFIGURE_COMMAND ""
                    INSTALL_COMMAND ""
                    BUILD_COMMAND make -C libretro
                    BUILD_IN_SOURCE 1)

set(MUPEN64PLUS_CUSTOM_BINARY ${PROJECT_BINARY_DIR}/mupen64plus/src/mupen64plus/mupen64plus_libretro${CMAKE_SHARED_LIBRARY_SUFFIX}
                              mupen64plus_libretro${CMAKE_SHARED_LIBRARY_SUFFIX} mupen64plus)

build_addon(game.mupen64plus MUPEN64PLUS DEPLIBS)
add_dependencies(game.mupen64plus mupen64plus)
SET_TARGET_PROPERTIES(game.mupen64plus PROPERTIES LINKER_LANGUAGE CXX)
