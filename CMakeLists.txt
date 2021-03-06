cmake_minimum_required(VERSION 3.8)

if($ENV{TRAVIS_BUILD_NUMBER})
    project(bulkmt VERSION 0.0.$ENV{TRAVIS_BUILD_NUMBER})
else()
    project(bulkmt VERSION 0.0.1)
endif()

set(CMAKE_CXX_STANDARD 17)

include_directories("${PROJECT_SOURCE_DIR}/include")
include_directories(${CMAKE_BINARY_DIR})

add_library(logger src/logger.cpp include/logger.h)
add_library(command src/command_processor.cpp src/command_accumulator.cpp
        include/command_processor.h include/command_accumulator.h)
if ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
    target_link_libraries(logger pthread)
endif()
target_link_libraries(command logger)
add_executable(bulkmt src/main.cpp)
target_link_libraries(bulkmt logger command)

configure_file(test/project_path.h.in project_path.h)
find_package(GTest REQUIRED)
add_executable(test_bulkmt test/test_bulk.cpp)
target_link_libraries(test_bulkmt logger command ${GTEST_LIBRARIES})
if ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
    target_link_libraries(test_bulkmt pthread)
endif()

install(TARGETS bulkmt RUNTIME DESTINATION bin)

set(CPACK_GENERATOR DEB)

set(CPACK_PACKAGE_VERSION_MAJOR "${PROJECT_VERSION_MAJOR}")
set(CPACK_PACKAGE_VERSION_MINOR "${PROJECT_VERSION_MINOR}")
set(CPACK_PACKAGE_VERSION_PATCH "${PROJECT_VERSION_PATCH}")

set(CPACK_PACKAGE_CONTACT artbataev@gmail.com)

include(CPack)

enable_testing()
add_test(test test_bulkmt)
