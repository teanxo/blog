---
title: mybatis  Could not create connection to database server. Attempted reconnect 3 times. Giving up
date: 2023-02-04 22:55:59
categories:
    - 错误排查
tags:
---


### mybatis  Could not create connection to database server. Attempted reconnect 3 times. Giving up

原文翻译
```
无法创建到数据库服务器的连接。尝试重新连接3次。放弃
```

今天在创建新项目时发生的错误，经过一系列的排查，发现是设置datasource.url原因所致

```yml
    # 此配置可运行
    url: jdbc:mysql://localhost:3306/master?useUnicode=true&characterEncoding=utf-8&useSSL=true&serverTimezone=UTC
    # 此配置会导致错误
    url: jdbc:mysql://localhost:3006/madmin?useUnicode=true&useSSL=true&characterEncoding=utf8&autoReconnect=true&serverTimezone=UTC
```

显然问题出在了`autoReconnect`上面，平时都是赋值粘贴这个配置，还没有好好的看过，然后就去搜了下这个参数

| 参数 | 说明 |
|--- |--- |
|autoReconnect|驱动程序是否应该尝试重新建立连接? 如果启用了，驱动程序将会对陈旧或死亡连接上发出的查询抛出异常，这些查询属于当前事务，但是在新事务中对连接发出的下一个查询之前会尝试重新连接。 不推荐这个功能的使用,因为它有副作用与会话状态和数据一致性当应用程序不妥善处理异常,仅设计用于当你无法配置您的应用程序来处理异常造成死亡和陈旧的正确连接。 另外，作为最后一个选项，研究将MySQL服务器变量“wait_timeout”设置为一个较高的值，而不是默认的8小时。|
|maxReconnects|如果autoReconnect为true，则尝试重新连接的最大次数，默认为’3’。|
|initialTimeout|如果启用了autoReconnect，重新连接尝试之间的初始等待时间(以秒为单位，默认为’2’)。|

