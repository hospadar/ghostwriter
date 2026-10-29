# SPDX-FileCopyrightText: 2022-2023 Megan Conkle <megan.conkle@kdemail.net>
#
# SPDX-License-Identifier: GPL-3.0-or-later

set(CMAKE_AUTOUIC ON)
set(CMAKE_INCLUDE_CURRENT_DIR ON)

set(UI_FILES settings/preferencesdialog.ui)

add_executable(ghostwriter ../resources.qrc ../icons.qrc ${UI_FILES})

target_compile_definitions(ghostwriter PRIVATE APPVERSION="${RELEASE_SERVICE_VERSION}")

target_include_directories(ghostwriter PUBLIC
    "${PROJECT_BINARY_DIR}"
    "${PROJECT_SOURCE_DIR}"
    "${CMAKE_CURRENT_SOURCE_DIR}"
    "${CMAKE_CURRENT_BINARY_DIR}/../3rdparty/cmark-gfm/src"
)

ecm_create_qm_loader(ghostwriter_QM_LOADER ghostwriter_qt)

target_sources(ghostwriter PRIVATE
    editor/actions/inlinemarkuptoggle.cpp
    editor/asynctextwriter.cpp
    editor/markdowndocument.cpp
    editor/markdowneditor.cpp
    editor/markdownhighlighter.cpp
    editor/textblockdata.h
    export/cmarkgfmexporter.cpp
    export/commandlineexporter.cpp
    export/exportdialog.cpp
    export/exporter.cpp
    export/exporterfactory.cpp
    export/exportformat.cpp
    markdown/cmarkgfmapi.cpp
    markdown/markdownast.cpp
    markdown/markdownnode.cpp
    markdown/memoryarena.cpp
    preview/htmlpreview.cpp
    preview/previewproxy.cpp
    preview/sandboxedwebpage.cpp
    settings/appsettings.cpp
    settings/localedialog.cpp
    settings/preferencesdialog.cpp
    settings/previewoptionsdialog.cpp
    settings/simplefontdialog.cpp
    spelling/spellcheckdecorator.cpp
    spelling/spellcheckdialog.cpp
    statistics/abstractstatisticswidget.cpp
    statistics/documentstatistics.cpp
    statistics/documentstatisticswidget.cpp
    statistics/sessionstatistics.cpp
    statistics/sessionstatisticswidget.cpp
    statistics/statisticsindicator.cpp
    theme/chromecolors.cpp
    theme/colorschemepreviewer.cpp
    theme/stylesheetbuilder.cpp
    theme/svgicontheme.cpp
    theme/theme.cpp
    theme/themeeditordialog.cpp
    theme/themerepository.cpp
    theme/themeselectiondialog.cpp
    appactions.cpp
    appmain.cpp
    bookmark.cpp
    documentmanager.cpp
    findreplace.cpp
    folderviewwidget.cpp
    library.cpp
    logging.cpp
    mainwindow.cpp
    messageboxhelper.cpp
    outlinewidget.cpp
    sidebar.cpp
    timelabel.cpp
    ${ghostwriter_QM_LOADER}
)



find_package(Qt${QT_MAJOR_VERSION} ${QT_MIN_VERSION} REQUIRED COMPONENTS
    Concurrent
    Core
    Gui
    Svg
    WebChannel
    WebEngineWidgets
    Widgets
)

find_package(
    KF${KF_MAJOR_VERSION} ${KF_MIN_VERSION}
    REQUIRED
    COMPONENTS
    ConfigWidgets
    CoreAddons
    Sonnet
    WidgetsAddons
    XmlGui
)

# For whatever reason, MSVC has issues statically linking with cmark-gfm.
if (MSVC)
    set(CMARK_LIBS
        libcmark-gfm
        libcmark-gfm-extensions
    )
else()
    set(CMARK_LIBS
        libcmark-gfm_static
        libcmark-gfm-extensions_static
    )
endif()

target_link_libraries(ghostwriter PRIVATE
    Qt::Concurrent
    Qt::Core
    Qt::Gui
    Qt::Svg
    Qt::WebChannel
    Qt::WebEngineWidgets
    Qt::Widgets
    KF${KF_MAJOR_VERSION}::ConfigCore
    KF${KF_MAJOR_VERSION}::ConfigWidgets
    KF${KF_MAJOR_VERSION}::CoreAddons
    KF${KF_MAJOR_VERSION}::SonnetCore
    KF${KF_MAJOR_VERSION}::SonnetUi
    KF${KF_MAJOR_VERSION}::XmlGui
    ${CMARK_LIBS}
)

set_target_properties(ghostwriter PROPERTIES
    WIN32_EXECUTABLE ON
    MACOSX_BUNDLE ON
)

# Add icon files to the application's source files to have CMake bundle them in
# the executable. This must stay in the target's directory: on macOS the .icns
# is placed in the bundle via a source file property, which is directory scoped.
ecm_add_app_icon(ICONS_SOURCES ICONS ${ICON_FILES})
target_sources(ghostwriter PRIVATE ${ICONS_SOURCES})

# See https://cmake.org/cmake/help/v3.15/prop_tgt/MACOSX_BUNDLE_INFO_PLIST.html
if(APPLE)
    string(TIMESTAMP CURRENT_YEAR "%Y")

    # These are substituted by CMake into MacOSBundleInfo.plist.in.
    set_target_properties(ghostwriter PROPERTIES
        MACOSX_BUNDLE_INFO_PLIST ${PROJECT_SOURCE_DIR}/resources/mac/MacOSBundleInfo.plist.in
        MACOSX_BUNDLE_GUI_IDENTIFIER "org.kde.ghostwriter"
        MACOSX_BUNDLE_BUNDLE_NAME "ghostwriter"
        MACOSX_BUNDLE_ICON_FILE "${MACOSX_BUNDLE_ICON_FILE}"
        MACOSX_BUNDLE_INFO_STRING "ghostwriter - A Markdown Editor"
        MACOSX_BUNDLE_LONG_VERSION_STRING "ghostwriter ${RELEASE_SERVICE_VERSION}"
        MACOSX_BUNDLE_SHORT_VERSION_STRING "${RELEASE_SERVICE_VERSION_MAJOR}.${RELEASE_SERVICE_VERSION_MINOR}"
        MACOSX_BUNDLE_BUNDLE_VERSION "${RELEASE_SERVICE_VERSION}"
        MACOSX_BUNDLE_COPYRIGHT "2014-${CURRENT_YEAR} the ghostwriter authors"
    )
endif()

