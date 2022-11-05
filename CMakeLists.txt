cmake_minimum_required(VERSION 3.16)
project(bread LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_COMPILER "/usr/bin/g++-12")
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_BUILD_TYPE debug)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++2a -fmodules-ts")

set(PREBUILT_MODULE_PATH ${CMAKE_BINARY_DIR}/modules)

function(add_module name)
    file(MAKE_DIRECTORY ${PREBUILT_MODULE_PATH})
    #message(STATUS "{name} ${name}")
    #message(STATUS "{CMAKE_CXX_COMPILER} ${CMAKE_CXX_COMPILER}")
    #message(STATUS "{CMAKE_CURRENT_SOURCE_DIR} ${CMAKE_CURRENT_SOURCE_DIR}")
    #message(STATUS "{ARGN} ${ARGN}")
    #message(STATUS "{CMAKE_CXX_FLAGS} ${CMAKE_CXX_FLAGS}")
    #message(STATUS "{PREBUILT_MODULE_PATH} ${PREBUILT_MODULE_PATH}")
    #message(STATUS " ")
          if (CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
            # using Clang
            add_custom_target(${name}.pcm
                    COMMAND
                        ${CMAKE_CXX_COMPILER}
                        -std=c++2a
                        -c
                        ${CMAKE_CURRENT_SOURCE_DIR}/${ARGN}
                        -Xclang -emit-module-interface
                        -o ${PREBUILT_MODULE_PATH}/${name}.pcm
                    )
          elseif (CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
            # using GCC
            add_custom_target(${name}.pcm
                    COMMAND
                        ${CMAKE_CXX_COMPILER}
                        -std=c++2a
                        -fmodules-ts
                        -c
                        ${CMAKE_CURRENT_SOURCE_DIR}/${ARGN}
                        -o ${PREBUILT_MODULE_PATH}/${name}.pcm
                    )
            #g++ -std=c++20 -fmodules-ts -xc++-system-header cstudio
          elseif (CMAKE_CXX_COMPILER_ID STREQUAL "MSVC")
            # using Visual Studio C++
            add_custom_target(${name}.obj
                    COMMAND
                        ${CMAKE_CXX_COMPILER} /experimental:module
                        /c
                        ${CMAKE_CURRENT_SOURCE_DIR}/${ARGN}
                    )
          endif()
endfunction()

if (CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
  # using Clang
  add_compile_options(-fprebuilt-module-path=${PREBUILT_MODULE_PATH})

  add_module(helloworld src/helloworld.cpp)
  add_executable(mainExe src/main.cpp src/helloworld.cpp)

  add_dependencies(main helloworld.pcm)

elseif (CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
  # using GCC
  add_compile_options(-fmodules-ts)
  add_module(jam src/jam.cc)
  add_module(butter src/butter.cc)
  add_module(spreadables src/spreadables.cc)
  add_executable(${PROJECT_NAME} src/bread.cc)

  add_custom_target(butter src/butter.cc)
  add_custom_target(jam src/jam.cc)
  add_custom_target(spreadables src/spreadables.cc)

  target_link_options(${PROJECT_NAME} PUBLIC "LINKER:modules/jam.pcm")
  target_link_options(${PROJECT_NAME} PUBLIC "LINKER:modules/butter.pcm")
  target_link_options(${PROJECT_NAME} PUBLIC "LINKER:modules/spreadables.pcm")
  
  add_dependencies(${PROJECT_NAME} jam.pcm butter.pcm spreadables.pcm)

elseif (CMAKE_CXX_COMPILER_ID STREQUAL "MSVC")
  # using Visual Studio C++
  add_compile_options(/experimental:module /c)

  add_module(helloworld helloworld.cpp)
  add_executable(mainExe main.cpp)


  add_dependencies(main helloworld.pcm)

endif()