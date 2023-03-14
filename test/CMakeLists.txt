cmake_minimum_required(VERSION 3.20)

project(libepoxy-cmake-test)

add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/.. ${CMAKE_CURRENT_BINARY_DIR}/libepoxy-cmake)
find_package(glfw3 REQUIRED)

add_executable(main
    src/main.cpp
)
# @todo Remove?
if(APPLE)
target_link_libraries(main PUBLIC "-framework Foundation")
target_link_libraries(main PUBLIC "-framework OpenGL")
endif()

target_link_libraries(main PUBLIC
    glfw
    epoxy
)
