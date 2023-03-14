# libepoxy-cmake

libepoxy-cmake provides a CMakeLists.txt for [libepoxy](https://github.com/anholt/libepoxy). This project is intended to be used with `add_subdirectory`. As such, install targets for libepoxy-cmake are disabled; if you need to install libepoxy to the system, clone the original and build/install libepoxy using Meson.

The current version of libepoxy-cmake is `rev2-SNAPSHOT` compiling `libepoxy-1.5.10`. Releases can be found in the `release` branch.

## Motivation

In my projects, I use CMake with [CPM.cmake](https://github.com/cpm-cmake/CPM.cmake) for dependency management; libraries installed into system directories via package managers are never used (personal philosophy, mostly to do with not having to deal with package managers for every system and what version of libraries are installed. Libepoxy uses the Meson build system, which is an absolute pain to integrate without manually building binaries. This becomes a huge mess when cross-compiling (I've tried) to Android and whatnot, and just became easier to write CMakeLists.txt for libepoxy instead.

## Tested Cases
Tested on an "as-needed" basis for my projects, but contributions welcome if you confirm that libepoxy-cmake is working on an untested platform.

| Platform    | Confirmed Working |
|-------------|-------------------|
| macOS       | Yes               |
| Windows     | No                |
| Linux (X11) | No                |
| Android     | No                |

## To Do
 - Warning flags are not quite worked out; building with libepoxy-cmake prints warnings that don't print when building libepoxy with Meson
 - Windows, Android, and Linux need to be tested

## License

libepoxy-cmake is licensed under The Unlicense. See `LICENSE` for more information.
