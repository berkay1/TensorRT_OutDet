cmake_minimum_required(VERSION 3.17)

set(CMAKE_CUDA_ARCHITECTURES 87)
project(outdet LANGUAGES CXX CUDA)

# Set C++ and CUDA standards
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_PREFIX_PATH=/opt/ros/noetic:$CMAKE_PREFIX_PATH)



set(CMAKE_CUDA_HOST_COMPILER ${CMAKE_CXX_COMPILER})
message("CMAKE_CUDA_HOST_COMPILER: ${CMAKE_CUDA_HOST_COMPILER}")

enable_language(CUDA)

set(CMAKE_CUDA_STANDARD 14)
set(CMAKE_CUDA_STANDARD_REQUIRED ON)

find_package(catkin REQUIRED COMPONENTS
  roscpp
  roslib
  rospy
  std_msgs
  genmsg
  sensor_msgs
)

include_directories(
  ${catkin_INCLUDE_DIRS}
  /opt/ros/noetic/include
  ${CUDNN_INCLUDE_DIR}
)


# Catkin package configuration
catkin_package(
  INCLUDE_DIRS ${sensor_msgs_INCLUDE_DIRS}
  CATKIN_DEPENDS sensor_msgs roscpp rospy std_msgs laser_geometry
)

set(CUDA_SEPARABLE_COMPILATION ON)

# Include Torch paths
set(CMAKE_PREFIX_PATH "/usr/local/lib/python3.8/dist-packages/torch/share/cmake/Torch" ${CMAKE_PREFIX_PATH})

# Set cuDNN include and library directories


set(TensorRT_LIBRARIES
    "/usr/lib/aarch64-linux-gnu/libnvcaffe_parser.so"
    "/usr/lib/aarch64-linux-gnu/libnvinfer.so"
    "/usr/lib/aarch64-linux-gnu/libnvinfer_plugin.so"
    "/usr/lib/aarch64-linux-gnu/libnvonnxparser.so"
    "/usr/lib/aarch64-linux-gnu/libnvparsers.so"
)
set(TensorRT_INCLUDE_DIRS "/usr/include/aarch64-linux-gnu")

# Find Torch
find_package(Torch REQUIRED)

set(CUDNN_INCLUDE_DIR "/usr/include/aarch64-linux-gnu")
set(CUDNN_LIBRARY "/usr/lib/aarch64-linux-gnu/libcudnn.so.8")

set(TensorRT_LIBRARIES
    "/usr/lib/aarch64-linux-gnu/libnvcaffe_parser.so"
    "/usr/lib/aarch64-linux-gnu/libnvinfer.so"
    "/usr/lib/aarch64-linux-gnu/libnvinfer_plugin.so"
    "/usr/lib/aarch64-linux-gnu/libnvonnxparser.so"
    "/usr/lib/aarch64-linux-gnu/libnvparsers.so"
)
set(TensorRT_INCLUDE_DIRS "/usr/include/x86_64-linux-gnu")

# Required libraries and dependencies

find_package(CUDA REQUIRED)
find_package(fmt REQUIRED)
find_package(Threads REQUIRED)

# Directories for CUDA libraries
# set(roscpp_DIR /opt/ros/noetic/share/roscpp/cmake)

# Include directories

# CUDA compilation options
add_library(cuda_comp STATIC
  include/outdet/knncuda.h
  include/outdet/knncuda.cu
)
set_target_properties(cuda_comp PROPERTIES CUDA_RUNTIME_LIBRARY Shared)

# ROS executable: point_publisher
add_executable(point_publisher src/point_publisher.cpp)
target_link_libraries(point_publisher
  ${catkin_LIBRARIES}
  ${sensor_msgs_LIBRARIES}
   ${CUDNN_LIBRARY}
)
add_dependencies(point_publisher
  ${point_publisher_EXPORTED_TARGETS}
  ${catkin_EXPORTED_TARGETS}
)

# Main executable: filter_with_outdet
add_executable(filter_with_outdet
  src/filter_with_outdet.cpp
  include/outdet/outdet.cpp
  include/outdet/util.cpp
)

target_include_directories(filter_with_outdet PUBLIC
  include
  /opt/ros/noetic/include
  ${catkin_INCLUDE_DIRS}
  ${CUDA_INCLUDE_DIRS}
  ${CMAKE_CUDA_TOOLKIT_INCLUDE_DIRECTORIES}
  ${TORCH_INCLUDE_DIRS}
  ${TensorRT_INCLUDE_DIRS}
)

target_link_libraries(filter_with_outdet PUBLIC 
  ${catkin_LIBRARIES}
  ${sensor_msgs_LIBRARIES}
  ${TORCH_LIBRARIES}
  ${CUDA_LIBRARIES}
  ${TensorRT_LIBRARIES}
  fmt::fmt
  cudart
  cuda
  cuda_comp
)

add_dependencies(filter_with_outdet
  ${filter_with_outdet_EXPORTED_TARGETS}
  ${catkin_EXPORTED_TARGETS}
)

# Set necessary Torch flags
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${TORCH_CXX_FLAGS}")

# Debug output to verify library paths
message(STATUS "Torch libraries: ${TORCH_LIBRARIES}")
message(STATUS "TorchVision library: ${TORCHVISION_LIBRARY}")
