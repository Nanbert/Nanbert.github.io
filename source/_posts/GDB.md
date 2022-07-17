---
title: GDB
date: 2020-01-16 11:47:33
subtitle:
categories:
tags:
index_img: /img/gdb.png
banner_img: /img/gdb.png
---

1.编译文件时要加上-g选项
　几种启动gdb方式:
　`gdb <program>`,`gdb <program> core`(core是程序非法执行后core dump后产生的文件),`gdb <program> <PID>`(可以指定这个服务程序运行时的进程ID)
2.`break`  
　a.加函数名,则会在函数内第一个非简单赋值语句处
　b.加行号,则会在该行号处停止
3.`step(s)`:进入函数
4.`next`:不进入函数
5.`print /<f> <expr>`:<f>为格式
　a.加表达式:其中$1、$2、....表示第几个print的表达式的值,$$n则表示倒数n+1的命令
　b.变量=表达式:赋值
　c.静态数组直接加数组名,动态数组格式为`*array@len`,@左边是array数组的首地址,右边则是数组的长度
　d.格式表
|符号|意义|
|:-:|:-:|
|x(a)|十六进制格式|
|d|十进制格式|
|u|十进制无符号整型|
|o|八进制格式|
|t|二进制格式|
|c|字符格式|
|f|浮点数格式|
|s|字符串格式|
|i|指令格式|
6.`display /<fmt> <expr>`:在使用display命令时,每次中断,挂起都会显示表达式的值,<fmt>指定格式可选有s和i
　a.`info display`:查看display设置的自动显示信息
　b.`enable/disable display <dnums>`:失效或恢复某个自动显示
　c.`undisplay <dnums> or delete display <dnums>`:删除某个自动显示,支持1-5这样的范围表示或者以空格分割不同号码
7.`run`
8.`finish`:结束执行当前函数,显示其返回值
9.`set`:设置变量新值
10.`continue(cont)`:后面可以加数字,表示忽略几个断点
11.`condition <断点号>　<条件表达式>`:条件为真时,执行断点
12.`tbreak`:临时断点等价于`break xx;enable delete <断点号>`
13.`enable <断点编号>`:恢复暂时失效的断点
14.`disable <断点编号>`:使断点失效
15.`delete <断点的编号或表达式>`:清除断点或者表达式
16.`clear <要清除的断点所在的行号>`:与delete不同的是给出行号,并且gdb会给出提示,delete则不会
17.`watch <条件表达式>`:在表达式为真时中断程序的运行
18.`info line <行号>or<函数名>or<文件名:行号>or<文件名:函数名>`:显示所指定源代码运行时的内存地址
19.`disassemble <函数名>`:该函数的机器指令(汇编码)
20.查看栈信息  
　a.`bt <n>或<-n>`:打印栈顶n层或栈底n层信息,不加n则表示打印当前所有函数栈的信息
　b.`frame(f) <n>`:frame 0表示栈顶,依次类推,不加n则表示输出当前层
　c.`up <n>或down <n>`:向栈底移动n层或向栈顶移动一层,栈底处于高地址区域,栈顶处于低地址区域
　d.`info frame(f)`:显示当前层更为详细的信息
　e.`info args`:显示当前函数的参数名及值
　f.`info locals`:显示当前函数所有局部变量及值
　g.`info catch`:显示当前函数中的异常处理信息
21.显示源代码
　a.`list <linenum>`:显示第linenum行的周围的源程序
　b.`list <function>`:显示function函数的源程序
　c.`list`:显示当前行后面的源程序
　d.`list -`:显示当前行前面的源程序
　e.`list <first>,<last>`:first行到last行之间的源程序
　f.`list ,<last>`:当前行到last行之间的源程序
22.搜索源代码
　a.`search <regexp>`:正向搜索
　b.`reverse-search <regexp>`:反向搜索
23.指定源文件路径
　a.`directory(dir) <dirname1:dirname2>`:添加路径到当前路径下
　b.`direcory`:清除所有自定义源文件搜索路径
　c.`show directories`:显示已定义的搜索路径
24.`examine(x)/<n/f/u> <addr>`　
　n、f、u是可选参数
　n:是一个正整数,表示一个显示内存的长度,也就是说从当前地址向后显示几个地址的内容
　f:表示显示的格式
　u:表示往后请求的字节数,默认是4bytes,b表示单字节,h表示双字节,w表示4字节,g表示8字节
　`x/3uh 0x54320`表示从内存地址0x54320读取内容,h表示以双字节为一个单位,3表示3个单位,u表示以十进制无符号整型显示
25.设置显示选项:`show\set <某个选项> (状态)`
　a.`set print address <on/off>`:系统默认打开,显示函数参数地址
　b.`set print array <on/off>`:系统默认关闭,显示数组元素是否占一行
　c.`set print elements <number of elements>`:显示数组最大显示长度,默认为0表示不做限制
　d.`set print null-stop <on/off>`:默认为off,表示字符串时，遇到结束符是否停止显示
　e.`set print pretty <on/off>`:为on时,结构体每个元素占一行
　f.`set print sevenbit-strings <on/off>`:为on时,字符显示ascll码
　g.`set print union <on/off>`:为on时,显示结构体中联合体数据
　h.`set print statci-members <on/off>`:是否显示c++对象中静态数据成员
　i.`set print object <on/off>`:是否按虚方法显示c++中的对象
　j.`set print vtbl <on/off>`:按规整的格式显示虚函数表
　k.`info frame`:查看当前函数语言
　l.`info source`:查看当前文件语言
　m.`show language`:查看当前语言环境
　n.`set language <language>`:设置语言环境
26.`set`:可以用set设置gdb的环境变量,如:`set $i = 0`,为了不与环境变量冲突,设置程序中的值时最好用`set var xx=xx`
27.寄存器情况
　a.`info registers`:查看除浮点寄存器外的所有寄存器
　b.`info all-registers`:查看所有寄存器
　c.`info registers <regname>`:查看指定寄存器
28.跳转
　a.`jump <linespec or address>`:可以是文件的行号、也可以是file:line、也可以是＋num偏移量、也可以是内存地址
29.产生信号量
　`signal <1-15>`:在断点处设置1-15的任意信号
30.强制函数返回
　`return (<expression>)`:忽略当前函数未执行语句,直接返回表达式的值
31.强制调用函数
　`cal <expr>`:调用某函数
32.`ptype`:显示某个量的类型
33.`until`:执行某个循环体直到结束
