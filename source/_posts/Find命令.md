---
title: Find命令
date: 2020-11-18 18:40:05
subtitle:
categories:
tags:
cover: css/images/find.jpg
---
1.**格式** 
find <范围> <条件> <动作>
2.**查询表**

|测试条件|描述|
|:-:|:-:|
|-cmin [num]|匹配文件的最后修改时间正好在[num]分钟之前|
|-cnewer [fileName]|匹配文件的最后修改时间晚于file的文件或目录|
|-ctime [num]|匹配文件的最后修改时间在num\*24小时之前|
|-empty|匹配空文件和目录|
|-group [groupName]|匹配属于一个组的文件|
|-iname [pattern]|与-name一样，但是不区分大小写|
|-inum [nodeId]|匹配inode号是[nodeId]的文件,可用来查找硬链接|
|-mmin [num]|匹配内容修改于[num]分钟之前|
|-mtime [num]|匹配内容修改于[num\*24]小时之前|
|-name [pattern]|用指定的通配符模式匹配文件名|
|-newer [file]|匹配晚于[file]的文件|
|-newerct [yyyy-mm-dd HH:mm:ss]|创建晚于某个时间,详见man|
|-nouser|匹配不属于一个有效用户的文件|
|-nogroup|匹配不属于一个有效组的文件|
|-perm [mode]|匹配权限设置为[mode]的文件,mode可用八进制或符号|
|-samefile [filename]|类似于-inum测试条件,匹配和文件[fileName]享有同样inode号的文件|
|-size [num]|匹配大小为[num]的文件,数字后加单位,详情见其他表|
|-type [fileType]|匹配文件类型是[fileType]的文件|
|-user name|匹配属于某个用户文件|


(注:一般[num]都可以在前面加±号,表示大于或少于,如+5M表示(5M,+∞),-5M表示(0,4M],5M则表示(4M,5M]))


|字符|单位|
|:-:|:-:|
|b|512个字节快,为默认值|
|c|字节|
|w|两个字节的字|
|k|千字节(1024个字节单位)|
|M|兆字节(1048576个字节单位)|
|G|千兆字节(1073741824个字节单位)|


|操作符|描述|
|:-:|:-:|
|-and(-a)|如果操作符两边的测试条件都是真,则匹配,默认使用|
|-or(-o)|如果操作符两边的测试条件,任一是真,则匹配|
|-not(!)|如果操作符后面的条件为假,则匹配|
|()|用来改变优先级,注意一般shell对其有特殊解释,所以用引号或反斜杠加来转义|


|操作|描述|
|:-:|:-:|
|-delete|删除当前匹配的文件|
|-print|默认操作，打印|
|-ls|对匹配文件执行等同ls -dils命令|
|-quit|匹配到第一个即立刻退出|
|`-exec [command] '{}' ';'`|"{}"代表匹配到的文件(为决定路径),因为是shell特殊符号所以要用单引号,结尾必须用";",同样是特殊符号,要用单引号|
|`-ok [command] '{}' ';'`|与-exec相似，只不过执行前会进行确认|
|-regex|文件路径的正则匹配|


|文件类型|描述|
|:-:|:-:|
|b|块特殊设备文件|
|c|字符特殊设备文件|
|d|目录|
|f|普通文件|
|l|符号链接|


控制find命令的搜索范围


|选项|描述|
|:-:|:-:|
|-depth|指定find程序先处理目录中的文件,再处理目录自身(深度优先搜索?)。当指定-delete行为时,会自动用这个选项|
|-maxdepth [num]|设置陷入目录树的最大级别数|
|-mindepth [num]|设置陷入目录树的最小级别数|
|-mount|不搜索挂载到其他系统下的目录|
|-noleaf|在搜索DOS/Win文件系统或CD/ROMS时的时候优化选项|


3.**-exec与"+"**
`find -type f -name 'foo*' -exec ls -l '{}' ';'`
`find -type f -name 'foo*' -exec ls -l '{}' +`
上述两个命令的区别在于,假设匹配到fooA与fooB两个文件,第一个相当于执行两次ls命令,相当于`ls -l fooA`和`ls -l fooB`而第二个只执行一次`ls -l fooA fooB`

4.**xargs**
`find ~ -type f -name 'foo*' -print | xargs ls -l`与上述带有"+"的-exec选项效果一样,注意xargs有最大参数个数限制,可以通过`xargs -show-limits`来查看最大值
5.**文件名存在空格的问题**
类 Unix 的系统允许在文件名中嵌入空格(甚至换行符)。这就给一些程序,如
为其它程序构建参数列表的 xargs 程序,造成了问题。一个嵌入的空格会被看作是
一个分隔符,生成的命令会把每个空格分离的单词解释为单独的参数。为了解决这
个问题,find 命令和 xarg 程序允许使用一个可选的 null 字符作为参数分隔符。一
个 null 字符被定义在 ASCII 码中,由数字零来表示(相反的,例如,空格字符在
ASCII 码中由数字 32 表示)。find 命令提供的 -print0 行为,则会产生由 null 字符
分离的输出,并且 xargs 命令有一个 –null 选项,这个选项会接受由 null 字符分离
的输入。这里有一个例子
`find ~ -iname '*.jpg' -print0 | xargs -null ls -l`
6.**例子**
`sudo find /tmp -type d -empty`
`find ~ -perm /a=x`查找所有可执行文件
`find . -regex '.*[^-\_./0-9a-zA-Z].*'`查找不符合规范字符的路径名(中文咋办?)
