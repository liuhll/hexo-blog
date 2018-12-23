---
title: Docker之基础
date: 2018-10-01 18:48:57
categories: "docker"
tags:
- docker
- 容器
---

## 什么是Docker?
Docker是一个开源的项目,它基于`Go`语言实现。 项目后来加入了 Linux 基金会，遵从了 Apache 2.0 协议，项目代码在 [GitHub](https://github.com/moby/moby) 上进行维护。
Docker项目的 **目标**是 实现轻量级的操作系统虚拟化解决方案。 Docker 的基础是Linux容器（`LXC`）等技术。

在`LXC`的基础上 Docker 进行了进一步的封装，让用户不需要去关心容器的管理，使得操作更为简便。用户操作Docker的容器就像操作一个快速轻量级的虚拟机一样简单。

###  Docker和传统虚拟机的比较

- 虚拟机
![虚拟机](https://doc.yonyoucloud.com/doc/docker_practice/_images/virtualization.png)
- Docker 
![Docker](https://doc.yonyoucloud.com/doc/docker_practice/_images/docker.png)

### Docker适用的场景 
官网上还有这么一句话"Docker is an open platform for developers and sysadmins to build, ship, and run distributed applications, whether
on laptops, data center VMs, or the cloud."， 可以概括为: **Build Once,Run AnyWhere.(构建一次,在任意地方运行)**，通常应用如下场景:

- web应用的自动化打包和发布；
- 自动化测试和持续集成、发布；
- 在服务型环境中部署和调整数据库或其他的后台应用；
- 从头编译或者扩展现有的OpenShift或Cloud Foundry平台来搭建自己的`PaaS环境`。

### Docker版本

#### Docker社区版(DockerCE)
Docker社区版（Docker CE）是开发人员和小团队的理想选择，希望开始使用Docker并尝试使用基于容器的应用程序。Docker CE在许多平台上可用，从桌面到云端到服务器。Docker CE可用于macOS和Windows，并提供本地体验，以帮助您专注于学习Docker。您可以在单一环境中构建和共享容器并自动化开发管道。

#### Docker企业版(Docker EE)
Docker企业版（Docker EE）专为企业开发和IT团队而设计，这些团队在规模生产中构建，运送和运行关键业务应用程序。Docker EE集成，认证和支持，为企业提供业界最安全的集装箱平台，使所有应用程序现代化。Docker CE具有稳定和边缘通道。

## 镜像

### 介绍
一个只读层被称为镜像(`Image`)，一个镜像是永久不变的。由于 Docker 使用一个统一文件系统，Docker 进程认为整个文件系统是以读写方式挂载的。 但是所有的变更都发生顶层的可写层，而下层的原始的只读镜像文件并未变化。由于镜像不 可写，所以镜像是无状态的。
![Image](https://doc.yonyoucloud.com/doc/chinese_docker/terms/images/docker-filesystems-debianrw.png)

#### 父镜像
每一个镜像都可能依赖于由一个或多个下层的组成的另一个镜像。我们有时说，下层那个镜像是上层镜像的父镜像。
![父镜像](https://doc.yonyoucloud.com/doc/chinese_docker/terms/images/docker-filesystems-multilayer.png)

#### 基础镜像
一个没有任何父镜像的镜像，谓之基础镜像。

#### 镜像ID
所有镜像都是通过一个 64 位十六进制字符串 （内部是一个 256 bit 的值）来标识的。 为简化使用，前 12 个字符可以组成一个短ID，可以在命令行中使用。短ID还是有一定的 碰撞机率，所以服务器(Docker Hub)总是返回长ID。


### 获取镜像
可以使用 `docker pull` 命令来从仓库(`Docker Hub`)获取所需要的镜像。
例如:
```shell
$ sudo docker pull ubuntu:12.04
```

由于防火长城的原因，如果从官方仓库中喜爱在较慢，可以从其他仓库下载。从其它仓库下载时需要指定完整的仓库注册服务器地址。

```shell
$ sudo docker pull dl.dockerpool.com:5000/ubuntu:12.04
```

#### 列出本地镜像
使用 `docker images` 显示本地已有的镜像。
例如:
```shell
PS C:\Users\Administrator> docker images
REPOSITORY                       TAG                      IMAGE ID            CREATED             SIZE
demo                             lasted                   66c0412035cd        18 hours ago        494MB
demo1                            dev                      6ed4d19a9352        18 hours ago        494MB
```
列出的镜像信息字段说明:
- `REPOSITORY` 来自于哪个仓库，比如 ubuntu
- `TAG`镜像的标记，比如 14.04
- `IMAGE ID`它的 ID 号（唯一）,如果`镜像Id`相同，就说明是同一个镜像
- `CREATED`创建时间
- `SIZE`镜像大小

### 创建镜像

#### 通过现有的容器修改创建
当修改了一个运行中的容器后，使用 `docker commit` 命令来提交更新后的副本。
```shell
sudo docker commit -m "Added json gem" -a "Docker Newbee" 0b2616b0e5a8 ouruser/sinatra:v2
```
- `-m` 来指定提交的说明信息，跟我们使用的版本控制工具一样；
- `-a` 可以指定更新的用户信息；之后是用来创建镜像的容器的 ID；
- 最后指定目标镜像的仓库名和 `tag` 信息。
- 创建成功后会返回这个镜像的 ID 信息。

> **Notes** 在开发过程中不推荐使用该种方式维护镜像，因为这么做修改镜像非常不方便，且不容易分享，在开发过程中，一般使用`Dockerfile`来维护Docker 镜像。

#### 利用 Dockerfile 来创建镜像
通过创建文件`Dockerfile`和指令维护镜像。

```shell
# This is a comment
FROM ubuntu:14.04
MAINTAINER Docker Newbee <newbee@docker.com>
RUN apt-get -qq update
RUN apt-get -qqy install ruby ruby-dev
RUN gem install sinatra
```
`Dockerfile` 中每一条指令都创建镜像的一层。
编写完成 `Dockerfile` 后可以使用 `docker build` 来生成镜像。

```shell
$sudo docker build -t="ouruser/sinatra:v2" .
```
- `-t` 标记来添加 `tag`，指定新的镜像的用户信息。 
- `.` 是 `Dockerfile` 所在的路径（当前目录），也可以替换为一个具体的 `Dockerfile` 的路径。

> **Notes**
> 注意一个镜像不能超过 127 层

#### 从本地文件系统导入
从本地文件系统导入一个镜像，可以使用 `openvz`（容器虚拟化的先锋技术）的模板来创建： `openvz` 的模板下载地址为 `templates`。
例如:先下载了一个 ubuntu-14.04 的镜像，之后使用以下命令导入

```shell
sudo cat ubuntu-14.04-x86_64-minimal.tar.gz  |docker import - ubuntu:14.04
```

### 存入和载入镜像

#### 存出镜像
如果要导出镜像到本地文件，可以使用 `docker save` 命令。

```shell
$sudo docker save -o ubuntu_14.04.tar ubuntu:14.04
```
#### 载入镜像
可以使用 `docker load` 从导出的本地文件中再导入到本地镜像库,这将导入镜像以及其相关的元数据信息（包括标签等）
```shell
$ sudo docker load --input ubuntu_14.04.tar
# 或
$ sudo docker load < ubuntu_14.04.tar
```

#### 移除本地镜像
使用 `docker rmi` 命令

> **Notes**
> 在删除镜像之前要先用 `docker rm` 删掉依赖于这个镜像的所有容器。

### 镜像的实现原理
每个镜像都由很多层次构成，Docker 使用`Union FS`将这些不同的层结合到一个镜像中。通常 Union FS 有两个用途：
- 一方面可以实现不借助`LVM`、`RAID` 将多个 `disk` 挂到同一个目录下
- 另一个更常用的就是将一个只读的分支和一个可写的分支联合在一起，Live CD 正是基于此方法可以允许在镜像不变的基础上允许用户在其上进行一些写操作。

## 容器
容器的核心为所执行的应用程序,所需要的资源都是应用程序运行所必需的。除此之外，并没有其它的资源。这种特点使得 Docker 对资源的利用率极高，是货真价实的轻量级虚拟化。
### 启动容器
启动容器有两种方式:
#### 新建并启动
命令: `docker run`
```shell
sudo docker run ubuntu:14.04 /bin/echo 'Hello world'
```
启动并进入容器内部:
```shell
sudo docker run -t -i ubuntu:14.04 /bin/bash
```
> **Notes:**
> - `-t` 选项让Docker分配一个伪终端（pseudo-tty）并绑定到容器的标准输入上
> - `-i` 则让容器的标准输入保持打开

使用`docker run`创建容器时,docker在后台的操作包括:
  1. 检查本地是否存在指定的镜像，不存在就从公有仓库下载
  2. 利用镜像创建并启动一个容器
  3. 分配一个文件系统，并在只读的镜像层外面挂载一层可读写层
  4. 从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中去
  5. 从地址池配置一个 ip 地址给容器
  6. 执行用户指定的应用程序
  7. 执行完毕后容器被终止

#### 启动已终止容器

利用 `docker start` 命令，直接将一个已经终止的容器启动运行。
利用 `docker restart` 命令会将一个运行态的容器终止，然后再重新启动它

### 守护态运行
更多的时候，需要让 Docker 容器在后台以守护态（Daemonized）形式运行。此时，可以通过添加`-d`参数来实现。


### 终止容器
- 可以使用`docker stop`来终止一个运行中的容器。
- 当Docker容器中指定的应用终结时，容器也自动终止。

终止状态的容器可以用 `docker ps -a` 命令看到。


### 进入容器
在使用 -d 参数时，容器启动后会进入后台。 某些时候需要进入容器进行操作。

> **Notes**
> 只用处于运行中的容器才可能进入到容器内部

#### 利用attach命令进入容器内部
`docker attach` 是Docker自带的命令，通过指定的`容器Id`、`容器名称`进入到容器内部。
使用`attach`命令的缺陷:
- 当多个窗口同时`attach`到同一个容器的时候，所有窗口都会同步显示。当某个窗口因命令阻塞时,其他窗口也无法执行操作了

#### 使用nsenter工具
- 如果系统中没有存在`nsenter`命令，需要先安装`nsenter`工具。

使用：
`nsenter` 可以访问另一个进程的名字空间。nsenter 要正常工作需要有`root`权限。

1. 为了连接到容器，你还需要找到容器的第一个进程的`PID`，可以通过下面的命令获取
```shell
PID=$(docker inspect --format "{{ .State.Pid }}" <container>)
```
2. 通过这个 `PID`，就可以连接到这个容器：
```shell
nsenter --target $PID --mount --uts --ipc --net --pid
```

#### .bashrc_docker
下载 `.bashrc_docker`，并将内容放到 `.bashrc` 中
```shell
$ wget -P ~ https://github.com/yeasy/docker_practice/raw/master/_local/.bashrc_docker;
$ echo "[ -f ~/.bashrc_docker ] && . ~/.bashrc_docker" >> ~/.bashrc; source ~/.bashrc
```
然后通过`docker-enter`可以进入容器或直接在容器内执行命令

### 导出和导入容器
#### 导出容器快照
导出本地某个容器，可以使用`docker export`命令。
```shell
sudo docker export <containerid> > ubuntu.tar
```
#### 导入容器快照
使用`docker import`从容器快照文件中再导入为镜像。
```shell
cat ubuntu.tar | sudo docker import - test/ubuntu:v1.0
# 也可以通过指定 URL 或者某个目录来导入
sudo docker import http://example.com/exampleimage.tgz example/imagerepo
```
> **Notes:**
> 用户既可以使用 `docker load` 来导入镜像存储文件到本地镜像库，也可以使用 `docker import` 来导入一个容器快照到本地镜像库
> 区别:
> - 容器快照文件将丢弃所有的历史记录和元数据信息（即仅保存容器当时的快照状态），而镜像存储文件将保存完整记录，体积也要大。
> - 从容器快照文件导入时可以重新指定标签等元数据信息

### 删除容器
使用 `docker rm` 来删除一个处于终止状态的容器

> **Notes**
> 如果要删除一个运行中的容器，可以添加 `-f`参数。Docker 会发送 `SIGKILL` 信号给容器

## Docker 注册表和仓库
### Docker Registry简介
我们需要一个集中的存储、分发镜像的服务，`Docker Registry`就是这样的服务。

一个 `Docker Registry` 中可以包含多个仓库（`Repository`）；每个仓库可以包含多个标签（Tag）；每个标签对应一个镜像。

一个仓库会包含同一个软件不同版本的镜像，而标签就常用于对应该软件的各个版本。我们可以通过 `<仓库名>:<标签>` 的格式来指定具体是这个软件哪个版本的镜像。如果不给出标签，将以`latest`作为默认标签。

仓库名经常以 两段式路径 形式出现，比如 `jwilder/nginx-proxy`，前者往往意味着 Docker Registry 多用户环境下的用户名，后者则往往是对应的软件名。但这并非绝对，取决于所使用的具体 Docker Registry 的软件或服务。

### Docker Registry 公开服务
Docker Registry 公开服务是开放给用户使用、允许用户管理镜像的 Registry 服务。一般这类公开服务允许用户免费上传、下载公开的镜像，并可能提供收费服务供用户管理私有镜像。

####  Docker Hub
Docker是默认的Docker Registry，也是官方提供的默认`Docker Registry`，拥有大量的高质量的官方镜像。除此以外，还有 CoreOS 的 Quay.io，CoreOS 相关的镜像存储在这里。

国内的一些云服务商提供了针对 `Docker Hub` 的镜像服务（`Registry Mirror`），这些镜像服务被称为加速器。常见的有 阿里云加速器、DaoCloud 加速器 等。使用加速器会直接从国内的地址下载 Docker Hub 的镜像，比直接从 Docker Hub 下载速度会提高很多

#### 修改默认的Docker Registry
1. Linux系统
  - 单次修改
  ```shell
  # 格式
  docker pull registry.docker-cn.com/myname/myrepo:mytag
  # 例如
  docker pull registry.docker-cn.com/library/ubuntu:16.04
  ```
  - 永久修改
  修改：`/etc/docker/daemon.json`增加如下内容
  ```json
  {
    "registry-mirrors": ["https://registry.docker-cn.com"]
  }
  ```
2. Windows系统
![仓库修改](https://img-blog.csdn.net/20180411111418128?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE1ODA3MTY3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)


### 私有 Docker Registry

用户还可以在本地搭建私有`Docker Registry`。Docker 官方提供了`Docker Registry`镜像，可以直接使用做为私有 Registry 服务
