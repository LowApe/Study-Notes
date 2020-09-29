# ElasticSearch 简介及发展历史

## 从开源到上市

- Elastic INc - 开源软件/上市公司
- 超过 3.5 亿次下载
- 10万 + 的社区成员
- 7200 + 订阅用户，分布在 100 + 国家
- 云服务  - Elastic，Amazon，阿里巴巴，腾讯



## 几款搜索引擎

- Elasticsearch - 开源分布式搜索分析引擎
	- 近实时 (Near Real Time)
	- 分布式存储/搜索/分析引擎
- Solr (Apache 开源项目)
- Splunk (商业上市公司)



## 起源 - Lucene

- 基于 Java 语言开发的**搜索引擎库类**
- 创建于 1999 年
- Lucene 具有高性能、易扩展的优待你
- Lucene 的局限性：
	- 只能**基于 Java 语言**开发
	- 类库的**接口学习**曲线陡峭
	- 原生并**不支持水平扩展**



## Elasticsearch 的诞生

- 2004年基于 Lucene 开发了 Compass
- 2010年 Shay Banon 重写了 Compass，取名 Elasticsearch 
	- 支持分布式，可水平扩展
	- 降低全文检索的学习曲线，可以被任何编程语言调用

## Elasticsearch 的分布式架构

 ![image.png](http://ww1.sinaimg.cn/large/006rAlqhly1g8l09uvd3gj30is0fa40f.jpg)

- 集群规模可以从单个扩展至数百个节点
- 高可用 & 水平扩展
	- 服务和数据两个维度
- 支持不同的节点类型
	- 支持 Hot & Warm 架构

## 支持多种方式集成接入

- 多种编程语言的类库
	- Java / .NET /Python /Ruby /PHP/Groovy /Perl
- RESTful API v.s Transport API
	- 9200 v.s 9300 (建议使用 RESTful API)
- JDBC & ODBC

## Elasticsearch 的主要功能

- 海量数据的分布式存储以及集群管理

	- 服务与数据的高可用，水平扩展

- 近实时**搜索**，性能卓越

	- 结构化/全文/地理位置/自动完成

- 海量数据的近实时分析

	- **聚合**功能

	

## Elasticsearch 版本与升级

0.4:2010

1.0:2014

2.0:2015

5.0:2016

6.0:2017

7.0:2019

### 新特性 5.x

- Lucene 6.x,性能提升，默认打分机制从 TF-IDF 改为 BM 25
- 支持 Ingest 节点 / Painless Scripting /Completion suggested 支持 / 原生的 Java REST 客户端
- Type 标记成 deprecated ，支持了 Keyword 的类型
- 性能优化
	- 内部引擎移除了避免同一文档并发更新的竞争锁，带来 15 %-20%的性能提升
	- Instant aggregation，支持分片上聚合的缓存
	- 新增了 Profile API

### 新特性 6.x

- Lucene 7.x
- 性功能
	- 跨集群复制(CCR)
	- 索引生命周期管理
	- SQL 的支持
- 更友好的升级及数据迁移
	- 在主要版本之间的迁移更为简化，体验升级
	- 全新的基于操作的数据复制框架，可加快恢复数据
- 性能优化
	- 有效存储稀疏字段的新方法，降低了存储成本
	- 在索引时进行排序，可加快排序的查询性能

### 新特性 7.x

- Lucene 8.0
- 重大改进 - 正式废除单个索引下多 Type 的支持
- Security 功能免费使用
- ECK - Elasticsearch Operator on Kubernetes
- 新功能
	- New Cluster coordination
	- Feature-Complete High Level REST Client
	- Script Score Query
- 性能优化
	- 默认的 Primary Shard 数从 5 改为 1，避免 Over Sharding 
	- 性能优化，更快的 Top K



