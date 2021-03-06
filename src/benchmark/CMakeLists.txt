cmake_minimum_required(VERSION 2.8.4)
if (POLICY CMP0054)
  cmake_policy(SET CMP0054 NEW)
endif()

include("${CMAKE_CURRENT_LIST_DIR}/../common/common.cmake")

######################################################################
# fp_benchmark target

add_executable(
        fp_benchmark
        ${CMAKE_CURRENT_LIST_DIR}/benchmark.cpp
)
set_target_properties(
        fp_benchmark
        PROPERTIES COMPILE_FLAGS "${COMMON_CXX_FLAGS}"
)

######################################################################
# add external project, google/benchmark

ExternalProject_Add(
  google_benchmark
  URL "https://github.com/google/benchmark/archive/v1.1.0.tar.gz"
  URL_MD5 "66b2a23076cf70739525be0092fc3ae3"
  CMAKE_ARGS -DCMAKE_BUILD_TYPE:STRING=${CMAKE_BUILD_TYPE} -DCMAKE_INSTALL_PREFIX:STRING=<INSTALL_DIR>
)
ExternalProject_Get_Property(google_benchmark source_dir)
include_directories(SYSTEM ${source_dir}/include)

# not sure why this isn't taken care of in google_benchmark/src/CMakeLists.txt
if(${CMAKE_SYSTEM_NAME} MATCHES "Windows")
  target_link_libraries(fp_benchmark Shlwapi)
endif()

find_package(Threads REQUIRED)

######################################################################
# hitch google_benchmark to test

add_dependencies(fp_benchmark google_benchmark)
ExternalProject_Get_Property(google_benchmark binary_dir)

if (${CMAKE_CXX_COMPILER_ID} STREQUAL MSVC)
  target_link_libraries(
    fp_benchmark
    debug ${binary_dir}/src/Debug/${CMAKE_FIND_LIBRARY_PREFIXES}benchmark${CMAKE_FIND_LIBRARY_SUFFIXES}
    optimized ${binary_dir}/src/Release/${CMAKE_FIND_LIBRARY_PREFIXES}benchmark${CMAKE_FIND_LIBRARY_SUFFIXES}
  )
else ()
  target_link_libraries(
    fp_benchmark
    general ${binary_dir}/src/${CMAKE_FIND_LIBRARY_PREFIXES}benchmark.a
    general pthread
  )
endif ()
