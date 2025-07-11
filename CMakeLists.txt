cmake_minimum_required(VERSION 3.23)
if (NOT ${CMAKE_VERSION} VERSION_LESS 3.27)
    cmake_policy(VERSION 3.27)
else()
    # This is mostly for setting the X_ROOT flags?
    message(STATUS "We recommend using cmake version 3.27 or newer.")
endif()


# set the project name and version
project(storm-project-starter VERSION 1.11)

# The following two options are useful for example in a CI to disable using one of the two ways of obtaining storm.
set(STORM_ROOT  "" CACHE STRING "Additional hint where to look for storm.")
option(ALWAYS_FETCH_STORM "If ON, skip search for an installed version of Storm." OFF)
option(NEVER_FETCH_STORM "If ON, skip fetching storm from a repo." OFF)
if (ALWAYS_FETCH_STORM AND NEVER_FETCH_STORM)
    message(FATAL_ERROR "ALWAYS_FETCH_STORM and NEVER_FETCH_STORM are mutually exclusive.")
endif()

# search for Storm library
if (NOT ALWAYS_FETCH_STORM)
    # The hint is no longer necessary due to CMP0144 from cmake 3.27 onwards.
    find_package(storm HINTS ${STORM_ROOT})
    if (storm_FOUND)
        message(STATUS "Found storm ${storm_VERSION} at ${storm_DIR}.")
    endif()
    if (storm_FOUND AND NOT STORM_HAVE_Z3)
        message(WARNING "This starter project requires a version of storm with z3 installed.")
    endif()
endif()

if(NOT TARGET storm)
    if(NOT NEVER_FETCH_STORM)
        message(STATUS "Fetching storm.")
        include(FetchContent)
        SET(FETCHCONTENT_QUIET OFF)
        FetchContent_Declare(
                storm
                GIT_REPOSITORY https://github.com/moves-rwth/storm.git
                GIT_TAG        master
        )
        FETCHCONTENT_MAKEAVAILABLE(storm)
        include(${storm_BINARY_DIR}/stormOptions.cmake)
    else()
        message(FATAL_ERROR "Storm not included.")
    endif()
endif()

# specify the C++ standard
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED True)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin")
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/lib")
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/lib")


#############################################################
##
##	RPATH settings
#############################################################

# don't skip the full RPATH for the build tree
SET(CMAKE_SKIP_BUILD_RPATH  FALSE)

# when building, don't use the install RPATH already (but only when installing)
SET(CMAKE_BUILD_WITH_INSTALL_RPATH FALSE)

if (MACOSX)
    SET(RPATH_ORIGIN_EQUIV "@loader_path")
else()
    SET(RPATH_ORIGIN_EQUIV "$ORIGIN")
endif()
# the RPATH to be used when installing
# Some parts here feel like a hack
# TODO move these to the right targets rather than setting them globally.
SET(CMAKE_INSTALL_RPATH "${CMAKE_INSTALL_PREFIX}/lib;${CMAKE_INSTALL_PREFIX}/lib/storm;${CMAKE_INSTALL_PREFIX}/lib/storm/resources;${RPATH_ORIGIN_EQUIV};${RPATH_ORIGIN_EQUIV}/resources")
# TODO this is less than ideal, but we had multiple issues with spot 2.13.1 in 25/06.
SET(CMAKE_BUILD_RPATH "${STORM_3RDPARTY_BINARY_DIR}/spot/lib")

# add the automatically determined parts of the RPATH
# which point to directories outside the build tree to the install RPATH
SET(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)

# specify main source file
set(MAIN_FILE src/main.cpp)

# set executable
add_executable(starter-project ${MAIN_FILE})

# Set include directories and dependencies
target_link_libraries(starter-project PRIVATE storm storm-parsers)
# Note that other libraries/includes from Storm might need to be added depending on the used functionality.
