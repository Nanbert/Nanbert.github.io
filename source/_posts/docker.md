---
title: docker
date: 2022-04-04 03:51:08
subtitle:
categories:
tags:
index_img: /img/docker.png
banner_img: /img/docker.png
---

## 镜像
### 拉取镜像
- 格式:`docker pull [选项] [Docker Registry 地址[:端口号]/]仓库名[:标签]`
- 默认地址是`docker.io`,仓库名默认为`library`,即官方镜像，标签默认是`latest`
- 简化版：`docker pull ubuntu`
### 列出镜像
- 一般命令：`docker image ls`

|选项|含义|举例|
|:-:|:-:|:-:|
|-a|显示包括中间层镜像在内的所有镜像||
|-f/--fliter|过滤参数,见下||
|--format|自定义格式，见|`docker image ls --format "{{.ID}}: {{.Repository}}"`|

#### -f选项
- since/before: `docker image ls -f since=mongo:3.2`,列出mongo:3.2之后建立的镜像
- label: `docker image ls -f label=com.example.version=0.1`
#### `<none>`虚悬镜像
- 原因：docker pull或docker build导致镜像名被转移到最新版
- 只列出虚悬镜像：`docker image ls -f dangling=true`
- 删除虚悬镜像：`docker image prune`
### 删除镜像
- 格式：`docker image rm [选项] <镜像1> [<镜像2> ...]`,镜像可以是长/短id,镜像名或镜像摘要
#### untagged和deleted
镜像对应多个tag,，只有所有tag都为untagged时，才真正删除，如果一个容器依赖一个镜像，要先删除容器，才能再删除镜像
### docker commit-添加一层形成新的镜像（慎用）
- 说明：该命令会使一个容器形成一个新的镜像,最好不要这样，而是使用Dockerfile
- 格式：`docker commit [选项] <容器ID或容器名> [仓库名:标签]`

|选项|含义|
|:-:|:-:|
|--author|说明作者|
|--message|添加说明信息|
#### docker history
- 说明：该命令用来查看某个镜像的提交历史
- 示例：`docker history [仓库名:标签]`


## 容器
### 运行容器`docker run`

|选项|含义|
|:-:|:-:|
|-i|交互式操作|
|-t|代表终端|
|--rm|容器退出后随之将其删除|

## 常用命令
- `docker system df`查看镜像、容器、数据卷所占用空间
