---
title: AWK
date: 2021-01-24 17:08:06
subtitle:
categories:
tags:
cover: css/images/awk.png
---
1.awk选项总结

|options|describe
|:-:|:-:|
|-F|改变字段分割符,默认为空格|
|-f|跟随脚本中的文件名|
|-v|跟随var=value,作用域在BEGIN之前|

2.awk内置变量

|variable|describe|
|:-:|:-:|
|ARGV|命令行参数的数组,不包括脚本本身(-f选项也不包括),下标从0开始(一般为awk),一般大于0的下标都是输入的文件名,最后一个是ARGC-1|
|ARGC|ARGV数组个数|
|ENVIRON|环境变量数组,下标是环境变量名|
|FILENAME|当前输入文件名称|
|FNR|当前输入文件的记录(行)个数|
|FS|字段分隔符,最好在BEGIN的时候定义|
|NF|每段记录(即行)的字段数(即单词)|
|NR|行号|
|OFMT|数值的输出格式,默认为"%.6g"|
|OFS|输出字段分割符,默认为" "|
|ORS|输出的记录分割符,默认为"\n"|
|RLENGTH|被函数match匹配的字符串长度|
|RS|记录分隔符|
|RSTART|被函数match匹配的字符串的开始|
|SUBSEP|下标分割符|
|$0|整行内容|

3.awk内置函数

|算术函数|描述|
|:-:|:-:|
|cos(x)|返回x的余弦|
|exp(x)|返回e的x次幂|
|sin(x)|返回x的正弦|
|int(x)|返回x的整数部分的值|
|log(x)|返回x的自然对数(以e为底)|
|sqrt(x)|返回x的平方根|
|atan2(y,x)|返回y/x的反正切|
|rand()|返回0-1之间的随机数|
|srand(x)|建立rand()的新的种子数。如果没有指定种子数,就用当天的时间。返回旧的种子值|
|字符串函数|描述|
|gsub(r,s,t)|在字符串t中用字符串s替换和正则表达式r匹配的所有字符串。返回替换的个数。如果没有给出t,默认$0|
|index(s,t)|返回子串t在字符串s中的位置,如果没有指定s,则返回0|
|length(s)|返回字符串s的长度,当没有给出s时,返回$0的长度|
|match(s,r)|如果正则表达式r在s中出现,则返回出现的起始位置;如果没有,则返回0|
|spllit(s,a,sep)|使用字段分隔符sep将字符串s分解到数组a的元素中,返回元素个数。如果没有给出sep,则使用FS。数组分割和字段分隔采用相同的方式|
|sub(r,s,t)|在字符串t中用s替换正则表达式r的首次匹配。如果成功则返回1,否则返回0,,如果没有给出t,默认为$0|
|substr(s,p,n)|返回字符串s中从位置p开始最大长度为n的子串。如果没有给出n,返回从p开始剩余的字符串。|
|tolower(s)|将字符串s中的所有大写字符转换为小写,并返回新串|
|toupper(s)|将字符串s中的小写字符转换为大写,并返回新串|

4.模式汇总

|格式|含义|
|:-:|:-:|
|BEGIN {statements}|在输入被读取之前,statements执行一次|
|END {statements}|当所有输入被读取完毕之后,statements执行一次|
|expression {statements}|每碰到一个使expression为真的输入行,statements就执行.expression为真指的是其值非零或非空|
|/regular expression/ {statements}|当输入行有一段字符串可以被正则表达式匹配,则执行statements|
|compound pattern {statements}|一个复合模式将表达式用&&,||,!,以及括号组合起来;当复合模式为真时,statements执行|
|pattern1,pattern2 {statements}|一个范围模式匹配多个输入行,这些输入行从匹配pattern1的开始,到匹配pattern2的行结束(包括这两行),对这其中的每一行执行statements|

5.字符串匹配模式

|格式|含义|
|:-:|:-:|
|/regexpr/|当当前输入行包含一段能够被regexexpr匹配的子字符串时,该模式匹配|
|expression ~ /regexpr/|如果expression的字符串值包含一段能够被regexpr匹配的子字符串时,该模式被匹配|
|expression !~ /regexpr/|与上述相反|

6.awk处理多行记录,主要改变分隔符
`BEGIN { FS="\n";RS=""}`,RS代表空行
7.`awk 'script' x=1 test1 x=2 test2`,awk可以这样传递变量
8.awk中的数组不需要声明定义,都是关联数组
`delete array[subscript]`删除一个元素
9.BEGIN与END
特殊的模式BEGIN在第一个输入文件的第一行之前被匹配,END在最后一个输入文件的最后一行被处理之后匹配。
10.next与exit
next使awk抓取下一行,exit会使awk执行END,如果已经在END,则结束程序
11.函数
函数参数变量都是局部变量,其余的对于awk来说都是全局变量,如果函数体内出现某个变量,该变量未在参数列表里,它也是全局变量
12.输入分隔符
内建变量 FS 的默认值是 " ", 也就是一个空格符. 当 FS 具有这个特定值时, 输入字段按照 空格和 (或) 制表符分割, 前导的空格与制表符会被丢弃, 所以下面三行数据中, 其每一行的第 1 个字段都相同:
```bash
 field1
 	field1
field1 field2
```
然而, 当 FS 是其他值时, 前导的空格与制表符不会被丢弃.  把一个字符串赋值给内建变量 FS 就可以改变字段分隔符. 如果字符串的长度多于一个字 符, 那么它会被当成一个正则表达式. 当前输入行中, 与该正则表达式匹配的最左, 最长, 非空且
不重叠的子字符串变成字段分隔符, 举例来说,
`BEGIN { FS = ",[ \t]*|[ \t]+" }`
如果某个子串由一个后面跟着空格或制表符的逗号组成, 或者没有逗号, 只有空格与制表符, 那 么这个子串就是字段分隔符.  如果 FS 被设置成单个字符 (除了空格符), 那么这个字符就变成字段分隔符. 这个约定使得
把正则表达式元字符当作字段分隔符来用, 变得很容易:
FS = "|"
把 | 变成字段分隔符
<font color=#FF0000>不管FS的值是什么,换行符总是多行记录的字段分隔符之一 </font>,如果RS被设置成""(即空行为记录分割符),则默认的字段分隔符就是空格,制表及换行;如果FS是\n,则换行符既是唯一分隔符
13.输出

|格式|含义|
|:-:|:-:|
|print|将$0打印到标准输出|
|print expression,expression,...|打印各个expression,expression之间由OFS分开,由ORS终止|
|print expression,expression,...\>filename|输出至filename|
|print expression,expression,...\>filename|追加到filename,不覆盖之前内容|
|print expression,expression,... | command|输出作为命令command标准输入|
|close(filename),close(command)|断开print与filename(或command)之间的连接|
|system(command)|执行command;函数的返回值是command的退出状态|

printf主要可以指定格式,以上都能用printf替换,如:`printf(format,expression,expression,...) > filename`

14.getline函数
(待建)用到再说把
13.例子
a.将每一行的字段逆序打印
```bash
{
	for (i=NF;i>0;i=i-1) printf("%s ",$i)
	printf("\n")
}
```