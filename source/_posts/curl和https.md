---
title: curl和https
date: 2022-03-15 20:35:58
subtitle:
categories:
tags:
index_img: /img/curl.png
banner_img: /img/curl.png
---
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
### @作用
### -#/--progress-bar
有时进度条无法估计时间
### -v/--verbose
### -l/--location
### -d/--data
### -K/--config
`-K <fileName>`
此选项是为了帮助过长的选项不好输在命令行中，fileName则可以保存这些选项
- 一行一个选项，长选项可以省略`--`
- 可以用`#`注释
- 选项和参数内容间可以用`=`或`:`使结构清晰
- 参数如果含空格，必须用双引号括起来,括号内只可以用下面这些转义字符`\\,\",\t,\n,\r,\v`,如果不用双引号,则遇到第一个空格就结束读取
- 文件内也可以指明url，必须如下格式
`url = "http://example.com"`
### --no-verbose
### --manual
### --proto-default
### --fail-early
### --remote-name-all
### -s / --silent
### -u/--user
指定用户名和密码，冒号分隔`user:passwd`,这种明文，最好不用，协议层方面使用https和ftps等，如果非要使用明文见选项`--digest`,`--negotiate`,`--ntlm`,或者直接通过配置文件###1禁用。
### --next
### -:
### -Z/--parallel
### --parallel-max
### -h/--help
传参数可以把参数放到文件里，如下:
`curl -d @json http://example.com`
