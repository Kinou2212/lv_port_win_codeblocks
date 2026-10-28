cmake_minimum_required(VERSION 3.16)

project(LVGL_SIM LANGUAGES C)

# resources.rc 需要 RC 语言（MinGW 下会自动调用 windres）
enable_language(RC)

# 对应原项目的 -std=c17
set(CMAKE_C_STANDARD 17)
set(CMAKE_C_STANDARD_REQUIRED ON)
#set(CMAKE_C_EXTENSIONS OFF)   # 用 -std=c17 而非 -std=gnu17

# 默认 Debug，对应原项目 ActiveTarget
if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Debug CACHE STRING "Build type (Debug/Release)" FORCE)
endif()

# 输出到 bin/Debug 或 bin/Release，对应原项目的 bin/${Target}/LVGL
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_SOURCE_DIR}/bin/${CMAKE_BUILD_TYPE}")

# ============ 收集源文件（对应 .cbp 里的 <Unit> 列表）============
file(GLOB_RECURSE LVGL_LIB_SOURCES CONFIGURE_DEPENDS
        "${CMAKE_SOURCE_DIR}/lvgl/src/*.c"
)
file(GLOB_RECURSE LVGL_DEMO_SOURCES CONFIGURE_DEPENDS
        "${CMAKE_SOURCE_DIR}/lvgl/demos/*.c"
)
file(GLOB_RECURSE LVGL_EXAMPLE_SOURCES CONFIGURE_DEPENDS
        "${CMAKE_SOURCE_DIR}/lvgl/examples/*.c"
)

# 排除移植模板（原项目虽然列了，但它们内部通常是 #if 0，不参与实际编译）
list(FILTER LVGL_EXAMPLE_SOURCES EXCLUDE REGEX "_template\\.c$")

add_executable(LVGL
        main.c
        ${LVGL_LIB_SOURCES}
        ${LVGL_DEMO_SOURCES}
        ${LVGL_EXAMPLE_SOURCES}
        resources.rc
)

# ============ 头文件路径（对应 <Add directory="." /> 和 "lvgl"）============
target_include_directories(LVGL PRIVATE
        "${CMAKE_SOURCE_DIR}"
        "${CMAKE_SOURCE_DIR}/lvgl"
)

# ============ 宏定义（对应 -DWINVER=0x0601 -DWIN32 -D_WIN32）============
target_compile_definitions(LVGL PRIVATE
        WINVER=0x0601
        WIN32
        _WIN32
        LV_CONF_INCLUDE_SIMPLE   # 让 lv_conf_internal.h 直接 #include "lv_conf.h"
)

# ============ 编译选项 ============
target_compile_options(LVGL PRIVATE -Wall)

if(CMAKE_BUILD_TYPE STREQUAL "Debug")
    target_compile_options(LVGL PRIVATE -g)      # 对应 Debug 的 -g
else()
    target_compile_options(LVGL PRIVATE -O3)     # 对应 Release 的 -O3
endif()

# ============ 链接选项（对应 -lmingw32 -mwindows -mconsole）============
target_link_options(LVGL PRIVATE -mwindows -mconsole)
target_link_libraries(LVGL PRIVATE mingw32)
