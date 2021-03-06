# General parameters
cmake_minimum_required(VERSION 2.6.3)
project(yaml-tests CXX)

set(CMAKE_MODULE_PATH "${PROJECT_SOURCE_DIR}/cmake/")
option(COLLECT_FILES "Copy libraries to <build>/lib, binaries to <build>/bin, headers to <build>/include." ON)

#####################################
## Settings
#####################################

##
## default options
##
set(CMAKE_VERBOSE_MAKEFILE ON)

if(COLLECT_FILES)
    set(RUNTIME_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}/bin")
    set(LIBRARY_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}/lib")
    set(ARCHIVE_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}/lib")

    set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}/bin")
    set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}/lib")
    set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}/lib")
endif(COLLECT_FILES)

##
## Mandatory dependencies
##

# Search for Eigen
find_package(Eigen3 REQUIRED)
include_directories(SYSTEM ${EIGEN3_INCLUDE_DIR})

#####################################
## Dependencies
#####################################

add_custom_target(TGT_yaml-cpp_build_libraries)
add_custom_target(TGT_yaml-cpp_copy_headers)

# yaml-cpp 
set(YAML-CPP_DIR "${PROJECT_SOURCE_DIR}/contrib/yaml-cpp")
set(CMAKE_CXX_FLAGS_DEBUG "")
set(CMAKE_BUILD_TYPE      Release)

add_subdirectory("${YAML-CPP_DIR}" EXCLUDE_FROM_ALL)

set_property(TARGET yaml-cpp APPEND_STRING PROPERTY COMPILE_FLAGS " -fPIC ")

add_dependencies(TGT_yaml-cpp_build_libraries yaml-cpp)

if(COLLECT_FILES)
    add_custom_target(TGT_copy_yaml-cpp_headers COMMAND ${CMAKE_COMMAND} -E
                      copy_directory "${YAML-CPP_DIR}/include" "${PROJECT_BINARY_DIR}/include/yaml-cpp")
    include_directories(SYSTEM "${PROJECT_BINARY_DIR}/include/yaml-cpp")
    add_dependencies(TGT_yaml-cpp_copy_headers TGT_copy_yaml-cpp_headers)
else(COLLECT_FILES)
    include_directories(SYSTEM "${YAML-CPP_DIR}/include/")
endif(COLLECT_FILES)

#############
## Variables
#############

set(CMAKE_BUILD_TYPE      ${YAML-CPP_BUILD_TYPE})
set(CMAKE_CXX_FLAGS_DEBUG ${YAML-CPP_CXX_FLAGS_DEBUG})

########
## main 
########

set(TGT_NAME main)
add_executable(${TGT_NAME} ${CMAKE_SOURCE_DIR}/src/main.cpp)
set_property(TARGET ${TGT_NAME} APPEND PROPERTY COMPILE_FLAGS "-std=c++11")

target_link_libraries(${TGT_NAME} yaml-cpp)

add_dependencies(${TGT_NAME} TGT_yaml-cpp_copy_headers)
add_dependencies(${TGT_NAME} TGT_yaml-cpp_build_libraries)

set(REF_FILENAME "config.yaml") 
file(COPY ${REF_FILENAME} DESTINATION ${CMAKE_RUNTIME_OUTPUT_DIRECTORY})
