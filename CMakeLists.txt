cmake_minimum_required(VERSION 3.19)
project(demo LANGUAGES CXX)

option(CMAKE_SUBMODULES "Automatically initialize and update each submodule in the repository." ON)
option(CMAKE_VCXPROJ_FILTERS "Automatically initialize and create vcxproj.filters files in the repository." ON)

find_package(Git QUIET)
if(GIT_FOUND AND EXISTS "${PROJECT_SOURCE_DIR}/.git")
  if(CMAKE_SUBMODULES)
    message(STATUS "Running git submodule update --init --recursive, please wait.")
    execute_process(COMMAND ${GIT_EXECUTABLE} submodule update --init --recursive WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR} RESULT_VARIABLE RESULT)
    if(NOT RESULT EQUAL "0")
      message(FATAL_ERROR "git submodule update --init --recursive failed with ${GIT_SUBMOD_RESULT}, please checkout submodules.")
    endif()
  endif()
endif()

function(target_create_vcxproj_filters target_name)
  if(CMAKE_VCXPROJ_FILTERS)
    message(STATUS "Creating vcxproj.filters files, please wait.")
    get_target_property(target_sources  "${target_name}" SOURCES)
    foreach(file_name IN LISTS target_sources)
      get_filename_component(dir_name "${file_name}" PATH)
      string(REPLACE "/" "\\" group_name "${dir_name}")
      source_group("${group_name}" FILES "${file_name}")
    endforeach()
  endif()
endfunction()

set(CMAKE_DISABLE_IN_SOURCE_BUILD ON)
if ("${CMAKE_SOURCE_DIR}" STREQUAL "${CMAKE_BINARY_DIR}")
  message(FATAL_ERROR "
  In-source builds are not allowed. Please create a build/ subdirectory and use `cmake ..` inside.
  NOTE: It is **IMPORTANT** that you delete `CMakeCache.txt and CMakeFiles/*` in order to proceed.")
endif()

add_subdirectory(src)
