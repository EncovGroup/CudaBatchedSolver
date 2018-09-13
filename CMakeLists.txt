cmake_minimum_required(VERSION 3.1.3)

project(CudaBatchedSolver LANGUAGES C CXX VERSION 1.0)

option(CudaBatchedSolver_BUILD_SAMPLES "Build the samples" ON)
option(CudaBatchedSolver_EXPORT_PACKAGE "Export the library as cmake package" OFF)
option(BUILD_SHARED_LIBS "Build shared libraries" OFF)

# There was a bug in FindCUDA fixed in cmake 3.10 that creates empty strings on the nvcc command line.
# The solution merged in cmake 3.10 use the command COMMAND_EXPAND_LISTS only available from cmake >= 3.8
# https://gitlab.kitware.com/cmake/cmake/merge_requests/1008
# So we use another solution compatible with cmake >= 3.4 proposed here:
# https://gitlab.kitware.com/cmake/cmake/issues/16411
# if cmake version < 3.10 use our hacked version of FindCUDA:
if(${CMAKE_MAJOR_VERSION}.${CMAKE_MINOR_VERSION} VERSION_LESS 3.10)
    set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CMAKE_CURRENT_SOURCE_DIR}/cmake/compatibilityCUDA)
endif()

if(BUILD_SHARED_LIBS)
    message(STATUS "BUILD_SHARED_LIBS ${BUILD_SHARED_LIBS}")
    # Need to declare CUDA_USE_STATIC_CUDA_RUNTIME as an option to ensure that it is not overwritten in FindCUDA.
    option(CUDA_USE_STATIC_CUDA_RUNTIME "Use the static version of the CUDA runtime library if available" OFF)
    set(CUDA_USE_STATIC_CUDA_RUNTIME OFF)
    # Workaround to force deactivation of cuda static runtime for cmake < 3.10
    set(CUDA_cudart_static_LIBRARY 0)
    set(CMAKE_POSITION_INDEPENDENT_CODE ON)
else()
    message(STATUS "BUILD_SHARED_LIBS ${BUILD_SHARED_LIBS}")
    option(CUDA_USE_STATIC_CUDA_RUNTIME "Use the static version of the CUDA runtime library if available" ON)
    set(CUDA_USE_STATIC_CUDA_RUNTIME ON)
    set(CMAKE_POSITION_INDEPENDENT_CODE OFF)
endif()

find_package(CUDA 7.0 REQUIRED)

if(CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
    if(CMAKE_CXX_COMPILER_VERSION VERSION_GREATER "4.9.3")
        if(CUDA_VERSION VERSION_LESS "8.0")
            MESSAGE(STATUS "Found gcc >=5 and CUDA <= 7.5, adding workaround C++ flags")
            set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -D_FORCE_INLINES -D_MWAITXINTRIN_H_INCLUDED -D__STRICT_ANSI__")
        endif(CUDA_VERSION VERSION_LESS "8.0")
    endif(CMAKE_CXX_COMPILER_VERSION VERSION_GREATER "4.9.3")
endif(CMAKE_CXX_COMPILER_ID STREQUAL "GNU")

set(CUDA_SEPARABLE_COMPILATION ON)

set(CUDA_NVCC_FLAGS_DEBUG   "${CUDA_NVCC_FLAGS_DEBUG};-G")

if(BUILD_SHARED_LIBS)
    # this goes here because findCuda reset the flags
    set(CUDA_NVCC_FLAGS         "${CUDA_NVCC_FLAGS};-Xcompiler;-fPIC;")
endif()

set(CUDA_NVCC_FLAGS         "${CUDA_NVCC_FLAGS};-gencode;arch=compute_30,code=sm_30")
set(CUDA_NVCC_FLAGS         "${CUDA_NVCC_FLAGS};-gencode;arch=compute_35,code=sm_35")
set(CUDA_NVCC_FLAGS         "${CUDA_NVCC_FLAGS};-gencode;arch=compute_50,code=sm_50")
set(CUDA_NVCC_FLAGS         "${CUDA_NVCC_FLAGS};-gencode;arch=compute_52,code=sm_52")
set(CUDA_NVCC_FLAGS         "${CUDA_NVCC_FLAGS};-gencode=arch=compute_52,code=compute_52")

if(NOT CUDA_VERSION VERSION_LESS 8.0) # cuda >= 8.0
    set(CUDA_NVCC_FLAGS         "${CUDA_NVCC_FLAGS};-gencode=arch=compute_60,code=sm_60")
    set(CUDA_NVCC_FLAGS         "${CUDA_NVCC_FLAGS};-gencode=arch=compute_61,code=sm_61")
    set(CUDA_NVCC_FLAGS         "${CUDA_NVCC_FLAGS};-gencode=arch=compute_62,code=sm_62")
endif()

if(NOT CUDA_VERSION VERSION_LESS 7.0) # cuda >= 7.0
    set(CUDA_NVCC_FLAGS         "${CUDA_NVCC_FLAGS};-std=c++11;")
endif()


# EXPORTING THE LIBRARY
#
include(GNUInstallDirs)
# place to put the cmake-related files
set(config_install_dir "lib/cmake/${PROJECT_NAME}")
# include directory for install
set(include_install_dir "include")

# build directory containing the generated files
set(generated_dir "${CMAKE_CURRENT_BINARY_DIR}/generated")

# Configuration
set(version_config "${generated_dir}/${PROJECT_NAME}ConfigVersion.cmake")
set(project_config "${generated_dir}/${PROJECT_NAME}Config.cmake")
set(targets_export_name "${PROJECT_NAME}Targets")
set(namespace "${PROJECT_NAME}::")

add_subdirectory(src/CudaBatchedSolver)

if(CudaBatchedSolver_BUILD_SAMPLES)
    add_subdirectory(src/sample)
endif()


# Include module with fuction 'write_basic_package_version_file'
include(CMakePackageConfigHelpers)

# Configure '<PROJECT-NAME>ConfigVersion.cmake'
# Note: major version number must be the same as requested
write_basic_package_version_file("${version_config}" COMPATIBILITY SameMajorVersion)

# Configure '<PROJECT-NAME>Config.cmake'
# Use variables:
#   * targets_export_name
#   * PROJECT_NAME
configure_package_config_file("${PROJECT_SOURCE_DIR}/cmake/Config.cmake.in"
        "${project_config}"
        INSTALL_DESTINATION "${config_install_dir}")


# Config
#   * <prefix>/lib/cmake/${PROJECT_NAME}/${PROJECT_NAME}Targets.cmake
install(EXPORT "${targets_export_name}"
        NAMESPACE "${namespace}"
        DESTINATION "${config_install_dir}")

# Config
#   * <prefix>/lib/cmake/${PROJECT_NAME}/${PROJECT_NAME}Config.cmake
#   * <prefix>/lib/cmake/${PROJECT_NAME}/${PROJECT_NAME}ConfigVersion.cmake
install(
        FILES "${project_config}" "${version_config}"
        DESTINATION "${config_install_dir}")

if(CudaBatchedSolver_EXPORT_PACKAGE)
    export(PACKAGE CudaBatchedSolver)
endif()

## Uninstall target
configure_file(
        "${CMAKE_CURRENT_SOURCE_DIR}/cmake/cmake_uninstall.cmake.in"
        "${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake"
        IMMEDIATE @ONLY)

add_custom_target(CudaBatchedSolver_uninstall
        COMMAND ${CMAKE_COMMAND} -P ${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake)
