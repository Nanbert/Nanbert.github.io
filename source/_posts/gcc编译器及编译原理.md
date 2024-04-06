---
title: gcc编译器及编译原理
date: 2020-12-19 17:25:41
subtitle:
categories:
tags:
index_img: /images/gcc.jpg
banner_img: /images/gcc.jpg
---
# 基本概念
**预处理:**`g++ -E main.cpp -o main.ii`,-E表示只进行预处理。预处理主要是处理各种宏的展开;添加行号和文件标识符,为编译器产生调试信息提供便利;删除注释;保留编译器用到的编译器指令等
**编译:**`g++ -S main.ii -o main.s`,-S表示只编译。编译是在预处理的基础上经过一系列词法分析、语法分析及优化后生成汇编代码。
**汇编:**`g++ -c main.s -o main.o`,汇编将汇编代码转换为机器可执行的指令
**链接:**`g++ main.o`,链接生成可执行程序,之所以需要链接是因为我们的代码不可能像main.cpp这么简单,现代软件动则成百上千万行,如果写在一个main.cpp既不利于分工合作,也无法维护,因此通常是由一堆cpp文件组成,编译器分别编译每个cpp,这些cpp里会引用别的模块中的函数或全局变量,在编译单个cpp的时候是没法知道它们的准确地址,因此在编译结束后,需要连接器将各种还没有准确地址的符号(函数、变量等)设置为正确的值,这样组装在一起就可以形成一个完整的可执行程序
# 选项
- `-l<library>` 链接动态库
- `-L<dir>` 动态库搜索目录
- `-D<expression>` 宏定义命令中定义
- `-I <dir>`头文件搜索目录
- `-isystem`specify the include path for system headers path
# 静态库编译和使用
- `gcc -c increase.c -o increase.o`把.c编译成.o
- `ar -r libincrease.a increase.o`归档成静态库.a
- `gcc main.c -L -static -o main`链接成可执行文件
- 环境变量：`LIBRARY_PATH`Specify the directories where search for static libraries .a at compile-time
# 调试
`g++ -O0 -g [-g3] <program.cpp> -o program
gdb [--args] ./program <args...>`
- -O0 Disable any code optimization for helping the debugger. It is implicit for most compilers
- -g Enable debugging
    - stores the symbol table information in the executable (mapping between assembly and source code lines)
    - for some compilers, it may disable certain optimizations
    - slow down the compilation phase and the execution
- -g3 Produces enhanced debugging information, e.g. macro definitions. Available for most compilers. Suggested instead of -g 
# 动态库编译和使用
- `gcc -shared -fPIC -o libinc.so increase.c`-fPIC生成位置独立的代码,此类代码可以在不同进程间共享
- `gcc -lincrease -o main main.c`链接动态库
```bash
g++ source1.c -c source1.o -fPIC
g++ source2.c -c source2.o -fPIC
g++ source1.o source2.o -shared -o libmydynamiclib.so
```
- 环境变量：`LD_LIBRARY_PATH`Specify the directories where search for dynamic/shared libraries .dll at run-time
# 内存检查
## -fsanitize
基本比Valgrind工具更好
### address
memory error detector,Similar to valgrind but faster (50X slowdown)
• heap/stack/global out-of-bounds
• memory leaks
• use-after-free, use-after-return, use-after-scope
• double-free, invalid free
• initialization order bugs
`clang++ -O1 -g -fsanitize=address -fno-omit-frame-pointer <program>`
### leak
a run-time memory leak detector
`clang++ -O1 -g -fsanitize=leak -fno-omit-frame-pointer <program>`
### memory
is detector of uninitialized reads
`clang++ -O1 -g -fsanitize=memory -fno-omit-frame-pointer <program>`
### undefined
a undefined behavior detector
- signed integer overflow, floating-point types overflow, enumerated not in range
- out-of-bounds array indexing, misaligned address
- divide by zero
- etc.
`clang++ -O1 -g -fsanitize=undefined -fno-omit-frame-pointer <program>`
### integer
Checks for undefined or suspicious integer behavior (e.g. unsigned integer overflow)
### nullability
Checks passing null as a function parameter, assigning null to an lvalue, and returning null from a function
# Warn
- `Wall`:Enables many standard warnings (∼50 warnings)
- `Wextra`:Enables some extra warning flags that are not enabled by -Wall (∼15 warnings)
- `Wpedantic`:Issue all the warnings demanded by strict ISO C/C++

# 栈有关选项
- `-Wstack-usage=<byte-size>` Warn if the stack usage of a function might exceed byte-size. The computation done to determine the stack usage is conservative (no VLA)
- `fstack-usage` Makes the compiler output stack usage information for the
program, on a per-function basis
- `-Wvla` Warn if a variable-length array is used in the code
- `-Wvla-larger-than=<byte-size>` Warn for declarations of variable-length arrays whose size is either unbounded, or bounded by an argument that allows the array size to exceed byte-size bytes
## _FORTIFY_SOURCE
Adding FORTIFY SOURCE define, the compiler provides buffer overflow checks for the
following functions:
memcpy , mempcpy , memmove , memset , strcpy , stpcpy , strncpy , strcat ,
strncat , sprintf , vsprintf , snprintf , vsnprintf , gets .
```C++
# include <cstring> // std::memset
# include <string> // std::stoi
int main(int argc, char** argv) {
int size = std::stoi(argv[1]);
char buffer[24];
std::memset(buffer, 0xFF, size);
}
```
```bash
$ gcc -O1 -D FORTIFY SOURCE program.cpp -o program
$ ./program 12 # OK
$ ./program 32 # Wrong
$ *** buffer overflow detected ***: ./program terminated
```
