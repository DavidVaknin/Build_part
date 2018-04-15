cmake_minimum_required(VERSION 2.8)

project(myProject)

enable_testing()

add_subdirectory(main)
add_subdirectory(test)
add_subdirectory(libfoo)
add_subdirectory(libbar)

if(${CMAKE_BUILD_TYPE} STREQUAL "Debug")
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -ggdb -O0 -DDEBUG_MODE -DBUILD_TYPE_DEBUG")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -ggdb -O0 -DDEBUG_MODE -DBUILD_TYPE_DEBUG")
  else(${CMAKE_BUILD_TYPE} STREQUAL "Debug")
    if(${CMAKE_BUILD_TYPE} STREQUAL "Release")
      set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -ggdb -DBUILD_TYPE_RELEASE")
      set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -ggdb -DBUILD_TYPE_RELEASE")
    else(${CMAKE_BUILD_TYPE} STREQUAL "Release")
    endif(${CMAKE_BUILD_TYPE} STREQUAL "Release")
  endif(${CMAKE_BUILD_TYPE} STREQUAL "Debug")
