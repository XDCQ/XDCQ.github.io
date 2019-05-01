---
layout: post
title: es
description:
image: assets/images/pic03.jpg
---

## elasticsearch

端口：
- 9200 端口：HTTP RESTful 接口的通讯端口
- 9300 端口：TCP 通讯端口，用于集群间节点通信和与 Java 客户端通信的端口

### 概念

逻辑层面：
- Index (索引)：这里的 Index 是名词，一个 Index 就像是传统关系数据库的 Database，它是 Elasticsearch 用来存储数据的逻辑区域
- Type (类型)：文档归属于一种 Type，就像是关系数据库中的一个 Table
- Document (文档)：Elasticsearch 使用 JSON 文档来表示一个对象，就像是关系数据库中一个 Table 中的一行数据
- Field (字段)：每个文档包含多个字段，类似关系数据库中一个 Table 的列


|Elasticsearch|	MySQL|
|-|-|
|Index|	Database|
|Type|	Table|
|Document|	Row|
|Field|	Column|

物理层面：

- Node (节点)：node 是一个运行着的 Elasticsearch 实例，一个 node 就是一个单独的 server
- Cluster (集群)：cluster 是多个 node 的集合
- Shard (分片)：数据分片，一个 index 可能会存在于多个 shard


### 使用

![es使用](https://user-gold-cdn.xitu.io/2017/3/22/af3f526a80422bfdf8b7fa8389a562b0.png?imageView2/0/w/1280/h/960/format/webp/ignore-error/1  "es使用")

- 用 url 表示一个资源，比如 /movie/adventure/1 就表示一个 index 为 movie，type 为 adventure，id 为 1 的 document
- 用 http 方法操作资源，如使用 GET 获取资源，使用 POST、PUT 新增或更新资源，使用 DELETE 删除资源等

#### 增

##### 添加文档

```
curl -i -X PUT "localhost:9200/movie/adventure/1" -d '{"name": "Life of Pi", "actors": ["Suraj", "Irrfan"]}'
```
如果我们的数据没有 id，也可以让 Elasticsearch 自动为我们生成，此时要使用 POST 请求，形式如下：
```
#use  httpie
POST /movie/adventure/
{
    "name": "Life of Pi"
}
```
>为了避免在误操作的情况下，原文档被替换，我们可以使用 _create 这个 API，表示只在文档不存在的情况下才创建新文档（返回 201 Created），如果文档存在则不做任何操作（返回 409 Conflict），命令如下：
```
http put :9200/movie/adventure/1/_create name="Life of Pi"
```
如果文档 id 存在，会返回 409 Conflict。

#### 查
##### 查看文档

```
#use  httpie
 http :9200/movie/adventure/1
```
原始的文档数据存在了 _source 字段中。
我们也可以直接检索出文档的 _source 字段，如下：

```
http :9200/movie/adventure/1/_source
```
##### 检索所有文档

我们可以使用 _search 这个 API 检索出所有的文档，命令如下：
```
http :9200/movie/adventure/_search
```
返回结果如：
```
{
    "_shards": {
        "failed": 0,
        "successful": 5,
        "total": 5
    },
    "hits": {
        "hits": [
            {
                "_id": "1",
                "_index": "movie",
                "_score": 1.0,
                "_source": {
                    "actors": [
                        "Suraj",
                        "Irrfan"
                    ],
                    "name": "Life of Pi"
                },
                "_type": "adventure"
            }
        ],
        "max_score": 1.0,
        "total": 1
    },
    "timed_out": false,
    "took": 299
}
```
可以看到，hits 这个 object 包含了 hits 数组，total 等字段，其中，hits 数组包含了所有的文档，这里只有一个文档，total 表明了文档的数量，默认情况下会返回前 10 个结果。我们也可以设定 From/Size 参数来获取某一范围的文档，可参考这里，比如：
```
http :9200/movie/adventure/_search?from=1&size=5
```
当不指定 from 和 size 时，会使用默认值，其中 from 的默认值是 0，size 的默认值是 10。

##### 检索某些字段

有时候，我们只需检索文档的个别字段，这时可以使用 _source 参数，多个字段可以使用逗号分隔，如下所示：
```
$ http :9200/movie/adventure/1?_source=name
$ http :9200/movie/adventure/1?_source=name,actors
```

##### query string 搜索

query string 搜索以 q=field:value 的形式进行查询，比如查询 name 字段含有 life 的电影：
```
http :9200/movie/adventure/_search?q=name:life
```

##### DSL 搜索

DSL（Domain Specific Language）查询语言，适用于复杂的搜索场景，比如全文搜索。我们可以将上面的 query string 搜索转换为 DSL 搜索，如下：
```
GET /movie/adventure/_search
{
    "query" : {
        "match" : {
            "name" : "life"
        }
    }
}
```

##### 文档是否存在

使用 HEAD 方法查看文档是否存在：
```
$ http head :9200/movie/adventure/1
```
如果文档存在则返回 200，否则返回 404。

#### 改

##### 全部更新
当我们使用 PUT 方法指明文档的 _index, _type 和 _id时，如果 _id 已存在，则新文档会替换旧文档，此时文档的 _version 会增加 1，并且 _created 字段为 false。比如：
```
#use  httpie
 http put :9200/movie/adventure/1 name="Life of Pi"
```
之前添加的actors 这个字段将不存在了，文档的 _version 变成了 2。

>为了避免在误操作的情况下，原文档被替换，我们可以使用 _create 这个 API，表示只在文档不存在的情况下才创建新文档（返回 201 Created），如果文档存在则不做任何操作（返回 409 Conflict），命令如下：
```
http put :9200/movie/adventure/1/_create name="Life of Pi"
```
由于文档 id 存在，会返回 409 Conflict。

##### 局部更新

使用 _update 这个 API。

```
echo '{"doc": {"actors": ["Suraj", "Irrfan"]}}' | http post :9200/movie/adventure/1/_update
```
上面的命令中，我们添加了一个新的字段：actors;
最简单的 update 请求接受一个局部文档参数 doc，它会合并到现有文档中：将对象合并在一起，存在的标量字段被覆盖，新字段被添加。

#### 删

使用 DELETE 方法删除文档：
```
$ http delete :9200/movie/adventure/1
```

#### 总结

- Elasticsearch 通过简单的 RESTful API 来隐藏 Lucene 的复杂性，从而让全文搜索变得简单
- 在创建文档时，我们可以用 POST 方法指定将文档添加到某个 _index/_type 下，来让 Elasticsearch自动生成唯一的 _id；而用 PUT 方法指定将文档的 _index/_type/_id
