---
title: "Mybatis Return Empty"
date: 2021-09-06T11:18:35+08:00
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
tags: ["Mybatis"]
categories: ["Mybatis"]
showToc: false
comments: true
description: ""
searchHidden: false
summary: "Mybatis 不同类型返回值记录"
---

# Mybatis 不同类型返回值记录

## resultType为string

如果select的结果为空，则dao接口返回结果为null

## resultType为基本类型，如int、double等

后台报异常：
org.apache.ibatis.binding.BindingException: Mapper method 'com.fkit.dao.xxDao.getUserById attempted to return null from a method with a primitive return type (int).
解释：查询结果为null，试图返回null但是方法定义的返回值是int，null转为int时报错
解决办法：

 1. 利用mysql的函数ifnull

    ```xml
    <select id="getweekalert" resultType="int">
    select IFNULL(SUM(alert_sum),0) as alert_sum from tb_checkresults
    </select>
    ```
 2. 将返回类型改为Integer等包装类型
    ```xml
    <select id="getweeekalert" resultType="Integer">
    select SUM(alert_sum) as alert_sum from tb_checkresults
    </select>
    ```

## resultType为类为map ，如map、hashmap

　　dao层接口返回值为null

## resultType 为list ，如list

　　dao层接口返回值为[]，即空集合。

注意：此时判断查询是否为空就不能用null做判断
## resultType 为类 ，如com.fkit.pojo.User

　　dao层接口返回值null

参考链接：

	1. [参考链接1](https://www.cnblogs.com/xxjcai/p/11664315.html)
 	2. [参考链接2](http://www.cppcns.com/ruanjian/java/366275.html)

