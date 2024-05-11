---
title: Linux命令_资源性能篇
date: 2024-04-26 03:52:39
tags:
---

# uptime
该命令输出如下：
```bash
23:51:26 up 21:31, 1 user, load average: 30.02, 26.43, 19.02
```
该平均负载指标了**要**运行的任务(进程)数量，包括要在CPU上运行的进程以及在不中断IO(磁盘IO)中阻塞的进程。
三个值分别代表了1min，5min,15min的平均值
# dmesg
操作系统消息日志
# free
![](https://linuxtools-rst.readthedocs.io/zh-cn/latest/tool/free.html)
# vmstat
`vmstat [-V] [-n] [delay [count]]`
- -V表示打印出版本信息；
- -n表示在周期性循环输出时，输出的头部信息仅显示一次；
- delay是两次输出之间的延迟时间；
- count是指按照这个时间间隔统计的次数。
## 输出信息
```bash
$ vmstat 1
procs ---------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
34  0    0 200889792  73708 591828    0    0     0     5    6   10 96  1  3  0  0
32  0    0 200889920  73708 591860    0    0     0   592 13284 4282 98  1  1  0  0
32  0    0 200890112  73708 591860    0    0     0     0 9501 2154 99  1  0  0  0
32  0    0 200889568  73712 591856    0    0     0    48 11900 2459 99  0  0  0  0
32  0    0 200890208  73712 591860    0    0     0     0 15898 4840 98  1  1  0  0
```
### Procs(进程)
- `r`:在CPU上运行并等待回合的进程数。由于它不包含IO，因此它比指示CPU饱和的平均负载提供了更多的信息。一个大于CPU核数的r值就是饱和的。
- `b`:等待IO的进程数量
### Memory(内存)
- free:空闲的内存(单位KB)
- swpd: 使用虚拟内存大小
- buff: 用作缓冲的内存大小
- cache: 用作缓存的内存大小
### Swap
- si: 每秒从交换区写到内存的大小
- so: 每秒写入交换区的内存大小
这两个非空，说明物理内存用完了，现在使用交换内存
### IO
- bi: 每秒读取的块数
- bo: 每秒写入的块数(每块1024bytes)
### system
- in: 每秒中断数，包括时钟中断
- cs: 每秒上下文切换数
### CPU
显示的是所有cpu的平均值
- `us、sy、id、wa、st`:这些是CPU时间的分类，
- us: 用户进程执行时间(user time)
- sy: 系统进程执行时间(system time)
- id: 空闲时间(包括IO等待时间)
- wa: 等待IO时间
- st: 被偷窃时间（被其它宾客系统进行使用，或宾客系统隔离的驱动程序域Xen）
通过将用户时间和系统时间这两个分类相加，即可判断CPU是否繁忙。一定的等待IO时间说明磁盘有可能是性能瓶颈。你可以认为等待IO时间是另一种形式的空闲时间，它提供了它是如何空闲的线索。 IO处理需要占用CPU系统时间。一个较高的CPU系统时间（超过20%）可能会很有趣，有必要进一步研究：也许内核在很低效地处理IO。
# mpstat
`mpstat -P ALL 1`：显示每个CPU的时间明细，用于检查不平衡状况
# iostat
`iostat [参数][时间][次数]`
这是了解块设备（磁盘），应用的工作负载和产生的性能影响的绝佳工具。
## 命令参数
-C 显示CPU使用情况
-d 显示磁盘使用情况
-k 以 KB 为单位显示
-m 以 M 为单位显示
-N 显示磁盘阵列(LVM) 信息
-n 显示NFS 使用情况
-p[磁盘] 显示磁盘和分区的情况
-t 显示终端和CPU的信息
-x 显示详细信息
-V 显示版本信息
## 输出
`iostat -xz 1`
```bash
Linux 3.13.0-49-generic (titanclusters-xxxxx)  07/14/2015  _x86_64_ (32 CPU)
avg-cpu:  %user   %nice %system %iowait  %steal   %idle
          73.96    0.00    3.73    0.03    0.06   22.21
Device:   rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
xvda        0.00     0.23    0.21    0.18     4.52     2.08    34.37     0.00    9.98   13.80    5.42   2.44   0.09
xvdb        0.01     0.00    1.02    8.94   127.97   598.53   145.79     0.00    0.43    1.78    0.28   0.25   0.25
xvdc        0.01     0.00    1.02    8.86   127.79   595.94   146.50     0.00    0.45    1.82    0.30   0.27   0.26
dm-0        0.00     0.00    0.69    2.32    10.47    31.69    28.01     0.01    3.23    0.71    3.98   0.13   0.04
dm-1        0.00     0.00    0.00    0.94     0.01     3.78     8.00     0.33  345.84    0.04  346.81   0.01   0.00
dm-2        0.00     0.00    0.09    0.07     1.35     0.36    22.50     0.00    2.55    0.23    5.62   1.78   0.03
```
### cpu属性值
- %user：CPU处在用户模式下的时间百分比。
- %nice：CPU处在带NICE值的用户模式下的时间百分比。
- %system：CPU处在系统模式下的时间百分比。
- %iowait：CPU等待输入输出完成时间的百分比。
- %steal：管理程序维护另一个虚拟处理器时，虚拟CPU的无意识等待时间百分比。
- %idle：CPU空闲时间百分比。
如果%iowait的值过高，表示硬盘存在I/O瓶颈，%idle值高，表示CPU较空闲，如果%idle值高但系统响应慢时，有可能是CPU等待分配内存，此时应加大内存容量。%idle值如果持续低于10，那么系统的CPU处理能力相对较低，表明系统中最需要解决的资源是CPU。
### disk属性值
- rrqm/s: 每秒进行 merge 的读操作数目。即 rmerge/s
- wrqm/s: 每秒进行 merge 的写操作数目。即 wmerge/s
- r/s: 每秒完成的读 I/O 设备次数。即 rio/s
- w/s: 每秒完成的写 I/O 设备次数。即 wio/s
- rsec/s: 每秒读扇区数。即 rsect/s
- wsec/s: 每秒写扇区数。即 wsect/s
- rkB/s: 每秒读K字节数。是 rsect/s 的一半，因为每扇区大小为512字节。
- wkB/s: 每秒写K字节数。是 wsect/s 的一半。
- avgrq-sz: 平均每次设备I/O操作的数据大小 (扇区)。
- avgqu-sz: 平均I/O队列长度。发给设备的平均请求数。值大于1可以表明已达到饱和状态（尽管设备通常可以并行处理请求，尤其是在多个后端磁盘所组成的前端虚拟设备的情况下）。
- await: 平均每次设备I/O操作的等待时间 (毫秒)。这是应用程序所感受到的时间，它包括IO排队时间和IO服务时间。大于预期的平均时间可能表示块设备饱和或设备出现问题了。
- svctm: 平均每次设备I/O操作的服务时间 (毫秒)。
- %util: 一秒中有百分之多少的时间用于 I/O 操作，即被io消耗的cpu百分比，设备利用率。这是一个表征繁忙度的百分比，它表示设备每秒工作的时间。尽管它的值取决于设备，但值大于60%通常会导致性能不佳（也会通过await的值观察到）。接近100␐%的值通常表示饱和。
### 信息意义
- 如果%iowait的值过高，表示硬盘存在I/O瓶颈。
- 如果 %util 接近 100%，说明产生的I/O请求太多，I/O系统已经满负荷，该磁盘可能存在瓶颈。
- 如果 svctm 比较接近 await，说明 I/O 几乎没有等待时间；
- 如果 await 远大于 svctm，说明I/O 队列太长，io响应太慢，则需要进行必要优化。
- 如果avgqu-sz比较大，也表示有大量io在等待。
# sar
sar是System Activity Reporter（系统活动情况报告）的缩写。sar工具将对系统当前的状态进行取样，然后通过计算数据和比例来表达系统的当前运行状态。需要启动service(systemctl start sysstat.service),并保存相关日志到文件
## 参数
- -A 汇总所有的报告
- -a 报告文件读写使用情况
- -B 报告附加的缓存的使用情况
- -b 报告缓存的使用情况
- -c 报告系统调用的使用情况
- -d 报告磁盘的使用情况
- -g 报告串口的使用情况
- -h 报告关于buffer使用的统计数据
- -m 报告IPC消息队列和信号量的使用情况
- -n 报告命名cache的使用情况
- -p 报告调页活动的使用情况
- -q 报告运行队列和交换队列的平均长度
- -R 报告进程的活动情况
- -r 报告没有使用的内存页面和硬盘块
- -u 报告CPU的利用率
- -v 报告进程、i节点、文件和锁表状态
- -w 报告系统交换活动状况
- -y 报告TTY设备活动状况
## example
### `sar -n DEV 1`
```bash
Linux 3.13.0-49-generic (titanclusters-xxxxx)  07/14/2015     _x86_64_    (32 CPU)
12:16:48 AM     IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s   %ifutil
12:16:49 AM      eth0  18763.00   5032.00  20686.42    478.30      0.00      0.00      0.00      0.00
12:16:49 AM        lo     14.00     14.00      1.36      1.36      0.00      0.00      0.00      0.00
12:16:49 AM   docker0      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
12:16:49 AM     IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s   %ifutil
12:16:50 AM      eth0  19763.00   5101.00  21999.10    482.56      0.00      0.00      0.00      0.00
12:16:50 AM        lo     20.00     20.00      3.25      3.25      0.00      0.00      0.00      0.00
12:16:50 AM   docker0      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
```
此命令可以检查网络接口的吞吐量：rxkB/s和txkB/s，作为工作负载的度量，还可以检查是否已达到网络接口的限制。在上面的示例中，eth0接收速率达到22MB/s，即176Mbit/s（远低于1Gbit/s的网络接口限制，假设是千兆网卡）。
### `sar -n TCP,ETCP 1`
```bash
Linux 3.13.0-49-generic (titanclusters-xxxxx)  07/14/2015    _x86_64_    (32 CPU)
12:17:19 AM  active/s passive/s    iseg/s    oseg/s
12:17:20 AM      1.00      0.00  10233.00  18846.00
12:17:19 AM  atmptf/s  estres/s retrans/s isegerr/s   orsts/s
12:17:20 AM      0.00      0.00      0.00      0.00      0.00
```
这是一些关键的TCP指标的摘要，包括：
- `active/s`：每秒本地启动的TCP连接数（例如，通过connect（））。
- `passive/s`：每秒远程启动的TCP连接数（例如，通过accept（））。
- `retrans/s`：每秒TCP重传的次数。
主动和被动计数通常作为服务器TCP负载的粗略度量：新接受的连接数（被动）和新出站的连接数（主动）。将主动视为出站，将被动视为入站可能对理解这两个指标有些帮助，但这并不是严格意义上的（例如，考虑从localhost到localhost的连接）。
重新传输是网络或服务器问题的迹象；它可能是不可靠的网络（例如，公共Internet），也可能是由于服务器过载并丢弃了数据包。上面的示例仅显示每秒一个新的TCP连接。
### `sar -u 1 2`
cpu使用率
- %user 用户模式下消耗的CPU时间的比例；
- %nice 通过nice改变了进程调度优先级的进程，在用户模式下消耗的CPU时间的比例
- %system 系统模式下消耗的CPU时间的比例；
- %iowait CPU等待磁盘I/O导致空闲状态消耗的时间比例；
- %steal 利用Xen等操作系统虚拟化技术，等待其它虚拟CPU计算占用的时间比例；
- %idle CPU空闲时间比例；

### `sar -q 1 2`
cpu平均负载
- runq-sz：运行队列的长度（等待运行的进程数）
- plist-sz：进程列表中进程（processes）和线程（threads）的数量
- ldavg-1：最后1分钟的系统平均负载 ldavg-5：过去5分钟的系统平均负载
- ldavg-15：过去15分钟的系统平均负载
### `sar -r 1 2`
内存情况
- kbmemfree：这个值和free命令中的free值基本一致,所以它不包括buffer和cache的空间.
- kbmemused：这个值和free命令中的used值基本一致,所以它包括buffer和cache的空间.
- %memused：物理内存使用率，这个值是kbmemused和内存总量(不包括swap)的一个百分比.
- kbbuffers和kbcached：这两个值就是free命令中的buffer和cache.
- kbcommit：保证当前系统所需要的内存,即为了确保不溢出而需要的内存(RAM+swap).
- %commit：这个值是kbcommit与内存总量(包括swap)的一个百分比.
### `sar -W 1 2`
页面交换情况
pswpin/s：每秒系统换入的交换页面（swap page）数量
pswpout/s：每秒系统换出的交换页面（swap page）数量
### summary
- 怀疑CPU存在瓶颈，可用 sar -u 和 sar -q 等来查看
- 怀疑内存存在瓶颈，可用sar -B、sar -r 和 sar -W 等来查看
- 怀疑I/O存在瓶颈，可用 sar -b、sar -u 和 sar -d 等来查看
# ipcs
- `ipcs -m`:共享内存资源
- `ipcs -q`:队列资源
- `ipcs -s`:信号量资源
- `ipcs -l`:类似于ulimit,列出系统资源限制
## 修改ipc资源限制
```bash
$cat /etc/sysctl.conf
# 一个消息的最大长度
kernel.msgmax = 524288

# 一个消息队列上的最大字节数
# 524288*10
kernel.msgmnb = 5242880

#最大消息队列的个数
kernel.msgmni=2048

#一个共享内存区的最大字节数
kernel.shmmax = 17179869184

#系统范围内最大共享内存标识数
kernel.shmmni=4096

#每个信号灯集的最大信号灯数 系统范围内最大信号灯数 每个信号灯支持的最大操作数 系统范围内最大信号灯集数
#此参数为系统默认，可以不用修改
#kernel.sem = <semmsl> <semmni>*<semmsl> <semopm> <semmni>
kernel.sem = 250 32000 32 128
```
修改保存后使用sysctl -p
## 清除IPC资源
- ipcrm -M shmkey  移除用shmkey创建的共享内存段
- ipcrm -m shmid    移除用shmid标识的共享内存段
- ipcrm -Q msgkey  移除用msqkey创建的消息队列
- ipcrm -q msqid  移除用msqid标识的消息队列
- ipcrm -S semkey  移除用semkey创建的信号
- ipcrm -s semid  移除用semid标识的信号
## example
- 清除当前用户创建的所有的IPC资源
```bash
ipcs -q | awk '{ print "ipcrm -q "$2}' | sh > /dev/null 2>&1;
ipcs -m | awk '{ print "ipcrm -m "$2}' | sh > /dev/null 2>&1;
ipcs -s | awk '{ print "ipcrm -s "$2}' | sh > /dev/null 2>&1;
```
# fuser
显示所有正在使用着指定file,file system或sockets的进程信息
`fuser -m -u redis-server`用来查找所有正在使用redis-server的所有进程的PID以及该进程的OWNER
`fuser –k /path/to/your/filename`:kill所有正在使用某一指定的file, file system or sockets的进程
# top(htop更强大的替换工具)
## 输出信息
```bash
$top
    top - 09:14:56 up 264 days, 20:56,  1 user,  load average: 0.02, 0.04, 0.00
    Tasks:  87 total,   1 running,  86 sleeping,   0 stopped,   0 zombie
    Cpu(s):  0.0%us,  0.2%sy,  0.0%ni, 99.7%id,  0.0%wa,  0.0%hi,  0.0%si,  0.2%st
    Mem:    377672k total,   322332k used,    55340k free,    32592k buffers
    Swap:   397308k total,    67192k used,   330116k free,    71900k cached
    PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND
    1 root      20   0  2856  656  388 S  0.0  0.2   0:49.40 init
    2 root      20   0     0    0    0 S  0.0  0.0   0:00.00 kthreadd
    3 root      20   0     0    0    0 S  0.0  0.0   7:15.20 ksoftirqd/0
    4 root      RT   0     0    0    0 S  0.0  0.0   0:00.00 migration/0
```
以上为例子
### 第一行
- 09:14:56 ： 系统当前时间
- 264 days, 20:56 ： 系统开机到现在经过了多少时间
- 1 users ： 当前2用户在线
- load average: 0.02, 0.04, 0.00： 系统1分钟、5分钟、15分钟的CPU负载信息
### Tasks
- 87 total：很好理解，就是当前有87个任务，也就是87个进程。
- 1 running：1个进程正在运行
- 86 sleeping：86个进程睡眠
- 0 stopped：停止的进程数
- 0 zombie：僵死的进程数
### Cpu(s)
- 0.0%us：用户态进程占用CPU时间百分比，不包含renice值为负的任务占用的CPU的时间。
- 0.7%sy：内核占用CPU时间百分比
- 0.0%ni：改变过优先级的进程占用CPU的百分比
- 99.3%id：空闲CPU时间百分比
- 0.0%wa：等待I/O的CPU时间百分比
- 0.0%hi：CPU硬中断时间百分比
- 0.0%si：CPU软中断时间百分比
注：这里显示数据是所有cpu的平均值，如果想看每一个cpu的处理情况，按1即可；折叠，再次按1；
### Men：内存
- 8175320kk total：物理内存总量
- 8058868k used：使用的物理内存量
- 116452k free：空闲的物理内存量
- 283084k buffers：用作内核缓存的物理内存量
### Swap：交换空间
- 6881272k total：交换区总量
- 4010444k used：使用的交换区量
- 2870828k free：空闲的交换区量
- 4336992k cached：缓冲交换区总量
### 进程信息
- PID：进程的ID
- USER：进程所有者
- PR：进程的优先级别，越小越优先被执行
- NInice：值
- VIRT：进程占用的虚拟内存
- RES：进程占用的物理内存
- SHR：进程使用的共享内存
- S：进程的状态。S表示休眠，R表示正在运行，Z表示僵死状态，N表示该进程优先值为负数
- %CPU：进程占用CPU的使用率
- %MEM：进程使用的物理内存和总内存的百分比
- TIME+：该进程启动后占用的总的CPU时间，即占用CPU使用时间的累加值。
- COMMAND：进程启动命令名称
## 交互模式
- q：退出top命令
- <Space>：立即刷新
- s：设置刷新时间间隔
- c：显示命令完全模式
- t:：显示或隐藏进程和CPU状态信息
- m：显示或隐藏内存状态信息
- l：显示或隐藏uptime信息
- f：增加或减少进程显示标志
- S：累计模式，会把已完成或退出的子进程占用的CPU时间累计到父进程的MITE+
- P：按%CPU使用率排行
- T：按MITE+排行
- M：按%MEM排行
- u：指定显示用户进程
- r：修改进程renice值
- kkill：进程
- i：只显示正在运行的进程
- W：保存对top的设置到文件^/.toprc，下次启动将自动调用toprc文件的设置。
- h：帮助命令。
- 1: 显示各个Cpu信息
## example
- `top -p pid`:显示指定进程信息(支持pid,pid多个进程id)
- 
# 查询软硬件信息
- `uname -a`,`lsb_release -a`linux系统版本
- `more /etc/release`操作系统版本
- `neofetch`系统概览
- `cat /proc/cpuinfo`:cpu信息
- `cat /proc/meminfo`:内存信息
- `pagesize`:显示内存page大小(kb)
- `ulimit -a`:显示系统资源限制信息
# lsof
## 命令参数
- -p:指定进程ID
- -d:指定要显示的文件描述符编号
- -a:列出打开文件存在的进程
- -c<进程名>:列出指定进程所打开的文件
- -g:列出GID号进程详情
- -d<文件号>:列出占用该文件号的进程
- +d<目录>:列出目录下被打开的文件
- +D<目录>:递归列出目录下被打开的文件
- -n<目录>:列出使用NFS的文件
- -i<条件>:列出符合条件的进程。（4、6、协议、:端口、 @ip ）
- -p<进程号>:列出指定进程号所打开的文件
- -u:列出UID号进程详情
- -h:显示帮助信息
- -v:显示版本信息
## 列信息
* COMMAND 正在运行的命令名字的前 9 个字符
* PID 进程的 PID
* USER 进程属主的登录名
* DEVICE 设备的设备号（主设备号和从设备号）
* SIZE 如果有的话，表示文件的大小
* NODE 本地文件的节点号
* NAME 文件名
### TYPE
TYPE 文件的类型
1) DIR：表示目录
2) CHR：表示字符类型
3) BLK：块设备类型
4) UNIX： UNIX 域套接字
5) FIFO：先进先出 (FIFO) 队列
6) IPv4：网际协议 (IP) 套接字
7) REG: 常规文件
### FD
FD 文件描述符号以及访问类型
1)cwd：表示current work dirctory，即：应用程序的当前工作目录，这是该应用程序启动的目录，除非它本身对这个目录进行更改
2)txt ：该类型的文件是程序代码，如应用程序二进制文件本身或共享库，如上列表中显示的 /sbin/init 程序
3)lnn：library references (AIX);
4)er：FD information error (see NAME column);
5)jld：jail directory (FreeBSD);
6)ltx：shared library text (code and data);
7)mxx ：hex memory-mapped type number xx.
8)m86：DOS Merge mapped file;
9)mem：memory-mapped file;
10)mmap：memory-mapped device;
11)pd：parent directory;
12)rtd：root directory;
13)tr：kernel trace file (OpenBSD);
14)v86  VP/ix mapped file;
15)0：表示标准输入
16)1：表示标准输出
17)2：表示标准错误
一般在标准输出、标准错误、标准输入后还跟着文件状态模式：r、w、u等
1)u：表示该文件被打开并处于读取/写入模式
2)r：表示该文件被打开并处于只读模式
3)w：表示该文件被打开并处于
4)空格：表示该文件的状态模式为unknow，且没有锁定
5)-：表示该文件的状态模式为unknow，且被锁定
同时在文件状态模式后面，还跟着相关的锁
1)N：for a Solaris NFS lock of unknown type;
2)r：for read lock on part of the file;
3)R：for a read lock on the entire file;
4)w：for a write lock on part of the file;文件的部分写锁)
5)W：for a write lock on the entire file;整个文件的写锁)
6)u：for a read and write lock of any length;
7)U：for a lock of unknown type;
8)x：for an SCO OpenServer Xenix lock on part      of the file;
9)X：for an SCO OpenServer Xenix lock on the      entire file;
10)space：if there is no lock.
## example
- `lsof -a -p $$ -d 0,1,2`显示当前进程0,1,2的文件描述符信息,信息含义如下:
- `lsof -i tcp`:列出所有tcp网络连接信息
- `lsof - i :3306`:列出谁在使用某个端口
- `lsof -a -u test -i`:列出某个用户的所有活跃的网络端口
- `lsof -i @nf5260i5-td:20,21,80 -r 3`:列出目前连接主机nf5260i5-td上端口为：20，21，80相关的所有文件信息，且每隔3秒重复执行
- `lsof -i 4 -a -p 1234`:列出被进程号为1234的进程所打开的所有IPV4 network files
# ps
## 命令参数
- a 显示所有进程
- -a 显示同一终端下的所有程序
- -A 显示所有进程
- c 显示进程的真实名称
- -N 反向选择
- -e 等于“-A”
- e 显示环境变量
- f 显示程序间的关系
- -H 显示树状结构
- r 显示当前终端的进程
- T 显示当前终端的所有程序
- u 指定用户的所有进程
- -au 显示较详细的资讯
- -aux 显示所有包含其他使用者的行程
- -C<命令> 列出指定命令的状况
- –lines<行数> 每页显示的行数
- –width<字符数> 每页显示的字符数
- –help 显示帮助信息
- –version 显示版本显示
## 输出列
- F 代表这个程序的旗标 (flag)， 4 代表使用者为 super user
- S 代表这个程序的状态 (STAT)
- UID 程序被该 UID 所拥有
- PID 进程的ID
- PPID 则是其上级父程序的ID
- C CPU 使用的资源百分比
- PRI 这个是 Priority (优先执行序) 的缩写，详细后面介绍
- NI 这个是 Nice 值
- ADDR 这个是 kernel function，指出该程序在内存的那个部分。如果是个 running的程序，一般就是 “-“
- SZ 使用掉的内存大小
- WCHAN 目前这个程序是否正在运作当中，若为 - 表示正在运作
- TTY 登入者的终端机位置
- TIME 使用掉的 CPU 时间。
- CMD 所下达的指令为何
### STAT
- D 不可中断 uninterruptible sleep (usually IO)
- R 运行 runnable (on run queue)
- S 中断 sleeping
- T 停止 traced or stopped
- Z 僵死 a defunct (”zombie”) process
### example
- `ps -ef`:显示所有进程信息，连同命令行
- `ps aux`:列出目前所有的正在内存中的程序
# strace
strace常用来跟踪进程执行时的系统调用和所接收的信号
## 输出信息
每一行都是一条系统调用，等号左边是系统调用的函数名及其参数，右边是该调用的返回值。
## 参数
- -c 统计每一系统调用的所执行的时间,次数和出错的次数等.
- -d 输出strace关于标准错误的调试信息.
- -f 跟踪由fork调用所产生的子进程.
- -ff 如果提供-o filename,则所有进程的跟踪结果输出到相应的filename.pid中,pid是各进程的进程号.
- -F 尝试跟踪vfork调用.在-f时,vfork不被跟踪.
- -h 输出简要的帮助信息.
- -i 输出系统调用的入口指针.
- -q 禁止输出关于脱离的消息.
- -r 打印出相对时间关于,,每一个系统调用.
- -t 在输出中的每一行前加上时间信息.
- -tt 在输出中的每一行前加上时间信息,微秒级.
- -ttt 微秒级输出,以秒了表示时间.
- -T 显示每一调用所耗的时间.
- -v 输出所有的系统调用.一些调用关于环境变量,状态,输入输出等调用由于使用频繁,默认不输出.
- -V 输出strace的版本信息.
- -x 以十六进制形式输出非标准字符串
- -xx 所有字符串以十六进制形式输出.
- -a <column>设置返回值的输出位置.默认 为40.
输出写入到指定文件中的数据.
- -o filename 将strace的输出写入文件filename
- -p pid 跟踪指定的进程pid.
- -s strsize 指定输出的字符串的最大长度.默认为32.文件名一直全部输出.
- -u username 以username 的UID和GID执行被跟踪的命令
### -e
-e expr 指定一个表达式,用来控制如何跟踪.格式如下:
`[qualifier=][!]value1[,value2]...`
qualifier只能是 trace,abbrev,verbose,raw,signal,read,write其中之一.value是用来限定的符号或数字.默认的 qualifier是 trace.感叹号是否定符号.例如:
-eopen等价于 -e trace=open,表示只跟踪open调用.而-etrace!=open表示跟踪除了open以外的其他调用.有两个特殊的符号 all 和 none.
注意有些shell使用!来执行历史记录里的命令,所以要使用\\.
- -e trace=<set>
只跟踪指定的系统 调用.例如:-e trace=open,close,rean,write表示只跟踪这四个系统调用.默认的为set=all.
    - -e trace=file 只跟踪有关文件操作的系统调用.
    - -e trace=process 只跟踪有关进程控制的系统调用.
    - -e trace=network 跟踪与网络有关的所有系统调用.
    - -e strace=signal 跟踪所有与系统信号有关的 系统调用
    - -e strace=ipc 跟踪所有与进程通讯有关的系统调用
- -e abbrev=<set> 设定 strace输出的系统调用的结果集.`-v`等价于`abbrev=none`.默认为abbrev=all.
- -e raw=<set> 将指定的系统调用的参数以十六进制显示.
- -e signal=<set> 指定跟踪的系统信号.默认为all.如 signal=!SIGIO(或者signal=!io),表示不跟踪SIGIO信号.
-e read=<set> 输出从指定文件中读出 的数据.例如:
    -e read=3,5.从文件描述符3,5读出的系统调用
    -e write=<set>
## example
`strace -o output.txt -T -tt -e trace=all -p 28979`
`strace -p <process-pid>`:实时输出程序的系统调用
`strace -f -F -o ~/straceout.txt myserver`:跟踪可执行程序


# 参考
[](https://jeremyxu2010.github.io/2019/12/60%E7%A7%92%E5%AE%8C%E6%88%90linux%E7%B3%BB%E7%BB%9F%E7%9A%84%E6%80%A7%E8%83%BD%E5%88%86%E6%9E%90%E8%AF%91/)
