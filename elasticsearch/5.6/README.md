> 说明：目前测试仅是本机，如果是不同机器，自选修改相关配置  
> elasticsearch5.6.10 + elasticsearch-head5 + kibana5.6.10   
> plugins: analysis-ik5.6 + analysis-pinyin5.6

## docker及docker-compose安装及说明
请参考: https://yeasy.gitbooks.io/docker_practice/compose


## 安装
```
cd ~/
git clone https://github.com/viky88/docker-lnmp.git
cd docker-lnmp

```
## alias 设置
```
# bash 修改 ~/.bash_profile  
# zsh 修改 ~/.zshrc

# 修改成自己的路径
export DOCKERCOMPOS_EHOME=/Users/viky/project/docker/github/viky88/docker-lnmp
alias godocker_elasticsearch5.6="/usr/local/bin/docker-compose -f $DOCKERCOMPOS_EHOME/docker-compose_elasticsearch5.6.yml up -d"

```
## 如何设置开机启动
```
# 根据系统先设置docker服务为开机启动
# 编辑开机启动文件
# 注意这里不用 sudo，本身是使用 root 运行的
$ sudo vim /etc/rc.local
/usr/local/bin/docker-compose -f /root/docker-lnmp/docker-elasticsearch5.6.yml up -d
# 重启测试
$ sudo reboot
```
## docker环境 快速使用elasticsearch-head插件

- 配置文件中已经配置，安装后可以直接使用
- 还有另外一种更简单安装 chrome 插件的方式:  
https://chrome.google.com/webstore/detail/elasticsearch-head/ffmkiejjmecolpfloofpjologoblkegm?utm_source=chrome-app-launcher-info-dialog

## 中文分词 elasticsearch-analysis-ik 安装
参考： [elasticsearch-analysis-ik github](https://github.com/medcl/elasticsearch-analysis-ik)

```
# 对应好版本，不然安装不上(此功能有bug，不用)
docker exec -it elasticsearch5.6_1 /bin/bash
./bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v5.6.10/elasticsearch-analysis-ik-5.6.10.zip

# 本地安装(可将文件先下载到宿主电脑，用docker cp 命令拷贝到容器中)
# 参考1：https://github.com/medcl/elasticsearch-analysis-ik#install
# 参考2：https://blog.csdn.net/u012915455/article/details/78952068

$ docker exec -it elasticsearch5.6_1 /bin/bash
$ cd plugins
$ wget https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v5.6.10/elasticsearch-analysis-ik-5.6.10.zip
$ unzip elasticsearch-analysis-ik-5.6.10.zip
$ rm elasticsearch-analysis-ik-5.6.10.zip
```

## 拼音 elasticsearch-analysis-pinyin 安装
参考：  [elasticsearch-analysis-pinyin github](https://github.com/medcl/elasticsearch-analysis-pinyin)

```
# 对应好版本，不然安装不上
$ docker exec -it elasticsearch5.6_1 /bin/bash
$ cd plugins
$ wget https://github.com/medcl/elasticsearch-analysis-pinyin/releases/download/v5.6.10/elasticsearch-analysis-pinyin-5.6.10.zip
$ unzip elasticsearch-analysis-pinyin-5.6.10.zip
$ rm elasticsearch-analysis-pinyin-5.6.10.zip
```
安装完插件需要，重新启动 Elastic，就会自动加载新安装的插件

```
# 重启docker
docker restart elasticsearch5.6_1
docker restart elasticsearch5.6_2

# 查看安装的插件
docker exec -it elasticsearch5.6_1 bash ./bin/elasticsearch-plugin list

# 结果
analysis-ik
analysis-pinyin
```

新建一个 Index，指定需要分词的字段。这一步根据数据结构而异，下面的命令只针对本文。基本上，凡是需要搜索的中文字段，都要单独设置一下
```
curl -X PUT 'localhost:9200/accounts' -d '
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
}'
```

上面代码中，首先新建一个名称为accounts的 Index，里面有一个名称为person的 Type。person有三个字段。
```
    user
    title
    desc
```
这三个字段都是中文，而且类型都是文本（text），所以需要指定中文分词器，不能使用默认的英文分词器。

Elastic 的分词器称为 analyzer。我们对每个字段指定分词器。
```
"user": {
  "type": "text",
  "analyzer": "ik_max_word",
  "search_analyzer": "ik_max_word"
}
```
上面代码中，analyzer是字段文本的分词器，search_analyzer是搜索词的分词器。ik_max_word分词器是插件ik提供的，可以对文本进行最大数量的分词。

## 测试

### 增加数据
 向指定的 /Index/Type 发送 PUT 请求，就可以在 Index 里面新增一条记录。   
 比如，向/accounts/person发送请求，就可以新增一条人员记录。

```

curl -X PUT 'localhost:9200/accounts/person/1' -d '
{
  "user": "张三",
  "title": "工程师",
  "desc": "数据库管理"
}' 
```
 服务器返回的 JSON 对象，会给出 Index、Type、Id、Version 等信息。
```
{
  "_index":"accounts",
  "_type":"person",
  "_id":"1",
  "_version":1,
  "result":"created",
  "_shards":{"total":2,"successful":1,"failed":0},
  "created":true
}%
```

如果你仔细看，会发现请求路径是/accounts/person/1，最后的1是该条记录的 Id。它不一定是数字，任意字符串（比如abc）都可以。

新增记录的时候，也可以不指定 Id，这时要改成 POST 请求。
```
curl -X POST 'localhost:9200/accounts/person' -d '
{
  "user": "李四",
  "title": "工程师",
  "desc": "系统管理"
}'
```
上面代码中，向/accounts/person发出一个 POST 请求，添加一个记录。  
这时，服务器返回的 JSON 对象里面，_id字段就是一个随机字符串 
```
{
  "_index":"accounts",
  "_type":"person",
  "_id":"AWUi_xE0RtXeSKsD88GU",
  "_version":1,
  "result":"created",
  "_shards":{"total":2,"successful":1,"failed":0},
  "created":true
}%
```
注意，如果没有先创建 Index（这个例子是accounts），直接执行上面的命令，Elastic 也不会报错，而是直接生成指定的 Index。所以，打字的时候要小心，不要写错 Index 的名称。

### 查看记录

向/Index/Type/Id发出 GET 请求，就可以查看这条记录
```
curl 'localhost:9200/accounts/person/1?pretty=true'
```
上面代码请求查看/accounts/person/1这条记录，URL 的参数pretty=true表示以易读的格式返回。

返回的数据中，found字段表示查询成功，_source字段返回原始记录。
```
{
  "_index" : "accounts",
  "_type" : "person",
  "_id" : "1",
  "_version" : 1,
  "found" : true,
  "_source" : {
    "user" : "张三",
    "title" : "工程师",
    "desc" : "数据库管理"
  }
}
```
如果 Id 不正确，就查不到数据，found字段就是false。  

```
curl 'localhost:9200/weather/beijing/abc?pretty=true'
```
返回
```
{
  "error" : {
    "root_cause" : [
      {
        "type" : "index_not_found_exception",
        "reason" : "no such index",
        "resource.type" : "index_expression",
        "resource.id" : "weather",
        "index_uuid" : "_na_",
        "index" : "weather"
      }
    ],
    "type" : "index_not_found_exception",
    "reason" : "no such index",
    "resource.type" : "index_expression",
    "resource.id" : "weather",
    "index_uuid" : "_na_",
    "index" : "weather"
  },
  "status" : 404
}
```

### 删除记录

删除记录就是发出 DELETE 请求。
```
curl -X DELETE 'localhost:9200/accounts/person/1'
```

### 更新记录

更新记录就是使用 PUT 请求，重新发送一次数据。

```
curl -X PUT 'localhost:9200/accounts/person/1' -d '
{
    "user" : "张三",
    "title" : "工程师",
    "desc" : "数据库管理，软件开发"
}' 

{
  "_index":"accounts",
  "_type":"person",
  "_id":"1",
  "_version":2,
  "result":"updated",
  "_shards":{"total":2,"successful":1,"failed":0},
  "created":false
}
```
上面代码中，我们将原始数据从"数据库管理"改成"数据库管理，软件开发"。 返回结果里面，有几个字段发生了变化
```
    "_version" : 2,
    "result" : "updated",
    "created" : false
```
可以看到，记录的 Id 没变，但是版本（version）从1变成2，操作类型（result）从created变成updated，created字段变成false，因为这次不是新建记录

## 数据查询

### 返回所有记录

Elastic 的查询非常特别，使用自己的[查询语法](https://www.elastic.co/guide/en/elasticsearch/reference/5.6/query-dsl.html)，要求 GET 请求带有数据体

```
 curl 'localhost:9200/accounts/person/_search'  -d '
{
  "query" : { "match" : { "desc" : "软件" }}
}'
```
上面代码使用 [Match 查询](https://www.elastic.co/guide/en/elasticsearch/reference/5.6/query-dsl-match-query.html)，指定的匹配条件是desc字段里面包含"软件"这个词。返回结果如下：

```
{
    "took": 7,
    "timed_out": false,
    "_shards": {
        "total": 5,
        "successful": 5,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": 1,
        "max_score": 1.122156,
        "hits": [
            {
                "_index": "accounts",
                "_type": "person",
                "_id": "1",
                "_score": 1.122156,
                "_source": {
                    "user": "张三",
                    "title": "工程师",
                    "desc": "数据库管理，软件开发"
                }
            }
        ]
    }
}
```

Elastic 默认一次返回10条结果，可以通过size字段改变这个设置。

```
 curl 'localhost:9200/accounts/person/_search'  -d '
{
  "query" : { "match" : { "desc" : "管理" }},
  "size": 1
}'
````
上面代码指定，每次只返回一条结果。

还可以通过from字段，指定位移。
```
 curl 'localhost:9200/accounts/person/_search'  -d '
{
  "query" : { "match" : { "desc" : "管理" }},
  "from": 1,
  "size": 1
}'
```
上面代码指定，从位置1开始（默认是从位置0开始），只返回一条结果

### 逻辑运算
如果有多个搜索关键字， Elastic 认为它们是or关系。

```
curl 'localhost:9200/accounts/person/_search'  -d '
{
  "query" : { "match" : { "desc" : "软件 系统" }}
}'
```
上面代码搜索的是软件 or 系统。

如果要执行多个关键词的and搜索，必须使用[布尔查询](https://www.elastic.co/guide/en/elasticsearch/reference/5.6/query-dsl-bool-query.html)。

```
 curl 'localhost:9200/accounts/person/_search'  -d '
{
  "query": {
    "bool": {
      "must": [
        { "match": { "desc": "软件" } },
        { "match": { "desc": "系统" } }
      ]
    }
  }
}'
```

---
## 参考：
- [Elasticsearch:5.6](https://www.elastic.co/guide/en/elasticsearch/reference/5.6/getting-started.html)
- [Elasticsearch: 权威指南](https://www.elastic.co/guide/cn/elasticsearch/guide/cn/index.html)
- [全文搜索引擎 Elasticsearch 入门教程](http://www.ruanyifeng.com/blog/2017/08/elasticsearch.html)
- [Elasticsearch搜索服务学习之九——索引管理](https://blog.belonk.com/c/elasticsearch_index_management.html)