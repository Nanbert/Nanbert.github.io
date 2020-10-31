---
title: Linux命令
date: 2019-07-18 16:15:42
subtitle:
categories: Linux
tags: 命令
cover:
---

1.  
`grep -r "关键字" "目录"`
跳过二进制选项:- -binary-files=without-match
跳过某个目录:- -exclude-dir=(目录路径)
2.  
同时对某个目录下多个文件操作用{*,*}
3.  
`sudo apt --fix-broken install`或者`sudo apt install -f`
3.  
`sudo chown -R nanbert xx目录`
4.  
图形界面的sudo权限
`sudo nautilus`
5.  
搜索某个软件包 
`sudo apt-cache search <关键字>`
6.  
挂载某个镜像到/mnt
`sudo mount xx.iso -o loop /mnt`
7.  
pon dsl-provider:开
poff dsl-provider:关
sudo pppoeconf:启用设置
8.  
快捷键:
	ctrl+a:命令行首部
	ctrl+e:命令行尾部
	ctrl+k:删除光标后的字符
    ctrl+u:删除光标钱的字符
    ctrl+s:锁定屏幕
    ctrl+q:解锁
	ctrl+r:搜索词,再次按该组合键可以循环搜索,回车选中,ctrl+G不做任何操作返回终端
	!$:重新使用上一个命令中的最后一项(`alt+.`可以跳转上几次)
	!!:重用上个命令(sudo !!一般这么用)
9.   
单独命令间:
 ";":一直执行无论成功与否
 "&&":前面的命令执行成功才能执行后面
 "||":前面的命令执行失败才能执行后面
10.  
`kill %2`
2代表jobs命令显示下的任务号码,%区别于进程ID
11.  
export相当于把局部变量扩展至全局
12.  
`man 3 fork`
`man 2 syscalls`
|章节|内容|
|:-:|:-:|
|1|用户命令|
|2|程序接口内核系统调用|
|3|c库函数程序接口|
|4|特殊文件,比如说设备节点和驱动程序|
|5|文件格式|
|6|游戏娱乐,如屏幕保护程序|
|7|其他方面|
|8|系统管理员命令|
`-k <pattern>`在文档中搜索某个关键字,
13. 
后台运行
` nohup [命令参数] &`
14.ln命令：`ln source target` 硬链接，`ln -s source target`软连接。
两者的区别:
  1.软连接不增加文件的链接数，而硬链接则增加
  2.不能跨文件系统创建硬链接，硬链接不能连接目录
  3.硬链接的inode号完全相同，指向同一文件，软连接则有不同的inode号，并且新文件类型为专有软链接类型，其指向的文件内容存储着实际文件的路径
  4.硬链接指向的文件当链接数减为0时，才真正删除，软链接若实际文件删除，则软链接失效。
15.`nslookup github.global.ssl.fastly.Net`
`nslookup github.com`
查到的域名加到/etc/hosts里可以加快访问速度 
`sudo /etc/init.d/networking restart`刷新缓存
16.
systemctl查看/etc/init.d中哪些服务进程会在引导时启动  
`systemctl list-unit-files --type=service | grep enabled`  
查找废弃服务
`systemctl --all | grep not-found`
停止某个服务进程、开机禁止启动以及某个进程的状态  
`sudo systemctl stop xxx.service`  
`sudo systemctl disable xxx.service`  
`sudo systemctl status xxx.service`  
注:不能启用或禁用静态服务，这些服务为其他进程所依赖。
查看前一次启动的开机日志  
`journalctl -b -1`  
查看启动服务消耗时间  
systemd-analyze blame`
mask禁用某种服务
`systemctl mask xx.service`
17.  
U盘只读不可写问题
a.`df -h`查找挂载点
b.`sudo umount 位置`卸载U盘而不拔掉
c.`sudo dosfsck -v -a 文件系统(如:/dev/sdb1)`修复故障
d.`sudo mount 文件系统 挂载点`重新挂载
18.  
命令行复制到剪贴板
`cat filename | xsel -b`
19.  
plymouth更换主题,主题文件夹：/usr/share/plymouth/themes/,设置文件夹:/etc/plymouth/plymouthd.conf
`plymouth-set-default-theme -l`列出主题列表
`sudo vi /etc/plymouth/plymouthd.conf`更改Theme=的内容
`sudo mkinitcpio -p linux`生成新的镜像,重启生效
需要'quite splash'静默启动参数,在grub里设置
20.  
`grub-mkconfig` 刷新grub的配置
21.  
同个命令对应不同软件版本配置
`update-alternatives --install <link> <name> <path> <priority>`
`update-alternatives --install /usr/bin/arm-linux-gnueabi-gcc arm-linux-gnueabi-gcc /usr/bin/arm-linux-gnueabi-gcc-5 5`
`update-alternatives --config arm-linux-gnueabi-gcc`
23.  
内核模块相关
`modinfo *.ko`查看某个编译好的模块
`sudo insmod *.ko`装入模块
`dmesg` 查看内核打印信息
`lsmod` 显示已加载的模块
已加载的模块会在/sys/module/下建立对应文件夹
`sudo rmmod *.ko`卸载模块
24.  
`mknod /dev/demo_drv c 252　0`生成字符设备的对应节点,/dev/demo_drv代表节点位置名称,c代表字符设备,252为主设备号,0为次设备号
主设备号代表一类设备，次设备号代表同一类设备的不同个体，每个次设备号都有一个不同设备节点
25.  
自动挂载磁盘:
　a.`fdisk -l`查看可挂载磁盘
　b.`df -h`查看已挂载的磁盘
　c.`blkid`获取目标磁盘的uuid和属性
　d.`vi /etc/fstab`添加开机mount,格式:UUID=***** /home/nanbert/disk ext4 defaults 1 1  
26. 
手机USB线wifi共享：
　a.`ip addr show`查看USB网络借口
　b.`dhclient enpxxx`给借口设置ip地址，手机热点要开
27.od命令
`od -t x1 x.dat`以十六进制打印文件x.dat各字节
`od -c /bin/bash`逐字符方式打印文件,遇到不可打印字符打印编码
`echo \' | od -t o1`查看单引号的八进制
28.md5sum命令
`md5sum file1 file2`计算两个文件的md5值,相同文件md5值相同
29.行律问题
`ctty -a`查看行律配置
`ctty erase ^H`设置退格键
30.windows文件不兼容
原因:windows每行结束存储回车与换行,而Linux只存储换行
`unix2dos filename`转换成windows类型的文件,也有dos2unix命令
31.gbk中文字符转换为utf
`iconv -f utf-8 -t gbk`
`echo "汉字" | iconv -f utf-8 -t gbk | od -t x1`
32.`--`显示终止选项命令,如`rm -- -i`表示删除名为'-i'的文件
33.增量拷贝
`cp -u <...>`只拷贝时间戳更新的文件,相同文件不拷贝
`rsync`远程拷贝,相比于cp -u,它对改动很小的文件的操作更快
34.find命令  
　格式:find <范围> <条件> <动作>
　a.条件选项  
　　`-name '<filename>'`查找某个文件
　　`-regex <pattern>`整个路径名与pattern匹配
　　`-type <option>`
　　　option选项:
　　　f:普通文件 d:目录 l:符号链接文件
　　　c:字符设备文件 b:块设备文件 p:管道文件
　　`-size ±n单位`指定文件大小(大于+,等于,小于-);单位有c(字符),b(块,512字节),k,M,G,默认为b
　　`-mtime ±ndays`文件最近修改时间
　　`-newer file`文件最近修改时间比file还晚
　　`复合条件`:与-a;或-o;非!
　b.动作选项
　　`-print`打印文件的路径名
　　`-exec`在-exec及随后的分号(分号要用反斜线)之间的内容作为一条命令,其中{}代表遍历到的目标文件的路径名
　　`-ok`与-exec类似，但有确认选项
　c.例子
　　`sudo find /tmp -type d -empty`
　　`find ~ -perm /a=x`查找所有可执行文件
35.在任何命令前加`time`,可以计时运行时间
36.xargs命令,将标准输入构造为命令的命令行参数,如果命令行参数过多,会启动多个进程,与单一普通管道相比就是批处理
举例:`find src -name \*.c -print | xargs grep -n -- --help`
37.文件归档tar
　格式:tar [options] [destination] [target]
|options|作用|
|:-:|:-:|
|c|Create创建新磁带,存在则覆盖|
|t|Table列表,磁带文件名列表|
|d|磁带与文件系统的不同|
|r|添加文件至磁带末尾|
|u|只添加更新的文件|
|x|extract抽取,抽取文件|
|v|verbose冗长|
|f|必须指定设备文件名|
|z|gzip算法压缩|
|j|bzip2算法压缩|
b.举例:
不压缩创建: `tar cvf my.tar *.[ch] makefile`
提取文件至指定文件夹:`tar xvf tar_file -C destination_direectory`
38.chmod命令
chmod [ugoa][+=-][rwxst] 文件名表
39.umask命令
例子:掩码值:022
　　二进制: 000 010 010
　　取消新文件和新目录的组w权限和其他用户w权限
40.输入重定向
`cat < filename`打印名为filename的文件内容
`cat << END`接下来直到END之间的内容
`cat <<< filename`打印filename这个单词
`> 文件名`清空文件
tee:从Stdin读取数据，并同时输出到Stdout和文件
`ls /usr/bin | tee ls.txt |grep zip`
41.du命令  
　`du -sh * | sort -rh`查看当前目录下所有文件大小并排序
　`du -sh`统计当前目录大小
　`du -h --max-depth=1 | sort`查看当前所有一级子目录大小并排序

41.at命令一次性任务计划  
`systemctl start atd.service`启动服务
`at <时间点>`时间点格式举例:19:33、3pm+7 days、20:00 tomorrow
之后进入交互界面,输入命令,ctrl+D退出
`at -l`查看列表
`at -c 任务号`查看任务内容
`at -r 任务号`取消任务
/etc/at.deny,文件中的用户不能执行at(系统默认存在)
/etc/at.allow,默认不存在,只要存在的用户才能执行
42.crontab服务定时任务计划
`sudo systemctl start cronie.service`启动服务
`crontab -e`新增任务或编辑任务
格式如下:
`<minute> <hour> <day> <month> <week> <command>`
minute:0到59之间任何整数
hour:0到23之间任何整数
day:1到31之间的任何整数
month:1到12之间任何整数
week:1到7之间任何整数,
command:执行的命令
(*)代表所有可能值
(,)一个列表范围
(-)表示一个整数范围
(/)指定时间间隔的频率,比如"0-23/2"表示每两小时执行
`crontab -l`显示crontab文件
`crontab -r`删除crontab文件
`crontab -ir`删除crontab文件前提醒用户
43.run-parts一个接一个运行同一目录下的脚本
`run-parts<directory-path>`
`run-parts --list --regex  '^s.*sh$' <directory> `
44.ffmpeg
裁剪:`ffmpeg -i xx.mp4 -vcodec copy -acodec copy -ss 00:00:00 -to 01:18:08 output.mp4`
合并:
先建立个文本文档file,格式如下:
```
file '1.mp4'
file '2.mp4'
```
`ffmpeg -f concat -i file -c copy output.mkv`
或者支持不好先转换ts
`ffmpeg -i 1.mp4 -vcodec copy -acodec copy -vbsf h264_mp4toannexb 1.ts`
`ffmpeg -i 2.mp4 -vcodec copy -acodec copy -vbsf h264_mp4toannexb 2.ts`
`ffmpeg -i "concat:1.ts|2.ts" -acodec copy -vcodec copy -absf aac_adtstoasc output.mp4`
添加字幕:
`ffmpeg -i 2020-07-13\ 08-20-26.mkv -vf subtitles=test.srt -y output.mkv`
srt格式:
```
1
00:00:03,000 --> 00:00:06,000
Hi,I am Nanbert Don De Niro

2
00:00:06,000 --> 00:00:08,000
Hi,I am Donald Trump

3
00:00:08,444 --> 00:00:10,000
It's you,Assole!
```
45.
`type <command>`识别命令,有以下几种
　a.可执行命令，给出路径
　b.shell自身的命令(builtins)
　c.一个shell函数
　d.别名命令
46.
`which <command>'给出可执行程序的位置，对别名和内建命令无效
47.
`whatis <command>`会给出命令简短的说明
48.
`rm !(*.csv)`删除除了csv结尾的所有文件(貌似只有bash支持，zsh不支持)
