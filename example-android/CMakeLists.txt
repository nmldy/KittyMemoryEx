cmake_minimum_required(VERSION 3.16)

project(ExampleProject
    VERSION 1.0
    LANGUAGES CXX
)

# -----------------------------
# Options (默认禁用 Keystone，省去 10MB 的库和路径配置烦恼)
# -----------------------------
option(KT_DISABLE_KEYSTONE "Disable keystone support" ON)

# -----------------------------
# C++ standard
# -----------------------------
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# -----------------------------
# KittyMemory library
# -----------------------------
set(KITTYMEMORY_PATH "${CMAKE_CURRENT_SOURCE_DIR}/../KittyMemoryEx")
file(GLOB KITTYMEMORY_SRC "${KITTYMEMORY_PATH}/*.cpp")

# 只有在没禁用 Keystone 的时候才去检查库文件，避免路径报错
if(NOT KT_DISABLE_KEYSTONE)
    set(KEYSTONE_LIB ${KITTYMEMORY_PATH}/Deps/Keystone/libs-android/${CMAKE_ANDROID_ARCH_ABI}/libkeystone.a)
    if(NOT EXISTS ${KEYSTONE_LIB})
        message(FATAL_ERROR "找不到 Keystone 静态库: ${KEYSTONE_LIB}\n请确认你的目标架构 ${CMAKE_ANDROID_ARCH_ABI} 是否有预编译库，或者把 KT_DISABLE_KEYSTONE 设为 ON。")
    endif()
endif()

# -----------------------------
# Project sources
# -----------------------------
file(GLOB PROJ_SRC "example.cpp")

# -----------------------------
# Executable
# -----------------------------
add_executable(ExampleProject ${PROJ_SRC} ${KITTYMEMORY_SRC})

# -----------------------------
# Includes
# -----------------------------
target_include_directories(ExampleProject
    PRIVATE
        ${KITTYMEMORY_PATH}
)

# -----------------------------
# Compile definitions
# -----------------------------
target_compile_definitions(ExampleProject
    PRIVATE
        $<$<CONFIG:Debug>:DEBUG>
        $<$<CONFIG:Release>:NDEBUG>
        $<$<BOOL:${KT_DISABLE_KEYSTONE}>:kNO_KEYSTONE>
)

# -----------------------------
# Compile options (移除了 -Werror，避免因微小警告中断编译)
# -----------------------------
target_compile_options(ExampleProject
    PRIVATE
        $<$<CONFIG:Debug>:-O0 -g>
        $<$<CONFIG:Release>:-O2>
        -Wall
        -Wextra
        -fexceptions
        -Wno-unused-command-line-argument
)

# -----------------------------
# Platform-specific libraries
# -----------------------------
if(ANDROID)
    target_link_libraries(ExampleProject PRIVATE log)
    
    # 只有没禁用 Keystone 才链接它
    if(NOT KT_DISABLE_KEYSTONE)
        target_link_libraries(ExampleProject PRIVATE ${KEYSTONE_LIB})
    endif()
endif()
