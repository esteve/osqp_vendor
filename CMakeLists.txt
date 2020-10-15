cmake_minimum_required(VERSION 3.9)

project(osqp_vendor)

find_package(ament_cmake REQUIRED)

macro(build_osqp)
  set(extra_cmake_args)

  get_property(multi_config GLOBAL PROPERTY GENERATOR_IS_MULTI_CONFIG)
  if(NOT multi_config AND DEFINED CMAKE_BUILD_TYPE)
    list(APPEND extra_cmake_args -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE})
  endif()

  list(APPEND extra_cmake_args -DBUILD_SHARED_LIBS=ON)
  list(APPEND extra_cmake_args -DBUILD_TESTING=OFF)
  # Disable MSVC warnings
  if(CMAKE_C_COMPILER_ID MATCHES "MSVC")
    set(win_c_flags "${CMAKE_C_FLAGS} /wd4244 /wd4267 /wd4996")
    list(APPEND extra_cmake_args -DCMAKE_C_FLAGS=${win_c_flags})
  endif()

  include(ExternalProject)
  externalproject_add(osqp-v0.6.1dev0
    URL https://github.com/oxfordcontrol/osqp/archive/v0.6.1dev0.zip
    URL_MD5 b595f0ab0735c04c7f6ed0a798a3eff7
    TIMEOUT 60
    CMAKE_ARGS
      -DCMAKE_INSTALL_PREFIX=${CMAKE_CURRENT_BINARY_DIR}/osqp_install
      ${extra_cmake_args}
  )

  # The external project will install to the build folder, but we'll install that on make install.
  install(
    DIRECTORY
      ${CMAKE_CURRENT_BINARY_DIR}/osqp_install/
    DESTINATION
      ${CMAKE_INSTALL_PREFIX}
    PATTERN config.h EXCLUDE
  )
endmacro()

build_osqp()

ament_export_libraries(osqp)
ament_export_dependencies(osqp)

if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  ament_lint_auto_find_test_dependencies()
endif()

ament_package(CONFIG_EXTRAS osqp_vendor-extras.cmake)
