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
## 输出信息
```bash
/opt/app/todeav1/test$ldd test
libstdc++.so.6 => /usr/lib64/libstdc++.so.6 (0x00000039a7e00000)
libm.so.6 => /lib64/libm.so.6 (0x0000003996400000)
libgcc_s.so.1 => /lib64/libgcc_s.so.1 (0x00000039a5600000)
libc.so.6 => /lib64/libc.so.6 (0x0000003995800000)
/lib64/ld-linux-x86-64.so.2 (0x0000003995400000)
```
- 第一列：程序需要依赖什么库
- 第二列：系统提供的与程序需要的库所对应的库
- 第三列：库加载的开始地址
# nm
The nm utility provides information on the symbols being used in an object file or executable file
## 参数
- -a或–debug-syms：显示所有的符号，包括debugger-only symbols。
- -B：等同于–format=bsd，用来兼容MIPS的nm。
- -C或–demangle：将低级符号名解析(demangle)成用户级名字。这样可以使得C++函数名具有可读性。
- –no-demangle：默认的选项，不需要将低级符号名解析成用户级名。
- -D或–dynamic：显示动态符号。该任选项仅对于动态目标(例如特定类型的共享库)有意义。
- -f format：使用format格式输出。format可以选取bsd、sysv或posix，该选项在GNU的nm中有用。默认为bsd。
- -g或–extern-only：仅显示外部符号。
- -n、-v或–numeric-sort：按符号对应地址的顺序排序，而非按符号名的字符顺序。
- -p或–no-sort：按目标文件中遇到的符号顺序显示，不排序。
- -P或–portability：使用POSIX.2标准输出格式代替默认的输出格式。等同于使用任选项-f posix。
- -s或–print-armap：当列出库中成员的符号时，包含索引。索引的内容包含：哪些模块包含哪些名字的映射。
- -r或–reverse-sort：反转排序的顺序(例如，升序变为降序)。
- –size-sort：按大小排列符号顺序。该大小是按照一个符号的值与它下一个符号的值进行计算的。
- –target=bfdname：指定一个目标代码的格式，而非使用系统的默认格式。
- -u或–undefined-only：仅显示没有定义的符号(那些外部符号)。
- –defined-only:仅显示定义的符号。
- -l或–line-numbers：对每个符号，使用调试信息来试图找到文件名和行号。
- -V或–version：显示nm的版本号。
- –help：显示nm的选项。
## 符号说明
对于每一个符号来说，其类型如果是小写的，则表明该符号是local的；大写则表明该符号是global(external)的
- A 该符号的值是绝对的，在以后的链接过程中，不允许进行改变。这样的符号值，常常出现在中断向量表中，例如用符号来表示各个中断向量函数在中断向量表中的位置。
- B 该符号的值出现在非初始化数据段(bss)中。例如，在一个文件中定义全局static int test。则该符号test的类型为b，位于bss section中。其值表示该符号在bss段中的偏移。一般而言，bss段分配于RAM中。
- C 该符号为common。common symbol是未初始话数据段。该符号没有包含于一个普通section中。只有在链接过程中才进行分配。符号的值表示该符号需要的字节数。例如在一个c文件中，定义int test，并且该符号在别的地方会被引用，则该符号类型即为C。否则其类型为B。
- D 该符号位于初始化数据段中。一般来说，分配到data section中。
- G 该符号也位于初始化数据段中。主要用于small object提高访问small data object的一种方式。
- I 该符号是对另一个符号的间接引用。
- N 该符号是一个debugging符号。
- R 该符号位于只读数据区。
  - 例如定义全局const int test[] = {123, 123};则test就是一个只读数据区的符号。
  - 值得注意的是，如果在一个函数中定义`const char *test = “abc”, const char test_int = 3`。使用nm都不会得到符号信息，但是字符串”abc”分配于只读存储器中，test在rodata section中，大小为4。
- S 符号位于非初始化数据区，用于small object。
- T 该符号位于代码区text section。
- U 该符号在当前文件中是未定义的，即该符号的定义在别的文件中。 例如，当前文件调用另一个文件中定义的函数，在这个被调用的函数在当前就是未定义的；但是在定义它的文件中类型是T。但是对于全局变量来说，在定义它的文件中，其符号类型为C，在使用它的文件中，其类型为U。
- V 该符号是一个weak object。
- W The symbol is a weak symbol that has not been specifically tagged as a weak object symbol.
- ? 该符号类型没有定义
## example
- `nm -uCA *.o|grep foo`:等价于`objdump -t; readelf -s`
- `nm -e a.out`:对象文件的静态和外部符
- `nm -xv a.out`:以十六进制显示符号大小和值并且按值排序符号
- `nm -X64 /usr/lib/libc.a`:显示 libc.a 中所有 64 位对象符号，忽略所有 32 位对象

# size
查看程序被映射到内存中的映像所占用的大小信息。
## 各个段
程序映射到内存中，从低地址到高地址依次为下列段:
- 代码段： 只读，可共享; 代码段（code segment/text segment ）通常是指用来存放程序执行代码的一块内存区域。这部分区域的大小在程序运行前就已经确定，并且内存区域通常属于只读, 某些架构也允许代码段为可写，即允许修改程序。在代码段中，也有可能包含一些只读的常数变量，例如字符串常量等。
- 数据段： 储存已被初始化了的静态数据。数据段（data segment ）通常是指用来存放程序中已初始化的全局变量的一块内存区域。数据段属于静态内存分配。
- BSS 段：未初始化的数据段. BSS 段（bss segment ）通常是指用来存放程序中未初始化的全局变量的一块内存区域。BSS 是英文Block Started by Symbol 的简称。BSS 段属于静态内存分配。
- 堆（heap ）： 堆是用于存放进程运行中被动态分配的内存段，它的大小并不固定，可动态扩张或缩减。当进程调用malloc 等函数分配内存时，新分配的内存就被动态添加到堆上（堆被扩张）；当利用free 等函数释放内存时，被释放的内存从堆中被剔除（堆被缩减）
- 栈(stack) ：栈又称堆栈，是用户存放程序临时创建的局部变量，也就是说我们函数括弧“{} ”中定义的变量（但不包括static 声明的变量，static 意味着在数据段中存放变量）。除此以外，在函数被调用时，其参数也会被压入发起调用的进程栈中，并且待到调用结束后，函数的返回值也会被存放回栈中。由于栈的先进先出特点，所以栈特别方便用来保存/ 恢复调用现场。从这个意义上讲，我们可以把堆栈看成一个寄存、交换临时数据的内存区。
# readelf
- `readelf`:displays information about ELF format object files
这个工具和objdump命令提供的功能类似，但是它显示的信息更为具体，并且它不依赖BFD库
## elf文件种类
- 可重定位的对象文件：由汇编器生成的.o文件
- 可执行对象文件
- 动态库文件
## 参数

|短选项|长选项|comment|
|:-:|:-:|:-:|
|-a|–all|全部,Equivalent to: -h -l -S -s -r -d -V -A -I|
|-h|–file-header|文件头Display the ELF file header|
|-l|–program-headers/-segments|程序Display the program headers|
|-S|–section-headers/--sections|段头Display the sections’ header|
|-e|–headers|全部头Equivalent to: -h -l -S|
|-s|–syms/--symbols|符号表 Display the symbol table|
|-n|–notes|内核注释 Display the core notes (if present)|
|-r|–relocs|重定位 Display the relocations (if present)|
|-u|–unwind|Display the unwind info (if present)|
|-d|–dynamic|动态段 Display the dynamic segment (if present)|
|-V|–version-info|版本 Display the version sections (if present)|
|-A|–arch-specific|CPU构架 Display architecture specific information (if any).|
|-D|–use-dynamic|动态段 Use the dynamic section info when displaying symbols|
|-x|–hex-dump=<number>|显示 段内内容Dump the contents of section <number>|
|-I|–histogram|Display histogram of bucket list lengths|
|-W|–wide|宽行输出 Allow output width to exceed 80 characters|
|-H|–help|Display this information|
|-v|–version|Display the version number of readelf|

## 常见输出信息
- .text section 里装载了可执行代码；
- .data section 里面装载了被初始化的数据；
- .bss section 里面装载了未被初始化的数据；
- 以 .rec 打头的 sections 里面装载了重定位条目；
- .symtab 或者 .dynsym section 里面装载了符号信息；
- .strtab 或者 .dynstr section 里面装载了字符串信息；
more:![](http://www.cnblogs.com/xmphoenix/archive/2011/10/23/2221879.html)
## example
- `readelf -h main| grep Machine`:查看程序的可运行的架构平台
- `readelf -S main|grep debug`:编译时是否使用了-g选项
# objdump
- `objdump`:displays information about object files
## 参数
-f 显示文件头信息
-D 反汇编所有section (-d反汇编特定section)
-h 显示目标文件各个section的头部摘要信息
-x 显示所有可用的头信息，包括符号表、重定位入口。-x 等价于 -a -f -h -r -t 同时指定。
-i 显示对于 -b 或者 -m 选项可用的架构和目标格式列表。
-r 显示文件的重定位入口。如果和-d或者-D一起使用，重定位部分以反汇编后的格式显示出来。
-R 显示文件的动态重定位入口，仅仅对于动态目标文件有意义，比如某些共享库。
-S 尽可能反汇编出源代码，尤其当编译的时候指定了-g这种调试参数时，效果比较明显。隐含了-d参数。
-t 显示文件的符号表入口。类似于nm -s提供的信息
## example
- `objdump -i`:查看本机目标结构(大小端)
- `objdump -d main.o`:反汇编程序
- `objdump -t main.o`:显示符号表入口
- below:
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
# pstack
pstack是一个脚本工具，可显示每个进程的栈跟踪。pstack命令必须由相应进程的属主或root运行
`pstrack <program-pid>`
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


