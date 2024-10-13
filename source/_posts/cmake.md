---
title: cmake
date: 2024-04-05 07:06:41
tags:
banner_img: /images/cmake.png
index_img: /images/cmake.png
---

# 命令行
- `cmake -S . -B build`:构建build树
- `cmake --build`:build二进制
- `cmake --fresh -S <source tree> -B <build tree>`:确保重头开始cmake

|长选项|短选项|含义|
|:-:|:-:|:-:|
|`--source <dir>`|`-S <dir>`|指定源码路径|
||`-B <dir>`|build树路径，若不存在创建目录|
|`-build <dir>`||build二进制路径|
|`--parallel <number-of-jobs>` |`-j <number-of-jobs>`|多核运行|
||`-G <generator name>`|指定生成器，可以--help查看系统上支持的生成器|
||`-C <initial cache script>`|预填充缓存信息|
||`-D <var>[:<type>]=<value>`|指定变量值,type可选以下值：BOOL，FILEPATH，PATH，STRING 或 INTERNAL|
||`-L`|列出缓存变量，-D指定的不会打印, -LH则还会打印变量提供的帮助信息|
||`-U <globbing_expr>`|删除一个变量|
|`--system-information [file]`||获取关于变量、命令、宏和其他设置的通用信息,可保存在file中|
|`--log-level=<level>`||可以是ERROR，WARNING，NOTICE，STATUS，VERBOSE，DEBUG，TRACE，输出打印级别|
|`--trace`||它会打印每个执行的命令及其文件名、调用它的行号，以及传递的参数列表|
|`--list-presets`||列出所有可用的预设|
|`--fresh`||清理目录，等价于手动删除|
|`--target <target>`|`-t <target>`|指定生成的目标（这些目标通常排除在正常构建外）|
|`--clean-first`|`-t clean`|只影响目标产物而不影响其他，如缓存|
|`--config <cfg>`||指定构建类型，只有visual studio等多配置生成器可用，也可以用`CMAKE_BUILD_TYPE`指定|
|`--install <build tree> --install-prefix <prefix>`||指定安装路径，prefix是前缀路径|
|`--verbose`|`-v`|输出细节|
||`-E <command> [options]`|以平台无关的方式运行单个命令——例如复制文件或计算校验和|

# 内置变量
|变量名|类型|含义|
|:-:|:-:|
|`CMAKE_GENERATOR_TOOLSET`||
|`CMAKE_GENERATOR_PLATFORM`||
|`CMAKE_BUILD_TYPE`|可选值Debug,Release,MinSizeRel,RelWithDebInfo|
## 主机系统变量
|变量名|类型|含义|
|:-:|:-:|:-:|
|`CMAKE_SIZEOF_VOID_P`|Int|8代表64位，4代表32位|
|`CMAKE_<LANG>_BYTE_ORDER`||Lang可选C，CXX等,存放字节序,值有可能是`LITTLE_ENDIAN`和`BIG_ENDIAN`|
|`CMAKE_SYSTEM_NAME`|String|存放操作系统类型：`Linux,Darwin,Windows,AIX`|
|`ANDROID,APPLE,CYGWIN,UNIX,IOS,WIN32,WINCE,WINDOWS_PHONE`|Bool||
|`HOSTNAME`|String|主机名|
|`FQDN`|String| 完全限定域名|
|`TOTAL_VIRTUAL_MEMORY`|String| 以 MiB 为单位的虚拟内存总量|
|`AVAILABLE_VIRTUAL_MEMORY`|String| 以 MiB 为单位的可用虚拟内存|
|`TOTAL_PHYSICAL_MEMORY`|String| 以 MiB 为单位的总物理内存|
|`AVAILABLE_PHYSICAL_MEMORY`|String| 以 MiB 为单位的可用物理内存|
|`OS_NAME`|String| 如果存在，则输出 uname -s; 无论是 Windows、Linux，还是 Darwin|
|`OS_RELEASE`|| 操作系统子类型，如 Windows Professional|
|`OS_VERSION`|String| 操作系统构建 ID|
|`OS_PLATFORM`|String| 在 Windows 上和 $ENV{PROCESSOR_ARCHITECTURE} 的值一样. 在 Unix/macOS 上和 uname -m 一样|
|`NUMBER_OF_LOGICAL_CORES`|| 逻辑核数|
|`NUMBER_OF_PHYSICAL_CORES`|| 物理核数|
|`HAS_SERIAL_NUMBER`|| 如果处理器有序列号，则为 1|
|`PROCESSOR_SERIAL_NUMBER`|| 处理器序列号|
|`PROCESSOR_NAME`|| 可读的处理器名称|
|`PROCESSOR_DESCRIPTION`|| 可读的完整处理器描述|
|`IS_64BIT`|| 如果处理器是 64 位的为 1|
|`HAS_FPU`|| 如果处理器有浮点单元为 1|
|`HAS_MMX`|| 如果处理器支持 MMX 指令为 1|
|`HAS_MMX_PLUS`|| 如果处理器支持 Ext. MMX 指令为 1|
|`HAS_SSE`|| 如果处理器支持 SSE 指令为 1|
|`HAS_SSE2`|| 如果处理器支持 SSE2 指令为 1|
|`HAS_SSE_FP`|| 如果处理器支持 SSE FP 指令为 1|
|`HAS_SSE_MMX`|| 如果处理器支持 SSE MMX 指令为 1|
|`HAS_AMD_3DNOW`|| 如果处理器支持 3DNow 指令为 1|
|`HAS_AMD_3DNOW_PLUS`|| 如果处理器支持 3DNow+ 指令为 1|
|`HAS_IA64`|| 如果 IA64 处理器模拟 x86，则为 1|

# 常见操作
## C++标准
- `set(CMAKE_CXX_STANDARD <version>)`
- `set_property(Target <target> PROPERTY CXX_STANDARD <version>)`
- `set_target_properties(<targets> PROPERTIES CXX_STANDARD <version>)`
- `target_compile_features(<target> PUBLIC cxx_std_26)`
version可选值有：98,11,14,17,20,23,26
使用target_compile_features可选值有cxx_std_14...
强制应用标准：`set(CMAKE_CXX_STANDARD_REQUIRED ON)`
## 检查支持的编译特性
```cmake
list(FIND CMAKE_CXX_COMPILE_FEATURES cxx_variable_templates result)
if(result EQUAL -1)
    message(FATAL_ERROR "Variable templates are required for compilation.")
endif()
```
可以在[c++支持特性列表里找到特性的完整列表](https://cmake.org/cmake/help/latest/prop_gbl/CMAKE_CXX_KNOWN_FEATURES.html)
## 过程间优化(ipo)
```cmake
include(CheckIPOSupported)
check_ipo_supported(RESULT ipo_supported)
set(CMAKE_INTERPROCEDURAL_OPTIMIZATION ${ipo_supported})
```
## 禁用源内构建
```cmake
cmake_minimum_required(VERSION 3.26.0)
project(NoInSource CXX)
if(PROJECT_SOURCE_DIR STREQUAL PROJECT_BINARY_DIR)
    message(FATAL_ERROR "In-source builds are not allowed")
endif()
message("Build successful!")
```
## 生成目标依赖图
- 使用命令`cmake --graphviz=test.dot .`
- 更多信息见官网**CMakeGraphVizOptions**模块,
- 默认自定义目标不会出现在图中，可以创建一个**CMakeGraphVizOptions.cmake**文件，里面设置`set(GRAPHVIZ_CUSTOM_TARGETS TRUE)`
- 生成的dot文件可以在线查看[Graphviz](https://dreampuf.github.io/GraphvizOnline/)
# 语法命令
## cmake_minimum-required
- 格式：`cmake_minimum_required(VERSION <x.xx>)`
- 意义：设置cmake期望的版本
## project
- 格式一：`project(<PROJECT-NAME> [<language-name>...])`
- 格式二：
```cmake
project(<PROJECT-NAME>
    [VERSION <major>[.<minor>[.<patch>[.<tweak>]]]]
    [DESCRIPTION <project-description-string>]
    [HOMEPAGE_URL <url-string>]
    [LANGUAGES <language-name>...])
```
此命令将设置以下变量:
- `PROJECT_NAME`
- `CMAKE_PROJECT_NAME (only in the top-level CMakeLists.txt)`
- `PROJECT_IS_TOP_LEVEL, <PROJECT-NAME>_IS_TOP_LEVEL`
- `PROJECT_SOURCE_DIR, <PROJECT-NAME>_SOURCE_DIR`
- `PROJECT_BINARY_DIR, <PROJECT-NAME>_BINARY_DIR`
支持语言(仅列常用的)
- C:C
- CXX: C++
指定VERSION关键字，将会设置以下变量:
- `PROJECT_VERSION, <PROJECT-NAME>_VERSION`
- `CMAKE_PROJECT_VERSION (only in the top-level CMakeLists.txt)`
- `PROJECT_VERSION_MAJOR, <PROJECT-NAME>_VERSION_MAJOR`
- `PROJECT_VERSION_MINOR, <PROJECT-NAME>_VERSION_MINOR`
- `PROJECT_VERSION_PATCH, <PROJECT-NAME>_VERSION_PATCH`
- `PROJECT_VERSION_TWEAK, <PROJECT-NAME>_VERSION_TWEAK`
类似设置DESCRIPTION和HOMEPAGE_URL将设置以下变量
- `PROJECT_DESCRIPTION, <PROJECT-NAME>_DESCRIPTION`
- `PROJECT_HOMEPAGE_URL, <PROJECT-NAME>_HOMEPAGE_URL`
## add_subdirectory
- 格式：`add_subdirectory(source_dir [binary_dir] [EXCLUDE_FROM_ALL])`
- 意义：`将计算source_dir 路径（相对于当前目录）并解析其中的CMakeLists.txt 文件`
- **[binary_dir]**: 构建的文件将写入该路径，默认是构建树
- **[EXCLUDE_FROM_ALL]**: 禁用子目录中定义的目标的自动构建
## add_executable
- 格式：`add_executable(<name> [WIN32] [MACOSX_BUNDLE] [EXCLUDE_FROM_ALL] [source1 source2...])`
- [WIN32],[MACOSX_BUNDLE]分别生成win和mac下的gui程序
- [EXCLUDE_FROM_ALL]将使得该目标在默认构建中排除在外，必须-t明确指明
## add_library
- 格式：`add_library(<name> [STATIC|SHARED|MODULE] [EXCLUDE_FROM_ALL] [source1 source2...])`
- [STATIC|SHARED|MODULE],分别对应静态，动态，模块
## add_custom_target
- 格式：`add_custom_target(Name [ALL] [COMMAND command2 [args2...] ...])`
[ALL]与[EXCLUDE_FROM_ALL]含义相反，自定义目标默认不生成,自定义目标通常用于以下场景：
- 计算其他二进制文件的校验和
- 运行代码消毒器并收集结果
- 将编译报告发送到指标通道
```cmake
cmake_minimum_required(VERSION 3.26)
project(BankApp CXX)
add_executable(terminal_app terminal_app.cpp)
add_executable(gui_app gui_app.cpp)
target_link_libraries(terminal_app calculations)
target_link_libraries(gui_app calculations drawing)
add_library(calculations calculations.cpp)
add_library(drawing drawing.cpp)
add_custom_target(checksum ALL
    COMMAND sh -c "cksum terminal_app>terminal.ck"
    COMMAND sh -c "cksum gui_app>gui.ck"
    BYPRODUCTS terminal.ck gui.ck
    COMMENT "Checking the sums..."
)
```
## 伪目标
### 别名目标
别名目标的确切作用就是你所期望的——为目标创建另一个不同的名称引用
- `add_executable(<name> ALIAS <target>)`
- `add_library(<name> ALIAS <target>)`
### 接口库
- `add_library(<name> INTERFACE [item1 ...])`
有两个作用：一是**代表仅包含头文件的库**,二是**将一堆传播属性打包成一个逻辑单元**
例子一：
```cmake
add_library(Eigen INTERFACE
  src/eigen.h src/vector.h src/matrix.h
)
target_include_directories(Eigen INTERFACE
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src>
  $<INSTALL_INTERFACE:include/Eigen>
)
target_link_libraries(executable Eigen)
```
例子二：
```cmake
add_library(warning_properties INTERFACE)
target_compile_options(warning_properties INTERFACE
    -Wall -Wextra -Wpedantic
)
5 target_link_libraries(executable warning_properties)
```
## 对象库
即.o对象文件
- `add_library(<target> OBJECT <sources>)`
可以使用target_link_libraries()作为依赖添加，抑或是如下：
```cmake
add_library(... $<TARGET_OBJECTS:objname> ...)
add_executable(... $<TARGET_OBJECTS:objname> ...)
```
## target_include_directories
```cmake
target_include_directories(<target> [SYSTEM] [AFTER|BEFORE]
<INTERFACE|PUBLIC|PRIVATE> [item1...]
[<INTERFACE|PUBLIC|PRIVATE> [item2...]
...])
```
SYSTEM 关键字告诉编译器给定的目录应该视为标准系统目录（与尖括号形式一起使用）。
BEFORE或AFTER决定是否将这些头文件放在已有的路径之前或之后
## target——compile_definitions
- ``
定义宏变量,相当于-D传递
# 目标的属性
- 获取属性值:`get_target_property(<var> <target> <property-name>)`
- 设置属性:`set_target_properties(<target1> <target2> ...  PROPERTIES <prop1-name> <value1> <prop2-name> <value2> ...)`或 `set_property(Target <target> PROPERTY <prop-name> <value>)`
## 属性传播
- PRIVATE:设置源目标属性
- INTERFACE:设置使用目标的目标属性
- PUBLIC：设置源目标和使用目标属性
当指定 PRIVATE 或 PUBLIC 关键字时，CMake 将在目 标的属性中存储提供的值，COMPILE_DEFINITIONS。此外，关键字是 INTERFACE 或 PUBLIC， 将在具有 INTERFACE_前缀的属性中存储值——INTERFACE_COMPILE_DEFINITIONS。配置阶 段，CMake 将读取源目标的接口属性，并将其内容附加到目标目标。就这样传播属性，或 CMake 所说的传递目标的使用要求。
设置属性时需要指定上述关键字，如`target_compile_definitions(<source> <INTERFACE|PUBLIC|PRIVATE> [items1...])`, `target_link_libraries(<target> <PRIVATE|PUBLIC|INTERFACE> <item1> [<PRIVATE|PUBLIC|INTERFACE> <item>...])`这个命令也需要传播关键字
## 自定义属性的传播
CMake 默认不会传播自定义属性（这个机制只适用 于内置目标属性），必须明确地将自定义属性添加到“兼容”属性列表中。
每个目标都有四个这样的列表：
- COMPATIBLE_INTEERFACE_BOOL
- COMPATIBLE_INTERFACE_STRING
- COMPATIBLE_INTERFACE_NUMBER_MAX
- COMPATIBLE_INTERFACE_NUMBER_MIN
将属性添加到它们中的任何一个，都会触发传播和兼容性检查。BOOL 列表将检查所有传递到 目标目标的属性是否评估为相同的布尔值。类似地，STRING 将评估为字符串。NUMBER_MAX 和 NUMBER_MIN 略有不同——传递的值不必匹配，但目标目标将只接收最高或最低值。
### 例子
```cmake
cmake_minimum_required(VERSION 3.26)
 project(PropagatedProperties CXX)

add_library(source1 empty.cpp)
set_property(TARGET source1 PROPERTY INTERFACE_LIB_VERSION 4)
set_property(TARGET source1 APPEND PROPERTY
    COMPATIBLE_INTERFACE_STRING LIB_VERSION)

add_library(source2 empty.cpp)
set_property(TARGET source2 PROPERTY INTERFACE_LIB_VERSION 4)
add_library(destination empty.cpp)
target_link_libraries(destination source1 source2)
```
CMake 将这个自定义属性传播到相应目标，并检查所有源目标的版本是否完全匹配（兼容性属性只需在目标目标上设置一次）。

# 参考连接
[19 reasons why cmake is actually awesome](https://kubasejdak.com/19-reasons-why-cmake-is-actually-awesome)
[an introduction to modern cmake](https://cliutils.gitlab.io/modern-cmake/)
[effective modern cmake](https://gist.github.com/mbinna/c61dbb39bca0e4fb7d1f73b0d66a4fd1)
[awesome cmake](https://github.com/onqtam/awesome-cmake)
[useful variables](https://gitlab.kitware.com/cmake/community/wikis/doc/cmake/Useful-Variables)
# CTest
testing tool(integrated in CMake)
