---
title: gcc编译器及编译原理
date: 2020-12-19 17:25:41
subtitle:
categories:
tags:
index_img: /img/gcc.jpg
banner_img: /img/gcc.jpg
---
**1.基本概念**
**预处理:**`g++ -E main.cpp -o main.ii`,-E表示只进行预处理。预处理主要是处理各种宏的展开;添加行号和文件标识符,为编译器产生调试信息提供便利;删除注释;保留编译器用到的编译器指令等
**编译:**`g++ -S main.ii -o main.s`,-S表示只编译。编译是在预处理的基础上经过一系列词法分析、语法分析及优化后生成汇编代码。
**汇编:**`g++ -c main.s -o main.o`,汇编将汇编代码转换为机器可执行的指令
**链接:**`g++ main.o`,链接生成可执行程序,之所以需要链接是因为我们的代码不可能像main.cpp这么简单,现代软件动则成百上千万行,如果写在一个main.cpp既不利于分工合作,也无法维护,因此通常是由一堆cpp文件组成,编译器分别编译每个cpp,这些cpp里会引用别的模块中的函数或全局变量,在编译单个cpp的时候是没法知道它们的准确地址,因此在编译结束后,需要连接器将各种还没有准确地址的符号(函数、变量等)设置为正确的值,这样组装在一起就可以形成一个完整的可执行程序
**2.选项**
`-l<library>` 链接动态库
`-L<dir>` 动态库搜索目录
`-D<expression>` 宏定义命令中定义
`-l <dir>`头文件搜索目录
**3.静态库编译和使用**
`gcc -c increase.c -o increase.o`把.c编译成.o
`ar -r libincrease.a increase.o`归档成静态库.a
`gcc main.c -L -static -o main`链接成可执行文件
**4.动态库编译和使用**
`gcc -shared -fPIC -o libinc.so increase.c`-fPIC生成位置独立的代码,此类代码可以在不同进程间共享
`gcc -lincrease -o main main.c`链接动态库
有错误
