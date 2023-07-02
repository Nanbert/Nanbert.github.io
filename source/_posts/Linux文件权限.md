---
title: Linux文件权限
date: 2021-10-20 20:41:40
subtitle:
categories:
tags:
index_img: /img/file_permission.jpg
banner_img: /img/file_permission.jpg
---
## 用户
### /etc/passwd文件
/etc/passwd包含了一些与用户有关的信息 以':'分割符依次为以下内容
* 登陆用户名
* 用户账户的UID(数字形式)
* 用户账户的组ID(GID)(数字形式)
* 用户账户的文本描述(备注字段)
* 用户HOME目录位置
* 用户默认shell
root的UID为0,1000以下的UID为系统服务账户预留,普通用户为1000以后

### /etc/shadow文件
/etc/shadow文件管理着各个用户的密码,** 最好不要擅自修改,可能会造成系统崩溃**,以冒号分隔符,有以下字段
* 与/etc/passwd 文件中的登录名字段对应的登录名
* 加密后的密码
* 自上次修改密码后过去的天数（自 1970 年 1 月 1 日开始计算）
* 多少天后才能更改密码
* 多少天后必须更改密码
* 密码过期前提前多少天提醒用户更改密码
* 密码过期后多少天禁用用户账户
* 用户账户被禁用的日期（用自 1970 年 1 月 1 日到当天的天数表示）
* 预留字段给将来使用

### useradd命令与useradd文件
`useradd`命令使用系统的默认配置存在/etc/default/useradd文件中,arch linux默认配置如下:

```bash
# useradd defaults file for ArchLinux
# original changes by TomK
GROUP=users
HOME=/home
INACTIVE=-1
EXPIRE=
SHELL=/bin/bash
SKEL=/etc/skel
CREATE_MAIL_SPOOL=no
```

含义如下:
* 新用户会被添加到 users 的公共组；
* 新用户的 HOME 目录将会位于/home/loginname；
* 新用户账户密码在过期后不会被禁用；
* 新用户账户未被设置过期日期；
* 新用户账户将 bash shell 作为默认 shell；
* 系统会将/etc/skel 目录下的内容复制到用户的 HOME 目录下；(一般用于一些bash或vim配置文件等)
* 系统不会为该用户账户在 mail 目录下创建一个用于接收邮件的文件。
最常用命令
`sudo useradd -m test`创建新HOME目录名为test。

### userdel
`sudo userdel -r test`删除用户test，-r选项表明删除test用户的HOME目录及邮件目录

### usermod
usermod命令很强大(基本可以替代接下来的修改命令)可以用来修改/etc/passwd中的大部分字段,常用选项如下:
* -c修改备注字段
* -e修改过期日期
* -g修改默认的登录组
* -l修改用户账户的登录名
* -L锁定账户,使用户无法登录
* -p修改账户的密码
* -U解除锁定,使用户能够登录
* `sudo usermod -G shared test`把test用户添加到组shared <span id = "usermod"></span>

### passwd和chpasswd
* `sudo passwd test`修改test用户的密码,如果只用passwd只会改变当前用户的密码,-e选项能强制用户下次登录时修改密码。
* chpasswd可以大量修改密码`sudo chpasswd < users.txt`能从users.txt中自动读取登录名和密码对(由冒号分割)列表

### chsh、chfn和chage
* `sudo chsh -s /bin/zsh test`快速修改默认的用户登录shell。必须全路径
* `sudo chfn test`为用户test添加备注字段,默认会用finger命令的输出作为备注字段,如果没装finger，会询问你。
* chage命令用来帮助管理用户的有效期
    * -d:设置上次修改密码到现在的天数
	* -E:设置密码过期的日期
	* -l:设置密码过期到锁定账户的天数
	* -m:设置修改密码之间最少要多少天
	* -W:设置密码过期前多久开始出现提醒信息
change命令的日期值可以用下面两种方式
	* YYYY-MM-DD
	* 代表从1970年1月1日起到该日期的天数

## 组
### /etc/group文件
与/etc/passwd文件类似,以冒号分割符有以下四个字段:
* 组名
* 组密码(用的不多,允许非成员通过它成为该组的成员)
* GID
* 属于该组的用户列表(**注意:文件中不一定列全了,一定要结合/etc/passwd来查看**)

### groupadd命令
`sudo groupad shared`创建新组,默认没有用户
见[usermod](#usermod)那节,为组添加用户

### groupmod
* -g:修改已有组GID`groupmod -g newGid groupName`
* -n:修改组名`groupmod -n newname oldname`

## umask
`umask`命令会输出4位数的掩码值,以**0022**为例(第一个0仅代表8进制的意思):
* 二进制: 000 010 010
* 取消新文件和新目录的组w权限和其他用户w权限
对照表

|Oct|Bin|File Mode|
|:-:|:-:|:-:|
|0|000|---|
|1|001|--x|
|2|010|-w-|
|3|011|-wx|
|4|100|r--|
|5|101|r-x|
|6|110|rw-|
|7|111|rwx|

问题是touch默认的权限是644(即rw-r--r--),怎么来的呢？首先文件有个全权限为666,目录的全权限为777,666把umask值的1位减去即是644

## 文件的属性
文件除了权限外,还有属性设置,也很有用
* chattr:为文件添加或删除属性`sudo chattr +i testfile`,`sudo chattr -i testfile`
* lsattr:查看文件的属性`sudo lsattr testfile`

可通过`man chattr`来查看可以设置哪些属性,下表列出常用属性

|字母|含义|
|:-:|:-:|
|i|文件不能被删除、改名、设置链接、也无法写入或新增数据|
|a|只能增加数据,而不能删除也不能修改数据|

## chmod
chmod [ugoa][+=-][rwxst] 文件名表
也可以用八进制数字设置:`chmod xxx file`,对应关系见上表

### 三个特殊权限:
* setuid(SUID)--仅作用于可执行文件,`chmod u+s program` 当应用到一个可执行文件(s标志会出现在拥有者的x权限上)，它把有效用户 ID 从真正的用户（实际运行程序的用户）设置成程序所有者的 ID。这种操作通常会应用到 一些由超级用户所拥有的程序。当一个普通用户运行一个程序，这个程序由根用户(root) 所有，并且设置了 setuid 位，这个程序运行时具有超级用户的特权，这样程序就可以 访问普通用户禁止访问的文件和目录。
b.setgid(SGID)--作用于可执行文件和目录(s标志会出现在组的x权限上) ,`chmod g+s dir` 把有效用户组 ID 从真正的用户组ID更改为文件所有者的组ID。如果设置了一个目录的 setgid 位，则目录中新创建的文件 具有这个目录用户组的所有权，而不是文件创建者所属用户组的所有权。对于共享目录来说， 当一个普通用户组中的成员，需要访问共享目录中的所有文件，而不管文件所有者的主用户组时， 那么设置 setgid 位很有用处。
c.sticky位(SBIT)--仅作用于目录(t标志会出现在其他的x权限上)`chmod o+t dir`,如果一个目录设置了sticky位，那么它能阻止用户删除或重命名文件,除非用户是这个目录的所有者,或者是文件所有者,或是超级用户。这个经常用来控制访问共享目录,比方说/tmp。
这3种权限也可以用另一套八进制来表示`chmod xxxx file`第一个数字是设置特殊权限的,这与umask的第一位**含义不同**

|Oct|Bin|File Mode|
|:-:|:-:|:-:|
|0|000|所有特殊位为0|
|1|001|sticky置位|
|2|010|SGID置位|
|3|011|SGID和sticky置位|
|4|100|SUID置位|
|5|101|SUID和sticky置位|
|6|110|SUID和SGID置位|
|7|111|所有位都置位|

### 可执行权限上的'S/T'(还有X有待研究)
没有执行权限的 UID/GID 和黏置位。小写的 s 与 t 都是取代 x 这个权限的，但是当下达 7666 权限时，也就是说,user,group 以及 others 都没有 x 这个可执行的标志时(因为是 666)，特殊权限位也不可能有权限执行，7666 的结果为-rwSrwSrwT。所以，这个 S, T 代表的就是“空的”执行权限，不具有执行权限。换个说法， SUID +s 是表示“该文件在执行的时候，具有文件拥有者的权限”，但是文件拥有者都无法执行时，也就不存在权限给其他人使用了。

## chown和chgrp
这两个命令除了root外,只有修改用户处于原用户组和新用户组才能修改
* `chown username.groupname filename`改变文件的用户和用户组,用户组名可选,当省略username,并在groupname前面加个`.`,则只改变用户组名,当省略groupname,在username后面加个`.`,则改变用户并把用户组改成与用户名同名的用户组
    * -R:递归目录
	* -h:该文件的所有链接文件也被改变所属关系
* `chgrp groupname filename`更改用户组
