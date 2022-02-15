---
title: zsh
date: 2022-02-03 19:54:32
subtitle:
categories:
tags:
index_img: https://s4.ax1x.com/2022/02/03/HEjd6e.jpg
banner_img: https://s4.ax1x.com/2022/02/03/HEjd6e.jpg
---

### 变量定义
Zsh的变量除了哈希表以外，都直接赋值使用,包括整数(64位带符号)、浮点数(64位带符号)、字符串、数组、哈希表,`$+var`,判断变量是否定义,未定义返回0,否则为1

### 字符串变量
1) 获取字符串长度
`str=abcde echo $#str`
2) 字符串拼接
`str2+=$str1;str3=$str1$str2`
<span id = "slice"></span>
3) 字符串切片
字符位置从1开始算,bash风格的则从0开始算
`echo $str[2,4]`
4) 字符串截断
```bash
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
<span id = "find"></span>
5) 字符串查找
```bash
#从左往右找cd字符串,找不到返回数组大小+1
echo $str[(i)cd]
#从右往左找cd字符串,找不到返回0
echo $str[(I)cd]
#从第二个位置开始找
echo ${str[(in:2:)cd]}
```
6) 遍历字符
```bash
for i ({1..$#str}){
	echo $str[i]
}
```
7) 字符串替换
```bash
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
8) 字符串是否匹配
```bash
#是否包含
[[ $str1 == *$str2* ]] && echo good
#正则匹配
[[ $str =~ 'c[0-9]' ]]
```
9) 大小写转换
```bash
#转成大写
echo ${(U)str}
echo ${str:u}
#转成小写
echo ${(L)str}
echo ${str:l}
#首字母大写
echo ${(C)str}
```
10) 目录文件名截取
```bash
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
11) 相对路径转绝对路径
```bash
#功能相当于$(readlink -f $filepath)
filepath1=a.txt
echo ${filepath1:A}
```
12) 分割字符串
```bash
# 使用空格作为分隔符，多个空格也只算一个分隔符
str='aa bb cc dd'
echo ${str[(w)2]}#bb

# 指定分隔符
str='aa--bb--cc'
# 如果分隔符是 : 就用别的字符作为左右界，比如 ws.:.
echo ${str[(ws:--:)3]}#cc
```

<span id = "arrAndStr"></span>
* 转成数组
```bash
array=()${=str})#默认按空格分隔,可以设置IFS环境变量设置,也可以按以下方法
str="1:2::4"

#可以是多个字符,如**s/::/**,以**::**为分隔,同时也可以写成**.::.**,不必一定
#使用**/**,可以用任意符号
str_array=(${(s/:/)str})
echo $str_array #1 2 4 #忽略空字符串
# 保留其中的空字符串
str_array=("${(@s/:/)str}")
echo $str_array[3]#该值为空
echo $str_array[4]
str+=(1234)#字符串直接变成一个包含两个元素的数组
```
13) 读取文件内容到字符串
```bash
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
14) 读取进程输出到字符串
就是把\$(<filename)换城\$(cmd)

### 条件语句
```bash
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
|([a-c]\|<1-100>)|a和c之间的一个字符或者1和100之间的整数|

**加强版,要支持需要加上`setopt EXTENDED_GLOB`**

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

### 数组变量
1) 数组定义
```bash
array=(a "bb cc" dd)
echo $array #打印所有元素在一行
print -l $array #每行输出一个元素
```
2) 元素读写
```bash
echo $array[3]
echo $array[-1]
echo $#array #获取长度
array[3]=some
array[3]=() #删除元素
array+=eeee #添加元素
unset array #删除整个数组
```
3) 数组拼接
```bash
array1+=(e f g)
array1+=($array2) #小括号必须加,不加的话,则array2视为一个字符串
```
4) [字符串与数组](#arrAndStr)
5) 数组遍历
```
for i ($array1){
	echo $i
}
#同时遍历两个数组
for i ($array1 $array2){
	echo $i
}
```
6) [切片访问](#slice)
7) [元素查找](#find)
8) 元素排序
```bash
echo ${(o)array} #升序排列
echo ${(O)array} #降序排列
echo ${(oi)array} #大小写不敏感升序排列
echo ${(on)array} #按数字升序排列
echo ${(Oa)array} #反转数组元素
```
9) 去重
```bash
echo ${(u)array}
```
10) 构造连续字符或数值数组
```bash
array=(aa{bb,cc,11}) && echo $array #aabb aacc aa11
array=(aa{1..3}) && echo $array #aa1 aa2 aa3
array=(aa{15..19..2}) && echo $array #aa15 aa17 aa19
array=(aa{19..15..2}) && echo $array #aa19 aa17 aa15
array=(aa{01..03}) && echo $array #aa01 aa02 aa03
array=(aa{a..c}) && echo $array #aaa aab aac
array=(aa{Y..c}) && echo $array #ASCII码顺序
```
11) 从文件构造数组
```bash
# f 的功能是将字符串以换行符分隔成数组
# 双引号不可省略，不然会变成一个字符串，引号也可以加在 ${ } 上
array=(${(f)"$(<test.txt)"})
print -l $array
#a
#bb
#ccc
#dddd
# 不加引号的效果
array=(${(f)$(<test.txt)})
print -l $array
#a bb ccc dddd

# 从文件构造数组，并将每行按分隔符 : 分隔后输出所有列
for i (${(f)"$(<test.txt)"}) {
    array=(${(s.:.)i})
    echo $array[1,-1]
}
```
12) 从文件列表构造数组
```bash
array=(/usr/bin/vim*)
print -l $array
#/usr/bin/vim
#/usr/bin/vimdiff
#/usr/bin/vimtutor

# 要比 ls /usr/bin/[a-b]?? | wc -l 快很多
array=(/usr/bin/[a-b]??) && print $#array
```
13) 数组交集差集
```bash
# 两个数组的交集，只输出两个数组都有的元素,如果有重复元素不会去重
echo ${array1:*array2}
# 两个数组的差集，只输出 array1 中有，而 array2 中没有的元素
echo ${array1:|array2}
```
14) 数组交叉合并
```bash
# 从 array1 取一个，再从 array2 取一个，以此类推，一个数组取完了就结束
echo ${array1:^array2}
# 如果用 :^^，只有一个数组取完了的话，继续从头取，直到第二个数组也取完了
% echo ${array1:^^array2}
```
15) 对数组中的字符串进行统一处理
```bash
# :t 是取字符串中的文件名，可以用在数组上，取所有元素的文件名
print -l ${array:t}
# :e 是取扩展名，如果没有没有扩展名，结果数组中不会添加空字符串
print -l ${array:e}
# 字符串替换等操作也可以对数组使用，替换所有字符串
print -l ${array/a/j}
# :# 是排除匹配到的元素，类似 grep -v
print ${array:#a*}
# 前边加 (M)，是反转后边的效果，即只输出匹配到的元素，类似 grep
print ${(M)array:#a*}
# 多个操作可以同时进行，(U) 是把字符串转成大写字母
print ${(UM)array:#a*}

# 截断或对齐数组中的字符串
array=(abc bcde cdefg defghi)

# 只取每个字符串的最后两个字符
echo ${(l:2:)array}
bc de fg hi

# 用空格补全字符串并且右对齐
print -l ${(l:7:)array}
    abc
   bcde
  cdefg
 defghi

# 用指定字符补全
print -l ${(l:7::0:)array}
0000abc
000bcde
00cdefg
0defghi

# 用指定字符补全，第二个字符只用一次
print -l ${(l:7::0::1:)array}
0001abc
001bcde
01cdefg
1defghi

# 左对齐
print -l ${(r:7::0::1:)array}
abc1000
bcde100
cdefg10
defghi1
```
### 字典变量
1) 定义
```bash
typeset -A table
# 先声明,或者用 local，二者功能是一样的
local -A table

# 赋值的语法和数组一样，但顺序依次是键、值、键、值
table=(k1 v1 k2 v2)
#可以声明赋值一块,local -A table=(k1 v1 k2 v2)

# 直接用 echo 只能输出值
echo $table #v1 v2

# 使用 (kv) 同时输出键和值，(kv) 会把键和值都放到同一个数组里
% echo ${(kv)table} #k1 v1 k2 v2

# 哈希表的大小是键值对的数量
echo $#table
```
2) 读写
```bash
echo $table[k2]
table[k2]="v2"
# 删除元素的方法和数组不同，引号不能省略
unset "table[k1]"
```
3) 哈希表拼接
```bash
# 追加元素的方法和数组一样
table+=(k4 v4 k5 v5)
# 拼接哈希表，要展开成数组再追加
table1+=(${(kv)table2})
```
4) 哈希表遍历
```bash
# 只遍历值
for i ($table) {
echo $i
}
# 只遍历键
for i (${(k)table}) {
echo $i
}
# 同时遍历键和值
for k v (${(kv)table}) {
echo "$k -> $v"
}
```
5) 键是否存在
```bash
(($+table[k1]))
```
6) 元素排序
和数组类似,增加k、v两个选项
```bash
# 只对值排序
echo ${(o)table}

# 只对键排序
echo ${(ok)table}

# 键值放在一起排序
echo ${(okv)table}
```
7) 从字符串、文件构造哈希表
```bash
str="k1 v1 k2 v2"
local -A table=(${=str})
#从文件构造和数组类似
```
8) 对哈希表的每个元素统一处理
可参见数组,同时增加kv两个选项
```bash
#值转成大写
print ${(U)table}
#键转成大写
print ${(Uk)table}
#键值转成大写
print ${(Ukv)table}
# 排除匹配到的值
echo ${table:#v1}
# 只输出匹配到的键
echo ${(Mk)table:#k[1-2]}
```

### 数值类型
zsh通常不指定数值是整形还是浮点型,通常直接赋值，**虽然默认为字符串**,但作数值计算时自动判断,但可以如下指明类型,同时在双小括号里做c语言的任何符号计算,同时括号内变量可以不需要加$符号(貌似是zsh的一般特性,适用于许多其他场合)
```bash
integer i=123
float f=12.56
#(t)用于输出变量类型
echo ${(t)i} #integer
echo ${(t)f} #float
# 注意一旦指定了变量类型，类型就不会变了，除非再重新指定其他类型，或者用 unset 删除掉 
# 如果把浮点数赋值给整数变量，会取整
i=12.34
echo $i #会输出12
```
* 数学函数
zsh/mathfunc模块包含数学函数
```
zmodload -i zsh/mathfunc
echo $((sin(0)+ceil(14.4)))
```
**函数列表**

|函数名|功能|
|:-:|:-:|
|abs|取绝对值|
|ceil|向上取整|
|floor|向下取整|
|int|截断取整|
|float|转换成浮点数|
|sqrt|开平方|
|cbrt|开立方|
|log|自然对数|
|log10|常用对数|
|rand48|随机数|

还有:acos、acosh、asin、asinh、atan、atanh、cos、cosh、erf、erfc、exp、 expm1、fabs、gamma、j0、j1、lgamma、log1p、logb、sin、sinh、tan、 tanh、y0、y1、ilogb、signgam、copysign、fmod、hypot、nextafter、jn、 yn、ldexp、scalb

### 变量修饰语
**一般两种格式:**
* `${(x)var}` var是变量名,x是一个或多个字母,
* `${var:x}` var是变量名,x是一个或多个字母,或其他符号

**注意:加了修饰语的变量依然是变量,可以当普通变量处理,可以嵌套使用,$符号后不可以有空格**

|修饰符|举例|说明|
|:-:|:-:|:-:|
|:-|`echo ${var:-abc}`|如果变量有值,则输出原值,如果变量不存在、为空字符串、空数组等,则输出abc|
|-|`echo ${var-abc}`|如果变量有值,则输出原值,如果变量不存在,则输出abc|
|:=|`echo ${var:=abc}`|如果变量有值,则输出原值,如果变量不存在、为空字符串、空数组等,则输出abc并且赋值给var|
|::=|`echo ${var::=123}`|不管有没有值，村不存在，都输出123，并且重新赋值|
|:?|`echo ${var:?error}`|var没有值或不存在,则直接报错,否则输出原值|
|:+|`echo ${var:+123}`|如果var有值,则输出123,否则输出空|
|(F)|`echo ${(F)array}`|把数组中的元素以换行符拼接成字符串,不加任何修饰的话则是空格拼接|
|(j/x/)|`echo ${(j/-=/)array}`|把数组中的元素以-=两个字符连接|
|(s/x/)|`echo ${(s/==/)str}`|把字符串中的字符以==两个字符为分隔符分成数组|
|(t)|`echo ${(t)var}`|输出变量的类型:integer float scalar array association|
|`(P)`|`var=abc abc=123 echo ${(P)var}`|多重替换,输出123|
|[#n]|`echo $(([#16] 255))`|以n进制显示十进制整数|
|n#|`echo $((16#ff))`|显示n进制整数为十进制|

### 函数
与bash基本一致,增加了`unfunction fun`删除某个函数的功能,还有就是规避了`$*`和`$@`的区别,zsh推荐只用`$*`

### 替代find和ls
需要开启扩展通配符:setopt EXTENDED_GLOB
**通配符修饰语列表**

|名称|含义|使用样例或补充说明|
|:-:|:-:|:-:|
|/|目录||
|F|非空|/F(非空目录) /^F(空目录)|
|.|普通文件||
|@|符号链接||
|=|socket文件||
|p|FIFO 文件||
|\*|可执行的普通文件||
|%|设备文件||
|%b|块设备文件||
|%c|字符设备文件||
|r|文件拥有着有读权限||
|w|文件拥有着有写权限||
|x|文件拥有着有执行权限||
|A|文件拥有组用户有读权限||
|I|文件拥有组用户有写权限||
|E|文件拥有组用户有执行权限||
|R|任何用户都有读权限||
|W|任何用户都有写权限||
|X|任何用户都有执行权限||
|s|设置了setuid的文件||
|S|设置了setgid的文件||
|t|设置了粘滞位（sticky bit）的文件||
|f|符合指定的权限|f0644 f4755 f700|
|e||暂无|
|+|大于某个数|通常跟数字,与其他配合使用|
|d|指定设备号||
|l|硬连接个数|l-2（小于 2） l+3（大于 3）|
|U|当前用户拥有||
|G|当前用户所在组拥有||
|u|指定用户 id 拥有|u1000|
|g|指定用户组 id 拥有|g1000|
|a|指定文件的 atime|访问时间,默认单位是天,可跟单位M(月)、w(周)、h(小时)、m(分钟)、s(秒)、+(指定时间之前)、-(指定时间之内)|
|m|指定文件的 mtime|修改时间,可跟单位、+-,`print -l *(.mm+1)`|
|c|指定文件的 ctime|文件状态属性修改时间,可跟单位、+-,`print -l *(.cm+1)`|
|L|指定文件大小|默认单位是字节,单位有k、m和p(512字节的块),也可以大写,`print -l *(.Lm-1)`|
|^|取反|/^F|
|-|小于某个数|通常跟数字,与其他配合使用|
|M||暂无|
|T||暂无|
|N|如果没匹配到，返回空而不报错|
|D|包含隐藏文件（. 开头）|
|n|按数值大小排序|下文有说明|
|o|递增排序|下文有说明|
|O|递减排序|下文有说明|
|[n]|只取第 n 个文件|.[5]|
|[n1,n2]|取第 n1 到 n2 个文件|/[5,10]|
|:X||暂无|

1) 文件排序

可供排序的因子:n(文件名),L(大小),I(硬连接数),a(atime),m(mtime),c(ctime),d(所在目录深度,从深到浅)
```bash
# 按文件名排序，同一目录下的文件和目录名会一起排，而不是先排目录再排文件
#**/*,指当前目录和子目录
print -l **/*(.on)
bb.txt
cc/aa.txt
cc/dd.txt
zz.txt

# 按文件的目录深度逆序排，d 是从深往浅排，O 是逆序
print -l **/*(.Od)
zz.txt
bb.txt
cc/dd.txt
cc/aa.txt

# 先按文件名排序，然后再按大小排序，这样大小相同的文件依然是按文件名排的
print -l **/*(.onoL)
bb.txt
cc/aa.txt
cc/dd.txt
cc.txt
```
2) 组合使用

	类型和类型之间要用逗号个开,逗号前后内容互不干扰(取反^只影响到逗号之前的内容)

	`print -l *(/m-2,.Lm-3oL,@D)`

3) 批量重命名zmv
```bash
# 使用前需要先加载进来
autoload -U zmv

# 将所有 txt 文件扩展名改成 conf
# 参数要用单引号扩起来，$1 代表第一个参数中括号中的内容
 zmv '(*).txt' '$1.conf'

# 如果加了 -W 参数，zmv 会自动识别文件名中需要保留的部分
 zmv -W '*.txt' '*.conf'

# 调整文件名各部分的前后顺序
zmv '(*).(*).txt' '$2.$1.txt'
# 加 -n 预览而不实际运行
zmv -n '(*).(*).txt' '$2.$1.txt'
mv -- a.b.txt b.a.txt

# 0 1 2 ... 前添加 0，以便和 10 11 12 ... 宽度一致
zmv '([0-9]).(*)' '0$1.$2'
# 去掉开头的一个 0
zmv '(0)(*)' '$2'

# 文件整理到目录
zmv '(*) - (*) - (*).txt' '$1/$2 - $3.txt'

# 转换大小写
zmv '(*).txt' '${(U)1}.txt'
zmv '(*).txt' '${(L)1}.txt'
```
4) 不展开通配符
```bash
calc() {
    zmodload zsh/mathfunc
    echo $(($*))
}
calc 12+12
calc 12*12 #会报错
noglob calc 12*12 #可以
```
### local和typeset 
local和typeset基本一样(除了不能用-f和-g这两个选项)

|选项|含义|举例|
|:-:|:-:|:-:|
|-l|强制字符串内容为小写|`local -l str=abcABC`|
|-u|强制字符串内容为大写|`local -u str=abcABC`|
|-x|设置为环境变量|`export str=abc`等价于`local -x str=abc`|
|-r|只读|`local -r strl=abc`等价于`readonly str1=abc`|
|-U|设置数组不包含重复元素|`local -U array=(aa bb aa cc)`|
|-Z n|设置整数位数|不够用0不全,超过会被截断|
|-i n|设置整数为其他进制显示|支持2-36|
|`local {i,j,k}=123`|赋值多个变量为同一个值||
|-T|绑定字符串和数组|`local -T DIR dir`,DIR为字符串,dir为数组,以冒号连接,主要用于path变量|
|-p|显示变量的定义方式|显示脚本如何定义该变量的|

### 双引号问题
zsh不需要像bash那样频繁加双引号来避免错误。
zsh需要加双引号的场景:
1) 像这样的包含字符或者特殊符号的字符串 `"aa bb \t \n *"` 出现在代码中时，两边要加双引号
2) 在用`$()`调用命令时，如果希望结果按一个字符串处理，需要加上双引
3) 如果想将数组当单个字符串处理，需要加双引号，`array=(a b); print -l "$array"`
4) 其他的原本不是单个字符串的东西，需要转成单个字符串的场景，要加双引号

其余通常不加,其中典型场景:
1) 任何情况下，字符串变量的两边都不需要加双引号，无论里边的内容多么特殊，或者变量存不存在，都没有关系，如`$str`
2) 如果不转换类型（比如数组转成字符串），任何变量的两边都不需要加双引号
3) `$1 $2 $*`这些参数（其实它们也都是单个字符串），都不需要加双引号，无论内容是什么，或者参数是否存在。

### mapfile读写文件
```bash
zmodload zsh/mapfile

# 这样就可以创建文件并写入内容，如果文件存在则会被覆盖
mapfile[test.txt]="ab cd"
cat test.txt
#ab cd

# 判断文件是否存在
(($+mapfile[test.txt])) && echo good
#good

# 读取文件
echo $mapfile[test.txt]
#ab cd

# 删除文件
unset "mapfile[test.txt]"

# 遍历文件
for i (${(k)mapfile}) {
echo $i
}
#test1.txt
#test2.txt
```

### 循环语句
```bash
while [some condition]{
	break/continue
}
```
```
until [some condition]{
	break/continue
}
```
```bash
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
```bash
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
```bash
select i (aa bb cc){
	echo $i
}
```
必须加break,否则会一直让用户选择

### 异常处理语句
```bash
{
	语句1
} always{
	语句2
}
```
无论语句1是否出错,都执行语句2

### socket模块
```bash
# 监听连接端=======
# 首先要加载 socket 模块
zmodload zsh/net/socket

zsocket -l test.sock
listenfd=$REPLY
# 此处阻塞等待连接
zsocket -a $listenfd
# 连接建立完成
fd=$REPLY
# 然后 $fd 就可读可写
cat <&$fd

# 发起连接端==========
zmodload zsh/net/socket
zsocket test.sock
fd=$REPLY
echo good >&$fd

# 关闭监听端========
exec {listenfd}>&-
exec {fd}>&-
rm test.sock

# 关闭连接端======
exec {fd}>&-
```

### TCP模块
```bash
# 监听连接端=======
# 首先要加载 tcp 模块
zmodload zsh/net/tcp

ztcp -l 1234
listenfd=$REPLY
# 此处阻塞等待连接
ztcp -a $listenfd
# 连接建立完成
fd=$REPLY

# 然后 $fd 就可读可写
cat <&$fd

# 发起连接端===========
# 首先要加载 tcp 模块
zmodload zsh/net/tcp

ztcp 127.0.0.1 1234
# 连接建立完成
fd=$REPLY

# 然后 $fd 就可读可写
echo good >&$fd

# 关闭发起连接端===============
# fd 是之前存放 fd 号的变量
% ztcp -c $fd

# 关闭监听连接端=============
% ztcp -c $listenfd
% ztcp -c $fd
```
接受端例子:
```bash
#!/bin/zsh

zmodload zsh/net/tcp

(($+1)) || {
    echo "Usage: ${0:t} port"
    exit 1
}

ztcp -l $1
listenfd=$REPLY

[[ $listenfd == <-> ]] || exit 1

while ((1)) {
    ztcp -a $listenfd
    fd=$REPLY
    [[ $fd == <-> ]] || continue

    cat <&$fd
    ztcp -c $fd
}
```
发送端例子:
```bash
#!/bin/zsh

zmodload zsh/net/tcp

(($# >= 2)) || {
    echo "Usage: ${0:t} [hostname] port message"
    exit 1
}

if [[ $1 == <0-65535> ]] {
    ztcp 127.0.0.1 $1
} else {
    ztcp $1 $2
    shift
}

fd=$REPLY
[[ "$fd" == <-> ]] || exit 1

echo ${*[2,-1]} >&$fd
ztcp -c $fd
```

### 日期模块:zmodload zsh/datetime

|例子|含义|
|:-:|:-:|
|`echo $EPOCHSECONDS`|从光标1970年到现在的秒数,等价于`date +%s`|
|`echo $EPOCHREALTIME`|输出高精度当前时间戳|
|`echo $epochtime`|输出当前时间戳的秒和纳秒|
|`strftime "%Y-%m-%d %H:%M:%S (%u)" $EPOCHSECONDS`|按指定格式输出|
|`strftime -s str "%Y-%m-%d %H:%M:%S (%u)" $EPOCHSECONDS`|存到变量str中|
|`strftime -r "%Y-%m-%d %H:%M:%S (%u)" "2017-09-01 10:10:58 (5)"`|上述的反操作|

### gdbm模块--存在文件里的哈希表
```bash
% zmodload zsh/db/gdbm

# 声明数据库文件对应的哈希表
local -A sampledb
# 创建数据库文件，文件名是 sample.gdbm，对应 sampledb 哈希表
# 如果该文件已经存在，则会继续使用该文件
ztie -d db/gdbm -f sample.gdbm sampledb

# 然后正常使用 sampledb 哈希表即可，数据会同步写入到数据库文件中
sampledb[k1]=v1
sampledb+=(k2 v2 k3 v3)
echo ${(kv)sampledb}
#k1 v1 k2 v2 k3 v3

# 获取数据库文件路径
% zgdbmpath sampledb
echo $REPLY
#/home/goreliu/sample.gdbm

# 释放数据库文件
zuntie -u sampledb


# 也可以用只读的方式加载数据库文件
ztie -r -d db/gdbm -f sample.gdbm sampledb
# 但这样的话，需要用 zuntie -u 释放数据库文件
zuntie -u sampledb
```

### sched-->计划调度命令
zmodload zsh/sched
* `sched +5 ls`5秒后运行ls
* `sched`列出已有任务
* `sched -n`去除第n个待运行命令

### 模块简介
可以用man zshmodules查看模块功能
* zsh/system:底层文件读写
* zsh/pcre:正则表达式库
* zsh/stat:内部stat，取代stat
* zsh/zftp:内部ftp客户端
* zsh/zprof:性能追踪工具
* zsh/zpty:操作pty的命令
* zsh/zselect:select系统调用的封装

### TRAPINT
该函数名,捕获任意信号?有待研究,如下的代码竟然捕捉到SIGINT信号
```
#!/bin/zsh

# SIGINT 是 2 信号，ctrl + c 会触发
TRAPINT() {
    # 处理一些退出前的善后工作
    sleep 333
}

sleep 1000
```
### 代码风格
1) 统一使用4个空格来缩进
2) 非特殊场景,每行代码不超过100个字符
3) 在前一行尾部加一个空格和 \ 折行，折行后缩进一层（4 个空格）。
4) 如果缩进的是一个文本块，可以使用对齐缩进，也可以使用 4 个空格的固定缩进。
5) 如果是在 aa && bb || cc、[[ ]] 或者 (( )) 中折行，&& || 放在下一行的行首。
6) 在缩进和对齐之外的场景，不允许出现逻辑上不必要的连续多个空格。
7) + && | 等双元运算符左右要加一个空格。
8) ! ~等一元运算符和作用对象之间不加空格。
9) ( ) 和 (( )) { } 内侧不加空格，[[ ]] 因为语法需要，内侧加一个空格。
10) `;`之前不加空格，之后加一个空格。
11) 定义函数时（以及在 (( )) 中调用函数时），函数名和 ( 之间不加空格。
12) if while 等关键字和后边的内容之间加一个空格
13) if`[[ ]] {`等场景中，`{`和前边的内容之间加一个空格。
14) 变量和`[ ]`之间不加空格，用`[ ]`取数组或者哈希表值时，`[ ]`内侧不加空格。
15) `> <`等重定向符号和文件或者文件描述符之间不加空格。
17) 非特殊场景，不允许出现超过两个连续空行。
18) #!/bin/zsh 后加一个空行。
19) if while 等语句块之后加一个空行。
20) 定义函数后加一个空行。
21) 逻辑关系不强的两行（或者两块）代码之间，根据逻辑关系强弱（自行判断），加一个或两个空行。
22) 在判断条件的场景，不使用`[ ]`，用`[[ ]]`代替。
23) 在数值计算的场景，使用`$(( ))`而不是`$[ ]`。
24) 使用数值时，两端不加引号。
25) 用`$var`取变量值时，两边不加双引号，除非需要将非字符串变量转换成字符串。
26) 在非必须场景，不需要加`${var}`中的大括号。
27) 变量使用前要明确指明是局部变量（用 local 定义）还是全局变量（用 typeset -g 定义）。
28) 能用局部变量的地方全部使用局部变量（用 local 定义）。
29) 变量名中的单词可以使用下划线分隔或者驼峰风格，在不影响可读性的情况也可以使用全小写字母，但在同一个文件中要一致。
30) 字符串常量两端可以添加双引号或者单引号，但同一个文件中风格要一致。
31) 可以使用 name() 或者 function name() 定义函数，但同一个文件中风格要一致。
32) 非特殊场景，单个脚本文件不超过 1000 行。 

### bash和zsh的简明对比

|bash|zsh|说明|
|:-:|:-:|:-:|
|`"$var"`|`$var`|避免变量中有空格导致异常|
|`"$@"`|`$*`|避免变量中有空格导致异常|
|`"${array[@]}"`|`$array`|取数组所有元素，@ 可改成 *|
|`"${#array[@]}"`|`$#array`|取数组中元素个数，@ 可改成 *|
|`"${array[n - 1]}"`|`$array[n]`|取数组第 n 个元素，bash 从 0 开始，zsh 从 1 开始|
|`"$array"`|`$array[1]`|Bash 中的 $array 是取数组的第一个元素|
|`echo a*b`|`echo "a*b"`|Zsh 默认配置中，通配符如果匹配不到文件会报错|
|`if true; then :; fi`|`if true {}`|Zsh 中不需要使用 : 作为空语句|
|`[ "$var" == value ]`|`[[ $var == value ]]`|Zsh 中的 [ ] 里不支持 ==，一律用 [[ ]]|
`ls \| tee file \| less`|`ls >file \| less`|Zsh 中不需要用 tee 即可实现相同功能|

