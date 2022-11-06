cmake_minimum_required(VERSION 3.16)
project(bread LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_COMPILER "/usr/bin/g++-12")
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_BUILD_TYPE debug)

function(getstdmodule name)
    if (CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
      # using GCC
      execute_process(COMMAND ${CMAKE_CXX_COMPILER} -std=c++20 -fmodules-ts -xc++-system-header ${name})
    elseif (NOT CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
      # not using GCC
      message(STATUS "Unable to compile standard modules... not using GCC.")
    endif()
endfunction()

if (CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
  # using GCC
  add_compile_options(-std=c++2a -fmodules-ts)

  getstdmodule(iostream)
  getstdmodule(string)

  add_library(butter OBJECT src/butter.cc)
  add_library(jam OBJECT src/jam.cc)
  add_library(spreadables STATIC $<TARGET_OBJECTS:butter> $<TARGET_OBJECTS:jam> src/spreadables.cc)

  add_executable(${PROJECT_NAME} src/bread.cc)
  target_link_libraries(${PROJECT_NAME} spreadables)

elseif (NOT CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
  # not using GCC
  message(STATUS "not using GCC")
endif()