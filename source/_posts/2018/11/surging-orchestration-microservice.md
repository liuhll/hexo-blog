---
title: 微服务框架Surging之编排微服务
date: 2018-11-05 21:00:03
categories: "微服务"
tags:
  - 微服务
  - Surging
  - 开源框架
---

# 编排微服务
使用docker-compose 编排微服务，可以简化开发环境的搭建，方便开发调试与docker镜像的构建与持续集成。但是在开发过程中，并不是必须使用docker-compose进行微服务的编排。

# 不使用docker-compose编排微服务
如果不使用docker-compose编排微服务，那么要求在开始运行微服务主机之前，运行环境必须提供consul服务和rabbitmq、redis中间件。其中，consul作为服务注册中心；rabbitmq作为事件总线的消息中间件;redis作为缓存中间件。当然，作为注册中心的consul也可以使用zookeeper替换，rabbitmq也可以使用kafka替换。

如果在开发过程中，团队需要对微服务进行联调，那么consul和rabbitmq必须部署在一台可访问的内部服务器。在开发和测试环境中可以搭建单个节点的服务提供开发和测试，但是为提高consul、rabbitmq的可用性，在生成环境中要求consul、rabbitmq必须搭建集群。consul的使用和部署请参考[consul使用手册](http://www.liangxiansen.cn/2017/04/06/consul/),rabbitmq的集群部署请参考[搭建 RabbitMQ Server 高可用集群](https://www.cnblogs.com/xishuai/p/centos-rabbitmq-cluster-and-haproxy.html)。如果是单人进行开发或测试，可以consul、rabbitmq安装到本地环境，或使用docker-compose进行编排。

# 使用docker-compose编排微服务
如果开发者在进行某个模块的业务开发，而在该模块下存在多个微服务组件，在该场景下，建议使用docker-compose编排微服务。使用docker-compose编排微服务的好处在于将开发环境使用到服务或中间件都可以编排到微服务集群中，以减少对开发环境的部署和调试的时间。例如：我们完全可以将使用到consul服务和rabbitmq、redis中间件等通过docker-compose编排到微服务组件的集群中。但是如果调试不同模块之间的微服务组件，则无法采用该种模式，因为微服务集群必须要有统一的服务注册中心。

关于docker-compose的编写请参考[官方文档](https://docs.docker.com/compose/compose-file/)。

一般地，使用docker-compose编排微服务我们通过如下步骤来实现：

1. 建立docker-compose.dcproj 项目文件,并在 `</ItemGroup>`节点中指定要引用的docker-compose 编排文件
```xml

<?xml version="1.0" encoding="utf-8"?>
<Project ToolsVersion="15.0" Sdk="Microsoft.Docker.Sdk">
  <PropertyGroup Label="Globals">
    <ProjectGuid>fea0c318-ffed-4d39-8781-265718ca43dd</ProjectGuid>
    <DockerTargetOS>Linux</DockerTargetOS>
    <ProjectVersion>2.1</ProjectVersion>
    <DockerLaunchAction>LaunchBrowser</DockerLaunchAction>
  </PropertyGroup>
  <ItemGroup>
    <None Include=".dockerignore" />
    <None Include="docker-compose.override.yml">
      <DependentUpon>docker-compose.yml</DependentUpon>
    </None>
    <None Include="docker-compose.yml" />
  </ItemGroup>
</Project>
```

2. 编写`docker-compose.yml`编排文件,一般地，该文件仅定义集群中各个微服务组件所使用的镜像，以及微服务组件构建的上下文,所使用到的环境变量一般在`docker-compose.override.yml`中定义。

```yml
version: '3.4'

services:  
  consul:
    image: consul:latest
    networks:
      - surging_demo
  
  redis:
    image: bitnami/redis:latest
    networks:
      - surging_demo
  
  rabbitmq:
    image: rabbitmq:management
    networks:
      - surging_demo

  dgitalreleaseeval:
    image: surgingdemo/dgitalreleaseeval:${TAG:-latest}
    build:
      context: .
      dockerfile: Sample/Services/DigitalReleaseEval/Surging.Demo.DigitalReleaseEvalHost/Dockerfile
    networks:
      - surging_demo
    depends_on:
      - consul
      - redis
      - rabbitmq
      - airportguaranteeeval
      - meteorologicaleval
      - operationrestrictioneval

  surgingserver_ws:
    image: surgingdemo/surgingserver_ws:${TAG:-latest}
    build:
      context: .
      dockerfile: Sample/Services/WebSocketServer/Surging.Demo.WebSocketHost/Dockerfile
    networks:
      - surging_demo
    depends_on:
      - consul
      - redis
      - rabbitmq

  airportguaranteeeval:
    image: surgingdemo/airportguaranteeeval:${TAG:-latest}
    build:
      context: .
      dockerfile: Sample/Services/AirportGuaranteeEval/Surging.Demo.AirportGuaranteeEvalHost/Dockerfile
    networks:
      - surging_demo
    depends_on:
      - consul
      - redis
      - rabbitmq
      - meteorologicaleval
   
  meteorologicaleval:
    image: surgingdemo/meteorologicaleval:${TAG:-latest}
    build:
      context: .
      dockerfile: Sample/Services/MeteorologicalEval/Surging.Demo.MeteorologicalEvalHost/Dockerfile
    networks:
      - surging_demo
    depends_on:
      - consul
      - redis
      - rabbitmq

  operationrestrictioneval:
    image: surgingdemo/operationrestrictioneval:${TAG:-latest}
    build:
      context: .
      dockerfile: Sample/Services/OperationRestrictionEval/Surging.Demo.OperationRestrictionEvalHost/Dockerfile
    networks:
      - surging_demo
    depends_on:
      - consul
      - redis
      - rabbitmq
networks:
  surging_demo: 
    driver: bridge
    ipam:
      driver: default
      config:
      - subnet:  172.18.0.1/16
```

3. 定义docker-compose.override.yml，对集群中所使用到的各个组件所使用到的环境变量、卷、端口映射等进行定义，其中，默认的环境变量可以在.evn中的设置覆盖。

```yml
version: '3.4'

services:
  consul:
    ports:
      - "18400:8400"
      - "18500:8500"
      - "18600:8600"
      - "18600:8600/udp"
    command: "agent -server -bootstrap-expect 1 -ui -client 0.0.0.0"
  redis:
    ports:
      - "16379:6379"
  rabbitmq:
    environment:
      RABBITMQ_ERLANG_COOKIE: "SWQOKODSQALRPCLNMEQG"
      RABBITMQ_DEFAULT_USER: "rabbitmq"
      RABBITMQ_DEFAULT_PASS: "rabbitmq"
      RABBITMQ_DEFAULT_VHOST: "/"
    ports:
      - "15673:15672"
      - "5673:5672"
  dgitalreleaseeval:
    environment:
      Surging_Server_Address: ${DGITALRELASE_Surging_SERVER_ADDRESS:-dgitalreleaseeval}
      Register_Conn: ${REGISTER_CONN:-consul:8500}
      Register_SessionTimeout: ${REGISTER_SESSION_TIMEOUT:-50}
      UseEngineParts: "DotNettyModule;NLogModule;ConsulModule;EventBusRabbitMQModule;CachingModule;KestrelHttpModule"
      EventBusConnection: ${RABBITMQ_CONNECTION:-rabbitmq}
      EventBusUserName:  ${RABBITMQ_USERNAME:-rabbitmq}
      EventBusPassword:  ${RABBITMQ_PASSWORD:-rabbitmq}
      EventBusPort: ${RABBITMQ_PORT:-5672}
#    volumes:
#      - ${DIGITALRELEASE_SERVER_LOG_DIR:-./logs/d}:/app/logs
    ports:
      - "101:100"
      - "8081:8080"
  surgingserver_ws:
    environment:
      Surging_Server_Address: ${DGITALRELASEWS_Surging_SERVER_ADDRESS:-surgingserver_ws}
      Register_Conn: ${REGISTER_CONN:-consul:8500}
      Register_SessionTimeout: ${REGISTER_SESSION_TIMEOUT:-50}
      UseEngineParts: "DotNettyModule;NLogModule;ConsulModule;EventBusRabbitMQModule;CachingModule;WSProtocolModule"
      EventBusConnection: ${RABBITMQ_CONNECTION:-rabbitmq}
      EventBusUserName:  ${RABBITMQ_USERNAME:-rabbitmq}
      EventBusPassword:  ${RABBITMQ_PASSWORD:-rabbitmq}
      EventBusPort: ${RABBITMQ_PORT:-5672}
#    volumes:
#      - ${DIGITALRELEASE_WSSERVER_LOG_DIR:-./logs/d_ws}:/app/logs
    ports:
      - "102:100"
      - "96:96"
         
  airportguaranteeeval:
    environment:
      Surging_Server_Address: ${AIRPORTGUARANTE_Surging_SERVER_ADDRESS:-airportguaranteeeval}
      Register_Conn: ${REGISTER_CONN:-consul:8500}
      Register_SessionTimeout: ${REGISTER_SESSION_TIMEOUT:-50}
      EventBusConnection: ${RABBITMQ_CONNECTION:-rabbitmq}
      EventBusUserName:  ${RABBITMQ_USERNAME:-rabbitmq}
      EventBusPassword:  ${RABBITMQ_PASSWORD:-rabbitmq}
      EventBusPort: ${RABBITMQ_PORT:-5672}
#    volumes:
#      - ${AIRPORTGUARANTEE_SERVER_LOG_DIR:-./logs/a}:/app/logs
    ports:
      - "103:100"
      - "8082:8080"

  meteorologicaleval:
    environment:
      Surging_Server_Address: ${METEOROGICAL_Surging_SERVER_ADDRESS:-meteorologicaleval}
      Register_Conn: ${REGISTER_CONN:-consul:8500}
      Register_SessionTimeout: ${REGISTER_SESSION_TIMEOUT:-50}
      EventBusConnection: ${RABBITMQ_CONNECTION:-rabbitmq}
      EventBusUserName:  ${RABBITMQ_USERNAME:-rabbitmq}
      EventBusPassword:  ${RABBITMQ_PASSWORD:-rabbitmq}
      EventBusPort: ${RABBITMQ_PORT:-5672}
#    volumes:
#      - ${METEOROLOGICCAL_SERVER_LOG_DIR:-./logs/m}:/app/logs
    ports:
      - "104:100"
      - "8083:8080"

  operationrestrictioneval:
    environment:
      Surging_Server_Address: ${OPERTIONRESTRICTION_Surging_SERVER_ADDRESS:-operationrestrictioneval}
      Register_Conn: ${REGISTER_CONN:-consul:8500}
      Register_SessionTimeout: ${REGISTER_SESSION_TIMEOUT:-50}
      EventBusConnection: ${RABBITMQ_CONNECTION:-rabbitmq}
      EventBusUserName:  ${RABBITMQ_USERNAME:-rabbitmq}
      EventBusPassword:  ${RABBITMQ_PASSWORD:-rabbitmq}
      EventBusPort: ${RABBITMQ_PORT:-5672}
#    volumes:
#      - ${OPERATIONRESTRICTION_SERVER_LOG_DIR:-./logs/o}:/app/logs
    ports:
      - "105:100"
      - "8084:8080"

```