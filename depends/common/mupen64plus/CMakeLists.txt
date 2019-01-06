cmake_minimum_required(VERSION 3.1)
project(mupen64plus)

list(APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR})

include(ExternalProject)

string(TOUPPER ${PROJECT_NAME} PROJECT_NAME_UC)

set(LIBRETRO_BINARY_DIR .)
set(LIBRETRO_SONAME mupen64plus_libretro${CMAKE_SHARED_LIBRARY_SUFFIX})
set(LIBRETRO_JNISONAME libretro${CMAKE_SHARED_LIBRARY_SUFFIX})

if(CMAKE_BUILD_TYPE MATCHES Debug)
  set(LIBRETRO_DEBUG DEBUG=1)
endif()

if(CORE_SYSTEM_NAME STREQUAL windows)
  find_package(MinGW REQUIRED)
  set(BUILD_COMMAND ${MINGW_MAKE} -j$ENV{NUMBER_OF_PROCESSORS} -C . -f Makefile platform=win  ${LIBRETRO_DEBUG} GIT_VERSION=)
elseif(CORE_SYSTEM_NAME STREQUAL linux)
  set(BUILD_COMMAND $(MAKE) -C . -f Makefile  platform=unix ${LIBRETRO_DEBUG} GIT_VERSION=)
elseif(CORE_SYSTEM_NAME STREQUAL osx)
  set(BUILD_COMMAND $(MAKE) -C . -f Makefile  platform=osx ${LIBRETRO_DEBUG} GIT_VERSION=)
elseif(CORE_SYSTEM_NAME STREQUAL ios)
  set(LIBRETRO_SONAME mupen64plus_libretro_ios${CMAKE_SHARED_LIBRARY_SUFFIX})
  set(BUILD_COMMAND IOSSDK=${CMAKE_OSX_SYSROOT} $(MAKE) -C . -f Makefile  platform=ios ${LIBRETRO_DEBUG} GIT_VERSION=)
elseif(CORE_SYSTEM_NAME STREQUAL android)
  if(NOT NDKROOT)
    message(FATAL_ERROR "${PROJECT_NAME} needs NDKROOT for Android. Missing Toolchain file?")
  endif()
  if(CPU STREQUAL armeabi-v7a OR CPU STREQUAL arm64-v8a)
    set(PLATFORM ${CPU})
  elseif(CPU STREQUAL i686)
    set(PLATFORM x86)
  else()
    message(FATAL_ERROR "${PROJECT_NAME} needs Android ${CPU} build command in CMakeLists.txt!")
  endif()
  set(BUILD_COMMAND GNUMAKE=$(MAKE) ${NDKROOT}/ndk-build -C jni APP_ABI=${PLATFORM} V7NEONOPTIMIZATION=1 GIT_VERSION=
                                    NDK_LIBS_OUT=${PROJECT_SOURCE_DIR}/${LIBRETRO_BINARY_DIR}  ${LIBRETRO_DEBUG} V=1
                    && cp ${PROJECT_SOURCE_DIR}/${LIBRETRO_BINARY_DIR}/${PLATFORM}/${LIBRETRO_JNISONAME} ${PROJECT_SOURCE_DIR}/${LIBRETRO_BINARY_DIR}/${LIBRETRO_SONAME})
elseif(CORE_SYSTEM_NAME STREQUAL rbpi)
  message(FATAL_ERROR "${PROJECT_NAME} needs RPi build command in CMakeLists.txt!")
elseif(CORE_SYSTEM_NAME STREQUAL freebsd)
  set(BUILD_COMMAND $(MAKE) -C . -f Makefile  platform=unix ${LIBRETRO_DEBUG} GIT_VERSION=)
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
        DESTINATION ${CMAKE_INSTALL_PREFIX}/lib/libretro RENAME mupen64plus_libretro${CMAKE_SHARED_LIBRARY_SUFFIX})
