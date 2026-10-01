cmake_minimum_required(VERSION 3.16)
project(KittyMemoryExStatic LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
option(KT_DISABLE_KEYSTONE "Disable keystone support" ON)

# 1. 只抓取 KittyMemoryEx 文件夹下的源码
file(GLOB KITTYMEMORY_SRC "${CMAKE_CURRENT_SOURCE_DIR}/KittyMemoryEx/*.cpp")

# 2. 声明为静态库
add_library(KittyMemoryEx STATIC ${KITTYMEMORY_SRC})

# 3. 公开包含目录，方便以后其他项目调用
target_include_directories(KittyMemoryEx PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/KittyMemoryEx)

# 4. 编译宏定义
target_compile_definitions(KittyMemoryEx PRIVATE $<$<BOOL:${KT_DISABLE_KEYSTONE}>:kNO_KEYSTONE>)

# 5. 编译选项（★ 新增体积优化参数 ★）
target_compile_options(KittyMemoryEx PRIVATE
    -Os                     # ★ 关键：优先优化体积，替代 -O2
    -ffunction-sections     # ★ 每个函数单独放一个段
    -fdata-sections         # ★ 每个数据单独放一个段
    -fvisibility=hidden     # ★ 隐藏不需要暴露的符号
    -Wall
    -Wextra
    -fexceptions
    -Wno-unused-command-line-argument
)

# 6. Android 平台专属设置
if(ANDROID)
    target_link_libraries(KittyMemoryEx PRIVATE log)
endif()

# 7. 强制指定输出名字为 libKittyMemoryEx.a
set_target_properties(KittyMemoryEx PROPERTIES
    OUTPUT_NAME "KittyMemoryEx"
    PREFIX "lib"
    SUFFIX ".a"
)
