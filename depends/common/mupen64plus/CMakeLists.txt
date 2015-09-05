project(mupen64plus)

cmake_minimum_required(VERSION 2.6)

list(APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR})

enable_language(CXX)

include(ExternalProject)

if(CMAKE_BUILD_TYPE MATCHES Debug)
  set(LIBRETRO_DEBUG DEBUG=1)
endif()

if("${CORE_SYSTEM_NAME}" STREQUAL "windows")
  find_package(MinGW REQUIRED)
  set(BUILD_COMMAND ${MINGW_MAKE} ${LIBRETRO_DEBUG})
elseif("${CORE_SYSTEM_NAME}" STREQUAL "linux")
  set(BUILD_COMMAND make ${LIBRETRO_DEBUG})
elseif("${CORE_SYSTEM_NAME}" STREQUAL "darwin" OR "${CORE_SYSTEM_NAME}" STREQUAL "osx")
  set(BUILD_COMMAND make platform=osx ${LIBRETRO_DEBUG})
elseif("${CORE_SYSTEM_NAME}" STREQUAL "ios")
  message(FATAL_ERROR "${PROJECT_NAME} needs IOS build command in CMakeLists.txt!")
elseif("${CORE_SYSTEM_NAME}" STREQUAL "android")
  message(FATAL_ERROR "${PROJECT_NAME} needs Android build command in CMakeLists.txt!")
elseif("${CORE_SYSTEM_NAME}" STREQUAL "rbpi")
  message(FATAL_ERROR "${PROJECT_NAME} needs RPi build command in CMakeLists.txt!")
elseif("${CORE_SYSTEM_NAME}" STREQUAL "freebsd")
  message(FATAL_ERROR "${PROJECT_NAME} needs FreeBSD build command in CMakeLists.txt!")
else()
  message(FATAL_ERROR "${PROJECT_NAME} - Unknown system: ${CORE_SYSTEM_NAME}")
endif()

externalproject_add(mupen64plus
                    SOURCE_DIR ${PROJECT_SOURCE_DIR}
                    PREFIX mupen64plus
                    CONFIGURE_COMMAND ""
                    INSTALL_COMMAND ""
                    BUILD_COMMAND ${BUILD_COMMAND}
                    BUILD_IN_SOURCE 1)

#install the generated shared library
install(FILES ${PROJECT_SOURCE_DIR}/mupen64plus_libretro${CMAKE_SHARED_LIBRARY_SUFFIX}
        DESTINATION ${CMAKE_INSTALL_PREFIX}/lib)

# write the mupen64plus-config.cmake script
file(WRITE ${CMAKE_INSTALL_PREFIX}/libretro-mupen64plus-config.cmake "set(MUPEN64PLUS_LIB ${CMAKE_INSTALL_PREFIX}/lib/mupen64plus_libretro${CMAKE_SHARED_LIBRARY_SUFFIX})")
