cmake_minimum_required(VERSION 3.2)

CMAKE_POLICY(SET CMP0079 NEW)

project(SQLite3-Cpp-Encryption VERSION 0.1.0)

set(WXSQLITE3_DIR "${CMAKE_CURRENT_SOURCE_DIR}/externals/wxsqlite3")
set(SQLITECPP_DIR "${CMAKE_CURRENT_SOURCE_DIR}/externals/SQLiteCpp")

message(STATUS "---------------------------------------------------------")
message(STATUS "Please note that the rijndael.* files from wxSQLite3 are" )
message(STATUS "subject to the LGPL-3.0+ WITH WxWindows-exception-3.1."   )
message(STATUS "Make sure you read and respect the licensing terms, if"   )
message(STATUS "linking statically with sqlite3lib or sqlite3cpplib."     )
message(STATUS "---------------------------------------------------------")

list(APPEND SQLITE3SECURE_DLL_SRC "${WXSQLITE3_DIR}/sqlite3secure/src/sqlite3secure.c")
list(APPEND SQLITE3SECURE_LIB_SRC "${WXSQLITE3_DIR}/sqlite3secure/src/sqlite3secure.c")
list(APPEND SQLITE3SECURE_SHL_SRC "${WXSQLITE3_DIR}/sqlite3secure/src/shell.c")

if (MSVC) 
  list(APPEND SQLITE3SECURE_DLL_SRC "${WXSQLITE3_DIR}/sqlite3secure/src/sqlite3.rc")
  list(APPEND SQLITE3SECURE_DLL_SRC "${WXSQLITE3_DIR}/sqlite3secure/src/sqlite3.def")
  list(APPEND SQLITE3SECURE_LIB_SRC "${WXSQLITE3_DIR}/sqlite3secure/src/sqlite3.rc")
  list(APPEND SQLITE3SECURE_SHL_SRC "${WXSQLITE3_DIR}/sqlite3secure/src/sqlite3shell.rc")
endif()

add_library(sqlite3dll SHARED ${SQLITE3SECURE_DLL_SRC})
add_library(sqlite3lib STATIC ${SQLITE3SECURE_LIB_SRC})
add_executable(sqlite3shell ${SQLITE3SECURE_SHL_SRC})
target_link_libraries(sqlite3shell sqlite3lib)

target_include_directories(sqlite3dll PUBLIC ${WXSQLITE3_DIR}/sqlite3secure/src)
target_compile_definitions(sqlite3dll PUBLIC 
  _USRDLL
  THREADSAFE=1
  SQLITE_MAX_ATTACHED=10
  SQLITE_SOUNDEX
  SQLITE_ENABLE_COLUMN_METADATA
  SQLITE_HAS_CODEC=1
  CODEC_TYPE=CODEC_TYPE_CHACHA20
  SQLITE_SECURE_DELETE
  SQLITE_ENABLE_FTS3
  SQLITE_ENABLE_FTS3_PARENTHESIS
  SQLITE_ENABLE_FTS4
  SQLITE_ENABLE_FTS5
  SQLITE_ENABLE_JSON1
  SQLITE_ENABLE_RTREE
  SQLITE_ENABLE_GEOPOLY
  SQLITE_CORE
  SQLITE_ENABLE_EXTFUNC
  SQLITE_ENABLE_CSV
  SQLITE_ENABLE_SHA3
  SQLITE_ENABLE_CARRAY
  SQLITE_ENABLE_FILEIO
  SQLITE_ENABLE_SERIES
  SQLITE_TEMP_STORE=2
  SQLITE_USE_URI
  SQLITE_USER_AUTHENTICATION
)

target_include_directories(sqlite3lib PUBLIC ${WXSQLITE3_DIR}/sqlite3secure/src)
target_compile_definitions(sqlite3lib PUBLIC 
  _LIB
  THREADSAFE=1
  SQLITE_MAX_ATTACHED=10
  SQLITE_ENABLE_EXPLAIN_COMMENTS
  SQLITE_SOUNDEX
  SQLITE_ENABLE_COLUMN_METADATA
  SQLITE_HAS_CODEC=1
  CODEC_TYPE=CODEC_TYPE_CHACHA20
  SQLITE_SECURE_DELETE
  SQLITE_ENABLE_FTS3
  SQLITE_ENABLE_FTS3_PARENTHESIS
  SQLITE_ENABLE_FTS4
  SQLITE_ENABLE_FTS5
  SQLITE_ENABLE_JSON1
  SQLITE_ENABLE_RTREE
  SQLITE_ENABLE_GEOPOLY
  SQLITE_CORE
  SQLITE_ENABLE_EXTFUNC
  SQLITE_ENABLE_CSV
#  SQLITE_ENABLE_SHA3
  SQLITE_ENABLE_CARRAY
#  SQLITE_ENABLE_FILEIO
  SQLITE_ENABLE_SERIES
  SQLITE_TEMP_STORE=2
  SQLITE_USE_URI
  SQLITE_USER_AUTHENTICATION
# Compatibility with official SQLite3 shell    
  SQLITE_ENABLE_DBSTAT_VTAB
  SQLITE_ENABLE_STMTVTAB
  SQLITE_ENABLE_UNKNOWN_SQL_FUNCTION
)

target_include_directories(sqlite3shell PUBLIC ${WXSQLITE3_DIR}/sqlite3secure/src)
target_compile_definitions(sqlite3shell PUBLIC 
  SQLITE_SHELL_IS_UTF8
  SQLITE_HAS_CODEC=1
  SQLITE_USER_AUTHENTICATION
)

set(SQLITECPP_INTERNAL_SQLITE OFF CACHE BOOL "Use an external SQLite3 library with SQLiteCpp.")
set(SQLITECPP_RUN_CPPLINT OFF CACHE BOOL "Run cpplint.py tool for Google C++ StyleGuide.")
set(SQLITECPP_RUN_CPPCHECK OFF CACHE BOOL "Run cppcheck C++ static analysis tool.")
add_subdirectory(${SQLITECPP_DIR})
target_include_directories(SQLiteCpp PRIVATE ${WXSQLITE3_DIR}/sqlite3secure/src)
target_compile_definitions(SQLiteCpp PUBLIC SQLITE_HAS_CODEC=1)
#include_directories(${WXSQLITE3_DIR}/sqlite3secure/src)
#if(BUILD_SHARED_LIBS)
#  target_link_libraries(SQLiteCpp sqlite3dll)
#else()
#  target_link_libraries(SQLiteCpp sqlite3lib)
#endif()



# list of test files of the library
set(SQLITECPP_TESTS
 ${SQLITECPP_DIR}/tests/Column_test.cpp
 ${SQLITECPP_DIR}/tests/Database_test.cpp
 ${SQLITECPP_DIR}/tests/Statement_test.cpp
 ${SQLITECPP_DIR}/tests/Backup_test.cpp
 ${SQLITECPP_DIR}/tests/Transaction_test.cpp
 ${SQLITECPP_DIR}/tests/VariadicBind_test.cpp
 ${SQLITECPP_DIR}/tests/Exception_test.cpp
)
# deactivate some warnings for compiling the gtest library
if (NOT MSVC)
    add_compile_options(-Wno-variadic-macros -Wno-long-long -Wno-switch-enum -Wno-float-equal -Wno-conversion-null -Wno-switch-default -Wno-pedantic)
endif (NOT MSVC)

# add the subdirectory containing the CMakeLists.txt for the gtest library
# TODO: under Linux, uses libgtest-dev if found
if (NOT EXISTS "${SQLITECPP_DIR}/googletest/CMakeLists.txt")
    message(FATAL_ERROR "Missing 'googletest' submodule! Either use 'git init submodule' and 'git update submodule' to get googletest according to the README, or deactivate unit tests with -DSQLITECPP_BUILD_TESTS=OFF")
endif ()
set(gtest_force_shared_crt ON CACHE BOOL "Force gtest to link MSVCRT dynamically.")
add_subdirectory(${SQLITECPP_DIR}/googletest)
include_directories("${SQLITECPP_DIR}/googletest/googletest/include")

# add the unit test executable
add_executable(SQLiteCpp_tests ${SQLITECPP_TESTS})
target_compile_definitions(SQLiteCpp_tests PUBLIC SQLITE_HAS_CODEC=1)
target_link_libraries(SQLiteCpp_tests gtest_main SQLiteCpp sqlite3lib)
# Link target with dl for linux
if (UNIX AND NOT APPLE)
    target_link_libraries(SQLiteCpp_tests dl)
endif ()

# add a "test" target:
enable_testing()

# does the tests pass?
add_test(UnitTests SQLiteCpp_tests)
