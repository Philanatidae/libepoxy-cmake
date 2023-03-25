include(CPM)

macro(ff_AddRepoDepNoBuild dep_name dep_version dep_type dep_repo dep_tag)
    CPMAddPackage(
        NAME ${dep_name}
        VERSION ${dep_version}
        ${dep_type}_REPOSITORY ${dep_repo}
        ${dep_type}_TAG ${dep_tag}
        DOWNLOAD_ONLY YES
        SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/${dep_name}
    )
    if(NOT ${dep_name}_ADDED)
        message(FATAL_ERROR "Could not add ${dep_name}.")
    endif()
endmacro()

#########################################
# GLFW
#########################################
ff_AddRepoDepNoBuild(glfw 3.3.8 GIT "https://github.com/glfw/glfw" 7482de6)

set(GLFW_BUILD_EXAMPLES OFF CACHE BOOL "" FORCE)
set(GLFW_BUILD_TESTS OFF CACHE BOOL "" FORCE)
set(GLFW_BUILD_DOCS OFF CACHE BOOL "" FORCE)
if(WIN32)
    set(BUILD_SHARED_LIBS OFF CACHE BOOL "" FORCE)
endif()
add_subdirectory(glfw)
