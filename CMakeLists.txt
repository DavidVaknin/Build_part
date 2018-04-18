cmake_minimum_required(VERSION 2.8)
message( STATUS "CMAKE_CXX_COMPILER: " ${CMAKE_CXX_COMPILER} )

project(myProject)

enable_testing()

add_subdirectory(main)
add_subdirectory(test)
add_subdirectory(libfoo)
add_subdirectory(libbar)



########################################################################
# Toolchain Features
########################################################################

set(CMAKE_SYSTEM_NAME Linux)
#set(CMAKE_SYSTEM_PROCESSOR arm) -supportes below

set(CMAKE_SYSROOT /home/devel/rasp-pi-rootfs)
set(CMAKE_STAGING_PREFIX /home/devel/stage)

set(tools /home/devel/gcc-4.7-linaro-rpi-gnueabihf)
set(CMAKE_C_COMPILER ${tools}/bin/arm-linux-gnueabihf-gcc)
set(CMAKE_CXX_COMPILER ${tools}/bin/arm-linux-gnueabihf-g++)

set(CMAKE_FIND_ROOT_PATH_MODE_PROGRAM NEVER)
set(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)
set(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)
set(CMAKE_FIND_ROOT_PATH_MODE_PACKAGE ONLY)

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

message( STATUS "CMAKE_BUILD_TYPE: " ${CMAKE_BUILD_TYPE} )

  if(${CMAKE_SYSTEM_PROCESSOR} MATCHES "arm")
    set(CMAKE_C_FLAGS  "${CMAKE_C_FLAGS} -mfpu=neon -march=native -DIS_ARM -DHAVE_NEON")
    message(STATUS "have ARM")
    set(HAVE_NEON "True")
  else(${CMAKE_SYSTEM_PROCESSOR} MATCHES "arm")
    set(HAVE_NEON "False")
  endif(${CMAKE_SYSTEM_PROCESSOR} MATCHES "arm")
  set(CMAKE_REQUIRED_FLAGS ${CMAKE_C_FLAGS})

  if(NOT WIN32)
      ADD_CXX_COMPILER_FLAG_IF_AVAILABLE(-fvisibility=hidden HAVE_VISIBILITY_HIDDEN)
  endif(NOT WIN32)
 if (ENABLE_ASAN)
   set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fsanitize=address")
   set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fsanitize=address")
 endif (ENABLE_ASAN)
endif(CMAKE_C_COMPILER_ID MATCHES "GNU" OR CMAKE_C_COMPILER_ID MATCHES "Clang")

message(STATUS "CMAKE_C_FLAGS is ${CMAKE_C_FLAGS}")
message(STATUS "CMAKE_CXX_FLAGS is ${CMAKE_CXX_FLAGS}")
