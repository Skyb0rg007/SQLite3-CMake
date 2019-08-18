cmake_minimum_required(VERSION 3.11 FATAL_ERROR)
project(SQLite3
    DESCRIPTION "Sqlite3, built with CMake"
    VERSION 3.30.0
    LANGUAGES C)

find_package(Threads)
include(CMakeDependentOption)

# Options
cmake_dependent_option(SQLITE3_THREADSAFE "Enable multithread sqlite access" ON
    "NOT Threads_FOUND" OFF)

# Library
add_library(sqlite3)
target_sources(sqlite3
    PRIVATE
    src/sqlite3.c)
target_include_directories(sqlite3
    PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>)
set_target_properties(sqlite3 PROPERTIES
    OUTPUT_NAME sqlite3
    VERSION ${PROJECT_VERSION})
if(Threads_FOUND AND SQLITE3_THREADSAFE)
    target_link_libraries(sqlite3
        PRIVATE
        Threads::Threads)
else()
    target_compile_definitions(sqlite3
        PRIVATE
        SQLITE_THREADSAFE=0)
endif()

add_library(SQLite::SQLite3 ALIAS sqlite3)

# Executable
add_executable(sqlite3_shell EXCLUDE_FROM_ALL)
target_sources(sqlite3_shell
    PRIVATE
    src/sqlite3.c
    src/shell.c)
target_include_directories(sqlite3_shell
    PRIVATE
    include)
target_link_libraries(sqlite3_shell
    PRIVATE
    ${CMAKE_DL_LIBS})
if(Threads_FOUND AND SQLITE_THREADSAFE)
    target_link_libraries(sqlite3_shell
        PRIVATE
        Threads::Threads)
else()
    target_compile_definitions(sqlite3_shell
        PRIVATE
        SQLITE_THREADSAFE=0)
endif()

add_executable(SQLite::shell ALIAS sqlite3_shell)
