cmake_minimum_required(VERSION 3.10.0 FATAL_ERROR)

project(simple-wayland-compositor)

set(CMAKE_INCLUDE_CURRENT_DIR ON)

find_package(PkgConfig REQUIRED)

pkg_get_variable(wayland_scanner wayland-scanner wayland_scanner)
pkg_get_variable(wayland-protocols_pkgdatadir wayland-protocols pkgdatadir)

find_file(xdg_shell stable/xdg-shell/xdg-shell.xml ${wayland-protocols_pkgdatadir})

add_custom_command(
    OUTPUT xdg-shell.c
    COMMAND ${wayland_scanner} ARGS private-code ${xdg_shell} xdg-shell.c
    DEPENDS ${xdg_shell}
)

add_custom_command(
    OUTPUT xdg-shell.h
    COMMAND ${wayland_scanner} ARGS server-header ${xdg_shell} xdg-shell.h
    DEPENDS ${xdg_shell}
)

add_executable(${PROJECT_NAME} wayland-compositor.c backend-x11.c xdg-shell.c xdg-shell.h)

pkg_check_modules(wayland-server REQUIRED IMPORTED_TARGET wayland-server)
pkg_check_modules(x11 REQUIRED IMPORTED_TARGET x11)
pkg_check_modules(x11-xcb REQUIRED IMPORTED_TARGET x11-xcb)
pkg_check_modules(xkbcommon REQUIRED IMPORTED_TARGET xkbcommon)
pkg_check_modules(xkbcommon-x11 REQUIRED IMPORTED_TARGET xkbcommon-x11)

find_package(OpenGL REQUIRED COMPONENTS EGL)

target_link_libraries(
    ${PROJECT_NAME}

    PkgConfig::wayland-server
    PkgConfig::x11
    OpenGL::GL
    OpenGL::EGL
    PkgConfig::x11-xcb
    PkgConfig::xkbcommon-x11
    PkgConfig::xkbcommon
)
