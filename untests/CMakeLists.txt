# 3.11 version is required for `FetchContent`
cmake_minimum_required(VERSION 3.11 FATAL_ERROR)

project(promise.hpp.untests)

#
# coverage
#

option(BUILD_WITH_COVERAGE "Build with coverage" OFF)
if(BUILD_WITH_COVERAGE AND (CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang"))
    set(COVERAGE_FLAGS "--coverage")
    set(CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} ${COVERAGE_FLAGS}")
    set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} ${COVERAGE_FLAGS}")
    set(CMAKE_LINKER_FLAGS_DEBUG "${CMAKE_LINKER_FLAGS_DEBUG} ${COVERAGE_FLAGS}")
endif()

#
# executable
#

file(GLOB UNTESTS_SOURCES "*.cpp" "*.hpp")
add_executable(${PROJECT_NAME} ${UNTESTS_SOURCES})
target_link_libraries(${PROJECT_NAME} promise.hpp)

target_compile_options(${PROJECT_NAME}
    PRIVATE
    $<$<CXX_COMPILER_ID:MSVC>:
        /W4 /bigobj>
    PRIVATE
    $<$<OR:$<CXX_COMPILER_ID:GNU>,$<CXX_COMPILER_ID:Clang>,$<CXX_COMPILER_ID:AppleClang>>:
        -Wall -Wextra -Wpedantic>)

add_test(${PROJECT_NAME} ${PROJECT_NAME})

#
# dependencies
#

find_package(Threads REQUIRED)
target_link_libraries(${PROJECT_NAME} Threads::Threads)

include(FetchContent)
FetchContent_Declare(
    catch2
    GIT_REPOSITORY https://github.com/catchorg/catch2)

FetchContent_GetProperties(catch2)
if(NOT catch2_POPULATED)
    FetchContent_Populate(catch2)
    target_include_directories(${PROJECT_NAME}
        PRIVATE ${catch2_SOURCE_DIR}/single_include)
endif()
