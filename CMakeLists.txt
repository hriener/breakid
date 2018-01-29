cmake_minimum_required(VERSION 3.0 FATAL_ERROR)

project(breakid)

#--------------------------------------------------------------------------------------------------
# Configurable options:

option(STATIC_BINARIES "Link binaries statically." ON)
option(USE_SORELEASE   "Use SORELEASE in shared library filename." ON)

#--------------------------------------------------------------------------------------------------
# Library version:

set(BREAKID_SOMAJOR   2)
set(BREAKID_SOMINOR   3)
set(BREAKID_SORELEASE 0)

# Compute VERSION and SOVERSION:
if (USE_SORELEASE)
  set(BREAKID_VERSION ${BREAKID_SOMAJOR}.${BREAKID_SOMINOR}.${BREAKID_SORELEASE})
else()
  set(BREAKID_VERSION ${BREAKID_SOMAJOR}.${BREAKID_SOMINOR})
endif()
set(BREAKID_SOVERSION ${BREAKID_SOMAJOR})

#--------------------------------------------------------------------------------------------------
# Dependencies:

include_directories(${breakid_SOURCE_DIR}/src/)

#--------------------------------------------------------------------------------------------------
# Source files:

set(BREAKID_LIB_CPP_SOURCES
    src/Algebraic.cpp
    src/Breaking.cpp
    src/Graph.cpp
    src/Theory.cpp
    src/global.cpp)

set(SAUCY_LIB_SOURCES
    src/saucy.c)

set(BREAKID_LIB_SOURCES
    ${BREAKID_LIB_CPP_SOURCES}
    ${SAUCY_LIB_SOURCES})

#--------------------------------------------------------------------------------------------------
# Compile flags:

add_definitions(-O3 -DVERSION="${BREAKID_VERSION}" -DNDEBUG -std=c++11)
set_source_files_properties(${BREAKID_LIB_CPP_SOURCES} PROPERTIES COMPILE_FLAGS "")
set_source_files_properties(${SAUCY_LIB_SOURCES} PROPERTIES LANGUAGE CXX)

#--------------------------------------------------------------------------------------------------
# Build Targets:

add_library(breakid-lib-static STATIC ${BREAKID_LIB_SOURCES})
add_library(breakid-lib-shared SHARED ${BREAKID_LIB_SOURCES})

target_include_directories(breakid-lib-static PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})
target_include_directories(breakid-lib-shared PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})

target_link_libraries(breakid-lib-static ${ZLIB_LIBRARY_RELEASE})
target_link_libraries(breakid-lib-shared ${ZLIB_LIBRARY_RELEASE})

add_executable(breakid src/BreakID.cpp)

if(STATIC_BINARIES)
  target_link_libraries(breakid breakid-lib-static)
else()
  target_link_libraries(breakid breakid-lib-shared)
endif()

set_target_properties(breakid-lib-static PROPERTIES OUTPUT_NAME "breakid")
set_target_properties(breakid-lib-shared
  PROPERTIES
    OUTPUT_NAME "breakid" 
    VERSION ${BREAKID_VERSION}
    SOVERSION ${BREAKID_SOVERSION})

set_target_properties(breakid       PROPERTIES OUTPUT_NAME "breakid")

#--------------------------------------------------------------------------------------------------
# Installation targets:

install(TARGETS breakid-lib-static breakid-lib-shared breakid 
        RUNTIME DESTINATION bin
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION lib)

install(DIRECTORY src/
        DESTINATION include/breakid
        FILES_MATCHING PATTERN "*.h")
