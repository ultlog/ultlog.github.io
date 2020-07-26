---
title: ula
toc: true
date: 2020-07-26 20:04:42
tags: ula
categories: 
---
# ula

## 前置
需要7.x版本的[Elasticsearch](https://www.elastic.co/)

## 安装
### 下载
点击[此处](https://github.com/ultlog/ula/releases)下载ula-0.0.1.jar

### 运行
````shell
java -jar -D"ultlog.es.address.host={es-ip}" -D"ultlog.es.address.port={es-port}"   ula-0.0.1.jar
 ````
替换其中es-ip与es-port
### 个性化

如果需要修改端口ip等配置信息可以在启动项加入
````shell
-Dspring.port=8888
````

更建议创建一个配置文件在同级目录，修改配置更方便。


## 更多
更多关于ulu的信息可以访问[ultlog文档中心](http://ultlog.com)查阅。
