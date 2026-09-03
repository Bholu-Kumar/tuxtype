#
# CMakeLists.txt for TuxType (SDL3 Edition)
#
# Provides native Windows (MinGW/MSYS2) build support alongside the
# existing Autotools build for Debian/Linux.
# All Windows-specific blocks are guarded by if(MINGW) so the Linux
# build is completely unaffected.
#

cmake_minimum_required(VERSION 2.6)

set(PACKAGE_NAME "tuxtype")
set(PACKAGE_VERSION "1.8.3")
set(APP_NAME "TuxType")
project(${APP_NAME})

set(CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake-modules")

# -------------------------------------------------------------------
# Find required libraries via pkg-config (works on both Linux + MinGW)
# -------------------------------------------------------------------
find_package(PkgConfig REQUIRED)

pkg_check_modules(SDL3       REQUIRED IMPORTED_TARGET sdl3)
pkg_check_modules(SDL3_IMAGE REQUIRED IMPORTED_TARGET sdl3-image)
pkg_check_modules(SDL3_TTF   REQUIRED IMPORTED_TARGET sdl3-ttf)
pkg_check_modules(SDL3_MIXER REQUIRED IMPORTED_TARGET sdl3-mixer)
pkg_check_modules(SDL3_NET   IMPORTED_TARGET sdl3-net)
pkg_check_modules(LIBRSVG    REQUIRED librsvg-2.0)
pkg_check_modules(LIBXML2    REQUIRED libxml-2.0)
pkg_check_modules(ESPEAK_NG           espeak-ng)

find_package(LibXml2 REQUIRED)

# t4kcommon — the shared library we built earlier
pkg_check_modules(T4K_COMMON t4k_common)
if(NOT T4K_COMMON_FOUND)
    # Fallback: look next to this repo (typical development layout)
    set(T4K_COMMON_INCLUDE_DIRS "${CMAKE_SOURCE_DIR}/../t4kcommon/src")
    set(T4K_COMMON_LIBRARY_DIRS "${CMAKE_SOURCE_DIR}/../t4kcommon/build/src")
    set(T4K_COMMON_LIBRARIES    "t4k_common")
    message(STATUS "t4k_common not found via pkg-config, using fallback path: ${T4K_COMMON_LIBRARY_DIRS}")
endif()

# -------------------------------------------------------------------
# Platform-specific data prefix (where the game looks for its files)
# -------------------------------------------------------------------
if(APPLE)
    set(DATA_PREFIX "${APP_NAME}.app/Contents/Resources")
elseif(WIN32)
    # On Windows, data sits next to the .exe in the install directory
    set(DATA_PREFIX "data")
    if(MINGW)
        add_definitions(-DBUILD_MINGW32)
    endif()
elseif(UNIX)
    set(DATA_PREFIX "${CMAKE_INSTALL_PREFIX}/share/${PACKAGE_NAME}")
else()
    message(SEND_ERROR "Unknown platform!")
endif()

# -------------------------------------------------------------------
# espeak-ng (TTS) — optional
# -------------------------------------------------------------------
if(ESPEAK_NG_FOUND)
    set(WITH_ESPEAK 1)
    set(WITH_ESPEAK_NG 1)
else()
    pkg_check_modules(ESPEAK espeak)
    if(ESPEAK_FOUND)
        set(WITH_ESPEAK 1)
        set(WITH_ESPEAK_NG 0)
    endif()
endif()

# -------------------------------------------------------------------
# Process the src/ subdirectory
# -------------------------------------------------------------------
add_subdirectory(src)
