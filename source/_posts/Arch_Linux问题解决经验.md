---
title: Arch Linux问题解决
date: 2020-12-31 14:59:37
subtitle:
categories:
tags:
index_img: https://z3.ax1x.com/2021/04/03/cnh3iq.jpg
banner_img: https://z3.ax1x.com/2021/04/03/cnh3iq.jpg
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
注意:如果u盘表现异常,一插有两个部分(一个部分只有几百兆),制作不会成功.也无法直接格式化,此时可以借助deepin官网启动盘制作,先格式化,再按上述操作
5.pacman更新过程中崩溃,断电
a.先制作启动盘(按上述方法)
b.assistant键进入,选择U盘启动(bios里外部设备启动要打开)
c.`iwctl`进入无线网配置
(1)查看无线设备
`device list`
(2)扫描无线网
`station wlan0 scan`
(3)列出网络名
`station wlan0 get-networks`
(4)根据设备(wlan0)连接wifi名称
`station wlan0 connect CMCC-A9wF`并输入密码(75ij3tw7)
(5)
exit退出
d.挂载根文件系统,proc,sys和dev
`mount /dev/sda[x] /mnt;mount -t proc proc /mnt/proc;mount --rbind /sys /mnt/sys;mount --rbind /dev /mnt/dev`
e.有锁的话,删除锁
`rm /mnt/var/lib/pacman/db.lck`要加/mnt绝对路径
f.查找损坏包
`pacman --sysroot /mnt -S $(pacman --sysroot /mnt -Qnq)`
损坏包会有`vim: /usr/xxx/xxx exists in filesystem`,这里vim就是损坏包
注意如果没有损坏包,就不要执行上述命令,因为他是重装所有包,这里只是检查以下
g.把上述命令重定向到一个文件,用vim修改成每一行如`rm -f /mnt/usr/xxx/xxx`
列模式修改(选中按c即可)':'前面的所有字符为`rm -f /mnt`(要加/mnt,绝对路径),全局替换掉所有'exists in filesystem'为空
h.`chmod +x`赋予那个重定向文件执行权限,`./(那个文件)`
i.更新或安装损坏包,
`pacman --sysroot /mnt -S (package)`
j.如果找不到镜像,重装下列包
`pacman --sysroot /mnt -S mkinitcpio systemd linux`
6.无线问题
`如果无线标志都没出来`网卡重装,回到本博客开头的第一个问题
`如果标志出来,收不到任何无线网`重装`wpa_supplicant`
7.wps界面不显示中文
卸载,用yay重装wps-office-cn版本,提示你可选依赖，装上所有可选依赖
8.
gdm进不去,重装gtk,gdm,gnome,xorg,gnome相关的全部删除
9.yay包降级
在wiki上找到那个包,在右边点击View Changes,选择想要的版本,点击下载,然后进入下载的解压缩文件,输入命令makepkg,sudo pacman -U xx.tar*
10.自动挂载手机
安装`gvfs-mtp`和`gvfs-gphoto2`
11.gnome的压缩软件->file-roller
12.lantern缺少的依赖->'libappindicator-gtk2/3'
