cmake_minimum_required(VERSION 3.16)
project(concordpp)

set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# enable ccache
find_program(CCACHE_PROGRAM ccache)
if (CCACHE_PROGRAM)
    set_property(GLOBAL PROPERTY RULE_LAUNCH_COMPILE "${CCACHE_PROGRAM}")
endif ()

add_library(${PROJECT_NAME} STATIC
    include/concordpp.hpp
)

set_target_properties(${PROJECT_NAME} PROPERTIES
        CXX_STANDARD 20
        CXX_STANDARD_REQUIRED ON
        POSITION_INDEPENDENT_CODE ON
        LINKER_LANGUAGE CXX
)

include(ExternalProject)
ExternalProject_Add(project_concord
  GIT_REPOSITORY https://github.com/Cogmasters/concord
  PREFIX ${CMAKE_CURRENT_BINARY_DIR}/concord
  CONFIGURE_COMMAND ""
  BUILD_COMMAND make
  BUILD_IN_SOURCE true
)
ExternalProject_Get_Property(project_concord SOURCE_DIR)

target_include_directories(${PROJECT_NAME} PUBLIC ${CMAKE_CURRENT_LIST_DIR}/include)
target_link_libraries(${PROJECT_NAME} ${SOURCE_DIR}/src/project_concord/lib/libdiscord.a)

if(BUILD_TESTING)
    set(ENV_FILE_SOURCE ${CMAKE_SOURCE_DIR}/.env)
    set(ENV_FILE_DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/.env)
    configure_file(${ENV_FILE_SOURCE} ${ENV_FILE_DESTINATION} COPYONLY)

    add_executable(${PROJECT_NAME}_test test.cpp)
    target_link_libraries(${PROJECT_NAME}_test PRIVATE ${PROJECT_NAME})

    set_target_properties(${PROJECT_NAME}_test PROPERTIES
        CXX_STANDARD 20
        CXX_STANDARD_REQUIRED ON
    )
endif()