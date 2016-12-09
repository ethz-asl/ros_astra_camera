cmake_minimum_required(VERSION 2.8.3)
project(astra_camera)

find_package(catkin_simple REQUIRED)
catkin_simple()

find_package(catkin REQUIRED camera_info_manager dynamic_reconfigure image_transport  nodelet sensor_msgs roscpp message_generation)

find_package(Boost REQUIRED COMPONENTS system thread)

option(FILTER "OB Filter library" ON)

if( ${FILTER} STREQUAL "ON" )
set(obfilter "On")
else()
set(obfilter "Off")
endif()

include(ExternalProject)

ExternalProject_Add(astra_openni2
  PREFIX astra_openni2
  # SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/astra_openni2/OpenNI2
  GIT_REPOSITORY https://github.com/ethz-asl/OpenNI2.git
  GIT_TAG fix/bit_shifting
  CONFIGURE_COMMAND echo "no need to configure"
  #${CMAKE_CURRENT_SOURCE_DIR}/libantlr/configure --prefix=<INSTALL_DIR>
  BUILD_IN_SOURCE 1
  BUILD_COMMAND make release FILTER=${obfilter}
  INSTALL_DIR openni2
  INSTALL_COMMAND tar -xjf <SOURCE_DIR>/Packaging/Final/OpenNI-MacOSX-2.3.tar.bz2 -C <INSTALL_DIR> --strip 1 && mkdir -p <INSTALL_DIR>/include && ln -fs <INSTALL_DIR>/Include <INSTALL_DIR>/include/openni2
)

cs_add_library(${PROJECT_NAME} src/dependency_tracker.cc)

link_directories(${CMAKE_CURRENT_BINARY_DIR}/openni2/Redist)

include_directories(include
                    ${Boost_INCLUDE_DIRS}
                    ${CMAKE_CURRENT_BINARY_DIR}/openni2/include
                    )


cs_add_library(astra_wrapper
   src/astra_convert.cpp
   src/astra_device.cpp
   src/astra_device_info.cpp
   src/astra_timer_filter.cpp
   src/astra_frame_listener.cpp
   src/astra_device_manager.cpp
   src/astra_exception.cpp
   src/astra_video_mode.cpp
)
target_link_libraries(astra_wrapper ${catkin_LIBRARIES} -lOpenNI2Orbbec
  ${Boost_LIBRARIES} )

cs_add_library(astra_driver_lib
                src/astra_driver.cpp
)
target_link_libraries(astra_driver_lib astra_wrapper ${catkin_LIBRARIES} ${Boost_LIBRARIES} )

cs_add_library(astra_camera_nodelet
   ros/astra_camera_nodelet.cpp
)
target_link_libraries(astra_camera_nodelet astra_driver_lib ${catkin_LIBRARIES} ${Boost_LIBRARIES} )

cs_add_executable(astra_camera_node
   ros/astra_camera_node.cpp
)
target_link_libraries(astra_camera_node astra_driver_lib ${catkin_LIBRARIES} ${Boost_LIBRARIES} )

cs_add_executable(astra_list_devices
   src/list_devices.cpp
)
target_link_libraries(astra_list_devices astra_wrapper)

cs_add_executable(astra_test_wrapper test/test_wrapper.cpp )
target_link_libraries(astra_test_wrapper astra_wrapper ${Boost_LIBRARIES})
if (UNIX AND NOT APPLE)
  add_executable(astra_usb_reset src/usb_reset.c)
  set(ADDITIONAL_EXECUTABLES "astra_usb_reset")
endif()

cs_install()
cs_export(INCLUDE_DIRS ${CATKIN_DEVEL_PREFIX}/include
          # ${CMAKE_CURRENT_BINARY_DIR}/openni2/Redist/libOpenNI2Orbbec.dylib ${CATKIN_DEVEL_PREFIX}/lib
          # ${CMAKE_CURRENT_BINARY_DIR}/openni2/Redist/OpenNI2 ${CATKIN_DEVEL_PREFIX}/lib
          # astra_nodelets.xml ${CATKIN_PACKAGE_SHARE_DESTINATION}
          CFG_EXTRAS orbbec_astra.cmake)

# install(FILES ${CMAKE_CURRENT_BINARY_DIR}/openni2/Redist/libOpenNI2Orbbec.dylib
#   DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
# )
#
# install(DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/openni2/Redist/OpenNI2
#   DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
# )
#
# install(DIRECTORY include/${PROJECT_NAME}/
#   DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
# )
#
# # add xml file
# install(FILES astra_nodelets.xml
#         DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
# )
