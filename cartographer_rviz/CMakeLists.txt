# Copyright 2016 The Cartographer Authors
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#      http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

cmake_minimum_required(VERSION 3.5)

project(cartographer_rviz)

# Default to C++14
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 14)
endif()

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  # we dont use add_compile_options with pedantic in message packages
  # because the Python C extensions dont comply with it
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wextra -Wpedantic")
endif()

find_package(rosidl_default_generators REQUIRED)

set(PACKAGE_DEPENDENCIES
  eigen_conversions
  message_runtime
  roscpp
  roslib
  rviz
)
find_package(cartographer_ros)

find_package(cartographer_ros_msgs)

find_package(cartographer REQUIRED)
include("${CARTOGRAPHER_CMAKE_DIR}/functions.cmake")
google_initialize_cartographer_project()

find_package(rviz2 REQUIRED)
find_package(Eigen3 REQUIRED)
find_package(Boost REQUIRED COMPONENTS system iostreams)
find_package(ament_cmake REQUIRED)
find_package(rviz_ogre_vendor REQUIRED)


set(CMAKE_AUTOMOC ON)

find_package(Qt5 REQUIRED Core Widgets)
set(QT_LIBRARIES Qt5::Widgets)
include_directories(${Qt5Widgets_INCLUDE_DIRS})


add_library(${PROJECT_NAME}
  cartographer_rviz/drawable_submap.cc
  cartographer_rviz/ogre_slice.cc
  cartographer_rviz/submaps_display.cc)
target_link_libraries(${PROJECT_NAME} PUBLIC ${QT_LIBRARIES})

add_definitions(-DQT_NO_KEYWORDS)

# Add the binary directory first, so that port.h is included after it has
# been generated.
target_include_directories(${PROJECT_NAME} PUBLIC
    $<BUILD_INTERFACE:${PROJECT_BINARY_DIR}>
    $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}>
    $<INSTALL_INTERFACE:include>
)
ament_target_dependencies(${PROJECT_NAME}
  cartographer_ros_msgs
  Boost
  cartographer
  Eigen3
  )
#
#target_include_directories(${PROJECT_NAME} SYSTEM PUBLIC
#  "${EIGEN3_INCLUDE_DIR}")
#target_link_libraries(${PROJECT_NAME} PUBLIC ${EIGEN3_LIBRARIES})
#
#target_include_directories(${PROJECT_NAME} SYSTEM PUBLIC
#  "${Boost_INCLUDE_DIRS}")
#target_link_libraries(${PROJECT_NAME} PUBLIC ${Boost_LIBRARIES})

set(TARGET_COMPILE_FLAGS "${TARGET_COMPILE_FLAGS} ${GOOG_CXX_FLAGS}")
set_target_properties(${PROJECT_NAME} PROPERTIES
  COMPILE_FLAGS ${TARGET_COMPILE_FLAGS})

target_link_libraries(${PROJECT_NAME} PUBLIC cartographer)

#install(TARGETS ${PROJECT_NAME}
#  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
#  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
#  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
#)

#install(FILES rviz_plugin_description.xml
#  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
#)
#
#install(DIRECTORY ogre_media
#  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
#)

ament_export_dependencies(message_runtime)
ament_package()