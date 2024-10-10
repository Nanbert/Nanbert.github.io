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
|变量名|含义|
|:-:|:-:|
|`CMAKE_GENERATOR_TOOLSET`||
|`CMAKE_GENERATOR_PLATFORM`||
|`CMAKE_BUILD_TYPE`|可选值Debug,Release,MinSizeRel,RelWithDebInfo|

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

# 参考连接
[19 reasons why cmake is actually awesome](https://kubasejdak.com/19-reasons-why-cmake-is-actually-awesome)
[an introduction to modern cmake](https://cliutils.gitlab.io/modern-cmake/)
[effective modern cmake](https://gist.github.com/mbinna/c61dbb39bca0e4fb7d1f73b0d66a4fd1)
[awesome cmake](https://github.com/onqtam/awesome-cmake)
[useful variables](https://gitlab.kitware.com/cmake/community/wikis/doc/cmake/Useful-Variables)
# CTest
testing tool(integrated in CMake)
