---
title: Linux文件
date: 2019-07-18 16:28:04
subtitle:
categories: Linux
tags: 文件
cover:
---
1.  
/etc/resolv.conf:加快网速的

2.  
/etc/apt/sources.list:源列表
3.  
/etc/passwd:用户名位置
4.  
/etc/group:组名
5.  
/etc/sudoers:sudo权限设置
6.  
/etc/xinetd.d/:守护进程文件夹
7.  
/etc/inputrc:登录式bash的热键设置
8.  
登录式bash加载设置文件顺序:/etc/profile -> \~/.bash_profile(\~/.bash_login、\~/.profile) -> \~/.bashrc
9.  
非登录式bash加载设置文件顺序:\~/.bashrc -> /etc/bashrc -> /etc/profile.d/\*.sh  
这其中的/etc/profile.d/\*.sh里为bash操作界面、语系等
10.  
/dev/pts:伪终端设备的目录
/dev/:字符设备对应的节点
11.  
/proc/net/dev:网络终端接口
12.
/etc/mkinitcpio.conf:钩子的配置文件
13.
/etc/default/grub:设置当前系统的启动参数(仅deepin有)
/boot/grub/grub.cfg:设置所有系统的启动参数(arch的也可以设置,存在于deepin目录)
14.  
/proc/kallsyms:内核导出的符号表，第一列表示内核地址空间地址，第二列表示符号属性，第三段表示符号的字符串，也就是EXPORT_SYMBOL()导出的符号，第四列表示那些模块在使用这些符号
15.  
/proc/devices:字符设备,第一列数字代表主设备号,第二列则是设备内核模块名
/proc/(number):进程号的相关动态信息
16.  
超级块是对一个文件系统的描述
索引节点是对一个文件物理属性的描述
目录项是对一个文件逻辑属性的描述
一个进程所处的位置是由fs_struct来描述的,而一个进程(或用户)打开的文件是由files_struct来描述的,而整个系统所打开的文件是由file结构来描述的
17./etc/ld.so.conf.d/*.conf
里面是库文件所在目录，配置完用`ldconfig`更新一下
18./usr/share/doc目录下为各个安装软件的文档
19./etc/modprobe.d/sony-laptop.conf 索尼背光灯设置文件
