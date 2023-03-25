# libepoxy-cmake

libepoxy-cmake provides a CMakeLists.txt for [libepoxy](https://github.com/anholt/libepoxy). This project is intended to be used with `add_subdirectory`. As such, install targets for libepoxy-cmake are disabled; if you need to install libepoxy to the system, clone the original and build/install libepoxy using Meson.

The current version of libepoxy-cmake is `rev3` compiling `libepoxy-1.5.10`. Releases can be found in the `release` branch.

## Motivation

In my projects, I use CMake with [CPM.cmake](https://github.com/cpm-cmake/CPM.cmake) for dependency management; libraries installed into system directories via package managers are never used (personal philosophy, mostly to do with not having to deal with package managers for every system and what version of libraries are installed. Libepoxy uses the Meson build system, which is an absolute pain to integrate without manually building binaries. This becomes a huge mess when cross-compiling (I've tried) to Android and whatnot, and just became easier to write CMakeLists.txt for libepoxy instead.

## Configuration

**NOTE:** libepoxy-cmake requires at least CMake 3.20.

Currently, libepoxy-cmake does not enable GLX/EGL/X11 automatically. WGL is enabled/disabled automatically. Set the following before calling `add_subdirectory`:
 - `EPOXY_ENABLE_GLX`, defaults to `OFF`
 - `EPOXY_ENABLE_EGL`, defaults to `OFF`
 - `EPOXY_ENABLE_X11`, defaults to `OFF`

For example, when building for Android:
```
set(EPOXY_ENABLE_GLX OFF)
set(EPOXY_ENABLE_EGL ON)
set(EPOXY_ENABLE_X11 OFF)

add_subdirectory(libepoxy-cmake)
```

### Configuring for Android

You will need to provide a `FindEGL.cmake` in the CMake include path. Libepoxy-cmake does not include any Find\*.cmake modules for you. I use [this one](https://github.com/rpavlik/cmake-modules/blob/main/FindEGL.cmake).

## Tested Cases
Tested on an "as-needed" basis for my projects, but contributions welcome if you confirm that libepoxy-cmake is working on an untested platform.

Note that some platforms may need additional configuration. See the [Configuration](#configuration) section for more information.

| Platform    | Confirmed Working |
|-------------|-------------------|
| macOS       | Yes               |
| Windows     | Yes               |
| Linux (X11) | No                |
| Android     | Yes               |

## To Do
 - Warning flags are not quite worked out; building with libepoxy-cmake prints warnings that don't print when building libepoxy with Meson
 - Linux needs to be tested

## License

libepoxy-cmake is licensed under The Unlicense. See `LICENSE` for more information.
