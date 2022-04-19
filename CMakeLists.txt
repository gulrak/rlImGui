cmake_minimum_required(VERSION 3.16)

project(rlImGui LANGUAGES C CXX)

set(RLIMGUI_DEFAULT_BUILD_TYPE "Release")
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
    message(STATUS "Setting build type to '${RLIMGUI_DEFAULT_BUILD_TYPE}' as none was specified.")
    set(CMAKE_BUILD_TYPE "${RLIMGUI_DEFAULT_BUILD_TYPE}" CACHE STRING "Choose the type of build." FORCE)
    # Set the possible values of build type for cmake-gui
    set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "Debug" "Release" "MinSizeRel" "RelWithDebInfo")
endif()

get_directory_property(RLIMGUI_IS_MAIN PARENT_DIRECTORY)
option(BUILD_EXAMPLES "Build the examples." ${RLIMGUI_IS_MAIN})

find_package(Git QUIET)
if(GIT_FOUND AND EXISTS "${PROJECT_SOURCE_DIR}/.git")
	# Update submodules as needed
    option(GIT_SUBMODULE "Check submodules during build" ON)
    if(GIT_SUBMODULE)
        message(STATUS "Submodule update")
        execute_process(COMMAND ${GIT_EXECUTABLE} submodule update --init --recursive
                        WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
                        RESULT_VARIABLE GIT_SUBMOD_RESULT)
        if(NOT GIT_SUBMOD_RESULT EQUAL "0")
            message(FATAL_ERROR "git submodule update --init --recursive failed with ${GIT_SUBMOD_RESULT}, please checkout submodules")
        endif()
    endif()
endif()

if(NOT EXISTS "${PROJECT_SOURCE_DIR}/raylib/CMakeLists.txt" OR NOT EXISTS "${PROJECT_SOURCE_DIR}/imgui/imgui.h")
    message(FATAL_ERROR "The submodules were not downloaded! GIT_SUBMODULE was turned off or failed. Please update submodules and try again.")
endif()

if(NOT DEFINED CMAKE_CXX_STANDARD)
	message(STATUS "CMAKE_CXX_STANDARD not set, defaulting to C++17")
    set(CMAKE_CXX_STANDARD 17)
    set(CMAKE_CXX_STANDARD_REQUIRED ON)
    set(CMAKE_CXX_EXTENSIONS ON)
elseif(CMAKE_CXX_STANDARD LESS 11)
    message(FATAL_ERROR "CMAKE_CXX_STANDARD is less than 11, rlImGui only works with C++11 and above.")
endif()

include_directories(${CMAKE_CURRENT_SOURCE_DIR} imgui raylib/src raylib/src/external/glfw/include)
set(IMGUI_SOURCES
	imgui/imgui.cpp
	imgui/imgui_demo.cpp
	imgui/imgui_draw.cpp
	imgui/imgui_widgets.cpp
	imgui/imgui_tables.cpp
	imgui/imgui.h
	imgui/imgui_internal.h
	imgui/misc/cpp/imgui_stdlib.cpp
	imgui/misc/cpp/imgui_stdlib.h
	)
add_library(imgui ${IMGUI_SOURCES})
target_include_directories(imgui PUBLIC imgui)

add_subdirectory(raylib)

add_library(rlimgui rlImGui.cpp rlImGui.h)
target_link_libraries(rlimgui PUBLIC imgui raylib)
target_include_directories(rlimgui PUBLIC raylib/src/external/glfw/include)

if(BUILD_EXAMPLES)
    add_subdirectory(examples)
endif()
