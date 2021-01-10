---
title: Arch Linux问题解决
date: 2020-12-31 14:59:37
subtitle:
categories:
tags:
cover: css/images/fix.jpg
---
1.广通无线网卡问题
`lspci`查看那个shit一样的网卡名字"BCM43142"
`sudo pacman -S linux-headers`装这个东西
`sudo pacman -S broadcom-wl-dkms`安装这个屎样的驱动
你可能会说,没网下个屁啊
打开archwiki的package页面搜索包,点想要的包,在右边有个小字download from mirrors,点他下载离线包,sudo pacman -U 那个包
最新包可能出现问题,降级的时候要连linux那个包一块降级,linux和linux-headers版本号要对应上,
2.降级问题
最新包可能出现问题,先在/var/cache/pacman/pkg/那个目录下找缓存的文件
找不到再从网上找。
也可以用downgrade工具,没用过就不说它了
3.背光灯问题
/etc/modprobe.d/sony-laptop.conf 索尼背光灯设置文件
4.制作启动盘
直接用如下命令制作,deepin官网的也不行
`df`查看启动U盘的设备号
`sudo dd bs=4M if=deepin-desktop-community-1010-amd64.iso of=/dev/sdb status=progress && sync`

