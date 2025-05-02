cmake_minimum_required (VERSION 3.21)

set(MYPROJ_VERSION 0.1.0)

project(myproj 
  VERSION ${MYPROJ_VERSION} 
  DESCRIPTION "A config populator" 
  LANGUAGES CXX)

list(APPEND CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake/modules)

if(PROJECT_IS_TOP_LEVEL) 
  set(CMAKE_CXX_STANDARD 23)
  set(CMAKE_CXX_STANDARD_REQUIRED ON)
  set(CMAKE_CXX_EXTENSIONS OFF)
endif()

add_library(myproj INTERFACE)
add_library(myproj::myproj ALIAS myproj)

include(GNUInstallDirs)

target_include_directories( myproj
    INTERFACE
        $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/src>
        $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
)

target_compile_features(myproj INTERFACE cxx_std_23)

find_package(Boost REQUIRED) # Requires header-only boost::pfr
target_include_directories(myproj INTERFACE ${Boost_INCLUDE_DIRS})

option(MYPROJ_BUILD_TESTS "Enable building tests." OFF)
option(MYPROJ_ENABLE_INSTALL "Enable the install target" ON)

option(MYPROJ_USE_SANITIZERS "Enable sanitizers by adding -fsanitize=address -fno-omit-frame-pointer -fsanitize=undefined flags if available." OFF)
option(MYPROJ_USE_STATIC_ANALYZERS "Enable static analyzers" OFF)
option(MYPROJ_DEV_MODE "Set defaults for developing." OFF)

if(MYPROJ_DEV_MODE)
  set(MYPROJ_USE_SANITIZERS ON)
  set(MYPROJ_USE_STATIC_ANALYZERS ON)
  set(MYPROJ_BUILD_TESTS ON)
  set(CMAKE_EXPORT_COMPILE_COMMANDS ON) # for LSP support
endif()

if(MYPROJ_USE_SANITIZERS)
  if(CMAKE_CXX_COMPILER_ID MATCHES "Clang|GNU")
      target_compile_options(myproj INTERFACE $<$<CONFIG:Debug>:-fsanitize=address -fno-omit-frame-pointer -fsanitize=undefined>)
      target_link_libraries(myproj INTERFACE $<$<CONFIG:Debug>:-fsanitize=address -fno-omit-frame-pointer -fsanitize=undefined>)
  else()
    message("Sanitizers are only enabled for gcc and clang")
  endif()
endif()

if(MYPROJ_USE_STATIC_ANALYZERS)
  set(MYPROJ_ENABLE_CPPCHECK ON) 
  set(MYPROJ_ENABLE_INCLUDE_WHAT_YOU_USE ON)
  include(StaticAnalyzers)
endif()

#
# Install myproj
#
if(MYPROJ_ENABLE_INSTALL)
  include(CMakePackageConfigHelpers)

  install(
      TARGETS myproj
      EXPORT myprojTargets
  )

  install(
      EXPORT myprojTargets
      DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/myproj
      NAMESPACE myproj::
  )

  configure_package_config_file(
      ${PROJECT_SOURCE_DIR}/cmake/in/myprojConfig.cmake.in
      myprojConfig.cmake
      INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/myproj
  )

  write_basic_package_version_file(
      ${PROJECT_BINARY_DIR}/myprojConfigVersion.cmake
      VERSION ${PROJECT_VERSION}
      COMPATIBILITY AnyNewerVersion
  )

  install(
      FILES
          ${PROJECT_BINARY_DIR}/myprojConfig.cmake
          ${PROJECT_BINARY_DIR}/myprojConfigVersion.cmake
      DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/myproj
  )

  install( DIRECTORY ${PROJECT_SOURCE_DIR}/src/
    DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}"
    FILES_MATCHING 
    PATTERN "*.hpp"
  )
endif()

if(MYPROJ_BUILD_TESTS)
  add_subdirectory(test)
endif()
