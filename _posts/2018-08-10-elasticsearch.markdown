---
layout: post
title: elasticsearch
date: 2018-08-10 13:32:20 +0800
description: 认识elasticsearch
img: es.jpg # Add image post (optional)
tags: [elasticsearch]
---

ElasticSearch
===========
>ES的底层是开源库Lucene，它对Lucene进行了封装提供了REST API的操作接口。

# 安装
  * 官网下载并解压缩，然后去bin目录下各环境运行各环境的执行文件，即可跑去服务
  * 测试服务是否正常启动：curl localhost:9200

# 概念

 * Node : 一个单节点
 * Cluster ： 一个集群
 * Index ：ES数据管理的顶层单位
 * Document : Index里面单条的记录称为Document，用JSON格式表示，最好保持相同的结构（scheme）,有利于提高搜索效率
 * Type ：Document 可以分组，比如weather这个 Index 里面，可以按城市分组（北京和上海），也可以按气候分组（晴天和雨天）。这种分组就叫做 Type，它是虚拟的逻辑分组，用来过滤 Document。(6.x 版只允许每个 Index 包含一个 Type，7.x 版将会彻底移除 Type。)

# 创建和删除Index

 创建
 curl -X PUT localhost:9200/weather

 返回
 (```)
 {
 	"acknowledged" : true,
 	"shards_acknowledged" : true,
 	"index" : "weather"
 }
(```)
 删除

 curl -X DELETE localhost:9200/weather

 返回
 {"acknowledged":true}

# 中文分词设置

在Windows环境安装插件命令，Linux环境执行elasticsearch-plugin
elasticsearch-plugin.bat install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v6.3.2/elasticsearch-analysis-ik-6.3.2.zip
安装完会提示：-> Installed analysis-ik， 然后重启ES

## 测试
使用PostMan发送PUT请求，请求地址localhost:9200/accounts，请求body如下：
(```)
{
  "mappings": {
    "person": {
      "properties": {
        "user": {
          "type": "text",
          "analyzer": "ik_max_word",
          "search_analyzer": "ik_max_word"
        },
        "title": {
          "type": "text",
          "analyzer": "ik_max_word",
          "search_analyzer": "ik_max_word"
        },
        "desc": {
          "type": "text",
          "analyzer": "ik_max_word",
          "search_analyzer": "ik_max_word"
        }
      }
    }
  }
}
(```)
解析：首先新建一个名称为accounts的 Index，里面有一个名称为person的 Type。person有三个字段(user、title、desc)
上面的analyzer是字段文本的分词器，search_analyzer是搜索词的分词器。ik_max_word分词器是插件ik提供的，可以对文本进行最大数量的分词。

#健康检查
(```)
  "cluster_name" : "elasticsearch",   集群名称
  "status" : "yellow",  green：最健康得状态，说明所有的分片包括备份都可用；yellow ：基本的分片可用，但是备份不可用（或者是没有备份）；red：部分的分片可用，表明分片有一部分损坏。此时执行查询部分数据仍然可以查到，遇到这种情况，还是赶快解决比较好
  "timed_out" : false,
  "number_of_nodes" : 1,           集群个数
  "number_of_data_nodes" : 1,
  "active_primary_shards" : 5,
  "active_shards" : 5,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 5,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 50.0
(```)

#问题
(```)
{
    "error": {
        "root_cause": [
            {
                "type": "cluster_block_exception",
                "reason": "blocked by: [FORBIDDEN/12/index read-only / allow delete (api)];"
            }
        ],
        "type": "cluster_block_exception",
        "reason": "blocked by: [FORBIDDEN/12/index read-only / allow delete (api)];"
    },
    "status": 403
}
(```)
原因：
这是由于ES新节点的数据目录data存储空间不足，导致从master主节点接收同步数据的时候失败，此时ES集群为了保护数据，会自动把索引分片index置为只读read-only


解决步骤：

1、提供足够的存储空间供数据写入，如需在配置文件中更改ES数据存储目录，注意重启ES

2、剩余磁盘空间达到es最小值，添加数据被block

PUT [xxx]_all/_settings {"index.blocks.read_only_allow_delete": null}
