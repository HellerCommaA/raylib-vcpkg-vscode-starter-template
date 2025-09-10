cmake_minimum_required (VERSION 3.13...3.21)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_CURRENT_SOURCE_DIR}/cmake/modules/")

include(FetchContent)
function(add_git_dependency libName gitURL)
    FetchContent_Declare(${libName}
        GIT_REPOSITORY ${gitURL}
        GIT_SHALLOW    TRUE
        GIT_PROGRESS   TRUE
    )
    FetchContent_MakeAvailable(${libName})
    target_compile_options(${libName} PRIVATE "-w")
endfunction()

if(NOT DEFINED CMAKE_TOOLCHAIN_FILE)
    set(CMAKE_TOOLCHAIN_FILE
        "${CMAKE_CURRENT_SOURCE_DIR}/vcpkg/scripts/buildsystems/vcpkg.cmake"
        CACHE STRING "Vcpkg toolchain file")
endif()

project(
    raylib-template  # Project name, change this as needed.
    LANGUAGES C CXX
)

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}/bin")  # Keep all runtime files in one directory.

file(
    GLOB_RECURSE SOURCE_FILES
    CONFIGURE_DEPENDS  # Automatically reconfigure if source files are added/removed.
    ${PROJECT_SOURCE_DIR}/src/*.cpp
    ${PROJECT_SOURCE_DIR}/src/*.hpp
)

add_subdirectory(src)

# Ensure the C++17 standard is available.
target_compile_features(${PROJECT_NAME} PRIVATE cxx_std_17)

# Enforce UTF-8 encoding on MSVC.
if (MSVC)
    target_compile_options(${PROJECT_NAME} PRIVATE /utf-8)
endif()

# Enable warnings recommended for new projects.
if (MSVC)
    target_compile_options(${PROJECT_NAME} PRIVATE /W4)
else()
    target_compile_options(${PROJECT_NAME} PRIVATE -Wall -Wextra -Wno-c++11-narrowing -Wno-unused-parameter)
endif()

find_package(EnTT CONFIG REQUIRED)
target_link_libraries(${PROJECT_NAME} PRIVATE EnTT::EnTT)

find_package(SFML COMPONENTS Network Graphics Window Audio System CONFIG REQUIRED)
target_link_libraries(${PROJECT_NAME} PRIVATE SFML::Network SFML::Graphics SFML::Window SFML::Audio SFML::System)

target_link_options(${PROJECT_NAME} PRIVATE -fsanitize=address,undefined)

# Define the source directory for your data
set(DATA_SOURCE_DIR "${CMAKE_SOURCE_DIR}/data")

# Define the destination directory where the data should be copied
set(DATA_DEST_DIR "${CMAKE_BINARY_DIR}/data")

# if you want to run the command without relying on any specific target
add_custom_target(copy_data ALL
    COMMAND ${CMAKE_COMMAND} -E copy_directory
    "${DATA_SOURCE_DIR}" $<TARGET_FILE_DIR:${PROJECT_NAME}>/data/
    COMMENT "Copying data directory to output..."
)

# Ensure the custom target runs before building the executable
add_dependencies(${PROJECT_NAME} copy_data)

