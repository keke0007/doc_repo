# Elasticsearch 学习整理

> 本文档由 `es1.md`、`es2.md`、`es3.md` 归纳整理而来，按学习路径重新组织：先理解概念，再掌握索引、映射、分词、查询、聚合、集群和 ELK 数据同步。每个知识点都包含代码示例和典型应用场景。

## 目录

1. [Elasticsearch 概览](#1-elasticsearch-概览)
2. [核心概念](#2-核心概念)
3. [安装与启动](#3-安装与启动)
4. [索引管理](#4-索引管理)
5. [Mapping 映射与字段类型](#5-mapping-映射与字段类型)
6. [分词器与中文分词](#6-分词器与中文分词)
7. [文档 CRUD](#7-文档-crud)
8. [查询 DSL](#8-查询-dsl)
9. [高亮、模糊搜索与搜索建议](#9-高亮模糊搜索与搜索建议)
10. [聚合查询](#10-聚合查询)
11. [GEO 地理位置查询](#11-geo-地理位置查询)
12. [集群、节点、分片与副本](#12-集群节点分片与副本)
13. [故障转移与扩容](#13-故障转移与扩容)
14. [ELK 与异构数据同步](#14-elk-与异构数据同步)
15. [Kibana 数据分析](#15-kibana-数据分析)
16. [学习练习路线](#16-学习练习路线)

## 1. Elasticsearch 概览

### 1.1 Elasticsearch 是什么

Elasticsearch，简称 ES，是基于 Apache Lucene 构建的分布式搜索和分析引擎。它以 JSON 文档为核心数据模型，通过 RESTful API 提供索引、搜索、分析和聚合能力。

ES 适合处理以下问题：

- 数据量大，传统数据库 `LIKE '%关键字%'` 查询性能差。
- 需要全文检索、相关性排序、关键词高亮、纠错、搜索建议。
- 需要对日志、指标、事件、业务数据做实时分析和可视化。
- 需要水平扩展和高可用。

### 1.2 ES 的特点

| 特点 | 说明 | 应用场景 |
| --- | --- | --- |
| 近实时 | 写入后通常约 1 秒可搜索 | 日志检索、商品搜索 |
| 分布式 | 数据自动切分到多个分片和节点 | 大规模数据、横向扩容 |
| 全文检索 | 基于倒排索引和分词 | 博客、文档、商品标题搜索 |
| JSON 文档 | 数据以 JSON 文档存储 | Web 后端、数据同步 |
| RESTful API | HTTP 接口易接入 | Java、Python、Go、Node.js 等系统 |
| 聚合分析 | 支持指标、分桶、管道聚合 | 报表、统计、仪表盘 |

### 1.3 ES 不适合什么

ES 不是关系型数据库的完全替代品，不适合把强事务作为核心诉求的场景。

| 不适合场景 | 原因 |
| --- | --- |
| 强事务业务 | ES 不提供 MySQL 那样的 ACID 多表事务 |
| 高频更新同一文档 | ES 更新本质是重新索引，成本高于普通数据库 |
| 复杂 JOIN | ES 更适合反范式文档模型 |
| 作为唯一数据源保存核心业务数据 | 通常 MySQL/PostgreSQL 作为主库，ES 作为检索库 |

## 2. 核心概念

### 2.1 Index 索引

索引类似数据库中的表，用于存放一类文档。

代码示例：

```http
PUT products
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 1
  }
}
```

应用场景：

- `products`：商品搜索索引。
- `articles`：文章全文检索索引。
- `logstash-nginx-2026.05.10`：按天保存日志索引。

### 2.2 Document 文档

文档是 ES 中最小的数据单元，本质是 JSON。

代码示例：

```http
PUT products/_doc/1
{
  "name": "小米手机 15",
  "brand": "小米",
  "category": "手机",
  "price": 3999,
  "tags": ["5G", "快充", "拍照"],
  "created_at": "2026-05-10 10:00:00"
}
```

应用场景：

- 一个商品是一篇文档。
- 一条日志是一篇文档。
- 一个小区、一篇文章、一个用户画像都可以是一篇文档。

### 2.3 Field 字段

字段是文档中的属性。不同字段可以有不同类型，例如 `keyword`、`text`、`integer`、`date`、`geo_point`。

代码示例：

```json
{
  "name": "龙源居住区",
  "province": "北京市",
  "houses": 1200,
  "location": {
    "lat": 40.066258,
    "lon": 116.349936
  }
}
```

应用场景：

- `name` 用于小区名称精确查询。
- `province` 用于按省份聚合统计。
- `location` 用于附近小区查询。

### 2.4 倒排索引

传统数据库更像“从文档找字段”，ES 的全文检索更像“从词找文档”。倒排索引会记录某个词出现在哪些文档中。

示例数据：

| 文档 ID | 内容 |
| --- | --- |
| 1 | 我喜欢篮球和跑步 |
| 2 | 篮球比赛很好看 |
| 3 | 我喜欢看电影 |

倒排索引简化后：

| 词 | 文档 ID |
| --- | --- |
| 喜欢 | 1, 3 |
| 篮球 | 1, 2 |
| 跑步 | 1 |
| 比赛 | 2 |
| 电影 | 3 |

应用场景：

- 用户搜索“篮球”时，ES 可以快速定位文档 1 和 2。
- 搜索引擎、站内搜索、日志检索都依赖倒排索引。

## 3. 安装与启动

### 3.1 Linux 单机启动

代码示例：

```bash
# 切换到 ES 用户，ES 不建议使用 root 启动
su elasticsearch

# 前台启动
sh bin/elasticsearch

# 后台启动
sh bin/elasticsearch -d
```

应用场景：

- 本地学习 ES 基础 API。
- 单机开发环境验证 Mapping、查询 DSL、聚合语法。

### 3.2 常见 Linux 参数

代码示例：

```bash
# 查看最大虚拟内存映射数
sysctl -a | grep vm.max_map_count

# 临时设置
sysctl -w vm.max_map_count=262144

# 永久设置：写入 /etc/sysctl.conf
vm.max_map_count=262144
```

```bash
# 修改文件句柄和线程数：/etc/security/limits.conf
* soft nofile 65536
* hard nofile 65536
* soft nproc 4096
* hard nproc 4096
```

应用场景：

- 生产或测试服务器启动 ES 报系统参数不足。
- 大量索引、大量 segment、大量连接时避免资源限制。

### 3.3 Kibana 启动

代码示例：

```bash
# 前台启动
sh bin/kibana

# 后台启动
nohup sh bin/kibana >/dev/null 2>&1 &
```

`kibana.yml` 示例：

```yaml
server.port: 5601
server.host: "0.0.0.0"
elasticsearch.hosts: ["http://localhost:9200"]
```

应用场景：

- 使用 Dev Tools 执行 ES API。
- 创建仪表盘、图表、地图。

## 4. 索引管理

### 4.1 查看所有索引

代码示例：

```http
GET _cat/indices?v
```

应用场景：

- 检查当前集群有哪些索引。
- 排查日志索引是否按天正常生成。

### 4.2 创建索引

代码示例：

```http
PUT articles
{
  "settings": {
    "number_of_shards": 3,
    "number_of_replicas": 1
  }
}
```

应用场景：

- 创建文章搜索索引。
- 创建商品检索索引。

注意：

- `number_of_shards` 创建后通常不能直接修改。
- `number_of_replicas` 可以动态修改。

### 4.3 查看索引详情

代码示例：

```http
GET articles
```

应用场景：

- 查看索引 settings、mappings、aliases。
- 排查字段类型是否符合预期。

### 4.4 判断索引是否存在

代码示例：

```http
HEAD articles
```

应用场景：

- 程序启动时判断索引是否需要初始化。
- 自动化脚本避免重复创建索引。

### 4.5 关闭与打开索引

代码示例：

```http
POST articles/_close
POST articles/_open
```

应用场景：

- 历史日志暂时不查询但想保留数据。
- 降低旧索引占用的运行资源。

### 4.6 删除索引

代码示例：

```http
DELETE articles
```

应用场景：

- 删除测试索引。
- 清理错误创建的索引。

注意：删除索引会删除索引内所有文档，生产环境必须谨慎。

## 5. Mapping 映射与字段类型

Mapping 定义字段类型、分词器、是否索引、日期格式等。它类似数据库中的表结构，但 ES 的 Mapping 更强调检索行为。

### 5.1 创建带 Mapping 的索引

代码示例：

```http
PUT village
{
  "settings": {
    "number_of_shards": 3,
    "number_of_replicas": 1
  },
  "mappings": {
    "dynamic": "strict",
    "properties": {
      "name": {
        "type": "keyword"
      },
      "province": {
        "type": "keyword"
      },
      "addr": {
        "type": "text",
        "analyzer": "ik_smart"
      },
      "producer": {
        "type": "text",
        "analyzer": "ik_smart"
      },
      "houses": {
        "type": "integer"
      },
      "volume": {
        "type": "float"
      },
      "greening": {
        "type": "float"
      },
      "built_year": {
        "type": "integer"
      },
      "created_at": {
        "type": "date",
        "format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis"
      },
      "location": {
        "type": "geo_point"
      }
    }
  }
}
```

应用场景：

- 小区检索系统：名称精确查、地址全文查、经纬度附近查、按省份聚合。
- 商品系统：商品名全文检索，品牌、分类用 `keyword` 精确过滤。

### 5.2 text 与 keyword

| 类型 | 是否分词 | 适合查询 | 适用字段 |
| --- | --- | --- | --- |
| `text` | 是 | `match`、全文检索 | 标题、正文、地址、描述 |
| `keyword` | 否 | `term`、过滤、排序、聚合 | ID、状态、分类、品牌、省份 |

代码示例：

```http
PUT mapping_demo
{
  "mappings": {
    "properties": {
      "name": {
        "type": "keyword"
      },
      "city": {
        "type": "text",
        "analyzer": "ik_smart"
      }
    }
  }
}
```

写入数据：

```http
PUT mapping_demo/_doc/1
{
  "name": "北京小区",
  "city": "北京市昌平区回龙观街道"
}
```

精确查询 `keyword`：

```http
GET mapping_demo/_search
{
  "query": {
    "term": {
      "name": "北京小区"
    }
  }
}
```

全文查询 `text`：

```http
GET mapping_demo/_search
{
  "query": {
    "match": {
      "city": "昌平区"
    }
  }
}
```

应用场景：

- `name` 如果必须完整匹配，使用 `keyword`。
- `city`、`addr` 需要按词搜索，使用 `text`。

### 5.3 数字类型

常见数字类型：

| 类型 | 说明 | 应用场景 |
| --- | --- | --- |
| `byte` | 小整数 | 状态码、小范围枚举 |
| `short` | 短整数 | 年龄、评分 |
| `integer` | 整数 | 户数、库存、点击数 |
| `long` | 长整数 | 雪花 ID、累计 PV |
| `float` | 单精度浮点 | 绿化率、评分 |
| `double` | 双精度浮点 | 金额计算不推荐直接用 |
| `scaled_float` | 缩放浮点 | 金额、价格、比例 |

代码示例：

```http
PUT product_price
{
  "mappings": {
    "properties": {
      "price": {
        "type": "scaled_float",
        "scaling_factor": 100
      },
      "stock": {
        "type": "integer"
      }
    }
  }
}
```

应用场景：

- 价格 `3999.99` 可以使用 `scaled_float`，内部按 `399999` 存储，适合排序和聚合。

### 5.4 日期类型

ES 内部会把日期转换为 UTC 毫秒值存储，展示时再按格式返回。

代码示例：

```http
PUT date_demo
{
  "mappings": {
    "properties": {
      "created_at": {
        "type": "date",
        "format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis"
      }
    }
  }
}
```

写入数据：

```http
PUT date_demo/_doc/1
{
  "created_at": "2026-05-10 12:30:00"
}
```

范围查询：

```http
GET date_demo/_search
{
  "query": {
    "range": {
      "created_at": {
        "gte": "2026-05-01",
        "lte": "2026-05-31"
      }
    }
  }
}
```

应用场景：

- 日志时间过滤。
- 订单创建时间范围查询。
- 按天、按月聚合统计。

### 5.5 布尔类型

代码示例：

```http
PUT boolean_demo
{
  "mappings": {
    "properties": {
      "is_school_district": {
        "type": "boolean"
      }
    }
  }
}
```

应用场景：

- 是否上架。
- 是否学区房。
- 是否删除。

### 5.6 范围类型

ES 支持 `integer_range`、`float_range`、`long_range`、`double_range`、`date_range`、`ip_range`。

代码示例：

```http
PUT range_demo
{
  "mappings": {
    "properties": {
      "age_range": {
        "type": "integer_range"
      },
      "active_time": {
        "type": "date_range"
      }
    }
  }
}
```

写入数据：

```http
PUT range_demo/_doc/1
{
  "age_range": {
    "gte": 20,
    "lt": 30
  },
  "active_time": {
    "gte": "2026-05-01",
    "lte": "2026-05-31"
  }
}
```

应用场景：

- 年龄区间筛选。
- 活动有效期匹配。
- IP 段匹配。

## 6. 分词器与中文分词

### 6.1 分词器是什么

分词器 Analyzer 用于把一段文本转换成一组 token。ES 建立倒排索引和执行全文检索时都会用到分词。

Analyzer 由三部分组成：

| 组件 | 作用 | 示例 |
| --- | --- | --- |
| Character Filter | 处理原始字符 | 去 HTML 标签 |
| Tokenizer | 切词 | 按空格、按标准语法切分 |
| Token Filter | 处理 token | 小写化、去停用词 |

### 6.2 测试分词

代码示例：

```http
POST _analyze
{
  "analyzer": "standard",
  "text": "Java is the best language in the world."
}
```

应用场景：

- 验证字段分词结果是否符合预期。
- 排查 `match` 查询为什么搜不到或搜出太多结果。

### 6.3 内置分词器

| 分词器 | 说明 | 应用场景 |
| --- | --- | --- |
| `standard` | 默认分词器，按 Unicode 规则切分 | 英文、通用文本 |
| `simple` | 按非字母切分，并转小写 | 简单英文搜索 |
| `whitespace` | 按空白字符切分 | 已经预处理好的文本 |
| `stop` | 类似 simple，但会去停用词 | 英文文章 |
| `keyword` | 不分词，整体作为一个 token | 精确匹配、枚举值 |
| `pattern` | 按正则切分 | 特殊格式日志 |

代码示例：

```http
POST _analyze
{
  "analyzer": "keyword",
  "text": "The 2 QUICK Brown-Foxes"
}
```

应用场景：

- 订单号、手机号、身份证号不应该被拆分，可用 `keyword`。

### 6.4 IK 中文分词器

IK 常用两种模式：

| 分词器 | 说明 | 应用场景 |
| --- | --- | --- |
| `ik_smart` | 智能粗粒度分词 | 搜索时使用，减少噪音 |
| `ik_max_word` | 最细粒度分词 | 建索引时使用，提高召回 |

测试示例：

```http
GET _analyze
{
  "analyzer": "ik_smart",
  "text": "传智教育的教学质量是杠杠的"
}
```

```http
GET _analyze
{
  "analyzer": "ik_max_word",
  "text": "传智教育的教学质量是杠杠的"
}
```

应用场景：

- 中文商品标题搜索。
- 小区地址和开发商搜索。
- 文章正文全文检索。

### 6.5 建索引和搜索时指定不同分词器

代码示例：

```http
PUT analyzer_demo
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "analyzer": "ik_max_word",
        "search_analyzer": "ik_smart"
      }
    }
  }
}
```

应用场景：

- 建索引时尽量细分，提高可搜索范围。
- 搜索时适度粗分，减少无关结果。

## 7. 文档 CRUD

### 7.1 新增或覆盖文档

代码示例：

```http
PUT products/_doc/1
{
  "name": "MacBook Pro 14",
  "brand": "Apple",
  "price": 14999,
  "category": "笔记本电脑"
}
```

应用场景：

- 同步 MySQL 商品数据到 ES。
- 指定业务主键作为文档 ID，方便更新。

### 7.2 自动生成 ID

代码示例：

```http
POST products/_doc
{
  "name": "iPad Pro",
  "brand": "Apple",
  "price": 8999
}
```

应用场景：

- 写入日志、事件数据，业务不关心文档 ID。

### 7.3 查询单个文档

代码示例：

```http
GET products/_doc/1
```

应用场景：

- 根据 ID 查看文档详情。
- 排查某条数据是否同步成功。

### 7.4 局部更新

代码示例：

```http
POST products/_update/1
{
  "doc": {
    "price": 13999
  }
}
```

应用场景：

- 更新商品价格。
- 更新文章浏览数。

### 7.5 删除文档

代码示例：

```http
DELETE products/_doc/1
```

应用场景：

- 商品下架后从搜索索引删除。
- 数据源删除后同步删除 ES 文档。

### 7.6 批量写入 Bulk

代码示例：

```http
POST _bulk
{"index":{"_index":"products","_id":"1"}}
{"name":"小米手机 15","brand":"小米","price":3999}
{"index":{"_index":"products","_id":"2"}}
{"name":"华为 MateBook","brand":"华为","price":6999}
{"delete":{"_index":"products","_id":"3"}}
```

应用场景：

- 批量导入历史数据。
- Logstash、Canal、应用程序批量同步数据。

## 8. 查询 DSL

### 8.1 查询所有文档

代码示例：

```http
GET products/_search
{
  "query": {
    "match_all": {}
  }
}
```

应用场景：

- 验证索引内是否有数据。
- 分页浏览后台数据。

### 8.2 match 全文查询

`match` 会对查询词进行分词，适合查 `text` 字段。

代码示例：

```http
GET village/_search
{
  "query": {
    "match": {
      "addr": "北京市昌平区回龙观"
    }
  }
}
```

应用场景：

- 地址搜索。
- 商品名称搜索。
- 文章正文搜索。

### 8.3 term 精确查询

`term` 不会对查询值分词，适合查 `keyword`、数字、布尔值。

代码示例：

```http
GET village/_search
{
  "query": {
    "term": {
      "province": "北京市"
    }
  }
}
```

应用场景：

- 按省份过滤。
- 按状态、分类、品牌过滤。
- 按 ID、编码精确查询。

注意：

- 不要直接用 `term` 查普通 `text` 字段的完整中文句子，因为字段已被分词，完整句子可能不存在于倒排索引中。

### 8.4 match 与 term 的区别

| 查询 | 是否分词 | 适合字段 | 用途 |
| --- | --- | --- | --- |
| `match` | 是 | `text` | 全文搜索 |
| `term` | 否 | `keyword`、数字、布尔 | 精确过滤 |

应用场景：

- 搜索“上海新安房地产有限公司”这种开发商名称，如果字段是 `text`，`match` 会拆词，可能返回不完全相关结果。
- 如果需要完整公司名精确匹配，应额外建一个 `producer.keyword` 或把字段设为 `keyword`。

### 8.5 multi_match 多字段搜索

代码示例：

```http
GET village/_search
{
  "query": {
    "multi_match": {
      "query": "回龙观 学区",
      "fields": ["name^3", "addr", "info"]
    }
  }
}
```

应用场景：

- 搜索框同时查标题、地址、简介。
- `name^3` 表示名称字段权重更高。

### 8.6 range 范围查询

代码示例：

```http
GET village/_search
{
  "query": {
    "range": {
      "built_year": {
        "gte": 2020,
        "lte": 2026
      }
    }
  }
}
```

应用场景：

- 查询 2020 年以后新建小区。
- 查询价格区间。
- 查询时间范围内的日志。

### 8.7 exists 字段存在查询

代码示例：

```http
GET village/_search
{
  "query": {
    "exists": {
      "field": "school"
    }
  }
}
```

应用场景：

- 查有学区信息的小区。
- 查某字段已补齐的数据。

### 8.8 bool 组合查询

`bool` 是最常用的组合查询。

| 子句 | 是否参与评分 | 说明 |
| --- | --- | --- |
| `must` | 是 | 必须匹配 |
| `filter` | 否 | 必须匹配，结果可缓存 |
| `should` | 是 | 可选匹配，用于提高相关性 |
| `must_not` | 否 | 必须不匹配 |

代码示例：

```http
GET village/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "addr": "回龙观"
          }
        }
      ],
      "filter": [
        {
          "term": {
            "province": "北京市"
          }
        },
        {
          "range": {
            "built_year": {
              "gte": 2015
            }
          }
        }
      ],
      "must_not": [
        {
          "term": {
            "property_type": "商业"
          }
        }
      ]
    }
  }
}
```

应用场景：

- 搜索北京市回龙观附近、2015 年后建成、排除商业地产的小区。
- 电商搜索中，关键词用 `must`，品牌、价格、分类用 `filter`。

### 8.9 排序、分页与字段过滤

代码示例：

```http
GET village/_search
{
  "_source": ["name", "province", "city", "houses", "built_year"],
  "query": {
    "term": {
      "province": "北京市"
    }
  },
  "sort": [
    {
      "houses": {
        "order": "desc"
      }
    }
  ],
  "from": 0,
  "size": 10
}
```

应用场景：

- 后台列表分页。
- 搜索结果只返回前端需要的字段。

注意：

- 深分页不要长期使用大 `from`，大量数据应考虑 `search_after`。

## 9. 高亮、模糊搜索与搜索建议

### 9.1 高亮显示

代码示例：

```http
GET village/_search
{
  "query": {
    "match": {
      "addr": "回龙观"
    }
  },
  "highlight": {
    "pre_tags": ["<em>"],
    "post_tags": ["</em>"],
    "fields": {
      "addr": {}
    }
  }
}
```

应用场景：

- 搜索结果中标红关键词。
- 日志检索页面突出错误关键字。

### 9.2 fuzzy 模糊查询

`fuzzy` 基于编辑距离，能容忍少量拼写错误。

代码示例：

```http
GET products/_search
{
  "query": {
    "fuzzy": {
      "name": {
        "value": "iphne",
        "fuzziness": "AUTO"
      }
    }
  }
}
```

应用场景：

- 英文商品名拼写纠错。
- 用户输入有轻微错误时仍能搜索到结果。

注意：

- 中文模糊查询效果受分词影响较大，常与拼音分词、自定义词典、suggest 组合使用。

### 9.3 搜索建议 Suggester

代码示例：

```http
GET village/_search
{
  "suggest": {
    "name_suggestion": {
      "text": "龙原居住区",
      "term": {
        "field": "name"
      }
    }
  }
}
```

应用场景：

- 用户输入错误时给出“你是不是想搜 xxx”。
- 搜索框自动补全。

### 9.4 前缀搜索

代码示例：

```http
GET products/_search
{
  "query": {
    "prefix": {
      "brand": "App"
    }
  }
}
```

应用场景：

- 输入前几个字符后给出品牌或关键词提示。

## 10. 聚合查询

聚合用于统计分析，类似 SQL 中的 `count`、`sum`、`avg`、`group by`。

### 10.1 max 最大值

代码示例：

```http
GET village/_search
{
  "size": 0,
  "aggs": {
    "max_volume": {
      "max": {
        "field": "volume",
        "missing": 0
      }
    }
  }
}
```

应用场景：

- 查询容积率最高的小区。
- 查询商品最高价格。

### 10.2 min 最小值

代码示例：

```http
GET village/_search
{
  "size": 0,
  "aggs": {
    "min_parkings": {
      "min": {
        "field": "parkings",
        "missing": 0
      }
    }
  }
}
```

应用场景：

- 查询停车位最少的小区。
- 查询库存最低的商品。

### 10.3 stats 综合指标

`stats` 一次返回 `count`、`min`、`max`、`avg`、`sum`。

代码示例：

```http
GET village/_search
{
  "size": 0,
  "query": {
    "term": {
      "province": "河南省"
    }
  },
  "aggs": {
    "stats_greenings": {
      "stats": {
        "field": "greening"
      }
    }
  }
}
```

应用场景：

- 统计某省小区绿化率情况。
- 统计某类商品价格分布。

### 10.4 terms 分桶聚合

代码示例：

```http
GET village/_search
{
  "size": 0,
  "aggs": {
    "province_count": {
      "terms": {
        "field": "province",
        "size": 10
      }
    }
  }
}
```

应用场景：

- 统计各省小区数量。
- 统计各品牌商品数量。
- 统计日志中各状态码数量。

### 10.5 分桶后继续统计

代码示例：

```http
GET village/_search
{
  "size": 0,
  "aggs": {
    "province_bucket": {
      "terms": {
        "field": "province",
        "size": 10
      },
      "aggs": {
        "sum_houses": {
          "sum": {
            "field": "houses"
          }
        }
      }
    }
  }
}
```

应用场景：

- 统计各省小区总户数。
- 统计各分类商品总销售额。

### 10.6 带查询条件的聚合

代码示例：

```http
GET village/_search
{
  "size": 0,
  "query": {
    "exists": {
      "field": "school"
    }
  },
  "aggs": {
    "province_bucket": {
      "terms": {
        "field": "province",
        "size": 10
      },
      "aggs": {
        "sum_houses": {
          "sum": {
            "field": "houses"
          }
        }
      }
    }
  }
}
```

应用场景：

- 只统计学区房数据。
- 只统计某时间范围内的日志。

### 10.7 管道聚合 max_bucket

代码示例：

```http
GET village/_search
{
  "size": 0,
  "query": {
    "exists": {
      "field": "school"
    }
  },
  "aggs": {
    "province_bucket": {
      "terms": {
        "field": "province",
        "size": 10
      },
      "aggs": {
        "sum_houses": {
          "sum": {
            "field": "houses"
          }
        }
      }
    },
    "max_houses": {
      "max_bucket": {
        "buckets_path": "province_bucket>sum_houses"
      }
    }
  }
}
```

应用场景：

- 找出学区房住户总数最多的省份。
- 找出销售额最高的分类。

## 11. GEO 地理位置查询

### 11.1 geo_point 映射

代码示例：

```http
PUT geo_demo
{
  "mappings": {
    "properties": {
      "name": {
        "type": "keyword"
      },
      "location": {
        "type": "geo_point"
      }
    }
  }
}
```

写入数据：

```http
PUT geo_demo/_doc/1
{
  "name": "金燕龙办公楼",
  "location": {
    "lat": 40.066258,
    "lon": 116.349936
  }
}
```

应用场景：

- 门店位置。
- 小区位置。
- 外卖、打车、附近的人。

### 11.2 查询某坐标附近数据

代码示例：

```http
GET geo_demo/_search
{
  "query": {
    "bool": {
      "filter": [
        {
          "geo_distance": {
            "distance": "5km",
            "location": {
              "lat": 40.066258,
              "lon": 116.349936
            }
          }
        }
      ]
    }
  }
}
```

应用场景：

- 查询 5 公里内的小区。
- 查询附近门店。
- 查询某个地理点附近的设备或事件。

## 12. 集群、节点、分片与副本

### 12.1 集群健康状态

代码示例：

```http
GET _cluster/health
```

状态说明：

| 状态 | 说明 | 影响 |
| --- | --- | --- |
| `green` | 主分片和副本分片都正常 | 集群健康 |
| `yellow` | 主分片正常，部分副本未分配 | 数据可用，但高可用不足 |
| `red` | 有主分片不可用 | 部分数据不可用 |

应用场景：

- 监控集群是否健康。
- 排查索引副本未分配或节点故障。

### 12.2 节点

节点是一个 ES 进程。多个节点组成集群。

常见节点角色：

| 角色 | 说明 | 应用场景 |
| --- | --- | --- |
| master eligible | 可竞选主节点 | 管理集群元数据 |
| data | 存储数据和执行查询 | 业务数据节点 |
| ingest | 数据预处理 | 写入前清洗字段 |
| coordinating | 协调请求 | 接收客户端请求 |

应用场景：

- 小集群可以一个节点承担多个角色。
- 大集群通常拆分 master、data、coordinating 节点。

### 12.3 主分片

主分片是索引数据的基本切分单元。

代码示例：

```http
PUT shard_demo
{
  "settings": {
    "number_of_shards": 3,
    "number_of_replicas": 0
  }
}
```

应用场景：

- 数据量较大时使用多个主分片分散存储和查询压力。
- 三节点集群中，常见配置是 3 个主分片起步。

注意：

- 分片不是越多越好。过多分片会增加集群管理、内存和文件句柄成本。

### 12.4 副本分片

副本是主分片的复制，用于高可用和提升查询吞吐。

代码示例：

```http
PUT replica_demo
{
  "settings": {
    "number_of_shards": 3,
    "number_of_replicas": 1
  }
}
```

动态修改副本数：

```http
PUT replica_demo/_settings
{
  "number_of_replicas": 2
}
```

应用场景：

- 节点宕机时，副本可提升为主分片。
- 查询压力大时，副本可以分摊查询。

注意：

- 同一个主分片和它的副本不会分配到同一节点。
- 三节点集群中，`number_of_replicas: 2` 意味着每份主分片有两个副本，可以让三台节点都保存一份数据。

### 12.5 分片与副本组合示例

1 个主分片，2 个副本：

```http
PUT test3
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 2
  }
}
```

3 个主分片，2 个副本：

```http
PUT test7
{
  "settings": {
    "number_of_shards": 3,
    "number_of_replicas": 2
  }
}
```

应用场景：

- 三节点学习集群可以用 `3 主分片 + 1 或 2 副本` 理解分片分布。
- 生产环境要结合数据量、节点数、查询量、恢复时间综合设计。

## 13. 故障转移与扩容

### 13.1 故障转移

当某个节点宕机后，master 节点会重新分配分片。如果主分片丢失但有副本，副本会被提升为主分片。

应用场景：

- 一台数据节点故障，集群仍然可查询和写入。
- 运维重启节点时，集群自动恢复。

验证示例：

```bash
# 暂停节点
docker pause node-2

# 恢复节点
docker unpause node-2
```

观察命令：

```http
GET _cluster/health
GET _cat/shards?v
```

### 13.2 为什么副本数量不能超过可用节点承载能力

如果一个索引有 1 个主分片，设置 3 个副本，但集群只有 3 个节点，那么每个主分片最多只能分配到另外 2 个节点上的副本，多出来的副本会 `unassigned`，集群可能变成 `yellow`。

应用场景：

- 看到 `yellow` 时，检查副本数是否大于可分配节点数。

修复示例：

```http
PUT test4/_settings
{
  "number_of_replicas": 2
}
```

### 13.3 扩容节点

节点配置示例：

```yaml
cluster.name: elastic
node.name: node-4
node.master: true
node.data: true
path.data: /usr/share/elasticsearch/data
path.logs: /usr/share/elasticsearch/log
network.host: 0.0.0.0
http.port: 9200
transport.tcp.port: 9300
discovery.seed_hosts: ["node-1:9300", "node-2:9300", "node-3:9300"]
cluster.initial_master_nodes: ["node-1", "node-2", "node-3"]
```

应用场景：

- 数据量增长，需要增加存储和查询能力。
- 某些节点压力过高，需要横向扩展。

扩容后观察：

```http
GET _cat/nodes?v
GET _cat/allocation?v
GET _cat/shards?v
```

## 14. ELK 与异构数据同步

### 14.1 ELK 是什么

| 组件 | 作用 | 应用场景 |
| --- | --- | --- |
| Elasticsearch | 存储、搜索、分析 | 日志索引、业务检索 |
| Logstash | 采集、清洗、转换、输出 | 从 MQ、文件、数据库同步数据 |
| Kibana | 可视化、查询、仪表盘 | 日志分析、运营报表 |

### 14.2 日志收集场景

传统 `grep`、`awk`、`sed` 适合单机小规模排查。系统规模变大后，需要集中日志系统。

典型链路：

```text
应用日志 -> Filebeat/Logstash -> Elasticsearch -> Kibana
```

应用场景：

- 多台服务器统一查日志。
- 统计接口错误率、响应时间。
- 创建告警和仪表盘。

### 14.3 MySQL 到 ES 的异构同步

典型链路：

```text
MySQL -> Canal 解析 binlog -> RabbitMQ 缓冲 -> Logstash 清洗 -> Elasticsearch
```

组件职责：

| 组件 | 作用 |
| --- | --- |
| MySQL | 主业务数据库 |
| Canal | 伪装成 MySQL 从库，解析 binlog |
| RabbitMQ | 削峰填谷，缓冲同步消息 |
| Logstash | 解析、转换、清洗数据 |
| Elasticsearch | 保存检索数据 |
| Kibana | 查看和分析同步后的数据 |

应用场景：

- 房产小区数据从 MySQL 同步到 ES 做全文检索。
- 商品、文章、订单搜索索引增量同步。

### 14.4 MySQL 开启 binlog

配置示例：

```ini
[mysqld]
server-id=1
log-bin=mysql-bin
binlog_format=ROW
```

创建 Canal 用户：

```sql
CREATE USER 'canal'@'%' IDENTIFIED BY 'canal';
GRANT SELECT, REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'canal'@'%';
FLUSH PRIVILEGES;
```

应用场景：

- Canal 需要读取 MySQL binlog，必须具备复制相关权限。

### 14.5 Logstash 从 RabbitMQ 读取并写入 ES

示例配置：

```conf
input {
  rabbitmq {
    host => "rabbit"
    port => 5672
    user => "guest"
    password => "guest"
    queue => "direct_queue"
    durable => true
    codec => "json"
  }
}

filter {
  if [data][name] {
    mutate {
      add_field => { "name" => "%{[data][name]}" }
    }
  }

  if [data][province] {
    mutate {
      add_field => { "province" => "%{[data][province]}" }
    }
  }

  if [data][lat_gps] and [data][lon_gps] {
    mutate {
      add_field => { "[location][lat]" => "%{[data][lat_gps]}" }
      add_field => { "[location][lon]" => "%{[data][lon_gps]}" }
      convert => { "[location][lat]" => "float" }
      convert => { "[location][lon]" => "float" }
    }
  }

  mutate {
    remove_field => ["data"]
  }
}

output {
  elasticsearch {
    hosts => ["http://192.168.245.151:9200"]
    index => "logstash-village-%{+YYYY.MM.dd}"
  }
}
```

应用场景：

- 把 Canal 消息转换成 ES 文档。
- 清洗字段名、转换数字类型、组装 `geo_point`。

### 14.6 索引模板

按天创建日志或业务索引时，应使用索引模板统一 settings 和 mappings。

代码示例：

```http
PUT _index_template/logstash-village
{
  "index_patterns": ["logstash-village-*"],
  "template": {
    "settings": {
      "number_of_shards": 3,
      "number_of_replicas": 2
    },
    "aliases": {
      "logstash-village": {}
    },
    "mappings": {
      "dynamic": "strict",
      "properties": {
        "@timestamp": {
          "type": "date",
          "format": "strict_date_optional_time||epoch_millis||yyyy-MM-dd HH:mm:ss"
        },
        "name": {
          "type": "keyword"
        },
        "province": {
          "type": "keyword"
        },
        "city": {
          "type": "keyword"
        },
        "addr": {
          "type": "text",
          "analyzer": "ik_smart"
        },
        "location": {
          "type": "geo_point"
        },
        "houses": {
          "type": "integer"
        },
        "built_year": {
          "type": "integer"
        },
        "parkings": {
          "type": "integer"
        },
        "volume": {
          "type": "float"
        },
        "greening": {
          "type": "float"
        },
        "producer": {
          "type": "text",
          "analyzer": "ik_smart"
        },
        "school": {
          "type": "text",
          "analyzer": "ik_smart"
        },
        "info": {
          "type": "text",
          "analyzer": "ik_smart"
        }
      }
    }
  }
}
```

应用场景：

- 每天自动创建 `logstash-village-YYYY.MM.dd`，但字段结构保持一致。
- 避免动态映射把数字识别成字符串。

## 15. Kibana 数据分析

### 15.1 创建索引模式

操作路径：

```text
Stack Management -> Data Views -> Create data view
```

示例：

```text
logstash-village-*
```

应用场景：

- 让 Kibana 识别一组按天生成的 ES 索引。
- 在 Discover 中搜索和过滤数据。

### 15.2 Discover 查看数据

应用场景：

- 检查 Logstash 数据是否写入。
- 快速过滤某省、某市、某时间范围的数据。

常见过滤条件：

```text
province : "北京市"
built_year >= 2020
school : *
```

### 15.3 柱状图

目标：按省份统计平均住户数。

配置思路：

| 配置 | 字段 |
| --- | --- |
| 横轴 | `province` terms |
| 纵轴 | `houses` average |

应用场景：

- 各省小区平均户数对比。
- 各品牌平均价格对比。

### 15.4 折线图

目标：按年份统计新增建筑面积走势。

配置思路：

| 配置 | 字段 |
| --- | --- |
| 横轴 | `built_year` |
| 纵轴 | `floorage` sum |

应用场景：

- 每年新增小区面积走势。
- 每天日志量、订单量、访问量趋势。

### 15.5 树状图

目标：按省份和城市展示平均绿化率。

配置思路：

| 配置 | 字段 |
| --- | --- |
| 第一层分组 | `province` |
| 第二层分组 | `city` |
| 大小指标 | `greening` average |

应用场景：

- 多层分类占比。
- 省市维度数据对比。

### 15.6 地图

目标：展示小区地理位置。

前提：

- ES 中存在 `geo_point` 类型字段，例如 `location`。

应用场景：

- 门店地图。
- 小区分布图。
- 设备、车辆、事件位置展示。

## 16. 学习练习路线

### 16.1 第一阶段：基础 API

练习内容：

1. 创建索引。
2. 写入 5 条商品文档。
3. 使用 `match_all`、`term`、`match` 查询。
4. 修改和删除文档。

推荐练习索引：

```http
PUT products_practice
{
  "mappings": {
    "properties": {
      "name": {
        "type": "text",
        "analyzer": "ik_smart"
      },
      "brand": {
        "type": "keyword"
      },
      "category": {
        "type": "keyword"
      },
      "price": {
        "type": "scaled_float",
        "scaling_factor": 100
      },
      "created_at": {
        "type": "date",
        "format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd"
      }
    }
  }
}
```

### 16.2 第二阶段：Mapping 和分词

练习内容：

1. 分别用 `keyword` 和 `text` 存储同一个字段。
2. 测试 `term` 和 `match` 查询差异。
3. 使用 `_analyze` 对比 `ik_smart` 和 `ik_max_word`。

示例：

```http
POST _analyze
{
  "analyzer": "ik_smart",
  "text": "北京市昌平区回龙观街道"
}
```

### 16.3 第三阶段：组合查询

练习内容：

1. 用 `bool` 组合关键词、分类、价格范围。
2. 用 `_source` 控制返回字段。
3. 用 `sort` 和分页实现列表页。

示例：

```http
GET products_practice/_search
{
  "_source": ["name", "brand", "price"],
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "name": "手机"
          }
        }
      ],
      "filter": [
        {
          "term": {
            "brand": "小米"
          }
        },
        {
          "range": {
            "price": {
              "gte": 1000,
              "lte": 5000
            }
          }
        }
      ]
    }
  },
  "sort": [
    {
      "price": "asc"
    }
  ]
}
```

### 16.4 第四阶段：聚合分析

练习内容：

1. 按品牌统计商品数量。
2. 按分类统计平均价格。
3. 找出销售额最高的分类。

示例：

```http
GET products_practice/_search
{
  "size": 0,
  "aggs": {
    "brand_count": {
      "terms": {
        "field": "brand"
      }
    },
    "avg_price_by_category": {
      "terms": {
        "field": "category"
      },
      "aggs": {
        "avg_price": {
          "avg": {
            "field": "price"
          }
        }
      }
    }
  }
}
```

### 16.5 第五阶段：集群与同步

练习内容：

1. 创建不同分片和副本数量的索引。
2. 用 `_cat/shards` 观察分片分布。
3. 暂停一个 Docker 节点，观察集群健康状态。
4. 使用 Logstash 或程序批量写入数据。

观察命令：

```http
GET _cluster/health
GET _cat/nodes?v
GET _cat/shards?v
GET _cat/allocation?v
```

## 常用命令速查

```http
# 集群健康
GET _cluster/health

# 节点列表
GET _cat/nodes?v

# 索引列表
GET _cat/indices?v

# 分片列表
GET _cat/shards?v

# 创建索引
PUT my_index

# 查看索引
GET my_index

# 删除索引
DELETE my_index

# 测试分词
POST _analyze
{
  "analyzer": "standard",
  "text": "hello elasticsearch"
}

# 查询
GET my_index/_search
{
  "query": {
    "match_all": {}
  }
}
```

## 学习重点总结

| 重点 | 必须掌握的点 |
| --- | --- |
| 倒排索引 | 为什么 ES 全文检索快 |
| Mapping | `text` 和 `keyword` 的区别 |
| 分词器 | 建索引和搜索时都会影响结果 |
| Query DSL | `match`、`term`、`bool`、`range` |
| 聚合 | 指标聚合、分桶聚合、管道聚合 |
| 分片副本 | 分片负责扩展，副本负责高可用和查询吞吐 |
| ELK | Logstash 采集清洗，ES 存储搜索，Kibana 可视化 |

