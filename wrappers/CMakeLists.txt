#  minimum required cmake version: 3.1.0
cmake_minimum_required(VERSION 3.1.0)

project(RealuvcWrappers)

# Save the command line compile commands in the build output
set(CMAKE_EXPORT_COMPILE_COMMANDS 1)
# View the makefile commands during build
set(CMAKE_VERBOSE_MAKEFILE on)

include(CheckCXXCompilerFlag)
CHECK_CXX_COMPILER_FLAG("-std=c++11" COMPILER_SUPPORTS_CXX11)
CHECK_CXX_COMPILER_FLAG("-std=c++0x" COMPILER_SUPPORTS_CXX0X)
if(COMPILER_SUPPORTS_CXX11)
  message(WARN "****************************** COMPILER_SUPPORTS_CXX11 ${CMAKE_C_FLAGS} -pthread")
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -pthread")
  message(WARN "****************************** COMPILER_SUPPORTS_CXX11 CMAKE_CXX_FLAGS -std=c++11")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
elseif(COMPILER_SUPPORTS_CXX0X)
   message(WARN "****************************** COMPILER_SUPPORTS_CXX0X ${CMAKE_CXX_FLAGS} -std=c++0x")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x")
endif()

set(DEPENDENCIES librealuvc)

set(BUILD_PYTHON_BINDINGS ON CACHE BOOL "")

if(BUILD_PYTHON_BINDINGS)
  add_subdirectory(python)
endif()
