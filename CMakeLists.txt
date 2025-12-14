cmake_minimum_required(VERSION 3.5)

project(PointCloudViewer VERSION 0.1 LANGUAGES CXX)

set(CMAKE_INCLUDE_CURRENT_DIR ON)

set(CMAKE_AUTOUIC ON)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)

set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

find_package (VTK REQUIRED)
IF (VTK_FOUND)
    MESSAGE(STATUS "VTK found.")
    INCLUDE(${VTK_USE_FILE})
ENDIF()

find_package (PCL 1.7.0 REQUIRED)
link_directories(${PCL_LIBRARY_DIRS})
add_definitions(${PCL_DEFINITIONS})

# Ensure Boost include directories are available to all targets (needed for headers
# such as <boost/date_time/posix_time/posix_time.hpp> used by PCL/other code).
find_package(Boost REQUIRED COMPONENTS date_time)
if(Boost_FOUND)
  message(STATUS "Boost include dirs: ${Boost_INCLUDE_DIRS}")
else()
  # Try common fallbacks: use BOOST_ROOT env, or common installation paths
  if(DEFINED ENV{BOOST_ROOT} AND EXISTS "$ENV{BOOST_ROOT}")
    set(Boost_INCLUDE_DIRS "$ENV{BOOST_ROOT}")
    message(STATUS "Using BOOST_ROOT environment for Boost includes: ${Boost_INCLUDE_DIRS}")
  elseif(EXISTS "D:/Programmer/Boost1.89/boost_1_89_0")
    set(Boost_INCLUDE_DIRS "D:/Programmer/Boost1.89/boost_1_89_0")
    message(STATUS "Using fallback Boost include dir: ${Boost_INCLUDE_DIRS}")
  elseif(EXISTS "D:/Programmer/PCL191/3rdParty/Boost/include/boost-1_68")
    set(Boost_INCLUDE_DIRS "D:/Programmer/PCL191/3rdParty/Boost/include/boost-1_68")
    message(STATUS "Using PCL bundled Boost include dir: ${Boost_INCLUDE_DIRS}")
  else()
    message(WARNING "Boost not found by find_package and no fallback found. Consider setting BOOST_ROOT or -DBoost_INCLUDE_DIR to your Boost include directory.")
  endif()
endif()

find_package(QT NAMES Qt6 Qt5 COMPONENTS Widgets REQUIRED)
find_package(Qt${QT_VERSION_MAJOR} COMPONENTS Widgets REQUIRED)

set(PROJECT_SOURCES
        main.cpp
        pclvisualizer.cpp
        pclvisualizer.ui
        # 测试
        inputdialog.cpp
        inputdialog.ui
                images.qrc
		logo.rc

)

if(${QT_VERSION_MAJOR} GREATER_EQUAL 6)
    qt_add_executable(PointCloudViewer
        MANUAL_FINALIZATION
        ${PROJECT_SOURCES}
    )
# Define target properties for Android with Qt 6 as:
#    set_property(TARGET PointCloudViewer APPEND PROPERTY QT_ANDROID_PACKAGE_SOURCE_DIR
#                 ${CMAKE_CURRENT_SOURCE_DIR}/android)
# For more information, see https://doc.qt.io/qt-6/qt-add-executable.html#target-creation
else()
    if(ANDROID)
        add_library(PointCloudViewer SHARED
            ${PROJECT_SOURCES}
        )
# Define properties for Android with Qt 5 after find_package() calls as:
#    set(ANDROID_PACKAGE_SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/android")
    else()
        add_executable(PointCloudViewer
           # 是否打开控制台
           # WIN32
            ${PROJECT_SOURCES}
        )
    endif()
endif()

if(TARGET PointCloudViewer)
  target_include_directories(PointCloudViewer PRIVATE ${PCL_INCLUDE_DIRS} ${Boost_INCLUDE_DIRS})
  target_link_libraries(PointCloudViewer PRIVATE Qt${QT_VERSION_MAJOR}::Widgets ${PCL_LIBRARIES} ${VTK_LIBRARIES})
endif()

set_target_properties(PointCloudViewer PROPERTIES
    MACOSX_BUNDLE_GUI_IDENTIFIER my.example.com
    MACOSX_BUNDLE_BUNDLE_VERSION ${PROJECT_VERSION}
    MACOSX_BUNDLE_SHORT_VERSION_STRING ${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}
)

if(QT_VERSION_MAJOR EQUAL 6)
    qt_finalize_executable(PointCloudViewer)
endif()
