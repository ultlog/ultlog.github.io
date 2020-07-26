---
title: ulu
toc: true
date: 2020-07-26 20:06:42
tags: ulu
categories: 
---
## 前言

ulu全称未ultlog-ui，是ultlog系统中展示日志的web程序。

## 安装

#### 下载
点击[此处](https://github.com/ultlog/ulu/releases)下载。

``如果部署在windows系统，可以选择下载ulu-nginx，其自带一个免安装版本的nginx。``

#### 安装
将下载的文件解压到nginx的html路径下，目录结构为
``{base_path}/nginx/html/index.html`` 和 ``{base_path}/nginx/html/static``

修改nginx配置文件中的反向代理配置，添加如下代码
````
location ^~ /api {
    proxy_pass http://{ula-host}:{ula-ip};
}
````
将其中ula-ip/port替换为ula服务部署的地址和端口

#### 启动nginx
````shell
start nginx
````

## 更多
更多关于ulu的信息可以访问[ultlog文档中心](http://ultlog.com)查阅。
