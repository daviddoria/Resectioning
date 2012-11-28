cmake_minimum_required(VERSION 2.6)

INCLUDE_DIRECTORIES(${CMAKE_CURRENT_SOURCE_DIR} ${CMAKE_CURRENT_BINARY_DIR})

PROJECT(Resectioning)

if(UNIX)
  SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -std=gnu++0x")
endif(UNIX)

# Eigen
set(CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR})
if(NOT DEFINED EIGEN3_INCLUDE_DIR) # If the build is absolutely fresh, this will be the case if the CMake variable EIGEN3_INCLUDE_DIR is not set.
  set(EIGEN3_INCLUDE_DIR $ENV{EIGEN3_INCLUDE_DIR})
endif()

FIND_PACKAGE(Eigen3 REQUIRED) #requires you to copy FindEigen3.cmake into the source directory
include_directories(${EIGEN3_INCLUDE_DIR})

# Need to set EIGEN3_INCLUDE_DIR in this project CMake settings (or do the below:)
#include_directories(/media/portable/src/Eigen)

# VTK
FIND_PACKAGE(VTK REQUIRED)
INCLUDE(${VTK_USE_FILE})

if(${VTK_MAJOR_VERSION} LESS 6)
  message(FATAL_ERROR "Your VTK version (${VTK_MAJOR_VERSION}.${VTK_MINOR_VERSION}) is too old! At least 6.0 is required.")
endif()

# if(NOT VTK_USE_QVTK)
#   message(FATAL_ERROR "You must compile VTK with VTK_USE_QT=ON. To do this, you must also use VTK_USE_GUISUPPORT=ON and BUILD_SHARED_LIBS=ON.")
# endif()

if(NOT vtkGUISupportQt_LOADED)
  message(FATAL_ERROR "You must turn on the vtkGUISupportQt module in your VTK build.") # because the UI is Qt
endif()

if(NOT vtkFiltersFlowPaths_LOADED)
  message(FATAL_ERROR "You must turn on the vtkFiltersFlowPaths module in your VTK build.") # for vtkModifiedBSPTree
endif()

FIND_PACKAGE(Qt4 REQUIRED)
INCLUDE(${QT_USE_FILE})

FIND_PACKAGE(ITK 4 REQUIRED)
INCLUDE(${ITK_USE_FILE})

QT4_WRAP_UI(ResectioningUISrcs ResectioningWidget.ui)
QT4_WRAP_CPP(ResectioningMOCSrcs ResectioningWidget.h)

add_subdirectory(CameraCalibration)

# Submodules
# Only include PTXTools if it is not already available.
get_property(FoundPTXTools GLOBAL PROPERTY PTXToolsIncludeDirs SET)
if(NOT FoundPTXTools)
  add_subdirectory(PTXTools)
endif()

get_property(PTXToolsIncludeDirs GLOBAL PROPERTY PTXToolsIncludeDirs)
set(Resectioning_include_dirs ${Resectioning_include_dirs} ${PTXToolsIncludeDirs})
get_property(PTXToolsLibs GLOBAL PROPERTY PTXToolsLibs)
set(Resectioning_libraries ${Resectioning_libraries} ${PTXToolsLibs})


# Give the compiler all required include directories
include_directories(${Resectioning_include_dirs})


# Create executable
ADD_EXECUTABLE(ResectioningInteractive
ResectioningInteractive.cpp
ResectioningWidget.cpp

Resectioning.cpp

ResectioningHelpers.cpp

Pane.cpp
Pane2D.cpp
Pane3D.cpp

Custom3DStyle.cpp

PointSelectionStyle.cpp
PointSelectionStyle2D.cpp
PointSelectionStyle3D.cpp

${ResectioningUISrcs} ${ResectioningMOCSrcs} ${ResourceSrcs} )
TARGET_LINK_LIBRARIES(ResectioningInteractive libCameraCalibration ${Resectioning_libraries})


# SET(ResectionSmart ON CACHE BOOL "Append one ptx to the right of another")
# if(ResectionSmart)
#   ADD_EXECUTABLE(ResectionSmart ColorPTXFromImage.cpp Resectioning.cpp ResectioningHelpers.cpp)
#   TARGET_LINK_LIBRARIES(ResectionSmart ${PTXLibraries})
#   INSTALL( TARGETS ResectionSmart RUNTIME DESTINATION ${INSTALL_DIR} )
# endif()
