---
title: c++_tools
date: 2024-04-05 04:05:08
tags:
---

# valgrind
## 描述
用来检测内存和线程bug
## 用法
`valgrind <-options> ./program <args>`,program should be compiled with -g
## 选项
- `--leak-check=full` print details for each “definitely lost” or “possibly lost”
block, including where it was allocated
- `--show-leak-kinds=all` to combine with --leak-check=full. Print all leak kinds
- `--track-fds=yes` list open file descriptors on exit (not closed)
- `--track-origins=yes` tracks the origin of uninitialized values (very slow execution)
# c++filt
- `c++filt`可以把mangle的字符串demangle
# ldd
- `ldd` shows the shared objects (shared libraries) required by a program or other shared objects
# nm
- `nm`The nm utility provides information on the symbols being used in an object file or executable file
# readelf
- `readelf`:displays information about ELF format object files
```bash
$ readelf --symbols something.so | c++filt
... OBJECT LOCAL DEFAULT 17 __frame_dummy_init_array_
... FILE LOCAL DEFAULT ABS prog.cpp
... OBJECT LOCAL DEFAULT 14 CC1
... OBJECT LOCAL DEFAULT 14 CC2
... FUNC LOCAL DEFAULT 12 g()
# --symbols: display symbol table
```
# objdump
- `objdump`:displays information about object files
```bash
$ objdump -t -C something.so | c++filt
... df *ABS* ... prog.cpp
... O .rodata ... CC1
... O .rodata ... CC2
... F .text ... g()
... O .rodata ... (anonymous namespace)::CC3
... O .rodata ... (anonymous namespace)::CC4
... F .text ... (anonymous namespace)::h()
... F .text ... (anonymous namespace)::B::j1()
... F .text ... (anonymous namespace)::B::j2()
# --t: display symbols
# -C: Decode low-level symbol names
```
# cppcheck
# gcovr(有bug)
覆盖率工具，需要配合g++的`--coverage`选项
```bash
$ gcc -g --coverage program.cpp -o program
$ ./program 9
first
$ gcovr -r --html --html-details <path> # generate html
```
# clang-tidy
代码风格及静态检查
# doxygen
生成帮助文档，配置文件Doxyfile
- `doxygen -g`
- comment the code with `///` or `/** comment *`
- generate doxygen base configuration file
## 注释关键字
- `@file` Document a file
- `@brief` Brief description for an entity
- `@param` Run-time parameter description
- `@tparam` Template parameter description
- `@return` Return value description
## 配置信息
```bash
HAVE DOT = YES
GRAPHICAL HIERARCHY = YES
CALL GRAPH = YES
CALLER GRAPH = YES
```
## example
```C++
/**
* @file
* @copyright MyProject
* license BSD3, Apache, MIT, etc.
* @author MySelf
* @version v3.14159265359
* @date March, 2018
*/
/// @brief Namespace brief description
namespace my_namespace {
/// @brief "Class brief description"
/// @tparam R "Class template for"
template<typename R>
class A {
```
```C++
/**
* @brief "What the function does?"
* @details "Some additional details",
* Latex/MathJax: $\sqrt a$
* @tparam T Type of input and output
* @param[in] input Input array
* @param[out] output Output array
* @return `true` if correct,
* `false` otherwise
* @remark it is *useful* if ...
* @warning the behavior is **undefined** if
* @p input is `nullptr`
* @see related_function
*/
template<typename T>
bool my_function(const T* input, T* output);
/// @brief
void related_function();
```
# cloc
行数统计信息
# lizard
圈复杂度分析工具
`lizard my_project/`
## example
```bash
$lizard my_project/
==============================================================
NLOC CCN token param function@line@file
--------------------------------------------------------------
10 2 29 2 start_new_player@26@./html_game.c
6 1 3 0 set_shutdown_flag@449@./httpd.c
24 3 61 1 server_main@454@./httpd.c
--------------------------------------------------------------
```
- CCN:圈复杂度
- NLOC：无注释的代码行数
- token：number of conditional statements
# clang-format
# compiler explorer
编码实时二进制查看
# cppinsights
编译器实时查看
# AI补全工具
- CoPilot
- TabNine
- Kite
# 代码在线搜索
- [searchcode](https://searchcode.com/)
- [grep.app](https://grep.app/)
# 在线代码段性能比较
- [benchmark](http://quick-bench.com/)


