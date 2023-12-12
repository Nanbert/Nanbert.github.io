---
title: docker
date: 2022-04-04 03:51:08
subtitle:
categories:
tags:
index_img: /images/docker.png
banner_img: /images/docker.png
---
## 各个文件位置
- 本地资源的默认总目录：`/var/lib/docker/`,**迁移**docker就是复制该文件夹，**重置**就是删除该文件夹
- 容器信息：`containers`
- 镜像信息：`image`
- 镜像层文件：`overlay2`
### 修改文件位置
- 方式1：为默认位置建立软链接
- 方式2：daemon启动时指定-g选项
- 方式3：修改/etc/docker/daemon.json的"data-root"项
## 镜像加速
- 查看是否在docker.service文件中配置过镜像地址：`$ systemctl cat docker | grep '\-\-registry\-mirror'`
- 上一步如果出现输出内容，修改文件去掉内容`--registry-mirror`，按接下来步骤配置
- 在/etc/docker/daemon.json中写入下面内容：
```bash
{
  "registry-mirrors": [
    "https://hub-mirror.c.163.com",
    "https://mirror.baidubce.com"
  ]
}
```
- 重启服务：`sudo systemctl daemon-reload;sudo systemctl restart docker`
- 检查是否生效：`docker info`
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
### 容器运行流程
1) 检查本地是否存在指定的镜像，不存在就从registry下载
2) 利用镜像创建并启动一个容器
3) 分配一个文件系统，并在只读的镜像层外面挂载一层可读写层
4) 从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中去
5) 从地址池配置一个ip地址给容器
6) 执行用户指定的应用程序
7) 执行完毕后容器被终止
### 运行容器`docker run`
- **格式：**`docker run [OPTIONS] image`

|选项|含义|
|:-:|:-:|
|-c/--cpu-shares|调整容器使用cpu的权重|
|-d|后台运行，此时容器不会把输出的结果打印到宿主机上面，`docker container logs <container id or name>`可以查看结果|
|--dns=[ip_address]|设置dns服务器|
|--dns-search=[DOMAIN]|设定容器搜索域|
|--env [VAR=value]|传递环境变量|
|--env-file [file]|从file中引入环境变量|
|-h/--hostname=[HOSTNAME]|设定容器主机名，比较鸡肋，只能在容器内部显示|
|-i|交互式操作,让容器的标准输入保持打开|
|-m|--memory[=MEMORY]|调整使用内存大小|
|--mount|挂载数据卷或主机目录|
|--name [name]|指定容器的名字|
|--network [my-net]|见连接容器|
|-p|见端口映射|
|-P|随机映射镜像的端口|
|--rm|容器退出后随之将其删除|
|-t|代表终端，让Docker分配一个伪终端并邦到标准输入|
|-u=[username]|设置进程用户名|
|-v [宿主机路径]:[容器路径]|映射路径|
|--volumes-from|从别的容器中挂载卷|

### 查看容器的信息--docker container ls
- **格式：**`docker container ls`
- -a:列出终止状态的容器
### 终止容器--docker container stop
- **格式：**`docker container stop [container id or names]`
### 启动已终止容器--docker container start
- **格式：**`docker container start [container id or names]`
### 重启容器--docker container restart
- **格式：**`docker container restart [container id or names]`
### 查看容器的日志--docker container logs
- **格式：**`docker container logs [container ID or NAMES]`
### 进入后台运行的容器--docker attach/exec
- 建议使用exec,使用attach,从stdin退出，会导致容器的停止，exec就没这个问题
- attach格式：`docker attach [id or names]`
#### docker exec
- **格式：**`docker exec -it [id or names]`
- -it选项和docker run一样
### 导出、导入容器--docker export/import
- 导出容器：`docker export [id] > xx.tar`
- 导入容器快照成镜像：`cat xx.tar | docker import - test/ubuntu:v1.0`,支持url:`$ docker import http://example.com/exampleimage.tgz example/imagerepo`
- docker load和docker import:docker import会丢弃所有历史记录和元数据信息，docker load则会保留完整记录
### 删除容器--docker container rm
- **格式：**`docker container rm [container id or names]`
- -f:删除一个运行中的容器
## 仓库
### Docker Hub
- 注册：在官网https://hub.docker.com免费注册一个Docker帐号
- 登录：`docker login`命令交互式输入用户名和密码
- 搜索：`docker search [关键字]`支持`--filter=stars=N`仅显示收藏数量N以上的镜像
- 拉取：见镜像拉取
- 推送：推送前先标记下`dockeer tag [image:tag] [username/xx:xx]`，`docker push username/xx:xx`
### 私有仓库-->docker-registry
- 使用官方registry镜像：`$ docker run -d -p 5000:5000 --restart=always --name registry registry`
- 默认情况下仓库会在容器中的`/var/lib/registry`目录下,可以用-v选项映射
- 标记镜像`git tag`:`docker tag IMAGE[:TAG] [REGISTRY_HOST[:REGISTRY_PORT]/]REPOSITORY[:TAG]`
- 推送：`docker push 127.0.0.1:5000/ubuntu:latest`
- 拉取：`docker pull 127.0.0.1:5000/ubuntu:latest`
- 取消非https限制：默认情况下docker不允许非https方式推送，对于使用systemd的系统，可以在/etc/docker/daemon.json中写入如下内容：
```bash
{
  "registry-mirror": [
    "https://hub-mirror.c.163.com",
    "https://mirror.baidubce.com"
  ],
  "insecure-registries": [
    "192.168.199.100:5000"
  ]
}
```
- 私有仓库删除镜像可能不会回收空间，可以使用Nexus3.x软件搭配
## 数据管理
### 数据卷
- 特性
  - 数据卷可以在容器之间共享和重用
  - 对数据卷的修改会立马生效
  - 对数据卷的更新，不会影响镜像
  - 数据卷会默认一直存在，即使容器被删除
- 创建一个数据卷：`docker volume create my-vol`
- 查看数据卷：`docker volume ls`
- 查看指定数据卷信息：`docker volume inspect my-vol`
- 挂载数据卷
```bash
docker run -d -P \
    --name web \
    # -v my-vol:/usr/share/nginx/html \
    --mount source=my-vol,target=/usr/share/nginx/html \
    nginx:alpine
```
- 删除数据卷：`docker volume rm my-vol`
### 挂载主机目录为数据卷
```bash
# 可以不加readonly,那样权限就是读写
docker run -d -P \
    --name web \
    # -v /src/webapp:/usr/share/nginx/html:ro \
    --mount type=bind,source=/src/webapp,target=/usr/share/nginx/html,readonly \
    nginx:alpine
```
可以挂载指定文件，把上面source和target换成文件即可
## 网络
### 端口映射
使用-p选项
- 映射所有接口地址：`-p hostPort:containerPort`
- 映射到指定地址的指定端口：`-p ip:hostPort:containerPort`
- 映射到指定地址的任意端口：`-p ip::containerPort`
- 标记udp: `-p xxport:xxport/udp`
- 查看容器的端口映射：`docker port [id or name] [containerPort]`
### 容器互联(两个容器)
如果大于两个容器的互联建议使用Docker Compose
- 新建网络：`docker network create -d bridge my-net`,-d选项有bridge和overlay(见swarm mode)
- 连接容器：`$ docker run -it --rm --name busybox1 --network my-net busybox sh`,`$ docker run -it --rm --name busybox1 --network my-net busybox2 sh`
- 在各个容器中ping：`ping busybox2`
### DNS配置
容器的DNS设置跟随宿主主机，默认使用主机上的/etc/resolv.conf(--dns-search --dns会改变默认设置),如果配置全部容器的DNS可以在`/etc/docker/daemon.json`增加以下内容设置：
```bash
{
  "dns" : [
    "114.114.114.114",
    "8.8.8.8"
  ]
}
```
## docker命令汇总
### 客户端选项
- --config=""：指定客户端配置文件，默认为 ~/.docker；
- -D=true|false：是否使用 debug 模式。默认不开启；
- -H, --host=[]：指定命令对应 Docker 守护进程的监听接口，可以为 unix 套接字 unix:///path/to/socket，文件句柄 fd://socketfd 或 tcp 套接字 tcp://[host[:port]]，默认为 unix:///var/run/docker.sock；
- -l, --log-level="debug|info|warn|error|fatal"：指定日志输出级别；
- --tls=true|false：是否对 Docker 守护进程启用 TLS 安全机制，默认为否；
- --tlscacert=/.docker/ca.pem：TLS CA 签名的可信证书文件路径；
- --tlscert=/.docker/cert.pem：TLS 可信证书文件路径；
- --tlscert=/.docker/key.pem：TLS 密钥文件路径；
- --tlsverify=true|false：启用 TLS 校验，默认为否。
### 客户端命令
- attach：依附到一个正在运行的容器中；
- build：从一个 Dockerfile 创建一个镜像；
- commit：从一个容器的修改中创建一个新的镜像；
- cp：在容器和本地宿主系统之间复制文件中；
- create：创建一个新容器，但并不运行它；
- diff：检查一个容器内文件系统的修改，包括修改和增加；
- events：从服务端获取实时的事件；
- exec：在运行的容器内执行命令；
- export：导出容器内容为一个 tar 包；
- history：显示一个镜像的历史信息；
- images：列出存在的镜像；
- import：导入一个文件（典型为 tar 包）路径或目录来创建一个本地镜像；
- info：显示一些相关的系统信息；
- inspect：显示一个容器的具体配置信息；
- kill：关闭一个运行中的容器 (包括进程和所有相关资源)；
- load：从一个 tar 包中加载一个镜像；
- login：注册或登录到一个 Docker 的仓库服务器；
- logout：从 Docker 的仓库服务器登出；
- logs：获取容器的 log 信息；
- network：管理 Docker 的网络，包括查看、创建、删除、挂载、卸载等；
- node：管理 swarm 集群中的节点，包括查看、更新、删除、提升/取消管理节点等；
- pause：暂停一个容器中的所有进程；
- port：查找一个 nat 到一个私有网口的公共口；
- ps：列出主机上的容器；
- pull：从一个Docker的仓库服务器下拉一个镜像或仓库；
- push：将一个镜像或者仓库推送到一个 Docker 的注册服务器；
- rename：重命名一个容器；
- restart：重启一个运行中的容器；
- rm：删除给定的若干个容器；
- rmi：删除给定的若干个镜像；
- run：创建一个新容器，并在其中运行给定命令；
- save：保存一个镜像为 tar 包文件；
- search：在 Docker index 中搜索一个镜像；
- service：管理 Docker 所启动的应用服务，包括创建、更新、删除等；
- start：启动一个容器；
- stats：输出（一个或多个）容器的资源使用统计信息；
- stop：终止一个运行中的容器；
- swarm：管理 Docker swarm 集群，包括创建、加入、退出、更新等；
- tag：为一个镜像打标签；
- top：查看一个容器中的正在运行的进程信息；
- unpause：将一个容器内所有的进程从暂停状态中恢复；
- update：更新指定的若干容器的配置信息；
- version：输出 Docker 的版本信息；
- volume：管理 Docker volume，包括查看、创建、删除等；
- wait：阻塞直到一个容器终止，然后输出它的退出符。
### 服务端命令选项
- --api-cors-header=""：CORS 头部域，默认不允许 CORS，要允许任意的跨域访问，可以指定为 `*`；
- --authorization-plugin=""：载入认证的插件；
- -b=""：将容器挂载到一个已存在的网桥上。指定为 none 时则禁用容器的网络，与 --bip 选项互斥；
- --bip=""：让动态创建的 docker0 网桥采用给定的 CIDR 地址; 与 -b 选项互斥；
- --cgroup-parent=""：指定 cgroup 的父组，默认 fs cgroup 驱动为 /docker，systemd cgroup 驱动为 system.slice；
- --cluster-store=""：构成集群（如 Swarm）时，集群键值数据库服务地址；
- --cluster-advertise=""：构成集群时，自身的被访问地址，可以为 host:port 或 interface:port；
- --cluster-store-opt=""：构成集群时，键值数据库的配置选项；
- --config-file="/etc/docker/daemon.json"：daemon 配置文件路径；
- --containerd=""：containerd 文件的路径；
- -D, --debug=true|false：是否使用 Debug 模式。缺省为 false；
- --default-gateway=""：容器的 IPv4 网关地址，必须在网桥的子网段内；
- --default-gateway-v6=""：容器的 IPv6 网关地址；
- --default-ulimit=[]：默认的 ulimit 值；
- --disable-legacy-registry=true|false：是否允许访问旧版本的镜像仓库服务器；
- --dns=""：指定容器使用的 DNS 服务器地址；
- --dns-opt=""：DNS 选项；
- --dns-search=[]：DNS 搜索域；
- --exec-opt=[]：运行时的执行选项；
- --exec-root=""：容器执行状态文件的根路径，默认为 /var/run/docker；
- --fixed-cidr=""：限定分配 IPv4 地址范围；
- --fixed-cidr-v6=""：限定分配 IPv6 地址范围；
- -G, --group=""：分配给 unix 套接字的组，默认为 docker；
- -g, --graph=""：Docker 运行时的根路径，默认为 /var/lib/docker；
- -H, --host=[]：指定命令对应 Docker daemon 的监听接口，可以为 unix 套接字 unix:///path/to/socket，文件句柄 fd://socketfd 或 tcp 套接字 tcp://[host[:port]]，默认为 unix:///var/run/docker.sock；
- --icc=true|false：是否启用容器间以及跟 daemon 所在主机的通信。默认为 true。
- --insecure-registry=[]：允许访问给定的非安全仓库服务；
- --ip=""：绑定容器端口时候的默认 IP 地址。缺省为 0.0.0.0；
- --ip-forward=true|false：是否检查启动在 Docker 主机上的启用 IP 转发服务，默认开启。注意关闭该选项将不对系统转发能力进行任何检查修改；
- --ip-masq=true|false：是否进行地址伪装，用于容器访问外部网络，默认开启；
- --iptables=true|false：是否允许 Docker 添加 iptables 规则。缺省为 true；
- --ipv6=true|false：是否启用 IPv6 支持，默认关闭；
- -l, --log-level="debug|info|warn|error|fatal"：指定日志输出级别；
- --label="[]"：添加指定的键值对标注；
- --log-driver="json-file|syslog|journald|gelf|fluentd|awslogs|splunk|etwlogs|gcplogs|none"：指定日志后端驱动，默认为 json-file；
- --log-opt=[]：日志后端的选项；
- --mtu=VALUE：指定容器网络的 mtu；
- -p=""：指定 daemon 的 PID 文件路径。缺省为 /var/run/docker.pid；
- --raw-logs：输出原始，未加色彩的日志信息；
- --registry-mirror=<scheme>://<host>：指定 docker pull 时使用的注册服务器镜像地址；
- -s, --storage-driver=""：指定使用给定的存储后端；
- --selinux-enabled=true|false：是否启用 SELinux 支持。缺省值为 false。SELinux 目前尚不支持 overlay 存储驱动；
- --storage-opt=[]：驱动后端选项；
- --tls=true|false：是否对 Docker daemon 启用 TLS 安全机制，默认为否；
- --tlscacert=/.docker/ca.pem：TLS CA 签名的可信证书文件路径；
- --tlscert=/.docker/cert.pem：TLS 可信证书文件路径；
- --tlscert=/.docker/key.pem：TLS 密钥文件路径；
- --tlsverify=true|false：启用 TLS 校验，默认为否；
- --userland-proxy=true|false：是否使用用户态代理来实现容器间和出容器的回环通信，默认为 true；
- --userns-remap=default|uid:gid|user:group|user|uid：指定容器的用户命名空间，默认是创建新的 UID 和 GID 映射到容器内进程。
## 调试docker
- 开启debug模式：在/etc/docker/daemon.json中添加：
```bash
{
	"debug":true	
}
```
- 重启守护进程`sudo kill -SIGHUP $(pidof dockerd)`
- 检查内核日志`sudo dmesg | grep dockerd`,`sudo dmesg|grep runc`
- 不响应时，杀死进程查看堆栈：`sudo kill -SIGUSR1 $(pidof dockerd)`
## 高级网络
## 集群--Swarm mod
## Docker Buildx-->未来docker build
## 常用命令及问题
- `docker system df`查看镜像、容器、数据卷所占用空间
- `docker image prune`批量清理临时镜像
- `docker container prune`删除所有终止状态的容器
- `docker volume prune`删除无主的数据卷
- `docker run [image] env`查看镜像支持的环境变量
- `docker stop $(docker container ls -q)`停止所有正在运行的容器
- `docker inspect --format '{{ .State.Pid }}' <CONTAINER ID or NAME>`获取某个容器的PID信息
- `docker inspect --format '{{ .NetworkSettings.IPAddress }}' <CONTAINER ID or NAME>`获取某个容器的ip地址
- 给容器指定一个固定ip地址
```bash
docker network create -d bridge --subnet 172.25.0.0/16 my-net
docker run --network=my-net --ip=172.25.3.3 -itd --name=my-container busybox
```
- 退出一个交互的终端而不终止，按`ctrl-p ctrl-q`
- Error:No public port 80 published for xxx
  - 创建镜像时 Dockerfile 要通过 EXPOSE 指定正确的开放端口；
  - 容器启动时指定 PublishAllPort = true。
- WARNING: Your kernel does not support cgroup swap limit. WARNING: Your kernel does not support swap limit capabilities. Limitation discarded.
  这是因为系统默认没有开启对内存和swap使用的统计功能，该功能会降低性能，可以通过以下方式开启：
  - 编辑`/etc/default/grub`配置`GRUB_CMDLINE_LINUX="cgroup_enable=memory swapaccount=1"`
  - 更新grub:`sudo update-grub`
  - 重启
