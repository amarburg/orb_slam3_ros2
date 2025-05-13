cmake_minimum_required(VERSION 3.8)
project(orb_slam3_ros2)

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

# find dependencies
find_package(ament_cmake REQUIRED)

set(AMENT_DEPENDENCIES
    Pangolin
    Sophus
    Eigen3
)

foreach(Dependency IN ITEMS ${AMENT_DEPENDENCIES})
  find_package(${Dependency} REQUIRED)
endforeach()

find_package(OpenCV REQUIRED COMPONENTS highgui)

# Location of ORB_SLAM3 submodule
set(ORB_SLAM3_ROOT_DIR ${CMAKE_CURRENT_SOURCE_DIR}/ORB_SLAM3)

include_directories(
  include
  ${ORB_SLAM3_ROOT_DIR}/include
  ${ORB_SLAM3_ROOT_DIR}/include/CameraModels
  ${ORB_SLAM3_ROOT_DIR}/Thirdparty/Sophus
  ${OpenCV_INCLUDE_DIRS}
)

# Of the original ORB_SLAM3 / third_party dependencies, we only Build
# DBoW2
# Assume Sophus, eigen, and g2o come from system packages (rosdep, etc)

# ====
#
# DBoW2
#
set(DBOW2_ROOT_DIR ${ORB_SLAM3_ROOT_DIR}/Thirdparty/DBoW2 )
set(HDRS_DBOW2
  ${DBOW2_ROOT_DIR}/DBoW2/BowVector.h
  ${DBOW2_ROOT_DIR}/DBoW2/FORB.h 
  ${DBOW2_ROOT_DIR}/DBoW2/FClass.h       
  ${DBOW2_ROOT_DIR}/DBoW2/FeatureVector.h
  ${DBOW2_ROOT_DIR}/DBoW2/ScoringObject.h   
  ${DBOW2_ROOT_DIR}/DBoW2/TemplatedVocabulary.h)
set(SRCS_DBOW2
  ${DBOW2_ROOT_DIR}/DBoW2/BowVector.cpp
  ${DBOW2_ROOT_DIR}/DBoW2/FORB.cpp      
  ${DBOW2_ROOT_DIR}/DBoW2/FeatureVector.cpp
  ${DBOW2_ROOT_DIR}/DBoW2/ScoringObject.cpp)

set(HDRS_DUTILS
  ${DBOW2_ROOT_DIR}/DUtils/Random.h
  ${DBOW2_ROOT_DIR}/DUtils/Timestamp.h)
set(SRCS_DUTILS
  ${DBOW2_ROOT_DIR}/DUtils/Random.cpp
  ${DBOW2_ROOT_DIR}/DUtils/Timestamp.cpp)

add_library( dbow2 SHARED
  ${SRCS_DBOW2}
  ${SRC_DUTILS})

ament_export_targets( export_dbow2 HAS_LIBRARY_TARGET)

install(
  DIRECTORY include/
  DESTINATION include
)

install(
  TARGETS dbow2
  EXPORT export_dbow2
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib
  RUNTIME DESTINATION bin
  INCLUDES DESTINATION include
)

# ====
#
# OSB_SLAM3
#

SET(SRCS_ORBSLAM
  ${ORB_SLAM3_ROOT_DIR}/src/System.cc
  ${ORB_SLAM3_ROOT_DIR}/src/Tracking.cc
  ${ORB_SLAM3_ROOT_DIR}/src/LocalMapping.cc
  ${ORB_SLAM3_ROOT_DIR}/src/LoopClosing.cc
  ${ORB_SLAM3_ROOT_DIR}/src/ORBextractor.cc
  ${ORB_SLAM3_ROOT_DIR}/src/ORBmatcher.cc
  ${ORB_SLAM3_ROOT_DIR}/src/FrameDrawer.cc
  ${ORB_SLAM3_ROOT_DIR}/src/Converter.cc
  ${ORB_SLAM3_ROOT_DIR}/src/MapPoint.cc
  ${ORB_SLAM3_ROOT_DIR}/src/KeyFrame.cc
  ${ORB_SLAM3_ROOT_DIR}/src/Atlas.cc
  ${ORB_SLAM3_ROOT_DIR}/src/Map.cc
  ${ORB_SLAM3_ROOT_DIR}/src/MapDrawer.cc
  ${ORB_SLAM3_ROOT_DIR}/src/Optimizer.cc
  ${ORB_SLAM3_ROOT_DIR}/src/Frame.cc
  ${ORB_SLAM3_ROOT_DIR}/src/KeyFrameDatabase.cc
  ${ORB_SLAM3_ROOT_DIR}/src/Sim3Solver.cc
  ${ORB_SLAM3_ROOT_DIR}/src/Viewer.cc
  ${ORB_SLAM3_ROOT_DIR}/src/ImuTypes.cc
  ${ORB_SLAM3_ROOT_DIR}/src/G2oTypes.cc
  ${ORB_SLAM3_ROOT_DIR}/src/CameraModels/Pinhole.cpp
  ${ORB_SLAM3_ROOT_DIR}/src/CameraModels/KannalaBrandt8.cpp
  ${ORB_SLAM3_ROOT_DIR}/src/OptimizableTypes.cpp
  ${ORB_SLAM3_ROOT_DIR}/src/MLPnPsolver.cpp
  ${ORB_SLAM3_ROOT_DIR}/src/GeometricTools.cc
  ${ORB_SLAM3_ROOT_DIR}/src/TwoViewReconstruction.cc
  ${ORB_SLAM3_ROOT_DIR}/src/Config.cc
  ${ORB_SLAM3_ROOT_DIR}/src/Settings.cc
)

add_library( orbslam3 SHARED
  ${SRCS_ORBSLAM}
)
target_include_directories( orbslam3 
  PRIVATE
  ${ORB_SLAM3_ROOT_DIR}
  ${EIGEN3_INCLUDE_DIRS}
)
target_link_libraries( orbslam3 dbow2)

ament_export_targets( export_orbslam3 HAS_LIBRARY_TARGET)
ament_export_dependencies( ${AMENT_DEPENDENCIES} )

install(
  DIRECTORY ORB_SLAM3/include/
  DESTINATION include
)

install(
  TARGETS orbslam3
  EXPORT export_orbslam3
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib
  RUNTIME DESTINATION bin
  INCLUDES DESTINATION include
)

#
#


if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  # the following line skips the linter which checks for copyrights
  # comment the line when a copyright and license is added to all source files
  set(ament_cmake_copyright_FOUND TRUE)
  # the following line skips cpplint (only works in a git repo)
  # comment the line when this package is in a git repo and when
  # a copyright and license is added to all source files
  set(ament_cmake_cpplint_FOUND TRUE)
  ament_lint_auto_find_test_dependencies()
endif()

ament_package()
