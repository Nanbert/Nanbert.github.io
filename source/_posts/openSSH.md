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
把客户端生成的公钥复制粘贴到文件~/.ssh/authorized_keys中去,一个公钥占据一行
*ssh-copy-id--自动上传公钥*
`ssh-copy-id -i key_file user@host` 自动上传公钥到服务器
公钥文件可以不指定路劲和.pub后缀,会自动在~/.ssh目录下寻找
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



