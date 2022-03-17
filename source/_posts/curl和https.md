---
title: curl和https
date: 2022-03-15 20:35:58
subtitle:
categories:
tags:
cover:
---
##URLs(URIs)
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

## curl
### @作用
### -v/--verbose
### -l/--location
### -d/--data
### --no-verbose
### --proto-default
### --fail-early
### --remote-name-all
### --next
### -:
### -Z/--parallel
### --parallel-max
传参数可以把参数放到文件里，如下:
`curl -d @json http://example.com`
