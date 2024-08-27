cmake_minimum_required(VERSION 3.6)

project(rknn_cam_demo)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# skip 3rd-party lib dependencies
set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -Wl,--allow-shlib-undefined")

# install target and libraries
set(CMAKE_INSTALL_PREFIX ${CMAKE_SOURCE_DIR}/install/rknn_cam_demo_${CMAKE_SYSTEM_NAME})

set(CMAKE_SKIP_INSTALL_RPATH FALSE)
set(CMAKE_BUILD_WITH_INSTALL_RPATH TRUE)
set(CMAKE_INSTALL_RPATH "${CMAKE_INSTALL_PREFIX}/lib")

if(CMAKE_C_COMPILER MATCHES "aarch64")
  set(LIB_ARCH aarch64)
else()
  set(LIB_ARCH armhf)
endif()

include_directories(${CMAKE_SOURCE_DIR})

# rknn api
set(RKNN_API_PATH ${CMAKE_SOURCE_DIR}/../../runtime/${CMAKE_SYSTEM_NAME}/librknn_api)

if(CMAKE_SYSTEM_NAME STREQUAL "Android")
  set(RKNN_RT_LIB ${RKNN_API_PATH}/${CMAKE_ANDROID_ARCH_ABI}/librknnrt.so)
else()
  set(RKNN_RT_LIB ${RKNN_API_PATH}/${LIB_ARCH}/librknnrt.so)
endif()

include_directories(${RKNN_API_PATH}/include)
include_directories(${CMAKE_SOURCE_DIR}/../3rdparty)

# opencv
# need to use system OpenCV Package for supporting video capture
find_package(OpenCV REQUIRED)

# rga
# comes from https://github.com/airockchip/librga
set(RGA_PATH ${CMAKE_SOURCE_DIR}/../3rdparty/rga/)
if(CMAKE_SYSTEM_NAME STREQUAL "Android")
  set(RGA_LIB ${RGA_PATH}/libs/AndroidNdk/${CMAKE_ANDROID_ARCH_ABI}/librga.so)
else()
  set(RGA_LIB ${RGA_PATH}/libs/${CMAKE_SYSTEM_NAME}/gcc-${LIB_ARCH}/librga.so)
endif()
include_directories(${RGA_PATH}/include)

set(CMAKE_INSTALL_RPATH "lib")

# rknn_cam_demo
include_directories(${CMAKE_SOURCE_DIR}/include)

add_executable(rknn_cam_demo
        src/main.cc
        src/postprocess.cc
)

target_link_libraries(rknn_cam_demo
	${RKNN_RT_LIB}
	${RGA_LIB}
	${OpenCV_LIBS}
)

# install target and libraries
set(CMAKE_INSTALL_PREFIX ${CMAKE_SOURCE_DIR}/install/rknn_cam_demo_${CMAKE_SYSTEM_NAME})
install(TARGETS rknn_cam_demo DESTINATION ./)
install(PROGRAMS ${RKNN_RT_LIB} DESTINATION lib)
install(PROGRAMS ${RGA_LIB} DESTINATION lib)
install(DIRECTORY model DESTINATION ./)
