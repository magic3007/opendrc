cmake_minimum_required(VERSION 3.18)

project(
  OpenDRC
  VERSION 0.2.0
  DESCRIPTION "A Design Rule Checker with Hierarchical GPU Acceleration"
  LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

execute_process(
    COMMAND ${CMAKE_CXX_COMPILER} -dumpversion
    OUTPUT_VARIABLE GCC_VERSION
    OUTPUT_STRIP_TRAILING_WHITESPACE
)

message(STATUS "GCC version: ${GCC_VERSION}")

if (GCC_VERSION VERSION_GREATER_EQUAL 8.0)
    set(USE_FILESYSTEM_LIB true)
elseif (GCC_VERSION VERSION_GREATER_EQUAL 5.0)
    set(USE_EXPERIMENTAL_FILESYSTEM_LIB true)
else()
    message(FATAL_ERROR "Unsupported g++ version. Minimum version required: 5.0 for experimental filesystem library, 8.0 for std::filesystem")
endif()

if (USE_EXPERIMENTAL_FILESYSTEM_LIB)
    link_libraries(stdc++fs)
endif()


option(ODRC_BUILD_TESTS "Enables builds of tests" ON)
option(ODRC_BUILD_EXAMPLES "Enables builds of examples" ON)
option(ODRC_BUILD_CUDA "Enables builds of CUDA" ON)

if(ODRC_BUILD_CUDA)
  ENABLE_LANGUAGE(CUDA)
  set(CMAKE_CUDA_STANDARD 17)
  set(CMAKE_CUDA_STANDARD_REQUIRED True)
  if(NOT DEFINED CMAKE_CUDA_ARCHITECTURES)
    include(FindCUDA/select_compute_arch)
    CUDA_DETECT_INSTALLED_GPUS(INSTALLED_GPU_CCS_1)
    string(STRIP "${INSTALLED_GPU_CCS_1}" INSTALLED_GPU_CCS_2)
    string(REPLACE " " ";" INSTALLED_GPU_CCS_3 "${INSTALLED_GPU_CCS_2}")
    string(REPLACE "." "" CUDA_ARCH_LIST "${INSTALLED_GPU_CCS_3}")
    SET(CMAKE_CUDA_ARCHITECTURES ${CUDA_ARCH_LIST})
  endif()
endif()

add_subdirectory(odrc)

if(ODRC_BUILD_TESTS)
  enable_testing()
  add_subdirectory(tests)
endif()

if(ODRC_BUILD_EXAMPLES)
  add_subdirectory(examples)
endif()
