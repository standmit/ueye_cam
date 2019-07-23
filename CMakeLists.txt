cmake_minimum_required(VERSION 2.8.3)

project(ueye_cam)

# C++11 Needed
include(CheckCXXCompilerFlag)

CHECK_CXX_COMPILER_FLAG("-std=c++11" COMPILER_SUPPORTS_CXX11)
CHECK_CXX_COMPILER_FLAG("-std=c++0x" COMPILER_SUPPORTS_CXX0X)

if(COMPILER_SUPPORTS_CXX11)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
elseif(COMPILER_SUPPORTS_CXX0X)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x")
else()
    message(FATAL_ERROR "The compiler ${CMAKE_CXX_COMPILER} has no C++11 support. Please use a different C++ compiler.")
endif()

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -g")

find_package(catkin REQUIRED COMPONENTS
  roscpp
  nodelet
  dynamic_reconfigure
  image_transport
  sensor_msgs
  camera_calibration_parsers
  camera_info_manager
  )
catkin_destinations() # define ${CATKIN_PACKAGE_SHARE_DESTINATION}

#set(UEYE_LIBRARY_PATH /usr/lib)
find_library(UEYE_LIBRARY ueye_api)
if(NOT UEYE_LIBRARY)
  message(FATAL_ERROR "libueye_api was not found in system's library path")
endif()
message(STATUS "libueye_api library: ${UEYE_LIBRARY}")

set(UEYECAM_NODELET_NAME ueye_cam_nodelet)
set(UEYECAM_LIB_NAME ueye_wrapper)

set(UEYECAM_LIB_SOURCES
  src/ueye_cam_driver.cpp
)

set(UEYECAM_LIB_HEADERS
  include/ueye_cam/logging_macros.hpp
  include/ueye_cam/ueye_cam_driver.hpp
)

set(UEYECAM_NODELET_SOURCES
  src/ueye_cam_nodelet.cpp
)

set(UEYECAM_NODELET_HEADERS
  include/ueye_cam/ueye_cam_nodelet.hpp
)

generate_dynamic_reconfigure_options(
  config/UEyeCam.cfg
  )

catkin_package(
  INCLUDE_DIRS include
  LIBRARIES ${UEYECAM_LIB_NAME} ${UEYECAM_NODELET_NAME}
  CATKIN_DEPENDS roscpp nodelet dynamic_reconfigure image_transport sensor_msgs camera_calibration_parsers camera_info_manager
  )

include_directories(
  include
  ${catkin_INCLUDE_DIRS}
  ${UEYE_INCLUDE_DIR}
)

add_library(${UEYECAM_LIB_NAME} ${UEYECAM_LIB_SOURCES} ${UEYECAM_LIB_HEADERS})
target_link_libraries(${UEYECAM_LIB_NAME} ${catkin_LIBRARIES} ${UEYE_LIBRARY})

add_library(${UEYECAM_NODELET_NAME} ${UEYECAM_NODELET_SOURCES} ${UEYECAM_NODELET_HEADERS})
target_link_libraries(${UEYECAM_NODELET_NAME} ${catkin_LIBRARIES} ${UEYECAM_LIB_NAME})
add_dependencies(${UEYECAM_NODELET_NAME} ${PROJECT_NAME}_gencfg)

# Do not package the barebones IDS drivers, since they will mask over the official drivers (when present)
#if(USE_UNOFFICIAL_UEYE_DRIVERS)
#  get_filename_component(UNOFFICIAL_UEYE_DRIVERS_LIB_REALPATH ${UEYE_LIBRARY} REALPATH)
#  install(FILES ${UEYE_LIBRARY} ${UNOFFICIAL_UEYE_DRIVERS_LIB_REALPATH}
#    DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION})
#endif()

# Mark cpp header files for installation
install(DIRECTORY include/${PROJECT_NAME}/
  DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
  FILES_MATCHING PATTERN "*.h"
  PATTERN ".svn" EXCLUDE
)

install(FILES nodelet_plugins.xml
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
  )

install(DIRECTORY launch/
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}/launch
  )
