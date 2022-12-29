cmake_minimum_required(VERSION 3.20.0)
project(imgui-sdl-template)

# Specify the C++ standard.
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)

# Export compile_commands.json file.
set(CMAKE_EXPORT_COMPILE_COMMANDS True)

execute_process(COMMAND uname OUTPUT_VARIABLE uname)
if (uname MATCHES "^MSYS" OR uname MATCHES "^MINGW")
    set(MSYS2 TRUE)
    message(STATUS "This is MSYS")
else()
    set(MSYS2 FALSE)
    message(STATUS "This is not MSYS")
endif()

# Setup source locations.
set(SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/src)
set(IMGUI_SOURCES_DIR ${CMAKE_CURRENT_SOURCE_DIR}/ext/imgui)
set(IMGUI_SOURCES
    ${IMGUI_SOURCES_DIR}/imgui.cpp
    ${IMGUI_SOURCES_DIR}/imgui_demo.cpp
    ${IMGUI_SOURCES_DIR}/imgui_draw.cpp
    ${IMGUI_SOURCES_DIR}/imgui_tables.cpp
    ${IMGUI_SOURCES_DIR}/imgui_widgets.cpp
    ${IMGUI_SOURCES_DIR}/backends/imgui_impl_sdl.cpp
    ${IMGUI_SOURCES_DIR}/backends/imgui_impl_opengl3.cpp
)
set(SOURCES
    ${SOURCE_DIR}/main.cpp
)
set(HEADERS
    ${IMGUI_SOURCES_DIR}/imgui.h
    ${IMGUI_SOURCES_DIR}/imconfig.h
    ${IMGUI_SOURCES_DIR}/imgui_internal.h
    ${IMGUI_SOURCES_DIR}/imstb_rectpack.h
    ${IMGUI_SOURCES_DIR}/imstb_textedit.h
    ${IMGUI_SOURCES_DIR}/imstb_truetype.h
    ${IMGUI_SOURCES_DIR}/backends/imgui_impl_sdl.h
    ${IMGUI_SOURCES_DIR}/backends/imgui_impl_opengl3.h
)

# Find external packages.
find_package(OpenGL REQUIRED)
find_package(PkgConfig REQUIRED)
pkg_check_modules(SDL2 REQUIRED sdl2)

# Executable and set WIN32 to make sure it's GUI application, not console application.
# Console application in this case means that on Windows, starting the program will open
# terminal window beside the GUI application.
set(EXECUTABLE imgui_example)
add_executable(${EXECUTABLE} WIN32)
target_sources(${EXECUTABLE} PUBLIC ${HEADERS} PRIVATE ${IMGUI_SOURCES} ${SOURCES})

# Enable compiler warnings.
target_compile_options(${EXECUTABLE} PRIVATE -Wall -Wformat -Wpedantic -Werror)

target_include_directories(
    ${EXECUTABLE}
    PRIVATE ${IMGUI_SOURCES_DIR}
    PRIVATE ${IMGUI_SOURCES_DIR}/backends
    PUBLIC ${SDL2_INCLUDE_DIRS}
)

# On Windows MSYS2 link libraries statically to the binary. This way binary can
# be more easily distributed and does not depend on MSY2 being installed. On
# Linux just load used shared libraries instead.
if (MSYS2)
    target_link_libraries(${EXECUTABLE} ${SDL2_STATIC_LIBRARIES} OpenGL::GL)
    target_link_libraries(${EXECUTABLE} -static)

else()
    target_link_libraries(${EXECUTABLE} ${SDL2_LIBRARIES} OpenGL::GL)
endif()

target_compile_options(${EXECUTABLE} PUBLIC ${SDL2_CFLAGS_OTHER})
