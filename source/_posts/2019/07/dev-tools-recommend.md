---
title: 开发者工具推荐
date: 2019-07-25 13:43:38
tags:
- dev-tools
- 微服务
---

## 版本管理工具
### git
- 开源的分布式版本控制系统，可以有效、高速地处理从很小到非常大的项目版本管理，创建分支和合并分支异常简单、方便
- https://git-scm.com/
- https://www.liaoxuefeng.com/wiki/896043488029600

> 非常不建议使用svn作为版本管理工具,无论是版本管理、分支管理、代码检视都无法做到，团队中如果有人代码不规范，后期代码就会变得越来越差，产品质量根本得不到保证

### gitlab
- GitLab 是一个用于仓库管理系统的开源项目，使用Git作为代码管理工具，并在此基础上搭建起来的web服务
- https://about.gitlab.com/pricing/#gitlab-com

## 持续集成工具
### jenkins
- 基于java开发的一个持续集成工具,提供超过1000个插件来支持构建、部署、自动化， 满足任何项目的需要。
- https://jenkins.io/zh/

### gitlab-runner
- gitlab提供的持续集成工具，能够与gitlab整合在一起，方便的实现持续集成与构建
- https://docs.gitlab.com/ee/ci/quick_start/README.html

## 文档管理(API文档、在线文档)
### docfx
- 微软官方提供的在线文档工具
- https://dotnet.github.io/docfx/

### hexo
- Hexo 是一个快速、简洁且高效的博客框架。Hexo 使用 Markdown（或其他渲染引擎）解析文章，在几秒内，即可利用靓丽的主题生成静态网页。

> 如果需要团建的话可以考虑通过该工具维护内部的学习文档

### swagger
- Swagger 可以生成一个具有互动性的API控制台,开发者可以用来快速学习和尝试API
- https://swagger.io/

### easy-mock
- Easy Mock 是一个可视化，并且能快速生成 模拟数据 的持久化服务
- https://www.easy-mock.com/

### gitbook
- https://www.gitbook.com/

## 容器与容器管理
### docker
- 一个开源的服务引擎管理工具,可以说改变软件行业开发与交付的模式(Build Once,Run AnyWhere)不是吹的。
- https://www.docker.com/

### k8s
- Google开源的容器集群管理系统
- https://www.kubernetes.org.cn/k8s
- https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/

### docker-swarm
- docker 官方提供的容器集群管理平台
- https://docs.docker.com/swarm/

### rancher
- Rancher是一个开源的企业级容器管理平台，1.6.x支持docker-swarm,2.0+支持k8s
- https://rancher.com/docs/rancher/v1.6/zh/
- https://www.cnrancher.com/docs/rancher/v2.x/cn/overview/

> 什么时候使用k8s什么时候使用docker-swarm?
> 我个人推荐优先使用k8s作为容器管理平台。但是目标客户不愿意投入的情况下，并且服务节点相对较少(比如只有不到几十个)的情况下，推荐使用docker-swarm作为容器管理平台,大型的集群服务管理推荐使用k8s


## 仓库管理
### nuget.server
- 提供nuget包管理工具
- https://github.com/NuGet/NuGet.Server

### helm
- k8s软件包(源)管理工具
- https://helm.sh/

### harbor
- docker私有镜像仓库
- https://goharbor.io/

### nexus
- 除了支持上面所述的各种软件源(包)管理之外,还支持yum、python、npm等包管理(推荐)
- https://www.sonatype.com/nexus-repository-sonatype

## 微服务框架
### surging
- dotnetcore社区的一个微服务框架,主要维护者范亮
- https://github.com/dotnetcore/surging
- https://github.com/liuhll/Surging.Sample

## spring cloud
- https://spring.io/projects/spring-cloud
### eShopOnContainers
- 微软官方案列
- https://github.com/dotnet-architecture/eShopOnContainers
- https://docs.microsoft.com/zh-cn/dotnet/standard/microservices-architecture/

### microdot
- .net framework平台下的一个微服务框架,可以与Orleans(奥尔良)结合使用
- https://github.com/gigya/microdot

### enode
- cqrs + ddd实现的soa框架
- https://github.com/tangxuehua/enode

## 推荐使用的工具
- 版本管理工具(git+gitlab) 
- 持续集成工具(gitlab-runnber|Jenkins)
- 文档工具(在线文档docfx+api文档swagger+模拟数据easymock)
- 仓库管理工具(nexus)
- 微服务框架(推荐surging|spring-cloud)
- 服务器(centos|ubuntu)
- 容器和容器管理工具(docker|rancher2.0+|k8s)

