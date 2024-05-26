cmake_minimum_required(VERSION 3.22...3.28)

project(
  argon2
  VERSION 0
  LANGUAGES C
)

SET(SOVERSION 1)

if(PROJECT_SOURCE_DIR STREQUAL PROJECT_BINARY_DIR)
  message(
    FATAL_ERROR
      "In-source builds not allowed. Please make a new directory (called a build directory) and run CMake from there."
  )
endif()

option(BUILD_SHARED_LIBS "Build using shared libraries" ON)

include(GNUInstallDirs)

# ---- Add dependencies via CPM ----
# see https://github.com/TheLartians/CPM.cmake for more info

include(cmake/CPM.cmake)

# PackageProject.cmake will be used to make our target installable
CPMAddPackage("gh:TheLartians/PackageProject.cmake@1.11.2")

find_package(Threads)
find_package(PkgConfig)

# ---- Add source files ----

# Note: globbing sources is considered bad practice as CMake's generators may not detect new files
# automatically. Keep that in mind when changing files, or explicitly mention them here.
set(headers
  "${CMAKE_SOURCE_DIR}/include/argon2.h"
)
set(sources
  "${CMAKE_SOURCE_DIR}/src/argon2.c"
  "${CMAKE_SOURCE_DIR}/src/core.c"
  "${CMAKE_SOURCE_DIR}/src/blake2/blake2b.c"
  "${CMAKE_SOURCE_DIR}/src/thread.c"
  "${CMAKE_SOURCE_DIR}/src/encoding.c"
)

list(APPEND sources
  "${CMAKE_SOURCE_DIR}/src/ref.c"
)

add_library(${PROJECT_NAME} ${headers} ${sources})
set_target_properties(${PROJECT_NAME} PROPERTIES C_STANDARD 90)

if (NOT Threads_FOUND)
  target_compile_definitions(${PROJECT_NAME} PUBLIC ARGON2_NO_THREADS)
else()
  target_link_libraries(${PROJECT_NAME} PUBLIC Threads::Threads)
endif()

# being a cross-platform target, we enforce standards conformance on MSVC
target_compile_options(${PROJECT_NAME} PUBLIC "$<$<COMPILE_LANG_AND_ID:CXX,MSVC>:/permissive->")

target_include_directories(
  ${PROJECT_NAME} PUBLIC $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include>
                         $<INSTALL_INTERFACE:include/${PROJECT_NAME}-${PROJECT_VERSION}>
)

# ---- Configure pkg-config ----
set(PREFIX ${CMAKE_INSTALL_PREFIX})
set(UPSTREAM_VER "ZERO")
set(HOST_MULTIARCH "${CMAKE_INSTALL_LIBDIR}/${PROJECT_NAME}")
set(INCLUDE "${CMAKE_INSTALL_INCLUDEDIR}/${PROJECT_NAME}")
set(EXTRA_LIBS "")
# get_target_property(EXTRA_LIBS ${PROJECT_NAME} LINK_LIBRARIES)
configure_file(
        "${argon2_SOURCE_DIR}/libargon2.pc.in"
        "${argon2_BINARY_DIR}/libargon2.pc"
        @ONLY)

install(FILES "${argon2_BINARY_DIR}/libargon2.pc"
  DESTINATION "${CMAKE_INSTALL_LIBDIR}/pkgconfig")

# ---- Create an installable target ----
# this allows users to install and find the library via `find_package()`.

# the location where the project's version header will be placed should match the project's regular
# header paths
string(TOLOWER version.h VERSION_HEADER_LOCATION)

packageProject(
  NAME ${PROJECT_NAME}
  VERSION ${PROJECT_VERSION}
  NAMESPACE ${PROJECT_NAME}
  BINARY_DIR ${PROJECT_BINARY_DIR}
  INCLUDE_DIR ${PROJECT_SOURCE_DIR}/include
  INCLUDE_DESTINATION include/${PROJECT_NAME}
  VERSION_HEADER "${VERSION_HEADER_LOCATION}"
  DISABLE_VERSION_SUFFIX YES
)
