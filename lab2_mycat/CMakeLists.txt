cmake_minimum_required(VERSION 3.15)

set(PROJECT_NAME mycat)

project(${PROJECT_NAME} CXX)

set(CMAKE_CXX_STANDARD 17)

# Options
option(WARNINGS_AS_ERRORS "Treat compiler warnings as errors." OFF)
option(ENABLE_SANITIZERS "Use sanitizers to detect errors." OFF)
option(ENABLE_CONAN "Use Conan as a package manager." OFF)

## PVS Studio
include(../cmake/PVS-Studio.cmake)
SET(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# Include fixed CMake configuration
include(../cmake/config.cmake)

## include Headers
include_directories(inc)
file(GLOB_RECURSE SRC "inc/*.h" "src/*.cpp")

# Project source compilation
add_executable(${PROJECT_NAME} src/main.cpp ${SRC})

# Add external packages
find_package(Boost 1.71.0 COMPONENTS program_options REQUIRED)
include_directories(${Boost_INCLUDE_DIR})
target_link_libraries(${PROJECT_NAME} Boost::program_options)

# Sanitizers configuration
if (ENABLE_SANITIZERS)
    set(SANITIZE_UNDEFINED ON)
    # Only one of Memory, Address, or Thread sanitizers is applicable at the time
    set(SANITIZE_MEMORY ON)
    #    set(SANITIZE_ADDRESS ON)
    #    set(SANITIZE_THREAD ON)
    find_package(Sanitizers)

    # CMake targets to which apply sanitizers
    add_sanitizers(operations ${PROJECT_NAME})
endif ()

#####################################
# create own target for library
pvs_studio_add_target(TARGET ${PROJECT_NAME}.analyze ALL
        ANALYZE ${PROJECT_NAME}
        OUTPUT FORMAT errorfile
        MODE GA:1,2,3 OP:1,2,3 GA:1,2,3 OP:1,2,3 64:1,2,3 CS:1,2,3 MISRA:1,2,3
        LOG target_${PROJECT_NAME}.err)
#####################################

