---
title: "Eureka to Nacos"
date: 2021-11-02T23:45:28+08:00
# aliases: ["/first"]
author: "drizzl4"
# author: ["Me", "You"] # multiple authors
TocOpen: false
draft: false
hidemeta: false
canonicalURL: "https://canonical.url/to/page"
disableHLJS: true # to disable highlightjs
disableShare: false
disableHLJS: false
hideSummary: false
ShowReadingTime: true
ShowBreadCrumbs: false
ShowPostNavLinks: true
cover:
    image: "<image path/url>" # image path/url
    alt: "<alt text>" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: true # only hide on current single page
editPost:
    URL: "https://github.com/drizzl4/drizzl4.github.io/blob/master/content"
    Text: "Edit" # edit text
    appendFilePath: true # to append file path to Edit link

# weight: 1
tags: ["Nacos"]
categories: ["Nacos"]
showToc: true
comments: true
description: ""
searchHidden: false
summary: "Eureka to Nacos"
---

# Eureka to Nacos

记一次eureka迁移到nacos，使用nacos做配置中心和注册中心  

各依赖版本：

* Spring Boot 2.1.7
* Spring Cloud Greenwich.RELEASE
* Nacos 2.1.2

## 1.Nacos服务端搭建

此时按照官方文档，正常搭建即可  

**注意：**

* 根据Nacos提供的sql脚本创建表
* 集群配置
* 开启权限认证，开启后需要在bootstrap中配置账号密码

## 2.将Eureka相关依赖配置删除，增加Nacos相关配置

删除部分设计项目本身，不记录  

nacos配置：

```yaml
 logging:
  level:
    # Nacos 注册中心客户端心跳日志禁用 get changedGroupKeys:[] 刷屏
    com.alibaba.nacos.client.config.impl: WARN
    
 cloud:
    #Nacos 的配置
    nacos:
      # 配置中心的配置
      config:
        # 配置中心的服务地址
        server-addr: 127.0.0.1:8848
        # 配置项的命名空间，如果不指定将会在public命名空间下查找配置项
        namespace: 2109f1a6-7fd9-4592-98a4-e0e20323e2cc
        # 配置所属的分组，如果组名为：DEFAULT_GROUP，则可以省略此项的配置（namespace如果是按环境分[dev,prod]，那group则可以是按项目名来定）
        group: bcbase
        # 文件格式
        file-extension: yaml
        # 共享配置
      #        ext-config:
      #          - data-id: shareconfig.yml
      #            group: DEFAULT_GROUP  # 可以不写，默认值为DEFAULT_GROUP
      #            refresh: true # 默认是false,如果需要支持自动刷新需要配置true,搭配@RefreshScope实现动态刷新
      # 注册中心的配置
      discovery:
        # 注册中心的服务地址，因为与配置中心的相同所以使用变量引用配置中心的设置
        server-addr: ${spring.cloud.nacos.config.server-addr}
        # 指定服务要注册到哪个命名空间中去，不设置本项会注册到public中，多环境或版本时要注意这一项的设置
        # 为了让配置，服务注册两者的命名空间相同这里用了一个变量，引用了配置的命名空间
        namespace: ${spring.cloud.nacos.config.namespace}
        # 指定服务要注册到哪个组中去，不设置本项会注册到默认的DEFAULT_GROUP，多项目时可以按项目名来定
        # 为了让配置，服务注册两者的分组相同，这里用了一个变量，引用了配置的分组
        group: ${spring.cloud.nacos.config.group}
```

main方法修改

```java
@EnableDiscoveryClient
@ComponentScan(basePackages = {"com.snow"}, excludeFilters = {
        @ComponentScan.Filter(type =
                FilterType.ASSIGNABLE_TYPE, classes = {
                NacosRefreshProperties.class})})
```

扫描包的路径需要在二级包之下，因为公司项目使用的原生Spring Cloud，所以对nacos对其支持不友好，需要对**NacosRefreshProperties** 类进行过滤，而且此类已经弃用  

在Spring Cloud Alibaba相关的项目中，应该是不需要过滤的

## 3.配置文件设计

![image-20211102235655839](/images/image-20211102235655839.png)

在bootstrap.yml中设置，公用并且优先级高的配置

```yml
server:
  port: 8111
  servlet:
    context-path: /

logging:
  level:
    # Nacos 注册中心客户端心跳日志禁用 get changedGroupKeys:[] 刷屏
    com.alibaba.nacos.client.config.impl: WARN
  path: xxx

spring:
  profiles:
    active: test
  mvc:
    async:
      request-timeout: 20000
  application:
    name: xxxxx
  servlet:
    multipart:
      enabled: true
      max-file-size: 100MB
      max-request-size: 500MB
```

在不同环境的bootstrap 中，配置对应的Nacos配置

```yml
spring:
  cloud:
    nacos:
      config:
        #内网
        server-addr: xxx
        #dev
        namespace: xxx
        # 文件格式
        file-extension: yaml
        # 共享配置
#        extension-configs:
#          - data-id: bcuser.yaml
#            refresh: true # 默认是false,如果需要支持自动刷新需要配置true,搭配@RefreshScope实现动态刷新
      discovery:
        server-addr: ${spring.cloud.nacos.config.server-addr}
        namespace: ${spring.cloud.nacos.config.namespace}
      # 在nacos服务端配置中开启权限认证，则需要设置账号密码
      username: xxx
      password: xxx
```

## 4.Nacos分组设置

Nacos中，命名空间各项目是相互隔离的，各个分组之间也是相互隔离的  

本次改造中，我们并不使用分组功能，将使用命名空间进行隔离，如果本地启动项目，也可以使用开发环境，但是需要在本地统一修改分组  



各环境的分配，看到官方其实推荐不同分组为不同项目，但是未找到请求的方法  

