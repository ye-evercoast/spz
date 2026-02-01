cmake_minimum_required(VERSION 3.10)

project(spz
  DESCRIPTION "A 3D Gaussians format"
  LANGUAGES C CXX
  VERSION 1.1.0)

include(GNUInstallDirs)

set(spz_sources
  "${CMAKE_CURRENT_SOURCE_DIR}/src/cc/load-spz.cc"
  "${CMAKE_CURRENT_SOURCE_DIR}/src/cc/splat-c-types.cc"
  "${CMAKE_CURRENT_SOURCE_DIR}/src/cc/splat-types.cc"
)

set(spz_headers
  "${CMAKE_CURRENT_SOURCE_DIR}/src/cc/load-spz.h"
  "${CMAKE_CURRENT_SOURCE_DIR}/src/cc/splat-c-types.h"
  "${CMAKE_CURRENT_SOURCE_DIR}/src/cc/splat-types.h"
)

# create the dynamic library and configure it
add_library(spz SHARED ${spz_sources} ${spz_headers})
add_library(spz::spz ALIAS spz)
target_compile_definitions(spz PRIVATE LIBSPZ_EXPORTS LIBSPZ_DLL)
if (WIN32)
  set_target_properties(spz PROPERTIES OUTPUT_NAME "libspz")
else()
  set_target_properties(spz PROPERTIES OUTPUT_NAME "spz")
endif()


target_include_directories(spz
  PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src/cc>
  INTERFACE $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
)

set_target_properties(spz PROPERTIES
  PUBLIC_HEADER "${spz_headers}"
  CXX_STANDARD 17
  CXX_STANDARD_REQUIRED ON
)

if(ANDROID)
    target_link_libraries(spz PRIVATE log)
endif()

if (IOS)
  set_target_properties(spz PROPERTIES
      FRAMEWORK TRUE
      FRAMEWORK_VERSION A
      XCODE_ATTRIBUTE_INSTALL_PATH "@rpath"
      PUBLIC_HEADER "${spz_headers}"
      OUTPUT_NAME "spz"
      XCODE_ATTRIBUTE_CODE_SIGN_IDENTITY ""
      XCODE_ATTRIBUTE_CODE_SIGNING_ALLOWED "NO"
      XCODE_ATTRIBUTE_CODE_SIGNING_REQUIRED "NO"
      MACOSX_FRAMEWORK_IDENTIFIER com.evercoast.libspz
      MACOSX_FRAMEWORK_SHORT_VERSION_STRING 1.0
      MACOSX_FRAMEWORK_BUNDLE_VERSION 1
  )
endif()

# Installation

## Install spzConfig.cmake spz::spz target can be found when calling find_package(spz)
include(CMakePackageConfigHelpers)
configure_package_config_file(
  "${CMAKE_CURRENT_SOURCE_DIR}/cmake/spzConfig.cmake.in"
  "${CMAKE_BINARY_DIR}/cmake/spzConfig.cmake"
  INSTALL_DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake/spz")
write_basic_package_version_file(
  "${CMAKE_BINARY_DIR}/cmake/spzConfigVersion.cmake"
  VERSION "${spz_VERSION}"
  COMPATIBILITY SameMajorVersion)
install(
  FILES
    "${CMAKE_BINARY_DIR}/cmake/spzConfig.cmake"
    "${CMAKE_BINARY_DIR}/cmake/spzConfigVersion.cmake"
  DESTINATION
    "${CMAKE_INSTALL_LIBDIR}/cmake/spz"
)

install(TARGETS spz
  EXPORT spzTargets
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
  FRAMEWORK DESTINATION Frameworks
  PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)

install(EXPORT spzTargets
  NAMESPACE spz::
  DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake/spz"
)
