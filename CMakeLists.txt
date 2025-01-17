cmake_minimum_required(VERSION 3.11)

project(RGBD_DVO)

set(CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake_modules/" ${CMAKE_MODULE_PATH})

# Set default build type if not specified otherwise.
# See https://cmake.org/pipermail/cmake/2012-May/050243.html
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
  set(CMAKE_BUILD_TYPE RelWithDebInfo CACHE STRING "Choose the type of build." FORCE)
  message(STATUS "Setting build type to '${CMAKE_BUILD_TYPE}' as none was specified.")
  # Set the possible values of build type for cmake-gui
  set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "Debug" "Release"
    "MinSizeRel" "RelWithDebInfo")
endif()

# Passing arch=native for all build types is currently
# the only viable option to avoid suble bugs with Eigen.
SET(CXX_MARCH native)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

set(CMAKE_CXX_FLAGS_DEBUG  "-O0 -g -DEIGEN_INITIALIZE_MATRICES_BY_NAN")          # cmake default: "-g"
set(CMAKE_CXX_FLAGS_RELWITHDEBINFO "-O3 -g -DEIGEN_INITIALIZE_MATRICES_BY_NAN")  # cmake default: "-O2 -g -DNDEBUG"
set(CMAKE_CXX_FLAGS_RELEASE "-O3 -DNDEBUG")                                      # cmake default: "-O3 -DNDEBUG"
SET(CMAKE_CXX_FLAGS " -ftemplate-backtrace-limit=0 -Wall ${EXTRA_WARNING_FLAGS} -march=${CXX_MARCH} ${CMAKE_CXX_FLAGS}")

include_directories(include)

find_package(Eigen3 REQUIRED)
include_directories(${EIGEN3_INCLUDE_DIR})
message(STATUS "Found Eigen headers in: ${EIGEN3_INCLUDE_DIR}")

find_package(Sophus REQUIRED)
include_directories(${Sophus_INCLUDE_DIR})
message(STATUS "Found Sophus headers in: ${Sophus_INCLUDE_DIR}")

find_package(TBB REQUIRED)
include_directories(${TBB_INCLUDE_DIR})
message(STATUS "Found TBB headers in: ${TBB_INCLUDE_DIR}")

find_package(Pangolin REQUIRED)
include_directories(${Pangolin_INCLUDE_DIR})
message(STATUS "Found Pangolin ${Pangolin_VERSION} headers in: ${Pangolin_INCLUDE_DIR}")

find_package(OpenCV REQUIRED COMPONENTS core imgproc highgui imgcodecs)
message(STATUS "Found OpenCV ${OpenCV_VERSION} headers in: ${OpenCV_INCLUDE_DIR}")

add_library(utils src/utils.cpp)
target_link_libraries(utils ${Pangolin_LIBRARIES})

add_library(frame src/frame.cpp)
target_link_libraries(frame utils ${OpenCV_LIBS})

add_library(directOdometry src/directOdometry.cpp)
target_link_libraries(directOdometry frame utils ${OpenCV_LIBS})

add_executable(main src/main.cpp)
target_link_libraries(main directOdometry frame utils ${OpenCV_LIBS} ${Pangolin_LIBRARIES} ${TBB_LIBRARIES})

