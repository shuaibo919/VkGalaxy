cmake_minimum_required(VERSION 3.12)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

include(FetchContent)
project(vkgalaxy VERSION 1.0)
set(TARGET_NAME vkgalaxy)
# add fsanitize
if(NOT WIN32)
    set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -fsanitize=address,undefined")
endif()

# set output dir
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_RELEASE ${CMAKE_CURRENT_SOURCE_DIR}/bin/Release)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_DEBUG ${CMAKE_CURRENT_SOURCE_DIR}/bin/Debug)

# find include diretories and libraries
FetchContent_Declare(
    glfw
    GIT_REPOSITORY https://github.com/glfw/glfw
    GIT_TAG "3.4" # 3.4
    GIT_PROGRESS TRUE
    GIT_SHALLOW     TRUE
)
set(GLFW_BUILD_EXAMPLES  OFF)
set(GLFW_BUILD_TESTS      OFF)
set(GLFW_BUILD_DOCS      OFF)
set(GLFW_INSTALL      OFF)
FetchContent_MakeAvailable(glfw)
find_package(Vulkan REQUIRED)

# find source files:
file(GLOB_RECURSE vkgalaxy_src CONFIGURE_DEPENDS "src/*.cpp" "src/*.c")
add_executable(${TARGET_NAME} ${vkgalaxy_src})
target_include_directories(${TARGET_NAME} PRIVATE "src/")
target_link_libraries(${TARGET_NAME} PRIVATE Vulkan::Vulkan)
target_link_libraries(${TARGET_NAME} PRIVATE glfw )

# set working directory:
if(MSVC)
    set_property(TARGET ${TARGET_NAME} PROPERTY VS_DEBUGGER_WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/")
    set_property(TARGET ${TARGET_NAME} PROPERTY VS_DPI_AWARE "PerMonitor") # set dpi awareness (only works on windows)
    set_property(TARGET ${TARGET_NAME} PROPERTY MSVC_RUNTIME_LIBRARY MultiThreadedDLL)
else()
    set_property(TARGET ${TARGET_NAME} PROPERTY WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/")
endif()

# compile shaders:
file(GLOB_RECURSE vkgalaxy_shader_src CONFIGURE_DEPENDS "${CMAKE_CURRENT_SOURCE_DIR}/assets/shaders/*")
add_custom_target(vkgalaxy_shaders ALL DEPENDS ${TARGET_NAME} ${vkgalaxy_shader_src})

message("starting shader compilation")

if(WIN32) 
    add_custom_command(
        TARGET vkgalaxy_shaders POST_BUILD
        COMMAND ${CMAKE_CURRENT_SOURCE_DIR}/shader_compile.bat
        WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
    )
else()
    add_custom_command(
        TARGET vkgalaxy_shaders POST_BUILD
        COMMAND sh ${CMAKE_CURRENT_SOURCE_DIR}/shader_compile.sh
        WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
    )
endif()