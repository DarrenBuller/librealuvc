#  minimum required cmake version: 3.1.0
cmake_minimum_required(VERSION 3.1.0)

project(RealuvcPythonWrappers)

# Save the command line compile commands in the build output
set(CMAKE_EXPORT_COMPILE_COMMANDS 1)
# View the makefile commands during build
#set(CMAKE_VERBOSE_MAKEFILE on)

include(CheckCXXCompilerFlag)
CHECK_CXX_COMPILER_FLAG("-std=c++11" COMPILER_SUPPORTS_CXX11)
CHECK_CXX_COMPILER_FLAG("-std=c++0x" COMPILER_SUPPORTS_CXX0X)
if(COMPILER_SUPPORTS_CXX11)
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -pthread")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
elseif(COMPILER_SUPPORTS_CXX0X)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x")
endif()

set(DEPENDENCIES librealuvc ${OpenCV_LIBS})

if(NOT PYTHON_EXECUTABLE)
  if(NumPy_FIND_QUIETLY)
    find_package(PythonInterp QUIET)
  else()
    find_package(PythonInterp)
    set(__numpy_out 1)
  endif()
endif()

if (PYTHON_EXECUTABLE)
  # Find out the include path
  execute_process(
    COMMAND "${PYTHON_EXECUTABLE}" -c
      "from __future__ import print_function\ntry: import numpy; print(numpy.get_include(), end='')\nexcept:pass\n"
    OUTPUT_VARIABLE __numpy_path
  )
  # And the version
  execute_process(
    COMMAND "${PYTHON_EXECUTABLE}" -c
      "from __future__ import print_function\ntry: import numpy; print(numpy.__version__, end='')\nexcept:pass\n"
    OUTPUT_VARIABLE __numpy_version
  )
elseif(__numpy_out)
  message(STATUS "Python executable not found.")
endif(PYTHON_EXECUTABLE)

find_path(PYTHON_NUMPY_INCLUDE_DIR numpy/arrayobject.h
  HINTS "${__numpy_path}" "${PYTHON_INCLUDE_PATH}" NO_DEFAULT_PATH)

if(PYTHON_NUMPY_INCLUDE_DIR)
  set(PYTHON_NUMPY_FOUND 1 CACHE INTERNAL "Python numpy found")
endif(PYTHON_NUMPY_INCLUDE_DIR)

include(FindPackageHandleStandardArgs)
find_package_handle_standard_args(
  NumPy
  REQUIRED_VARS PYTHON_NUMPY_INCLUDE_DIR
  VERSION_VAR __numpy_version
)

add_subdirectory(third_party/pybind11)

set(RAW_RS_CPP 
  pybackend.cpp
  pybackend_extras.cpp
  ../../src/backend.cpp
  ../../src/driver_peripheral.cpp
  ../../src/driver_rigel.cpp
  ../../src/linux/backend-hid.cpp
  ../../src/linux/backend-v4l2.cpp
  ../../src/log.cpp
  ../../src/realuvc_driver.cpp
  ../../src/types.cpp
  ../../src/videocapture.cpp
  ../../src/win/win-backend.cpp
  ../../src/win/win-helpers.cpp
  ../../src/win/win-hid.cpp
  ../../src/win/win-usb.cpp
  ../../src/win/win-uvc.cpp
)

set(RAW_RS_HPP
  pybackend_extras.h
  ../../src/backend.h
  ../../src/linux/backend-v4l2.h
  ../../src/linux/backend-hid.h
  ../../src/types.h
  ../../include/librealuvc/realuvc.h
  ../../include/librealuvc/realuvc_driver.h
  ../../include/librealuvc/ru_common.h
  ../../include/librealuvc/ru_exception.h
  ../../include/librealuvc/ru_hid.h
  ../../include/librealuvc/ru_opencv.h
  ../../include/librealuvc/ru_option.h
  ../../include/librealuvc/ru_usb.h
  ../../include/librealuvc/ru_uvc.h
  ../../include/librealuvc/ru_videocapture.h
)

if(${BACKEND} STREQUAL "RS2_USE_LIBUVC_BACKEND")
  list(APPEND RAW_RS_CPP
    ../../src/libuvc/ctrl-gen.cpp
    ../../src/libuvc/ctrl.cpp
    ../../src/libuvc/device.cpp
    ../../src/libuvc/diag.cpp
    ../../src/libuvc/frame.cpp
    ../../src/libuvc/init.cpp
    ../../src/libuvc/libuvc.cpp
    ../../src/libuvc/misc.cpp
    ../../src/libuvc/stream.cpp
  )
endif()

pybind11_add_module(pyrealuvc MODULE ${RAW_RS_CPP} ${RAW_RS_HPP})
if(USE_EXTERNAL_USB)
  #add_dependencies(pyrealuvc libusb)
endif()
target_link_libraries(pyrealuvc
  PRIVATE
    ${LIBUSB1_LIBRARIES}
    ${CMAKE_THREAD_LIBS_INIT}
    ${OpenCV_LIBS}
)
set_target_properties(pyrealuvc PROPERTIES
  VERSION   ${REALUVC_VERSION_STRING}
  SOVERSION ${REALUVC_VERSION_MAJOR}
)
set_target_properties(pyrealuvc PROPERTIES FOLDER Wrappers/python)
include_directories(pyrealuvc
  ../../include
  ${PYTHON_NUMPY_INCLUDE_DIR}
)

install(TARGETS pyrealuvc
  EXPORT realuvcTargets
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
)
