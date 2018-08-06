find_package(Qt5Core CONFIG REQUIRED)
if (Qt5Core_VERSION VERSION_LESS 5.6)
    message(FATAL_ERROR "Qt >= 5.6 is required")
endif()
find_package(Qt5Concurrent CONFIG REQUIRED)
find_package(Qt5Network CONFIG REQUIRED)
find_package(Qt5Gui CONFIG REQUIRED)

set(CMAKE_INCLUDE_CURRENT_DIR ON)
set(CMAKE_AUTOMOC ON)

qt5_add_resources(resources resources.qrc)

set(tremotesf_sources
    libtremotesf/rpc.cpp
    libtremotesf/serversettings.cpp
    libtremotesf/serverstats.cpp
    libtremotesf/torrent.cpp
    libtremotesf/tracker.cpp
    alltrackersmodel.cpp
    baseproxymodel.cpp
    basetorrentfilesmodel.cpp
    downloaddirectoriesmodel.cpp
    ipcserver.cpp
    localtorrentfilesmodel.cpp
    main.cpp
    peersmodel.cpp
    servers.cpp
    serversmodel.cpp
    settings.cpp
    statusfilterstats.cpp
    torrentfileparser.cpp
    torrentfilesmodel.cpp
    torrentfilesmodelentry.cpp
    torrentfilesproxymodel.cpp
    torrentsmodel.cpp
    torrentsproxymodel.cpp
    trackersmodel.cpp
    trpc.cpp
    utils.cpp
    ${resources}
)

set(tremotesf_properties
    OUTPUT_NAME "${TREMOTESF_NAME}"
    CXX_STANDARD 11
    CXX_STANDARD_REQUIRED ON
)

set(tremotesf_libs
    Qt5::Core
    Qt5::Concurrent
    Qt5::Network
    Qt5::Gui
)

set(tremotesf_includes)

file(TO_CMAKE_PATH "${DATA_PATH}/translations" translations_path)
set(TRANSLATIONS_PATH "${translations_path}" PARENT_SCOPE)

set(tremotesf_defines
    QT_DEPRECATED_WARNINGS
    QT_DISABLE_DEPRECATED_BEFORE=0x050800
    TREMOTESF_VERSION="${TREMOTESF_VERSION}"
    TRANSLATIONS_PATH="${translations_path}"
)

set(tremotesf_cxxflags -Wall -Wextra -pedantic)

if (SAILFISHOS)
    find_package(Qt5Qml CONFIG REQUIRED)
    find_package(Qt5Quick CONFIG REQUIRED)
    find_package(PkgConfig REQUIRED)
    pkg_check_modules(SAILFISHAPP REQUIRED sailfishapp)

    list(APPEND tremotesf_sources
        sailfishos/directorycontentmodel.cpp
        sailfishos/selectionmodel.cpp
    )
    list(APPEND tremotesf_libs
        Qt5::Qml
        Qt5::Quick
        ${SAILFISHAPP_LDFLAGS}
    )
    list(APPEND tremotesf_defines TREMOTESF_SAILFISHOS)
    list(APPEND tremotesf_includes ${SAILFISHAPP_INCLUDE_DIRS})
    list(APPEND tremotesf_cxxflags ${SAILFISHAPP_CFLAGS_OTHER})
else()
    find_package(Qt5Widgets CONFIG REQUIRED)
    find_package(KF5WidgetsAddons CONFIG REQUIRED)

    list(APPEND tremotesf_libs
        Qt5::Widgets
        KF5::WidgetsAddons
    )

    set_property(SOURCE utils.cpp PROPERTY SKIP_AUTOMOC ON)

    list(APPEND tremotesf_sources
        desktop/aboutdialog.cpp
        desktop/addtorrentdialog.cpp
        desktop/basetreeview.cpp
        desktop/commondelegate.cpp
        desktop/fileselectionwidget.cpp
        desktop/mainwindow.cpp
        desktop/mainwindowsidebar.cpp
        desktop/mainwindowstatusbar.cpp
        desktop/remotedirectoryselectionwidget.cpp
        desktop/servereditdialog.cpp
        desktop/serversdialog.cpp
        desktop/serversettingsdialog.cpp
        desktop/serverstatsdialog.cpp
        desktop/settingsdialog.cpp
        desktop/textinputdialog.cpp
        desktop/torrentfilesview.cpp
        desktop/torrentpropertiesdialog.cpp
        desktop/torrentsview.cpp
        desktop/trackersviewwidget.cpp
    )

    file(TO_CMAKE_PATH "${DATA_PATH}/icons" icons_path)
    set(ICONS_PATH "${icons_path}" PARENT_SCOPE)
    list(APPEND tremotesf_defines ICONS_PATH="${icons_path}")

    if (WIN32)
        list(APPEND tremotesf_sources "desktop/tremotesf.rc")
        string(TOLOWER "${CMAKE_BUILD_TYPE}" build_type)
        if (NOT build_type STREQUAL "debug")
            list(APPEND tremotesf_properties WIN32_EXECUTABLE ON)
        endif()
    else()
        find_package(Qt5DBus CONFIG REQUIRED)
        list(APPEND tremotesf_libs Qt5::DBus)
    endif()
endif()

add_executable(tremotesf ${tremotesf_sources})
set_target_properties(tremotesf PROPERTIES ${tremotesf_properties})

target_link_libraries(tremotesf ${tremotesf_libs})
target_include_directories(tremotesf PRIVATE ${tremotesf_includes})
target_compile_definitions(tremotesf PRIVATE ${tremotesf_defines})
target_compile_options(tremotesf PRIVATE ${tremotesf_cxxflags})

install(TARGETS tremotesf DESTINATION "${CMAKE_INSTALL_BINDIR}")