cmake_minimum_required(VERSION 3.20)

project(libepoxy-cmake-test)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_CURRENT_SOURCE_DIR}/cmake/")

set(BUILD_SHARED_LIBS OFF CACHE BOOL "" FORCE)
add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/.. ${CMAKE_CURRENT_BINARY_DIR}/libepoxy-cmake)
add_subdirectory(deps)

add_executable(main
    src/main.cpp
)

if(APPLE)
target_link_libraries(main PUBLIC "-framework Foundation")
target_link_libraries(main PUBLIC "-framework OpenGL")
endif()

target_link_libraries(main PUBLIC
    glfw
    epoxy
)
