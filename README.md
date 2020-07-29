![](https://travis-ci.com/ultlog/ultlog.github.io.svg?branch=hexo)
## ultlog
ultlog是一个日志收集，过滤，展示的系统。
基于高性能地[Elasticsearch](https://www.elastic.co/cn/cloud/?elektra=home&storm=sub1)以及[logback](https://github.com/qos-ch/logback)等java框架，可以快速地对线上日志数据进行收集，并且提供ula页面进行日志地展示。
相比于原始地日志系统/框架，ultlog有如下优势：
- 分布式系统日志能够地统一收集
- 避免了登录虚拟机/跳板机查询日志
- 可以通过时间或关键字进行日志检索，省去了编写查询语句的时间
- 可以以服务/模块/系统唯一标识等属性区分日志，方便微服务架构下日志的准确查找
- 性能优越，单机ultlog以及单机elasticsearch服务可以轻松满足数十人团队，日均多余五十万条日志的存储与查询。

....

[了解更多](ultlog.com)

