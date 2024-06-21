参考自官方教程
[https://cmake.org/cmake/help/latest/guide/tutorial/index.html](https://cmake.org/cmake/help/latest/guide/tutorial/index.html).   
基于cmake 3.26.3

# 目录

1. [最基础的工程](#basic)    
2. [指定c++版本](#version)     
3. [传递宏](#marco)     
4. [自建库的方法](#self-lib)     
5. [为自建库添加开关](#self-lib-switch)     
6. [更好的链接库的方法](#better-self-lib)     
7. [安装](#install)     
8. [传递宏](#macro)

<a name="basic"></a>
## 最基础的工程 
目录结构如下：
```
.
├── 1.cpp
└── CMakeLists.txt
```
其中1.cpp:
```
#include <iostream>

int main()
{
	std::cout<<"hello cmake"<<std::endl;
	return 0;
}
```
CMakeLists.txt:
```
cmake_minimum_required(VERSION 3.12)
project(basic)
add_executable(${PROJECT_NAME} 1.cpp)
```
编译的两种方法：  
方法1
```
mkdir build
cd build
cmake ..
make
```
方法2
```
mkdir build
cmake -S . -B build #-S指定源码路径，-B指定构建路径
cmake --build build
```
此时目录变为：
```
.
├── 1.cpp
├── CMakeLists.txt
└── build
    ├── CMakeCache.txt
    ├── CMakeFiles
    ├── Makefile
    ├── basic
    └── cmake_install.cmake
```
执行：  
```
[~/code/cmake_learn/basic]$ ./build/basic      
hello cmake
```

<a name="version"></a>
## 指定c++版本
方法1:
```
set(CMAKE_CXX_STANDARD 20) #指定默认使用c++20
set(CMAKE_CXX_STANDARD_REQUIRED ON) #不满足c++20时报错
```
可以指定的c++版本可以通过[CXX_STANDARD](https://cmake.org/cmake/help/latest/prop_tgt/CXX_STANDARD.html#prop_tgt:CXX_STANDARD)查看。  

方法2:
```
#注意要写在add_executable之后
target_compile_feature(${PROJECT_NAME} PUBLIC cxx_std_20)
```
所有支持的feature可以通过[CMAKE_KNOWN_FEATURES](https://cmake.org/cmake/help/latest/prop_gbl/CMAKE_CXX_KNOWN_FEATURES.html#prop_gbl:CMAKE_CXX_KNOWN_FEATURES)查看。此外还有CMAKE_C_KNOWN_FEATURES，CMAKE_CUDA_KNOWN_FEATURES等。

<a name="marco"></a>
## 传递宏
通过configure_file可以将CMakeLists.txt中定义的参数通过形如`@VAR@`的宏的形式传入源文件，其流程如下：
![configure_file](/static/img/configure_file.png)  
首先在工程目录中添加config.h.in：
```
.
├── 1.cpp
├── CMakeLists.txt
├── build
└── config.h.in
```
1.传递版本号  
project中添加版本号：
```
project(basic VERSION 1.2) #此时名为basic_VERSION_MAJOR和basic_VERSION_MINOR的两个变量会被内部自动创建
```
2.添加自定义的变量：
```
set(VAR1 "this is var1")
```
3.修改config.h.in文件为：
```
#define version_major @basic_VERSION_MAJOR@
#define version_minor @basic_VERSION_MINOR@
#define var1 "@VAR1@"
```
4.关联到cmake
```
#cmake将修改config.h.in，并将修改后的结果复制到config.h
configure_file(config.h.in config.h) 
#由于生成的config.h在构建目录(本例中即build目录)里，因此需要将改目录包含进来
#注意要写到add_executable之后
target_include_directories(${PROJECT_NAME} PUBLIC "${PROJECT_BINARY_DIR}") 
```
5.完整的CMakeLists.txt如下：
```
cmake_minimum_required(VERSION 3.12)
project(basic VERSION 1.2)

set(VAR1 "this is var1")
configure_file(config.h.in config.h)

add_executable(${PROJECT_NAME} 1.cpp)

target_compile_features(${PROJECT_NAME} PUBLIC cxx_std_20)
target_include_directories(${PROJECT_NAME} PUBLIC "${PROJECT_BINARY_DIR}")
```
6.修改1.cpp为：
```
#include <iostream>
#include "config.h"

int main()
{
	std::cout<<version_major<<","<<version_minor<<std::endl;
	std::cout<<var1<<std::endl;
	return 0;
}
```
7.编译：
```
cmake -S . -B build
cmake --build build
```
之后build目录中便会出现config.h文件：
```
#define version_major 1
#define version_minor 2
#define var1 "this is var1"
```
可见cmake将定义的变量传入了config.h文件，而1.cpp中include了config.h，因此执行后结果如下：
```
[~/code/cmake_learn/basic/build]$ ./basic
1,2
this is var1
```

<a name="self-lib"></a>
## 自建库的方法
根目录下创建strlib目录，中间放入strlib.h，str.cpp，CMakeLists.txt文件：
```bash
.
├── 1.cpp
├── CMakeLists.txt
├── build
│   ├── CMakeCache.txt
│   ├── CMakeFiles
│   ├── Makefile
│   ├── basic
│   ├── cmake_install.cmake
│   ├── config.h
│   ├── mathlib
│   └── strlib
├── config.h.in
└── strlib
    ├── CMakeLists.txt
    ├── strlib.cpp
    └── strlib.h
```
/strlib/CMakeLists.txt：
```
add_library(strlib strlib.cpp)
```
/strlib/strlib.h：
```
#pragma once
#include <iostream>

namespace strlib{
	std::string hello();
}
```
/strlib/strlib.cpp
```
#include "strlib.h"

namespace strlib{
	std::string hello(){
		return "hello";
	}
}
```
至此完成了名为strlib的库的准备，然后在/CMakelists.txt中添加这个库：
```
add_subdirectory(strlib) #添加库
#add_executable....
target_include_directories(
	${PROJECT_NAME} PUBLIC 
	"${PROJECT_BINARY_DIR}" 
	"${PROJECT_SOURCE_DIR}/strlib" #指向库里面的头文件
)
target_link_libraries(
	${PROJECT_NAME} PUBLIC 
	strlib #链接库
)
```
之后就可以在1.cpp中使用strlib了：
```
#include <iostream>
#include "config.h"
#include "strlib.h"

int main()
{
	std::cout<<version_major<<","<<version_minor<<std::endl;
	std::cout<<var1<<std::endl;
	std::cout<<strlib::hello()<<std::endl;
	return 0;
}
```
编译后执行输出：
```
[~/code/cmake_learn/basic/build]$ ./basic 
1,2
this is var1
hello
```

<a name="self-lib-switch"></a>
## 为自建库添加开关
也就是通过option指令来在cmake时通过参数来选择是否将库包含入源文件:
```
option(USE_STRLIB "if use strlib" ON) #添加USE_STRLIB选项作为开关
```
之后将库的include和link方式修改为根据list添加，然后根据USE_STRLIB的ON与否来向list中添加内容：
```
if(USE_STRLIB)
	add_subdirectory(strlib)
	list(APPEND extra_include "${PROJECT_SOURCE_DIR}/strlib")
	list(APPEND extra_lib strlib)
endif()
target_include_directories(${PROJECT_NAME} PUBLIC ${extra_include})
target_link_libraries(${PROJECT_NAME} PUBLIC ${extra_lib})
```
当USE_STRLIB为ON时，strlib被添加进两个list，最终加入target。
在源文件中我们还需要知道USE_STRLIB是否被设置成了ON，这时就需要之前的configure_file指令了。在config.h.in中添加：
```
#cmakedefine USE_STRLIB
```
当USE_STRLIB=ON时，生成的config.h会被添加```#define USE_STRLIB```；  
当USE_STRLIB=OFF时，生成的config.h会被添加```/* #udef USE_STRLIB*/```，   
于是源文件include了config.h之后就可以通过判断USE_STRLIB是否被定义来决定是否使用strlib：
```
#include <iostream>
#include "config.h"
#ifdef USE_STRLIB
	#include "strlib.h"
#endif

int main()
{
	std::cout<<version_major<<","<<version_minor<<std::endl;
	std::cout<<var1<<std::endl;
#ifdef USE_STRLIB
	std::cout<<strlib::hello()<<"(lib)"<<std::endl;
#else
	std::cout<<"hello(plain)"<<std::endl;
#endif
	return 0;
}
```
完整的CMakeLists.txt如下：
```
cmake_minimum_required(VERSION 3.12)
project(basic VERSION 1.2)

#--- 设置config.h.in的指令注意要写在configure_file前面 ---#
set(VAR1 "this is var1")
option(USE_STRLIB "if use the strlib" ON)
#-----------------------------------------------------#
configure_file(config.h.in config.h)

add_executable(${PROJECT_NAME} 1.cpp)

if(USE_STRLIB)
	add_subdirectory(strlib)
	list(APPEND extra_include "${PROJECT_SOURCE_DIR}/strlib")
	list(APPEND extra_lib strlib)
endif()

target_compile_features(${PROJECT_NAME} PUBLIC cxx_std_20)
target_include_directories(
	${PROJECT_NAME} PUBLIC 
	"${PROJECT_BINARY_DIR}" 
	${extra_include}
)
target_link_libraries(
	${PROJECT_NAME} PUBLIC 
	${extra_lib}
)
```
之后在编译时就可以通过传入-DUSE_STRLIB来开关strlib，  
不使用时：
```
[~/code/cmake_learn/basic/build]$ cmake .. -DUSE_STRLIB=OFF;make;./basic
...
1,2
this is var1
hello(plain)
```
使用时：
```
[~/code/cmake_learn/basic/build]$ cmake .. -DUSE_STRLIB=ON;make;./basic
...
1,2
this is var1
hello(lib)
```

<a name="better-self-lib"></a>
## 更好的链接库的方法
在strlib的CMakeLists.txt里include需要的头文件而不在basic的CMakeLists.txt里include。  
strlib的CMakeLists.txt：
```
add_library(strlib strlib.cpp)
target_include_directories(strlib INTERFACE "${CMAKE_CURRENT_SOURCE_DIR}")
```
这样一来basic的CMakeLists.txt中就可以不用再引用strlib的头文件了：
```
cmake_minimum_required(VERSION 3.12)
project(basic VERSION 1.3)

set(VAR1 "this is var1")
option(USE_STRLIB "if use the strlib" ON)
configure_file(config.h.in config.h)

add_executable(${PROJECT_NAME} 1.cpp)

if(USE_STRLIB)
	add_subdirectory(strlib)
	list(APPEND extra_lib strlib)
endif()

target_compile_features(${PROJECT_NAME} PUBLIC cxx_std_20)
target_include_directories(
	${PROJECT_NAME} PUBLIC 
	"${PROJECT_BINARY_DIR}" 
)
target_link_libraries(
	${PROJECT_NAME} PUBLIC 
	${extra_lib}
)
```

<a name="install"></a>
## 安装
有时我们希望可以直接用指令的方式调用程序，在cmake中通过install就可以做到。
在这个例子中打算用[date](https://github.com/HowardHinnant/date)库做一个简单的查询本地时间的程序。

1.工程目录下ctime.cpp和cmakelist文件，并创建timelib目录，cd进去克隆date库：
```
mkdir timelib
cd timelib
git clone https://github.com/HowardHinnant/date.git
```
在内部创建timelib.cpp，timelib.h以及cmakelist.txt文件。  
之后项目的结构如下：
```bash
.
├── CMakeLists.txt
├── ctime.cpp
└── timelib
    ├── CMakeLists.txt
    ├── date
    ├── timelib.cpp
    └── timelib.h
```
按照用timelib封装date的思路，timelib/CMakeLists.txt如下：
```
add_library(timelib timelib.cpp date/src/tz.cpp)

find_package(CURL REQUIRED)

add_subdirectory(date)

target_compile_features(timelib PRIVATE cxx_std_14)

target_include_directories(timelib 
    INTERFACE 
    "${CMAKE_CURRENT_SOURCE_DIR}"
    PRIVATE
    "${CMAKE_CURRENT_SOURCE_DIR}/date/include"
)

target_link_libraries(timelib PRIVATE CURL::libcurl)
```
**说明：**

+ 根据date的[安装文档](https://howardhinnant.github.io/date/tz.html#Installation)需要用到curl，所以cmakelist中链接了curl库：
```
find_package(CURL REQUIRED)
...
target_link_libraries(timelib PRIVATE CURL::libcurl)
```

+ macOS上推荐c++14：
```
target_compile_features(timelib PRIVATE cxx_std_14)
```

+ timelib自身的头文件目录需要暴露给工程目录下的源文件，因此是INTERFACE；自身的源文件和date的源文件需要使用date下的include目录，因此是PRIVATE：
```
target_include_directories(timelib 
    INTERFACE 
    "${CMAKE_CURRENT_SOURCE_DIR}"
    PRIVATE
    "${CMAKE_CURRENT_SOURCE_DIR}/date/include"
)
```
+ 当然为了省事直接一个PUBLIC也是没问题的：
```
target_include_directories(timelib 
    PUBLIC 
    "${CMAKE_CURRENT_SOURCE_DIR}"
    "${CMAKE_CURRENT_SOURCE_DIR}/date/include"
)
```

2.工程目录下的cmakelist.txt如下：
```
cmake_minimum_required(VERSION 3.5)
project(ctime)

add_subdirectory(timelib)

add_executable(ctime ctime.cpp)

target_link_libraries(ctime PUBLIC timelib)

install(TARGETS ctime DESTINATION bin)
```
最后一行的install指令指定将生成的build目录下的ctime可执行文件安装到bin目录下(默认为/usr/local/bin)。

3.具体实现如下    
ctime.cpp：
```
#include "timelib.h"
#include <iostream>

int main()
{
	std::cout<<timelib::get_current_time()<<std::endl;
	return 0;
}
```
timelib.h：
```
#pragma once
#include <iostream>

namespace timelib
{
	std::string get_current_time();
}

```
timelib.cpp：
```
#include "date/tz.h"
#include <iostream>

namespace timelib
{
	std::string get_current_time()
	{
		using namespace date;
		using namespace std::chrono;
		auto local = make_zoned(current_zone(), system_clock::now());		
		return format("%Y-%m-%d %H:%M:%S %Z (%A)", local);
	}
}

```
4.编译安装：
```
mkdir build
cd build
cmake ..
make
```
a)默认的安装路径是/usr/local/bin需要root权限：
```
sudo cmake --install .
#或者sudo make install
```
b)如果没有root权限可以安装到用户目录：
```
#如果没有则会在$HOME下自动创建bin目录
cmake --install . --prefix "$HOME"
```
这种情况下需要在～/.zshrc(或～/.bashrc)中将$HOME/bin加入path，添加：
```
export PATH="$PATH:$HOME/bin"
```
之后source ～/.zshrc(或～/.bashrc)

5.安装好之后就可以用指令的方式调用程序了：
```
~/code/cmake_learn/ctime]$ ctime   
2023-04-15 20:21:27.076781 JST (Saturday)
```
具体可以查看[github仓库](https://github.com/jooooow/ctime)。

<a name="macro"></a>
## 传递宏
add_defintions命令为源文件的编译添加-D指令，也就是添加宏。
通过定义cmake变量来控制add_defintions的执行，从而可以实现宏的开关/传递：
```
option(DEFINE_TEST_TREE "test tree" OFF)
if(DEFINE_TEST_TREE)
    add_definitions(-DTEST_TREE) #cmake官方更推荐下面的指令
    #add_compile_definitions(TEST_TREE)
endif()
```
设置DEFINE_TEST_TREE=ON即可在源文件中产生一个TEST_TREE的宏定义
```
cmake .. -DDEFINE_TEST_TREE=ON
make
```
```
#ifdef TEST_TREE
printf("test tree\n");
#endif
```
需要注意的是：

+ 如果存在subdirectory，那么开关需要写在subdirectory前面以保证传递
+ cmake指令中设置的DEFINE_TEST_TREE会被cache，所以最好根据需要显式的设置。
+ 设置结果可以通过COMPILE_DEFINITIONS查看：
```
get_directory_property( DirDefs COMPILE_DEFINITIONS)
message( "COMPILE_DEFINITIONS = ${DirDefs}" )
```