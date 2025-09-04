cmake_minimum_required(VERSION 3.5)
project(mupen64plus-nx)

list(APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR})

include(ExternalProject)

string(TOUPPER ${PROJECT_NAME} PROJECT_NAME_UC)

# Jenkins CI jobs can have this env var set
# Use it if available
if(DEFINED ENV{BUILDTHREADS})
  set(build_job_count -j$ENV{BUILDTHREADS})
  message(STATUS "Using $ENV{BUILDTHREADS} threads from ENV var BUILDTHREADS")
else()
  # Most likely a non CI build, so do a best attempt just using ProcessorCount
  include(ProcessorCount)
  ProcessorCount(cpu_count)
  if(NOT cpu_count EQUAL 0)
    set(build_job_count -j${cpu_count})
    message(STATUS "Using ${cpu_count} threads deduced by ProcessorCount")
  endif()
endif()

set(LIBRETRO_BINARY_DIR .)
set(LIBRETRO_SONAME mupen64plus_next_libretro${CMAKE_SHARED_LIBRARY_SUFFIX})
set(LIBRETRO_JNISONAME libretro${CMAKE_SHARED_LIBRARY_SUFFIX})

if(CMAKE_BUILD_TYPE MATCHES Debug)
  set(LIBRETRO_DEBUG DEBUG=1)
endif()

if(CORE_SYSTEM_NAME STREQUAL windows)
  find_package(MinGW REQUIRED)

  if(MINGW_MAKE MATCHES "/mingw64/")
    set(MSYSTEM MINGW64)
  elseif(MINGW_MAKE MATCHES "/mingw32/")
    set(MSYSTEM MINGW32)
  else()
    message(FATAL_ERROR "Unknown MinGW flavor in MINGW_MAKE: ${MINGW_MAKE}")
  endif()

  message(STATUS "Using ${MSYSTEM} (from MINGW_MAKE='${MINGW_MAKE}')")

  set(BUILD_COMMAND ${MINGW_MAKE}
                    -C .
                    -f Makefile
                    ${build_job_count}
                    ${LIBRETRO_DEBUG}
                    GIT_VERSION=
                    MSYSTEM=${MSYSTEM}
                    platform=win
                    )
elseif(CORE_SYSTEM_NAME STREQUAL linux)
  set(BUILD_COMMAND $(MAKE)
                    -C .
                    -f Makefile
                    ${build_job_count}
                    ${LIBRETRO_DEBUG}
                    GIT_VERSION=
                    platform=unix
                    )
elseif(CORE_SYSTEM_NAME STREQUAL osx)
  if(CPU STREQUAL arm64)
    set(ARCH arm)
  else()
    set(ARCH intel)
  endif()
  set(BUILD_COMMAND $(MAKE)
                    -C .
                    -f Makefile
                    ${build_job_count}
                    ${LIBRETRO_DEBUG}
                    arch=${ARCH}
                    CROSS_COMPILE=1
                    GIT_VERSION=
                    LIBRETRO_APPLE_ISYSROOT=${CMAKE_OSX_SYSROOT}
                    LIBRETRO_APPLE_PLATFORM=${CPU}-apple-macos
                    platform=osx
                    )
elseif(CORE_SYSTEM_NAME STREQUAL ios OR CORE_SYSTEM_NAME STREQUAL darwin_embedded)
  if(CORE_PLATFORM_NAME STREQUAL ios)
    set(LIBRETRO_SONAME mupen64plus_next_libretro_ios${CMAKE_SHARED_LIBRARY_SUFFIX})
    if(CPU STREQUAL arm64)
      set(PLATFORM ios-arm64)
    else()
      set(PLATFORM ios)
    endif()
    set(BUILD_COMMAND IOSSDK=${CMAKE_OSX_SYSROOT} $(MAKE)
                                                  -C .
                                                  -f Makefile
                                                  ${build_job_count}
                                                  ${LIBRETRO_DEBUG}
                                                  GIT_VERSION=
                                                  platform=${PLATFORM}
                                                  )
  elseif(CORE_PLATFORM_NAME STREQUAL tvos)
    set(LIBRETRO_SONAME mupen64plus_next_libretro_tvos${CMAKE_SHARED_LIBRARY_SUFFIX})
    set(BUILD_COMMAND IOSSDK=${CMAKE_OSX_SYSROOT} $(MAKE)
                                                  -C .
                                                  -f Makefile
                                                  ${build_job_count}
                                                  ${LIBRETRO_DEBUG}
                                                  GIT_VERSION=
                                                  platform=tvos-arm64
                                                  )
  endif()
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
  set(BUILD_COMMAND GNUMAKE=$(MAKE) ${NDKROOT}/ndk-build
                                    -C libretro/jni
                                    ${build_job_count}
                                    ${LIBRETRO_DEBUG}
                                    APP_ABI=${PLATFORM}
                                    APP_SHORT_COMMANDS=true
                                    GIT_VERSION=
                                    NDK_LIBS_OUT=${PROJECT_SOURCE_DIR}/${LIBRETRO_BINARY_DIR}
                                    V=1
                                    V7NEONOPTIMIZATION=1
                                    
                    && cp ${PROJECT_SOURCE_DIR}/${LIBRETRO_BINARY_DIR}/${PLATFORM}/${LIBRETRO_JNISONAME} ${PROJECT_SOURCE_DIR}/${LIBRETRO_BINARY_DIR}/${LIBRETRO_SONAME})
elseif(CORE_SYSTEM_NAME STREQUAL freebsd)
  set(BUILD_COMMAND $(MAKE)
                    -C .
                    -f Makefile
                    ${build_job_count}
                    ${LIBRETRO_DEBUG}
                    GIT_VERSION=
                    platform=unix
                    )
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
        DESTINATION ${CMAKE_INSTALL_PREFIX}/lib/libretro RENAME mupen64plus_next_libretro${CMAKE_SHARED_LIBRARY_SUFFIX})
