cmake_minimum_required(VERSION 3.24)
#PROJECT(BdbSql VERSION 0.1.0)
PROJECT(BdbSql LANGUAGES C CXX)
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED OFF)


include_directories(${CMAKE_SOURCE_DIR}/src)
add_library(dbsql STATIC src/sqlite3.c src/sqlite3.h)
add_executable(dbsql3 src/sqlite3.c src/shell.c src/sqlite3.h)

# Common inc/lib/src locations by default
set(_opt /opt)
set(_opt_TC ${_opt}/TC)
set(_opt_TC_ext ${_opt_TC}/ext)

# Project source directory
set(PrjSrc ${CMAKE_SOURCE_DIR}/src)


# Platform defaults and settings
message(STATUS "CMAKE_HOST_SYSTEM_NAME: ${CMAKE_HOST_SYSTEM_NAME}")
message(STATUS "CMAKE_HOST_SYSTEM_VERSION: ${CMAKE_HOST_SYSTEM_VERSION}")

if(${CMAKE_HOST_SYSTEM_NAME} STREQUAL "Linux") 
  message(STATUS "Linux detected.")
  include_directories(${_opt_TC}/local/include ${_opt}/local/include)
  link_directories(${_opt_TC}/local/lib ${_opt}/local/lib)
endif()


# Identify compiler:
if (CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
  # using Gcc
  # lots of warnings and all warnings as errors
  #add_compile_options(-Wall -Wextra -pedantic -Werror)
elseif (CMAKE_CXX_COMPILER_ID STREQUAL "MSVC")
  # using Visual Studio C++

  # Berkeley DB
  set(bdb_prj ${_opt_TC_ext}/db-18.1.40)
  set(bdb_inc ${bdb_prj}/build_windows)
  set(bdb_stl_inc ${bdb_prj}/lang/cxx/stl)
  set(bdb_lib ${bdb_prj}/build_windows/x64/Debug)
  add_library(bdblib STATIC IMPORTED)
  set_target_properties(bdblib PROPERTIES IMPORTED_LOCATION ${bdb_lib}/libdb181d.lib)
  add_library(bdbstllib STATIC IMPORTED)
  set_target_properties(bdbstllib PROPERTIES IMPORTED_LOCATION ${bdb_lib}/libdb_stl181d.lib)

  # Needed for combining PDB files into single directory (WinDbg support)
  set(pdb_out ${CMAKE_CURRENT_BINARY_DIR}/pdb)
  set(CMAKE_PDB_OUTPUT_DIRECTORY ${pdb_out})
  set(CMAKE_COMPILE_PDB_OUTPUT_DIRECTORY ${pdb_out})
  # end of MSVC
elseif (CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
 # using Clang/LLVM
elseif (CMAKE_CXX_COMPILER_ID STREQUAL "Intel")
 # using Intel C++
else()
  message(WARNING "Unknown compiler!") 
endif()





#[[
add_definitions(-DSQLITE_ENABLE_RTREE)
add_definitions(-DSQLITE_ENABLE_FTS4)
add_definitions(-DSQLITE_ENABLE_FTS5)
add_definitions(-DSQLITE_ENABLE_JSON1)
add_definitions(-DSQLITE_ENABLE_RBU)
add_definitions(-DSQLITE_ENABLE_STAT4)
#]]
# Uncomment this for single-threaded variant (faster)
#add_definitions(-DSQLITE_THREADSAFE=0)

if(WIN32)
  add_definitions(/D SQLITE_EXPORTS /D _CRT_SECURE_NO_WARNINGS /D _CRT_NONSTDC_NO_DEPRECATE /D SQLITE_CORE /D SQLITE_ENABLE_COLUMN_METADATA /D SQLITE_ENABLE_FTS3 /D SQLITE_ENABLE_LOAD_EXTENSION /D SQLITE_DEBUG /D SQLITE_MEMDEBUG /D "SQLITE_API=__declspec(dllexport)" )
  #add_definitions(-DSQLITE_OS_WIN=1)
  #add_definitions(-DSQLITE_OS_UNIX=0)

  include_directories(${bdb_inc})
  target_link_libraries(dbsql3 PUBLIC bdblib)
  #[[ Evaluate later:
  add_custom_command(TARGET dbsql3 POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy $<TARGET_FILE:dbsql3> ${CMAKE_BINARY_DIR}/dbsql.exe
    DEPENDS dbsql3
  )
  install(FILES dbsql.exe DESTINATION bin)
  #]]
else()
  add_definitions(-D_HAVE_SQLITE_CONFIG_H)
  include(FindThreads)
  target_link_libraries(dbsql3 -L${_opt}/local/lib -ldb ${CMAKE_THREAD_LIBS_INIT} ${CMAKE_DL_LIBS})
  install(TARGETS dbsql3 RUNTIME DESTINATION bin)
endif()

install(FILES src/sqlite3.h DESTINATION include)
install(TARGETS dbsql LIBRARY DESTINATION lib ARCHIVE DESTINATION lib)