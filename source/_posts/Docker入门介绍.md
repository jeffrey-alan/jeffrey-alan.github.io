---
title: Docker入门介绍
date: 2019-12-17 13:12:52
tags: 
- Docker
categories: 
- 容器化技术
---

# Docker简介

Docker 使用 Google 公司推出的 Go 语言 进行开发实现，基于Linux 内核的 cgroup，namespace，以及 AUFS 类的 Union FS 等技术，对进程进行封装隔离，属于操作系统层面的虚拟化技术。由于隔离的进程独立于宿主和其它的隔离的进程，因此也称其为容器。

<!-- more -->

Docker 在容器的基础上，进行了进一步的封装，从文件系统、网络互联到进程隔离等等，极大的简化了容器的创建和维护。使得 Docker 技术比传统虚拟机技术更为轻便、快捷Docker。

# Docker VS Visual Machine

## Visual Machine
传统虚拟机技术是虚拟出一套硬件后，在其上运行一个完整操作系统，在该系统上再运行所需应用进程。

## Docker
容器内的应用进程直接运行于宿主的内核，容器内没有自己的内核，而且也没有进行硬件虚拟。因此容器比传统虚拟机更轻便。


# 为什么要使用Docker

* 更高效的利用系统资源
* 更快速的启动时间
* 一致的运行环境
* 持续交付和部署
* 更轻松的迁移
* 更轻松的维护和扩展


| 特性 | 容器 | 虚拟机 |
| --- | --- | --- |
| 启动 | 秒级 | 分钟级 |
| 硬盘使用 | MB | GB |
| 性能 | 接近原生 | 较弱 |
| 系统支持量 | 上千个 | 几十个 |

# Docker 架构

![20191217131541](http://tc.llx-cn.com/20191217131541.png)

Docker 使用CS架构模式，使用远程API来管理和创建Docker容器。

# 基本概念
Docker 包括三个基本概念（镜像、容器、仓库），理解了这三个概念，就理解了Docker的整个生命周期。

## 镜像（Image）

我们都知道，操作系统分为内核和用户空间。对于Linux而言，内核启动后，会挂载文件系统为其提供用户空间支持。而 Docker 镜像，相当于是一个 root 文件系统。

比如官方镜像 centos:7.6 就包含了完整的一套 centos 7.6最小系统的 root 文件系统。

Docker 镜像是一个特殊的文件系统，除了提供容器运行时所需的程序、库、资源、配置等文件外，还包含了一些为运行时准备的一些配置和参数（匿名卷、环境变量、用户等）。

镜像不包含任何动态数据，其内容在构建之后也不会被改变。

### 镜像分层存储

因为镜像包含操作系统完整的 root 文件系统，其体积往往是庞大的，因此在 Docker 设计时将其设计为分层存储的架构。镜像只是一个虚拟的概念，其实际体现并非由一个文件组成，而是由一组文件系统组成。

镜像构建时，会一层层构建，前一层是后一层的基础。每一层构建完成就不会再发生改变，后一层上的任何改变只发生在自己这一层。在构建镜像的时候，需要额外小心，每一层尽量只包含该层需要添加的东西，任何额外的东西应该在该层构建结束前清理掉。

分层存储的特征还是得镜像的附庸、定制变得更为容易。甚至可以用之前构建好的镜像作为基础层，然后进一步添加新的层，以定制自己所需的内容，构建新的镜像。

## 容器（Container）

镜像和容器的关系，就像 Java 中的类和实例一样，镜像是静态的定义，容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等。

前面讲过镜像使用的是分层存储，容器也是如此。每一个容器运行时，是以镜像为基础层，在其上创建一个当前容器的存储层，我们可以称这个为容器运行时读写而准备的存储层为容器存储层。

容器存储层的生命周期和容器一样，容器消亡时，容器存储层也随之消亡。因此，任何保存于容器存储层的信息都会随着容器删除而丢失。

按照 Docker 最佳实践的要求，容器不应该向其存储层内写入任何数据，容器存储层要保持无状态化。所有的文件写入操作，都应该使用 Volume 数据卷、或者绑定宿主目录，在这些位置的读写会跳过容器存储层，直接对宿主（或网络存储）发生读写，其性能和稳定性更高。

数据卷的生命周期独立于容器，容器消亡，数据卷不会消亡。因此，使用数据卷后，容器删除或者重新运行之后，数据不会丢失。

## 仓库（Repository）

镜像构建完成后，可以很容易的在当前宿主机上运行，但是，如果需要在其它服务器上使用这个镜像，我们就需要一个集中存储、分发镜像的服务，Docker Registry 就是这个的服务。

一个 Docker Registry 中可以包含多个仓库（Repository）；
每个仓库可以包含多个标签（Tag）；
每个标签对应一个镜像。

通常，一个仓库会包含同一个软件不同版本的镜像，而标签就常用于对应该软件的各个版本。我们可以通过<仓库名>:<标签> 的格式来指定具体是这个软件哪个版本的镜像。如果不给出标签，将以 latest 作为默认标签。

以cengos 镜像为例，cengos 是仓库名称，其内包含不同的版本标签，如6.9、7.5。我们可以通过 cengos:6.9 或者 centos:7.5 来具体指定所需哪个版本的镜像。如果忽略标签，比如 centos,那就将视为 centos:latest。

仓库名经常以两段式路径形式出现，比如 study/nginx，前者往往意味着 Docker Registry 多用户环境下的用户名，后者则往往是对应的软件名。但这并非绝对，取决于所使用的具体 Docker Registry 的软件或服务。

### Registry 公开仓库
常用的Registry 是官方的 Docker Hub，也就是默认的Registry。除此之外，还有CentOS的Quay.io，Centos 相关的镜像存储在这里。Google 的 Google Container Registry，Kubernetes 的镜像使用的就是这个服务。

国内的一些云服务商提供了针对 Docker Hub 的镜像服务，这些镜像服务被称为加速器。常见的有 阿里云加速器、DaoCloud 加速器等。使用加速器会直接从国内的地址下载 Docker Hub 的镜像，比直接从Docker Hub 下载速度会提高很多。

国内也有一些云提供商提供类似于 Docker Hub 的公开服务。比如网易云镜像服务、DaoCloud 镜像市场、阿里云镜像库等。

### Registry 私有仓库

除了使用公开服务外，用户还可以搭建本地私有 Docker Registry。官方还提供了 Docker Registry 镜像，可以直接使用作为私有 Registry 服务。

开源的 Docker Registry 镜像只提供了 Docker Registry API 的服务端实现，足以支持docker命令，不影响使用。但不包含图形界面，以及镜像维护、用户管理、访问控制等高级功能。在官方的商业化版本 Docker Trusted Registry 中，提供了这些高级功能。

除了官方的 Docker Registry 外，还有第三方软件实现了 Docker Registry API，甚至提供了用户界面及一些高级功能。比如VMware Harbor 和 Sonatype Nexus。


# Docker安装

## 版本命名

| 项目 | 说明 |
| --- | --- |
| 版本格式 | YY.MM |
| Stable 版本 | 每个季度发行 |
| Edge 版本 | 每个月发行 |
| 当前 Docker CE Stable 版本 | 18.09 |
| 当前 Docker CE Edge 版本 | 18.09 |

同时 Docker 划分为以下两个版本。

社区版（CE）：免费，支持周期三个月
企业版（EE）：强调安全，付费使用


## 开始安装

### 系统要求

```
// 查看内核版本，不能低于3.10 
uname -r
```

### 卸载旧版本

```
// 旧版本的Docker 称为 docker 或者 docker-engine
sudo yum remove docker docker-common docker-selinux docker-engine
```

### 使用yum安装

```
// 安装依赖
yum install -y yum-utils device-mapper-persistent-data lvm2

// 添加yum源
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

// 开始安装
sudo yum install -y docker-ce
```

### 使用脚本安装

```
// 下载脚本
curl -fsSL https://get.docker.com -o get-docker.sh

// 开始一键安装
sudo sh get-docker.sh --mirror Aliyun
```

### 启动Docker CE

```
// 设置开机启动
sudo systemctl enable docker

// 开启服务
sudo systemctl start docker
```

### 建立 docker 用户组

默认情况下，docker 命令会使用 Unix socket 与 Docker 引擎通讯。而只有 root用户和docker组的用户才可以访问 Docker 引擎的 Unix socket。

一般 Linux 系统上不会直接使用root用户进行操作。因此需要将使用docker的用户加入到docker用户组。

```
// 建立 docker 组
sudo groupadd docker

// 将当前用户加入docker组
sudo usermod -aG docker $USER
```
### 检测是否安装正确

```
// 启动一个基于hello-world镜像的容器
docker run hello-world
```

## 卸载

### 删除安装包
```
sodu yum remove docker-ce
```
### 删除镜像

```
sudo rm -rf /var/lib/docker
```

## 镜像加速器

国内从 Docker Hub 拉取镜像有时会遇到困难，此时可以配置镜像加速器。

* Docker 官方提供的中国 Registry Mirror
* 阿里云加速器
* DaoCloud加速器
* 163加速器


```
// 创建文件
vi //etc/docket/daemon.json

// 填写内容
{
  "registry-mirrors":[
    "http://hub.mirror.c.163.com"
  ]
}

```
重新启动服务生效

```
// daemon 生效
sudo systemctl daemon-reload

// docker 重启生效
sudo systemctl restart docker

// 查看信息
docker info

// 看到配置完成的信息
Registry Mirrors: http://hub-mirror.c.163.com

```

# 命令
## 镜像操作

Docker 运行容器前需要本地存在对应的镜像，如果本地不存在该镜像，Docker 会从镜像仓库下载该镜像。

## 从仓库获取镜像
从Docker镜像仓库获取镜像的命令是：docker pull

```
// 具体选项帮助
docker pull --help
docker pull [选项] [Docker Registry 地址[:端口号/]]仓库名[:标签]

// 默认从官方 library/ubuntu 仓库中获取标签为16.04的镜像
docker pull ubuntu:16.04
```

镜像仓库地址：<域名/IP>[:端口号]
仓库名称：<用户名称>/<软件名称>

官方仓库地址：Docker Hub
官方镜像：library


```
// -it：终端交互操作
// --rm： 容器退出后随之将其删除
// bash： 交互式的shell
docker run -it --rm  ubuntu:16.04 bash

// 退出容器
exit
```

## 管理本地主机上的镜像

### 列出镜像
```
// 查看镜像列表，包含仓库名、标签、镜像ID、创建时间、占用空间
docker image ls

// 查看镜像、容器、数据卷占用空间
docker system df

// 仓库名、标签均为 <none>的镜像称为虚悬镜像
// 查看虚悬镜像
docker image ls -f dangling=true

// 删除虚悬镜像
docker image prune

```

### 删除本地镜像

```
// 镜像可以是短ID、长ID、名称、摘要
docker image rm [选项] <镜像1> [<镜像2>]

// 删除所有仓库名为ubuntu的镜像
docker image rm $(docker image ls -q ubuntu)

// 删除ubuntu:16.04 以前的镜像
docker image rm $(docker image ls -q -f before=ubuntu:16.04)
```



## 介绍镜像实现的基本原理

## 容器操作
容器是独立运行的一个或一组应用，以及它们的运行态环境。相应的，虚拟机可以理解为模拟运行的一整套操作系统（提供了运行态环境和其它系统环境）和跑在上面的应用。

## 启动容器
启动容器有两种方式，

* 基于镜像新建一个容器并启动
* 将在终止状态的容器重新启动

因为Docker 的容器是轻量级的，用户可以随时删除和创建容器。

```
docker run
```

### 新建并启动

### 启动已终止的容器

```
// 方式一：启动
docker container start

// 方式二：启动
docker start

// 启动一个bash终端，允许用户进行交互
// -t 分配一个伪终端并绑定到容器的标准输入上
// -i 让容器的标准输入保持打开
docker run -t -i ubuntu:16:04 /bin/bash

```
当利用docker run 来创建容器时，Docker 在后台运行的标准操作包括：

* 检查本地是否存在指定的镜像，不存在就从公有仓库下载
* 利用镜像创建并启动一个容器
* 分配一个文件系统，并在只读的镜像层外面挂载一层可读写层
* 从宿主机配置的网桥接口中桥接一个虚拟接口到容器中去
* 从地址池配置一个IP地址给容器
* 执行用户指定的应用程序
* 执行完毕后容器被终止

docker run ubuntu:16.04 /bin/echo 'hello World'


### 后台运行

很多时候，需要让Docker 在后台运行而不是直接把执行命令的结构输出在当前宿主机下。此时，可以通过 -d 参数 来实现。

如果不使用 -d 参数 运行容器，比如 docker run hello-world 会把日志打印在控制台。
如果使用率 -d 参数 

```
// 运行容器，加上参数 -d ，不会输出日志，智慧打印容器ID，输出结果可以去 docker logs查看
docker run -d hello-world

docker run -it ubuntu:16:04 /bin/bash


```

### 停止运行的容器

// 容器终止
docker container stop 容器ID

// 查看终止状态的容器
docker container ls -a

// 容器启动
docker container start 容器ID

// 容器重启
docker container restart 容器ID

### 进入容器

```
docker exec -it 容器ID /bin/bash
```


### 导出和导入容器

```
// 导出本地某个容器
docker export 容器ID > 导出文件名.tar

// 导入容器
cat 导出文件名.tar | docker import - 镜像用户/镜像名:镜像版本

// 也可以通过制定URL 或者某个目录来导入
docker import http://study.163.com/image.tgz example/imagerepo

```

### 删除容器

```
// 删除容器， -f 表示删除一个运行中的容器
docker container rm ubuntu:16:04

// 查看已经
docker container ls -a 

// 删除掉处于终止状态的容器
docker container prune
```