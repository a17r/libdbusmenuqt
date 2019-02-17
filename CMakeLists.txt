project(dbusmenu-qt)
cmake_minimum_required(VERSION 2.8.11)

# Versions
## Package version
set(dbusmenu_qt_VERSION_MAJOR 0)
set(dbusmenu_qt_VERSION_MINOR 9)
set(dbusmenu_qt_VERSION_PATCH 2)
set(dbusmenu_qt_VERSION ${dbusmenu_qt_VERSION_MAJOR}.${dbusmenu_qt_VERSION_MINOR}.${dbusmenu_qt_VERSION_PATCH})

## Lib version
### Bump this one when a binary-incompatible change is introduced
set(dbusmenu_qt_lib_SOVERSION 2)

### Bump this one when the API is extended in a binary-compatible way
set(dbusmenu_qt_lib_API_VERSION 6)

### Bump this one when changes do not extend the API
set(dbusmenu_qt_lib_PATCH_VERSION 0)

set(dbusmenu_qt_lib_VERSION ${dbusmenu_qt_lib_SOVERSION}.${dbusmenu_qt_lib_API_VERSION}.${dbusmenu_qt_lib_PATCH_VERSION})

set(QT_MIN_VERSION "5.9.0")
set(KF5_MIN_VERSION "5.25.0")

find_package(Qt5 ${QT_MIN_VERSION} CONFIG REQUIRED COMPONENTS Core DBus Gui Widgets)
include_directories(${Qt5Widgets_INCLUDE_DIRS} ${Qt5DBus_INCLUDE_DIRS})
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTOMOC_RELAXED_MODE ON)
set(CMAKE_POSITION_INDEPENDENT_CODE ON)

set(QT_SUFFIX "qt5")

find_package(ECM ${KF5_MIN_VERSION} REQUIRED NO_MODULE)
set(CMAKE_MODULE_PATH ${ECM_MODULE_PATH})

include(CMakePackageConfigHelpers)
include(GNUInstallDirs)
include(ECMQtDeclareLoggingCategory)

include (CheckCXXCompilerFlag)
# Check some compiler flags
check_cxx_compiler_flag(-fvisibility=hidden __DBUSMENU_HAVE_GCC_VISIBILITY)
if (__DBUSMENU_HAVE_GCC_VISIBILITY AND NOT WIN32)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fvisibility=hidden")
endif (__DBUSMENU_HAVE_GCC_VISIBILITY AND NOT WIN32)

check_cxx_compiler_flag(-Woverloaded-virtual __DBUSMENU_HAVE_W_OVERLOADED_VIRTUAL)
if (__DBUSMENU_HAVE_W_OVERLOADED_VIRTUAL)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Woverloaded-virtual")
endif (__DBUSMENU_HAVE_W_OVERLOADED_VIRTUAL)

check_cxx_compiler_flag(-std=c++11 __DBUSMENU_HAVE_CXX11)
if (__DBUSMENU_HAVE_CXX11)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
endif (__DBUSMENU_HAVE_CXX11)

set(INCLUDE_INSTALL_DIR "${CMAKE_INSTALL_INCLUDEDIR}/dbusmenu-${QT_SUFFIX}")

configure_file(dbusmenu-qt.pc.in ${CMAKE_BINARY_DIR}/dbusmenu-${QT_SUFFIX}.pc @ONLY)

install(FILES ${CMAKE_BINARY_DIR}/dbusmenu-${QT_SUFFIX}.pc
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig
    )

add_subdirectory(src)
if(BUILD_TESTING)
    add_subdirectory(tests)
endif()
# add_subdirectory(tools) # FindQJSON is not a thing with Qt5

find_package(Doxygen)

if(Doxygen_FOUND)
    configure_file(Doxyfile.in ${CMAKE_BINARY_DIR}/Doxyfile @ONLY)

    add_custom_target(doc ALL doxygen
        WORKING_DIRECTORY ${CMAKE_BINARY_DIR}
        )

    install(DIRECTORY ${CMAKE_BINARY_DIR}/html/
        DESTINATION ${CMAKE_INSTALL_DOCDIR}
        )
endif()

# Generate dbusmenu-qt-config* files
configure_package_config_file(
    dbusmenu-qt-config.cmake.in
    ${CMAKE_BINARY_DIR}/dbusmenu-${QT_SUFFIX}-config.cmake
    INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/dbusmenu-${QT_SUFFIX}
    PATH_VARS INCLUDE_INSTALL_DIR
    )

write_basic_package_version_file(
    ${CMAKE_BINARY_DIR}/dbusmenu-${QT_SUFFIX}-config-version.cmake
    VERSION ${dbusmenu_qt_VERSION}
    COMPATIBILITY SameMajorVersion
    )

# Install dbusmenu-qt-config* files
install(FILES
    ${CMAKE_BINARY_DIR}/dbusmenu-${QT_SUFFIX}-config.cmake
    ${CMAKE_BINARY_DIR}/dbusmenu-${QT_SUFFIX}-config-version.cmake
    DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake/dbusmenu-${QT_SUFFIX}"
    COMPONENT Devel
    )
