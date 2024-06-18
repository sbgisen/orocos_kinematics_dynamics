#
# Test CMake version
#
cmake_minimum_required(VERSION 3.0.2)

###################################################
#                                                 #
#       Start project customization section       #
#                                                 #
###################################################

project(orocos_kdl)

set(KDL_VERSION 1.5.1)
string(
  REGEX MATCHALL
        "[0-9]+"
        KDL_VERSIONS
        ${KDL_VERSION}
)
list(
  GET
  KDL_VERSIONS
  0
  KDL_VERSION_MAJOR
)
list(
  GET
  KDL_VERSIONS
  1
  KDL_VERSION_MINOR
)
list(
  GET
  KDL_VERSIONS
  2
  KDL_VERSION_PATCH
)

message(STATUS "Orocos KDL version ${VERSION} (${KDL_VERSION_MAJOR}.${KDL_VERSION_MINOR}.${KDL_VERSION_PATCH})")

set(PROJ_SOURCE_DIR ${orocos_kdl_SOURCE_DIR})
set(PROJ_BINARY_DIR ${orocos_kdl_BINARY_DIR})

# catkin-specific configuration (optional)
find_package(catkin QUIET)
if(catkin_FOUND)
  catkin_package(SKIP_CMAKE_CONFIG_GENERATION SKIP_PKG_CONFIG_GENERATION)
endif()

if(NOT CMAKE_INSTALL_PREFIX)
  set(CMAKE_INSTALL_PREFIX
      /usr/local/
      CACHE PATH
            "Installation directory"
            FORCE
  )
  message(STATUS "Setting installation directory to ${CMAKE_INSTALL_PREFIX}")
endif(NOT CMAKE_INSTALL_PREFIX)

set(CMAKE_VERSION "${CMAKE_MAJOR_VERSION}.${CMAKE_MINOR_VERSION}.${CMAKE_PATCH_VERSION}")

if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE
      Release
      CACHE
        STRING
        "Choose the type of build, options are: None(CMAKE_CXX_FLAGS or CMAKE_C_FLAGS used) Debug Release RelWithDebInfo MinSizeRel."
        FORCE
  )
  message(STATUS "Setting build type to '${CMAKE_BUILD_TYPE}'")
else(NOT CMAKE_BUILD_TYPE)
  message(STATUS "Build type set to '${CMAKE_BUILD_TYPE}' by user.")
endif(NOT CMAKE_BUILD_TYPE)

set(KDL_CFLAGS "")

find_package(Eigen3 QUIET)
if(NOT EIGEN3_FOUND)
  include(${PROJ_SOURCE_DIR}/cmake/FindEigen3.cmake)
endif()
include_directories(${EIGEN3_INCLUDE_DIR})
set(KDL_CFLAGS "${KDL_CFLAGS} -I\"${EIGEN3_INCLUDE_DIR}\"")

# Check the platform STL containers capabilities
include(cmake/CheckSTLContainers.cmake)
check_stl_containers()

# Set the default option appropriately
if(HAVE_STL_CONTAINER_INCOMPLETE_TYPES)
  set(KDL_USE_NEW_TREE_INTERFACE_DEFAULT OFF)
else(HAVE_STL_CONTAINER_INCOMPLETE_TYPES)
  set(KDL_USE_NEW_TREE_INTERFACE_DEFAULT ON)
endif(HAVE_STL_CONTAINER_INCOMPLETE_TYPES)

# Allow the user to select the Tree API version to use
set(KDL_USE_NEW_TREE_INTERFACE
    ${KDL_USE_NEW_TREE_INTERFACE_DEFAULT}
    CACHE BOOL "Use the new KDL Tree interface"
)

#Sanity check, inform the user
if(NOT HAVE_STL_CONTAINER_INCOMPLETE_TYPES AND NOT KDL_USE_NEW_TREE_INTERFACE)
  message(WARNING "You have chosen to use the current Tree Interface, but your platform doesn't support containers of "
                  "incomplete types, this configuration is likely invalid"
  )
endif()

# The new interface requires the use of shared pointers
if(KDL_USE_NEW_TREE_INTERFACE)
  # We need shared_ptr from boost since not all compilers are c++11 capable
  find_package(Boost REQUIRED)
  include_directories(${Boost_INCLUDE_DIRS})
endif(KDL_USE_NEW_TREE_INTERFACE)

option(
  ENABLE_TESTS
  OFF
  "Enable building of tests"
)
if(ENABLE_TESTS)
  # If not in standard paths, set CMAKE_xxx_PATH's in environment, eg.
  # export CMAKE_INCLUDE_PATH=/opt/local/include
  # export CMAKE_LIBRARY_PATH=/opt/local/lib
  find_library(CPPUNIT cppunit)
  set(CPPUNIT ${CPPUNIT} "dl")
  find_path(CPPUNIT_HEADERS cppunit/TestRunner.h)
  if(CPPUNIT AND CPPUNIT_HEADERS)
    message(STATUS "-- Looking for Cppunit - found")
  else(CPPUNIT AND CPPUNIT_HEADERS)
    message(FATAL_ERROR "-- Looking for Cppunit - not found")
  endif(CPPUNIT AND CPPUNIT_HEADERS)
endif(ENABLE_TESTS)

option(
  ENABLE_EXAMPLES
  OFF
  "Enable building of examples"
)

add_subdirectory(doc)
add_subdirectory(src)
add_subdirectory(tests)
add_subdirectory(models)
add_subdirectory(examples)

export(TARGETS orocos-kdl FILE "${PROJECT_BINARY_DIR}/OrocosKDLTargets.cmake")

export(PACKAGE orocos_kdl)

# Generate CMake package configuration
configure_file(
  orocos_kdl-config.cmake.in
  ${PROJECT_BINARY_DIR}/orocos_kdl-config.cmake
  @ONLY
)
configure_file(
  orocos_kdl-config-version.cmake.in
  ${PROJECT_BINARY_DIR}/orocos_kdl-config-version.cmake
  @ONLY
)

install(
  FILES cmake/FindEigen3.cmake
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/orocos_kdl
)
install(
  FILES ${PROJECT_BINARY_DIR}/orocos_kdl-config.cmake
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/orocos_kdl
)
install(
  FILES ${PROJECT_BINARY_DIR}/orocos_kdl-config-version.cmake
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/orocos_kdl
)
install(
  EXPORT OrocosKDLTargets
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/orocos_kdl
)
# Generate pkg-config package configuration
configure_file(
  orocos_kdl.pc.in
  ${CMAKE_CURRENT_BINARY_DIR}/orocos-kdl.pc
  @ONLY
)
configure_file(
  orocos_kdl.pc.in
  ${CMAKE_CURRENT_BINARY_DIR}/orocos_kdl.pc
  @ONLY
)

install(
  FILES ${CMAKE_CURRENT_BINARY_DIR}/orocos-kdl.pc
  DESTINATION lib${LIB_SUFFIX}/pkgconfig
)
install(
  FILES ${CMAKE_CURRENT_BINARY_DIR}/orocos_kdl.pc
  DESTINATION lib${LIB_SUFFIX}/pkgconfig
)
