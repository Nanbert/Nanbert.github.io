---
title: Make
date: 2020-01-17 12:45:15
subtitle:
categories:
tags:
index_img:
---

1.make中遇到的第一条规则是最终目标  
2.变量相关  
　a.内置变量

|变量类型|特殊变量|含义|用例或说明|
|:-:|:-:|:-:|:-:|
|特殊变量|`VPATH`|寻找依赖的路径|`VPATH = src:../headers`|
|特殊变量|.DIRSEPSTR|路径分隔符一般是斜杠||
|特殊变量|.MAKEDIR|调用make的绝对路径名||
|特殊变量|.NULL|空字符串||
|特殊变量|.OS|正在运行的操作系统名称||
|特殊变量|.PWD|运行时活动工作目录的绝对路径名||
|特殊变量|.SHELL|启动的shell类型||
|特殊变量|`AR`|函数库打包程序,默认命令是ar||
|特殊变量|`CC`|C语言编译程序,默认命令是cc||
|特殊变量|`RM`|删除文件命令,默认命令是rm -f||
|特殊变量|`ARFLAGS`|函数库打包程序AR命令参数,默认值是rv||
|特殊变量|`CFLAGS`|C语言编译器参数,默认为空||
|特殊变量|`CXXLAGS`|C++语言编译器参数,默认为空||
|特殊变量|`CPPLAGS`|C预处理器参数,默认为空||
|特殊变量|`CXX`|C++语言编译程序,默认命令是g++||
|特殊变量|`CPP`|C程序的预处理器,默认命令是$(CC) -E||
|自动变量|`$@`|目标集|它代表一个量,遍历目标集,一般与依赖集相匹配|
|自动变量|`$%`|目标集|仅当目标是函数库文件时,表示规则中的目标成员名,foo.a(bar.o),$%就是bar.o,$@就是foo.a|
|自动变量|`$<`|依赖集|它代表一个量,遍历目标集,一般用于目标集相匹配|
|自动变量|`$?`|依赖集|所有比目标新的依赖目标的集合,以空格分隔|
|自动变量|`$^`|依赖集|所有依赖目标的集合,去掉重复以空格分隔|
|自动变量|`$+`|依赖集|所有依赖目标的集合,不去掉重复以空格分隔|
|自动变量|`$*`||对应模式的'%'及之前的部分,包括路径|

所有自动变量都可以与`D`,`F`搭配使用,表示匹配的目录部分和文件部分,如`$(@D)`
　b.`:=`操作符,操作符右边只能出现已定义的变量,如果是未定义的变量,则会自动忽略 
　`=`操作符右边可以出现未定义的变量
　`?=`如果变量没有定义过,则使用后面的值
　`+=`追加变量值
　c.变量定义要注意空格
　d.替换`.o`到`.c`
　　方法一:
```bash
foo:=a.o b.o c.o
bar:=$(foo:.o=.c)
```
　　方法二:
```bash
foo:=a.o b.o c.o
bar:=$(foo:%.o=%.c)
```
　e.`override <variable>=<value>`:强制覆盖,make命令行参数可以用这个强制覆盖,否则覆盖不了
　f.替换变量中字符串
　`变量名: s/原字符串/新字符串`
　g.加前后缀
　`变量:^ "前缀"`
　`变量:+ "后缀"`
　h.取部分
　`$(VARIABLE:<option>)`,option有3个选项d(仅取路径)、b(文件名,不包括扩展)、f(文件名,包括扩展)
3.关键字vpath
　用法一: vpath <pattern> <directories>;符合模式的在指定文件夹搜索
　用法二: vpath <pattern>;清除对应模式搜索目录
　用法三: vpath;清除所有搜索目录
***注意:都必须包含%,意思是包含一个以上的匹配字符***
　f.局部作用的变量:
```bash
prog: CFLAGS = -g
	prog: prog.o foo.o bar.o
		$(gcc) $(CFLAGS) prog.o foo.o bar.o
	prog.o: prog.c
		$(gcc) $(CFLAGS) prog.c
	foo.o: foo.c
		$(gcc) $(CFLAGS) bar.c
```
不管全局的$(CFLAGS)的值是什么,prog目标及其所引发的所有规则中(prog.o foo.o bar.o),$(CFLAGS)的值都是-gl
4.伪目标:`.PHONY`
　-伪目标并不是文件，只是个标签
　-只有显式地指名才能使其生效
　-最终目标可以是伪目标,一个用法如下,使一个make文件生成多个目标:
```bash
all: prog1 prog2 prog3
.PHONY: all
prog1: prog1.o utils.o
	gcc -o prog1 prog1.o utils.o
prog2: prog2.o
	gcc -o prog2 prog2.o
prog3: prog3.o sort.o utils.o
	gcc -o prog3 prog3.o sort.o utils.o
```
5.静态模式:更方便定义多目标`<targets>: <target-pattern>: <prereq-patterns>`
例子:
```bash
objects = foo.o bar.o
all: $(objects)
$(objects): %.o: %.c
	$(gcc) -c $(CFLAGS) $< -o $@
```
等价于
```bash
foo.o: foo.c
	$(gcc) -c $(CFLAGS) foo.c -o foo.o
bar.o: bar.c
	$(gcc) -c $(CFLAGS) bar.c -o bar.o
```
6.gcc -MM选项可以为.c源文件自动生成依赖
7.make默认会显示命令,在命令前加@可以阻止输出
8.要想前一个命令影响后一个命令要以‘;’分隔,不能分行写,
9.在命令前加上"-",则命令都会被认为成功执行,如`-mkdir dir`,这样可以避免目录已存在的错误而终止规则
10.嵌套make  
```bash
subsystem:
	cd subdir && $(MAKE) #也可以这么写$(MAKE) -C subdir
```
　要想传递变量给嵌套的make,使用export,不想传递使用unexport
　其中`SHELL`和`MAKEFLAGS`总会影响下层make,但-C、-f、-h、-o、-W几个参数并不往下传递
11.定义命令包(即命令集)
例子:
```bash
define run-yacc
	mkdir dir
	mv dir newdir
endef
crap: 
	$(run-yacc)
```
12.条件判断
```bash
<conditional-directive> #可以是ifeq(arg1,arg2)、ifneq、ifdef <variable-name>、ifndef
	<text-if-true>
else
	<text-if-false>
endif
```
13.函数
　a.调用语法:`$(<function> <arguments>)`,参数间用','分隔
　b.字符串函数

|函数|功能|返回|
|:-:|:-:|:-:|
|`$(subst <from>,<to>,<text>)`|把字符串<text>中的<from>换成<to>|`被替换过的字符串`|
|`$(patsubst <pattern>,<replacement>,<text>)`|查找<text>中的单词符合<pattern>,替换<replacement>|被替换后的字符串|
|`$(strip <string>)`|去掉<string>中开头和结尾的空字符|去掉空字符的字符串|
|`$(findstring <find>,<in>)`|在字符串<in>中查找<find>|如果找到,返回<find>,否则返回空字符串|
|`$(filter <pattern...>,<text>)`|以<pattern>模式过滤<text>字符串中的字符串,可以有多个模式,以空格分割|返回符合<pattern>的字符串|
|`$(filter-out <pattern...>,<text>)`|以<pattern>模式过滤<text>字符串中的字符串,可以有多个模式,以空格分割|返回不符合<pattern>的字符串|
|`$(sort <list>)`|给字符串<list>中的单词升序|返回排序后的字符串(会去掉相同的单词)|
|`$(word <n>,<text>)`|取字符串<text>中的第<n>个单词|返回该单词,如果n过大,则返回空字符串|
|`$(wordlist <n>,<m>,<text>)`|取第<n>-第<m>个单词|返回那些单词|
|`$(words <text>)`|统计<text>中的单词数|返回个数|

　c.文件名函数

|格式|例子|返回值|
|:-:|:-:|:-:|
|`$(dir <names...>)`|`$(dir src/foo.c hacks)`|`src/ ./`|
|`$(notdir <names...>)`|`$(notdir src/foo.c hacks)`|`foo.c hacks`|
|`$(suffix <names...>)`|`$(suffix src/foo.c src-1.0/bar.c hacks)`|`.c .c`|
|`$(basename <names...>)`|`$(basename src/foo.c src-1.0/bar.c hacks)`|`src/foo src-1.0/bar hacks`|
|`$(addsuffix <suffix>,<names...>)`|`$(addsuffix .c,foo bar)`|`foo.c bar.c`|
|`$(addprefix <prefix>,<names...>)`|`$(addprefix src/,foo bar)`|`src/foo src/bar`|
|`$(join <list1>,<list2>)`|`$(join aaa bbb,111 222 333)`|`aaa111 bbb222`|

　d.foreach函数
```bash
names:= a b c d
	files:= $(foreach n,$(names),$(n).o)
```
　$(files)的值是‘a.o b.o c.o d.o’
　e.if函数
　`$(if <condition>,<then-part>,<else-part>)`,<condition>若返回为非空,则执行<then-part>,其是整个函数的返回值
　f.call函数
`$(call <expression>,<parm1>,<parm2>,<parm3>...)`;<expression>中$(1),$(2)等,会被参数<parm1>、<parm2>等代替
```bash
reverse=$(2) $(1)
foo=$(call reverse,a,b)
```
此时foo的值就是'b a'
  g.shell函数
  `files:=$(shell echo *.c)`
  h.origin函数
  `$(origin <variable>)`:告知变量来源情况

|返回值|含义|
|:-:|:-:|
|undefined|未定义|
|default|默认定义|
|environment|环境变量|
|file|定义在make文件中|
|override|被override重新定义|
|automatic|命令运行中的自动化变量|
|command line|命令行定义|

 14.一些常用伪目标命名

|名称|含义|
|:-:|:-:|
|all|这个伪目标一般是所有目标的目标,一般为编译所有的目标|
|clean|这个伪目标的功能一般是删除所有make创建的文件|
|install|安装已编译好的程序,其实是把目标执行文件复制到指定文件夹|
|print|这个伪目标的功能是列出改变过的源文件|
|tar|这个伪目标的功能是打包备份源程序|
|dist|一般是把打包文件进行压缩|
|tags|这个伪目标的功能用于更新所有的目标,以备完整地重新编译|
|check、test|一般用来测试makefile文件流程|

15.make选项参数

|短选项|长选项|含义|
|:-:|:-:|:-:|
|-n|--just-print、--dry-run、--recon|不管目标更不更新,只打印命令,不执行|
|-t|--touch|把目标文件时间更新,但不更改目标文件,假装编译文件|
|-q|--question|寻找目标,如果目标存在,什么也不输出,也不执行编译.如果目标不存在,打印一条出错信息|
|-B|--always-make|认为所有目标都需要更新(重编译)|
|-W <file>|--what-if=<file>;--assume-new=<file>;--new-file=<file>|make会根据规则推导来运行依赖于这个文件的命令|
|-C||指定makefile文件目录|
|-e|--environment-overrides|指定环境变量值,覆盖makefile文件中定义的变量值|
|-i|--ignore-errors|在执行时忽略所有的错误|
|-I <dir>|--include-dir=<dir>|指定一个包含makefile文件的搜索目标|
|-k|--keep-going|出错也不停止,目标失败,依赖于其上的就不会执行|
|-o <file>|--old-file=<file>;--assume-old=<file>|不生成指定的<file>,即使这个目标的依赖文件比他新|
|-l <load>|--load-average [=<load>];--max-load[=<load>]|指定make运行命令的负载|
|-p|--print-database|输出makefile文件中所有数据,包括所有的规则和变量|
|-r|--no-builtin-rules|禁止使用任何隐式规则|
|-R|--no-builtin-variables|禁止使用任作用于变量上的何隐式规则|
|-s|--silent;--quiet|命令运行时不显示命令的输出|
|-S|--no-keep-going;--stop|取消-k选项的作用|
|-w|--print-directory|输出运行makefile文件之前之后的信息,跟踪嵌套make时很有用|
|长|--no-print-directory|禁止-w选项|
|长|--warn-undefined-variables|警告未定义的变量|

--debug <options>,options可以是以下:
a:也就是all,输出所有的调试信息
b:也就是basic,只输出简单的调试信息,即输出不需要重新编译的目标
v:也就是verbose,输出的信息包括哪一个makefile文件被解析,不需要重新编译的依赖文件(或是依赖目标)
i:也就是implicit,输出所有的隐含规则
j:也就是jobs,输出执行规则中命令的详细信息,如PID、返回码等
m:也就是makefile文件,输出make,读取makefile,更新makefile文件,并执行makefile文件的信息
16.隐含规则  
　a.C程序隐含规则:<n>.o的目标的依赖目标会是:<n>.c,命令是`$(CC) -c $(CPPFLAGS)$(CFLAGS)`
　b.C++程序隐含规则:<n>.o的目标的依赖目标是:<n>.cc或<n>.C,命令是`$(CXX) -c  $(CPPFLAGS)$(CXXFLAGS)`
　c.模式规则示例:
```bash
%.o: %.c
 $(CC) -c $(CFLAGS) $(CPPFLAGS) $< -o $@
```
17.函数库打包
函数库文件也就是对.o文件的打包文件
示例:
```bash
foolib(hack.o): hack.o
	ar cr foolib hack.o  #foolib是库名,hack.o是包含文件
```
