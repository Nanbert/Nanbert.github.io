---
title: Make
date: 2020-01-17 12:45:15
subtitle:
categories:
tags:
index_img: /img/makefile.png
banner_img: /img/makefile.png
---

### 引用其它的Makefile
`include <filename>`
make会在当前目录寻找,接着在以下目录下找:
* 如果make执行时，有 -I 或 --include-dir 参数，那么make就会在这个参数所指定的目录下去寻找。
* 如果目录 <prefix>/include （一般是： /usr/local/bin 或 /usr/include ）存在的话，make也会去找。
### 变量相关  
变量在声明时需要给予初值，而在使用时，需要给在变量名前加上`$`符号，但最好用小括号`()`或是大括号`{}`把变量给包括起来。如果你要使用真实的`$`字符，那么你需要用`$$`来表示

#### 赋值操作
* `:=`操作符,操作符右边只能出现已定义的变量,如果是未定义的变量,则会自动忽略,用来避免递归展开 
* `=`操作符右边可以出现未定义的变量
*  `?=`如果变量没有定义过,则使用后面的值
* `+=`追加变量值

#### 内置变量

|变量类型|特殊变量|含义|用例或说明|
|:-:|:-:|:-:|:-:|
|特殊变量|`VPATH`|寻找依赖或目标的路径,以冒号为分隔符,当前目录永远最优先|`VPATH = src:../headers`|
|特殊变量|`SUFFIXE`|定义默认的后缀列表,最好不要直接改变，通过`.SUFFIXE`|| |特殊变量|.DIRSEPSTR|路径分隔符一般是斜杠||
|特殊变量|.MAKEDIR|调用make的绝对路径名||
|特殊变量|.NULL|空字符串||
|特殊变量|.OS|正在运行的操作系统名称||
|特殊变量|.PWD|运行时活动工作目录的绝对路径名||
|特殊变量|.SHELL|启动的shell类型||
|命令变量|`AR`|函数库打包程序,默认命令是ar||
|命令变量|`AS`|汇编语言编译程序,默认命令是as||
|命令变量|`CC`|C语言编译程序,默认命令是cc||
|命令变量|`CXX`|C++语言编译程序,默认命令是g++||
|命令变量|`CPP`|C程序的预处理器,默认命令是$(CC) -E||
|命令变量|`RM`|删除文件命令,默认命令是rm -f||
|参数变量|`ARFLAGS`|函数库打包程序AR命令参数,默认值是rv||
|参数变量|`ASFLAGS`|汇编语言编译器参数,默认值是空||
|参数变量|`CFLAGS`|C语言编译器参数,默认为空||
|参数变量|`CPPFLAGS`|C预处理器参数,默认为空||
|参数变量|`CXXFLAGS`|C++语言编译器参数,默认为空||
|参数变量|`LDFLAGS`|ld链接器参数,默认为空||
|系统变量|`MAKELEVEL`|当前Makefile的调用层数,从0开始||
|系统变量|`MAKECMDGOALS`|存放那个你命令行中所指定的终极目标的列表,没有指定则为空||
|自动变量|`$@`|目标集|它代表一个量,遍历目标集,一般与依赖集相匹配|
|自动变量|`$%`|目标集|仅当目标是函数库文件时,表示规则中的目标成员名,foo.a(bar.o),$%就是bar.o,$@就是foo.a|
|自动变量|`$<`|依赖集|它代表一个量,遍历目标集,一般用于目标集相匹配|
|自动变量|`$?`|依赖集|所有比目标新的依赖目标的集合,以空格分隔|
|自动变量|`$^`|依赖集|所有依赖目标的集合,去掉重复以空格分隔|
|自动变量|`$+`|依赖集|所有依赖目标的集合,不去掉重复以空格分隔|
|自动变量|`$*`||对应模式的'%'及之前的部分,包括路径|

**所有自动变量都可以与`D`,`F`搭配使用,表示匹配的目录部分和文件部分,如`$(@D)`**
#### 高级用法
* 替换`.o`到`.c`
**方法一:**
```bash
foo:=a.o b.o c.o
bar:=$(foo:.o=.c)
```
**方法二:**
```bash
#静态模式
foo:=a.o b.o c.o
bar:=$(foo:%.o=%.c)
```
* 强制覆盖 

`override <variable>=<value>`make命令行参数可以用这个强制覆盖,否则覆盖不了
* 替换变量中字符串
`变量名: s/原字符串/新字符串`
* 加前后缀
`变量:^ "前缀"`
`变量:+ "后缀"`
* 取部分
`$(VARIABLE:<option>)`,option有3个选项d(仅取路径)、b(文件名,不包括扩展)、f(文件名,包括扩展)
### 关键字vpath
* 用法一:`vpath <pattern> <directories>`符合模式的在指定文件夹搜索
* 用法二:`vpath <pattern>`清除对应模式搜索目录
* 用法三:`vpath`清除所有搜索目录
***注意:<pattern>都必须包含%,意思是包含一个以上的匹配字符***
### 局部作用的变量
```bash
prog: CFLAGS = -g
	prog: prog.o foo.o bar.o
		$(gcc) $(CFLAGS) prog.o foo.o bar.o
	prog.o: prog.c
		$(gcc) $(CFLAGS) prog.c
	foo.o: foo.c
		$(gcc) $(CFLAGS) bar.c
```
不管全局的`$(CFLAGS)`的值是什么,prog目标及其所引发的所有规则中(prog.o foo.o bar.o),$(CFLAGS)的值都是-gl
### 伪目标
用`.PHONY`指明伪目标
- 伪目标并不是文件，只是个标签,最终不产生文件
- 只有显式地指名才能使其生效
- 最终目标可以是伪目标,一个用法如下,使一个make文件生成多个目标:
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
### 静态模式:更方便定义多目标
`<targets>: <target-pattern>: <prereq-patterns>`
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
### gcc -MM选项
该选项可以为.c源文件自动生成依赖的非标准库的头文件,按习惯称为.d文件
可以用以下模式规则来产生.d文件
```bash
%.d: %.c
    @set -e; rm -f $@; \
    $(CC) -M $(CPPFLAGS) $< > $@.$$$$; \
    sed 's,\($*\)\.o[ :]*,\1.o $@ : ,g' < $@.$$$$ > $@; \
    rm -f $@.$$$$
```
然后用include
```bash
sources = foo.c bar.c
include $(sources:.c=.d)
```
### `@`关键字
make默认会显示命令,在命令前加@可以阻止输出,例如
`@echo 正在编译XXX模块`
### `-`关键字
在命令前加`-`会忽略该命令产生的错误
### 嵌套make  
```bash
subsystem:
	cd subdir && $(MAKE) #也可以这么写$(MAKE) -C subdir
```
　要想传递变量给嵌套的make,使用export,不想传递使用unexport
　其中`SHELL`和`MAKEFLAGS`总会影响下层make,但-C、-f、-h、-o、-W几个参数并不往下传递
### 定义命令包(即命令集)
其实这是多行变量
例子:
```bash
define run-yacc
mkdir dir
mv dir newdir
endef
crap: 
	$(run-yacc)
```
### 条件判断
make是在读取Makefile时就计算条件表达式的值,而不是运行,所以不要用自动变量
```bash
xx: xc
<conditional-directive> #可以是ifeq(arg1,arg2)、ifneq、ifdef <variable-name>(不加美元符号)、ifndef
	<text-if-true>
else
	<text-if-false>
endif
```
### 函数
#### 调用语法
`$(<function> <arguments>)`,参数间用','分隔
#### 字符串函数

|函数|功能|返回|
|:-:|:-:|:-:|
|`$(subst <from>,<to>,<text>)`|把字符串<text>中的<from>换成<to>|`被替换过的字符串`|
|`$(patsubst <pattern>,<replacement>,<text>)`|查找<text>中的单词(以空格、Tab、回车、换行)符合<pattern>,替换<replacement>|被替换后的字符串|
|`$(strip <string>)`|去掉<string>中开头和结尾的空字符|去掉空字符的字符串|
|`$(findstring <find>,<in>)`|在字符串<in>中查找<find>|如果找到,返回<find>,否则返回空字符串|
|`$(filter <pattern...>,<text>)`|以<pattern>模式过滤<text>字符串中的单词(以空格等作为分隔符),可以有多个模式,模式间以空格分割|返回符合<pattern>的字符串|
|`$(filter-out <pattern...>,<text>)`|以<pattern>模式过滤<text>字符串中的单词(以空格等作为分隔符),可以有多个模式,模式间以空格分割|返回不符合<pattern>的字符串|
|`$(sort <list>)`|给字符串<list>中的单词升序|返回排序后的字符串(会去掉相同的单词)|
|`$(word <n>,<text>)`|取字符串<text>中的第<n>个单词|返回该单词,如果n过大,则返回空字符串|
|`$(wordlist <n>,<m>,<text>)`|取第<n>-第<m>个单词|返回那些单词|
|`$(words <text>)`|统计<text>中的单词数|返回个数|
|`$(firstword <text>)`|返回字符串<text>中第一个单词|返回第一个单词|

#### 文件名函数

|格式|例子|返回值|
|:-:|:-:|:-:|
|`$(dir <names...>)`|`$(dir src/foo.c hacks)`|`src/ ./`|
|`$(notdir <names...>)`|`$(notdir src/foo.c hacks)`|`foo.c hacks`|
|`$(suffix <names...>)`|`$(suffix src/foo.c src-1.0/bar.c hacks)`|`.c .c`|
|`$(basename <names...>)`|`$(basename src/foo.c src-1.0/bar.c hacks)`|`src/foo src-1.0/bar hacks`|
|`$(addsuffix <suffix>,<names...>)`|`$(addsuffix .c,foo bar)`|`foo.c bar.c`|
|`$(addprefix <prefix>,<names...>)`|`$(addprefix src/,foo bar)`|`src/foo src/bar`|
|`$(join <list1>,<list2>)`|`$(join aaa bbb,111 222 333)`|`aaa111 bbb222`|

#### foreach函数
格式:`$(foreach <var>,<list>,<text>)`
```bash
names:= a b c d
files:= $(foreach n,$(names),$(n).o)
#$(files)的值是‘a.o b.o c.o d.o’
```
#### if函数
`$(if <condition>,<then-part>,<else-part>)`,<condition>若返回为非空,则执行<then-part>,其是整个函数的返回值
#### call函数
`$(call <expression>,<parm1>,<parm2>,<parm3>...)`;<expression>中`$(1)`,`$(2)`等,会被参数`<parm1>`、`<parm2>`等代替
```bash
reverse=$(2) $(1)
foo=$(call reverse,a,b)
```
此时foo的值就是'b a'
#### shell函数
`files:=$(shell echo *.c)`
#### origin函数
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

#### error函数和warning函数
`$(error <text ...>)`
`$(warning <text ...>)`
error函数产生一个致命的错误,<text ...>是错误信息
warning函数只是输出警告信息,make会继续执行

例1：
```bash
ifdef ERROR_001
	#运行到下面的会出错跳出脚本
    $(error error is $(ERROR_001))
endif
```
例2：
```bash
#这里并不会出错跳出脚本
ERR = $(error found an error!)

.PHONY: err
#这里才会跳出
err: $(ERR)
```

### make的退出码
* 0:表示成功执行
* 1:表示出错
* 2:如果你使用了make的“-q”选项，并且make使得一些目标不需要更新，那么返回2。

### 一些常用伪目标命名

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

### make选项参数

|短选项|长选项|含义|
|:-:|:-:|:-:|
|-b,-m||忽略其他版本make兼容性|
|-B|--always-make|认为所有目标都更新(重编译)|
|-C <dir>|--directory=<dir>|指定读取makefile的目录。如果有多个“-C”参数，make的解释是后面的路径以前面的作为相对路径，并以最后的目录作为被指定目录。如：“make -C ~hchen/test -C prog”等价于“make -C ~hchen/test/prog”|
|-d||相当于`-debug=a`|
|-e|--environment-overrides|指定环境变量值,覆盖makefile文件中定义的变量值|
|-f|--file、--makefile|指定某个makefile文件|
|-i|--ignore-errors|在执行时忽略所有的错误|
|-I <dir>|--include-dir=<dir>|指定一个包含makefile文件的搜索目标|
|-j <jobsnum>|--jobs=<jobsnum>|指同时运行的命令数,如果没有这个参数,能运行多少就多少,只有最后一个-j选项有效|
|-k|--keep-going|出错也不停止,执行其它目标,失败的目标,依赖于其上的就不会执行|
|-l <load>|--load-average [=<load>];--max-load[=<load>]|指定make运行命令的负载|
|-n|--just-print、--dry-run、--recon|不管目标更不更新,只打印命令,不执行|
|-o <file>|--old-file=<file>;--assume-old=<file>|不生成指定的<file>,即使这个目标的依赖文件比他新|
|-p|--print-database|输出makefile文件中所有数据,包括所有的规则和变量|
|-q|--question|寻找目标,如果目标存在,什么也不输出,也不执行编译,返回0.如果目标不存在,打印一条出错信息,返回2|
|-r|--no-builtin-rules|禁止使用任何隐式规则，会使得SUFFIXE变量为空|
|-R|--no-builtin-variables|禁止使用任作用于变量上的何隐式规则|
|-s|--silent;--quiet|命令运行时不显示命令的输出|
|-S|--no-keep-going;--stop|取消-k选项的作用|
|-t|--touch|把目标文件时间更新,但不更改目标文件,假装编译文件|
|-w|--print-directory|输出运行makefile文件之前之后的信息,跟踪嵌套make时很有用|
|-W <file>|--what-if=<file>;--assume-new=<file>;--new-file=<file>|假定目标<file>;需要更新，如果和“-n”选项使用，那么这个参数会输出该目标更新时的运行动作。如果没有“-n”那么就像运行UNIX的“touch”命令一样，使得<file>;的修改时间为当前时间。|
||--no-print-directory|禁止-w选项|
||--warn-undefined-variables|警告未定义的变量|

#### --debug \<options\>
options可以是以下:
* 也就是all,输出所有的调试信息
* 也就是basic,只输出简单的调试信息,即输出不需要重新编译的目标
* 也就是verbose,输出的信息包括哪一个makefile文件被解析,不需要重新编译的依赖文件(或是依赖目标)
* 也就是implicit,输出所有的隐含规则
* 也就是jobs,输出执行规则中命令的详细信息,如PID、返回码等
* 也就是makefile文件,输出make,读取makefile,更新makefile文件,并执行makefile文件的信息
#### 常用组合
- `make -qp`只输出信息而不执行
- `make -p -f /dev/null`查看makefile前的预设变量和规则
### 模式的匹配
一般来说，一个目标的模式有一个有前缀或是后缀的%，或是没有前后缀，直接就是一个%。因为%代表一个或多个字符，所以在定义好了的模式中，我们把%所匹配的内容叫做“茎”，例如%.c所匹配的文件“test.c”中“test”就是“茎”。因为在目标和依赖目标中同时有%时，依赖目标的“茎”会传给目标，当做目标中的“茎”。

当一个模式匹配包含有斜杠（实际也不经常包含）的文件时，那么在进行模式匹配时，目录部分会首先被移开，然后进行匹配，成功后，再把目录加回去。在进行“茎”的传递时，我们需要知道这个步骤。例如有一个模式e%t，文件src/eat匹配于该模式，于是src/a就是其“茎”，如果这个模式定义在依赖目标中，而被依赖于这个模式的目标中又有个模式c%r，那么，目标就是src/car。（“茎”被传递）
### 一些"老东西"--后缀规则
- `.c.o:`等价于`%.o : %.c`
- `.c:`等价于`% : %.c`
以上称为后缀规则，要想这么用，必须为默认后缀，你可以用以下来添加默认后缀
```bash
.SUFFIXES:              # 删除默认的后缀
.SUFFIXES: .a .b .c c # 定义自己的后缀
```
### 隐含规则  
#### 各个规则
- C程序隐含规则:<n>.o的目标的依赖目标会是:<n>.c,命令是`$(CC) -c $(CPPFLAGS)$(CFLAGS)`
- C++程序隐含规则:<n>.o的目标的依赖目标是:<n>.cc或<n>.C,命令是`$(CXX) -c  $(CPPFLAGS)$(CXXFLAGS)`
- 汇编和预处理隐含规则:<n>.o的目标的依赖目标会自动推导为<n>.s，默认使用编译器as，并且其生成命令是：`$ (AS) $(ASFLAGS)`。<n>.s的目标的依赖目标会自动推导为<n>.S，默认使用C预编译器cpp，并且其生成命令是：`$(AS) $(ASFLAGS)`。
- 链接Object文件的隐含规则:<n>目标依赖于<n>.o，通过运行C的编译器来运行链接程序生成（一般是 ld ），其生成命令是：`$(CC) $(LDFLAGS) <n>.o $(LOADLIBES) $(LDLIBS)`。这个规则对于只有一个源文件的工程有效，同时也对多个Object文件（由不同的源文件生成）的也有效。例如如下规则:
```bash
x : y.o z.o
```
隐含规则执行如下(x.c、y.c、z.c都存在):
```bash
cc -c x.c -o x.o
cc -c y.c -o y.o
cc -c z.c -o z.o
cc x.o y.o z.o -o x
rm -f x.o
rm -f y.o
rm -f z.o
```
#### 一些tip
- 隐式规则产生的中间目标，最终会被自动删除，被makefile指定成的目标或依赖目标不能被当作中介，但是可以通过`.INTERMEDIATE`来强制声明为中间目标。如`.intermediate: mid`
- 也可以阻止自动删除中间目标，通过`.SECONDARY`来强制声明。如`.SECONDARY`。或以模式的方式指定(如：%.o)成为伪目标`.PRECIOUS`的依赖目标。
- Make会优化一些特殊的隐含规则，而不生成中间文件。从文件.c直接生成执行文件，不产生目标文件
#### 模式规则来定义一个隐含规则
模式规则，目标的定义需要有`%`字符。依赖目标随便，与变量不同的是，模式规则的展开发生在运行期间。
示例：
```bash
%.o: %.c
 $(CC) -c $(CFLAGS) $(CPPFLAGS) $< -o $@
```
#### 隐含规则搜索算法
比如我们有一个目标叫T。下面是搜索目标T的规则的算法。请注意，在下面，我们没有提到后缀规则，原因是，所有的后缀规则在Makefile被载入内存时，会被转换成模式规则。如果目标是archive(member)的函数库文件模式，那么这个算法会被运行两次，第一次是找目标T，如果没有找到的话，那么进入第二次，第二次会把member当作T来搜索。

1) 把T的目录部分分离出来。叫D，而剩余部分叫N。（如：如果T是src/foo.o，那么，D就是src/，N就是foo.o）
2) 创建所有匹配于T或是N的模式规则列表。
3) 如果在模式规则列表中有匹配所有文件的模式，如%，那么从列表中移除其它的模式。
4) 移除列表中没有命令的规则。
5) 对于第一个在列表中的模式规则：
  *  推导其“茎”S，S应该是T或是N匹配于模式中%非空的部分。
  *  计算依赖文件。把依赖文件中的%都替换成“茎”S。如果目标模式中没有包含斜框字符，而把D加在第一个依赖文件的开头。
  * 测试是否所有的依赖文件都存在或是理当存在。（如果有一个文件被定义成另外一个规则的目标文件，或者是一个显式规则的依赖文件，那么这个文件就叫“理当存在”）
  * 如果所有的依赖文件存在或是理当存在，或是就没有依赖文件。那么这条规则将被采用，退出该算法。
6) 如果经过第5步，没有模式规则被找到，那么就做更进一步的搜索。对于存在于列表中的第一个模式规则：
  * 如果规则是终止规则，那就忽略它，继续下一条模式规则。
  * 计算依赖文件。（同第5步）
  * 测试所有的依赖文件是否存在或是理当存在。
  * 对于不存在的依赖文件，递归调用这个算法查找他是否可以被隐含规则找到。
  * 如果所有的依赖文件存在或是理当存在，或是就根本没有依赖文件。那么这条规则被采用，退出该算法。
  * 如果没有隐含规则可以使用，查看.DEFAULT规则，如果有，采用，把.DEFAULT的命令给T使用。
### 函数库打包
函数库文件也就是对.o文件的打包文件
示例:
```bash
foolib(hack.o xx.o): hack.o xx.o
	ar cr foolib hack.o xx.o #foolib是库名,hack.o是包含文件
```

### Tips
* make中遇到的第一条规则是最终目标  
* 和bash一样用空格和"\"进行换行
* 命令总是以tab键开头,其余不是命令
* `MAKEFILES`最好不用该环境变量,该变量类似于include动作,但其中文件中的目标不会起作用,最好置为空,以免引入未考虑到的东西,莫名奇妙出现问题时,可以查看该变量
* 要想前面命令作用于后面命令,需要写在同一行上,用分号分隔,如下:
```bash
exec:
	cd /home/nanbert;pwd
```
* 可以在命令前加`-`,来忽略该命令可能执行失败,这也可用于include
