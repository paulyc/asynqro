cmake_minimum_required(VERSION 3.12.0)
project(asynqro LANGUAGES CXX VERSION 0.1.0)
include(GenerateExportHeader)

option(ASYNQRO_BUILD_TESTS
    "Build tests or not at asynqro build. Will download googletest from github during configure time and build it with tests"
    ON
)

find_package(Qt5Core CONFIG REQUIRED)

add_library(asynqro SHARED
    src/future.cpp
    src/failure_handling.cpp
    src/tasksdispatcher.cpp

    include/asynqro/asynqro
    include/asynqro/future.h
    include/asynqro/tasks.h
    include/asynqro/impl/promise.h
    include/asynqro/impl/cancelablefuture.h
    include/asynqro/impl/failure_handling.h
    include/asynqro/impl/spinlock.h
    include/asynqro/impl/typetraits.h
    include/asynqro/impl/zipfutures.h
    include/asynqro/impl/containers_helpers.h
    include/asynqro/impl/containers_traverse.h
    include/asynqro/impl/tasksdispatcher.h
    include/asynqro/impl/taskslist_p.h
)
add_library(asynqro::asynqro ALIAS asynqro)
generate_export_header(asynqro EXPORT_FILE_NAME generated_include/asynqro/impl/asynqro_export.h)

set_target_properties(asynqro PROPERTIES
    CXX_STANDARD 17
    CXX_STANDARD_REQUIRED ON
    CXX_EXTENSIONS OFF
    C_VISIBILITY_PRESET hidden
    CXX_VISIBILITY_PRESET hidden
    POSITION_INDEPENDENT_CODE ON
    AUTOMOC ON
)

target_include_directories(asynqro PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<BUILD_INTERFACE:${CMAKE_BINARY_DIR}/generated_include>
    $<INSTALL_INTERFACE:include>
)
target_link_libraries(asynqro PUBLIC Qt5::Core)

install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/include/asynqro
    DESTINATION include
    FILES_MATCHING
    PATTERN "*.h"
    PATTERN "*_p.h" EXCLUDE
)

install(FILES ${CMAKE_CURRENT_SOURCE_DIR}/include/asynqro/asynqro
    DESTINATION include/asynqro
)

install(DIRECTORY ${CMAKE_BINARY_DIR}/generated_include/asynqro
    DESTINATION include
    FILES_MATCHING
    PATTERN "*.h"
    PATTERN "*_p.h" EXCLUDE
)

install(TARGETS asynqro
   EXPORT asynqro-targets
   RUNTIME DESTINATION lib
   LIBRARY DESTINATION lib
   ARCHIVE DESTINATION lib
)

install(EXPORT asynqro-targets DESTINATION lib/cmake
   NAMESPACE asynqro::
)

install(FILES ${CMAKE_CURRENT_SOURCE_DIR}/cmake/asynqro-config.cmake
   DESTINATION lib/cmake
)

if(ASYNQRO_BUILD_TESTS)
    enable_testing()
    add_subdirectory(tests)
endif()

# TODO: add gcov/codecov support
#set(CMAKE_CXX_FLAGS "-g -O0 --coverage")
#set(CMAKE_EXE_LINKER_FLAGS "--coverage")
#set(CMAKE_SHARED_LINKER_FLAGS "--coverage")
#set(CMAKE_CXX_OUTPUT_EXTENSION_REPLACE ON)