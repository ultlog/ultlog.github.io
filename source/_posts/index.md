---
title: 关于ultlog
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
[开始使用](#开始使用)

# 架构
## 日志数据流转
{% asset_img ultlog.png "ultlog日志数据流转图:vi/vim-cheat-sheet" %}

## searcher
searcher为ultlog系统日志收集程序。通过对系统中指定文件夹进行监控，实时获取目标文件中的日志数据，并且将日志发送到[ula](#ula)系统。
## collector
collector是基于logback开发的日志收集组件，适用于使用logback日志框架的系统。collector通过对logback框架中的appender接口扩展而将收集到的日志发送到[ula](#ula)系统。相比于searcher，
collector能够适应更改多变的日志格式，并且在部署时不需要编写额外的脚本。

## ula
ula为ultlog系统中的日志与网络请求处理服务，收集到的日志通过ula存储进Elasticsearch数据库，并且通过Restful风格的接口对外提供查询功能。

## ulu
ulu为ultlog系统日志展示程序。可以通过系统名称，日志产生时间，日志信息等多个维度进行日志的查询。

# 开始使用

(开始使用)[]
# 未来规划
