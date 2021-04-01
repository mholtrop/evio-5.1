##########################################################
# CMake configuration for EVIO                           #
#                                                        #
# @author Maurik Holtrop, University of New Hampshire    #
#                                                        #
##########################################################

cmake_minimum_required(VERSION 3.8)
project(eviocxx VERSION 5.1 LANGUAGES CXX)

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
set(default_build_type "Release")
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
set(INSTALL_CMAKE_DIR lib/cmake CACHE PATH  "Installation directory evioConfig.cmake")
if( NOT INSTALL_CMAKE_DIR)
    set(INSTALL_CMAKE_DIR lib/cmake/eviocxx CACHE PATH  "Installation directory evioConfig.cmake")
else()
    set(INSTALL_CMAKE_DIR ${INSTALL_CMAKE_DIR}/eviocxx)
endif()

# Make relative paths absolute (needed later on)
foreach(p LIB BIN INCLUDE CMAKE CMAKECXX)
    set(var INSTALL_${p}_DIR)
    if(NOT IS_ABSOLUTE "${${var}}")
        set(${var} "${CMAKE_INSTALL_PREFIX}/${${var}}")
    endif()
endforeach()


set(INCLUDE_INSTALL_DIR ${INSTALL_INCLUDE_DIR}/eviocxx)
set(LIB_INSTALL_DIR ${INSTALL_LIB_DIR})

include_directories(${CMAKE_HOME_DIRECTORY}/src/libsrc)

message("---------------  ${PROJECT_NAME} ${PROJECT_VERSION} -------------------------")
message("We have CMAKE_BUILD_TYPE   = ${CMAKE_BUILD_TYPE}")
message("CMAKE_CXX_FLAGS            = ${CMAKE_CXX_FLAGS}")
message("INCLUDE_INSTALL_DIR        = ${INCLUDE_INSTALL_DIR}")
message("LIB_INSTALL_DIR            = ${LIB_INSTALL_DIR}")
message("INSTALL_CMAKE_DIR          = ${INSTALL_CMAKE_DIR}")
message("--------------------------------------------------")
#
# The C++ bindings for EVIO
#
set(SOURCE_FILES_EVIOCXX
  evioBankIndex.cc
  evioBufferChannel.cc
  evioFileChannel.cc
  evioSocketChannel.cc
  evioDictionary.cc
  evioException.cc
  evioUtil.cc
  )

set(HEADER_FILES_EVIOCXX
    evioBankIndex.hxx
    evioBufferChannel.hxx
    evioFileChannel.hxx
    evioSocketChannel.hxx
    evioChannel.hxx
    evioDictionary.hxx
    evioException.hxx
    evioTypedefs.hxx
    evioUtil.hxx
    evioUtilTemplates.hxx
    )

include(CMakePackageConfigHelpers)
write_basic_package_version_file(
        "${CMAKE_CURRENT_BINARY_DIR}/eviocxxConfigVersion.cmake"
        VERSION ${PROJECT_VERSION} COMPATIBILITY AnyNewerVersion)

add_library(eviocxx-objs  OBJECT ${SOURCE_FILES_EVIOCXX})
add_library(eviocxx       SHARED $<TARGET_OBJECTS:eviocxx-objs>)
add_library(eviocxx_static STATIC $<TARGET_OBJECTS:eviocxx-objs>)
set_target_properties(eviocxx_static PROPERTIES OUTPUT_NAME eviocxx)   # So that the lib is not called libhipo4_static.a

add_dependencies(eviocxx evio)
add_dependencies(eviocxx_static evio_static)

target_include_directories(eviocxx PUBLIC
                           $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
                           $<INSTALL_INTERFACE:${INCLUDE_INSTALL_DIR}> )

configure_package_config_file( cmake/eviocxxConfig.cmake.in
                               ${CMAKE_CURRENT_BINARY_DIR}/eviocxxConfig.cmake
                               INSTALL_DESTINATION ${INSTALL_CMAKE_DIR}
                               )

install(FILES ${CMAKE_CURRENT_BINARY_DIR}/eviocxxConfig.cmake ${CMAKE_CURRENT_BINARY_DIR}/eviocxxConfigVersion.cmake
        DESTINATION ${INSTALL_CMAKE_DIR})

target_link_libraries(eviocxx PUBLIC evio expat z pthread)

install(TARGETS eviocxx
        EXPORT  eviocxx-export
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION lib)

install(TARGETS eviocxx_static
        EXPORT  eviocxx-export
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION lib)

install(FILES ${HEADER_FILES_EVIOCXX}
        DESTINATION ${INCLUDE_INSTALL_DIR})

install(EXPORT eviocxx-export
        FILE eviocxxTargets.cmake
        DESTINATION ${INSTALL_CMAKE_DIR})

export(TARGETS eviocxx
       FILE "${PROJECT_BINARY_DIR}/eviocxxTargets.cmake")

# The following line will register the package in the *BUILD TREE* in the
# package registry. For production, this is probably not what we want.
# We want production code to build against the installed library target.
# For a DEBUG version however, it may be practical to build against the debug build tree.
#if(CMAKE_BUILD_TYPE STREQUAL "Debug" )
#    export(PACKAGE eviocxx)
#endif()
