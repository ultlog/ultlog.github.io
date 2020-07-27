---
title: searcher
toc: true
date: 2020-07-26 20:02:42
tags: searcher
categories: 
---

## 前言
searcher 是ultlog系统中收集日志的一个程序。通过对系统中日志文件的监控，实时的将产生的日志发送到ula中。
与[collector](https://github.com/ultlog/collector)不同，searcher不需要在工程中集成，转而在操作系统中集成。
因此一些非基于logback的java项目（非常少）或非java项目也可以通过修改日志的格式而将日志托管给ultlog，享受ultlog带来的便利。

## 前置

searcher使用java编写，因此在应用searcher的操作系统上需要[jdk8](https://www.oracle.com/java/technologies/javase/javase-jdk8-downloads.html)以上的环境，若系统中没有以上环境则可以使用自带jdk的searcher包进行部署。


建议将执行脚本下载到内网，通过minio，ftp或http服务挂载，并以此编写自动集成脚本，具体可见[自动集成实践](#自动集成实践)。

如果有大量环境需要集成建议查看[自动集成实践](#自动集成实践)来快速编写自动集成脚本。

## 安装
### 程序配置
将要应用searcher的程序的logback日志格式应如下：
````
%d{yyyy-MM-dd HH:mm:ss} [%-5level] [%-5thread] %logger{20} - %msg%n</pattern>
````
如果非logback项目或非java项目应将日志格式调整为如此：
````
2020-07-21 09:16:37 [ERROR] [pool-420-thread-7] c.t.g.h.HttpConnectionPoolUtil - test error
java.lang.RuntimeException: test error
at com.example.demo.DemoApplicationTests.contextLoads(DemoApplicationTests.java:21)
....
````

### 选择软件包
|  环境|   软件包 |
| ------ | ------ | 
| <  jdk 1.8 |[searcher_jdk.tar.gz]() | 
| \>=  jdk 1.8 | [searcher.tar.gz]() | 
| 无jdk | [searcher_jdk.tar.gz]() | 

### 手动/单机集成


#### docker linux
- 使用exec等命令进入docker镜像
- 将下载好的软件包传入（可以在镜像内使用curl等命令进行下载）
- 解压文件
- 执行 sh searcher.sh [{args}](#配置项说明)

#### linux
- 将下载好的软件包传入 可以在镜像内使用curl等命令进行下载
- 解压文件
- 执行 sh searcher.sh [{args}](#配置项说明)

#### windows
修改{searcher path}/application.yml，正确填写其中配置项
执行语句启动
````shell
java -jar searcher.jar
````
如果是非jdk8环境
````shell
{searcher_path}/java -jar searcher.jar
````

### 自动集成实践

自动集成提供docker环境文件的改造，如需在jenkins或其他平台集成可以参照此进行改造。

#### 原DOCKERFILE
````shell
FROM java:8
ADD demo.jar /demo.jar
ENTRYPOINT ["java","jar","/demo.jar"]

````
#### 改造后
在程序同级**.jar同级目录添加名称为start.sh的脚本（可以通过cp等命令移动到jar所在目录，下同），内容如下：
````shell
sh startSearcher.sh -f /opt/application.yml
java -jar /demo.jar

````
在原jar目录添加文件
- startSearcher.sh
- application.yml

修改DOCKERFILE

````shell
FROM java:8
ADD demo.jar /demo.jar
ADD startSearcher.sh /startSearcher.sh
ADD application.yml /opt/application.yml
ENTRYPOINT ["sh","start.sh"]

````

### 配置项说明
#### 使用配置文件
使用配置文件即为将软件包内部的配置文件进行修改，然后存放到需要集成searcher的系统某个地址下，通过参数指定路径即可

|  参数|   说明 |是否必填| 默认值| 实例
| ------ | ------ | ------ | ------ | ------ | 
| -f , --file | 配置文件的路径 | √ | | /searcher/application.yml


#### 直接传参

不使用配置文件直接通过以下参数启动searcher服务

|  参数|   说明 |是否必填| 默认值| 实例 |
| ------ | ------ | ------ | ------ | ------ | 
| -p , --path | 产生日志文件的路径 | √ | | /logs/|
| --pattern | 日志文件的关键字 | √ | |error.log|
| --project  | 项目名称 |√ |  | ultlog|
| -m , --module | 模块名称 |√ | | searcher|
| --uuid | 服务唯一属性 | √ |  | 4RFS23Q
| --max | 同时监控的最多文件数 |  × | 10 | 12|
| --level | 收集日志的最低等级 | × | INFO | WARN| 
| -u , --ula | ula服务地址 |√ | | http://192.168.2.2:8080/ | 

#### 配置实例
以下是测试时使用的配置文件，仅供参考
````yaml
ultlog:
  searcher:
    path: F:\Java\Workspace\ultlog\collector\logs
    pattern: demo
    max: 10
    project: searcherTest
    module: test
    uuid: test
    log:
      level: INFO
  ula:
    url:  http://localhost:8080/
````

