---
layout: post
book: true
background-image: http://ot1cc1u9t.bkt.clouddn.com/17-7-16/91630214.jpg
category: 工具
title: ElasticSearch简单使用
tags:
- elasticsearch
- Lucene
- 全文搜索
---

ElasticSearch
===
elasticsearch 是使用 Java 编写的，并且采用了 Lucene 来实现索引与搜索的功能。在使用它做全文搜索时，只需要使用简单流畅的 RESTful API 即可，并不需要了解 Lucene 背后复杂的的运行原理。

它不仅可以实现全文搜索功能，还可以完成以下工作:

1. 分布式实时文档存储，并将每一个字段都编入索引，使其可以被搜索。
2. 分布式实时分析与搜索引擎。
3. 可以扩展到上百台服务器，处理PB级别的结构化或非结构化数据。

和 Solr 的区别有：

1. Solr建立索引时候，搜索效率下降，实时搜索效率不高
2. Solr利用Zookeeper进行分布式管理，而ES自身带有分布式协调管理功能
3. Solr支持更多格式的数据，比如JSON、XML、CSV，而ES仅支持json文件格式
4. Solr官方提供的功能更多，而ES本身更注重于核心功能，高级功能多有第三方插件提供
5. Solr在传统的搜索应用中表现好于ES，但在处理实时搜索应用时效率明显低于ES
6. 随着数据量的增加，Solr的搜索效率会变得更低，而ES却没有明显的变化

安装
---

***JAVA 环境安装***

要先安装 JAVA 环境。下载地址: http://www.oracle.com/technetwork/cn/java/javase/downloads/index-jsp-138363-zhs.html

这里我选择的是 java8,下载地址是: http://www.oracle.com/technetwork/cn/java/javase/downloads/jdk8-downloads-2133151-zhs.html

下载的是 jdk-8u172-linux-x64.tar.gz

```
tar -zxf jdk-8u172-linux-x64.tar.gz
mv jdk1.8.0_172/ /usr/local/jdk1.8
```

修改 /etc/profile，在最后添加以下配置

```
export JAVA_HOME=/usr/local/jdk1.8
export CLASSPATH=$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:.
export PATH=$JAVA_HOME/bin:$PATH
```

使文件生效: ```source /etc/profile```,完成后查看JAVA版本: ```java -version```
输出如下:

```
java version "1.8.0_172"
Java(TM) SE Runtime Environment (build 1.8.0_172-b11)
Java HotSpot(TM) 64-Bit Server VM (build 25.172-b11, mixed mode)
```

说明安装成功。

***elasticsearch 安装***

下载地址是: https://www.elastic.co/downloads/past-releases

这里选择的是 6.2.4 版本: https://www.elastic.co/downloads/past-releases/elasticsearch-6-2-4s

```
groupadd elsearch
useradd elsearch -g elsearch -p ES

wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.2.4.tar.gz
tar -zxf ES-6.2.4.tar.gz
mv ES-6.2.4 /usr/local/ES

chown -R elsearch:elsearch /usr/local/elasticsearch

su elsearch
/usr/local/elasticsearch/bin/elasticsearch
```

运行成功后，会开启 9200， 9300 端口的监听。可以在本机执行 ```curl http://localhost:9200/``` 查看内容：

```
{
  "name" : "9c8KDQF",
  "cluster_name" : "ES",
  "cluster_uuid" : "fII7r5BAQzWMnGrbw8pyMQ",
  "version" : {
    "number" : "6.2.4",
    "build_hash" : "ccec39f",
    "build_date" : "2018-04-12T20:37:28.497551Z",
    "build_snapshot" : false,
    "lucene_version" : "7.2.1",
    "minimum_wire_compatibility_version" : "5.6.0",
    "minimum_index_compatibility_version" : "5.0.0"
  },
  "tagline" : "You Know, for Search"
}
```

默认的时候，9200 端口是绑定在 127.0.0.1 的，无法在远程机器上访问。可以修改 /usr/local/elasticsearch/conf/elasticsearch.yml:

```
network.host: 0.0.0.0
http.port: 9200
```

启动时提示错误:

```
bound or publishing to a non-loopback address, enforcing bootstrap checks
node validation exception
[2] bootstrap checks failed
[1]: max file descriptors [65535] for elasticsearch process is too low, increase to at least [65536]
[2]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
```
更改 /etc/security/limits.conf， 在最后加上: 

```
* soft nofile 65536
* hard nofile 131072
* soft nproc 2048
* hard nproc 4096
```

使用 ```ulimit -n```可以查看该值。

执行 ```sysctl -w vm.max_map_count=655360```解决 max virtual memory areas vm.max_map_count 的问题。

如果出现错误:

```
java.lang.RuntimeException: can not run elasticsearch as root
```

可以新建一个用户，然后以它来执行。我们的步骤中已经新建了，所以没问题。解决所有问题后，可以在远程访问了：http://[服务器IP]:9200

目前只是安装了单机版的，实际上我们可以根据需要安装集群。

***管理端 Kibana 安装***

```
wget https://artifacts.elastic.co/downloads/kibana/kibana-6.2.4-linux-x86_64.tar.gz

tar -zxf kibana-6.2.4-linux-x86_64.tar.gz
mv kibana-6.2.4-linux-x86_64 /usr/local/kibana

```

启动:

```
/usr/local/kibana/bin/kibana
```

成功后会监听 5601 端口。同样，它会默认监听 127.0.0.1，无法在外网访问。需要修改它的配置文件 /usr/local/kibana/config/kibana.yml:

```
server.port: 5601
server.host: "0.0.0.0"
```

启动后，可以在外网通过 http://[服务器IP]:5601/ 来查看图形管理界面。

简单试用
---
对数据进行管理，可以通过SDK，RESTful API 甚至 curl 也可以。所以很灵活。如果使用 JAVA 的SDK，需要确保SDK的版本和服务端版本一致，以保证兼容。如果想要进行多语言的通用，可以使用 RESTful 的 API，它使用的是 json 格式。

```
curl -XGET 'http://localhost:9200/_count?pretty' 

返回内容:

{
  "count" : 0,
  "_shards" : {
    "total" : 0,
    "successful" : 0,
    "skipped" : 0,
    "failed" : 0
  }
}

```

在 ES 中，存储数据的行为就叫做 索引(indexing) ，但是在我们索引数据前，我们需要决定将数据存储在哪里。在 ES 中，文档属于一种 类型(type)，各种各样的类型存在于一个 索引 中。

一个 ES 集群可以包含多个 索引（数据库），也就是说其中包含了很多 类型（表）。这些类型中包含了很多的 文档（行），然后每个文档中又包含了很多的 字段（列）。

***示例***

为 megacorp 公司的员工档案创建索引。这样每个文档都代表着一个员工。为了创建员工名单，我们需要进行如下操作:

1. 为每一个员工的 文档 创建索引，每个 文档 都包含了一个员工的所有信息。(相当于表中的一行数据)
2. 每个文档都会被标记为 employee 类型。(相当于 表 employee)
3. 这种类型将存活在 megacorp 这个 索引 中。(相当于 megacorp 库)
4. 这个索引将会存储在 ES 的集群中。

在实际操作中，我们可以通过:

```
curl -XPUT -H "Content-Type: application/json" http://localhost:9200/megacorp/employee/1?pretty -d '
{
    "first_name" : "John",
    "last_name" :  "Smith",
    "age" :        25,
    "about" :      "I love to go rock climbing",
    "interests": [ "sports", "music" ]
}
'
```
这个简单的命令来实现。返回内容：

```
{
  "_index" : "megacorp",
  "_type" : "employee",
  "_id" : "1",
  "_version" : 4,
  "result" : "updated",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 3,
  "_primary_term" : 1
}
```

这时候查看文档数：

```
[root@sunvipyu ~]# curl -XGET 'http://localhost:9200/_count?pretty'
{
  "count" : 1,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  }
}
```

可以尝试多插入几条数据;

```
curl -XPUT -H "Content-Type: application/json" http://localhost:9200/megacorp/employee/2?pretty -d '
{
    "first_name" :  "Jane",
    "last_name" :   "Smith",
    "age" :         32,
    "about" :       "I like to collect rock albums",
    "interests":  [ "music" ]
}
'

curl -XPUT -H "Content-Type: application/json" http://localhost:9200/megacorp/employee/3?pretty -d '
{
    "first_name" :  "Douglas",
    "last_name" :   "Fir",
    "age" :         35,
    "about":        "I like to build cabinets",
    "interests":  [ "forestry" ]
}
'
```

如果想查询某个用户的信息，可以:

```
curl -XGET -H "Content-Type: application/json" http://localhost:9200/megacorp/employee/1?pretty
```

返回结果:

```
{
  "_index" : "megacorp",
  "_type" : "employee",
  "_id" : "1",
  "_version" : 7,
  "found" : true,
  "_source" : {
    "first_name" : "John",
    "last_name" : "Smith",
    "age" : 25,
    "about" : "I love to go rock climbing",
    "interests" : [
      "sports",
      "music"
    ]
  }
}
```

这里只是将前面的 PUT 方式改为了 GET，同理，如果将请求方式改为 DELETE 就表示删除文档。HEAD 表示查询文档是否存在。

如果要进行一些条件查询，就需要用 _search 命令。它会默认返回前 10 个文档。如：

```
curl -XGET -H "Content-Type: application/json" http://localhost:9200/_search?pretty
```

返回内容是:

```
{
  "took" : 20,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 3,
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "megacorp",
        "_type" : "employee",
        "_id" : "2",
        "_score" : 1.0,
        "_source" : {
          "first_name" : "Jane",
          "last_name" : "Smith",
          "age" : 32,
          "about" : "I like to collect rock albums",
          "interests" : [
            "music"
          ]
        }
      },
      {
        "_index" : "megacorp",
        "_type" : "employee",
        "_id" : "1",
        "_score" : 1.0,
        "_source" : {
          "first_name" : "John",
          "last_name" : "Smith",
          "age" : 25,
          "about" : "I love to go rock climbing",
          "interests" : [
            "sports",
            "music"
          ]
        }
      },
      {
        "_index" : "megacorp",
        "_type" : "employee",
        "_id" : "3",
        "_score" : 1.0,
        "_source" : {
          "first_name" : "Douglas",
          "last_name" : "Fir",
          "age" : 35,
          "about" : "I like to build cabinets",
          "interests" : [
            "forestry"
          ]
        }
      }
    ]
  }
}
```

可以看到，结果里有总的结果数 total 及各个文档的详细信息。我们还可以指定具体的过滤条件，如：

```
curl -XGET -H "Content-Type: application/json" http://localhost:9200/_search?q=last_name:Smith
```

如果还有许多查询条件一些配合，还可以用更灵活的方式，如：

```
curl -XGET -H "Content-Type: application/json" http://localhost:9200/_search -d '
{
    "query" : {
        "match" : {
            "last_name" : "Smith"
        }
    }
}
'
```

搜索结果还可以进行高亮关键词以及诸如同义词，统计汇总等一系列功能。

数据操作详细操作
---
对每个文档（每条信息），信息都处理成 json 格式的对象。键是一个字段或者属性的名字，值可以是一个字符串、数字、布尔值、对象、数组或者是其他的特殊类型，比如代表日期的字符串或者代表地理位置的对象。如：

```
{
    "name":         "John Smith",
    "age":          42,
    "confirmed":    true,
    "join_date":    "2014-06-01",
    "home": {
        "lat":      51.5,
        "lon":      0.1
    },
    "accounts": [
        {
            "type": "facebook",
            "id":   "johnsmith"
        },
        {
            "type": "twitter",
            "id":   "johnsmith"
        }
    ]
}
```

一个文档除了包括我们给它设置的信息外，还包括一些元信息（ES自我管理的一些信息）。有三个是必须存在的：```_index```, ```_type```, ```_id```。

```_index``` 表示索名称，相当于是数据库名。

```_type``` 表示索引类型，相当于数据表的名称。

```_id``` 是文档的唯一编号。我们可以自己提交 个唯一 ```_id```，也可以让 ES帮我们生成。

通过 /_index/_type/_id，可以确认文档的唯一性。定位到某个文档上。

***索引文档***

我们通过API将文档添加至索引。文档被存储并可搜索。我们可以通过 PUT 操作提供相应的数据及用来确定文档唯一性的元数据，也可以让 ES 帮我们维护一个唯一的ID(通常是用 POST)。如：

1.自己维护唯一ID
```
curl -XPUT -H "Content-Type: application/json" http://localhost:9200/megacorp/employee/5?pretty -d '
{
    "first_name" :  "Test",
    "last_name" :   "Some",
    "age" :         32,
    "about" :       "Test API",
    "interests":  [ "programe" ]
}
'
```

返回内容:

```
{
  "_index" : "megacorp",
  "_type" : "employee",
  "_id" : "5",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 0,
  "_primary_term" : 1
}
```

2.让 ES 自动生成唯一ID

```
curl -XPOST -H "Content-Type: application/json" http://localhost:9200/megacorp/employee/?pretty -d '
{
    "first_name" :  "AutoIncress",
    "last_name" :   "AutoName",
    "age" :         32,
    "about" :       "Test Id",
    "interests":  [ "programe" ]
}
'
```

返回内容:

```
{
  "_index" : "megacorp",
  "_type" : "employee",
  "_id" : "k50XOWMBd_jrJmg2FhIb",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 1,
  "_primary_term" : 1
}
```

从返回内容可以看到 ```_id```的值。

>自生成ID是由22个字母组成的，安全 universally unique identifiers 或者被称为UUIDs。

***访问文档***

要从 ES 中获取文档，我们需要使用同样的 ```_index```，```_type```以及 ```_id```但是不同的HTTP请求变成GET：

```
curl -XGET -H "Content-Type: application/json" http://localhost:9200/megacorp/employee/k50XOWMBd_jrJmg2FhIb?pretty
```

返回内容:

```
{
  "_index" : "megacorp",
  "_type" : "employee",
  "_id" : "k50XOWMBd_jrJmg2FhIb",
  "_version" : 1,
  "found" : true,
  "_source" : {
    "first_name" : "AutoIncress",
    "last_name" : "AutoName",
    "age" : 32,
    "about" : "Test Id",
    "interests" : [
      "programe"
    ]
  }
}
```

>pretty 参数是为了让返回的 json 格式显示的更容易阅读。

如果只想搜索文档中的某一些字段，不是获得所有信息，可以：

```
curl -XGET -H "Content-Type: application/json" http://localhost:9200/megacorp/employee/k50XOWMBd_jrJmg2FhIb?_source=first_name,age
```

如果只想获得我们提交的数据，不想返回元数据，可以：

```
curl -XGET -H "Content-Type: application/json" http://localhost:9200/megacorp/employee/k50XOWMBd_jrJmg2FhIb/_source
```

返回内容：

```
{
    "first_name" :  "AutoIncress",
    "last_name" :   "AutoName",
    "age" :         32,
    "about" :       "Test Id",
    "interests":  [ "programe" ]
}
```

***检测文档是否存在***

如果确实想检查一下文档是否存在，你可以试用HEAD来替代GET方法，这样就是会返回HTTP头文件。如:

```
curl -i -XHEAD -H "Content-Type: application/json" http://localhost:9200/megacorp/employee/k50XOWMBd_jrJmg2FhIb
```

返回内容:

```
HTTP/1.1 200 OK
content-type: application/json; charset=UTF-8
content-length: 266
```

如果文档不存在，会返回 404:

```
HTTP/1.1 404 Not Found
content-type: application/json; charset=UTF-8
content-length: 85
```

***更新文档***

文档添加到索引后，如果我们数据修改了，可以重新提交：

```
curl -XPUT -H "Content-Type: application/json" http://localhost:9200/megacorp/employee/6?pretty -d '
{
    "first_name" :  "Test Modify",
    "last_name" :   "Some Test",
    "age" :         32,
    "about" :       "Test API Modify",
    "interests":  [ "programe" ]
}
'
```

这时候可以先用 GET 看一下索引的内容:

```
curl -XGET -H "Content-Type: application/json" http://localhost:9200/megacorp/employee/6?pretty

{
  "_index" : "megacorp",
  "_type" : "employee",
  "_id" : "6",
  "_version" : 1,
  "found" : true,
  "_source" : {
    "first_name" : "Test Modify",
    "last_name" : "Some Test",
    "age" : 32,
    "about" : "Test API Modify",
    "interests" : [
      "programe"
    ]
  }
}
```

然后再执行一次 PUT:

```
curl -XPUT -H "Content-Type: application/json" http://localhost:9200/megacorp/employee/6?pretty -d '
{
    "first_name" :  "Test Modify",
    "last_name" :   "Some Test",
    "age" :         33,
    "about" :       "Test API Modify Has Done",
    "interests":  [ "programe" ]
}
'
```

现在再 GET 一下发现内容变了:

```
curl -XGET -H "Content-Type: application/json" http://localhost:9200/megacorp/employee/6?pretty

{
  "_index" : "megacorp",
  "_type" : "employee",
  "_id" : "6",
  "_version" : 2,
  "found" : true,
  "_source" : {
    "first_name" : "Test Modify",
    "last_name" : "Some Test",
    "age" : 33,
    "about" : "Test API Modify Has Done",
    "interests" : [
      "programe"
    ]
  }
}
```

而且，元数据 ```_version``` 从之前的 1 累加为 2。这里的 ```_version``` 是为了解决 ES 中类似锁机制而维护的一个元数据。

>当使用索引API来更新一个文档时，我们先找到了原始文档，然后修改它，最后一次性地将整个新文档进行再次索引处理。涉及到的是多次操作。
>
>在内部，ES 已经将旧文档标记为删除并且添加了新的文档。旧的文档并不会立即消失，但是你也无法访问他。ES 会在你继续添加更多数据的时候在后台清理已经删除的文件。

***创建文档***

我们可以通过 /PUT 操作进行文档的索引，同时存储新文档。而且，当我们使用自定义的 ```_id``` 时，如果我们多次 PUT 操作传递同一个 ```_id``` 时，它会覆盖之前的数据。如果我们让 ES 自己维护ID，我们可以使用 POST 提交。

但是，如果我想又想要用 PUT 方式，使用自己的ID，又不想数据被覆盖，当数据已经存在时，拒绝请求。可以在 PUT时添加 op_tpe 参数，如：

```
curl -XPUT -H "Content-Type: application/json" http://localhost:9200/megacorp/employee/6?op_type=create -d '
{
    "first_name" :  "Test Modify",
    "last_name" :   "Some Test",
    "age" :         33,
    "about" :       "Test API Modify Has Done",
    "interests":  [ "programe" ]
}
'

{"error":{"root_cause":[{"type":"version_conflict_engine_exception","reason":"[employee][6]: version conflict, document already exists (current version [2])","index_uuid":"y-xZR7W9QeeFwpBVTsGBCg","shard":"2","index":"megacorp"}],"type":"version_conflict_engine_exception","reason":"[employee][6]: version conflict, document already exists (current version [2])","index_uuid":"y-xZR7W9QeeFwpBVTsGBCg","shard":"2","index":"megacorp"},"status":409}
```

返回了数据已存在的错误，状态码是 409。或者：

```
curl -XPUT -H "Content-Type: application/json" http://localhost:9200/megacorp/employee/6/_create -d '
{
    "first_name" :  "Test Modify",
    "last_name" :   "Some Test",
    "age" :         33,
    "about" :       "Test API Modify Has Done",
    "interests":  [ "programe" ]
}
'
```

如果创建成功，会返回常见的元数据及 201 状态:

```
curl -i -XPUT -H "Content-Type: application/json" http://localhost:9200/megacorp/employee/7/_create?pretty -d '
{
    "first_name" :  "Test PUT",
    "last_name" :   "Some Test",
    "age" :         31,
    "about" :       "Test API PUT Has Done",
    "interests":  [ "programe" ]
}
'

HTTP/1.1 201 Created
Location: /megacorp/employee/7
content-type: application/json; charset=UTF-8
content-length: 227

{
  "_index" : "megacorp",
  "_type" : "employee",
  "_id" : "7",
  "_version" : 3,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 9,
  "_primary_term" : 1
}
```

***删除文档***

删除和获取文档基本上一样，只不过需要把 GET 换成 DELETE 如：

```
curl -i -XDELETE -H "Content-Type: application/json" http://localhost:9200/megacorp/employee/7?pretty
```

如果文档不存在，会返回 404 状态码及一些提示信息，如：

```
curl -i -XDELETE -H "Content-Type: application/json" http://localhost:9200/megacorp/employee/77?pretty
HTTP/1.1 404 Not Found
content-type: application/json; charset=UTF-8
content-length: 229

{
  "_index" : "megacorp",
  "_type" : "employee",
  "_id" : "7",
  "_version" : 2,
  "result" : "not_found",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 15,
  "_primary_term" : 1
}
```
>尽管文档并不存在（"found"值为false），但是```_version```的数值仍然增加了,变为 2。这个就是内部管理的一部分，它保证了我们在多个节点间的不同操作的顺序都被正确标记了。

如果成功则返回 200 状态码及一些信息:

```
HTTP/1.1 200 OK
content-type: application/json; charset=UTF-8
content-length: 226

{
  "_index" : "megacorp",
  "_type" : "employee",
  "_id" : "6",
  "_version" : 3,
  "result" : "deleted",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 3,
  "_primary_term" : 1
}
```

>删除一个文档也不会立即生效，它只是被标记成已删除。ES 将会在你之后添加更多索引的时候才会在后台进行删除内容的清理

***多版本控制***

当我们更新文档时，会先 GET 到文档，然后修改它，最后一次性将整个新文档进行索引处理。

ES 会根据请求发出的顺序来选择出最新的一个文档进行保存。但是，如果你修改文档的同时其他人也发出了指令，那么他们的修改将会丢失。诸如电商系统中的库存数量的存储。特别是在集群部署的时候要注意。

通常，我们的基础数据都是存在关系数据库中，ES 只是用来进行提供搜索功能。在数据库内容变更时就会更新 ES 里的值，这时候如果有多进程并发，就可能出现同时对一个值进行变更的冲突问题。

ES 使用 ```_version```元数据来确保所有的变更操作都被正确排序，可以参考 MySQL 里 innodb 事务的 mvcc 策略。

比如，有一条数据当前的 ```_version``` 的值是 1。我们提交变更时，ES 内部的操作顺序是先 GET，得到所有数据及 ```_version```的值 1。然后再执行修改，再执行索引。如果在我们执行索引前，有另外的程序已经对文档修改了，它此时的版本值是 2。这时候我们的索引程序就会报错，提示当前版本已经是 2，而我们还要修改版本 1 的内容。并返回 409 的错误码。

有时候，我们需要进行冲突后的重试。可以在API上加上 ```retry_on_conflict```来实现，如：

```
curl -i -XPOST -H "Content-Type: application/json" http://localhost:9200/megacorp/employee/2/_update?retry_on_conflict=5 -d '{"doc" : {"school":"someone"}}'
```

上面表明API会重试 5 次。

***局部更新***

我们要更新一个文档，通过 PUT 进行重新索引。它会先 GET ，然后修改，再索引。除此之外，还可以通过 UPDATE 来做部分更新，它的流程和 PUT 一样，只不过它的操作会在一个片中完成，可以节省多次请求的开销。比如，我们要给现有数据添加新的字段，UPDATE 就很合适。如：

```
curl -i -XPOST -H "Content-Type: application/json" http://localhost:9200/megacorp/employee/2/_update?pretty -d '{"doc" : {"school":"someone"}}'

HTTP/1.1 200 OK
content-type: application/json; charset=UTF-8
content-length: 226

{
  "_index" : "megacorp",
  "_type" : "employee",
  "_id" : "2",
  "_version" : 2,
  "result" : "updated",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 4,
  "_primary_term" : 1
}
```

再获取数据就可以看到我们新添加的 school 的值了:

```
curl -i -XGET -H "Content-Type: application/json" http://localhost:9200/megacorp/employee/2/_source?pretty

HTTP/1.1 200 OK
content-type: application/json; charset=UTF-8
content-length: 171

{
  "first_name" : "Jane",
  "last_name" : "Smith",
  "age" : 32,
  "about" : "I like to collect rock albums",
  "interests" : [
    "music"
  ],
  "school" : "someone"
}
```

***mget 获得多个文档***

mget 需要指定多个文档的 ```_index, _type, _id```，哪怕这些文档不在同一个索引里。如：

```
curl -XGET -H "Content-Type: application/json" http://localhost:9200/_mget?pretty -d'
{
   "docs" : [
      {
         "_index" : "megacorp",
         "_type" :  "employee",
         "_id" :    2
      },
      {
         "_index" : "megacorp",
         "_type" :  "employee",
         "_id" :    1
      }
   ]
}
'
```

返回内容:

```
{
  "docs" : [
    {
      "_index" : "megacorp",
      "_type" : "employee",
      "_id" : "2",
      "_version" : 2,
      "found" : true,
      "_source" : {
        "first_name" : "Jane",
        "last_name" : "Smith",
        "age" : 32,
        "about" : "I like to collect rock albums",
        "interests" : [
          "music"
        ],
        "school" : "someone"
      }
    },
    {
      "_index" : "megacorp",
      "_type" : "employee",
      "_id" : "1",
      "_version" : 7,
      "found" : true,
      "_source" : {
        "first_name" : "John",
        "last_name" : "Smith",
        "age" : 25,
        "about" : "I love to go rock climbing",
        "interests" : [
          "sports",
          "music"
        ]
      }
    }
  ]
}
```

如果要找的文档都在同一个索引或类别中，可以在 url 中指定这些，如：

```
curl -XGET -H "Content-Type: application/json" http://localhost:9200/megacorp/employee/_mget?pretty -d'
{
   "docs" : [
      {
         "_id" :    2
      },
      {
         "_id" :    1
      }
   ]
}
'
```

甚至进一步简化为:

```
curl -XGET -H "Content-Type: application/json" http://localhost:9200/megacorp/employee/_mget?pretty -d'
{
   "ids" : [ "2", "1"]
}
'
```

>注意：请求多个文档时，如果有某个文档不存在，不会影响其它文档的返回。而且，就算所有文档都没找到。请求返回的HTTP状态码还是 200，因为对于 mget 这个操作来说，它是成功了。

***bulk 批量操作***

mget 可以同时获得多个文档，如果我们想做多个操作打包处理。可以使用 bulk。如：

```
curl -i -XPOST -H "Content-Type: application/json" http://localhost:9200/_bulk?pretty -d '
{ "delete": { "_index": "megacorp", "_type": "employee", "_id": "1" }}
{ "create": { "_index": "megacorp", "_type": "employee", "_id": "9" }}
{ "first_name":    "bulk name" }
{ "update": { "_index": "megacorp", "_type": "employee", "_id": "2", "_retry_on_conflict" : 3} }
{ "doc" : {"first_name" : "bulk update name"} }

'
```

这里，我们为每个子句都分别指定了 ```_index, _type, _id```,如果操作的数据是同一个索引或类型的，我们也可以直接在 url 里指定，如:

```
curl -i -XPOST -H "Content-Type: application/json" http://localhost:9200/megacorp/employee/_bulk?pretty -d '
{ "delete": { "_id": "1" }}
{ "create": { "_id": "9" }}
{ "first_name":    "bulk name" }
'
```

返回内容:

```
HTTP/1.1 200 OK
Warning: 299 Elasticsearch-6.2.4-ccec39f "Deprecated field [_retry_on_conflict] used, expected [retry_on_conflict] instead" "Mon, 07 May 2018 07:15:35 GMT"
content-type: application/json; charset=UTF-8
content-length: 1132

{
  "took" : 119,
  "errors" : false,
  "items" : [
    {
      "delete" : {
        "_index" : "megacorp",
        "_type" : "employee",
        "_id" : "1",
        "_version" : 8,
        "result" : "deleted",
        "_shards" : {
          "total" : 2,
          "successful" : 1,
          "failed" : 0
        },
        "_seq_no" : 16,
        "_primary_term" : 2,
        "status" : 200
      }
    },
    {
      "create" : {
        "_index" : "megacorp",
        "_type" : "employee",
        "_id" : "9",
        "_version" : 1,
        "result" : "created",
        "_shards" : {
          "total" : 2,
          "successful" : 1,
          "failed" : 0
        },
        "_seq_no" : 2,
        "_primary_term" : 2,
        "status" : 201
      }
    },
    {
      "update" : {
        "_index" : "megacorp",
        "_type" : "employee",
        "_id" : "2",
        "_version" : 3,
        "result" : "updated",
        "_shards" : {
          "total" : 2,
          "successful" : 1,
          "failed" : 0
        },
        "_seq_no" : 5,
        "_primary_term" : 2,
        "status" : 200
      }
    }
  ]
}
```

每个子请求都被单独执行。但当有任何一个执行有问题时，结果的 error 都会返回 true。具体错误信息会在各个子句的结果里显示出来。

通常，使用 bulk 是为了提升性能。但，ES运行时使用的内存是有限的，当 bulk 的内容太长时，留给其它功能使用的内存就不够了。所以 bulk 的值不要太多。可以尝试使用 500 (根据每个文档的数据量而定)，然后观测监控数据，如果性能没影响，则可以增加，当增加到性能出现下降时，就可以适当减少一点并稳定。

