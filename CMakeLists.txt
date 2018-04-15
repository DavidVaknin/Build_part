cmake_minimum_required(VERSION 2.8)
message( STATUS "CMAKE_CXX_COMPILER: " ${CMAKE_CXX_COMPILER} )

project(myProject)

enable_testing()

add_subdirectory(main)
add_subdirectory(test)
add_subdirectory(libfoo)
add_subdirectory(libbar)


########################################################################
# Compiler specific setup
########################################################################

macro(ADD_CXX_COMPILER_FLAG_IF_AVAILABLE flag have)
    include(CheckCXXCompilerFlag)
    check_cxx_compiler_flag(${flag} ${have})
    if(${have})
        add_definitions(${flag})        
    endif(${have})
endmacro(ADD_CXX_COMPILER_FLAG_IF_AVAILABLE)

if(CMAKE_CXX_COMPILER_ID MATCHES "GNU" OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -march=${GCC_ARCH} -Wall -Wno-comment -Wno-reorder -Wno-unused-but-set-variable -Wno-unused-variable -Wformat -std=c++03")

  find_package(SSE)
  if (HAVE_AVX2)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -mfpmath=sse -mavx2 -DLV_HAVE_AVX2 -DLV_HAVE_AVX -DLV_HAVE_SSE")
  else (HAVE_AVX2)
    if(HAVE_AVX)
      set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -mfpmath=sse -mavx -DLV_HAVE_AVX -DLV_HAVE_SSE")
    elseif(HAVE_SSE)
      set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -mfpmath=sse -msse4.1 -DLV_HAVE_SSE")
    endif(HAVE_AVX)
  endif (HAVE_AVX2)
endif(CMAKE_CXX_COMPILER_ID MATCHES "GNU" OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")

ADD_CXX_COMPILER_FLAG_IF_AVAILABLE("-Werror=incompatible-pointer-types" HAVE_ERROR_INCOMPATIBLE)

if(CMAKE_C_COMPILER_ID MATCHES "GNU" OR CMAKE_C_COMPILER_ID MATCHES "Clang")
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -march=${GCC_ARCH} -Wall -Wno-comment -Wno-write-strings -Wno-format-extra-args -Winline -Wno-unused-result -Wno-format -std=c99 -D_GNU_SOURCE")
if(${CMAKE_BUILD_TYPE} STREQUAL "Debug")
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -ggdb -O0 -DDEBUG_MODE -DBUILD_TYPE_DEBUG")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -ggdb -O0 -DDEBUG_MODE -DBUILD_TYPE_DEBUG")
  else(${CMAKE_BUILD_TYPE} STREQUAL "Debug")
    if(${CMAKE_BUILD_TYPE} STREQUAL "Release")
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -ggdb -DBUILD_TYPE_RELWITHDEBINFO")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -ggdb -DBUILD_TYPE_RELWITHDEBINFO")
    else(${CMAKE_BUILD_TYPE} STREQUAL "Release")
    endif(${CMAKE_BUILD_TYPE} STREQUAL "Release")
  endif(${CMAKE_BUILD_TYPE} STREQUAL "Debug")


  if(${CMAKE_SYSTEM_PROCESSOR} MATCHES "arm")
    set(CMAKE_C_FLAGS  "${CMAKE_C_FLAGS} -mfpu=neon -march=native -DIS_ARM -DHAVE_NEON")
    message(STATUS "have ARM")
    set(HAVE_NEON "True")
  else(${CMAKE_SYSTEM_PROCESSOR} MATCHES "arm")
    set(HAVE_NEON "False")
  endif(${CMAKE_SYSTEM_PROCESSOR} MATCHES "arm")
  set(CMAKE_REQUIRED_FLAGS ${CMAKE_C_FLAGS})

 if(NOT HAVE_SSE AND NOT HAVE_NEON AND NOT DISABLE_SIMD)
  message(FATAL_ERROR "no SIMD instructions found")
 endif(NOT HAVE_SSE AND NOT HAVE_NEON AND NOT DISABLE_SIMD)

  if(NOT WIN32)
      ADD_CXX_COMPILER_FLAG_IF_AVAILABLE(-fvisibility=hidden HAVE_VISIBILITY_HIDDEN)
  endif(NOT WIN32)
 if (ENABLE_ASAN)
   set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fsanitize=address")
   set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fsanitize=address")
 endif (ENABLE_ASAN)
endif(CMAKE_C_COMPILER_ID MATCHES "GNU" OR CMAKE_C_COMPILER_ID MATCHES "Clang")

if(${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
   # The following is needed for weak linking to work under OS X
   set(CMAKE_SHARED_LINKER_FLAGS "-undefined dynamic_lookup")
endif(${CMAKE_SYSTEM_NAME} MATCHES "Darwin")

message(STATUS "CMAKE_C_FLAGS is ${CMAKE_C_FLAGS}")
message(STATUS "CMAKE_CXX_FLAGS is ${CMAKE_CXX_FLAGS}")
