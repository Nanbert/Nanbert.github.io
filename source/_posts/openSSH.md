---
title: openSSH
date: 2020-12-21 20:30:23
subtitle:
categories:
tags:
cover: css/images/ssh.png
---
1.登录服务器
`ssh <user>@<hostname>`hostname是主机名,可以是域名,也可能是IP地址,不指定<user>@时,默认为客户端的当前用户名
`ssh -p 8821 foo.com`默认端口为21,可以用-p选项指定服务器的端口
2.文件信息
服务器公钥的指纹既是SSH服务器公钥的哈希值,每台SSH服务器都有唯一一对密钥
`ssh-keygen -l -f [*.pub]`查看某个公钥的指纹
ssh会将本机链接过的所有服务器公钥的指纹存储在"~/.ssh/known_hosts"文件中。每次链接服务器都会通过该文件判断是否为陌生服务器,如果是陌生的则会产生警告,如果输入yes忽略警告,将会自动添加该服务器的公钥指纹到该文件
`ssh-keygen -R <hostname>`用于删除某服务器的公钥(用于失效的情况),也可以手动删除known_hosts文件中的相关内容
3.执行远程命令
一般在登陆后,输入命令,
也可以一步到位`ssh [username@hostname] <command>`登陆成功后立即执行命令,如果命令是交互式的,则需要加上-t选项,如:`ssh -t foo.com vi foo.txt`

