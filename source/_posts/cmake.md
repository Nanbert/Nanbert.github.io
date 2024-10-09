---
title: cmake
date: 2024-04-05 07:06:41
tags:
banner_img: /images/cmake.png
index_img: /images/cmake.png
---

# 命令
- `cmake -S . -B build`:
- `cmake --fresh -S <source tree> -B <build tree>`:确保重头开始cmake

|长选项|短选项|含义|
|:-:|:-:|:-:|
|`--source <dir>`|`-S <dir>`|指定源码路径|
|`--build <dir>`|`-B <dir>`|build路径，若不存在创建目录|
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

# 内置变量
|变量名|含义|
|:-:|:-:|
|`CMAKE_GENERATOR_TOOLSET`||
|`CMAKE_GENERATOR_PLATFORM`||
|`CMAKE_BUILD_TYPE`|可选值Debug,Release,MinSizeRel,RelWithDebInfo|


[19 reasons why cmake is actually awesome](https://kubasejdak.com/19-reasons-why-cmake-is-actually-awesome)
[an introduction to modern cmake](https://cliutils.gitlab.io/modern-cmake/)
[effective modern cmake](https://gist.github.com/mbinna/c61dbb39bca0e4fb7d1f73b0d66a4fd1)
[awesome cmake](https://github.com/onqtam/awesome-cmake)
[useful variables](https://gitlab.kitware.com/cmake/community/wikis/doc/cmake/Useful-Variables)
# CTest
testing tool(integrated in CMake)
