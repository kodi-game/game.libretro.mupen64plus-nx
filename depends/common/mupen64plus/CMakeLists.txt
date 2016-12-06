cmake_minimum_required(VERSION 3.1)
project(mupen64plus)

list(APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR})

include(ExternalProject)

string(TOUPPER ${PROJECT_NAME} PROJECT_NAME_UC)

set(LIBRETRO_BINARY_DIR .)
set(LIBRETRO_SONAME mupen64plus_libretro${CMAKE_SHARED_LIBRARY_SUFFIX})

if(CMAKE_BUILD_TYPE MATCHES Debug)
  set(LIBRETRO_DEBUG DEBUG=1)
endif()

if(CORE_SYSTEM_NAME STREQUAL windows)
  find_package(MinGW REQUIRED)
  set(BUILD_COMMAND ${MINGW_MAKE} -j$ENV{NUMBER_OF_PROCESSORS} -C . -f Makefile platform=win  ${LIBRETRO_DEBUG})
elseif(CORE_SYSTEM_NAME STREQUAL linux)
  set(BUILD_COMMAND $(MAKE) -C . -f Makefile  ${LIBRETRO_DEBUG})
elseif(CORE_SYSTEM_NAME STREQUAL osx)
  set(BUILD_COMMAND $(MAKE) -C . -f Makefile  platform=osx ${LIBRETRO_DEBUG})
elseif(CORE_SYSTEM_NAME STREQUAL ios)
  message(FATAL_ERROR "${PROJECT_NAME} needs iOS build command in CMakeLists.txt!")
elseif(CORE_SYSTEM_NAME STREQUAL android)
  message(FATAL_ERROR "${PROJECT_NAME} needs Android build command in CMakeLists.txt!")
elseif(CORE_SYSTEM_NAME STREQUAL rbpi)
  message(FATAL_ERROR "${PROJECT_NAME} needs RPi build command in CMakeLists.txt!")
elseif(CORE_SYSTEM_NAME STREQUAL freebsd)
  message(FATAL_ERROR "${PROJECT_NAME} needs FreeBSD build command in CMakeLists.txt!")
else()
  message(FATAL_ERROR "${PROJECT_NAME} - Unknown system: ${CORE_SYSTEM_NAME}")
endif()

externalproject_add(${PROJECT_NAME}
                    SOURCE_DIR ${PROJECT_SOURCE_DIR}
                    PREFIX ${PROJECT_NAME}
                    CONFIGURE_COMMAND ""
                    INSTALL_COMMAND ""
                    BUILD_COMMAND ${BUILD_COMMAND}
                    BUILD_IN_SOURCE 1)

# install the generated shared library
install(FILES ${PROJECT_SOURCE_DIR}/${LIBRETRO_BINARY_DIR}/${LIBRETRO_SONAME}
        DESTINATION ${CMAKE_INSTALL_PREFIX}/lib)

# write the config.cmake script
file(WRITE ${CMAKE_INSTALL_PREFIX}/libretro-${PROJECT_NAME}-config.cmake "set(${PROJECT_NAME_UC}_LIB ${CMAKE_INSTALL_PREFIX}/lib/${LIBRETRO_SONAME})")