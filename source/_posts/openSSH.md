---
title: openSSH
date: 2020-12-21 20:30:23
subtitle:
categories:
tags:
cover: https://z3.ax1x.com/2021/04/03/cnhnsg.png
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
4.命令行配置项

|参数|意义|例子|
|:-:|:-:|:-:|
|-c|指定加密算法|`ssh -c blowfish,3des server.example.com`,`ssh -c blowfish -c 3des server.example.com`|
|-C|压缩数据传输|`ssh -C server.example.com`|
|-d|设置打印的debug信息级别,数值越高越详细|`ssh -d 1 foo.com`|
|-D|指定本机的Socks监听端口,该端口收到的请求,都将转发到远程SSH主机,又称动态端口转发|`ssh -D 1080 server`|
|-f|表示SSH链接在后台运行||
|-F|指定配置文件|`ssh -F /usr/local/ssh/other_config`|
|-i|用于指定私钥,默认值是\~/.ssh/id_dsa|`ssh -i my-key server.example.com`|
|-l|参数指定远程登录的账户名|`ssh -l nanbert server.example.com`|
|-L|设置本地端口转发|`ssh -L 9999:targetServer:80 user@remoteServer`所有发向本地9999端口的请求,都会经过remoteServer发往targetServer的80端口,相当于直接连上了80的端口|
|-m|指定校验数据完整性的算法(MAC)|`ssh -m hmac-sha1,hmac-md5 server.example.com`|
|-o|参数用来指定一个配置命令,来覆盖配置文件设置|`ssh -o "User sally" server.example.com`|
|-p|指定链接的服务器的端口|`ssh -p 2305 server.example.com`|
|-q|安静模式,不输出任何警告|`ssh -q foo.com`|
|-R|指定远程端口转发|`ssh -R 9999:targetServer:902 local`该命令需要在跳板服务器执行,指定本机计算机local监听自己的9999端口,所有发向这个端口的请求,都会转向targetServer的902端口|
|-t|在ssh直接运行远端命令时,提供一个交互shell|`ssh -t server.example.com emacs`|
|-v|显示详细信息,可重复多次,表示详细程度|`ssh -vvv server.example.com`|
|-V|显示客户端版本信息||
|-X|表示打开X窗口转发|`ssh -X server.example.com`|
|-1,-2,-4,-6|1表示SSH1协议,2表示SSH2协议,4表示IPv4协议(默认值),6表示Ipv6协议||

5.配置文件

|路径|作用|
|:-:|:-:|
|/etc/ssh/ssh_config|全局配置文件|
|~/.ssh/config|用户个人的配置文件|
|~/.ssh/id_ecdsa|用户的ECDSA私钥|
|~/.ssh/id_ecdsa.pub|用户的ECDSA公钥|
|~/.ssh/rsa|用于SSH2的rsa私钥|
|~/.ssh/rsa.pub|用于SSH2的rsa公钥|
|~/.ssh/identity|用于SSH1的rsa私钥|
|~/.ssh/identity.pub|用于SSH1的rsa公钥|
|~/.ssh/known_hosts|包含SSH服务器的公钥指纹|

配置文件示例:
```bash
Host *
	Port 2222
Host remoteserver
	HostName remote.example.com
	User nanbert
	Port 2112
```
常见配置命令:
`AddressFamily <option>`option可以是inet,表示IPV4协议,也可以是inet6,表示IPV6协议
`BindAddress 192.168.10.235`指定本机的IP地址(如果本机有多个Ip地址)
`CheckHostIP <yesOrNo>`是否检查SSH的服务器IP地址是否跟公钥数据库吻合
`Ciphers <option>`:指定加密算法
`Compression <yesOrNo>`是否压缩传输信号
`ConnectionAttempts <num>`客户端进行连接时,最大尝试次数
`ConnectTimeout <num>`客户端进行连接时,服务器在指定秒数内没有回复,则中断连接尝试
`DynamicForward <portNum>`指定动态转发端口
`GlobalKnownHostsFile <filePath>`指定全局的公钥数据库文件的位置
`Host <serverName>`指定连接的域名或IP地址,也可以是别名,支持通配符,后面所有配置都是针对该主机,直到遇到下一个Host
`HostKeyAlgorithm <options>`指定密钥算法,优先级从高到低排列,以逗号分隔
`HostName <serverAddress>`在Host命令使用别名的情况下用
`IdentityFile <fileName>`指定私钥文件
`LocalForward 2001 localhost:143`指定本地端口转发
`LogLevel <options>`指定日志详细程度。如果设为`QUIET`,将不输出大部分的警告和提示
`MACs <options>`指定数据校验算法,以逗号分隔
`NumberOfPasswordPrompts <num>`输错密码最大尝试数
`PasswordAuthentication <yesOrNo>`是否支持密码登录,这里只是客户端,需要服务器也有相同的设置
`Port <portNum>`指定客户端链接的SSH服务器端口
`PreferredAutentications publickey,hostbased,password`指定各种登录方法优先级
`Protocol <1,2>`支持的SSH协议版本,可以用逗号分隔同时支持两个版本
`PubKeyAuthentication <yesOrNo>`是否支持密钥登录,这里只是客户端设置,需要服务器相同的设置
`RemoteForward 2001 server:143`指定远程端口转发
`SendEnv <variable>`客户端向服务器发送环境变量名,多个环境变量之间用空格分隔,变量的值从当前环境拷贝
`ServerAliveCountMax <num>`如果没有收到服务器的回应,客户端发送多少次keepalive信号,才断开连接,默认为3
`ServerAliveInterval <num>`客户端建立连接后,如果在给定的数秒内,没有收到服务器发来的消息,客户端向服务器发送keepalive消息,如果不希望客户端发送,这一项设为0
`StrictHostKeyChecking <yesOrNo>`yes表示严格检查,服务器公钥为未知或发生变化,则拒绝连接。no表示如果服务器公钥未知,则加入客户端公钥数据库,如果公钥发生变化,不改变客户端公钥数据库,输出一条警告,依然允许连接继续进行。ask(默认值)表示向用户询问是否继续
`TCPKeepAlive <yesOrNo>`客户端是否定期向服务器发送keepalive信息
`User <userName>`指定登录账户名
`UserKnownHostsFile <filePath>`指定当前用户的服务器公钥指纹列表(known_hosts)的文件位置
`VerifyHostKeyDNS <yesOrNo>`是否检查SSH服务器的DNS记录,确认公钥指纹是否与known_hosts文件保持一致
6.密钥
*概念*:
密钥是一个非常大的数字,通过加密算法得到。对称加密只需要一个密钥,非对称加密需要成对使用,分为公钥和私钥。
SSH密钥登录采用非对称加密,每个永不通过自己的密钥登录。其中,私钥必须私密保存,不能泄露;公钥则公开,对外发送。它们的关系是,公钥和私钥是一一对应
*过程*:
预备步骤,客户端通过`ssh-keygen`生产自己的公钥和私钥
第一步,手动将客户端的公钥放入远程服务器的指定位置
第二步,客户端向服务器发起SSH登录请求
第三步,服务器收到用户SSH登录的请求,发送一些随机数据给用户,要求用户证明自己的身份
第四步,客户端收到服务器发来的数据,私用私钥对数据进行签名,然后再发给服务器
第五步,服务器收到客户端发来的加密签名后,使用对应的公钥解密,然后跟原始数据比较。如果一致,就允许用户登录
*ssh-key--生成密钥*
该命令会生成一对密钥,私钥默认存在~/.ssh/id_rsa,公钥默认存在~/.ssh/id_rsa.pub

|选项|含义|
|:-:|:-:|
|-b [num]|指定密钥的二进制位数。这个参数越大,密钥越不容易破解,但是加密解密的开销也会越大,一般至少应该是1024|
|-C "[string]"|可以为密钥文件指定新的注释,格式一般为`username@host`|
|-f [filename]|参数指定生成的私钥文件。不指定的话,会在~/.ssh文件夹下生成一对密钥|
|-F [hostname]|检查某个主机名是否在known_hosts文件里面|
|-N [secretword]|指定私钥的密码|
|-p[secretword]|重新指定私钥的密码|
|-R [hostname]|将指定的主机公钥移除出known_hosts文件|
|-t [algorithm]|指定加密算法,一般为dsa或rsa|

*手动上传公钥*
把客户端生成的公钥复制粘贴到文件\~/.ssh/authorized_keys中去,一个公钥占据一行
*ssh-copy-id--自动上传公钥*
`ssh-copy-id -i key_file user@host` 自动上传公钥到服务器
公钥文件可以不指定路劲和.pub后缀,会自动在\~/.ssh目录下寻找
确保authorized_keys文件末尾为换行符,否则两个公钥连在一起,两个都会失效

*ssh-agent命令*
私钥设置了密码后,每次使用都必须输入密码,连续使用scp命令时,这就很麻烦,ssh-agent命令就是为了解决这个问题而设计的,它让用户在整个bash对话中,只在第一次使用SSH命令是输入密码,然后将私钥保存在内存中
第一步,新建一次命令对话
`ssh-agent bash(zhs、fish)`
第二步,添加私钥
`ssh-add [filename]`可以不指定私钥名称,使用默认文件~/.ssh/id_rsa
第三步登录远程服务器
`ssh remoteHost`
最后,如果要退出ssh-agent,可以按Ctrl+d,也可以用如下命令
`ssh-agent -k`
*ssh-add命令*
ssh-add命令用来将私钥加入ssh-agent

|选项|含义|
|:-:|:-:|
|-d|从内存中删除指定的私钥|
|-D|从内存中删除所有已经添加的私钥|
|-l|列出所有已经添加的私钥|

<font size=6>**服务器端sshd**</font>
**1.sshd配置文件**
`/etc/ssh/sshd_config`配置文件
`/etc/ssh/ssh_host_ecdsa_key`ECDSA私钥
`/etc/ssh/ssh_host_ecdsa_key.pub`ECDSA公钥
`/etc/ssh/ssh_host_key`用于SSH1协议版本的RSA私钥
`/etc/ssh/ssh_host_key.pub`用于SSH1协议版本的RSA公钥
`/etc/ssh/ssh_host_rsa_key`用于SSH2协议版本的RSA私钥
`/etc/ssh/ssh_host_rsa_key.pub`用于SSH2协议版本的RSA公钥
`/etc/pam.d/sshd`PAM配置文件
<font color=#FF0000>重装会使这些文件失效,可以先备份</font>
**sshd配置项**

|配置项|含义|
|:-:|:-:|
|AcceptEnv [variables...]|允许接受客户端通过SendEnv命令发来的哪些环境变量,变量名用空格分隔|
|AllowGroups [groupNames...]|指定允许登录的用户组,多个组之间用空格隔开,若不用该项,则所有组都可以用|
|AllowUsers [userNames...]|指定允许登录的用户,用户名之间用空格隔开,支持通配符|
|AllowTcpForwarding [options]|默认值为yes,允许端口转发,local只允许本地端口转发,remote表示只允许远程端口转发|
|AuthorizedKeysFile [directory]|指定存储用户公钥的目录,默认是`~/.ssh/authorized_keys`|
|Banner [file]|指定用户登录后,sshd向其展示的信息文件,默认不展示任何内容|
|ChallengeResponseAuthentication [yesOrNo]|指定是否用"键盘交互"身份验证方案,默认值为yes,如果完全禁用基于密码的验证,PasswordAuthentication也设为no|
|Ciphers [algorithms]|指定sshd可以接受的加密算法,多个算法之间使用逗号分割|
|ClientAliveCountMax [num]|指定建立连接后,客户端失去响应时,服务器尝试连接的次数|
|ClientAliveInterval [num]|允许客户端发呆的时间,单位为秒,如果超过这时间,连接将会关闭|
|Compression [yesOrNo]|Compression指定客户端与服务器之间的数据传输是否为压缩,默认为yes|
|DenyGroups [groupNames...]|指定不允许登录的用户组,组间空格分开|
|DenyUsers [userNames...]|指定不允许登录的用户,空格分开不同用户|
|FascistLogging [yesOrNo]|SSH1版本专用,指定日志是否输出全部Debug信息|
|HostKey [filePath]|指定服务器密钥的文件路径|
|KeyRegenerationInterval [num]|指定SSH1版本的密钥重新生成的时间间隔,单位为秒,默认为3600|
|ListenAddress [ipAddress]|指定sshd监听本机的IP地址,即sshd启用的IP地址,默认是0.0.0.0,表示在本机所有网络接口启用。可以改成只在某个网络接口启用,可以多次使用该配置项,来监听多个ip地址|
|LoginGraceTime [num]|指定允许客户端登录时发呆的最长时间,超过该时间就断开,0表示没有限制|
|LogLevel [options]|指定日志的详细程度,可能的值有:QUIET,FATAL,ERROR,INFO,VERNBOSE,DEBUG,DEBUG1,DEBUG2,DEBUG3,默认为INFO|
|MACs [algorithms]|指定sshd可以接受的数据校验算法(MACs hmac-sha1),多个算法之间使用逗号分隔|
|MaxAuthTries [num]|指定SSH登录允许的最大密码尝试数|
|MaxStartups [num]|指定允许同时并发的SSH链接数量,0表示没有限制,也可以是A:B:C形式,如10:50:20,表示如果达到10个并发链接,后面的连接有50%的概率被拒绝,如果达到20个并发连接,则后面的100%拒绝|
|PasswordAuthentication [yesOrNo]|是否允许密码登录,默认值为yes|
|PermitEmptyPasswords [yesOrNo]|指定是否允许空密码登录,默认为yes|
|PermitRootLogin [yesOrNo]|是否允许根用户登录,默认为yes,也可以设为prohibit-password,表示允许密钥登录root,但禁止密码登录|
|PermitUserEnvironment [yesOrNo]|是否允许sshd加载客户端的~/.ssh/environment文件和~/.ssh/authorized_keys文件里面的environment=options 环境变量设置.默认值为no|
|Port [num]|指定sshd监听的端口,默认22,可以多次设置,监听多个端口|
|PrintMoth [yesOrNo]|指定用户登录后,是否向其展示系统的motd的信息文件/etc/motd,默认为yes|
|Protocol [options]|1表示使用SSH1协议,'1,2'表示支持两个版本的协议|
|PubKeyAuthentication [yesOrNo]|指定是否允许公钥登录,默认为yes|
|QuietMode [yesOrNo]|SSH1专用,yes表示日志只输出致命的错误信息|
|RSAAuthentication [yesOrNo]||指定是否允许RSA认证,默认值为yes|
|ServerKeyBits [num]|指定SSH1版本的密钥重新生成时的位数,默认为767|
|StrictModes [yesOrNo]|指定sshd是否检查用户的一些重要文件和目录权限,即对于用户的SSH配置文件,密钥文件和所在目录,SSH要求拥有者必须是根用户或用户本人,其他人的写权限必须关闭|
|SyslogFacility [options]|指定Syslog如何处理sshd日志,默认是AUTH|
|TCPKeepAlive [unknown]|指定打开sshd跟客户端tcp链接的keepalive参数|
|UseDNS [yesOrNo]|指定用户SSH登录一个域名时,服务器是否使用DNS,确认该域名对应的IP地址包含本机,建议关闭|
|UserLogin [yesOrNo]|指定用户认证内部是否使用/user/bin/login代替SSH工具,默认为no|
|UserPrivilegeSeparation|指定用户认证通过后,使用另一个子线程处理用户权限相关的操作,这样利于提高安全性|
|VerboseMode|SSH2版本专用,指定日志输出详细的Debug信息|
|X11Forwarding|指定是否打开X window的转发,默认值为no|
**sshd命令行配置项**

|参数|含义|
|:-:|:-:|
|-d|用于显示debug信息|
|-D|指定sshd不作为后台守护进程运行|
|-e|将sshd写入系统日志syslog的内容导向标准错误|
|-f [filePath]|指定配置文件位置|
|-h [filePath]|指定密钥|
|-o [Key Value]|指定配置文件的一个配置项和对应的值,如:sshd -o "Port 2034"|
|-p [num]|指定sshd的服务端口|
|-t|检查配置文件语法是否正确|

<font size=6>**scp命令**</font>
**简介**
它的底层是SSH协议,默认端口22,相当于先用ssh命令登陆远程主机,然后在执行拷贝,可以用于两个远程系统之间的复制

|参数|含义|
|:-:|:-:|
|-c|指定传输的加密算法,如blowfish|
|-C|传输时压缩文件|
|-F|用来指定ssh_config文件,供ssh链接使用|
|-i|用来指定密钥的文件|
|-l|用来限制传输数据的带宽速率,单位是kb/s|
|-p|用来保留修改时间,访问时间,文件状态等原始文件的信息|
|-P|用来指定远程主机的SSH端口,默认用22|
|-q|用来关闭显示拷贝的进度条|
|-r|递归复制|
|-v|显示详细的输出|

<font size=6>**sftp命令**</font>
**简介**
sftp是ssh提供的一个客户端应用程序,主要用来安全地访问FTP。因为FTP是不加密协议,很不安全,sftp相当于将FTP放入SSH:
`sftp username@hostname`
进入sftp后,使用那些命令,如get获取远程文件,put上传文件等
<font size=6>**端口转发**</font>
(待建)可见<https://wangdoc.com/ssh/port-forwarding.html>
<font size=6>**证书登录**</font>
(待建)可见<https://wangdoc.com/ssh/ca.html>
