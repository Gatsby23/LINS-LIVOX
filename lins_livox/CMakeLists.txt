cmake_minimum_required(VERSION 3.0.2)
project(lins_livox)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++14 -O3")
SET(CMAKE_BUILD_TYPE Release)

find_package(catkin REQUIRED COMPONENTS
  cloud_msgs
  cv_bridge
  geometry_msgs
  image_transport
  nav_msgs
  pcl_conversions
  pcl_ros
  roscpp
  rospy
  sensor_msgs
  std_msgs
  tf
  livox_ros_driver
  visualization_msgs
)

# 这里比较重要的是使用GTSAM库，怎么用GTSAM来做优化，还是很亮的
find_package(GTSAM REQUIRED QUIET)
find_package(PCL REQUIRED QUIET)
find_package(OpenCV REQUIRED QUIET)
find_package(Eigen3 3.3 REQUIRED)

catkin_package(
   INCLUDE_DIRS include
   LIBRARIES ${PROJECT_NAME}
   CATKIN_DEPENDS cloud_msgs livox_ros_driver
   DEPENDS eigen opencv PCL
)

include_directories(
  include
  ${catkin_INCLUDE_DIRS}
  ${PCL_INCLUDE_DIRS}
  ${OpenCV_INCLUDE_DIRS}
  ${GTSAM_INCLUDE_DIR}
  ${EIGEN3_INCLUDE_DIR}
)

link_directories(
  include
  ${catkin_INCLUDE_DIRS}
  ${OpenCV_LIBRARY_DIRS}
  ${PCL_LIBRARY_DIRS}
  ${GTSAM_LIBRARY_DIRS}
  ${EIGEN3_INCLUDE_DIR}
)

# 这个是发布吧？
# 这里需要注意的是：这里相当于4个进程，不是线程，所以效率更高
add_executable(livox_repub src/livox_repub.cpp)
target_link_libraries(livox_repub ${catkin_LIBRARIES} ${PCL_LIBRARIES} ${OpenCV_LIBS})

# 这里应该是特征抽取
add_executable(imageProjection src/imageProjection.cpp src/cluster.cpp)
add_dependencies(imageProjection ${catkin_EXPORTED_TARGETS} cloud_msgs_gencpp)
target_link_libraries(imageProjection ${catkin_LIBRARIES} ${PCL_LIBRARIES} ${OpenCV_LIBRARIES})

# 这里去掉了，感觉有点奇怪，这里应该是做特征关联？->源码中直接删了
# add_executable(featureAssociation src/featureAssociation.cpp)
# add_dependencies(featureAssociation ${catkin_EXPORTED_TARGETS} cloud_msgs_gencpp)
# target_link_libraries(featureAssociation ${catkin_LIBRARIES} ${PCL_LIBRARIES} ${OpenCV_LIBRARIES})

# 融合里程计
add_executable(fusionOdometry src/fusionOdometry.cpp)
add_dependencies(fusionOdometry ${catkin_EXPORTED_TARGETS} cloud_msgs_gencpp)
target_link_libraries(fusionOdometry ${catkin_LIBRARIES} ${PCL_LIBRARIES} ${OpenCV_LIBRARIES})

# 地图优化
add_executable(mapOptmization src/mapOptmization.cpp)
add_dependencies(mapOptmization ${catkin_EXPORTED_TARGETS} cloud_msgs_gencpp)
target_link_libraries(mapOptmization ${catkin_LIBRARIES} ${PCL_LIBRARIES} ${OpenCV_LIBRARIES} gtsam)
