---
title: AWK
date: 2021-01-24 17:08:06
subtitle:
categories:
tags:
index_img: https://z3.ax1x.com/2021/11/13/Iy56Ds.jpg
banner_img: https://z3.ax1x.com/2021/11/13/Iy56Ds.jpg
---
### awk选项总结

|options|describe
|:-:|:-:|
|-F|改变字段分割符,默认为空格|
|-f|跟随脚本中的文件名|
|-v|跟随var=value,作用域在BEGIN之前|
|-mf [N]|指定处理的数据文件中最大字段数|
|-mr [N]|指定处理的数据文件中最大行数|
|-W keyword|指定gawk的兼容模式或警告等级|

### awk内置变量

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

### awk内置函数

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
|sprintf(fmt,expr-list)|根据格式字符串fmt返回格式化后的expr-list,fmt格式见printf那里|
|index(s,t)|返回子串t在字符串s中的位置,如果没有指定s,则返回0|
|length(s)|返回字符串s的长度,当没有给出s时,返回$0的长度|
|match(s,r)|如果正则表达式r在s中出现,则返回出现的起始位置;如果没有,则返回0|
|spllit(s,a,sep)|使用字段分隔符sep将字符串s分解到数组a的元素中,返回元素个数。如果没有给出sep,则使用FS。数组分割和字段分隔采用相同的方式|
|sub(r,s,t)|在字符串t中用s替换正则表达式r的首次匹配。如果成功则返回1,否则返回0,,如果没有给出t,默认为$0|
|substr(s,p,n)|返回字符串s中从位置p开始最大长度为n的子串。如果没有给出n,返回从p开始剩余的字符串。|
|tolower(s)|将字符串s中的所有大写字符转换为小写,并返回新串|
|toupper(s)|将字符串s中的小写字符转换为大写,并返回新串|
### 模式汇总

|格式|含义|
|:-:|:-:|
|BEGIN {statements}|在输入被读取之前,statements执行一次,**注意BEGIN里一些内建变量可能为空(如FILENAME)**|
|END {statements}|当所有输入被读取完毕之后,statements执行一次|
|expression {statements}|每碰到一个使expression为真的输入行,statements就执行.expression为真指的是其值非零或非空|
|/regular expression/ {statements}|当输入行有一段字符串可以被正则表达式匹配,则执行statements|
|compound pattern {statements}|一个复合模式将表达式用&&,||,!,以及括号组合起来;当复合模式为真时,statements执行|
|pattern1,pattern2 {statements}|一个范围模式匹配多个输入行,这些输入行从匹配pattern1的开始,到匹配pattern2的行结束(包括这两行),对这其中的每一行执行statements|

### 字符串匹配模式

|格式|含义|
|:-:|:-:|
|/regexpr/|当当前输入行包含一段能够被regexexpr匹配的子字符串时,该模式匹配|
|expression ~ /regexpr/|如果expression的字符串值包含一段能够被regexpr匹配的子字符串时,该模式被匹配,返回该字符串,否则返回（待测试）|
|expression !~ /regexpr/|与上述相反|

### 正则和字符串中的转移序列

|序列|意义|
|:-:|:-:|
|`\a`|报警字符|
|`\b`|退格|
|`\f`|换页|
|`\n`|换行|
|`\r`|回车|
|`\t`|制表符|
|`\v`|垂直制表符|
|`\ddd`|八进制数ddd, ddd含有1到3个数字,每个数字的值在0到7 之间|
|`\xbex`|十六进制|

### 数组
awk中的数组不需要声明定义,都是关联数组
- 赋值一个数组`array[0]=1;array[1]=2`
- 删除一个元素`delete array[subscript]`
- `i in a`如果a[i]存在,则表达式为1,否则为0
- 遍历数组
```bash
#var是arr的每个下标
for (var in arr)
{
	statements
}
```
### 字符串
- 字符串的连接`"pre"str"suf"`不需要加号
### 自定义函数
自定义函数的参数为值传递，不改变遍历本身，但是数组例外,函数体内出现的任何变量都是全局变量，想要使用局部变量，只能放在参数列表中（参数列表中没有实际参数对应的参数都将作为局部变量使用）
```bash
function name(patameter-list){
	statements
}
```
### awk处理多行记录,主要改变分隔符
`BEGIN { FS="\n";RS=""}`,RS代表空行
### `awk 'script' x=1 test1 x=2 test2`,awk可以这样传递变量
### BEGIN与END
特殊的模式BEGIN在第一个输入文件的第一行之前被匹配,END在最后一个输入文件的最后一行被处理之后匹配。
### next与exit
next使awk抓取下一行,并返回到脚本底部;exit会使awk执行END,如果已经在END,则结束程序
### 输入分隔符
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
### print与printf

|格式|含义|
|:-:|:-:|
|print|将$0打印到标准输出|
|print expression,expression,...|打印各个expression,expression之间由OFS分开,由ORS终止|
|print expression,expression,...\>filename|输出至filename|
|print expression,expression,...\>filename|追加到filename,不覆盖之前内容|
|print expression,expression,... | command|输出作为命令command标准输入|
|close(filename),close(command)|断开print与filename(或command)之间的连接,同一命令想要两次之间毫无关联,必须先close|
|system(command)|执行command;函数的返回值是command的退出状态|

printf主要可以指定格式,以上都能用printf替换,如:`printf(format,expression,expression,...) > filename`末尾不会自动添加换行符
printf格式控制符(每一个格式说明符都以%开始,以转换字符结束)

|字符|表达式将被打印成|
|c|ASCII 字符|
|d|十进制整数|
|e|[-]d.dddddde[+-]dd|
|E|[-]d.ddddddE[+-]dd|
|f|[-]ddd.dddddd|
|g|照e或f进行转换, 选择较短的那个, 无意义的零会被抑制|
|G|照E或f进行转换, 选择较短的那个, 无意义的零会被抑制|
|i|十进制|
|O|无符号八进制数|
|s|字符串|
|x|无符号十六进制数|
|X|无符号十六进制数,字母大写|
|%|打印一个百分号 %, 不会有参数被吸收|

|修饰符|含义|
|:-:|:-:|
|-|表达式在它的域内左对齐|
|width|为了达到规定的宽度,必要时填充空格，前导0表示用0填充|
|.prec|字符串最大宽度, 或十进制数的小数部分的位数|

|fmt|$1|printf(fmt,$1)|
|:-:|:-:|:-:|
|%c|97|a|
|%d|97.5|97|
|%5d|97.5|  97|
|%e|97.5|9.750000e+01|
|%f|97.5|97.500000|
|%7.2f|97.5|  97.50|
|%g|97.5|97.5|
|%.6g|97.5|97.5|
|%o|97|141|
|%06o|97|000141|
|%x|97|61|
|%s|January|`|January|`|
|%10s|January|`|   January|`|
|%-10s|January|`|January   |`|
|%.3s|January|`Jan`|
|%10.3s|January|`|       Jan|`|
|%-10.3s|January|`|Jan      |`|
|%%|January|%|
### 管道的奇怪用法(好好理解到底啥是管道)
```bash
{
	#注意sort命令用引号括起来,当成字符串
	print xx | "sort -t'\t' +1rn"
	#上面的管道名就是"sort -t'\t' +1rn"
	close("sort -t'\t' +1rn")
}
```
假设有以下文件
```
France	211	55	Europe
Japan	144	120	Asia
Germany	96	61	Europe
England	94	56	Europe
USSR	8649	275	Asia
Canada	3852	25	North America
China	3705	1032	Asia
USA	3615	237	North America
Brazil	3286	134	South America
India	1267	746	Asia
Mexico	762	78	North America
```
以下程序会简单排序
```bash
# prep1 - prepare countries by continent and pop. den.
BEGIN { FS = "\t" }
{ 
	printf("%s:%s:%d:%d:%.1f\n",
	$4, $1, $3, $2, 1000*$3/$2) | "sort -t: -k 1,1 -k 5rn"
}
```

### getline函数
- 描述:函数getline可以从当前输入行,或文件,或管道,读取输入.getline抓取下一个记录,按照通常的方式把记录分割成一个个的字段.它会设置NF,NR,和FNR;如果存在一个记录,返回1,若遇到文件末尾,返回0,发生错误时返回-1(例如打开文件失败).
- `getline x`读取下一条记录到变量x中，并递增NR与FNR,不会对记录进行分隔，不会设置NF
- `getline <"file"`从文件file读取输入.它不会对NR与FNR产生影响, 但是会执行字段分割,并且设置NF.
- `getline x <"file"`从file读取下一条记录,存到变量x中.记录不会被分割成字段,变量NF,NR,与FNR都不会被修改.

|表达式|被设置的变量|
|:-:|:-:|
|getline|\$0,NF,NR,FNR|
|getline var|var,NR,FNR|
|getline \<file|\$0,NF|
|getline var\<file|var|
|`cmd|getline`|\$0,NF|
|`cmd|getline var`|var|

例子：
```bash
#必须判断大于0,否则文件不存在则就是死循环
while (getline x<file >0)
	print x
	next
while ("who"|getline)
	n++
```

13.例子
a.将每一行的字段逆序打印
```bash
{
	for (i=NF;i>0;i=i-1) printf("%s ",$i)
	printf("\n")
}
```
### awk的正则表达式
- awk的圆括号用法有点异样:`(r1)(r2)`若匹配xy,其中x匹配r1,y匹配r2。
  举例:`(Asian|European|North American) (male|female) (black|blue)bird`12种组合
### shell中包含awk
  见例子1
### 例子
```bash
# field -打印每个文件的指定字段顺序
# usage: field n n n ... file file file ...
awk '
#开始时,寻找数字,并使之清空,这样他们就不会被当成文件名
BEGIN {
for (i = 1; ARGV[i] ~ /^[0-9]+$/; i++) { # collect numbers
fld[++nf] = ARGV[i]
ARGV[i] = ""
}
if (i >= ARGC)
# no file names so force stdin
ARGV[ARGC++] = "-"
}
{
for (i = 1; i <= nf; i++)
printf("%s%s", $fld[i], i < nf ? " " : "\n")
}
' $*
```
```bash
#交互式awk脚本
BEGIN {
	maxnum = ARGC > 1 ? ARGV[1] : 10
	# default size is 10
	ARGV[1] = "-"# read standard input subsequently
	srand()# reset rand from time of day
	do {
		n1 = randint(maxnum)
		n2 = randint(maxnum)
		printf("%g + %g = ? ", n1, n2)
		while ((input = getline) > 0)
			if ($0 == n1 + n2) {
				print "Right!"
				break
			} else if ($0 == "") {
				print n1 + n2
				break
			} else
				printf("wrong, try again: ")
		} while (input > 0)
}
function randint(n) { return int(rand()*n)+1 }
```
- 去除字符串`gsub(/"([^"]|\\")*"/, "", line)`
- 去除正则表达式`gsub(/\/([^\/]|\\\/)+\//, "", line)`
- 去除注释`sub(/#.*/, "", line)`
