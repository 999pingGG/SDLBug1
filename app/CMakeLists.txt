cmake_minimum_required(VERSION 3.17)

project(bug C)

add_library(bug SHARED jni/src/main.c)

target_compile_options(bug PUBLIC -fsanitize=address -fno-omit-frame-pointer)
set_target_properties(bug PROPERTIES LINK_FLAGS -fsanitize=address)

find_package(SDL3 REQUIRED CONFIG COMPONENTS SDL3-shared)
target_link_libraries(bug PRIVATE SDL3::SDL3)