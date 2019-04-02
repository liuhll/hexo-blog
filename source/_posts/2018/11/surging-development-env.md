---
title: 微服务框架Surging之开发环境
date: 2018-11-4 20:45:20
categories: "微服务"
tags:
  - 微服务
  - Surging
  - 开源框架
---

# 搭建开发环境

## 安装开发工具--VS2017
 
使用 visio studio 2017作为开发者工具，在安装vs2017时,请勾选.net core 开发必要的组件。如果已经安装了vs2017，通过Nuget服务仍无法安装 Surging 组件包时、或是无法通过编译时,请更新vs2017到最新版本，并勾选开发.net core必要的组件。

## 配置Nuget服务
推荐搭建公司内部nuget环境，用于管理公司内的开发的组件包，关于Nuget服务的搭建[请参考](http://www.imooc.com/article/20944)。
Surging的架构通过内部的Nuget服务器进行发布和升级，所以，在安装完 vs2017 后，需要新增内部的Nuget服务的配置。
通过下图对nuget进行配置:

![nuget配置](sunwin.nuget.config.png)

## 安装Docker For Windows(推荐)
1. 到[docker官网](https://store.docker.com/editions/community/docker-ce-desktop-windows)下载 **Docker for Windows Installer.exe** 安装包,并安装到系统中。

2. 将docker的registry-mirrors设置为国内地址: `https://registry.docker-cn.com`。
![docker国内镜像](docker-cn-registry-mirrors.png)

3. 将使用到的镜像`microsoft/dotnet:2.1-sdk`、`microsoft/dotnet:2.1-runtime`、`consul:latest`、`rabbitmq:latest`、`redis:latest`等通过命令`docker pull`拉取到本地。


> **Notes** 
> 1. 使用和下载docker都必须注册docker账号,请先到docker-store[注册地址](https://store.docker.com/signup)通过邮箱注册docker账号
> 2. docker在Windows系统中默认使用`Hyper-V`作为虚拟化组件,由于版本冲突,在Windows系统中安装docker后，将无法使用`VmWare`,如果要使用虚拟机可以通过`Hyper-V`或`VirtualBox`创建。
> 3. docker for windows 只支持 Windows 专业版及以上，不支持家庭版,如果是家庭版的操作系统，想使用docker的话，需要先更新