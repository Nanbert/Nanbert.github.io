---
title: zsh
date: 2022-02-03 19:54:32
subtitle:
categories:
tags:
cover: https://s4.ax1x.com/2022/02/03/HEjd6e.jpg
---

### 变量定义
Zsh的变量除了哈希表以外，都直接赋值使用,包括整数、浮点数、字符串、数组、哈希表

### 字符串变量
1) 获取字符串长度
* `str=abcde echo $#str`
2) 字符串拼接
* `str2+=$str1;str3=$str1$str2`
3) 字符串切片
字符位置从1开始算,bash风格的则从0开始算
* `echo $str[2,4]`
4) 字符串截断
```
str=abcdeabcde
#删除左端匹配到的内容,最小匹配
echo ${str#*b} #输出cdeabcde
#删除右端匹配到的内容,最小匹配
echo ${str%d*} #输出abcdeabc
#删除左端匹配到的内容,最大匹配
echo ${str##*b} #输出cde
#删除右端匹配到的内容,最大匹配
echo ${str%%d*} #输出abc
```
5) 字符串查找
```
#从左往右找cd字符串,找不到返回数组大小+1
echo $str[(i)cd]
#从右往左找cd字符串,找不到返回0
echo $str[(I)cd]
```
6) 遍历字符
```
for i ({1..$#str}){
	echo $str[i]
}
```
7) 字符串替换
```
str=abcabc
#只替换找到第一个
echo ${str/bc/ef} #aefabc
#只删除找到第一个
echo ${str/bc} #aabc
#上面两个的所有版本
echo ${str//bc/ef}
echo ${str//bc}

str=abcABCabcABCabc

# /# 只从字符串开头开始匹配，${str/#abc} 也同理
echo ${str/#abc/123} #123ABCabcABCabc

# /% 只从字符串结尾开始匹配，echo ${str/%abc} 也同理
echo ${str/%abc/123} #abcABCabcABC123

str=abc
# 如果匹配到了则输出空字符串
echo ${str:#ab*}

# 如果匹配不到，则输出原字符串
echo ${str:#ab}

#按位置删除字符和替换字符,就是指定位置赋值
str[1]=
str[2,4]=
str[1]=k
str[2,4]=sjkg #可以不一一对应
```
8) 判断字符串是否存在
```
(($+str)) && echo good
#[[ "$str" == "" ]]只能判断内容是否为空,而不是未定义
```
9) 字符串是否匹配
```
#是否包含
[[ $str1 == *$str2* ]] && echo good
#正则匹配
[[ $str =~ 'c[0-9]' ]]
```
10) 大小写转换
```
#转成大写
echo ${(U)str}
echo ${str:u}
#转成小写
echo ${(L)str}
echo ${str:l}
#首字母大写
echo ${(C)str}
```
11) 目录文件名截取
```
filepath=/a/b/c.x

# :h 是取目录名，即最后一个 / 之前的部分，如果没有 / 则为 .
echo ${filepath:h} #/a/b

# :t 是取文件名，即最后一个 / 之后的部分，如果没有 / 则为字符串本身
echo ${filepath:t} #c.x

# :e 是取文件扩展名，即文件名中最后一个点之后的部分，如果没有点则为空
echo ${filepath:e} #x

# :r 是去掉末尾扩展名的路径
echo ${filepath:r} #/a/b/c
```
12) 相对路径转绝对路径
```
#功能相当于$(readlink -f $filepath)
filepath1=a.txt
echo ${filepath1:A}
```
13) 分割字符串
```
# 使用空格作为分隔符，多个空格也只算一个分隔符
str='aa bb cc dd'
echo ${str[(w)2]}#bb

# 指定分隔符
str='aa--bb--cc'
# 如果分隔符是 : 就用别的字符作为左右界，比如 ws.:.
echo ${str[(ws:--:)3]}#cc
```
* 转成数组
```
str="1:2::4"
str_array=(${(s/:/)str})
echo $str_array #1 2 4
echo $str_array[2]
echo $str_array[3]
# 保留其中的空字符串
str_array=("${(@s/:/)str}")
echo $str_array[3]#该值为空
echo $str_array[4]
```
14) 读取文件内容到字符串
```
# 比用 str=$(cat filename) 性能好很多
str=$(<filename)

# 比用 cat filename 性能好很多，引号不能省略
echo "$(<filename)"

# 遍历每行，引号不能省略
for i (${(f)"$(<filename)"}) {
    echo $i
}
# 小文件或者需要频繁调用时，尽量不要用 sed
#输出第2行
echo ${"$(<test.txt)"[(f)2]}
# 输出包含 “ang” 的第一行
echo ${"$(<test.txt)"[(fr)*ang*]}
```
15) 读取进程输出到字符串
就是把$(<filename)换城$(cmd)
### 条件语句
```
if [some condition]{
} elif{
} else{
}
```

### print和printf
zsh支持print和printf,`print -`再按tab健就可以查看所有选项,`printf %`再按tab键就可以查看所有格式化的东西

print支持的选项

|选项|功能|参数|
|:-:|:-:|:-:|
|-C|按列输出|列数|
|-D|替换路径成带~的版本|无|
|-N|使用\x00(null)作为字符串的间隔,默认是空格|无|
|-O|降序排列|无|
|-P|输出颜色和特殊样式|无|
|-R|模拟echo命令|无|
|-S|放命令放入了历史命令文件(要加引号)|无|
|-X|替换所有tab为空格|tab对应空格数|
|-a|和-c/-C一起使用时,改为从左到右|无|
|-b|识别出bindkey转义字符串|无|
|-c|按列输出(自动决定列数)|无|
|-f|同printf|无|
|-i|和-o/-O一起使用时,大小写不敏感排序|无|
|-l|使用换行符作为字符串分隔符|无|
|-m|只输出匹配的字符串|匹配模式字符串|
|-n|不自动添加最后的换行符|无|
|-o|升序排列|无|
|-r|不处理转义字符|无|
|-s|放命令放入历史命令文件(不要加引号)|无|
|-u|指定fd输出|fd号|
|-v|把内容保存到变量|变量名|
|-x|替换行首的tab为空格|tab对应空格数|
|-z|把内容放置到命令行编辑区|无|

* 颜色
```
# %B 加粗 %b 取消加粗
# %F{red} 前景色 %f 取消前景色
# %K{red} 背景色 %k 取消背景色
# %U 下滑线 %u 取消下滑线
# %S 反色 %s 取消反色
#
# black or 0  red     or 1
# green or 2  yellow  or 3
# blue  or 4  magenta or 5
# cyan  or 6  white   or 7

# 显示加粗的红色 abc
print -P '%B%F{red}abc'

# 没覆盖到的功能可以用原始的转义符号，可读性比较差
# 4[0-7] 背景色
# 3[0-7] 前景色
# 0m 正常 1m 加粗 2m 变灰 3m 斜体 4m 下滑钱 5m 闪烁 6m 快速闪烁 7m 反色

# 显示闪烁的红底绿字 abc
% print "\033[41;32;5mabc\033[0m"
```

### 通配符

|通配符|含义|
|:-:|:-:|
|*|任意数量的字符|
|?|任意一个字符|
|[abcd]|abcd中的任意一个字符|
|[^abcd]|除abcd中的任意一个字符|
|[a-c]|a和c之间的一个字符|
|[a-cB-Dxyz]|a和c之间、B和D之间以及xyz中的一个字符|
|<1-100>|1和100之间的整数|
|<-50>|0和50之间的整数|
|<100->|大于100的整数|
|-|任意正整数和0|
|([a-c]|<1-100>)|a和c之间的一个字符或者1和100之间的整数|

加强版,要支持需要加上`setopt EXTENDED_GLOB`
|通配符|含义|匹配的样式|
|:-:|:-:|:-:|
|^abc|除了 abc 外的任意字符串|aaa|
|abc^abc|以 abc 开头，但后边不是 abc 的字符串|abcabd|
|a\*c~abc|符合 a\*c 但不是 abc 的字符串|adc|
|a#|任意数量（包括 0）个 a|aaa|
|b##|一个或者多个b|b|
|(ab)##|一个或者多个ab|abab|
|(#i)abc|忽略大小写的abc|AbC|
|(#i)ab(#I)c|忽略大小写的 ab 接着 c|ABc|
|(#l)aBc| 和 c 忽略大小写，但 B 必须大写 的 aBc|aBC|
|(#a1)abc|最多错（多或缺也算）一个字符的 abc|a2c 或 ab 或 abcd|

### 循环语句
```
while [some condition]{
	break/continue
}
```
```
until [some condition]{
	break/continue
}
```
```
#样例
for i (aa bb cc){
	echo $i
}
array=(aa bb cc)
for i ($array) {
	echo $i
}
for ((i=0; i < 10; i++)){
	echo $i
}
for i ({1..10}){
	echo $i
}
```
```
repeat 5{
	echo good
}
```

### 分支语句
```
case $i {
	(a)
	echo 1
	;;

	(b)
	echo 2
	#继续执行下一个匹配的语句(不再进行匹配)
	;&

	(c)
	echo 3
	#继续向下匹配，看是否有满足条件的分支
	;|

	(c)
	echo 33
	;;

	(*)
	echo oher
	;;
}
```

### 用户输入选择语句
```
select i (aa bb cc){
	echo $i
}
```
必须加break,否则会一直让用户选择

### 异常处理语句
```
{
	语句1
} always{
	语句2
}
```
无论语句1是否出错,都执行语句2
