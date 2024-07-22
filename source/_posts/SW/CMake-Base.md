---
title: CMake入门与进阶
date: 2021-08-12 21:04:18
tags:
    - CMake
categories: 
    - 软件
---


CMake为源码提供多项目的支持，是非常好的跨平台编译器。
<!-- more -->  


但是这工具随着版本更新，越来越冗杂，社区也提出modern CMake帮助用户更好的使用cmake。当构建的项目复杂到一定程序时，维护CMake将变得非常不便，本帖将从最简单的功能出发，一步步建立起CMake架构，希望对你能有所帮助。


# 安装
官网下载：[Cmake](https://cmake.org/)
```bash
sudo apt-get install cmake make
cmake --version
```
cmake学习参考：
* 官方文档：[Cmake documentation](https://cmake.org/documentation/)
* wiki：[wiki](https://gitlab.kitware.com/cmake/community/-/wikis/home)
* 博客：[Linux下CMake简明教程](https://blog.csdn.net/whahu1989/article/details/82078563?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522162864933516780261984309%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=162864933516780261984309&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-1-82078563.first_rank_v2_pc_rank_v29&utm_term=cmake&spm=1018.2226.3001.4449)
* cmake examples，很好的新手教程，下载：
  
```bash
git clone git@github.com:ttroy50/cmake-examples.git
```

***OK, let's start !***

注：本文不使用link_libraries这类不带target前缀的函数，后者通常在add_executable之后加入，前者可以任意地方引入，尽管前者更为简单，但是不能对多类目标构建清晰的结构，新版本的CMake建议使用target前缀的函数。
# 基础功能
## 第一个Cmake项目
CMakeLists.txt如下，功能十分简单，设置最低版本要求，新建project，增加从源码编译可执行文件的规则。
```bash
# Set the minimum version of CMake that can be used
# To find the cmake version run
# $ cmake --version
cmake_minimum_required(VERSION 3.5)

# Set the project name
project (hello_cmake)

# set (EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/bin)

# Add an executable
add_executable(hello_cmake main.cpp)
```
编译：
```bash
mkdir build && cd build
cmake .. && make -j4
```
我们可以通过下面命令将可执行文件重定向到bin目录下：
```bash
set (EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/bin)
```
## 增加头文件
通常头文件放在include目录下，源码放在src下，main文件放在src的一级目录，函数文件可以继续递归放置：
```bash
.
├── CMakeLists.txt
├── include
│   └── Hello.h
└── src
    ├── Hello.cpp
    └── main.cpp
```
看一下CMakeLists.txt，新建一个SOURCE的变量（在多源码情况下可以建立多个不同的名称），该变量的值为一个“数组”，接下来在生成可执行文件函数`add_executable`中使用该变量，在多个变量情况下可以在末尾继续添加，也可以使用file命令`file(GLOB SOURCES "src/*.cpp")`通过搜索通配符添加，也可以使用`aux_source_directory(src SOURCES)`添加。最后链接头文件，使用`target_include_directories`函数，该函数第一个变量是目标target，这里指可执行文件，之后是`PRIVATE|PUBLIC|INTERFACE`，顾名思义这是指作用域，之后再解释，最后一个参数是路径，用`PROJECT_SOURCE_DIR`指向路径。
```bash
cmake_minimum_required(VERSION 3.5)
project (hello_headers)

# Create a sources variable with a link to all cpp files to compile
set(SOURCES
    src/Hello.cpp
    src/main.cpp
)
# file(GLOB SOURCES "src/*.cpp")
# aux_source_directory(src SOURCES)

# Add an executable with the above sources
add_executable(hello_headers ${SOURCES})

# Set the directories that should be included in the build command for this target
# when running g++ these will be included as -I/directory/path/
target_include_directories(hello_headers
    PRIVATE 
        ${PROJECT_SOURCE_DIR}/include
)
```
## 生成静态库
静态库给其他程序编译时使用并全部编译进可执行程序，因此在程序运行时不需要链接，后缀为`.a`。
```bash
.
├── build
├── CMakeLists.txt
├── src
│   ├── Hello.cpp				   # 库的头文件
│   └── main.cpp
├── src
│   └── Hello.cpp                  # 库的接口文件
└── static_libs
    └── libhello_library.a
```
CMakeLists.txt，生成静态库，通过`add_library`函数导入库源码，第一个参数时target，这里是库名称xxx，最后生成的也是libxxx.a，第二个参数是类型`STATIC|SHARED|MODULE`，第三个参数是源文件。使用`target_include_directories`函数为目标链接头文件。这里`PUBLIC|PRIVATE|INTERFACE`区别如下：
* PRIVATE - the directory is added to this target's include directories，只给库自己调用，或者外部引用库的程序使用
* INTERFACE - the directory is added to the include directories for any targets that link this library，可以给外部的或者内部其他目标使用
* PUBLIC - As above, it is included in this library and also any targets that link this library，同上，但只导出符号


```bash
cmake_minimum_required(VERSION 3.5)
project(hello_library)

set(LIBRARY_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/static_libs)   # 重定向输出目录

#Generate the static library from the library sources
add_library(hello_library STATIC 
    src/Hello.cpp
)

target_include_directories(hello_library
    PUBLIC 
        ${PROJECT_SOURCE_DIR}/include
)
```
编译后在static_libs下面生成了静态库。让我们在这个project内部使用这个静态库，使用`target_link_libraries`函数为可执行文件目标添加静态库。这里的范围关键字参考[CMake target_link_libraries Interface Dependencies](https://stackoverflow.com/questions/26037954/cmake-target-link-libraries-interface-dependencies)
```bash
# Add an executable with the above sources
add_executable(hello_binary 
    src/main.cpp
)

# link the new hello_library target with the hello_binary target
target_link_libraries( hello_binary
    PRIVATE 
        hello_library
)
```
注意：在main.cpp中必须设置正确的头文件路径：
```cpp
#include "static/Hello.h"
```
在外部程序中如何使用静态库？可以将库文件和头文件拷贝到新的project中，这里注意库和新的可执行文件的编译工具需要相同。
```bash
.
├── build
├── CMakeLists.txt
├── include
│   └── Hello.h
├── libs
│   └── libhello_library.a
└── src
    └── main.cpp
```
```bash
cmake_minimum_required(VERSION 3.5)
project(test_libs)

# set(CMAKE_C_COMPILER "arm-none-eabi-gcc")
# set(CMAKE_CXX_COMPILER "arm-none-eabi-g++")

# Add an executable with the above sources
add_executable(hello_binary 
    src/main.cpp
)

# link the new hello_library target with the hello_binary target
target_link_libraries( hello_binary
    PRIVATE 
    	${PROJECT_SOURCE_DIR}/libs/libhello_library.a
)

target_include_directories(hello_binary
    PRIVATE
        ${PROJECT_SOURCE_DIR}/include
)
```
这里`target_link_libraries`函数在使用时，既可以是具体的库文件目录也可以是系统中安装的库。我们的库是使用gcc编译的，如果将上面CMakeLists.txt文件中打开交叉编译工具，则将无法编译通过。
## 生成动态库
动态库不会编译到程序中，因此可执行文件体积较小，但是运行时需要链接动态库。动态库的后缀为`.so`，可以方便库函数的升级而不需要升级程序本体。
```bash
.
├── build
├── CMakeLists.txt
├── include
│   └── shared
│       └── Hello.h
├── README.adoc
└── src
    ├── Hello.cpp
    └── main.cpp
```
CMakeLists.txt
```bash
cmake_minimum_required(VERSION 3.5)
project(hello_library)

############################################################
# Create a library
############################################################

#Generate the shared library from the library sources
add_library(hello_library SHARED 
    src/Hello.cpp
)
add_library(hello::library ALIAS hello_library)   # ALIAS关键字，内部重命名

target_include_directories(hello_library
    PUBLIC 
        ${PROJECT_SOURCE_DIR}/include
)

############################################################
# Create an executable
############################################################

# Add an executable with the above sources
add_executable(hello_binary
    src/main.cpp
)

# link the new hello_library target with the hello_binary target
target_link_libraries( hello_binary
    PRIVATE 
        hello::library
)
```
编译后在build目录下生成了`libhello_library.so`和可执行文件，注意在库文件删除或者移动时，程序无法执行，如何安装库文件将在之后介绍。

第三方程序引用库可以参考：
```bash
target_link_libraries( hello_binary
    PRIVATE 
        ${PROJECT_SOURCE_DIR}/build/libhello_library.so
)

target_include_directories(hello_binary
    PRIVATE 
        ${PROJECT_SOURCE_DIR}/include
)
```
## 安装
大型的CMake开源软件在编译后会提供可执行文件、头文件和动态库的安装，通常安装到`/usr/loacl/lib`和`/usr/local/bin`目录中，也可以自定义安装位置，修改变量`${CMAKE_INSTALL_PREFIX}`，像交叉编译时，一般输出到其他地方以供拷贝。
```
.
├── cmake-examples.conf
├── CMakeLists.txt
├── include
│   └── installing
│       └── Hello.h
└── src
    ├── Hello.cpp
    └── main.cpp
```
CMakeLists.txt：
```bash
cmake_minimum_required(VERSION 3.5)
project(cmake_examples_install)

# 生成动态库
add_library(cmake_examples_inst SHARED src/Hello.cpp)
target_include_directories(cmake_examples_inst PUBLIC  ${PROJECT_SOURCE_DIR}/include)

# 生成可执行文件
add_executable(cmake_examples_inst_bin src/main.cpp)
target_link_libraries( cmake_examples_inst_bin PRIVATE cmake_examples_inst)  #使用库

#安装可执行文件
install (TARGETS cmake_examples_inst_bin DESTINATION bin)

# 安装库
install (TARGETS cmake_examples_inst LIBRARY DESTINATION lib)

# 安装头文件
install(DIRECTORY ${PROJECT_SOURCE_DIR}/include/ DESTINATION include)

# 安装配置文件
install (FILES cmake-examples.conf DESTINATION etc)
```
先运行一下：
```bash
mkdir build install && cd build
cmake .. -DCMAKE_INSTALL_PREFIX=../install
make & make install
cd ../install && tree
.
├── bin
│   └── cmake_examples_inst_bin
├── etc
│   └── cmake-examples.conf
├── include
│   └── installing
│       └── Hello.h
└── lib
    └── libcmake_examples_inst.so
```
CMake不提供卸载命令`make uninstall`，如果想卸载库和可执行文件，将编译生成的`install_manifest.txt`中的文件对应删除即可。参考[Can I do "make uninstall" with CMake?](https://gitlab.kitware.com/cmake/community/-/wikis/FAQ#can-i-do-make-uninstall-with-cmake)
## 编译类型
CMake支持几种编译类型：`Debug`，`Release`，`MinSizeRel`，`RelWithDebInfo`，由`${CMAKE_BUILD_TYPE}`决定。
```bash
# Set the minimum version of CMake that can be used
# To find the cmake version run
# $ cmake --version
cmake_minimum_required(VERSION 3.5)

# Set a default build type if none was specified
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
  message("Setting build type to 'RelWithDebInfo' as none was specified.")
  set(CMAKE_BUILD_TYPE RelWithDebInfo CACHE STRING "Choose the type of build." FORCE)
  # Set the possible values of build type for cmake-gui
  set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "Debug" "Release"
    "MinSizeRel" "RelWithDebInfo")
endif()

# Set the project name
project (build_type)

# Add an executable
add_executable(cmake_examples_build_type main.cpp)
```
默认采用的是`RelWithDebInfo`的编译方式，使用size工具查看不同类型的大小：
```bash
size -B cmake_examples_build_type
   text    data     bss     dec     hex filename
   2357     656     280    3293     cdd cmake_examples_build_type
```
比较：

| 类型 | text |data|bss|dec|
|--|--|--|--|--|
| Debug | 2477 | 664|280 | 3421|
| RelWithDebInfo| 2357 | 656|280 |3293 |
| Release|  2357 |656 |280 | 3293|
| MinSizeRel |2257  | 656|280 | 3193|

## 编译标志/变量
本节涉及到一个很重要的功能就是控制源码中的编译条件，我们先看C++代码：
```cpp
#include <iostream>

int main(int argc, char *argv[])
{
   std::cout << "Hello Compile Flags!" << std::endl;

   // only print if compile flag set
#ifdef EX2
  std::cout << "Hello Compile Flag EX2!" << std::endl;
#endif

#ifdef EX3
  std::cout << "Hello Compile Flag EX3!" << std::endl;
#endif

   return 0;
}
```
这里有两个宏`EX2`和`EX3`，它们可以由CMake决定是否定义，这是一个很强大的功能，相对也比较不容易理解。先来看下面CMakeLists.txt：
```bash
cmake_minimum_required(VERSION 3.5)

# 添加CXX编译控制符-DX2，缓冲CACHE类型，强制FORCE设置
set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DEX2" CACHE STRING "Set C++ Compiler Flags" FORCE)

project (compile_flags)
add_executable(cmake_examples_compile_flags main.cpp)

# 直接将EX3导入到C程序中
target_compile_definitions(cmake_examples_compile_flags 
    PRIVATE EX3
)
```
编译后可以看到：
```bash
Hello Compile Flags!
Hello Compile Flag EX2!
Hello Compile Flag EX3!
```
这里有两种方式，一种是直接使用`${CMAKE_CXX_FLAGS }`变量，C语言是`${CMAKE_C_FLAGS}`，在其中不断添加新的控制符，使用`-D`开头，类似于命令行传递值，这时候这个标志会传递到C++程序中，表示已经定义。第二种方式是使用`target_compile_definitions`函数传递标志。除此之外还可以在命令行中传递：
```bash
cmake .. -DCMAKE_CXX_FLAGS="-DEX2 -DEX2"
```
### set
[set](https://cmake.org/cmake/help/v3.21/command/set.html?highlight=set)可以设置很多种类型，像BOOL、String，filepath等等，使用CACHE和FORCE时注意：

```bash
SET(var1 13)
```

变量 var1 被设置成 13，如果 var1 在cache中已经存在，该命令不会overwrite cache中的值

```bash
SET(var1 13 ... CACHE ...)
```

如果cache存在该变量，使用cache中变量，如果cache中不存在，将该值写入cache

```bash
SET(var1 13 ... CACHE ... FORCE)
```

不论cache中是否存在，始终使用该值。

上面程序中，也可以使用布尔变量来控制C++程序中的宏，让我们先感受一下set和CACHE的妙用以及复杂的CMake程序架构：
```bash
.
├── build
├── CMakeLists.txt
├── config
│   ├── default_config.cmake
│   └── set_config.cmake
├── doc
│   └── README.adoc
└── src
    └── main.cpp
```
`.cmake`文件通常是CMake的配置文件，你可以认为这是CMake的头文件，定义变量和规则或者函数通常都可以放在这类文件中。它们可以通过include函数在CMakeLists.txt中引入，程序默认在`${CMAKE_MODULE_PATH}`下搜索此文件。作用域是从导入点到后面的CMake程序，导入点之前无法使用其中的值或者函数。先看一下CMakeLists.txt：
```bash
cmake_minimum_required(VERSION 3.5)

project (compile_flags)
add_executable(cmake_examples_compile_flags src/main.cpp)

include(config/set_config.cmake)
include(config/default_config.cmake)
```
比较简单，先为程序添加可执行文件的目标，然后在set_config中：
```bash
# if(EX2)
#     target_compile_definitions(cmake_examples_compile_flags 
#         PRIVATE EX2
#     )
# endif()

# if(EX3)
#     target_compile_definitions(cmake_examples_compile_flags 
#         PRIVATE EX3
#     )
# endif()

target_compile_definitions(cmake_examples_compile_flags
    PRIVATE
        $<$<BOOL:${EX2}>:EX2>
        $<$<BOOL:${EX3}>:EX3>
)
```
这里使用两种方法导入标志，第一种是if的写法，第二种是表达式写法，相对第二种简单但可读性稍差，尤其是当组合变量的表达式变得复杂时，需要格外小心。表达式采用NOT、AND、OR的组合，或者数值运算>，<，或=，以及STRING的equals等等。这个之后再补充介绍。

最后是引入`default_config.cmake`中默认的值，即命令行没有输入时，两个变量选择的值。注意如果之前CACHE中已经有设置这个值，比如ON，默认的值CACHE设置不会Over Write这个变量，除非使用FORCE。这是CACHE带来的不便,，但从另一方面来说，这也方便不需要重复运行cmake指令。
```bash
set(EX2 OFF CACHE BOOL "EX2")
set(EX3 OFF CACHE BOOL "EX3")
```
因为是布尔值，所以编译时参数应当设置为ON/OFF或者True/False。如果没有输入，则默认OFF/False。
```bash
cmake .. -DEX3=ON -DEX2=ON
make && ./cmake_examples_compile_flags
Hello Compile Flags!
Hello Compile Flag EX2!
Hello Compile Flag EX3!
```
注意如果此时继续cmake ..不输入任何变量，缓存中这两个仍然是打开的，除非删除build重新编译或者输入`-DEX3=OFF -DEX2=OFF`或者对缓存中的变量进行修改。在CMakeCache.txt中可以找到：
```bash
//EX2
EX2:BOOL=ON

//EX3
EX3:BOOL=ON
```
可以手动修改CACHE中的值，然后重新编译，比如这里设置EX2:BOOL=OFF，重新make后，EX2所控制的输出将被静默。

有关CMake set的CACHE set和Normal set还可以参考：[cmake cache变量](https://blog.csdn.net/weixin_39732534/article/details/110658282)
## 第三方库
我们的程序常常使用第三方库，例如现在我们有一个程序要使用Mbedtls的库和函数，一种方法是我们在系统中安装mbedtls的库和头文件，然后程序编译时链接，这其实和GCC使用默认的库是一回事；另一种方法我们下载源码，并将源码放在我们的工程当中，再进行编译。让我们来手动一步步尝试实现。
### 安装并使用
首先下载mbedtls并编译安装：
```bash
git clone https://github.com/ARMmbed/mbedtls.git
cd mbedtls && mkdir build && cd build
cmake .. && make -j32 && sudo make install
```
然后我们新建一个目录：
```bash
.
├── build
├── CMakeLists.txt
├── include
│   ├── mbedtls          # mbedtls头文件
│   ├── psa		         # psa头文件
│   └── mbedtls_config.h # 配置文件
└── src
    └── main.c
```
CMakeLists.txt文件：
```bash
cmake_minimum_required(VERSION 3.8)
project("hash")
add_executable(hash src/main.c)
target_include_directories(hash
    PUBLIC
        ${PROJECT_SOURCE_DIR}/include
)
target_link_libraries(hash 
    PUBLIC
        mbedtls
        mbedcrypto   # 直接使用libmbedcrypto.a库文件，在系统目录下
)
```
main.c是一个单向散列函数计算的源码：
```c
#include <stdio.h>
#include <string.h>
#include <stdint.h>

#include "mbedtls/md.h"
#include "mbedtls/platform.h"

static void dump_buf(char *info, uint8_t *buf, uint32_t len)
{
    mbedtls_printf("%s", info);
    for (int i = 0; i < len; i++) {
        mbedtls_printf("%s%02X%s", i % 16 == 0 ? "\n\t":" ", 
                        buf[i], i == len - 1 ? "\n":"");
    }
    mbedtls_printf("\n");
}

int main(void)
{
    uint8_t digest[32];
    char *msg = "abc";

    mbedtls_md_context_t ctx;
    const mbedtls_md_info_t *info;

    mbedtls_md_init(&ctx);
    info = mbedtls_md_info_from_type(MBEDTLS_MD_SHA256);

    mbedtls_md_setup(&ctx, info, 0);
    mbedtls_printf("\n  md info setup, name: %s, digest size: %d\n", 
                   mbedtls_md_get_name(info), mbedtls_md_get_size(info));

    mbedtls_md_starts(&ctx);
    mbedtls_md_update(&ctx, msg, strlen(msg));
    mbedtls_md_finish(&ctx, digest);

    dump_buf("\n  md sha-256 digest:", digest, sizeof(digest));

    mbedtls_md_free(&ctx); 

    return 0;
}
```
配置头文件mbedtls_config.h：
```c
/* System support */
#define MBEDTLS_PLATFORM_C
#define MBEDTLS_PLATFORM_MEMORY
#define MBEDTLS_MEMORY_BUFFER_ALLOC_C
#define MBEDTLS_PLATFORM_NO_STD_FUNCTIONS
#define MBEDTLS_PLATFORM_EXIT_ALT
#define MBEDTLS_NO_PLATFORM_ENTROPY
#define MBEDTLS_NO_DEFAULT_ENTROPY_SOURCES
#define MBEDTLS_PLATFORM_PRINTF_ALT

/* mbed TLS modules */
#define MBEDTLS_BIGNUM_C

#include "mbedtls/check_config.h"
```
这样编译运行：
```bash
./hash        

  md info setup, name: SHA256, digest size: 32

  md sha-256 digest:
        BA 78 16 BF 8F 01 CF EA 41 41 40 DE 5D AE 22 23
        B0 03 61 A3 96 17 7A 9C B4 10 FF 61 F2 00 15 AD
```
### 直接下载并使用
这里借用git工具远程下载并使用。参考：[git_repository](https://cmake.org/cmake/help/v3.21/module/ExternalProject.html?highlight=git_repository)
```bash
.
├── build
├── CMakeLists.txt
├── config
│   └── config.cmake
├── include
│   └── mbedtls_config.h
├── lib
│   └── CMakeLists.txt
└── src
    └── main.c
```
先看一下根目录的CMakeLists.txt：
```bash
cmake_minimum_required(VERSION 3.8)
project("hash")

# 导入MBEDTLS第三方库信息
include(config/config.cmake)

# 下载MBEDTLS库，路径保存在 MBEDCRYPTO_PATH 中
add_subdirectory(${PROJECT_SOURCE_DIR}/lib)

# 添加库的所有源码
aux_source_directory(${MBEDCRYPTO_PATH}/library MBEDTLS_SOURCES)

# 建立可执行文件目标
add_executable(hash
    ${CMAKE_CURRENT_LIST_DIR}/src/main.c
    ${MBEDTLS_SOURCES}
)

# 添加头文件
target_include_directories(hash
    PUBLIC
        ${PROJECT_SOURCE_DIR}/include
        ${MBEDCRYPTO_PATH}/include
        ${MBEDCRYPTO_PATH}/library
)
```
第一步导入config.cmake中关于Mbedtls相关的变量，指定了远程的位置和版本：
```bash
set(MBEDCRYPTO_PATH                     "DOWNLOAD"  CACHE PATH      "Path to Mbed Crypto (or DOWNLOAD to fetch automatically")
set(MBEDCRYPTO_VERSION                  "mbedtls-3.0.0" CACHE STRING "The version of Mbed Crypto to use")
set(MBEDCRYPTO_GIT_REMOTE               "https://github.com/ARMmbed/mbedtls.git" CACHE STRING "The URL (or path) to retrieve MbedTLS from.")
set(MBEDCRYPTO_BUILD_TYPE               "${CMAKE_BUILD_TYPE}" CACHE STRING "Build type of Mbed Crypto library")
set(TFM_MBEDCRYPTO_CONFIG_PATH          "${CMAKE_SOURCE_DIR}/include/mbedtls_config.h" CACHE PATH "Config to use for Mbed Crypto")
```
然后`lib/CMakeLists.txt`再根据输入和默认值判断是否要下载第三方库，借助[FetchContent](https://cmake.org/cmake/help/v3.21/module/FetchContent.html)工具来引入第三方GIT下载源码。
```bash
include(FetchContent)
set(FETCHCONTENT_QUIET FALSE)

if ("${MBEDCRYPTO_PATH}" STREQUAL "DOWNLOAD")
    find_package(Git)
    FetchContent_Declare(mbedcrypto
        GIT_REPOSITORY ${MBEDCRYPTO_GIT_REMOTE}
        GIT_TAG ${MBEDCRYPTO_VERSION}
        GIT_SHALLOW TRUE
        GIT_PROGRESS TRUE
        GIT_SUBMODULES ""
    )

    FetchContent_GetProperties(mbedcrypto)

    if(NOT mbedcrypto_POPULATED)
        FetchContent_Populate(mbedcrypto)
        set(MBEDCRYPTO_PATH ${mbedcrypto_SOURCE_DIR} CACHE PATH "Path to mbed-crypto (or DOWNLOAD to get automatically" FORCE)
    endif()
endif()
```
如果已经下载过mbedtls，可以在命令行手动输入`-DMBEDCRYPTO_PATH`，这样CMake将不会自动下载目录。编译过程：
```bash
cmake ..
-- The C compiler identification is GNU 9.3.0
-- The CXX compiler identification is GNU 9.3.0
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Check for working C compiler: /usr/bin/cc - skipped
-- Detecting C compile features
-- Detecting C compile features - done
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Check for working CXX compiler: /usr/bin/c++ - skipped
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Found Git: /usr/bin/git (found version "2.25.1") 
-- Populating mbedcrypto
-- Configuring done
-- Generating done
-- Build files have been written to: /home/sjl/learn/cmake-examples/mbedtls_test/build/_deps/mbedcrypto-subbuild
Scanning dependencies of target mbedcrypto-populate
[ 11%] Creating directories for 'mbedcrypto-populate'
[ 22%] Performing download step (git clone) for 'mbedcrypto-populate'
Cloning into 'mbedcrypto-src'...
remote: Enumerating objects: 20471, done.        
remote: Counting objects: 100% (20471/20471), done.        
remote: Compressing objects: 100% (9915/9915), done.        
remote: Total 20471 (delta 16054), reused 13968 (delta 10412), pack-reused 0        
Receiving objects: 100% (20471/20471), 20.12 MiB | 8.92 MiB/s, done.
Resolving deltas: 100% (16054/16054), done.
Note: switching to 'mbedtls-3.0.0'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by switching back to a branch.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -c with the switch command. Example:

  git switch -c <new-branch-name>

Or undo this operation with:

  git switch -

Turn off this advice by setting config variable advice.detachedHead to false

HEAD is now at 8df2f8e7 Merge pull request #842 from ARMmbed/mbedtls-3.0.0rc0-pr
[ 33%] Performing update step for 'mbedcrypto-populate'
[ 44%] No patch step for 'mbedcrypto-populate'
[ 55%] No configure step for 'mbedcrypto-populate'
[ 66%] No build step for 'mbedcrypto-populate'
[ 77%] No install step for 'mbedcrypto-populate'
[ 88%] No test step for 'mbedcrypto-populate'
[100%] Completed 'mbedcrypto-populate'
[100%] Built target mbedcrypto-populate
-- Configuring done
-- Generating done
-- Build files have been written to: /home/sjl/learn/cmake-examples/mbedtls_test/build
```
可以看出有下载和分支切换的过程。
## 标准C/C++
有时候我们需要指定一个编译标准以确保程序跨平台兼容，所以CMake需要能够检查并设置编译的标准。CMake提供`CHECK_CXX_COMPILER_FLAG`函数校验C++标准，然后同样可以在`${CMAKE_CXX_FLAGS}`中输入标准。
```bash
# Set the minimum version of CMake that can be used
# To find the cmake version run
# $ cmake --version
cmake_minimum_required(VERSION 2.8)

# Set the project name
project (hello_cpp11)

# try conditional compilation
include(CheckCXXCompilerFlag)
CHECK_CXX_COMPILER_FLAG("-std=c++11" COMPILER_SUPPORTS_CXX11)
CHECK_CXX_COMPILER_FLAG("-std=c++0x" COMPILER_SUPPORTS_CXX0X)

# check results and add flag
if(COMPILER_SUPPORTS_CXX11)#
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
elseif(COMPILER_SUPPORTS_CXX0X)#
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x")
else()
    message(STATUS "The compiler ${CMAKE_CXX_COMPILER} has no C++11 support. Please use a different C++ compiler.")
endif()

# Add an executable
add_executable(hello_cpp11 main.cpp)
```
上面写法比较安全，下面是简单使用C++11的写法：
```bash
# set the C++ standard to C++ 11
set(CMAKE_CXX_STANDARD 11)
```
也可以使用`target_compile_features`函数为目标选定编译标准，补充：可以使用`target_compile_options`为目标选的编译选项，如是否输出警告信息等。