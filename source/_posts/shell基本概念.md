---
title: shell基本概念
date: 2021-10-15 20:10:51
subtitle:
categories:
tags:
index_img: https://z3.ax1x.com/2021/11/13/IyR7zd.jpg
banner_img: https://z3.ax1x.com/2021/11/13/IyR7zd.jpg
---
### 注释
* 单行注释: \#
* 多行注释:  `:<<EOF ... EOF`或者 `:<<! ... !`

### 变量
* 创建普通变量: **name="test"** (=两边不可有空格)
* 创建函数体中的局部变量: ** local name="test"**,函数体及整个脚本中的变量默认都是全局变量,函数体内外皆可访问和改变,函数体内的变量最好用这个加以限制
* 使用变量: **echo $name 或者 echo $(name)** 使用时要加$,重新赋值时不需要
* 只读变量: **name="only_read" -> readonly name**
* 删除变量: **unset name**
  
### 字符串变量
字符串与变量展开有密切联系,可以参考那节
1) 单引号
* 单引号变量var='test',只能原样输出,不能解释变量
* 单引号中不能出现一个单引号,转义也不行

2) 双引号
* 双引号变量`var="my name is ${name}"`,可以解释变量
* 可以出现转移字符

3) 拼接字符串
* `name="this is""my name";name="this is my name";name="this"is"my name"`等效

4) 获取字符串长度
```bash
echo ${#str}
```
  
5) 提取子字符串
```bash
name="this is my name";
echo ${name:1:4} #输出his
echo ${name::4} #输出this
```
6) 大小写
```bash
declare -u upper
declare -l lower
#upper将会强制转成SL
upper="sl"
#lower将会强制转成sl
lower="SL"
```
### 数组
bash只支持以为数组,不支持多维数组
* 定义数组: **array_name=(li wang xiang zhang)** (小括号做边界、使用空格分离)或者**declare -a array_name**
* 单独定义数组的元素: **arraypara[0]="w";arraypara[3]="s"** (定义时下标可以不连续,同样可以用于赋值)
* 获取数组元素
```bash
array_name[0]="li"
array_name[3]="zhang"
echo ${array_name[0]} #输出"li"
echo ${array_name[1]} #输出空
echo ${array_name[@]} #输出"li zhang"输出数组所有元素,没有元素的下标省略
#等价于下面
echo ${array_name[*]}
```
* 取得元素个数:
```bash
${#arrayname[@]}
#或者
${#arrayname[\*]}
```
* 取得单个元素长度: 
```bash
${#array_name[1]}
```
* `for i in "${!foo[@]}"`取得每个非空元素的下标
* `foo+=(d e f)`添加3个元素
* 删除数组不能通过赋空值,只能通过unset,unset还能删除某个元素(这样后面元素往前补)
* 没有指明数组下标的访问或赋值皆指向第一个值
* 最新版本bash支持关联数组`declare -A colors;colors["red"]="red"`
* [函数与数组](#funAndArr)
* 取切片`echo ${array:0:3}`
* 遍历数组
```bash
#方法1
for(( i=0;i<${#array[@]};i++)) do
echo ${array[i]};
done;
#方法2
for element in ${array[@]}
do
echo $element
done
#方法3
for i in "${!arr[@]}";
do
    printf "%s\t%s\n" "$i" "${arr[$i]}"
done
```
### 环境变量
#### FUNCNAME
该环境变量是个数组，存储当前位置函数调用的堆栈，当只有主程序时，个数为0,当存在调用函数时，第一个总是当前函数名，最后一个为main表示主程序
#### BASH_SOURCE
是一个数组，不过它的第一个元素是当前脚本名称，然后是source它的脚本，依次类推，常用如下
```bash
# 如果脚本是被source的话
if [ -n "$BASH_SOURCE" -a "$BASH_SOURCE" != "$0" ]
then
    do_something
else # Otherwise, run directly in the shell
    do_other
fi
```

### 参数传递的相关特殊变量

|变量|意义|
|:-:|:-:|
|`$0`| 代表执行的文件名|
|`$n`| 代表传入的第n个参数|
|`$#`|参数个数,不包括程序名本身|
|`$`|以一个单字符串显示所有向脚本传递的参数。即为"$1 $2...$n"|
|`$@`|展开成一个从 1 开始的位置参数列表。当它被用双引号引 起来的时候，展开成一个由双引号引起来的字符串，包含了 所有的位置参数，每个位置参数由 shell 变量 IFS 的第一个 字符（默认为一个空格）分隔开。|
|`$*`|把所有参数当成一个大字符串|
|`$$`| 该脚本进程ID|
|`$!`| 后台运行的最后一个进程ID|
|`$?`|上个调用(最后命令)返回值,0表示没有错误|

有个**shift**命令可以方便处理命令行参数,每次执行该命令的时候,变量$2会移动到$1,变量$3会移动到变量$2,以此类推,$#值也会减1。例子:
```bash
count=1
while [[ $# -gt 0 ]];
do
	echo "Argument $count = $1"
	count=$((count + 1))
	shift
done
```
[getopt和getopts两个命令经常被用来处理传递的参数](#jumpopt)


### 运算符

#### 算数运算
 * `+ - * /`
 * 加法运算
     * `bash val=$(expr 2 + 2)` 这么写乘号要加转义,空格也是必须
     * `val=$[2+2]` (4个空格不是必要的,不同于条件判断)
     * `val=$((2+2))`(4个空格不是必要的)

bash支持任意进制

|表示法|描述|
|:-:|:-:|
|number|默认10进制|
|0number|8进制|
|0xnumber|16进制|
|base#number|base进制|

在双括号中会被解释成数字,否则默认为字符串

#### 数字关系运算符
关系运算符只支持数字,不支持字符串,除非字符串是数字

|符号|意义|
|:-:|:-:|
|`-eq`|相等返回true,`[$a -eq $b]`|
|`-ne`|不相等返回true,`[$a -ne $b]`||
|`-gt`|大于号|
|`-lt`|小于号|
|`-ge`|大于等于号|
|`-le`|小于等于号|

#### 字符串运算符

|符号|意义|
|:-:|:-:|
|**=**|相等返回true,`[$a = $b]`|
|**!=**|不相等返回true,`[$a != $b]`|
|**-z**|字符串长度为0返回true,`[-z $b]`|
|**-n**|字符串长度不为0返回true,`[-n $b]`|
|**$**|不为空返回true,`[$a]`|

#### 逻辑运算符

|符号|意义|
|:-:|:-:|
|**!**|非运算,`[! false]`|
|**&&**|与运算,`[[ $a -lt 20 ]] && [[$b -gt 100 ]]`|
|**||**|或运算,`[[ $a -lt 20 ]] || [[$b -gt 100 ]]`,`[[ $a -lt 20 || $b -gt 100 ]]`|

逻辑判断的括号
1.`[]`:中括号旁边和运算符两边必须添加空格(可以使用，等价于test命令,本文不讲test命令,不推荐)
2.`[[]]`:中括号旁边和运算符两边必须添加空格(字符串验证,文件名时，推荐)
3.`(())`:中括号旁边和运算符两边必须添加空格(数字验证时，推荐)
4.`[[]]和(())`分别是针对数学表达式和字符串表达式的加强版
5.`[]`基本舍弃的原因，它与**&&、||、<和>**不兼容,会报错,它只能用-ne等等这些,例如下面是等价的
`if [[ $a != 1 && $a !=2 ]]`,`if [ $a -ne 1 ] && [ $a != 2 ]`,`if [ $a -ne 1 -a $a !=2 ]`
6.双括号支持以下额外的符号(没列全,C语言能用的都能用,包括?:)

|符号|描述|
|:-:|:-:|
|val++|后增|
|val--|后减|
|++val|前增|
|--val|前减|
|!|逻辑求反|
|~|位求反|
|**|幂运算|
|<<|左位移|
|>>|右位移|
|&|布尔和|
|`|`|布尔或|
|`||`|逻辑或|
|&&|逻辑与|
|=~|用于字符串的模式匹配|
|==|用于字符串的类型匹配(通配符等),例子`if [[ $FILE == foo.* ]]`|


#### 文件运算符
file是代表文件名的字符串

|符号|意义|
|:-:|:-:|
|$file1 -ef $file2|拥有相同的索引号返回True(硬连接)|
|$file1 -nt $file2|file1新于file2返回true|
|$file1 -ot $file2|file1早于file2返回true|
|`[-b $file]`|是块设备文件返回true|
|`[-c $file]`|是字符备文件返回true|
|`[-d $file]`|是目录返回true|
|`[-f $file]`|是普通文件返回true|
|`[-g $file]`|设置了SGID位文件返回true|
|`[-h $file]`|是符号链接返回true|
|`[-G $file]`|由有效组(即当前进程用户组)ID拥有返回true|
|`[-L $file]`|是符号链接返回true|
|`[-k $file]`|设置了stick位文件返回true|
|`[-N $file]`|atime和mtime一样的文件|
|`[-p $file]`|是有名管道文件返回true|
|`[-u $file]`|设置了SUID位文件返回true|
|`[-O $file]`|由有效用户(即当前进程用户)件D拥有返回true|
|`[-r $file]`|是可读文件返回true|
|`[-w $file]`|是可写文件返回true|
|`[-x $file]`|是可执行文件返回true|
|`[-s $file]`|非空文件返回true|
|`[-S $file]`|是一个网络Socket返回true|
|`[-e $file]`|文件存在返回true|
|`[-t $fd]`|fd是一个定向到终端/从终端定向的文件描述符。这可以被用来判断是否重定向了标准输入/错误|

### 输出
**echo**只用于字符串,自动添加换行符号`echo nanbert male 66.1234`
**printf**不自动加换行符号,例:`printf "%-10s %-8s %-4.2f\n" nanbert male 66.1234`

### 流程控制
#### if
```bash
#then后必须有语句,空语句可以用:
if condition
then 
	command1
	...
elif condition
then
	command2
	...
else
	command3
	...
fi
```
#### for
```bash
for var in item1 item2 ... itemN
do 
	command1
	...
done
#等价于c语言
for (( expression1;expression2; expression3 ));do
	commands
done
```
如果省略in，默认处理位置参数
#### while
```bash
while condition
do 
	command1
	...
done
#无限循环
while :
do
	command
done
```
#### until
```bash
until condition
do 
	command1
	...
done
```
#### case
Shell case匹配一个值与一个模式,用两个分号表示break
```bash
case value in
	pattern1)
		command1
		...
		commandN
		;;
	pattern2)
		command1
		...
		commandN
		;;
esac
```

循环都支持**continue**和**break**,`break n`则可以指定跳出n层循环,n默认为1
,continue也支持数字。
### 定义函数
* 函数定义
```bash
function fun(){
	action;
	[return int;]
}
#等价于下面
fun1(){
	action;
	[return int;]
}
```
* 参数传递
函数中直接使用特殊变量来获取参数,可以加上{}
```bash
funWithParam(){
    echo "第一个参数为 $1 !"
    echo "第二个参数为 $2 !"
    echo "第十个参数为 $10 !"
    echo "第十个参数为 ${10} !"
    echo "第十一个参数为 ${11} !"
    echo "参数总数有 $# 个!"
    echo "作为一个字符串输出所有参数 $* !"}
funWithParam 1 2 3 4 5 6 7 8 9 34 73
echo $?  # 判断执行是否成功
```
* 函数返回值
    * 返回值是可选的
	* return只能为**return [0-255]**,可通过$?获取该值
	* 如果不加return,则最后一条语句的执行状态为返回值,0为成功
	* 如果用反引号执行函数,结果是函数内的所有输出而非返回值
* [函数与数组](#funAndArr)

### 读取外部输入
`read arg`从键盘读取输入并赋值给arg

|选项|说明|
|:-:|:-:|
|-a array|把输入赋值到数组array中|
|-d delimiter|用字符串delimiter中的第一个字符指示输入结束,而不是一个换行符|
|-e|使用readline来处理输入|
|-n num|读取num个输入字符,而不是整行|
|-p prompt|为输入显示提示信息,使用字符串prompt|
|-r |Raw mode,不把反斜杠解释为转义字符|
|-s |Silent mode,不会在屏幕上显示输入的字符。输入密码的时候很有用|
|-t seconds|超过时间，终止输入,read会非0状态退出|
|-u fd|使用文件描述符fd中的输入,而不是标准输入|
|-k n|只读n个字节|

IFS是字段分割符,默认为空格,tab,换行符。可以自行改变。如下:
`IFS=:`改成冒号
`IFS=$'\n':;"`改成换行符、冒号、分号和双引号
read不应该使用管道线来接受赋值，如下是错误的:
```bash
echo "$file_info" | IFS=":" read user pw uid gid name home shell
```
而应该这么写
```bash
IFS=":" read user pw uid gid name home shell <<< "$file_info"
```
这是因为管道线会开个子进程,子进程变量的变化不会影响父进程,可以用[进程替换](#jump)解决
如果输入的数多余接受的变量,则多出来的会保存在变量REPLY中

### 包含其他shell文件
* `. filepath/filename`
* `source filepath/filename`

### 颜色标识
```bash
printf  "\033[32m SUCCESS: yay \033[0m\n";
printf  "\033[33m WARNING: hmm \033[0m\n";
printf  "\033[31m ERROR: fubar \033[0m\n";
```
具体内容有待研究

### 长句换行
在shell中为避免一个语句过长,可以使用"\"进行换行,注意"\"前加一个空格,之后无空格直接换行。

### 退出脚本
`exit [num]`num为0表示执行成功,可以不加num
`set -e 或 set +e`set -e表示从当前位置开始,如果出现任何错误都将触发exit。相反,set +e表示不管出现任何错误继续执行脚本

### 调试
-n表示检查有无语法错误,-x表示调试

### 变量展开

|操作符|意义|
|:-:|:-:|
|`:${parameter:=word}`|如果parameter没有设置或者为空,展开的结果是word的值,并且word的值会赋给parameter;否则展开为parameter的值|
|`${parameter:-word}`|如果parameter没有设置或者为空,展开的结果是word的值,否则是parameter的值|
| `${parameter:?word}` |如果parameter没有设置或者为空,会带有错误的推出,否则是parameter的值|
| `${parameter:+word}` |如果parameter没有设置或者为空,展开为空,否则是word的值;不管如何parameter值不会变|
|`${parameter#pattern}`|pattern是通配符模式,会从parameter开头开始最短匹配pattern,删除匹配中的部分,留下剩余部分,注意是开头不是从左到右第一次的意思,即第一个字符一定要匹配上|
|`${parameter##pattern}`|pattern是通配符模式,会从parameter开头开始最长匹配pattern,删除匹配中的部分,留下剩余部分,注意是开头不是从左到右第一次的意思,即第一个字符一定要匹配上|
|`${parameter%pattern}`|pattern是通配符模式,会从parameter结尾开始最短匹配pattern,删除匹配中的部分,留下剩余部分,注意是结尾不是从右到左第一次的意思,即最后一个字符一定要匹配上|
|`${parameter%%pattern}`|pattern是通配符模式,会从parameter结尾开始最长匹配pattern,删除匹配中的部分,留下剩余部分,注意是结尾不是从右到左第一次的意思,即最后一个字符一定要匹配上|
|`${parameter/pattern/string}`|在parameter中找到匹配通配符pattern的文本,用string替换,只替换第一次匹配到的|
|`${parameter//pattern/string}`|在parameter中找到匹配通配符pattern的文本,用string替换,替换所有匹配到的|
|`${parameter/#pattern/string}`|在parameter中找到匹配通配符pattern的文本,用string替换,只能从parameter开头开始匹配,注意是开头不是从左到右第一次的意思,即第一个字符一定要匹配上|
|`${parameter/%pattern/string}`|在parameter中找到匹配通配符pattern的文本,用string替换,只能从parameter结尾开始匹配,注意是结尾不是从右到左第一次的意思,即最后一个字符一定要匹配上|
|`${parameter,,}`|把parameter全部展开成小写字母|
|`${parameter,}`|把parameter首字母展开成小写字母|
|`${parameter^^}`|把parameter全部展开成大写字母|
|`${parameter^}`|把parameter首字母展开成大写字母|


`${!prefix*}`等价于`${!prefix@}`这种展开会返回以prefix开头的已有变量名(而不是变量的值)
字符串的长度和切片其实也是一种展开

<span id = "jump"></span>
### 进程替换
进程替换可以用来解决子进程问题,它实质是把子进程的输出当作一个用于重定向的普通文件(文件描述符)。
标准输出
`<(cmd1;cmd2)`
标准输入
`>(cmd1;cmd2)`
由此可以解决read管道线的问题
```bash
while read attr links owner group size date time filename;do
	cat <<- EOF
		Filename: $filename
		Size:     $size
		Owner:    $owner
		Group:    $group
		Modified: $date $time
		Links:    $links
		Attributes: $attr
	EOF
done < <(ls -l | tail -n +2)
```
<span id = "jumpopt"></span>
### getopt和getopts

#### getopt
格式:`getopt [options] optstring parameters`,例:
`getopt ab:cd -a -b test1 -cd test2 test3`该命令会产生如下输出:
`-a -b test1 -c -d -- test2 test3`,optstring定义了四个有效项字母:a、b、c和d。冒号(:)表示b选项需要个参数值,它会将-cd选项分成两个单独选项,插入'--'来分隔额外的参数,如果提供'-cde'由于e不在optstring中,会报错**getopt: invalid option -- e**,但还会输出结果,可以加-q选项忽略报错结果。
在脚本中经常与set的'--'选项来使用,来把getopt的输出转成当前脚本的输入参数,示例如下:
```bash
#!/bin/bash
# Extract command line options & values with getopt
#
set -- $(getopt -q ab:cd "$@")
#
echo
while [ -n "$1" ]
do
    case "$1" in
    -a) echo "Found the -a option" ;;
    -b) param="$2"
        echo "Found the -b option, with parameter value $param"
        shift ;;
    -c) echo "Found the -c option" ;;
    --) shift
        break ;;
    *) echo "$1 is not an option";;
    esac
    shift
done
#
count=1
for param in "$@"
do
    echo "Parameter #$count: $param"
    count=$[ $count + 1 ]
done
#
```

#### getopts
格式:`getopts optstring variable`,这里variable为命令行上检测到的第一个参数(getopts会去除'-',这点与getopt不同),处理完所有参数后,它会返回一个大于0的退出状态码。可以用":optstring"格式来忽略未识别选项的错误信息。会用到两个环境变量,如果选项用到一个参数值,OPTARG会保存该值,OPTIND环境变量保存了参数列表中getopts正在处理的参数位置。
getopts可以识别双引号内的带括号参数值,而getopt不可以。getopts将命令行上找到的所有未定义选项统一输成问号,getopt遇到未识别的非选项值时,会结束识别,即使后面有正确的选项。可以用以下脚本自行测试getopts的行为:
```bash
echo
while getopts :ab:cd opt
do
    case "$opt" in
        a) echo "Found the -a option"  ;;
        b) echo "Found the -b option, with value $OPTARG" ;;
        c) echo "Found the -c option"  ;;
        d) echo "Found the -d option"  ;;
        *) echo "Unknown option: $opt" ;;
    esac
done
#
shift $[ $OPTIND - 1 ]
#
echo
count=1
for param in "$@"
do
    echo "Parameter $count: $param"
    count=$[ $count + 1 ]
done
```
### set命令
set 指令可根据不同的需求来设置当前所使用 shell 的执行方式，同时也可以用来设置或显示 shell 变量的值。当指定某个单一的选项时将设置 shell 的常用特性，如果在选项后使用 -o 参数将打开特殊特性，若是 +o 将关闭相应的特殊特性。而不带任何参数的 set 指令将显示当前 shell 中的全部变量，且总是返回 true，除非遇到非法的选项。
参数说明：

|参数|说明|
|:-:||:-:|
|-a|标示已修改的变量，以供输出至环境变量|
|-b|使被中止的后台程序立刻回报执行状态|
|-d|Shell预设会用杂凑表记忆使用过的指令，以加速指令的执行。使用-d参数可取消|
|-e|若指令传回值不等于0，则立即退出shell|
|-f|取消使用通配符|
|-h|自动记录函数的所在位置|
|-k|指令所给的参数都会被视为此指令的环境变量|
|-l|记录for循环的变量名称|
|-m|使用监视模式|
|-n|测试模式，只读取指令，而不实际执行|
|-p|启动优先顺序模式|
|-P|启动-P参数后，执行指令时，会以实际的文件或目录来取代符号连接|
|-t|执行完随后的指令，即退出shell|
|-u|当执行时使用到未定义过的变量，则显示错误信息|
|-v|显示shell所读取的输入值|
|-H shell|可利用"!"加<指令编号>的方式来执行 history 中记录的指令|
|-x|执行指令后，会先显示该指令及所下的参数|
|+<参数>|取消某个set曾启动的参数。与-<参数>相反|
|-o [option]|特殊属性有很多,见下面|

**option属性**
- pipefail:管道流水线中有一个失败，则返回失败值，从右往左数起(默认只返回最后一个命令的退出码)
- noclobber:防止>重定向操作符覆盖已有内容，你可以使用`command >| file`强制覆盖

### Tips
* 
* **<<**和**<<-**的区别在于,<<-会忽略接下来输入的tab建,一般用于格式化脚本,便于读代码
* 组命令(在当前shell执行)--`{ command1;command2;command3  }`,子shell--`(command1;command2;command3)`一般配合管道符
* trap命令
`trap "命令" "信号"`当脚本遇到信号前执行的命令
```bash
trap "echo I am ignoring you" SIGINT SIGTERM
```
常见信号有:

|编号|英文名|含义|
|:-:|:-:|:-:|
|1|SIGHUP|挂起进程|
|2|SIGINT|终止进程|
|3|SIGQUIT|停止进程|
|9|SIGKILL|无条件终止进程|
|15|SIGTERM|尽可能终止进程|
|17|SIGSTOP|无条件停止进程,但不是终止进程|
|18|IGTSTP|停止或暂停进程,但不终止进程|
|19|SIGCONT|继续运行停止的进程|

    * tap除了捕获信号外,还会捕获脚本退出:`trap "echo Goodbye..." EXIT`,不管正常还是非正常,都会打印Goodbye
    * `trap -- SIGINT`取消某个信号的设置
	* `trap -`恢复信号的默认行为
* wait命令
`wait $pid`等待子进程
```bash
#小火车子进程
sl &
pid=$!
#父进程继续干事
sleep 2
#干完事等子进程
wait $pid
echo "子父进程都结束了"
```
* 命名管道
```bash
#创建一个命名管道文件
mkfifo pipe1
#子进程写入(写入完会阻塞,等待另一端读,所以不要wait)
ls -l > pipe1 &
#父进程读
sleep 5
cat<pipe1
```
* 子shell的全局环境变量改变并不会影响父shell,甚至用export也不行
* 命令替换`$(command)`,子shell`(command)`两个是不同概念,命令替换会开个子shell(貌似都会开子shell)
* 文件描述符与exec
    * 配合exec可以使标准输入输出永久重定向:`exec 2>testerror`重定向标准错误至文件。
    * exec可以创建文件描述:`exec 3>testxx;echo hello>&3`这可以用来恢复正常的输入输出,如下:
```bash
exec 3>&1
exec 1>test14out
echo "这会输入到test14out"
exec 1>&3
echo "这会输入到屏幕"
```
    * exec创建读写描述符:`exec 3<>testfile`,这要特别小心,任何读或写都会从文件指针的上次位置开始
    * exec关闭文件描述符:`exec 3>&-`
	* lsof命令可以查看已经打开的文件描述符,见Linux命令博客

<span id = "funAndArr"></span>
* 函数与数组
将数组变量当作但个参数传递的话,它不会其作用,只会传递第一个值,可以借鉴以下例子:
```bash
#!/bin/bash
# array variable to function test
function testit {
    local newarray
    newarray=`echo "$@"`
    echo "The new array value is: ${newarray[*]}"
}
myarray=(1 2 3 4 5)
echo "The original array is ${myarray[*]}"
testit ${myarray[*]}
```
* 预处理替换优先级
a.shell替换:文件通配符
b.变量替换
c.命令替换,如下示例:  

* shell元字符汇总
  
|符号|意义|
|:-:|:-:|
|空格、制表符| 命令行参数的分隔符|
|回车| 执行键入的命令|
|&lt; > &brvbar;| 重定向与管道|
|; |多个命令分隔符|
|& |后台运行|
|$ |引用shell的变量|
|\`| 命令替换,`\\`代表反斜线自身,`\``代表反撇号自身|
|\* [] ?| 文件通配符,不匹配\*和/|
|()| 用于定义shell函数或子shell中执行命令|
|\\| 转义字符取消元字符特殊含义，若不用于元字符跟不加一样|
|" "| 其中的内容除$和\`外取消元字符的特殊含义|
|' '| 取消所有元字符特殊含义|

* RANDOM是个内建的随机值
* `<<<`
here-string语法，允许直接传递字符串给标准输入
* eval
eval 的功能是将字符串作为代码来执行。看上去好像很简单，但实际涉及很复杂的内容，主要是符号转义导致的语义问题。
```
str1=str2
str2=abc
eval echo \$$str1
```
