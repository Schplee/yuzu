set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)
set(CMAKE_AUTOUIC ON)
set(CMAKE_INCLUDE_CURRENT_DIR ON)
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${PROJECT_SOURCE_DIR}/CMakeModules)

add_executable(yuzu
    Info.plist
    about_dialog.cpp
    about_dialog.h
    aboutdialog.ui
    applets/error.cpp
    applets/error.h
    applets/profile_select.cpp
    applets/profile_select.h
    applets/software_keyboard.cpp
    applets/software_keyboard.h
    applets/web_browser.cpp
    applets/web_browser.h
    bootmanager.cpp
    bootmanager.h
    compatdb.ui
    compatibility_list.cpp
    compatibility_list.h
    configuration/config.cpp
    configuration/config.h
    configuration/configure.ui
    configuration/configure_audio.cpp
    configuration/configure_audio.h
    configuration/configure_audio.ui
    configuration/configure_debug.cpp
    configuration/configure_debug.h
    configuration/configure_debug.ui
    configuration/configure_dialog.cpp
    configuration/configure_dialog.h
    configuration/configure_filesystem.cpp
    configuration/configure_filesystem.h
    configuration/configure_filesystem.ui
    configuration/configure_gamelist.cpp
    configuration/configure_gamelist.h
    configuration/configure_gamelist.ui
    configuration/configure_general.cpp
    configuration/configure_general.h
    configuration/configure_general.ui
    configuration/configure_graphics.cpp
    configuration/configure_graphics.h
    configuration/configure_graphics.ui
    configuration/configure_hotkeys.cpp
    configuration/configure_hotkeys.h
    configuration/configure_hotkeys.ui
    configuration/configure_input.cpp
    configuration/configure_input.h
    configuration/configure_input.ui
    configuration/configure_input_player.cpp
    configuration/configure_input_player.h
    configuration/configure_input_player.ui
    configuration/configure_input_simple.cpp
    configuration/configure_input_simple.h
    configuration/configure_input_simple.ui
    configuration/configure_mouse_advanced.cpp
    configuration/configure_mouse_advanced.h
    configuration/configure_mouse_advanced.ui
    configuration/configure_per_general.cpp
    configuration/configure_per_general.h
    configuration/configure_per_general.ui
    configuration/configure_profile_manager.cpp
    configuration/configure_profile_manager.h
    configuration/configure_profile_manager.ui
    configuration/configure_service.cpp
    configuration/configure_service.h
    configuration/configure_service.ui
    configuration/configure_system.cpp
    configuration/configure_system.h
    configuration/configure_system.ui
    configuration/configure_touchscreen_advanced.cpp
    configuration/configure_touchscreen_advanced.h
    configuration/configure_touchscreen_advanced.ui
    configuration/configure_web.cpp
    configuration/configure_web.h
    configuration/configure_web.ui
    debugger/graphics/graphics_breakpoint_observer.cpp
    debugger/graphics/graphics_breakpoint_observer.h
    debugger/graphics/graphics_breakpoints.cpp
    debugger/graphics/graphics_breakpoints.h
    debugger/graphics/graphics_breakpoints_p.h
    debugger/console.cpp
    debugger/console.h
    debugger/profiler.cpp
    debugger/profiler.h
    debugger/wait_tree.cpp
    debugger/wait_tree.h
    discord.h
    game_list.cpp
    game_list.h
    game_list_p.h
    game_list_worker.cpp
    game_list_worker.h
    loading_screen.cpp
    loading_screen.h
    loading_screen.ui
    hotkeys.cpp
    hotkeys.h
    main.cpp
    main.h
    main.ui
    uisettings.cpp
    uisettings.h
    util/limitable_input_dialog.cpp
    util/limitable_input_dialog.h
    util/sequence_dialog/sequence_dialog.cpp
    util/sequence_dialog/sequence_dialog.h
    util/util.cpp
    util/util.h
    compatdb.cpp
    compatdb.h
    yuzu.rc
)

file(GLOB COMPAT_LIST
     ${PROJECT_BINARY_DIR}/dist/compatibility_list/compatibility_list.qrc
     ${PROJECT_BINARY_DIR}/dist/compatibility_list/compatibility_list.json)
file(GLOB_RECURSE ICONS ${PROJECT_SOURCE_DIR}/dist/icons/*)
file(GLOB_RECURSE THEMES ${PROJECT_SOURCE_DIR}/dist/qt_themes/*)


target_sources(yuzu
    PRIVATE
        ${COMPAT_LIST}
        ${ICONS}
        ${THEMES}
)

if (APPLE)
    set(MACOSX_ICON "../../dist/yuzu.icns")
    set_source_files_properties(${MACOSX_ICON} PROPERTIES MACOSX_PACKAGE_LOCATION Resources)
    target_sources(yuzu PRIVATE ${MACOSX_ICON})
    set_target_properties(yuzu PROPERTIES MACOSX_BUNDLE TRUE)
    set_target_properties(yuzu PROPERTIES MACOSX_BUNDLE_INFO_PLIST ${CMAKE_CURRENT_SOURCE_DIR}/Info.plist)
elseif(WIN32)
    # compile as a win32 gui application instead of a console application
    target_link_libraries(yuzu PRIVATE Qt5::WinMain)
    if(MSVC)
        set_target_properties(yuzu PROPERTIES LINK_FLAGS_RELEASE "/SUBSYSTEM:WINDOWS")
    elseif(MINGW)
        set_target_properties(yuzu PROPERTIES LINK_FLAGS_RELEASE "-mwindows")
    endif()
endif()

create_target_directory_groups(yuzu)

target_link_libraries(yuzu PRIVATE common core input_common video_core)
target_link_libraries(yuzu PRIVATE Boost::boost glad Qt5::OpenGL Qt5::Widgets)
target_link_libraries(yuzu PRIVATE ${PLATFORM_LIBRARIES} Threads::Threads)

target_compile_definitions(yuzu PRIVATE
    # Use QStringBuilder for string concatenation to reduce
    # the overall number of temporary strings created.
    -DQT_USE_QSTRINGBUILDER

    # Disable implicit conversions from/to C strings
    -DQT_NO_CAST_FROM_ASCII
    -DQT_NO_CAST_TO_ASCII

    # Disable implicit type narrowing in signal/slot connect() calls.
    -DQT_NO_NARROWING_CONVERSIONS_IN_CONNECT

    # Disable unsafe overloads of QProcess' start() function.
    -DQT_NO_PROCESS_COMBINED_ARGUMENT_START

    # Disable implicit QString->QUrl conversions to enforce use of proper resolving functions.
    -DQT_NO_URL_CAST_FROM_STRING
)

if (YUZU_ENABLE_COMPATIBILITY_REPORTING)
    target_compile_definitions(yuzu PRIVATE -DYUZU_ENABLE_COMPATIBILITY_REPORTING)
endif()

if (USE_DISCORD_PRESENCE)
    target_sources(yuzu PUBLIC
        discord_impl.cpp
        discord_impl.h
    )
    target_link_libraries(yuzu PRIVATE discord-rpc)
    target_compile_definitions(yuzu PRIVATE -DUSE_DISCORD_PRESENCE)
endif()

if (YUZU_USE_QT_WEB_ENGINE)
    target_link_libraries(yuzu PRIVATE Qt5::WebEngineCore Qt5::WebEngineWidgets)
    target_compile_definitions(yuzu PRIVATE -DYUZU_USE_QT_WEB_ENGINE)
endif ()

if (YUZU_ENABLE_BOXCAT)
    target_compile_definitions(yuzu PRIVATE -DYUZU_ENABLE_BOXCAT)
endif ()

if(UNIX AND NOT APPLE)
    install(TARGETS yuzu RUNTIME DESTINATION "${CMAKE_INSTALL_PREFIX}/bin")
endif()

if (MSVC)
    include(CopyYuzuQt5Deps)
    include(CopyYuzuSDLDeps)
    include(CopyYuzuUnicornDeps)
    copy_yuzu_Qt5_deps(yuzu)
    copy_yuzu_SDL_deps(yuzu)
    copy_yuzu_unicorn_deps(yuzu)
endif()
