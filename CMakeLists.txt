cmake_minimum_required(VERSION 3.21)
project(hexedit LANGUAGES C CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

if(WIN32)
    # Windows has no system curses: build PDCurses (Win32 console port) statically from source.
    include(FetchContent)
    FetchContent_Declare(pdcurses
        URL https://github.com/wmcbrine/PDCurses/archive/refs/tags/3.9.tar.gz
        URL_HASH SHA256=590dbe0f5835f66992df096d3602d0271103f90cf8557a5d124f693c2b40d7ec
        DOWNLOAD_EXTRACT_TIMESTAMP ON)
    FetchContent_MakeAvailable(pdcurses)
    file(GLOB PDCURSES_SOURCES ${pdcurses_SOURCE_DIR}/pdcurses/*.c ${pdcurses_SOURCE_DIR}/wincon/*.c)
    add_library(pdcurses STATIC ${PDCURSES_SOURCES})
    target_include_directories(pdcurses PUBLIC ${pdcurses_SOURCE_DIR} PRIVATE ${pdcurses_SOURCE_DIR}/wincon)
    target_compile_definitions(pdcurses PRIVATE _CRT_SECURE_NO_WARNINGS)
    set(CURSES_TARGET pdcurses)
else()
    find_package(Curses REQUIRED)
    add_library(curses INTERFACE)
    target_include_directories(curses INTERFACE ${CURSES_INCLUDE_DIRS})
    target_link_libraries(curses INTERFACE ${CURSES_LIBRARIES})
    set(CURSES_TARGET curses)
endif()

file(GLOB_RECURSE SOURCES CONFIGURE_DEPENDS src/*.cpp)
add_executable(hexedit ${SOURCES})
target_include_directories(hexedit PRIVATE include)
target_link_libraries(hexedit PRIVATE ${CURSES_TARGET})

if(MSVC)
    target_compile_options(hexedit PRIVATE /W4)
else()
    target_compile_options(hexedit PRIVATE -Wall -Wextra -pedantic)
endif()

if(MINGW)
    # Single self-contained .exe with no MinGW runtime DLLs.
    target_link_options(hexedit PRIVATE -static)
endif()

install(TARGETS hexedit)
