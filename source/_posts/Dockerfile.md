---
title: Dockerfile
date: 2022-04-04 10:33:38
subtitle:
categories:
tags:
banner_img: /images/dockerfile.jpg
index_img: /images/dockerfile.jpg
---
## 简介
Dockerfile是一个文本文件，其内包含了一条条指令，**每条指令构建一层**
- `#`进行注释
- `\`末尾进行换行，RUN执行多条命令十分有用
## docker build--构建镜像
- **格式：**`docker build [选项] [上下文路径/URL/-]`

|选项|含义|
|:-:|:-:|
|-t [reposity:tag]|指明镜像名称和tag|
|-f [dockerfile]|指定dockerfile，非主流，默认文件名为`Dockerfile`，且位于上下文路径中|
|--target \<some img\>|多阶段构建时，指定只构建某个镜像，而不是默认的最后一个|

### 上下文路径
- docker build命令其实是与服务器（即docker.service）通信，它会将上下文路径下的所有内容上传，而不是在本地进行构建的
- 支持`.dockerignore`文件，剔除不需要的内容
### 其他构建法
- Git repo:`docker build -t hello-world https://github.com/docker-library/hello-world.git`
- tar包:`docker build http://server/context.tar.gz`
- 标准输入：`docker build - < Dockerfile`或`cat Dockerfile| docker build -`(直接从标准输入读取，没有上下文，不可用依赖上下文的命令COPY等等)
- 标准输入+tar包：`docker build - < context.tar.gz`会自动解压，将里面视为上下文
## docker import-从rootfs压缩包导入(无需Dockerfile)
- **格式：**`docker import [选项] <文件>|<URL>|- [<仓库名>[:<标签>]]`
- **例子：**
```bash
$ docker import \
    http://download.openvz.org/template/precreated/ubuntu-16.04-x86_64.tar.gz \
    openvz/ubuntu:16.04
```
## docker save和docker load
是一种古老的保存和加载镜像的方法，已经非主流，实在没网，使用内网私有的Registry
### docker save
- 本质上是建立归档文件:`docker save [some image] -o [some file]`
- 使用压缩：`docker save [some image] | gzip > xx.tar.gz`
### docker load
`docker load -i some_file.tar.gz`
## FROM指定基础镜像
- **格式：**`FROM [image]`
- FROM是必备的指令，且必须是第一条
- 如果你想以空白镜像为基础，你可以这样`FROM scratch`
- 尽量小尺寸（推荐Alpine）
## RUN执行命令
- **格式：**`RUN [command]`
- 执行多条命令：使用`\`和`&&`，如下
  ```bash
	RUN apt-get update \
	&& apt-get install vim
  ```
- 永远不要`apt upgrade`，而是使用：
```bash
RUN apt-get update && apt-get install -y \
    aufs-tools \
    s3cmd=1.1.* \
 && rm -rf /var/lib/apt/lists/*
```
## COPY复制文件
- **格式：**`COPY [--chown=<user>:<group>] <源路径>... <目标路径>`
- **作用：**将从构建上下文目录中的路径复制的新的一层的镜像内的目标路径的位置，支持通配符，如果目录不存在会创建缺失的目录
- 如果源路径为文件夹，复制的时候不是复制该文件夹，而是将文件夹中的内容复制到目标路径
- `--from=[some]`:多阶段构建时，指定从某个镜像获取，而不是当前上下文
- 不要一次copy多个文件，这会使一层的缓存过大
## ADD更高级的复制文件
- **说明：**和COPY格式和性质基本一致,但是功能更多(不代表更好，只有自动解压缩时，使用该命令)：
1) 源路径可以是以个URL,文件权限自动设置为600,更改权限需要再加一层调整,另外如果下载是个压缩包，需要再加一层解压缩
2) 如果源路径是个tar压缩文件(gzip,bzip2,xz)，会自动解压缩文件
## CMD容器启动命令
- **格式1：**`CMD ["可执行文件"，"参数1"，"参数2"...]`(推荐使用这个，也支持bash格式，但会包装一层`sh -c`)
- **格式2：**`CMD ["参数1","参数2"]`,在指定了`ENTRYPOINT`后，可以直接指定参数
- **作用：**容器就是进程，该命令就是指定容器所运行默认的程序及参数,例如ubuntu的CMD就是`/bin/bash`,当然可以在命令行中用其他命令替换。
- **注意：**
1) 启动程序就是容器的应用进程，容器就是为了主进程存在的，主进程退出，容器就会退出，辅助进程不是它所关心的。所以必须是前台进程，例如`CMD service nginx start`被理解为`CMD ["sh","-c","service nginx start"]`,因此当sh进程结束，它就会结束。当然即使你用格式1清楚指明service为可执行程序也是不行的，正确做法是指明前台形式运行： `CMD ["nginx","-g","daemon off;"]`
2) 整个Dockerfile应该只出现一次该命令
## ENTRYPOINT入口点
**说明：**该命令和CMD一样，都是指定容器启动程序及参数，不过有下面几点需要注意：
1) 在运行时也可以替代，不过要加个`--entrypoint`指定
2) 当指定了`ENTRYPOINT`后，CMD的内容将作为参数传给`ENTRYPOINT`
3) 整个Dockerfile应该只出现一次该命令
**应用场景：**
1) 让镜像变成像命令一样使用:
```bash
FROM ubuntu:18.04
RUN apt-get update \
    && apt-get install -y curl \
    && rm -rf /var/lib/apt/lists/*
CMD [ "curl", "-s", "http://myip.ipip.net" ]
```
如果我们希望显示http头信息，需要加上-i参数,`docker run myip -i`是行不通的，而必须用`$ docker run myip curl -s http://myip.ipip.net -i`，显然很繁琐，如果把上面内容的CMD改成ENTRYPOINT，就可以使用`docker run myip -i`，此时CMD的内容是`-i`
2) 应用运行前的准备工作。
## ENV设置环境变量(容器运行时，这些环境变量保持有效)
- **格式：**`ENV <key> <value>`
- 如果key或value有空格，用`"`括起来,可以用`\`换行
- 下列指令可以支持环境变量展开：`ADD、COPY、ENV、EXPOSE、FROM、LABEL、USER、WORKDIR、VOLUME、STOPSIGNAL、ONBUILD、RUN。`
## ARG构建参数(容器运行时，这些参数无效)
- **格式：**`ARG <参数名>[=<默认值>]`
- **说明：**定义参数名称及其默认值，该默认值可以在构建命令中用`--build-arg <参数名>=<值>`来覆盖
- ARG指令有生效范围，如果在FROM指令之前指定，那么只能用于每个FROM指令中。多阶段中使用这些变量必须在每个阶段分别指定
## VOLUME定义匿名卷
- **格式：**`VOLUME ["<路径1>","<路径2>"...]`
- **说明：**路径1等目录会在容器运行时默认自动挂载为匿名卷，任何向指定路径中写入的信息都不会记录进容器存储层。
- 运行容器时可以覆盖这个挂载配置`$ docker run -d -v mydata:/data xxxx`，这里就用了mydata这个命名卷挂载到了/data这个位置，替代了Dockerfile中定义的匿名卷挂载配置
## EXPOSE声明端口
- **格式：**`EXPOSE <端口1> <端口2> ...`
- **作用：**这只是个声明，并不会开启这个端口的服务，主要帮助镜像使用者理解这个镜像服务的守护端口，方便映射，另外-P选项，会自动随机映射EXPOSE的端口，要想映射端口请使用-p选项
## WORKDIR指定工作目录
- **格式：**`WORKDIR <工作目录路径>`
- **作用：**指定当前目录，以后各层的当前目录就改为指定目录，如果目录不存在，会自动建立，若指定相对路径，则是在之前工作目录的基础上的。
- **注意：**单独一层cd不会影响之后的一层，因为一层一层是独立的，应该用WORKDIR
## USER指定当前用户
- **格式：**`USER <用户名>[:<用户组>]`
- **作用：**改变之后层的命令执行的身份，这个用户身份必须存在
## HEALTHCHECK健康检查
- **格式1：**`HEALTHCHECK [选项] CMD <命令>`设置检查容器健康状况的命令
- **格式2：**`HEALTHCHECK NONE`如果基础镜像有健康检查指令，使用这个可以屏蔽其健康检查指令
- **历史原因：**在没有该命令之前，docker通过主进程是否退出来判断是否异常，这忽略了一种情形，如果程序进入死锁或死循环，就不会检查出错误
- **功能作用：**当一个镜像指定了HEALTHCHECK指令后，启动容器的初始状态会是starting,在HEALTHCHECK指令检查成功后变为healthy,如果连续一定次数失败，则为unhealthy,**整个Dockerfile应该只出现一次该命令**
- **选项：**

|选项|含义|
|:-:|:-:|
|--interval=<间隔>|两次健康检查的间隔，默认为30秒|
|--timeout=<时长>|健康检查命令运行超时时间，默认为30秒|
|--retries=<次数>|当连续失败指定次数后，则将认定为unhealthy，默认为3次|

## LABEL指令
- **格式：**`LABEL <key>=<value> <key>=<value> <key>=<value> ...`
- **作用：**给镜像以键值对的形式添加些元数据，如镜像的作者、文档地址等
```bash
LABEL org.opencontainers.image.authors="yeasy"
LABEL org.opencontainers.image.documentation="https://yeasy.gitbooks.io"
```
## SHELL指令
- **格式：**`SHELL ["executable", "parameters"]`
- **作用：**指定RUN、ENTRYPOINT、CMD指令的shell,默认为`["/bin/sh","-c"]`,其中ENTRYPOINT,CMD只有以shell格式指定时，才起作用
## ONBUILD为他人做嫁衣
- **格式：**`ONBUILD <其他指令>`
- **作用：**它后面跟的是其它指令，比如 RUN, COPY 等，而这些指令，在当前镜像构建时并不会被执行。只有当以当前镜像为基础镜像，去构建下一级镜像的时候才会被执行。
- 这常常应用于基础镜像，例如npm包管理
## 多阶段构建
1) 之前多阶段构建一种方式把所有东西放在一个Dockerfile中，但这会造成层次太多，镜像体积过大，部署时间变长，源代码存在泄露问题，如下：
```bash
FROM golang:alpine

RUN apk --no-cache add git ca-certificates

WORKDIR /go/src/github.com/go/helloworld/

COPY app.go .

RUN go get -d -v github.com/go-sql-driver/mysql \
  && CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o app . \
  && cp /go/src/github.com/go/helloworld/app /root

WORKDIR /root/

CMD ["./app"]
```
2) 第二种方式分散多个Dockerfile，然后再写脚本整合，虽然镜像体积较小，但过程较复杂：
```bash
# 构建dockerfile-->Dockerfile.build
FROM golang:alpine

RUN apk --no-cache add git

WORKDIR /go/src/github.com/go/helloworld

COPY app.go .

RUN go get -d -v github.com/go-sql-driver/mysql \
  && CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o app .
# 部署dockerfile-->Dockerfile.copy
FROM alpine:latest

RUN apk --no-cache add ca-certificates

WORKDIR /root/

COPY app .

CMD ["./app"]
# 整合脚本--> build.sh
#!/bin/sh
echo Building go/helloworld:build

docker build -t go/helloworld:build . -f Dockerfile.build

docker create --name extract go/helloworld:build
docker cp extract:/go/src/github.com/go/helloworld/app ./app
docker rm -f extract

echo Building go/helloworld:2

docker build --no-cache -t go/helloworld:2 . -f Dockerfile.copy
rm ./app
```
3) 更加高效的方式
```bash
FROM golang:alpine as builder

RUN apk --no-cache add git

WORKDIR /go/src/github.com/go/helloworld/

RUN go get -d -v github.com/go-sql-driver/mysql

COPY app.go .

RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o app .

FROM alpine:latest as prod

RUN apk --no-cache add ca-certificates

WORKDIR /root/

# 这里的0代表上个阶段的镜像,当然可以指定其他镜像如`--from=nginx:latest`
COPY --from=0 /go/src/github.com/go/helloworld/app .

CMD ["./app"]
```
可以只构建某个阶段的镜像`$ docker build --target builder -t username/imagename:tag .`
## docker manifest--多系统架构支持
- **背景：**使用镜像创建一个容器，该镜像必须与Docker宿主机架构一致（Windows、macOS除外，在x86_64系统上，这两个系统可以运行arm等其他架构），为了支持多个架构，必须提供两个架构版本的镜像，manifest命令就是支持自动识别宿主机架构，然后拉取合适的镜像。
### 构建镜像
在两个架构上构建两个镜像
### 创建manifest列表
```bash
# $ docker manifest create MANIFEST_LIST MANIFEST [MANIFEST...]
$ docker manifest create username/test \
      username/x8664-test \
      username/arm64v8-test
```
当需要修改时加个-a或--amend参数
### 设置manifest列表
```bash
# $ docker manifest annotate [OPTIONS] MANIFEST_LIST MANIFEST
$ docker manifest annotate username/test \
      username/x8664-test \
      --os linux --arch x86_64

$ docker manifest annotate username/test \
      username/arm64v8-test \
      --os linux --arch arm64 --variant v8
```
### 查看manifest支持列表
`docker manifest inspect username/test`
### 推送manifest列表
`docker manifest push username/test`
## tips
- 如果想以root临时执行一个命令，不要使用su、sudo,而是使用gosu,见如下例子：
```bash
# 建立 redis 用户，并使用 gosu 换另一个用户执行命令
RUN groupadd -r redis && useradd -r -g redis redis
# 下载 gosu
RUN wget -O /usr/local/bin/gosu "https://github.com/tianon/gosu/releases/download/1.12/gosu-amd64" \
    && chmod +x /usr/local/bin/gosu \
    && gosu nobody true
# 设置 CMD，并以另外的用户执行
CMD [ "exec", "gosu", "redis", "redis-server" ]
```
