# Minimal build system for nuked-opp-xcent.
#
# Produces a shared library (NukedOPP.dll / libNukedOPP.dylib /
# libNukedOPP.so) suitable for substituting into a desktop XCent
# installation to exercise the LGPL §6 relink right. The build flags
# match XCent's CMakeLists.txt section for NukedOPP so the resulting
# library is ABI-compatible with the XCent binary that ships next to
# it.
#
# Usage:
#   cmake -B build -DCMAKE_BUILD_TYPE=Release
#   cmake --build build --config Release
#
# This file is part of the public LGPL mirror; the upstream Nuked-OPM
# repository does not ship a build system, and neither does the
# corresponding directory inside the (private) XCent source tree —
# the XCent CMake configuration builds NukedOPP via its own logic.
# This standalone CMakeLists.txt exists only to make end-user
# relinking practical without cloning the entire XCent build.

cmake_minimum_required(VERSION 3.16)
project(NukedOPP LANGUAGES C VERSION 1.0)

if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Release CACHE STRING "Build type" FORCE)
endif()

add_library(NukedOPP SHARED opp.c)

set_target_properties(NukedOPP PROPERTIES
    C_STANDARD 99
    C_STANDARD_REQUIRED ON
    POSITION_INDEPENDENT_CODE ON
    # Auto-export every extern C symbol on Windows so we don't have to
    # add __declspec(dllexport) to opp.h (the header is kept close to
    # upstream Nuked-OPM).
    WINDOWS_EXPORT_ALL_SYMBOLS ON
)

if(APPLE)
    set_target_properties(NukedOPP PROPERTIES
        INSTALL_NAME_DIR "@rpath"
        MACOSX_RPATH ON
    )
elseif(UNIX)
    set_target_properties(NukedOPP PROPERTIES
        INSTALL_RPATH "$ORIGIN"
    )
endif()

if(MSVC)
    target_compile_options(NukedOPP PRIVATE
        $<$<CONFIG:Release>:/Ox /GS->
    )
else()
    target_compile_options(NukedOPP PRIVATE
        $<$<CONFIG:Release>:-O3>
    )
endif()
