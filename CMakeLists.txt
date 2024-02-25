cmake_minimum_required(VERSION 3.20)
include(CheckIncludeFile)

# # Changing libepoxy version/repository
# By default, libepoxy-cmake will download the latest release of
# libepoxy via FetchContent. You can overwrite the version
# downloaded or point to a fork by setting all of the following
# options, useful if you want an older version or have a fork:
# 
# EPOXY_PROJECT_TAG ~ Set to the tag/commit of libepoxy you are
#       using (e.g. 1.5.10)
# EPOXY_PROJECT_VERSION ~ Set to the version of libepoxy you are
#       using (e.g. 1.5.10)
# EPOXY_CLONE_FROM_REPO ~ If TRUE, libepoxy-cmake will clone libepoxy
#       from EPOXY_REPOSITORY and checkout to EPOXY_PROJECT_TAG
# EPOXY_REPOSITORY ~ URL to git repository of libepoxy. Used when
#       EPOXY_CLONE_FROM_REPO is TRUE.
# EPOXY_LOCAL_DIR ~ Path to local directory of libepoxy. Used when
#       EPOXY_CLONE_FROM_REPO is FALSE.

set(EPOXY_PROJECT_VERSION 1.5.10 CACHE STRING "Version of libepoxy being used")
set(EPOXY_PROJECT_TAG ${EPOXY_PROJECT_VERSION} CACHE STRING "Version of libepoxy being used")
option(EPOXY_CLONE_FROM_REPO "Version of libepoxy being used" TRUE)
set(EPOXY_REPOSITORY "https://github.com/anholt/libepoxy" CACHE STRING "Repository to download libepoxy from")
set(EPOXY_LOCAL_DIR "" CACHE STRING "Local path to libepoxy")

# libepoxy-cmake VERSION 1.5.10-rev5-SNAPSHOT
set(EPOXY_PROJECT_NAME libepoxy)
project(${EPOXY_PROJECT_NAME} VERSION ${EPOXY_PROJECT_VERSION} LANGUAGES C)

if(${EPOXY_CLONE_FROM_REPO})
    include(FetchContent)
    FetchContent_Declare(epoxy_content
        GIT_REPOSITORY ${EPOXY_REPOSITORY}
        GIT_TAG ${EPOXY_PROJECT_TAG}
    )
    FetchContent_GetProperties(epoxy_content)
    if(NOT epoxy_content_POPULATED)
        FetchContent_Populate(epoxy_content)
    endif()

    set(LIBEPOXY_SRC_DIR ${epoxy_content_SOURCE_DIR})
    set(LIBEPOXY_BIN_DIR ${epoxy_content_BINARY_DIR})
else()
    set(LIBEPOXY_SRC_DIR ${EPOXY_LOCAL_DIR})
    set(LIBEPOXY_BIN_DIR ${CMAKE_CURRENT_BINARY_DIR}/libepoxy)
endif()

set(CMAKE_SKIP_INSTALL_RULES YES)

# meson.build
set(PACKAGE_NAME ${EPOXY_PROJECT_NAME})
set(PACKAGE_VERSION ${EPOXY_PROJECT_VERSION})
set(PACKAGE_STRING "${PACKAGE_NAME}-${PACKAGE_VERSION}")
# Most of these PACKAGE_*DIR variables are not correct; but since
# libepoxy-cmake does not support the install target, we don't
# worry about it.
set(PACKAGE_DATADIR ${CMAKE_INSTALL_PREFIX})
cmake_path(APPEND PACKAGE_DATADIR "share")
set(PACKAGE_LIBDIR ${CMAKE_INSTALL_PREFIX})
if(NOT WIN32 OR NOT MSVC)
    cmake_path(APPEND PACKAGE_LIBDIR "lib")
endif()
set(PACKAGE_LOCALEDIR ${CMAKE_INSTALL_PREFIX})
cmake_path(APPEND PACKAGE_LOCALEDIR "share/locale")
set(PACKAGE_LIBEXECDIR ${CMAKE_INSTALL_PREFIX})
cmake_path(APPEND PACKAGE_LOCALEDIR "libexec")
CHECK_INCLUDE_FILE("KHR/khrplatform.h" HAVE_KHRPLATFORM_H CMAKE_REQUIRED_QUIET)

option(BUILD_SHARED_LIBS "Build shared libraries" ON)
# @todo Enable glx, egl, x11 by default on appropriate platforms. See epoxy/meson.build.
option(EPOXY_ENABLE_GLX "Enable GLX support" OFF)
option(EPOXY_ENABLE_EGL "Enable EGL support" OFF)
option(EPOXY_ENABLE_X11 "Enable X11 support" OFF)

# Remaining platform specific API for GL/GLES are enabled
# depending on the platform we're building for.
if(WIN32)
    set(EPOXY_ENABLE_WGL ON)
    set(EPOXY_HAS_ZNOW YES)
elseif(APPLE)
    set(EPOXY_ENABLE_WGL OFF)
    set(EPOXY_HAS_ZNOW NO)
else()
    set(EPOXY_ENABLE_WGL OFF)
    set(EPOXY_HAS_ZNOW YES)
endif()

if(BUILD_SHARED_LIBS)
    add_library(epoxy SHARED)
    if(WIN32)
        target_compile_definitions(epoxy PRIVATE
            DLL_EXPORT
            "EPOXY_PUBLIC=__declspec(dllexport) extern"
        )
        if(NOT MSVC)
            target_compile_options(epoxy PRIVATE    
                -fvisibility=hidden
            )
        endif()
    else()
        target_compile_definitions(epoxy PRIVATE
            "EPOXY_PUBLIC=__attribute__((visibility(\"default\"))) extern"
        )
        target_compile_options(epoxy PRIVATE    
            -fvisibility=hidden
        )
    endif()
else()
    add_library(epoxy STATIC)
endif()

# The inline keyword is only available for C++ in MSVC.
# So we need to use Microsoft specific __inline.
if(WIN32)
    if(MSVC)
        target_compile_definitions(epoxy PRIVATE
            inline=__inline
        )
    endif()
endif()

find_library(DL_LIB dl)
find_package(OpenGL)
find_package(EGL)
if(EGL_FOUND)
    add_library(egl_headers INTERFACE)
    target_include_directories(egl_headers INTERFACE ${EGL_INCLUDE_DIRS})
    target_compile_definitions(egl_headers INTERFACE ${EGL_DEFINITIONS})
endif()

find_package(X11)
if(X11_FOUND)
    add_library(x11_headers INTERFACE)
    target_include_directories(x11_headers INTERFACE ${X11_INCLUDE_DIRS})
    target_compile_definitions(x11_headers INTERFACE ${X11_DEFINITIONS})
endif()

# GLES v2 and v1 may have pkg-config files, courtesy of downstream
# packagers; let's check those first, and fall back to find_library()
# if we fail
find_package(glesv2)
if(NOT glesv2_FOUND)
    find_library(glesv2_LIB libGLESv2)
endif()

find_package(glesv1_cm)
if(NOT glesv1_cm_FOUND)
    find_library(glesv1_cm_LIB libGLESv1_CM)
endif()

# On windows, the DLL has to have all of its functions
# resolved at link time, so we have to link directly against
# opengl32.  But that's the only GL provider, anyway.
if(WIN32)
    find_library(opengl32_lib opengl32 REQUIRED)

    find_library(gdi32_dep gdi32 REQUIRED)
endif()

find_package(Python3 REQUIRED)

if(NOT EXISTS ${LIBEPOXY_BIN_DIR}/include/epoxy)
    file(MAKE_DIRECTORY ${LIBEPOXY_BIN_DIR}/include/epoxy)
endif()
if(NOT EXISTS ${LIBEPOXY_BIN_DIR}/src)
    file(MAKE_DIRECTORY ${LIBEPOXY_BIN_DIR}/src)
endif()

add_custom_command(
    OUTPUT ${LIBEPOXY_BIN_DIR}/include/epoxy/gl_generated.h
    MAIN_DEPENDENCY ${LIBEPOXY_SRC_DIR}/registry/gl.xml
    COMMAND ${Python3_EXECUTABLE}
        ${LIBEPOXY_SRC_DIR}/src/gen_dispatch.py
        --header
        --no-source
        --outputdir=${LIBEPOXY_BIN_DIR}/include/epoxy
        ${LIBEPOXY_SRC_DIR}/registry/gl.xml
    COMMENT "Generating `gl_generated.h`"
)
add_custom_command(
    OUTPUT ${LIBEPOXY_BIN_DIR}/src/gl_generated_dispatch.c
    MAIN_DEPENDENCY ${LIBEPOXY_SRC_DIR}/registry/gl.xml
    COMMAND ${Python3_EXECUTABLE}
        ${LIBEPOXY_SRC_DIR}/src/gen_dispatch.py
        --source
        --no-header
        --outputdir=${LIBEPOXY_BIN_DIR}/src
        ${LIBEPOXY_SRC_DIR}/registry/gl.xml
    COMMENT "Generating `gl_generated_dispatch.c`"
)

# include/epoxy/meson.build
# src/meson.build
set(ENABLE_GLX ${EPOXY_ENABLE_GLX})
set(ENABLE_EGL ${EPOXY_ENABLE_EGL})
set(ENABLE_X11 ${EPOXY_ENABLE_X11})
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/config.h.in ${LIBEPOXY_BIN_DIR}/include/epoxy/config.h @ONLY)

target_include_directories(epoxy
    PUBLIC
        ${LIBEPOXY_SRC_DIR}/include
        ${LIBEPOXY_BIN_DIR}/include
        ${LIBEPOXY_SRC_DIR}/include/epoxy
        ${LIBEPOXY_BIN_DIR}/include/epoxy
        ${LIBEPOXY_SRC_DIR}/src)
target_sources(epoxy
    PUBLIC
        ${LIBEPOXY_BIN_DIR}/include/epoxy/config.h
        ${LIBEPOXY_SRC_DIR}/include/epoxy/common.h
        ${LIBEPOXY_SRC_DIR}/include/epoxy/gl.h
        ${LIBEPOXY_BIN_DIR}/include/epoxy/gl_generated.h
        ${LIBEPOXY_SRC_DIR}/src/dispatch_common.h
        ${LIBEPOXY_SRC_DIR}/src/dispatch_common.c
        ${LIBEPOXY_BIN_DIR}/src/gl_generated_dispatch.c
)
if(EPOXY_ENABLE_EGL)
    add_custom_command(
        OUTPUT ${LIBEPOXY_BIN_DIR}/include/epoxy/egl_generated.h
        MAIN_DEPENDENCY ${LIBEPOXY_SRC_DIR}/registry/egl.xml
        COMMAND ${Python3_EXECUTABLE}
            ${LIBEPOXY_SRC_DIR}/src/gen_dispatch.py
            --header
            --no-source
            --outputdir=${LIBEPOXY_BIN_DIR}/include/epoxy
            ${LIBEPOXY_SRC_DIR}/registry/egl.xml
        COMMENT "Generating `egl_generated.h`"
    )
    add_custom_command(
        OUTPUT ${LIBEPOXY_BIN_DIR}/src/egl_generated_dispatch.c
        MAIN_DEPENDENCY ${LIBEPOXY_SRC_DIR}/registry/egl.xml
        COMMAND ${Python3_EXECUTABLE}
            ${LIBEPOXY_SRC_DIR}/src/gen_dispatch.py
            --source
            --no-header
            --outputdir=${LIBEPOXY_BIN_DIR}/src
            ${LIBEPOXY_SRC_DIR}/registry/egl.xml
        COMMENT "Generating `egl_generated_dispatch.c`"
    )

    target_sources(epoxy PUBLIC
        ${LIBEPOXY_SRC_DIR}/include/epoxy/egl.h
        ${LIBEPOXY_BIN_DIR}/include/epoxy/egl_generated.h
        ${LIBEPOXY_SRC_DIR}/src/dispatch_egl.c
        ${LIBEPOXY_BIN_DIR}/src/egl_generated_dispatch.c
    )
endif()
if(EPOXY_ENABLE_GLX)
    add_custom_command(
        OUTPUT ${LIBEPOXY_BIN_DIR}/include/epoxy/glx_generated.h
        MAIN_DEPENDENCY ${LIBEPOXY_SRC_DIR}/registry/glx.xml
        COMMAND ${Python3_EXECUTABLE}
            ${LIBEPOXY_SRC_DIR}/src/gen_dispatch.py
            --header
            --no-source
            --outputdir=${LIBEPOXY_BIN_DIR}/include/epoxy
            ${LIBEPOXY_SRC_DIR}/registry/glx.xml
        COMMENT "Generating `glx_generated.h`"
    )
    add_custom_command(
        OUTPUT ${LIBEPOXY_BIN_DIR}/src/glx_generated_dispatch.c
        MAIN_DEPENDENCY ${LIBEPOXY_SRC_DIR}/registry/glx.xml
        COMMAND ${Python3_EXECUTABLE}
            ${LIBEPOXY_SRC_DIR}/src/gen_dispatch.py
            --source
            --no-header
            --outputdir=${LIBEPOXY_BIN_DIR}/src
            ${LIBEPOXY_SRC_DIR}/registry/glx.xml
        COMMENT "Generating `glx_generated_dispatch.c`"
    )

target_sources(epoxy PUBLIC
        ${LIBEPOXY_SRC_DIR}/include/epoxy/glx.h
        ${LIBEPOXY_BIN_DIR}/include/epoxy/glx_generated.h
        ${LIBEPOXY_SRC_DIR}/src/dispatch_glx.c
        ${LIBEPOXY_BIN_DIR}/src/glx_generated_dispatch.c
    )
endif()
if(EPOXY_ENABLE_WGL)
    add_custom_command(
        OUTPUT ${LIBEPOXY_BIN_DIR}/include/epoxy/wgl_generated.h
        MAIN_DEPENDENCY ${LIBEPOXY_SRC_DIR}/registry/egl.xml
        COMMAND ${Python3_EXECUTABLE}
            ${LIBEPOXY_SRC_DIR}/src/gen_dispatch.py
            --header
            --no-source
            --outputdir=${LIBEPOXY_BIN_DIR}/include/epoxy
            ${LIBEPOXY_SRC_DIR}/registry/wgl.xml
        COMMENT "Generating `wgl_generated.h`"
    )
    add_custom_command(
        OUTPUT ${LIBEPOXY_BIN_DIR}/src/wgl_generated_dispatch.c
        MAIN_DEPENDENCY ${LIBEPOXY_SRC_DIR}/registry/wgl.xml
        COMMAND ${Python3_EXECUTABLE}
            ${LIBEPOXY_SRC_DIR}/src/gen_dispatch.py
            --source
            --no-header
            --outputdir=${LIBEPOXY_BIN_DIR}/src
            ${LIBEPOXY_SRC_DIR}/registry/wgl.xml
        COMMENT "Generating `wgl_generated_dispatch.c`"
    )

    target_sources(epoxy PUBLIC
        ${LIBEPOXY_SRC_DIR}/include/epoxy/wgl.h
        ${LIBEPOXY_BIN_DIR}/include/epoxy/wgl_generated.h
        ${LIBEPOXY_SRC_DIR}/src/dispatch_wgl.c
        ${LIBEPOXY_BIN_DIR}/src/wgl_generated_dispatch.c
    )
endif()

if(UNIX AND NOT APPLE)
    if (CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
        target_link_options(epoxy PRIVATE
            -Wl,-Bsymbolic-functions
            -Wl,-z,relro
        )
    endif()
endif()

if(WIN32)
    target_link_libraries(epoxy PRIVATE ${opengl32_LIB} ${gdi32_LIB})
endif()
if(EPOXY_ENABLE_X11)
    target_link_libraries(epoxy PRIVATE ${x11_headers})
endif()
if(EPOXY_ENABLE_EGL)
    target_link_libraries(epoxy PRIVATE ${egl_headers})
endif()
