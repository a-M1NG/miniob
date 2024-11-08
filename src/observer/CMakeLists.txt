cmake_minimum_required(VERSION 3.10)
project(observer_project)

# 打印当前源码目录
message(STATUS "This is CMAKE_CURRENT_SOURCE_DIR dir ${CMAKE_CURRENT_SOURCE_DIR}")

# 包含当前源码目录
include_directories(${CMAKE_CURRENT_SOURCE_DIR})

# 查找所有源码文件
file(GLOB_RECURSE ALL_SRC *.cpp *.c)
set(MAIN_SRC ${CMAKE_CURRENT_SOURCE_DIR}/main.cpp)
message("MAIN SRC: ${MAIN_SRC}")

# 排除 main.cpp，并将其他源码文件设置为库源文件
foreach(F ${ALL_SRC})
    if(NOT ${F} STREQUAL ${MAIN_SRC})
        set(LIB_SRC ${LIB_SRC} ${F})
        message("Use ${F}")
    endif()
endforeach()

# 设置 libevent 的路径并查找 libevent
# set(Libevent_DIR "../deps/3rd/libevent/build")
find_package(Libevent CONFIG REQUIRED)

# 启用 PkgConfig 并使用它来查找 jsoncpp
find_package(PkgConfig REQUIRED)
pkg_check_modules(JSONCPP REQUIRED jsoncpp)

# 添加 jsoncpp 的库和头文件路径
include_directories(${JSONCPP_INCLUDE_DIRS})
list(APPEND LIBRARIES ${JSONCPP_LIBRARIES})

# 设置所需的库，包括 libevent 和其他依赖项
set(LIBRARIES ${LIBRARIES} common pthread dl libevent::core libevent::pthreads)

# 指定目标文件位置
set(EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR}/bin)
message("Binary directory: ${EXECUTABLE_OUTPUT_PATH}")
set(LIBRARY_OUTPUT_PATH ${PROJECT_BINARY_DIR}/lib)
message("Archive directory: ${LIBRARY_OUTPUT_PATH}")

# 添加可执行文件
add_executable(observer ${MAIN_SRC})
target_link_libraries(observer observer_static)

# 添加静态库
add_library(observer_static STATIC ${LIB_SRC})

# 链接 readline 库
include(readline)
MINIOB_FIND_READLINE()
if(HAVE_READLINE)
    target_link_libraries(observer_static ${READLINE_LIBRARY})
    target_include_directories(observer_static PRIVATE ${READLINE_INCLUDE_DIR})
    add_definitions(-DUSE_READLINE)
    message("observer_static uses readline")
else()
    message("readline is not found")
endif()

# 设置目标库属性
set_target_properties(observer_static PROPERTIES OUTPUT_NAME observer)
target_link_libraries(observer_static ${LIBRARIES})

# 安装目标文件
install(TARGETS observer observer_static 
    RUNTIME DESTINATION bin
    ARCHIVE DESTINATION lib)
