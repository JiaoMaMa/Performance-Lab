cmake_minimum_required(VERSION 3.18)
project(PerformanceLab LANGUAGES CUDA CXX)

set_property(GLOBAL PROPERTY USE_FOLDERS ON)

set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

# Enable C++11 for host code
set(CMAKE_CXX_STANDARD 11)
if(NOT DEFINED CMAKE_CUDA_STANDARD)
    set(CMAKE_CUDA_STANDARD 11)
    set(CMAKE_CUDA_STANDARD_REQUIRED ON)
endif()

# Set a default build type if none was specified
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
    SET(CMAKE_BUILD_TYPE Release CACHE STRING "Choose the type of build." FORCE)
    # Set the possible values of build type for cmake-gui
    SET_PROPERTY(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "Debug" "Release" "MinSizeRel" "RelWithDebInfo")
endif()

########################################
# Setup the executables
########################################
set(EXECUTABLES transpose reduction)
file(GLOB headers source/*.h source/*.hpp)
foreach(exe ${EXECUTABLES})
    add_executable(${exe} source/${exe}.cu ${headers})
    if(CMAKE_VERSION VERSION_LESS "3.23.0")
        set_target_properties(${exe} PROPERTIES CUDA_ARCHITECTURES OFF)
    elseif(CMAKE_VERSION VERSION_LESS "3.24.0")
        set_target_properties(${exe} PROPERTIES CUDA_ARCHITECTURES all-major)
    else()
        set_target_properties(${exe} PROPERTIES CUDA_ARCHITECTURES native)
    endif()
    target_compile_options(${exe} PRIVATE "$<$<AND:$<CONFIG:Debug,RelWithDebInfo>,$<COMPILE_LANGUAGE:CUDA>>:-G;-src-in-ptx>")
    target_compile_options(${exe} PRIVATE "$<$<AND:$<CONFIG:Release>,$<COMPILE_LANGUAGE:CUDA>>:-lineinfo;-src-in-ptx>")
endforeach()
########################################

set_property(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR} PROPERTY VS_STARTUP_PROJECT transpose)
