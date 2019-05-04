cmake_minimum_required(VERSION 2.8)

project(ORB_SLAM2)
set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED 1)


IF(NOT CMAKE_BUILD_TYPE)
  SET(CMAKE_BUILD_TYPE Release)
ENDIF()

MESSAGE("Build type: " ${CMAKE_BUILD_TYPE})

find_package(OpenCV REQUIRED)

find_package(Eigen3 REQUIRED CONFIG)
find_package(Pangolin REQUIRED CONFIG)
find_package(g2o REQUIRED CONFIG)
find_package(dbow2 REQUIRED CONFIG)


set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/lib)

file(GLOB HDRS include/*.h)

add_library(${PROJECT_NAME} 
	${HDRS}
    src/System.cc
    src/Tracking.cc
    src/LocalMapping.cc
    src/LoopClosing.cc
    src/ORBextractor.cc
    src/ORBmatcher.cc
    src/FrameDrawer.cc
    src/Converter.cc
    src/MapPoint.cc
    src/KeyFrame.cc
    src/Map.cc
    src/MapDrawer.cc
    src/Optimizer.cc
    src/PnPsolver.cc
    src/Frame.cc
    src/KeyFrameDatabase.cc
    src/Sim3Solver.cc
    src/Initializer.cc
    src/Viewer.cc
)

target_link_libraries(${PROJECT_NAME} PUBLIC
    ${OpenCV_LIBRARIES}
    Eigen3::Eigen
    pangolin
    ::g2o
    ::dbow2
)
include(GenerateExportHeader)
generate_export_header(${PROJECT_NAME} EXPORT_FILE_NAME ${CMAKE_CURRENT_BINARY_DIR}/gen/include/${PROJECT_NAME}_export.h)
target_include_directories(${PROJECT_NAME} PUBLIC  
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}/gen/include>
    $<INSTALL_INTERFACE:include/${PROJECT_NAME}>
)

if (MSVC)
    target_compile_options(${PROJECT_NAME} PUBLIC /bigobj)
endif()

set_target_properties(${PROJECT_NAME} PROPERTIES DEBUG_POSTFIX _d)
target_compile_definitions(${PROJECT_NAME} PRIVATE -DCOMPILEDWITHC11)

# Build examples
add_executable(rgbd_tum
Examples/RGB-D/rgbd_tum.cc)
target_link_libraries(rgbd_tum ${PROJECT_NAME})

add_executable(stereo_kitti
Examples/Stereo/stereo_kitti.cc)
target_link_libraries(stereo_kitti ${PROJECT_NAME})

add_executable(stereo_euroc
Examples/Stereo/stereo_euroc.cc)
target_link_libraries(stereo_euroc ${PROJECT_NAME})

add_executable(mono_tum
Examples/Monocular/mono_tum.cc)
target_link_libraries(mono_tum ${PROJECT_NAME})

add_executable(mono_kitti
Examples/Monocular/mono_kitti.cc)
target_link_libraries(mono_kitti ${PROJECT_NAME})

add_executable(mono_euroc
Examples/Monocular/mono_euroc.cc)
target_link_libraries(mono_euroc ${PROJECT_NAME})



# install
include(CMakePackageConfigHelpers)
write_basic_package_version_file(
  "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}/${PROJECT_NAME}-config-version.cmake"
  VERSION 0.0.1
  COMPATIBILITY AnyNewerVersion
)

configure_file(${CMAKE_CURRENT_SOURCE_DIR}/cmake/config.cmake.in
  "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}/${PROJECT_NAME}-config.cmake"
  @ONLY
)

install(FILES
    ${CMAKE_CURRENT_BINARY_DIR}/gen/include/${PROJECT_NAME}_export.h
    DESTINATION
        include/${PROJECT_NAME}
)

install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/include/
    DESTINATION
        include/${PROJECT_NAME}
)


install(
  FILES
    "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}/${PROJECT_NAME}-config.cmake"
    "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}/${PROJECT_NAME}-config-version.cmake"
  DESTINATION
    lib/cmake/${PROJECT_NAME}
)

install(TARGETS ${PROJECT_NAME} EXPORT ${PROJECT_NAME}-targets
        ARCHIVE DESTINATION lib
        LIBRARY DESTINATION lib
        RUNTIME DESTINATION bin)
install(EXPORT ${PROJECT_NAME}-targets DESTINATION lib/cmake/${PROJECT_NAME} NAMESPACE ::)

