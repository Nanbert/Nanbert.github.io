---
title: curl和https
date: 2022-03-15 20:35:58
subtitle:
categories:
tags:
index_img: /img/curl.png
banner_img: /img/curl.png
---
## HTTP
HTTP/2和HTTP/3的头部通常会被curl压缩发送，但是-v选项总是把它们解压成HTTP/1.1的样子
### 代理
- HTTP代理为了安全，使用**CONNECT**方法
- HTTP代理可以代理FTP,此时curl将认为就是HTTP,FTP所有特性无效
- MITM代理可以监控加密的流量
- 代理认证失败，拒绝代理，会返回407
- HTTPS代理的默认端口为443
- curl中与代理相关的环境变量
  - [scheme]_proxy:指定某个协议的默认代理,等价于-x选项,除了http_proxy只能全部小写(CGI的原因),其他形式可以用全部大写
  ```bash
	http_proxy=http://proxy.example.com:80
	curl -v www.example.com
  ```
  - ALL_PROXY:所有url都走该代理
  - NO_PROXY:某些url不走该代理，用`,`分隔多个url，等效于--noproxy
## curl支持的协议
curl -V查看
## URLs(URIs)
需要注意的是现代浏览器的地址栏里支持IRIs,一个URLs的超集(更强大，支持空白字符等等)
### Scheme
scheme就是分隔符前面的内容,是协议吗？不可以包含任何空格
- 分隔符:`://`curl只支持双斜杠的，有标准是单斜杠的
- curl会自动纠错一些scheme格式以及自动推测(通过选项--proto-default设置)
### Name and password(非主流)
scheme及分隔符之后可能跟着用户名和密码,不过这风险很大,正慢慢淘汰,同时你也可以通过选项等方式来指定。一般格式如下：
`curl ftp://user:password@example.com/`
### host
host就是主机，可以是数字地址本身或简单的域名
- ipv4:`curl http://127.0.0.1/`
- ipv6:`curl http://[::1]/`
### port number
如果端口号没指明，那么就会使用各个协议的默认端口。
通常在主机名后指明,用冒号和数字。为十进制数范围在[0-65535]
- ipv4:`curl http://example.com:8080/`
- ipv6:`curl http://[fdea::1]:8080/`
### path
每个url包含路径,如果没指明则是根目录`/`,如下是等价的
`https://example.com`==>`https://example.com/`
### FTP type(FTP特有很少用的特性)
该特性可以让你指明文件类型,指定不同的类型ftp协议相应的用不同传输,curl默认以二进制格式传递FTP协议
- A:指明传输类型为ASCII
`curl "ftp://example.com/foo;type=A"`
- I:指明传输类型为二进制
`curl "ftp://example.com/foo;type=I"`
- D:指明传输类型为目录(此时不要以斜杠结尾)
`curl "ftp://example.com/foo;type=D"`
### Fragment
由`#`标识
- URLs提供fragment，但curl传递时，会忽略它,(就是浏览器中的定位,跟网页屏幕位置有关)
- 有时`#`处于路径中,为了方便你可用`%23`来代替`#`
`https://www.example.com/info.html#the-plot`==>`curl https://www.example.com/info.html%23the-plot`
### curl与URLs
curl支持许多选项，不是可选项内容的都是URLs，可以支持多个URLs,从左到右一个个解析
- curl总是返回最后一个URL的错误代码(可以用--fail-early改变),
- curl的选项总是应用所有URLs(当然存在例外(-o、-O)),要想应用不同的URL不同的option,可以使用`--next、-:`
- 指定多个URLs,可以节省时间(如果多个URLs有相同主机，TCP将会一直保持链接)
- 默认是串行,可以指定并行(-Z),并行时的进度条和串行时有区别，该进度条是所有当前运行的并行任务的实时情况
#### globbing
有时想传递同一主机下的一系列资源，可以用`{}`和`[]`来指定，处于它们之间的内容称为[globbing],并且他们可以混用，此时为防止shell的副作用，应该用引号括起来URL，
- `[]`
  - `curl -O "http://example.com/[1-100].png"`
  - `curl -O "http://example.com/[001-100].png"`支持前导0
  - `curl -O "http://example.com/[001-100:2].png"`取奇数
  - `curl -O "http://example.com/[a-z].png"`支持字母
- `{}`
  - `curl -O "http://example.com/{one,two,three,alpha,beta}.html"`
- -o选项里面可以通过`#[num]`来代表globbing内容,num从1开始编号
  - `curl "http://{one,two}.example.com" -o "file_#1.txt"`
  - `curl "http://{site,host}.host[1-5].example.com" -o "subdir/#1_#2"`
## curl
### 配置文件curlrc
按以下优先级读取文件：
1) "$CURL_HOME/.curlrc"
2) "$XDG_CONFIG_HOME/.curlrc"
3) "$HOME/.curlrc"
## 选项

### -C/--continue-at [num/-]
指明从num byte offset开始继续下载或`-`curl根据已有的下载文件确定从哪开始继续下载。
**例1**:`curl --continue-at 100 ftp://example.com/bigfile`
**例2**:`curl --continue-at - http://example.com/bigfile -O`
### --cacert [path]
设置CA证书路径
### --cert [file:passwd]
TLS指定客户端的证书文件,待学习相关知识
```bash
curl --cert mycert:mypassword https://example.com
curl --cert mycert:mypassword --key mykey https://example.com
curl --cert mycert:mypassword --cert-type PEM \
     --key mykey --key-type PEM https://example.com
```
### --cert-type
见--cert例子
### --cert-status
这个TLS特性为OCSP stapling,比较新，目前仅有openssl,gnutls,nss支持，一般不使用
### --ciphers
TLS中ciphers,除非你知道你在干啥，否则慎用。
### --connect-to [source name:source port:destination name:destination port]
有时负载均衡，一个host name其实由多个服务提供，有时你只想测试其中一个服务，可以用此选项，如下：
`curl --connect-to www.example.com:80:load1.example.com:80 http://www.example.com`
### --connect-timeout [num]
指定tcp连接多长时间没反应算失败，单位秒，可以给出float类型数字
### --compressed
http/https,请求服务器提供压缩版本的内容，curl会在数据到达后自动解压，这只是加快传输速度，注意不能和另一个--tr-encoding混用，因为两者采用不同压缩
### -d/--data [string or num]
发送的数据
### --data-binary
### --dns-interface
指定dns走的网卡？
### --dns-ipv4-addr
指定ipv4的dns服务器
### --dns-ipv6-addr
指定ipv6的dns服务器
### --dns-servers
指定一个dns服务器
### -f/--fail
### -F
### --fail-early
### --ftp-port
### -h/--help
### -H [header content]
自定义头部内容
`curl -H "Host: www.example.com" http://localhost/`
### --interface [ip addr or some interface]
指定哪个网络接口来传输流量，或者使用哪个原始ip地址（前提你有多个ip）这个不影响dns的接口，dns接口可用--dns-interface
### -J/--remote-header-name
HTTP头可能提供`Content-Disposition:`,这其中包含了建议的文件名，这个选项使用该文件名作为输出，如果该内容存在，会覆盖-O选项。
- 它只会保留文件名部分，忽略目录
- CURL不会帮你解码，可能是个URL原码格式的文件名（浏览器会解码）

### -k/--insecure
tls/ssh协议中，curl会跳过检查known_hosts文件及本地安全证书，直接信任
### -K/--config
`-K <fileName>`
此选项是为了帮助过长的选项不好输在命令行中，fileName则可以保存这些选项
- 一行一个选项，长选项可以省略`--`
- 可以用`#`注释
- 选项和参数内容间可以用`=`或`:`使结构清晰
- 参数如果含空格，必须用双引号括起来,括号内只可以用下面这些转义字符`\\,\",\t,\n,\r,\v`,如果不用双引号,则遇到第一个空格就结束读取
- 文件内也可以指明url，必须如下格式
`url = "http://example.com"`
### --keepalive-time [num]
curl默认会保持无流量的tcp连接长达60s,这可以更改时间，单位为秒
### --key
指定cert健，见--cert
### --key-type
见--cert
### -L/--location
### --limit-rate <num>
参数是个数字，默认单位是byte，可以跟K/M/G，整个过程的平均速度将不超过这个值,也同样适用于上传速率
### --local-port [num or range]
通常不需要指定本地端口，但有时只有某些端口是开放的，指定curl的本地端口，可以指定一个范围，因为一个可能被占用了,最好不要指定1024以下的端口
### -m/--max-time [num]
整个命令允许运行的最长时间，即使正在下载，也会立刻退出(退出码28)
### --mail-from
smtp中指定发件人
### --mail-rcpt
smtp中指定收件人
### --manual
### --max-filesize [num]
单位是byte，如果curl在传输开始可以获得将要下载的内容大小，该选项才会起作用，如果超过该大小，curl将会自动放弃。
### -n/--netrc
读取`~/.netrc`配置文件，该文件存储用户名密码，例子如下
```bash
#以下的值都不许有空格，且可以写在一行
machine example.com #可以填default,此时不需要machine关键字
login nanbert
password xxx 
macdef xxx #该选项curl不支持，会忽略
```
### -netrc-file [path]
不读默认`~/.netrc`文件，而是具体某个文件
### --netrc-optional
这与`--netrc`区别在于，使得默认配置内容是可选的,不是强制的
### --next
### --no-keepalive
默认curl会保持tcp(无流量)连接60s,这个会关闭该功能
### --noproxy
不使用全局环境变量代理
### --no-verbose
### -o
输出到某个文件，一个该选项对应一个url,想要指明多个，必须声明多个-o
### -O/--remote-name
把结果输出到使用远程服务器的原始文件名,一个该选项对应一个url,想要指明多个，必须声明多个-O
### -p/--proxytunnel
使用隧道对代理加密
`curl -p -x http://proxy.example.com:80 ftp://ftp.example.com/file.txt`
### --parallel-max
### --pinnedpubkey [sha256//hashnum1;hashnum2;..]
TLS协议中，Certifiate pinning中直接指定sha256值
### --proto-default
### --proxy1.0 [ip addr]
指定代理，和`--x`一样，只是使用HTTP/1.0
### --proxy-anyauth
任意一种代理用户名认证方式,根据代理服务器要求自动匹配
### --proxy-digest
一种代理用户名认证方式
### --proxy-header
该头只发送给代理，真正的远程服务器不会收到，这比--header更精细化
`curl --proxy-header "User-Agent: magic/3000" -x proxy https://example.com/`
### --proxy-negotiate
一种代理用户名认证方式
### --proxy-ntlm
一种代理用户名认证方式
### --range [num1-num2]
只下num1 byte offset至num2 byte offset的内容
### --remote-name-all
所有结果均输出保存到服务器上的原始文件名
### --resolve [host name:port:ip address]
dns重定向，这会保存到curl的cache中
`curl --resolve example.com:80:127.0.0.1 http://example.com/`
### -s/--silent
此选项关闭进度条，并不显示错误，但是-S/--show-error不受此选项影响
### -S/--show-error
默认情况下curl会输出错误，该选项主要是抵消-s的作用
### --speed-time [num]
经常和--speed-limit一起用,下面的意思是速度小于1000并且持续15s就退出
`curl --speed-time 15 --speed-limit 1000 https://example.com/`
### --speed-limit [num]
见--speed-time
### --ssl
尝试ssl加密(FTP,IMAP,POP3,SMTP)
### --sslv2
使用SSL2版本
### --sslv3
使用SSL3版本
### --ssl-reqd
强制ssl加密(FTP,IMAP,POP3,SMTP)
### --raw
禁用内容或传输编码的所有内部http解码，而是使用未经修改的原始数据
### --retry [num]
curl会在发生transient error时，会重新尝试num次，默认失败一次就不会尝试，transient error包括以下：超时，FTP 4XX返回码，http5xx返回码
### --retry-all-errors
有时你确定一个服务器是好的，出现任何错误都想重试，该选项就可以帮你
### --retry-connrefused
重试只会发生在transient error时，但拒绝访问不属于，有时你确认服务器只是重启或其他原因，该选项可以使得出现拒绝访问时，也可以重试。
### --retry-max-time [num]
第一次重试之前,curl会等1s,然后第二次重试会等2s,如此指数增长下去，直到达到10min，该选项会指明等待时间不少于num秒，--max-time选项仍会起作用
### -T
- HTTP PUT就是上传某个完整资源上传或替换远程的现有资源，很少被服务器启用
- FTP或TFTP 上传文件，如下
`curl -T uploadthis ftp://example.com/this/directory/`
`curl -T uploadthis ftp://example.com/this/directory/remotename`
- SMTP上传body内容，通常需要其他选项(有关header的选项)配合（--mail-from,等）
`curl -T mail smtp://mail.example.com/ --mail-from user@example.com`
### -t/--telnet-option [keyword=value]
telnet选项特有，传递以下三个参数：
- TTYPE=[term]:设置终端类型
- XDISPLOC=[X display]:设置X展示位置
- NEW_ENV=[var,val]:设置环境变量值
### --tftp-blksize [num]
tftp通信传输块默认大小为512,此选项可以修改，支持8-65464
### --tftp-no-options
有些tftp服务器不接受任何选项，此时该选项可以应用
### --tlspassword [passwd]
TLS的特性，可以在命令行中直接使用用户名和密码
`curl --tlsuser daniel --tlspassword secret https://example.com`
### --tlsuser [name]
见--tlspassword
### --tlsv1/--tlsv1.0
使用TLS>=1.0的版本
### --tlsv1.1
使用TLS>=1.0的版本
### --tlsv1.2
使用TLS>=1.2的版本
### --tlsv1.3
使用TLS>=1.3的版本,curl默认
### --tr-encoding
采用Transfer-Encoded压缩传输内容，curl会自动解压,不能和--compressed混用,且该选项不常用
### --trace [filename]
会跟踪每拍接受的内容，以16进制显示,每行开头的十六进制加冒号代表该内容的byte offset
### --trace-ascii [filename]
会跟踪每拍接受的内容，以ascii码可读形式显示
### --trace-time
化学选项：-v/--verbose,--trace和--trace-ascii
这个选项会帮助化学选项所输出内容之前加高精度的时间。
### -u/--user
指定用户名和密码，冒号分隔`user:passwd`,这种明文，最好不用，协议层方面使用https和ftps等，如果非要使用明文见选项`--digest`,`--negotiate`,`--ntlm`,或者直接通过配置文件###1禁用。
### -U/--proxy-user [user:passwd]
指定代理的用户名和密码
`curl -U daniel:secr3t -x myproxy:80 http://example.com`
### --upload-file
指定上传内容文件，用于smtp
### -v/--verbose
该选项会使curl显示更多的内容，具体格式如下：
- `*`后面跟解释性内容
- `>`后面跟客户端发送的头协议，（FTP,SMTP,POP3等没有头协议的，命令和返回当成头）
- `<`后面跟从服务端接受的返回头协议
HTTP/2和HTTP/3协议头是压缩的，但在此选项下会展开和HTTP/1.1一样的格式
### -V/--version
会输出版本相关的信息，各行含义如下:
**第一行:**版本号+平台+第三方依赖信息
**第二行:**版本发布日期
**第三行:**支持的协议
**第四行:**支持的特性
### -w/--write-out 
参数:`<formatted string>`或`@[filename或-]`
该选项会在每个传输完毕后，在末尾加上formatted string或文件中的内容,%可以使用`\n,\r,\t`转义字符，特殊的变量跟在`%`后,`%%`输出真正的%
有以下formatted string:

|格式|含义|
|:-:|:-:|
|`%{content_type}`|字面义，有的内容没有类型|
|`%{errormsg}`|字面义，无错误为空|
|`%{exitcode}`|传输的退出码，无错返回0|
|`%{filename_effective}`|最终传输内容所保存的文件名，只当指定-o或--remote-name选项有意义|
|`%{ftp_entry_path}`|登陆ftp的初始路径|
|`%{http_code}`|也就是response_code|
|`%{http_connect}`|待探索|
|`%{http_version}`|字面义|
|`%{json}`|所有write-out的变量生成json格式|
|`%{local_ip}`|最近一次连接的ipv4或ipv6的本地地址|
|`%{local_port}`|最近一次连接的本地端口|
|`%{method}`|最近请求的方法|
|`%{num_connects}`|最近传输的新连接数量|
|`%{num_headers}`|上一次传输的反应头的数量|
|`%{num_redirects}`|请求的重定向次数|
|`%{onerror}`|如果传输发生错误，输出之后的string,内容，否则不输出|
|`%{proxy_ssl_verify_result}`|与代理通信时请求的SSL对等证书验证的结果,0表示成功|
|`%{redirect_url}`|当发出HTTP请求而没有-L重定向时，重定向会将您带到的实际URL|
|`%{remote_ip}`|远程的ipv4或6地址|
|`%{remote_port}`|远程的端口|
|`%{response_code}`|字面义|
|`%{scheme}`|url的scheme|
|`%{size_download}`|字面义|
|`%{size_header}`|头部的大小|
|`%{size_request}`|请求的大小|
|`%{size_upload}`|上传大小|
|`%{speed_download}`|平均下载速度|
|`%{speed_upload}`|平均上传速度|
|`%{ssl_verify_result}`|请求SSL对等证书验证的结果，0表示成功|
|`%{stderr}`|接下来的内容输出到标准错误|
|`%{stdout}`|接下来的内容输出到标准输出|
|`%{time_appconnect}`|从开始到完成SSL/SSH/etc到远程主机的连接/握手花费的时间，单位秒|
|`%{time_connect}`|从一开始直到TCP连接到远程主机（或代理）完成花费的时间，单位秒|
|`%{time_namelookup}`|从一开始直到名字解析完成所花费的时间,单位秒|
|`%{time_pretransfer}`|从一开始直到文件传输即将开始所花费的时间，单位秒|
|`%{time_redirect}`|所有重定向步骤，包括名称查找，连接，预传输和最终事务开始之前的传输所花费的时间,单位秒|
|`%{time_starttransfer}`|从一凯斯直到第一个字节即将被传输，这包括time_pretransfer和服务器计算结果所需的时间|
|`%{time_total}`|完整操作持续的总时间，时间为秒，精度达到毫秒|
|`%{url}`|命令行中指定url|
|`%{url_effective}`|真实有效的url|
|`%{urlnum}`|url的编号，从0开始计数|

### -x/--proxy [ip addr]
- 指定代理，默认scheme为http,默认端口为1080
`curl -x 192.168.0.1:8080 http://example.com/`
- 指定SOCKS协议代理，这可以直接使用各个版本，而不用-x，说明如下：
```bash
#SOCKS4版本
curl -x socks4://proxy.example.com http://www.example.com/
curl --socks4 proxy.example.com http://www.example.com/
#SOCKS4a版本
curl -x socks4a://proxy.example.com http://www.example.com/
curl --socks4a proxy.example.com http://www.example.com/
#SOCKS5版本
curl -x socks5://proxy.example.com http://www.example.com/
curl --socks5 proxy.example.com http://www.example.com/
#SOCKS5h版本
curl -x socks5h://proxy.example.com http://www.example.com/
curl --socks5-ostname proxy.example.com http://www.example.com/
```
### --xattr

### -Z/--parallel
### -:
### -#/--progress-bar
当内容重定向时，进度条是默认打开的,该选项会展示一种简单的进度条，有时进度条无法估计时间
### @作用
传参数可以把参数放到文件里，如下:
`curl -d @json http://example.com`
## curl退出码
- 1：不支持的协议
- 2：初始化失败，libcurl可能出了问题
- 3：url格式不对
- 4：某个请求需要某个特性或选项未能满足
- 5：无法解析代理
- 6：无法解析主机
- 7：无法连接主机。可能端口，主机名或防火墙的问题
- 8：未知的ftp服务返回。可能未支持，也可能未启用选项
- 9：ftp拒绝访问。没该文件或用户名或没权限
- 10：ftp接受失败。
- 11：ftp奇怪的PASS回复
- 12：在等待服务器连接的活动FTP会话期间，超过限期
- 13：FTP PASV命令未知的反应。通过--ftp-port选项可能解决该问题
- 14：未知FTP227格式。这肯定是个坏服务器，或者可以通过--ftp-port解决该问题
- 15：FTP无法获得主机
- 16：HTTP/2 error
- 17：FTP无法设置binary传输。坏的服务器
- 18：只传输了部分文件
- 19：FTP无法获得下载该文件。RETR命令失败
- 20：（保留）
- 21：引用错误。IMAP、POP3、SMTP、FTP发送自定义命令时出错，建议查看报头
- 22：HTTP页面未抓取。对应400反应码以上的错误，出现该错误只能在-f选项启用的时候
- 23：写错误。写到本地时发生的错误
- 24：（保留）
- 25：上传失败。服务器空间已满或拒绝上传
- 26：读错误。从本地读取时发生错误
- 27：内存不够。系统分配给curl内存不足
- 28：操作超时。由各种选项设置的各种超时
- 29：（保留）
- 30：FTP PORT命令错误。PORT命令有点非主流，可以试试PASV
- 31：FTP无法使用REST。可以在没有范围或恢复的情况下重试
- 32：（保留）
- 33：HTTP范围错误
- 34：HTTP post错误,需要反馈BUG
- 35：TLS/SSL连接错误
- 36：无法恢复下载。FILE,FTP,SFTP会发生此错误
- 37：无法读取该文件。FILE协议，可能不存在或没权限
- 38：绑定LDAP失败。可能用户密码错误
- 39：LDAP搜索失败。
- 40：（保留）
- 41：（保留）
- 42：回调错误。开发者编程的错误
- 43：错误函数参数。libcurl的调用问题
- 44：（保留）
- 45：网络接口错误
- 46：（保留）
- 47：太多重定向。默认最多50个，可以通过--max-redirs改变
- 48：libcurl未知选项。可能curl和libcurl版本不一
- 49：telnet错误的选项
- 50：（保留）
- 51：SSL/TLS、SSH认证失败
- 52：服务器未返回任何内容。可能是服务器有意为之
- 53：未发现SSL引擎
- 54：无法设置SSL加密引擎为默认
- 55：无法发送网络数据。网络底层的错误，需要Wireshark等工具查看
- 56：无法接受网络数据。网络底层错误。
- 57：（保留）
- 58：本地认证有问题。
- 59：无法使用ssl密码。密码有格式标准
- 60：对等证书无法使用已知ca认证
- 61：无法识别传输编码
- 62：无效LDAP URL
- 63：超过最大文件大小限制
- 64：FTP SSL失败
- 65：发送之前帧失败
- 66：初始化SSL引擎失败
- 67：用户名密码CURL无法登陆
- 68：TFTP服务没有该文件
- 69：TFTP服务权限问题
- 70：TFTP服务没有空间
- 71：非法TFTP操作
- 72：未知TFTP传输id
- 73：TFTP文件已存在
- 74：TFTP没有该用户
- 75：字符转换失败
- 76：需要字符转换函数
- 77：读SSL CA认证时发生问题
- 78：URL中的资源不存在
- 79：在SSH会话中发生错误
- 80：关闭SSL连接失败
- 81：（保留）
- 82：无法下载CRL文件，错误格式
- 83：TLS认证检查失败
- 84：FTP PRET命令失败
- 85：RTSP:CSeq数字不匹配
- 86：RTSP:会话标识符不匹配
- 87：无法解析ftp文件列表
- 88：FTP 块回掉错误
- 89：没有可用的连接，会话将排队
- 90：SSL公钥不匹配固定公钥
- 91：无效SSL认证状态
- 92：HTTP/2流错误
- 93：API回调错误
- 94：认证错误
- 95：HTTP/3错误
- 96：QUIC连接错误
## curl与浏览器的区别
- 浏览器会对接受的数据进行二次解码,更易懂些，curl就直接解码
- 有的服务器会根据不同客户端(甚至不同的浏览器)提供更适配的内容
- 你可以使用f12然后选中network,右键点击你想要的资源，选中`copy cURL`就可以复制相应的命令
## POP3(curl读邮件使用的协议)
```bash
#To list message numbers and sizes:
curl pop3://mail.example.com/
#To download message 1:
curl pop3://mail.example.com/1
#To delete message 1:
curl --request DELE pop3://mail.example.com/1
```
**TLS加密**
```bash
curl pop3://mail.example.com/ --ssl-reqd
curl pop3s://mail.example.com/
```
## IMAP(curl读邮件使用的协议，更常用现代)
```bash
#Get the mail using the UID 57 from mailbox 'stuff':
curl imap://server.example.com/stuff;UID=57
#get the mail with index 57 from the mailbox 'fun'
curl imap://server.example.com/fun;MAILINDEX=57
#List the mails in the mailbox 'boring':
curl imap://server.example.com/boring
#List the mails in the mailbox 'boring' and provide user and password:
curl imap://server.example.com/boring -u user:password
```
**TLS加密**
```bash
curl --ssl imap://mail.example.com/inbox
curl imaps://mail.example.com/inbox
```
## SMTP(cURL写邮件使用的协议)
必须指定收发邮箱，以及内容,默认端口587
```bash
curl smtp://mail.example.com --mail-from myself@example.com --mail-rcpt \
receiver@example.com --upload-file email.txt
```
email.txt:
```bash
From: John Smith <john@example.com>
To: Joe Smith <smith@example.com>
Subject: an example.com example email
Date: Mon, 7 Nov 2016 08:45:16

Dear Joe,
Welcome to this example email. What a lovely day.
```
和POP3、IMAP一样，可以使用SSL/TLS加密，schema改成smtps即可，或者使用--ssl或--ssl-reqd,此时默认端口为465
## MQTT
订阅推送的协议，不太懂
## TFTP
小文件传输协议，使用的是UDP
- 下载`curl -O tftp://localserver/file.boot`
- 上传`curl -T file.boot tftp://localserver/`
## TELNET
即时通信协议，默认端口23
## DICT
字典查询的协议
alias:
- m:match和find
- d:define和lookup
例子：
```bash
curl dict://dict.org/m:curl
curl dict://dict.org/d:heisenbug:jargon
curl dict://dict.org/d:daniel:gcide
curl dict://dict.org/find:curl
```
## TLS/SSL
TLS是建立于TCP上一层的安全加密层，SSL是旧称(且所有ssl版本都在淘汰中)，两个是一个概念。
各个版本协议都有TLS版本：
- HTTP-HTTPS
- LDAP-LDAPS
- FTP-FTPS
- POP3-POP3S
- IMAP-IMAPS
- SMTP-SMTPS
TLS属于third-party,你可以通过--version查看，如果你feature中有MultiSSL的特性，证明curl是支持多版本的，你可以通过`CURL_SSL_BACKEND`来设置使用那个版二
### 版本历史
SSL2(1995)->SSL3->TLS1.0(1999)->TLS1.1(2006)->TLS1.2(2008)->TLS1.3(2018)
### CA的存储
一般都是内建的，但你也可以用--cacert指定路径（一定要是PEM格式），或者设置CURL-CA_BUNDLE环境变量
## 上传
### POST
### multipart formpost
