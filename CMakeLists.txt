# Note: CMake support is community-based. The maintainers do not use CMake
# internally.

cmake_minimum_required(VERSION 3.5)

if (POLICY CMP0048)
  cmake_policy(SET CMP0048 NEW)
endif (POLICY CMP0048)

if (POLICY CMP0069)
  cmake_policy(SET CMP0069 NEW)
endif (POLICY CMP0069)

if (POLICY CMP0077)
  cmake_policy(SET CMP0077 NEW)
endif (POLICY CMP0077)

project(googletest-distribution)
set(GOOGLETEST_VERSION 1.12.1)

if(NOT CYGWIN AND NOT MSYS AND NOT ${CMAKE_SYSTEM_NAME} STREQUAL QNX)
  set(CMAKE_CXX_EXTENSIONS OFF)
endif()

enable_testing()

include(CMakeDependentOption)
include(GNUInstallDirs)

#Note that googlemock target already builds googletest
option(BUILD_GMOCK "Builds the googlemock subproject" ON)
option(INSTALL_GTEST "Enable installation of googletest. (Projects embedding googletest may want to turn this OFF.)" ON)

if(MSVC)
  set(MSVC_CRT /MD)
  set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreadedDLL")
  if(CMAKE_BUILD_TYPE STREQUAL "Debug")
      message("Disabling Debug CRT")
      add_definitions(-D_ITERATOR_DEBUG_LEVEL=0)
      add_compile_options(/Od /Zi)
      foreach(MODE "_DEBUG" "_MINSIZEREL" "_RELEASE" "_RELWITHDEBINFO")
          string(REPLACE "${MSVC_CRT}d" "${MSVC_CRT}" TMP1 "${CMAKE_C_FLAGS${MODE}}")
          string(REPLACE "${MSVC_CRT}d" "${MSVC_CRT}" TMP2 "${CMAKE_CXX_FLAGS${MODE}}")
          set(CMAKE_C_FLAGS${MODE} "${TMP1}" CACHE STRING "" FORCE)
          set(CMAKE_CXX_FLAGS${MODE} "${TMP2}" CACHE STRING "" FORCE)
          #message(STATUS "C_${MODE}=${CMAKE_C_FLAGS${MODE}}")
          #message(STATUS "CXX_${MODE}=${CMAKE_CXX_FLAGS${MODE}}")
      endforeach(MODE)
  endif()
endif()

if((CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang") AND (CMAKE_SYSTEM_PROCESSOR STREQUAL "x86_64"))
    # -march=x86-64-v3: (close to Haswell) AVX, AVX2, BMI1, BMI2, F16C, FMA, LZCNT, MOVBE, XSAVE
    # -march=x86-64-v4: AVX512F, AVX512BW, AVX512CD, AVX512DQ, AVX512VL (generally not available on VM-s in 2023)
    # NOTE: on Alder Lake and later, AVX512 is no longer supported! So to support broken Intel CPU-s we cannot have AVX512
    add_compile_options(-march=x86-64-v3)
    message(STATUS "BUGFIX: Using -march=x86-64-v3 for backwards compatibility")
endif()

if(BUILD_GMOCK)
  add_subdirectory( googlemock )
else()
  add_subdirectory( googletest )
endif()
