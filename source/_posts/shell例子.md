---
title: shell例子
date: 2019-07-20 17:04:05
subtitle:
categories: Linux
tags:
index_img:
---
* 追加信息于文件
`cat - >> filename`
* 赋值日期命令
`eval $(date '+weekday="%a" month="%b" day="%e" year="%G"')`
* 判断是否符合正则表达式
`expr string : pattern`
- 追加内容到文件
echo的-n选项不会在添加内容前加换行符
- 读取文件首行赋给变量
`read -r line < file`,-r选项保证读入的内容是原始内容，反斜杠不会发生转义，read命令会删除开头和结尾的`IFS`中的所有字符，如果想保留，把`IFS`置为空:`IFS= read -r line < file`
- 依次读入文件每一行
```bash
while IFS= read -r line; do
    # do something with $line
done < file
```
- 随机读取一行内容
`read -r random_line < <(shuf file)`
- 读取文件首行前三个字段
```bash
while read -r field1 field2 field3 _; do
    # do something with $field1, $field2, and $field3
done < file
```
`_`用来接受三个字段后的所有内容，如果没有他，field3会接受所有
- 从文件路径中获取文件名
`filename=${path##*/}`,这其实就是参数展开
- 从文件路径中获取目录名
`dirname=${path%/*}`
- 相同路径下的快速拷贝/移动写法
`cp /path/to/file{,_copy}`
- 生成a到z字母表
  - `echo {a..z}`
  - `printf "%c" {a..z}`,字母间不含空格
  - `printf "%c" {a..z} $'\n'`结尾加空行
  - `printf "%c\n" {a..z}`每个字符后加个空行
- 生成00-09数字
  - `printf "%02d" {0..9}`
  - `echo {00..09}`bash4以上支持
- 生成若干单词
`echo {w,t,}h{e{n{,ce{,forth}},re{,in,fore,with{,al}}},ither,at}`
- 重复输出10次字符串
`echo foo{,,,,,,,,,}`
- 分割字符串
  - `IFS=- read -r x y z <<< "$str"`
  - `IFS=- read -ra parts <<< "foo-bar-baz"`保存到数组
- 逐个字符处理字符串
```bash
while IFS= read -rn1 c; do
    # do something with $c
done <<< "$str"
```
-n1表示一次读一个字符
## 文件描述符与重定向
bash启动时，文件描述符表如下所示：
![](/img/file_descriptor.png)
当bash执行命令时，他会fork一个子进程，它会继承父进程的描述符表
### 重定向命令到stdout
- `command >file`到底发生了啥？
`>`是输出重定向操作符，bash首先打开文件准备写入，如果文件打开成功，command的stdout指向打开的文件，如果失败，不执行命令，其实等价于`command 1 >file`
### 重定向命令到stderr
`command 2>file`
### 重定向命令stdout和stderr到同一个文件
`command &>file`等价于`command >file 2>&1`
当只有一个重定向时，重定向位置可以任意放，甚至可以在命令的前面，但注意遇到多个重定向操作时,顺序很重要，会从左到右依次处理:  
首先`>file`会发生如下：
![](/img/file_descriptor1.png)
然后`2>&1`会发生如下：
![](/img/file_descriptor2.png)
如果顺序错了，为节省流量,你自己想会发生啥吧
### 重定向stdin
`command <file`Bash 在执行命令之前，打开文件file准备读入。如果打开文件出错，Bash 会直接返错，不会继续执行命令。相反如果打开成功，Bash 会使用打开的文件的文件描述符作为命令的标准输入
### 重定向一堆字符到stdin
这是最常见的here document语法`<<MARKER`,当bash遇到该操作符时，会从标输入读取每一行，直到遇到`MARKER`,例子：
```bash
sed 's|http://||' <<EOF
http://url1.com
http://url2.com
http://url3.com
EOF
```
### exec
见另一篇
### 通过bash访问web站点
```bash
exec 3<>/dev/tcp/www.google.com/80
echo -e "GET / HTTP/1.1\n\n" >&3
cat <&3
```
Bash 将/dev/tcp/host/port当作一种特殊的文件(套接字文件？)，它并不需要实际存在于系统中，这种类型的特殊文件是给 Bash 建立 tcp 连接用的。
### 重定向进程的stdout和stderr到另外一个进程的输入
`command1 |& command2`等价于`command1 2>&1|command2`
### 交换标准输出与标准错误
`command 3>&1 1>&2 2>&3`
### 重定向标准输出与错误给不同进程
主要就是使用进程替换
`command > >(stdout_cmd) 2> >(stderr_cmd)`
### 获取管道流中的所有命令执行退出码
```bash
echo 'pants are cool' | grep 'moo' | sed 's/o/x/' | awk '{ print $1 }'
echo ${PIPESTATUS[@]}
```
