---
title: ultlog
date: 2020-07-19 14:19:01
tags:
---
# 前言
ultlog是一个日志收集，过滤，展示的系统。
基于高性能地[Elasticsearch](https://www.elastic.co/cn/cloud/?elektra=home&storm=sub1)以及[logback](https://github.com/qos-ch/logback)等java框架，可以快速地对线上日志数据进行收集，并且提供ula页面进行日志地展示。
相比于原始地日志系统/框架，ultlog有如下优势：
- 分布式系统日志能够地统一收集
- 避免了登录虚拟机/跳板机查询日志
- 可以通过时间或关键字进行日志检索，省去了编写查询语句的时间
- 可以以服务/模块/系统唯一标识等属性区分日志，方便微服务架构下日志的准确查找


[开始使用](#开始使用)

# 架构
## 日志数据流转
{% asset_img ultlog.png "ultlog日志数据流转图:vi/vim-cheat-sheet" %}

## searcher

searcher 是ultlog系统中收集日志的一个程序。通过对系统中日志文件的监控，实时的将产生的日志发送到ula中。
与[collector](https://github.com/ultlog/collector)不同，searcher不需要在工程中集成，转而在操作系统中集成。
因此一些非基于logback的java项目（非常少）或非java项目也可以通过修改日志的格式而将日志托管给ultlog，享受ultlog带来的便利。
## collector
collector是基于logback开发的日志收集组件，适用于使用logback日志框架的系统。collector通过对logback框架中的appender接口扩展而将收集到的日志发送到[ula](#ula)系统。相比于searcher，
collector能够适应更改多变的日志格式，并且在部署时不需要编写额外的脚本。

## ula
ula为ultlog系统中的日志与网络请求处理服务，收集到的日志通过ula存储进Elasticsearch数据库，并且通过Restful风格的接口对外提供查询功能。

## ulu
ulu为ultlog系统日志展示程序。可以通过系统名称，日志产生时间，日志信息等多个维度进行日志的查询。

# 开始使用

[ula部署说明](/2020/07/26/系统说明/ula/ula/)
[ulu部署说明](/2020/07/26/系统说明/ulu/ulu/)
[collector部署说明](/2020/07/26/系统说明/collector/collector/)
[searcher部署说明](/2020/07/26/系统说明/searcher/searcher/)

# 未来规划
更多新功能正在开发，欢迎通过[未来规划](/2020/07/26/版本变更/feature/)查阅。
