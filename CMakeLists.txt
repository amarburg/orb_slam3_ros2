cmake_minimum_required(VERSION 3.8)
project(orb_slam3_ros2)

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)

  # Squelch the many, many warnings from the ORBSLAM3 build
  # This is ... somewhat redundant with -Wall -Wpedantic
  add_compile_options(
    -Wno-deprecated-copy
    -Wno-deprecated-declarations
    -Wno-reorder
    -Wno-sign-compare
    -Wno-unused-parameter
    -Wno-unused-variable
    -Wno-unused-but-set-variable
    -Wno-vla
   )
endif()

# ORB_SLAM has errors with C++17; downgrade to C++14
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 14)
endif()

find_package(ament_cmake REQUIRED)

# We use system-level Sophus, rather than the version included in ORB_SLAM
set(AMENT_DEPENDENCIES
    Pangolin
    Sophus
    Eigen3
)

foreach(Dependency IN ITEMS ${AMENT_DEPENDENCIES})
  find_package(${Dependency} REQUIRED)
endforeach()

find_package(Boost REQUIRED COMPONENTS serialization)
find_package(OpenCV REQUIRED COMPONENTS highgui)

# Location of ORB_SLAM3 submodule
set(ORB_SLAM3_ROOT_DIR ${PROJECT_SOURCE_DIR}/ORB_SLAM3)

add_subdirectory(${ORB_SLAM3_ROOT_DIR}/Thirdparty/DBoW2/)
add_subdirectory(${ORB_SLAM3_ROOT_DIR}/Thirdparty/g2o/)

add_library( orbslam3 SHARED )
target_sources( orbslam3
  PRIVATE
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
target_compile_definitions( orbslam3 PUBLIC REGISTER_TIMES )
target_include_directories( orbslam3
  PUBLIC
    "$<BUILD_INTERFACE:${ORB_SLAM3_ROOT_DIR}>"
    "$<BUILD_INTERFACE:${ORB_SLAM3_ROOT_DIR}/include>"
    "$<BUILD_INTERFACE:${ORB_SLAM3_ROOT_DIR}/include/CameraModels>"
    "$<INSTALL_INTERFACE:include/>"
    "$<INSTALL_INTERFACE:include/CameraModels>"
    ${EIGEN3_INCLUDE_DIRS}
    ${OpenCV_INCLUDE_DIRS}
)
target_link_libraries( orbslam3
  PUBLIC
    Eigen3::Eigen
    Sophus::Sophus
    ${Pangolin_LIBRARIES}
    DBoW2
    g2o
    Boost::boost
)

ament_export_targets( export_g2o HAS_LIBRARY_TARGET )
ament_export_targets( export_DBoW2 HAS_LIBRARY_TARGET )
ament_export_targets( export_orbslam3 HAS_LIBRARY_TARGET)
ament_export_dependencies( ${AMENT_DEPENDENCIES}  )

install(
  DIRECTORY
    ${ORB_SLAM3_ROOT_DIR}/include/
    ${ORB_SLAM3_ROOT_DIR}/Thirdparty
  DESTINATION include
  FILES_MATCHING PATTERN "*.h*"
)

install(
  TARGETS orbslam3
  EXPORT export_orbslam3
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib
  RUNTIME DESTINATION bin
  INCLUDES DESTINATION include
)

install(
  TARGETS DBoW2
  EXPORT export_DBoW2
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib
  RUNTIME DESTINATION bin
  INCLUDES DESTINATION include
)

install(
  TARGETS g2o
  EXPORT export_g2o
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
