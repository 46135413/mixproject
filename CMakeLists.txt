cmake_minimum_required(VERSION 3.6)

set(APP_NAME MixProject)

project(${APP_NAME})

if(XCODE)
    if(NOT DEFINED CMAKE_XCODE_ATTRIBUTE_IPHONEOS_DEPLOYMENT_TARGET)
        SET (CMAKE_XCODE_ATTRIBUTE_IPHONEOS_DEPLOYMENT_TARGET 8.0)
    endif()
endif()

set(COCOS2DX_ROOT_PATH "${CMAKE_CURRENT_SOURCE_DIR}/../../../cocos2d-x-3.17.2")
set(COCOS2DX_LUAJIT_ROOT "${COCOS2DX_ROOT_PATH}/tools/cocos2d-console/bin/../plugins/plugin_luacompile/bin")

if(NOT DEFINED COCOS2DX_ROOT_PATH OR NOT IS_DIRECTORY "${COCOS2DX_ROOT_PATH}")
    message("The value of COCOS2DX_ROOT_PATH is .",${COCOS2DX_ROOT_PATH})
    message(FATAL_ERROR "The value of COCOS2DX_ROOT_PATH is invalid. Aborting.")
endif()

set(COCOS2D_ROOT_DIR ${COCOS2DX_ROOT_PATH})
set(COCOS2D_EXTERNAL_DIR ${COCOS2DX_ROOT_PATH}/external)
set(COCOS2D_COCOS_DIR ${COCOS2DX_ROOT_PATH}/cocos)
set(COCOS2D_LIBICONV_DIR ${COCOS2DX_ROOT_PATH}/libiconv)
set(COCOS2D_EXTENSIONS_DIR ${COCOS2DX_ROOT_PATH}/extensions)

set(COMMON_DIR "F:/project3/client/common")

set(LIBHN_NET_DIR ${COMMON_DIR}/LibHNNet)
set(LIBHN_UI_DIR ${COMMON_DIR}/LibHNUI)
set(LIBHN_MARKET_DIR ${COMMON_DIR}/LibHNMarket)
set(LIBHN_LOBBY_DIR ${COMMON_DIR}/LibHNLobby)
set(LIBHN_OPEN_DIR ${COMMON_DIR}/LibHNOpen)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-deprecated-declarations")

include_directories(
        ${CMAKE_CURRENT_SOURCE_DIR}/Classes
        ${CMAKE_CURRENT_SOURCE_DIR}/Classes/wechat
        ${CMAKE_CURRENT_SOURCE_DIR}/Classes/android
        ${CMAKE_CURRENT_SOURCE_DIR}/Classes/um/common
        ${COCOS2D_ROOT_DIR}/cocos
        ${COCOS2D_ROOT_DIR}/cocos/editor-support
        ${COCOS2D_ROOT_DIR}/cocos/editor-support/cocostudio
        ${COCOS2D_ROOT_DIR}/cocos/scripting/deprecated
        ${COCOS2D_ROOT_DIR}/external
        ${LIBHN_NET_DIR}
        ${LIBHN_UI_DIR}
        ${LIBHN_MARKET_DIR}
        ${LIBHN_LOBBY_DIR}
        ${LIBHN_OPEN_DIR}
)

file(GLOB_RECURSE SRC_FILES
    ${CMAKE_CURRENT_SOURCE_DIR}/Classes/*.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/Classes/wechat/*.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/Classes/android/*.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/Classes/um/*.cpp
)

# record sources, headers, resources...
set(GAME_SOURCE)
set(GAME_HEADER)

# mark app complie info and libs info
set(all_code_files
        ${GAME_HEADER}
        ${GAME_SOURCE}
        ${SRC_FILES}
        )

#if(NOT DEFINED BUILD_ENGINE_DONE)
    set(CMAKE_MODULE_PATH ${COCOS2DX_ROOT_PATH}/cmake/Modules/)
    include(CocosBuildSet)
    set(BUILD_LUA_LIBS ON)
    add_subdirectory(${COCOS2DX_ROOT_PATH}/cocos ${ENGINE_BINARY_PATH}/cocos/core)
#endif()

if(NOT ANDROID)
    add_executable(${APP_NAME} ${all_code_files})
else()
    add_library(${APP_NAME} SHARED ${all_code_files})
    add_subdirectory(${COCOS2DX_ROOT_PATH}/cocos/platform/android ${ENGINE_BINARY_PATH}/cocos/platform)
    target_link_libraries(${APP_NAME} -Wl,--whole-archive cpp_android_spec -Wl,--no-whole-archive)
    add_subdirectory(${COCOS2DX_ROOT_PATH}/cocos/scripting/lua-bindings/proj.android ${ENGINE_BINARY_PATH}/cocos/lua-android)
    target_link_libraries(${APP_NAME} -Wl,--whole-archive lua_android_spec -Wl,--no-whole-archive)
endif()

# 正确示例，需要指定二进制目录：
add_subdirectory(${COCOS2D_ROOT_DIR}/libiconv ${CMAKE_CURRENT_BINARY_DIR}/libiconv)
add_subdirectory(${COMMON_DIR}/LibHNUI ${CMAKE_CURRENT_BINARY_DIR}/LibHNUI)
add_subdirectory(${COMMON_DIR}/LibHNMarket ${CMAKE_CURRENT_BINARY_DIR}/LibHNMarket)
add_subdirectory(${COMMON_DIR}/LibHNLobby ${CMAKE_CURRENT_BINARY_DIR}/LibHNLobby)
add_subdirectory(${COMMON_DIR}/LibHNNet ${CMAKE_CURRENT_BINARY_DIR}/LibHNNet)

target_link_libraries(${APP_NAME} hn_lobby_static hn_market_static hn_net_static hn_ui_static iconv_static luacocos2d cocos2d)

# mark app resources
setup_cocos_app_config(${APP_NAME})
if(APPLE)
    set_target_properties(${APP_NAME} PROPERTIES RESOURCE "${APP_UI_RES}")

    if(MACOSX)
        set_xcode_property(${APP_NAME} INFOPLIST_FILE "${CMAKE_CURRENT_SOURCE_DIR}/proj.mac/Info.plist")
    elseif(IOS)
        set_xcode_property(${APP_NAME} INFOPLIST_FILE "${CMAKE_CURRENT_SOURCE_DIR}/proj.ios/Info.plist")
        set_xcode_property(${APP_NAME} ASSETCATALOG_COMPILER_APPICON_NAME "AppIcon-${APP_NAME}")
    endif()

    # For code-signing, set the DEVELOPMENT_TEAM:
    #set_xcode_property(${APP_NAME} DEVELOPMENT_TEAM "GRLXXXX2K9")
elseif(WINDOWS)
    # "too large PDB" error often occurs in cpp-tests when using default "/Zi"
    target_compile_options(${APP_NAME} PRIVATE /Z7)
    cocos_copy_target_dll(${APP_NAME})
endif()


if(LINUX OR WINDOWS)
    cocos_get_resource_path(APP_RES_DIR ${APP_NAME})
    cocos_copy_target_res(${APP_NAME} LINK_TO ${APP_RES_DIR} FOLDERS ${GAME_RES_FOLDER})
endif()

if(WINDOWS)
    # precompiled header. Compilation time speedup ~4x.
    target_sources(${APP_NAME} PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/Classes/precheader.cpp")
    set_target_properties(${APP_NAME} PROPERTIES COMPILE_FLAGS "/Yuprecheader.h /FIprecheader.h")
    set_source_files_properties("${CMAKE_CURRENT_SOURCE_DIR}/Classes/precheader.cpp" PROPERTIES COMPILE_FLAGS "/Ycprecheader.h")
endif()