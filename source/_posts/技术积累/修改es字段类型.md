---
title: 如何修改elasticsearch字段类型
toc: true
date: 2020-08-16 20:59:42
tags: elasticsearch
categories: 
---
## 背景
在开发v1.1.0中的统计功能时发现ult_index索引中的字段不满足聚合查询的条件，当时是使用的开发时创建的elasticsearch索引，所以部分字段的类型存在问题。这个问题在v1.0.0版本中不会存在，通过[ula](/2020/07/26/系统说明/ula/ula/)部署说明可以创建正确的索引。
此时的索引结构通过以下语句查询
````shell

curl http://localhost:9200/ult_index?pretty
````
得到的结果为
<details>
<summary>ult_index.json</summary>
<pre><code>

````json
{
  "ult_index" : {
    "aliases" : { },
    "mappings" : {
      "properties" : {
        "acceptTime" : {
          "type" : "long"
        },
        "createTime" : {
          "type" : "long"
        },
        "level" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "message" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "text"
            }
          }
        },
        "module" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "text"
            }
          }
        },
        "project" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "stack" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "text"
            }
          }
        },
        "uuid" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        }
      }
    },
    "settings" : {
      "index" : {
        "creation_date" : "1597066526478",
        "number_of_shards" : "1",
        "number_of_replicas" : "1",
        "uuid" : "pSLksVk0SfavnrOV1OmeDQ",
        "version" : {
          "created" : "7060299"
        },
        "provided_name" : "ult_index"
      }
    }
  }
}
````
</code></pre>
</details>
其中的module字段应为keyword类型

# 思路
翻了下elasticsearch的[文档](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-put-mapping.html#updating-field-mappings)，其中一段写着
> ...you can’t change the mapping or field type of an existing field. Changing an existing field could invalidate data that’s already indexed.


虽然没法动态地进行修改，但是发现了[重建索引](https://www.elastic.co/guide/en/elasticsearch/reference/7.0/docs-reindex.html)在7.x版本地使用，于是思路就清晰了。先创建一个新的类型正确地索引，将旧索引地数据使用reindex移动过去，再删掉原索引，使用正确地脚本创建一个同名地索引，最后将数据移动回去。

# 执行
````shell
curl -XPUT  -H 'Content-Type: application/json' localhost:9200/ult_index_temp -d@data.json

````
<details>
<summary>json.data</summary>
<pre><code>

````json
{
    "mappings" : {
      "properties" : {
        "acceptTime" : {
          "type" : "long"
        },
        "createTime" : {
          "type" : "long"
        },
        "level" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "message" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "text"
            }
          }
        },
        "module" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "project" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "stack" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "text"
            }
          }
        },
        "uuid" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        }
      }
    },
    "settings":{}
}
````
</code></pre>
</details>

## 数据迁移
````
curl -X POST "localhost:9200/_reindex" -H 'Content-Type: application/json' -d'
{
  "source": {
    "index": "ult_index"
  },
  "dest": {
    "index": "ult_index_temp"
  }
}'
````

## 删除旧索引
````shell

curl -X DELETE "localhost:9200/ult_index"
````

## 数据迁回
重复以上步骤将数据迁移会ult_index索引

## 后续问题

重新构建好索引后发现依旧抛出异常
````
2020-08-15 20:29:53.505 [http-nio-8080-exec-2] ERROR o.a.c.c.C.[.[localhost].[/].[dispatcherServlet] - Servlet.service() for servlet [dispatcherServlet] in context with path [] threw exception [Request processing failed; nested exception is ElasticsearchStatusException[Elasticsearch exception [type=search_phase_execution_exception, reason=all shards failed]]; nested: ElasticsearchException[Elasticsearch exception [type=illegal_argument_exception, reason=Text fields are not optimised for operations that require per-document field data like aggregations and sorting, so these operations are disabled by default. Please use a keyword field instead. Alternatively, set fielddata=true on [module] in order to load field data by uninverting the inverted index. Note that this can use significant memory.]]; nested: ElasticsearchException[Elasticsearch exception [type=illegal_argument_exception, reason=Text fields are not optimised for operations that require per-document field data like aggregations and sorting, so these operations are disabled by default. Please use a keyword field instead. Alternatively, set fielddata=true on [module] in order to load field data by uninverting the inverted index. Note that this can use significant memory.]];] with root cause
org.elasticsearch.ElasticsearchException: Elasticsearch exception [type=illegal_argument_exception, reason=Text fields are not optimised for operations that require per-document field data like aggregations and sorting, so these operations are disabled by default. Please use a keyword field instead. Alternatively, set fielddata=true on [module] in order to load field data by uninverting the inverted index. Note that this can use significant memory.]
	at org.elasticsearch.ElasticsearchException.innerFromXContent(ElasticsearchException.java:491)
	at org.elasticsearch.ElasticsearchException.fromXContent(ElasticsearchException.java:402)
	at org.elasticsearch.ElasticsearchException.innerFromXContent(ElasticsearchException.java:432)
	at org.elasticsearch.ElasticsearchException.fromXContent(ElasticsearchException.java:402)
	at org.elasticsearch.ElasticsearchException.innerFromXContent(ElasticsearchException.java:432)
	at org.elasticsearch.ElasticsearchException.failureFromXContent(ElasticsearchException.java:598)
	at org.elasticsearch.rest.BytesRestResponse.errorFromXContent(BytesRestResponse.java:169)
	at org.elasticsearch.client.RestHighLevelClient.parseEntity(RestHighLevelClient.java:1721)
	at org.elasticsearch.client.RestHighLevelClient.parseResponseException(RestHighLevelClient.java:1698)
	at org.elasticsearch.client.RestHighLevelClient.internalPerformRequest(RestHighLevelClient.java:1461)
	at org.elasticsearch.client.RestHighLevelClient.performRequest(RestHighLevelClient.java:1418)
	at org.elasticsearch.client.RestHighLevelClient.performRequestAndParseEntity(RestHighLevelClient.java:1388)
	at org.elasticsearch.client.RestHighLevelClient.search(RestHighLevelClient.java:930)
	at com.ultlog.ula.service.impl.StatisticServiceImpl.getProjectStatisticByModuleAndTime(StatisticServiceImpl.java:69)
	at com.ultlog.ula.controller.StatisticController.getStatisticByProjectAndDate(StatisticController.java:44)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	.....

````

翻了下文档发现执行以下命令即可，这个命令也将添加再v1.1.0版本的安装说明中。相关的说明可见[官方文档](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/fielddata.html#fielddata-disabled-text-fields)
````shell
curl -X PUT "localhost:9200/megacorp/_mapping?pretty" -H 'Content-Type: application/json' -d'{"properties": {"module": { "type":"text","fielddata": true}}}'
````

# 其他
elasticsearch的官方文档是真的好用




