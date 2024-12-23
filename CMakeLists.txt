cmake_minimum_required(VERSION 3.10)
project(lua C)

# 设置C标准为C99
set(CMAKE_C_STANDARD 99)
set(CMAKE_C_STANDARD_REQUIRED ON)

# 根据编译器设置不同的编译选项
if(MSVC)
    # MSVC特定的编译选项
    add_compile_options(
        /W4     # 警告级别
        /MP     # 多处理器编译
    )
    
    # 在Release模式下启用优化
    set(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} /O2")
    # 在Debug模式下启用运行时检查
    set(CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} /RTC1")
else()
    # GCC/Clang的编译选项
    set(CWARNSCPP
        -Wfatal-errors
        -Wextra
        -Wshadow
        -Wundef
        -Wwrite-strings
        -Wredundant-decls
        -Wdisabled-optimization
        -Wdouble-promotion
        -Wmissing-declarations
        -Wconversion
        -Wuninitialized
        -Wstrict-overflow=2
    )

    set(CWARNSC
        -Wdeclaration-after-statement
        -Wmissing-prototypes
        -Wnested-externs
        -Wstrict-prototypes
        -Wc++-compat
        -Wold-style-definition
    )

    if(CMAKE_C_COMPILER_ID STREQUAL "GNU")
        set(CWARNGCC
            -Wlogical-op
            -Wno-aggressive-loop-optimizations
        )
    endif()

    # 合并所有警告选项
    set(CWARNS ${CWARNSCPP} ${CWARNSC} ${CWARNGCC})

    add_compile_options(
        -Wall
        -O2
        -fno-stack-protector
        -fno-common
        -march=native
        ${CWARNS}
    )
endif()

# 根据平台添加适当的定义
if(WIN32)
    # 移除 DLL 相关的定义
    # add_definitions(-DLUA_BUILD_AS_DLL)  
else()
    add_definitions(-DLUA_USE_LINUX)
endif()

# 源文件列表
set(CORE_SOURCES
    lapi.c lcode.c lctype.c ldebug.c ldo.c ldump.c lfunc.c lgc.c llex.c
    lmem.c lobject.c lopcodes.c lparser.c lstate.c lstring.c ltable.c
    ltm.c lundump.c lvm.c lzio.c ltests.c
)

set(AUX_SOURCES
    lauxlib.c
)

set(LIB_SOURCES
    lbaselib.c ldblib.c liolib.c lmathlib.c loslib.c ltablib.c lstrlib.c
    lutf8lib.c loadlib.c lcorolib.c linit.c
)

# 创建静态库
add_library(lua STATIC
    ${CORE_SOURCES}
    ${AUX_SOURCES}
    ${LIB_SOURCES}
)

# 创建可执行文件
add_executable(lua_demo lua.c)

# 根据平台设置不同的链接选项
if(WIN32)
    target_link_libraries(lua_demo PRIVATE lua)
else()
    target_link_libraries(lua_demo
        PRIVATE
        lua
        m  # 数学库
        dl # 动态链接库
    )
    target_link_options(lua_demo PRIVATE -Wl,-E)
endif()

# 安装规则
install(TARGETS lua_demo lua
    RUNTIME DESTINATION bin
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib
) 