project(LIBSODIUM)
set(APPLICATION_NAME "libsodium")

cmake_minimum_required(VERSION 2.6.0)
set(CMAKE_COLOR_MAKEFILE ON)
#set(CMAKE_VERBOSE_MAKEFILE ON)

set_property(GLOBAL PROPERTY USE_FOLDERS ON)

add_subdirectory(src)

enable_testing()
add_subdirectory(test)
