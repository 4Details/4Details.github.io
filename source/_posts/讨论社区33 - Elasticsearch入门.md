---
title: 讨论社区33  -  Elasticsearch入门
date: 2019-08-06 22:38:50
tags: [Elasticsearch]
category: [讨论社区项目]
---

https://elastic.co

# Elasticsearch简介

- 一个分布式的、Restful风格（请求标准的描述）的搜索引擎。
  - 支持对各种类型的数据的检索。
  - 搜索速度快，可以提供实时的搜索服务。
  - 便于水平扩展，每秒可以处理PB级海量数据。

# Elasticsearch术语

- 索引（对应数据库）、类型（对应表）、文档（表里一行）、字段（一列）。   最新的版本类型被废弃。                                                                                                                                                                                                                                                                                                                                                                                                                               
- 集群(服务器组合在一起)、节点（集群中每台服务器）、分片（对索引的划分）、副本（分片的备份）。

通过ES搜索的数据必须要在ES中转存一份，某种角度来说它是一个数据库。

# Elasticsearch使用

- 安装、修改配置文件

  - elasticsearch.yml文件，修改cluster.name，path.data，path.logs

  ```yaml
  # ----------------------------------- Paths ------------------------------------
  #
  # Path to directory where to store the data (separate multiple locations by comma):
  #
  path.data: d:\work\data\elasticsearch-6.4.3\data
  #
  # Path to log files:
  #
  path.logs: d:\work\data\elasticsearch-6.4.3\logs
  #
  ```

  - 配置环境变量



- 安装中文分词插件（ES仅支持中文分词）

  github下载   https://github.com/medcl/elasticsearch-analysis-ik

  - ik插件安装（解压缩）到plugins文件夹下，注意版本对应

  config目录下`IKAnalyzer.cfg.xml`可以配置自定义词典



- 安装postman(提交html数据给ES)模拟web客户端，或者在浏览器安装postman插件



- 启动ES:打开 bin/elasticsearch.bat

  - 查看集群健康状态：curl -X GET "localhost:9200/_cat/health?v"

  - 查看节点：curl -X GET "localhost:9200/_cat/nodes?v"
  - 查看索引：curl -X GET "localhost:9200/_cat/indices?v"
  - 创建索引：curl -X PUT "localhost:9200/test"
  - 删除索引：curl -X DELETE "localhost:9200/test"

![Es使用](https://i2.tiimg.com/726074/306767e7e8b55b5f.png)

`PUT`,`GET`,`DELETE` HTTP协议的请求

- 使用postman查询
  - 提交数据，PUT localhost:9200/test/_doc/1选择Body,raw,JSON
  - 搜索，GET localhost:9200/test/_search?q=title(/content):xxx
  - 搜索时ES对关键词进行了分词
  - 通过请求体构造复杂搜索条件

```html
<!--构造搜索条件  网址/索引名/_search?q=字段:字段名 -->
localhost:9200/test/_search?q=title:${search_condition}

<!--构造复杂搜索条件  网址/索引名/_search 提交请求体为JSON数据 -->
localhost:9200/test/_search
```

```json
插入test的数据请求体为
{
    "title":"...",
    "content":"...."
}

复杂查询请求体为：
{
"query":{
"multi_match":{
	"query": "条件",
	"fields": ["title","content"]
			}
}
}
```

