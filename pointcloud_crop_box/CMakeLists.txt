cmake_minimum_required(VERSION 3.11)
project(pointcloud_crop_box)

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic -Wshadow -Wold-style-cast)
endif()

set(PACKAGE_DEPENDENCIES
    ament_cmake
    generate_parameter_library
    laser_geometry
    pcl_conversions
    pcl_ros
    rclcpp
    rclcpp_components
    sensor_msgs
    tf2
    tf2_geometry_msgs
    tf2_ros
    tf2_sensor_msgs
    visualization_msgs)

foreach(PACKAGE IN ITEMS ${PACKAGE_DEPENDENCIES})
  find_package(${PACKAGE} REQUIRED)
endforeach()

find_package(PCL REQUIRED COMPONENTS filters)

include_directories(include ${PCL_INCLUDE_DIRS})

generate_parameter_library(pointcloud_crop_box_params
                           src/pointcloud_crop_box_params.yaml)

add_library(pointcloud_crop_box_node_component SHARED
            src/pointcloud_crop_box_node.cpp)

# ament_target_dependencies() was removed in ament_cmake 2.8 (ROS 2 Lyrical); link the
# modern per-package ${<pkg>_TARGETS} variables instead. generate_parameter_library is a
# build-tool macro package with no exported targets - its generated
# pointcloud_crop_box_params target below carries what is needed.
target_link_libraries(
  pointcloud_crop_box_node_component
  PUBLIC ${laser_geometry_TARGETS}
         ${pcl_conversions_TARGETS}
         ${pcl_ros_TARGETS}
         ${rclcpp_TARGETS}
         ${rclcpp_components_TARGETS}
         ${sensor_msgs_TARGETS}
         ${tf2_TARGETS}
         ${tf2_geometry_msgs_TARGETS}
         ${tf2_ros_TARGETS}
         ${tf2_sensor_msgs_TARGETS}
         ${visualization_msgs_TARGETS}
         ${PCL_LIBRARIES}
         pointcloud_crop_box_params)
target_include_directories(
  pointcloud_crop_box_node_component
  PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}/include>)

rclcpp_components_register_node(
  pointcloud_crop_box_node_component PLUGIN
  "pointcloud_crop_box::PointCloudCropBoxNode" EXECUTABLE
  pointcloud_crop_box_node)

install(
  TARGETS pointcloud_crop_box_node_component pointcloud_crop_box_params
  EXPORT export_${PROJECT_NAME}
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION bin)

install(DIRECTORY launch config DESTINATION share/${PROJECT_NAME})

ament_export_include_directories(include)

ament_export_dependencies(${PACKAGE_DEPENDENCIES})

ament_package()
