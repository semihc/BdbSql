#PROJECT(BdbSql VERSION 0.1.0)
PROJECT(BdbSql)
cmake_minimum_required(VERSION 3.24)

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

#[[
add_definitions(-DSQLITE_ENABLE_RTREE)
add_definitions(-DSQLITE_ENABLE_FTS4)
add_definitions(-DSQLITE_ENABLE_FTS5)
add_definitions(-DSQLITE_ENABLE_JSON1)
add_definitions(-DSQLITE_ENABLE_RBU)
add_definitions(-DSQLITE_ENABLE_STAT4)
#]]
add_definitions(-D_HAVE_SQLITE_CONFIG_H)

# Uncomment this for single-threaded variant (faster)
#add_definitions(-DSQLITE_THREADSAFE=0)

if(WIN32)
  add_custom_command(TARGET dbsql3 POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy $<TARGET_FILE:dbsql3> ${CMAKE_BINARY_DIR}/dbsql.exe
    DEPENDS dbsql3
  )
  install(FILES dbsql.exe DESTINATION bin)
else()
  include(FindThreads)
  target_link_libraries(dbsql3 -L${_opt}/local/lib -ldb ${CMAKE_THREAD_LIBS_INIT} ${CMAKE_DL_LIBS})
  install(TARGETS dbsql3 RUNTIME DESTINATION bin)
endif()

install(FILES src/sqlite3.h DESTINATION include)
install(TARGETS dbsql LIBRARY DESTINATION lib ARCHIVE DESTINATION lib)