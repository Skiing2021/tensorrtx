cmake_minimum_required(VERSION 2.6)

project(yolov5)

add_definitions(-std=c++11)

option(CUDA_USE_STATIC_CUDA_RUNTIME OFF)
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_BUILD_TYPE Debug)

find_package(CUDA REQUIRED)

if (WIN32)
    include_directories(${PROJECT_SOURCE_DIR}/include-win32)
else()
    include_directories(${PROJECT_SOURCE_DIR}/include)
endif()

# include and link dirs of cuda and tensorrt, you need adapt them if yours are different
# cuda
if (WIN32)
    include_directories(${CUDA_TOOLKIT_INCLUDE})
    link_directories(${CUDA_TOOLKIT_ROOT_DIR}/lib/x64)
else()
    include_directories(/usr/local/cuda/include)
    link_directories(/usr/local/cuda/lib64)
endif()

# tensorrt
if (WIN32)
    find_path(TensorRT_ROOT include/NvInfer.h HINTS $ENV{TensorRT_PATH})
    if (NOT TensorRT_ROOT)
        message(FATAL_ERROR "Could not find TensorRT, please set environment variable: TensorRT_PATH")
    endif()
    include_directories(${TensorRT_ROOT}/include)
    link_directories(${TensorRT_ROOT}/lib)
else()
    include_directories(/usr/include/x86_64-linux-gnu/)
    link_directories(/usr/lib/x86_64-linux-gnu/)
endif()

if (WIN32)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -Wall -Ofast -D_MWAITXINTRIN_H_INCLUDED")
else()
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -Wall -Ofast -Wfatal-errors -D_MWAITXINTRIN_H_INCLUDED")
endif()

cuda_add_library(myplugins SHARED ${PROJECT_SOURCE_DIR}/yololayer.cu)
target_link_libraries(myplugins nvinfer cudart)

if (NOT OpenCV_DIR)
    find_path(OpenCV_DIR OpenCVConfig.cmake HINTS $ENV{OpenCV_DIR}/lib)
endif()
find_package(OpenCV)
include_directories(OpenCV_INCLUDE_DIRS)

add_executable(yolov5 ${PROJECT_SOURCE_DIR}/yolov5.cpp)
target_link_libraries(yolov5 nvinfer)
target_link_libraries(yolov5 cudart)
target_link_libraries(yolov5 myplugins)
target_link_libraries(yolov5 ${OpenCV_LIBS})

add_definitions(-O2 -pthread)

