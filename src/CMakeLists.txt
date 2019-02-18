
check_cxx_compiler_flag(-Wall __DBUSMENU_HAVE_W_ALL)
if (__DBUSMENU_HAVE_W_ALL)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall")
endif (__DBUSMENU_HAVE_W_ALL)

# Check some compiler flags
check_cxx_compiler_flag(-fvisibility=hidden __DBUSMENU_HAVE_GCC_VISIBILITY)
if (__DBUSMENU_HAVE_GCC_VISIBILITY AND NOT WIN32)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fvisibility=hidden")
endif (__DBUSMENU_HAVE_GCC_VISIBILITY AND NOT WIN32)

check_cxx_compiler_flag(-Woverloaded-virtual __DBUSMENU_HAVE_W_OVERLOADED_VIRTUAL)
if (__DBUSMENU_HAVE_W_OVERLOADED_VIRTUAL)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Woverloaded-virtual")
endif (__DBUSMENU_HAVE_W_OVERLOADED_VIRTUAL)

check_cxx_compiler_flag(-Wall __DBUSMENU_HAVE_W_ALL)
if (__DBUSMENU_HAVE_W_ALL)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall")
endif (__DBUSMENU_HAVE_W_ALL)

check_cxx_compiler_flag(-std=c++11 __DBUSMENU_HAVE_CXX11)
if (__DBUSMENU_HAVE_CXX11)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
endif (__DBUSMENU_HAVE_CXX11)

set(dbusmenu_qt_SRCS
    dbusmenu_p.cpp
    dbusmenuexporter.cpp
    dbusmenuexporterdbus_p.cpp
    dbusmenuimporter.cpp
    dbusmenutypes_p.cpp
    dbusmenushortcut_p.cpp
    utils.cpp
    )

ecm_qt_declare_logging_category(dbusmenu_qt_SRCS HEADER debug.h
                                            IDENTIFIER DBUSMENUQT
                                            CATEGORY_NAME org.kde.libdbusmenuqt
                                            DEFAULT_SEVERITY Info)

set_source_files_properties(com.canonical.dbusmenu.xml PROPERTIES
   NO_NAMESPACE true
   INCLUDE "dbusmenutypes_p.h"
   CLASSNAME DBusMenuInterface
)
qt5_add_dbus_interface(dbusmenu_qt_SRCS com.canonical.dbusmenu.xml dbusmenu_interface)

include_directories(
    ${CMAKE_SOURCE_DIR}/src
    ${CMAKE_BINARY_DIR}/src
    )

qt5_add_dbus_adaptor(dbusmenu_qt_SRCS
    ${CMAKE_CURRENT_SOURCE_DIR}/com.canonical.dbusmenu.xml
    ${CMAKE_CURRENT_SOURCE_DIR}/dbusmenuexporterdbus_p.h DBusMenuExporterDBus
    )

configure_file(dbusmenu_version.h.in
    ${CMAKE_CURRENT_BINARY_DIR}/dbusmenu_version.h
    )

add_library(dbusmenu-${QT_SUFFIX} SHARED ${dbusmenu_qt_SRCS})
set_target_properties(dbusmenu-${QT_SUFFIX} PROPERTIES
    VERSION ${dbusmenu_qt_lib_VERSION}
    SOVERSION ${dbusmenu_qt_lib_SOVERSION}
    )


target_link_libraries(dbusmenu-${QT_SUFFIX}
    ${Qt5Gui_LIBRARIES}
    ${Qt5Core_LIBRARIES}
    ${Qt5DBus_LIBRARIES}
    ${Qt5Widgets_LIBRARIES}
    )

# Make sure linking to the target adds dbusmenu-qt install directory
target_include_directories(dbusmenu-${QT_SUFFIX}
    INTERFACE "$<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}/dbusmenu-${QT_SUFFIX}>")

install(TARGETS dbusmenu-${QT_SUFFIX}
    EXPORT dbusmenu-${QT_SUFFIX}-targets
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    RUNTIME DESTINATION bin
    )

install(EXPORT dbusmenu-${QT_SUFFIX}-targets
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/dbusmenu-${QT_SUFFIX})

install(DIRECTORY .
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/dbusmenu-${QT_SUFFIX}
    FILES_MATCHING PATTERN "*.h"
    PATTERN "*_p.h" EXCLUDE
    )

install(FILES ${CMAKE_CURRENT_BINARY_DIR}/dbusmenu_version.h
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/dbusmenu-${QT_SUFFIX}
    )
