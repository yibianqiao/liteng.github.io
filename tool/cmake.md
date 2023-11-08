# 通用模板

```cmake
# cmake最低版本要求
cmake_minimum_required(VERSION 3.5 FATAL_ERROR)
# 编码语言
project (main LANGUAGES C)
# 生成器
set(CMAKE_GENERATOR "Unix Makefiles")
# 编译器
set(CMAKE_C_COMPILER "/usr/bin/gcc")
set(CMAKE_CXX_COMPILER "/usr/bin/g++")
# 编译器信息
message(STATUS "Is the C compiler loaded? ${CMAKE_C_COMPILER_LOADED}")
if(CMAKE_C_COMPILER_LOADED)
    message(STATUS "The C compiler ID is: ${CMAKE_C_COMPILER_ID}")
    message(STATUS "Is the C from GNU? ${CMAKE_COMPILER_IS_GNUCC}")
    message(STATUS "The C compiler version is: ${CMAKE_C_COMPILER_VERSION}")
endif()
# 编译选项
set(CMAKE_BUILD_TYPE Debug CACHE STRING "Build type" FORCE)
# 增加编译选项，指定C语言版本和线程库
set(CMAKE_C_FLAGS "-W -D_GNU_SOURCE")
set(CMAKE_C_STANDARD "99")
set(CMAKE_C_FLAGS_DEBUG "-rdynamic -O0 -ggdb")
set(CMAKE_C_FLAGS_RELEASE "-O2 -DNDEBUG")

message(STATUS "Build type: ${CMAKE_BUILD_TYPE}")
message(STATUS "C flags, Debug configuration: ${CMAKE_C_FLAGS_DEBUG}")
message(STATUS "C flags, Release configuration: ${CMAKE_C_FLAGS_RELEASE}")
message(STATUS "C flags, Release configuration with Debug info: ${CMAKE_C_FLAGS_RELWITHDEBINFO}")
message(STATUS "C flags, minimal Release configuration: ${CMAKE_C_FLAGS_MINSIZEREL}")
# 静态库输出路径
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/program/lib)
# 动态库输出路径
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/release/lib)
# 可执行程序输出路径
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/release/bin)
# 头文件搜索路径
include_directories(${PROJECT_SOURCE_DIR}/program/include)
# 库文件搜索路径
link_directories(${PROJECT_SOURCE_DIR}/program/lib)
link_directories(${PROJECT_SOURCE_DIR}/release/lib)

# 将${PROJECT_SOURCE_DIR}/program/src文件夹作为子文件夹
macro(SUBDIRNAMELIST_MACRO dir)
    file(GLOB children ${dir}/*)
    FOREACH(subdir ${children})
        add_subdirectory(${subdir})
    ENDFOREACH()
endmacro()
SUBDIRNAMELIST_MACRO(${PROJECT_SOURCE_DIR}/program/src)

```

# 子文件

## 可执行程序
```cmake
# 项目名
set(NAME "main")
# 将同级所有源文件加入
aux_source_directory(. SRCS)
# 设置可执行程序目标
add_executable(${NAME} ${SRCS})
# 链接其他库
target_link_libraries(${NAME} example)
```

## 库

```cmake
# 项目名
set(NAME "example")
# 将同级所有源文件加入
aux_source_directory(. SRCS)
# 编译为动态库
add_library(${NAME} SHARED ${SRCS})
# 链接其他库
target_link_libraries(${NAME} pthread)
```