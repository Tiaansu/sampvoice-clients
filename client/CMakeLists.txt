project(sampvoice-client)

set(CMAKE_CXX_STANDARD 17)

set(SAMP_VERSION "R1" CACHE STRING "SAMP version to compile for")
set_property(CACHE SAMP_VERSION PROPERTY STRINGS "R1" "R3" "DL")

if(SAMP_VERSION STREQUAL "R1")
    add_compile_definitions(SAMP_R1)
elseif(SAMP_VERSION STREQUAL "R3")
    add_compile_definitions(SAMP_R3)
elseif(SAMP_VERSION STREQUAL "DL")
    add_compile_definitions(SAMP_DL)
else()
    message(FATAL_ERROR "Invalid SAMP_VERSION: ${SAMP_VERSION}. Must be R1, R3, or DL")
endif()

message(STATUS "Building for SAMP version: ${SAMP_VERSION}")

file(GLOB_RECURSE sampvoice_client_source_list "${CMAKE_CURRENT_SOURCE_DIR}/*.cpp" "${CMAKE_CURRENT_SOURCE_DIR}/*.hpp" "${CMAKE_CURRENT_SOURCE_DIR}/*.c" "${CMAKE_CURRENT_SOURCE_DIR}/*.h")

# Use local DirectX SDK from external folder
set(DXSDK_INCLUDE_DIR "${CMAKE_SOURCE_DIR}/external/dxsdk/include")
set(DXSDK_LIB_DIR "${CMAKE_SOURCE_DIR}/external/dxsdk/lib/x86")

include_directories(
    ${CMAKE_CURRENT_SOURCE_DIR}/include
    ${DXSDK_INCLUDE_DIR}
)

link_directories(
    ${CMAKE_CURRENT_SOURCE_DIR}/libraries
    ${DXSDK_LIB_DIR}
)

add_library(${PROJECT_NAME} SHARED ${sampvoice_client_source_list})

set_target_properties(${PROJECT_NAME} PROPERTIES PREFIX "")
set_target_properties(${PROJECT_NAME} PROPERTIES OUTPUT_NAME "sampvoice" SUFFIX ".asi")

# Shader compilation
file(GLOB shader_files "${CMAKE_CURRENT_SOURCE_DIR}/resources/*.hlsl")
foreach(shader ${shader_files})
    set_source_files_properties(${shader} PROPERTIES 
        VS_SHADER_TYPE Effect
        VS_SHADER_MODEL 2.0
    )
endforeach()
target_sources(${PROJECT_NAME} PRIVATE ${shader_files})

# Resource file
file(GLOB resource_files "${CMAKE_CURRENT_SOURCE_DIR}/resources/*.rc")
target_sources(${PROJECT_NAME} PRIVATE ${resource_files})

# Mark other resources as header-only
file(GLOB other_resources 
    "${CMAKE_CURRENT_SOURCE_DIR}/resources/*.ttf"
    "${CMAKE_CURRENT_SOURCE_DIR}/resources/*.png"
    "${CMAKE_CURRENT_SOURCE_DIR}/resources/*.aps"
    "${CMAKE_CURRENT_SOURCE_DIR}/resources/*.h"
)
set_source_files_properties(${other_resources} PROPERTIES HEADER_FILE_ONLY TRUE)
target_sources(${PROJECT_NAME} PRIVATE ${other_resources})

set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} /SAFESEH:NO")

target_link_libraries(${PROJECT_NAME} PRIVATE
    bass_fx.lib 
    bass.lib 
    opus.lib
    d3d9.lib
    d3dx9.lib
)

GroupSourcesByFolder(${PROJECT_NAME})