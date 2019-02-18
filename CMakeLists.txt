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

set(QT_SUFFIX "qt5")

find_package(ECM ${KF5_MIN_VERSION} REQUIRED NO_MODULE)
set(CMAKE_MODULE_PATH ${ECM_MODULE_PATH})

if(NOT CMAKE_VERSION VERSION_LESS "3.10.0")
    # CMake 3.9+ warns about automoc on files without Q_OBJECT, and doesn't know about other macros.
    # 3.10+ lets us provide more macro names that require automoc.
    # Remove this when depending on a Frameworks or ECM version that solves this upstream.
    list(APPEND CMAKE_AUTOMOC_MACRO_NAMES
        "DMRETURN_IF_FAIL" "DMRETURN_VALUE_IF_FAIL")
endif()

set(KDE_SKIP_RPATH_SETTINGS ON)

include(KDEInstallDirs)
include(KDECMakeSettings)
include(KDECompilerSettings NO_POLICY_SCOPE)
include(CMakePackageConfigHelpers)
include(ECMQtDeclareLoggingCategory)
include(FeatureSummary)

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
    add_custom_target(doc ${DOXYGEN_EXECUTABLE}
        WORKING_DIRECTORY ${CMAKE_BINARY_DIR}
        COMMENT "Generating API documentation with Doxygen" VERBATIM)
endif()

# Generate dbusmenu-qt-config* files
configure_package_config_file(
    dbusmenu-qt-config.cmake.in
    ${CMAKE_BINARY_DIR}/dbusmenu-${QT_SUFFIX}-config.cmake
    INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/dbusmenu-${QT_SUFFIX}
    PATH_VARS INCLUDE_INSTALL_DIR KDE_INSTALL_DBUSINTERFACEDIR
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

feature_summary (WHAT ALL INCLUDE_QUIET_PACKAGES FATAL_ON_MISSING_REQUIRED_PACKAGES)
