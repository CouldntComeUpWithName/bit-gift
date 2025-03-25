cmake_minimum_required(VERSION 3.14..3.24)

project( BitGift 
  VERSION 0.0.0
  DESCRIPTION "Utility for sharing and exchanging files between two machines"
  LANGUAGES CXX C )

set(CMAKE_CXX_STANDARD 20) # not the best practice, but who cares
set(CMAKE_CXX_STANDARD_REQUIRED OFF)

include(CTest)

include(cmake/target.cmake)

set(EXECUTABLE_NAME bitg)

add_executable(${EXECUTABLE_NAME})

target_compile_definitions(${EXECUTABLE_NAME} PRIVATE EXECUTABLE_NAME="${EXECUTABLE_NAME}")

set(SOURCES 
    src/main.cpp
)

target_sources(${EXECUTABLE_NAME} PRIVATE ${SOURCES})
target_link_libraries(${EXECUTABLE_NAME} PRIVATE asio)

if(CMAKE_CONFIGURATION_TYPES)
  set_target_properties(${EXECUTABLE_NAME} PROPERTIES RUNTIME_OUTPUT_DIRECTORY "${CMAKE_SOURCE_DIR}/bin/$<CONFIG>")
else()
  set_target_properties(${EXECUTABLE_NAME} PROPERTIES RUNTIME_OUTPUT_DIRECTORY "${CMAKE_SOURCE_DIR}/bin/${CMAKE_BUILD_TYPE}")
endif()

if(MSVC)
  set_property(DIRECTORY ${CMAKE_SOURCE_DIR} PROPERTY VS_STARTUP_PROJECT ${EXECUTABLE_NAME})
  set_property(TARGET ${EXECUTABLE_NAME} PROPERTY VS_DEBUGGER_WORKING_DIRECTORY "${CMAKE_SOURCE_DIR}/bin/$<CONFIG>")
  
  source_group(TREE ${CMAKE_CURRENT_SOURCE_DIR} FILES ${SOURCES})
endif()

add_subdirectory(3rd-party)
add_subdirectory(tools)
add_subdirectory(tests)

if(MSVC) 
  add_compile_options(/MP /W4)
else()
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

if(MSVC OR XCODE)

  set_property(GLOBAL PROPERTY USE_FOLDERS ON)
  set_property(GLOBAL PROPERTY PREDEFINED_TARGETS_FOLDER "CMake")
  
  get_target_list_from(third_party_targets "3rd-party")
  get_target_list_from(test_targets "tests")
  get_target_list_from(tools_targets "tools")
  
  foreach(target IN LISTS third_party_targets)
    if(TARGET ${target})
      
      set(folder_tree "Deps")
      get_target_property(folder ${target} FOLDER)

      if(folder)
        set(folder_tree "${folder_tree}/${folder}")
      endif()
      
      set_target_properties(${target} PROPERTIES 
        FOLDER "${folder_tree}"
        RUNTIME_OUTPUT_DIRECTORY "${CMAKE_SOURCE_DIR}/bin/$<CONFIG>" )

    endif()
  endforeach()

  foreach(target IN LISTS test_targets)
    if(TARGET ${target})
      
      set(folder_tree "Tests")
      get_target_property(folder ${target} FOLDER)

      if(folder)
        set(folder_tree "${folder_tree}/${folder}")
      endif()
      
      set_target_properties(${target} PROPERTIES 
        FOLDER "${folder_tree}"
        RUNTIME_OUTPUT_DIRECTORY "${CMAKE_SOURCE_DIR}/bin/tests/" )

    endif()
  endforeach()

  foreach(target IN LISTS tools_targets)
    if(TARGET ${target})
      
      set(folder_tree "Tools")
      get_target_property(folder ${target} FOLDER)

      if(folder)
        set(folder_tree "${folder_tree}/${folder}")
      endif()
      
      set_target_properties(${target} PROPERTIES 
        FOLDER "${folder_tree}"
        RUNTIME_OUTPUT_DIRECTORY "${CMAKE_SOURCE_DIR}/bin/tools/$<CONFIG>")

    endif()
  endforeach()
endif()