##########################################################
# CMake configuration for EVIO  and EVIOCXX              #
#                                                        #
# @author Maurik Holtrop, University of New Hampshire    #
#                                                        #
##########################################################

cmake_minimum_required(VERSION 3.8)
project(EVIO VERSION 5.1 LANGUAGES C CXX)
#
# Some user choices
#
set(BUILD_EVIO_ONLY OFF CACHE BOOL "If ON build only the libevio and not the libeviocxx library.")
set(INSTALL_EXECUTABLE OFF CACHE BOOL "Install the executables also.")
# Offer the user the choice of overriding the installation directories
set(INSTALL_LIB_DIR lib CACHE PATH "Installation directory for libraries")
set(INSTALL_BIN_DIR bin CACHE PATH "Installation directory for executables")
set(INSTALL_INCLUDE_DIR include CACHE PATH  "Installation directory for header files")
set(INSTALL_CMAKE_DIR lib/cmake CACHE PATH  "Base Installation directory for Config.cmake files.")
# Make relative paths absolute (needed later on)
foreach(p LIB BIN INCLUDE CMAKE)
    set(var INSTALL_${p}_DIR)
    if(NOT IS_ABSOLUTE "${${var}}")
        set(${var} "${CMAKE_INSTALL_PREFIX}/${${var}}")
    endif()
endforeach()

# Set a default build type if none was specified
set(default_build_type "RELEASE")
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
    message(STATUS "Setting build type to '${default_build_type}' as none was specified.")
    set(CMAKE_BUILD_TYPE "${default_build_type}" CACHE STRING "Choose the type of build." FORCE)
    # Set the possible values of build type for cmake-gui
    set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS
                 "Debug" "Release" "MinSizeRel" "RelWithDebInfo")
endif()

add_subdirectory(src/libsrc)

if( NOT BUILD_EVIO_ONLY )
    #
    # The C++ bindings for EVIO
    #
    add_subdirectory(src/libsrc++)
endif()
#
# Now deal with some of the useful executables in the package.
#
if( INSTALL_EXECUTABLES )
    add_executable(evio2xml src/execsrc/evio2xml.c)
    add_dependencies(evio2xml eviostatic)
    target_link_libraries(evio2xml eviostatic expat z pthread)

    add_executable(eviocopy src/execsrc/eviocopy.c)
    add_dependencies(eviocopy eviostatic)
    target_link_libraries(eviocopy eviostatic pthread)

    add_executable(evioconcat src/execsrc/evioconcat.cc)
    add_dependencies(evioconcat eviostatic )
    target_link_libraries(evioconcat eviostatic pthread)

    add_executable(eviosplit src/execsrc/eviosplit.cc)
    add_dependencies(eviosplit eviostatic)
    target_link_libraries(eviosplit eviostatic pthread)

    install(TARGETS evio2xml eviocopy evioconcat eviosplit    DESTINATION bin)
endif()