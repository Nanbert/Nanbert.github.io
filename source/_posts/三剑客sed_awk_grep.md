---
title: '三剑客sed_grep_awk'
date: 2019-07-18 18:16:15
subtitle:
categories: 文本处理
tags:
cover:
---

1.  
sed为流编辑器,一次读取一行内容,并执行相应的命令,格式如下:
`sed [options] '[command]' (file)`
2.  
sed选项总结

|options|describe|example|
|:-:|:-:|:-:|
|-e|编辑随后的指令|`sed -e 's/brown/green/;s/dog/cat/' data.txt`|
|-f|跟随脚本中的文件名|
|-n|阻止输入行的自动输出|
|-r|支持扩展的正则表达式|
|-i|直接修改读取的文件内容,而不是由屏幕输出|

3.
sed地址表示方法

|地址|说明|
|:-:|:-:|
|n|行号,n是一个正整数|
|$|最后一行|
|/regexp/|正则表达式匹配行|
|addr1,addr2|从addr1到addr2范围内的文本行,包含地址addr2在内,地址是上述任意的地址形式|
|first~step|匹配由数字first代表的文本行,然后随后的每个在step间隔处的文本行。例如1~2代表奇数行|
|addr1,+n|匹配地址addr1和随后的n个文本行|
|addr!|匹配所有文本行,除了addr之外,addr是上述任意的地址形式|

8.  
替换的标志位

|flag|含义|
|:-:|:-:|
|n|1到512之间的一个数字,表示对文本模式中指定模式第n次出现的情况进行替换|
|g|对模式空间的所有出现的情况进行全局更改|
|p|打印模式空间的内容|
|W|将模式空间的内容写到文件file中|

替换部分的元字符:&表示命中的正则表达式,`\`后面可接回车,`\<n>`表示匹配的部分

9.如果模式中含有'/'可以用感叹号作定界符,如`s!/usr/mail!/usr2/mail!`
10.其他命令
a.追加[line-address(单个行)]a\
		\<text\>
b.插入[line-address(单个行)]i\
		\<text\>
c.更改[address(可以是范围)]c\
		\<text\>
以上三个命令text内容如果是多行,则行末要加反斜线,最后一行不需要,文本内容不影响模式空间,也不参与后面的脚本运行
d.列表[address]l;不仅显示内容,非打印字符显示为两个数字的ASCLL代码
e.[line-address]=;不打印内容,但打印行号
f.[address]n;读取输入的下一行,并且后续命令用于新行
g.[line-address]r <filename>;在匹配行之后,读入某个文件
h.[address]w <filename>;写入某个文件
i.[line-address]q;停止读入新行
j.[address]y/abc/xyz/;转换(类似于tr),有用的例子转换大小写y/abcdefghijklmnopqrstuvwxyz/ABCDEFGHIJKLMNOPQRSTUVWXYZ/
8.sed:反斜线圆括号用法
`s/\([0-9]\)-\([0-9]\)/\2-\1/g`指交换斜杠前后的两个数字
12.脚本中所有编辑命令都将依次应用于每个输入行
12.　a.`cat jane.txt | tr '[A-Z]' '[a-z]' | tr ';.?\047,():"-' ' '|tr ' ' '\012' | grep -v '^ *$' | sort | uniq -c | sort -n`:统计单词数,首先大写替换为小写，然后替换标点为空格,再替换空格为换行,去空行,排序，计数，再排序。
13.sed分组命令格式
```bash
/pattern/ {
动作１
动作２
}
```
14.  
多行模式空间:多行Next(N)命令通过读取新的输入行,并将它添加到模式空间的现有内容来创建多行模式空间。模式空间最初的内容和新的输入行之间用换行符分隔。在模式空间中嵌入的换行符可以用转义序列"\n"来匹配。在多行模式空间中,元字符"^"匹配空间中的第一个字条,而不匹配换行符后面的字符。同样,"$"只匹配模式空间中最后的换行符,而不匹配任何嵌入的换行符。在执行next命令之后,控制将被传递给脚本中的后续命令。
举例:例如要将"Owner and Operator Guide"换成"Installation Guide",对于处于不同行的情况可以这样做:
```bash
/Operator$/{
N
s/Owner and Operator\nGuide/Installation Guide/
}
```
上面比较不普遍,更普遍做法是:
```bash
s/Owner and Operator Guide/Installtion Guide/
/Owner/{
N
s/ *\n/ /
s/Owner and Operator Guide *Installation Guide\
/
}
```
15.  
删除命令(d)删除模式空间的内容并导致读入新的输入行,从而在脚本的顶端重新使用编辑方法。删除命令(D)稍微有些不同:它删除模式空间中直到第一个嵌入的换行符的这部分内容。他不会导致读入新的输入行,相反,**它返回到脚本的顶端**,将这些指令应用于模式空间的剩余内容。
例子:
```bash
/^ $/{
N
/^ \n$/D
}
```
以上可以删除连续多个空行,只保留一个空行
16.  
保持空间:当改变模式空间中的原始内容时,用于保留当前输入行的副本。影响模式空间的命令
|命令|缩写|功能|
|:-:|:-:|:-:|
|Hold|h或H|将模式空间的内容复制或追加到保持空间(大写追加)|
|Get|g或G|将保持空间的内容复制或追加到模式空间(大写追加)|
|Exchange|x|交换保持空间和模式空间的内容|

17.  
分支:在脚本中将控制转移到另一行
```bash
:top
command1
command2
/pattern/b top
command3
```
如果没有定义top(可以为任意命名),则将跳转到脚本末尾,上面例子意思是如果找不到pattern,才执行commmand3
测试:如果当前匹配行上成功进行了替换,那么test命令就会转到标签处,用法和分支一样,把b改成t即可
