if (POLICY CMP0025)
    # Compiler id for Apple Clang is now AppleClang
    cmake_policy(SET CMP0025 NEW)
endif()

cmake_minimum_required(VERSION 3.1.0)

# Adhere the version number to http://semver.org/
project(cpp-ipfs-api VERSION 0.1.0 LANGUAGES CXX)

# Compile in C++11 mode
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Add compiler warnings
if(CMAKE_CXX_COMPILER_ID STREQUAL "AppleClang" OR CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wpedantic -Wextra -Werror")
endif()

# Generate compile_commands.json, to be used by YouCompleteMe.
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# Find curl
find_package(CURL REQUIRED)

# Find "JSON for Modern C++" (nlohmann/json.hpp)
find_package(nlohmann_json REQUIRED)

# Targets

set(IPFS_API_LIBNAME ipfs-api)
file(GLOB_RECURSE HEADERS "include/ipfs/*.h")

# To build and install a shared library: "cmake -DBUILD_SHARED_LIBS:BOOL=ON ..."
add_library(${IPFS_API_LIBNAME}
  ${HEADERS}
  src/client.cc
  src/http/transport-curl.cc
)

target_include_directories(${IPFS_API_LIBNAME} PRIVATE ${CURL_INCLUDE_DIRS} PUBLIC
  "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>"
  "$<INSTALL_INTERFACE:include>")

target_link_libraries(${IPFS_API_LIBNAME} INTERFACE nlohmann_json::nlohmann_json PRIVATE ${CURL_LIBRARIES})

set_target_properties(${IPFS_API_LIBNAME} PROPERTIES
  SOVERSION ${PROJECT_VERSION_MAJOR}
  VERSION ${PROJECT_VERSION}
)

install(TARGETS ${IPFS_API_LIBNAME} DESTINATION lib)
install(FILES include/ipfs/client.h DESTINATION include/ipfs)
install(FILES include/ipfs/http/transport.h DESTINATION include/ipfs/http)

# Tests, use "CTEST_OUTPUT_ON_FAILURE=1 make test" to see output from failed tests

# https://cmake.org/cmake/help/v3.0/module/CTest.html
include(CTest)

if(BUILD_TESTING)
  add_subdirectory(test)
endif()
