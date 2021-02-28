# Elasticsearch


# Elasticsearch

Elasticsearch是一个基于**Lucene**库的搜索引擎。提供了一个分布式，支持多租户的全文搜索引擎，具有HTTP Web接口和无模式JSON文档。

> Lucene库是一套用于全文检索和搜索的开放源码程序库，提供了一个简单却强大的应用程序接口，能够做全文索引和搜索。

## 概述

Elasticsearch 与传统的 SQL数据库的一个明显的不同点是，Elasticsearch 是一个 **非结构化** 的数据库，或者说是一个 **无模式** 的数据库。

Elasticsearch 中数据最重要的三要素当属：**索引**、**类型**、**文档**。

其中索引这个概念非常重要，我们可以粗略地将其类比到传统SQL数据库中的 **数据表**。本文就从 Elasticsearch 的索引映射如何配置开始讲起。

## 基本概念

### Node 与 Cluster

Elastic 本质是一个分布式数据库，允许多台服务器协同工作，每台服务器可运行多个Elastic实例。

单个Elastic实例称为一个Node，一组Node构成一个Cluster（集群）

### Index

Elastic会索引所有字段，经过处理后写入一个反向索引（Inverted Index）。查找数据的时候，直接查找该索引。

因此，Elastic数据管理的顶层单位就叫做Index（索引）。它是一个单个数据库的同义词。每个Index（即数据库）的名字必须是小写。

下面的命令可以查看当前节点的所有 Index。

> ```bash
> $ curl -X GET 'http://localhost:9200/_cat/indices?v'
> ```

### Document

Index里面的单条记录称为Document（文档），许多Document构成一个Index。

Documet 使用JSON格式表示，例：

```json
{
	"user": "djj",
	"age": "21",
	"phone": "1511234xxxx"
}
```

同一index里面的Document，不要求有相同的结构scheme，但是最好保持相同，有助于提高效率。

### Type

Document 可以分组，比如weather这个index里面，可以按城市分组（上海、深圳），也可以按气候（晴天、雨天）。按照虚拟逻辑来进行分组，用来过滤Document。

不同你的Type应该有相似的结构scheme，例
