# CMake configuration for PROJ unit tests

# FIXME: Deal with our old-school CMakeLists.txt behaving badly
set(_save_c_flags "${CMAKE_C_FLAGS}")
set(_save_cxx_flags "${CMAKE_CXX_FLAGS}")
string(REGEX REPLACE "\\-W[a-z\\-]+" "" CMAKE_C_FLAGS ${CMAKE_C_FLAGS})
string(REGEX REPLACE "\\-W[a-z\\-]+" "" CMAKE_CXX_FLAGS ${CMAKE_CXX_FLAGS})

#
# Build Google Test
#
# Source https://github.com/google/googletest/blob/master/googletest/README.md
# Download and unpack googletest at configure time
configure_file(
    ${CMAKE_SOURCE_DIR}/test/googletest/CMakeLists.txt.in
    ${CMAKE_BINARY_DIR}/googletest-download/CMakeLists.txt)
execute_process(COMMAND ${CMAKE_COMMAND} -G "${CMAKE_GENERATOR}" .
  RESULT_VARIABLE result
  WORKING_DIRECTORY ${CMAKE_BINARY_DIR}/googletest-download )
if(result)
  message(FATAL_ERROR "CMake step for googletest failed: ${result}")
endif()
execute_process(COMMAND ${CMAKE_COMMAND} --build .
  RESULT_VARIABLE result
  WORKING_DIRECTORY ${CMAKE_BINARY_DIR}/googletest-download)
if(result)
  message(FATAL_ERROR "Build step for googletest failed: ${result}")
endif()
# Prevent overriding the parent project's compiler/linker
# settings on Windows
set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)
# Add googletest directly to our build. This defines
# the gtest and gtest_main targets.
add_subdirectory(${CMAKE_BINARY_DIR}/googletest-src
                 ${CMAKE_BINARY_DIR}/googletest-build
                 EXCLUDE_FROM_ALL)
# The gtest/gtest_main targets carry header search path
# dependencies automatically when using CMake 2.8.11 or
# later. Otherwise we have to add them here ourselves.
if (CMAKE_VERSION VERSION_LESS 2.8.11)
  include_directories("${gtest_SOURCE_DIR}/include")
endif()

# FIXME: Deal with our old-school CMakeLists.txt behaving badly
set(CMAKE_C_FLAGS "${_save_c_flags}")
set(CMAKE_CXX_FLAGS "${_save_cxx_flags}")
unset(_save_c_flags)
unset(_save_cxx_flags)

#
# Build PROJ unit tests
#

if("${CMAKE_C_COMPILER_ID}" STREQUAL "MSVC" AND BUILD_LIBPROJ_SHARED)
add_definitions(-DPROJ_MSVC_DLL_IMPORT=1)
endif()

include_directories(${CMAKE_SOURCE_DIR}/include)
include_directories(${SQLITE3_INCLUDE_DIR})

add_executable(proj_pj_transform_test
  main.cpp
  pj_transform_test.cpp)
target_link_libraries(proj_pj_transform_test
  gtest
  ${PROJ_LIBRARIES})
add_test(NAME proj_pj_transform_test COMMAND proj_pj_transform_test)

add_executable(proj_errno_string_test
  main.cpp
  proj_errno_string_test.cpp)
target_link_libraries(proj_errno_string_test
  gtest
  ${PROJ_LIBRARIES})
add_test(NAME proj_errno_string_test COMMAND proj_errno_string_test)


if (MSVC AND BUILD_LIBPROJ_SHARED)
# ph_phi2_test not compatible of a .dll build
else()
add_executable(pj_phi2_test
        main.cpp
        pj_phi2_test.cpp)
target_link_libraries(pj_phi2_test
        gtest
        ${PROJ_LIBRARIES})
add_test(NAME pj_phi2_test COMMAND pj_phi2_test)
endif()

add_executable(proj_test_cpp_api
  main.cpp
  test_util.cpp
  test_common.cpp
  test_crs.cpp
  test_metadata.cpp
  test_io.cpp
  test_operation.cpp
  test_datum.cpp
  test_factory.cpp
  test_c_api.cpp)
target_link_libraries(proj_test_cpp_api
  gtest
  ${PROJ_LIBRARIES}
  ${SQLITE3_LIBRARY})
add_test(NAME proj_test_cpp_api COMMAND proj_test_cpp_api)

add_executable(gie_self_tests
  main.cpp
  gie_self_tests.cpp)
target_link_libraries(gie_self_tests
  gtest
  ${PROJ_LIBRARIES})
add_test(NAME gie_self_tests COMMAND gie_self_tests)
