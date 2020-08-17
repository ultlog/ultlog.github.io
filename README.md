<h1 align="center">ultlog docs</h1>
<p align="center">
  <a href="https://travis-ci.com/github/ultlog/ultlog.github.io"><img src="https://travis-ci.com/ultlog/ultlog.github.io.svg?branch=hexo"></a>
  <a target="_blank" href="https://github.com/ultlog/ultlog.github.io/blob/hexo/LICENSE"><img src="https://img.shields.io/badge/license-MIT-blue"></a>
  <a target="_blank" href="https://github.com/ultlog/ultlog.github.io/pulls"><img src=https://img.shields.io/badge/pr-welcome-green"></a>
  <a target="_blank" href="https://github.com/ultlog/ultlog.github.io/pulls?q=is%3Apr+is%3Aclosed"><img src="https://img.shields.io/github/issues-pr-closed/ultlog/ula"></a>
</p>
  
<p align="center">
  <a href="https://ultlog.com" target="_blank">
    文档
  </a>
  / 
  <a href="https://github.com/ultlog/ula/" target="_blank">
    ultlog-api
  </a>
  / 
  <a href="https://github.com/ultlog/ulu/" target="_blank">
    ultlog-ui
  </a>
  / 
  <a href="https://github.com/ultlog/collector" target="_blank">
    collector
  </a>
  /
  <a href="https://github.com/ultlog/searcher" target="_blank">
    searcher
  </a>
</p>

## ultlog
ultlog is a log collection, filtering, and display system。
Based on high performance [Elasticsearch](https://www.elastic.co/cn/cloud/?elektra=home&storm=sub1),[logback](https://github.com/qos-ch/logback) and more procedure and framework,can quickly collect online log data, and provide ula page for log display。
Compared with the original log system/framework, ultlog has the following advantages:
- Log of distributed system can be collected uniformly.
- Avoid logging in to the virtual machine/springboard to query logs.
- Log be query by time or keywords, saving time for writing query statements.
- Log can be distinguished by attributes such as uuid/module/system unique identifiers to facilitate accurate search of logs under the microservice architecture.
- Superior performance, stand-alone ultlog and stand-alone elasticsearch services can easily satisfy a team of dozens of people, with an average daily storage and query of more than 500,000 logs.

....

## Thanks
Thanks to [JetBrains](https://www.jetbrains.com/?from=ultlog.com) for their support during the development of ultlog

## More
[中文文档](http://ultlog.com)

