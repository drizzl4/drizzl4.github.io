---
title: "Mybatis connects Mysql copy"
date: 2021-08-31T00:28:54+08:00
# weight: 1
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

tags: ["Mybatis","Mysql"]
categories: ["Mybatis","Mysql"]
showToc: true
comments: true
description: ""
searchHidden: false
summary: "Mybatis connects Mysql"
---

# Mybatis之连接Mysql

## 环境介绍

JDK: 1.8

Mysql: 8.0.22

Mybatis: 3.5.7

## 过程

1. 下载Mybatis源码

2. 新建数据库

3. 在resources里面增加db.properties，对Mysql进行配置

4. 增加mybatis-cofig.xml文件

   ```xml
   <?xml version="1.0" encoding="UTF-8" ?>
   <!DOCTYPE configuration
     PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
     "http://mybatis.org/dtd/mybatis-3-config.dtd">
   <configuration>
     <!--指定配置文件-->
     <properties resource="resources/db.properties"/>
     <settings>
       <!--全局开启或关闭缓存，默认为true-->
       <setting name="cacheEnabled" value="false"/>
       <!--延迟加载的全局开关。可通过设置fetchType属性来覆盖该项的配置。默认值为false -->
       <setting name="lazyLoadingEnabled" value="false"/>
     </settings>
     <!--指定扫描的包-->
     <typeAliases>
       <package name="org.apache.ibatis.mysqltest.pojo"/>
     </typeAliases>
     <!--环境配置-->
     <environments default="development">
       <environment id="development">
         <transactionManager type="JDBC"/>
         <dataSource type="POOLED">
           <property name="driver" value="${driver-class-name}"/>
           <property name="url" value="${url}"/>
           <property name="username" value="${username}"/>
           <property name="password" value="${password}"/>
         </dataSource>
       </environment>
     </environments>
     <!--指定Mapper.xml所在位置-->
     <mappers>
       <mapper resource="resources/xml/BaseMapper.xml"/>
     </mappers>
   </configuration>
   ```
5. 创建对应的实体类、Mapper文件
6. 测试类
   ```java
        @Test
        public void mysqlTest() {
          //配置文件
          String resource = "resources/mybatis-config.xml";
          InputStream inputStream = null;
          try {
            inputStream = Resources.getResourceAsStream(resource);
          } catch (IOException e) {
            e.printStackTrace();
          }
          //创建SqlSessionFactory
          SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
          //创建SqlSession
          try (SqlSession sqlSession = sqlSessionFactory.openSession()) {
            //获取Mapper接口的代理对象
            BaseMapper mapper = sqlSession.getMapper(BaseMapper.class);
            List<User> u =  mapper.getUsers();
            //执行查询
            System.out.println(u);
          }
        }
   ```
## 对应文件结构
![image-20210831214253195](/static/images/image-20210831214253195.png)
