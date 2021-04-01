##########################################################
# CMake configuration for EVIO                           #
#                                                        #
# @author Maurik Holtrop, University of New Hampshire    #
#                                                        #
##########################################################

cmake_minimum_required(VERSION 3.8)
project(evio VERSION 5.1 LANGUAGES C)

if(APPLE)
    # use, i.e. don't skip the full RPATH for the build tree
    set(CMAKE_SKIP_BUILD_RPATH  FALSE)

    # when building, don't use the install RPATH already
    # (but later on when installing)
    set(CMAKE_BUILD_WITH_INSTALL_RPATH FALSE)

    set(CMAKE_INSTALL_RPATH "${CMAKE_LIBRARY_OUTPUT_DIRECTORY}")

    # add the automatically determined parts of the RPATH
    # which point to directories outside the build tree to the install RPATH
    set(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)

    # the RPATH to be used when installing, but only if it's not a system directory
    list(FIND CMAKE_PLATFORM_IMPLICIT_LINK_DIRECTORIES "${CMAKE_LIBRARY_OUTPUT_DIRECTORY}" isSystemDir)
    IF("${isSystemDir}" STREQUAL "-1")
        set(CMAKE_INSTALL_RPATH "${CMAKE_LIBRARY_OUTPUT_DIRECTORY}")
    ENDIF("${isSystemDir}" STREQUAL "-1")
endif(APPLE)

# Set a default build type if none was specified
set(default_build_type "RELEASE")
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
    message(STATUS "Setting build type to '${default_build_type}' as none was specified.")
    set(CMAKE_BUILD_TYPE "${default_build_type}" CACHE STRING "Choose the type of build." FORCE)
    # Set the possible values of build type for cmake-gui
    set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS
                 "Debug" "Release" "MinSizeRel" "RelWithDebInfo")
endif()

# Offer the user the choice of overriding the installation directories
set(INSTALL_LIB_DIR lib CACHE PATH "Installation directory for libraries")
set(INSTALL_BIN_DIR bin CACHE PATH "Installation directory for executables")
set(INSTALL_INCLUDE_DIR include CACHE PATH  "Installation directory for header files")
if( NOT INSTALL_CMAKE_DIR)
    set(INSTALL_CMAKE_DIR lib/cmake/evio CACHE PATH  "Installation directory evioConfig.cmake")
else()
    set(INSTALL_CMAKE_DIR ${INSTALL_CMAKE_DIR}/evio)
endif()

# Make relative paths absolute (needed later on)
foreach(p LIB BIN INCLUDE CMAKE)
    set(var INSTALL_${p}_DIR)
    if(NOT IS_ABSOLUTE "${${var}}")
        set(${var} "${CMAKE_INSTALL_PREFIX}/${${var}}")
    endif()
endforeach()

set(INCLUDE_INSTALL_DIR ${INSTALL_INCLUDE_DIR}/evio)
set(LIB_INSTALL_DIR ${INSTALL_LIB_DIR})

message("--------------- ${PROJECT_NAME} ${PROJECT_VERSION} -------------------------")
message("We have CMAKE_BUILD_TYPE   = ${CMAKE_BUILD_TYPE}")
message("CMAKE_CXX_FLAGS            = ${CMAKE_CXX_FLAGS}")
message("INCLUDE_INSTALL_DIR        = ${INCLUDE_INSTALL_DIR}")
message("LIB_INSTALL_DIR            = ${LIB_INSTALL_DIR}")
message("INSTALL_CMAKE_DIR          = ${INSTALL_CMAKE_DIR}")
message("--------------------------------------------------")

set(SOURCE_FILES_EVIO
    msinttypes.h
    evio.c
    eviofmt.c
    eviofmtdump.c
    eviofmtswap.c
    evioswap.c
    evioBankUtil.c
  )

set(HEADER_FILES_EVIO
    evio.h
    evioBankUtil.h
    )

include(CMakePackageConfigHelpers)
# Write a package versioning file based on the project version (see top)
write_basic_package_version_file(
        "${CMAKE_CURRENT_BINARY_DIR}/evioConfigVersion.cmake"
        VERSION ${PROJECT_VERSION} COMPATIBILITY AnyNewerVersion)

#Generate the shared library from the sources
add_library(evio_objs  OBJECT ${SOURCE_FILES_EVIO})
add_library(evio       SHARED $<TARGET_OBJECTS:evio_objs>)
add_library(evio_static STATIC $<TARGET_OBJECTS:evio_objs>)
set_target_properties(evio_static PROPERTIES OUTPUT_NAME evio)   # So that the lib is not called libhipo4_static.a

target_include_directories(evio
                           PUBLIC
                           $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
                           $<INSTALL_INTERFACE:${INCLUDE_INSTALL_DIR}>)

configure_package_config_file( cmake/evioConfig.cmake.in
                               ${CMAKE_CURRENT_BINARY_DIR}/evioConfig.cmake
                               INSTALL_DESTINATION ${INSTALL_CMAKE_DIR}
                               )

install(FILES ${CMAKE_CURRENT_BINARY_DIR}/evioConfig.cmake  ${CMAKE_CURRENT_BINARY_DIR}/evioConfigVersion.cmake
        DESTINATION ${INSTALL_CMAKE_DIR})

target_link_libraries(evio PUBLIC)

install(TARGETS evio
        EXPORT  evio-export
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION lib)

install(TARGETS evio_static
        EXPORT  evio-export
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION lib)

install(FILES ${HEADER_FILES_EVIO}
        DESTINATION ${INCLUDE_INSTALL_DIR})

# Place the evioTargets.cmake in the INSTALL area, using INSTALL_INTERFACE
install(EXPORT evio-export
        FILE evioTargets.cmake
        DESTINATION ${INSTALL_CMAKE_DIR})

# Also place a copy of evioTargets.cmake in the build tree, using the BUILD-INTERFACE
export(TARGETS evio
       FILE "${PROJECT_BINARY_DIR}/evioTargets.cmake")

# The following line will register the package in the *BUILD TREE* in the
# package registry. This is probably not what we want. We want any code building against this
# to build against the installed items.
# export(PACKAGE evio)

