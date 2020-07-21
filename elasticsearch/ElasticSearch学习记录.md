#  ES学习记录

### 1.ES介绍

> ES就是一个使用Java语言并且基于Lucene编写的搜索引擎框架，他提供了分布式的全文搜索功能，提供了一个基于RESTful风格的WEB接口，官方客户端提供了多种语言的API接口
>
> Lucene:是一个搜索引擎的底层
>
> 分布式：ES主要是为了突出它的横向扩展能力
>
> 全文分词：将一段词语进行分词，并且将分出的单个词语统一的放在一个分词库中，在搜索时，根据关键字去分词库中检索，找到匹配的内容（倒排索引）

##### 1.1 ES和Solr

- Solr查询四数据的时候，速度比ES快，但是数据如果时实时变化的，Solr的查询速度就会下降很多，ES查询效率几乎没有变化
- Solr搭建集群需要依赖ZooKeeper来帮助管理，ES本身支持集群的搭建，不需要第三方的接入
- Solr最开始的时候社区很火爆，但是中文文档比较少，而且ES出来之后，ES社区的火爆成功直线上升，而且中文文档比较健全
- ES对现在的云计算和大数据的支持很好

##### 1.2 倒排索引

倒排索引就是将存放的数据以一定的方式进行分词，并且将分词放到一个单独的分词库中，当用户查询时，将用户的查询关键词也进行分词，然后去分词库中进行匹配，最终得到数据的id标识，然后去数据的存放位置拉取数据

### 2.ES安装

```yml
version: "3.1"
services:
	elasticsearch:
		image: daocloud.io/library/elasticsearch:6.5.4
		restart: always
		container_name: elasticsearch
		ports:
			- 9200:9200
	kibana:
		image: daocloud.io/library/kibana:6.5.4
		restart: always
		container_name: kibana
		ports:
			- 5601:5601
		environment: 
			- elasticsearch_url=http://47.98.122.87:9200
		depends_on:
			- elasticsearch
```

ES安装完成之后，记得安装ik分词器，**且必须与ES版本相同**，且安装成功成功后需要重启

### 3. ES的基本操作

##### 3.1 ES的基本结构

- **索引Index**：类似于数据库中的库(还是有很大区别的) 在ES服务中可以创建多个索引，每个索引默认被分为5个分片，每个分片都至少有一个备份分片，备份分片默认不进行数据检索，当索引压力特别大的时候，备份分片才会帮助检索。

  > 备份分片只会存在于集群的架构下，因为备份分片只会存在另外的服务器中，单服务器情况下，不存在备份分片

- **类型Type**: 类似于数据库中的表，5.x版本中一个索引下可以创建多个类型，6.X版本中只能有一个类型，7.X版版本中没有类型

- **文档Document**: 类似于数据库中的数据记录

- **属性Field**: 类似于数据库表中的字段

##### 3.2 操作ES的RESTful语法

- GET请求：

  http://ip:port/index: 查询索引信息

  http://ip:port/index/type/doc_id: 查询指定的文档信息（简单的查询）

- POST请求：

  http://ip:port/index/type/_search:查询文档，可以在请求体中添加json字符串来代表查询条件(复杂的查询)

  http://ip:port/index/type/doc_id/_update:修改文档，可以在请求体中添加json字符串来代表修改的具体信息

- PUT请求：

  http://ip:port/index：创建一个索引，需要在请求体中指定索引的信息、类型和结构

  http://ip:port/index/type/_mappings: 代表创建索引时，指定索引文档存储的属性信息

- DELETE请求：

  http://ip:port/index：删除索引

  http://ip:port/index/type/doc_id: 删除指定文档

##### 3.3 索引的操作

创建索引（没有mappings信息）

```json
//创建一个person索引，设置5个分片，一个备份
PUT /person
{
  "settings": {
    "number_of_shards": 5,
    "number_of_replicas": 1
  }
}
```

查询索引

```json
GET /person
```

删除

```
DELETE /person
```

##### 3.4 ES中field可以指定的类型

字符串类型：

- text: 一般用于全文检索，将当前的field进行分词
- keyword:这种类型的field不会进行分词，比如设置城市、姓名等

数值类型：

- long
- integer
- short
- byte
- double
- float
- half_float:精度比float小一半
- scaled_float：根据一个long和scaled来表示一个浮点  long-45, scaled-100 -> 0.45

时间类型:

- date:针对时间类型可以指定具体的格式

布尔类型：

- boolean:标识true和false

二进制类型：

- binary :暂时支持Base64 encode string

范围类型：

- long_range:赋值时，无需指定一个具体的值，只需存储一个范围即可，lt,lte,gt,gte
- integer_range: 同上
- double_range: 同上
- float_range：同上
- date_range: 日期范围
- ip_range: ip 范围

经纬度类型:

- geo_point:用来存储经纬度

ip类型：

- ip:可以存储ipv4和ipv6

##### 3.5 创建索引并制定具体结构

```json
PUT /book
{
  "settings": {
      #设置备份数
    "number_of_replicas": 1,
      #设置分片数
    "number_of_shards": 5
  },
    #指定数据结构
  "mappings": {
    #设置Type名称
    "novels":{
    #设置文档的field
      "properties":{
    	#field名称
        "name":{
    		#类型是text
          "type":"text",
    		#设置分词器
          "analyzer":"ik_max_word",
    		#指定当前field可以作为查询条件
          "index":true,
    		#是否需要额外存储
          "store":false
        },
        "author":{
          "type":"keyword"
        },
        "count":{
          "type":"long"
        },
        "on-sale":{
          "type":"date",
            #设置date格式
          "format":"yyyy-MM-dd HH:mm:ss||yyyy-MM-dd"
        },
        "descr":{
          "type":"text",
          "analyzer":"ik_max_word"
        }
      }
    }
  }
}
```

##### 3.6 生成、修改、删除文档

- 生成 

  自动生成id: `POST /index名称/type名称`

  ```json
  #创建文档，自动生成id
  POST /book/novels
  {
    "name":"西游记",
    "author":"吴承恩",
    "count":"123456",
    "on-sale":"1965-02-21",
    "descr":"唐僧，孙悟空，猪八戒，沙和尚，白龙马"
  }
  ```

  手动设置id: `PUT /index名称/type名称/具体的id`

  ```json
  #创建文档，手动设置id
  PUT /book/novels/1
  {
    "name":"红楼梦",
    "author":"曹雪芹",
    "count":"200000",
    "on-sale":"1965-02-21",
    "descr":"林黛玉，贾宝玉，薛宝钗..."
  }
  ```

- 修改

  以覆盖的方式修改：`PUT /index名称/type名称/具体的id`（和手动生成id的创建文档的格式一样）

  ```json
  #修改文档，以覆盖的方式
  PUT /book/novels/1
  {
    "name":"红楼梦",
    "author":"曹雪芹",
    "count":"200000",
    "on-sale":"1965-02-21",
    "descr":"林黛玉，贾宝玉，薛宝钗..."
  }
  ```

  修改指定的field，其他的不变：`POST /index名称/type名称/具体的id/_update`

  ```json
  #修改文档，基于doc方式，只修改设置的field，其他的不变
  POST /book/novels/1/_update
  {
    "doc":{
      "on-sale":"1992-02-21"
    }
  }
  ```

- 删除

  删除指定文档: `DELETE /index名称/type名称/具体的id`

  ```json
  #删除指定文档
  DELETE /book/novels/Pdy3aHIBQ0xEf8HRM6KP
  ```

### 4. PHP操作ES

> PHP操作ES中文文档：https://www.elastic.co/guide/cn/elasticsearch/php/current/_overview.html

### 5.ES的各种查询

##### 5.1 term和terms查询

> **注意：**
>
> **7.x版本之后，term下的字段名后直接跟值，如："name":"张三",**
>
> **6.x版本以及之前，还要使用如下写法"name":{"value":"张三"}**

- **term查询**:term查询是代表完全匹配，**搜索之前不会对搜索的关键词进行分词**，直接对输入的关键词去文档分词库中进行匹配

  <!--注意：只是不对搜索的关键词进行分词，并不是说只能搜索keyword类型，其他类型都可以搜-->

  ```json
  # /index名称/type名称/_search
  POST /search_index/search_type/_search
  {
      #分页，类似于sql中的limit
  	"from":0,
  	"size":5,
  	"query":{
  		"term":{
  			"name":{
  				"value":"张三"
  			}
  		}
  	}
  }
  ```

- **terms查询**：和term查询机制相同，**搜索之前不会对搜索的关键词进行分词**，直接对输入的关键词去文档分词库中进行匹配，

  > terms和term的区别是：针对一个字段包含多个值的时候使用
  >
  > 类似于sql中的where
  >
  > term: where name = 张三
  >
  > terms: where name = 张三 or name = 李四...

  ```json
  POST /search_index/search_type/_search
  {
    "query": {
      "terms": {
        #查询多个名字，类似于sql中的or
        "name": [
          "张三",
          "李四",
          "王五"
        ]
      }
    }
  }
  ```

##### 5.2 match查询

match查询属于高层查询，**它会根据你查询的字段的类型不同，采用不同的查询方式**

- 查询日期或者数值的时候，它会将你基于字符串的查询转换成日期或者数值对待
- 查询的内容不能被分词(keyword), match查询不会对你的查询的关键字进行分词
- 查询的内容是一个可以被分词的内容(text等),match会对查询的关键字进行一定的分词，然后去分词库中匹配

match查询，**底层其实就是多个term查询，然后将多个term查询的结果分装起来**

- match_all查询：查询所有内容，没有条件

  <!--es返回的total的个数是真实的数据条数，但是hits中的数据只有10条，是因为es默认返回的只有10条，可以通过设置size来控制显示多少条-->

  ```json
  POST /search_index/search_type/_search
  {
    "size":100,
    "query": {
      "match_all": {}
    }
  }
  ```

- match查询：根据条件查询内容

  ```json
  #查询content字段中包含今天天气真好的文档
  POST /search_index/search_type/_search
  {
    "query": {
      "match": {
        "content": "今天天气真好"
      }
    }
  }
  ```

- 布尔match查询：基于一个Field匹配的内容，采用and或者or的方式连接

  ```json
  #查询content字段中包含中国且者包含健康的文档
  POST /search_index/search_type/_search
  {
    "query": {
      "match": {
        "content": {
        	"query":"中国  健康",
        	#内容既包含中国又包含健康
        	"operator":"and"
        }
      }
    }
  }
  #查询content字段中包含中国或者包含健康的文档
  POST /search_index/search_type/_search
  {
    "query": {
      "match": {
        #查内容
        "content": {
        	"query":"中国  健康",
        	#内容既包含中国又包含健康
        	"operator":"or"
        }
      }
    }
  }
  ```

- multi_match查询：match是针对一个field的检索，multi_match是针对多个field的检索，多个field对应一个搜索的内容

  ```json
  #在content和address字段中找上海
  POST /search_index/search_type/_search
  {
    "query": {
      "multi_match": {
        "query": "上海",
        "fields": ["content","address"]
      }
    }
  }
  ```

##### 5.3 其他查询

###### 	5.3.1查询指定id的信息：

```json
GET /search_index/search_type/1
```

###### 5.3.2 查询多个id的信息

```json
#查询id为1,2,3的数据信息
POST /search_index/search_type/_search
{
  "query": {
    "ids": {
      "values": [1,2,3]
    }
  }
}
```

###### 5.3.3 prefix查询（查询效率低）

前缀查询，可以通过一个关键词去匹配一个field的前缀，从而查到指定文档，这个和match或者term查询的区别：举个例子：一个字段name是keyword类型，我想通过输入`张`来找到姓张的人，如果用term或者match是找不到的，因为name是keyword类型，但是prefix查询能找到

```json
#找name字段中姓张的文档，name是keyword类型
POST /search_index/search_type/_search
{
  "query": {
    "prefix": {
      "name": {
        "value": "张"
      }
    }
  }
}
```

###### 5.3.4 fuzzy查询（查询效率低）

模糊查询，我们输入字符的大概，es根据输入的内容的大概去匹配(比如输入的关键字中有错别字)，搜索的结果不稳定，如果跟真实的信息差别太大，也会搜索不出来，如：盒马鲜生，我们输入`盒马先生`是可以匹配到`盒马鲜生`的，但是如果输入`河马先生`就不行了，**该查询可以添加限制条件`prefix_length`标识前多少个字符不允许出现错误**，也可以不用加该限制条件

```json
#查询name可能是盒马先生的数据，限定前两个字必须正确
POST /search_index/search_type/_search
{
  "query": {
    "fuzzy": {
      "name": {
        "value": "盒马先生",
        "prefix_length": 2
      }
    }
  }
}
```

###### 5.3.5 wildcard查询（查询效率低）

通配查询，跟sql中的`like`类似

- 用`*`代表通配符，如`中国*`表示以中国开头的名称
- 用`?`代表占位符，如`中国??`表示以中国开头且后面有两个字的名称

```json
#查询name以中国开头的数据
POST /search_index/search_type/_search
{
  "query": {
    "wildcard": {
      "name": {
        "value": "中国*"
      }
    }
  }
}
```

###### 5.3.6 range查询

范围查询，只能查询数值类型的字段，对某个字段进行大于或者小于的范围指定

```json
#查询count字段>=10且<=20的数据，gt是大于，lt是小于
POST /search_index/search_type/_search
{
  "query": {
    "range": {
      "count": {
        "gte": 10,
        "lte": 20
      }
    }
  }
}
```

###### 5.3.7 regexp查询(查询效率低)

正则查询，根据写的正则类匹配

```json
#查询phone字段以176开头后面是8位数字的手机号
POST /search_index/search_type/_search
{
  "query": {
    "regexp": {
      "phone": "176[0-9]{8}"
    }
  }  
}
```

##### 5.4 游标查询scroll

ES对from+size来分页是有限制的，from+size最好不要超过1w条，否则对性能会造成很大的影响

ES查询原理：

> from+size在查询时的方式：
>
> 1. 将用户指定的关键词进行分词
> 2. 将词汇去分词库中进行检索，得到多个文档id
> 3. 去各个分片中拉取指定的数据，耗时较长
> 4. 将数据根据score进行排序，耗时较长
> 5. 根据from的值将查询到的数据舍弃一部分
> 6. 返回数据
>
> Scroll+size在查询时的方式：
>
> 1. 将用户指定的关键词进行分词
> 2. 将词汇去分词库中进行检索，得到多个文档id
> 3. 将文档的id存放在一个ES的上下文中(内存)
> 4. 根据你指定的size的个数去ES中检索指定个数的数据，拿完数据的文档id,会从上下文中移除（**这里的排序方式是默认的，根据文档的_doc排序，也可以手动指定排序方式**）
> 5. 如果需要下一页的数据，直接去ES的上下文中找后续数据
> 6. 循环4步和5步
>
> **scroll不适合做实时查询 **：因为游标查询会取某个时间点的快照数据。 查询初始化之后索引上的任何变化会被它忽略。
>
> scroll一般用于批量拉取大量数据，或者批量处理任务
>
> scroll+scan一般用于1千万粉丝的微信大V，要给所有粉丝群发消息 等场景

```json
#scroll=1m表示上下文在内存中保存一分钟
POST /search_index/search_type/_search?scroll=1m
{
  "size":100,
  "query": {
    "match_all": {}
  },
  #手动设置排序方式,可以设置根据多个字段排序
  "sort":[
      {
          "count":{
              "order":"desc"
          }
      }
  ]
}
#根据scroll查询下一页数据
POST /_search/scroll
{
	#scroll_id是第一次查询返回的scroll_id 	
  "scroll_id":"NASDREIDJCMISHEEUDKEBDH2AHHDHEBDJ34303DJ3HD9CJ3BDHDHAKJDHFAAAAAAAAAH3B3NDDU3JD9DHCB475HDDJHGDSKGDKFGNCXEU34G==",
   #因为如果不指定保存时间，这次查询之后这个上下文就没有了，所以还要指定1分钟 
    "scroll":"1m"
}
#删除scroll在ES上下文中的数据
#/_search/scroll/后面跟的是scroll_id
DELETE /_search/scroll/NASDREIDJCMISHEEUDKEBDH2AHHDHEBDJ34303DJ3HD9CJ3BDHDHAKJDHFAAAAAAAAAH3B3NDDU3JD9DHCB475HDDJHGDSKGDKFGNCXEU34G==
```

##### 5.5 按条件删除delete_by_query

根据term、match等查询删除大量的文档，**如果需要删除的内容是index中的大部分内容，不推荐使用这种删除，因为这种删除是逐条删除的，速度慢还消耗性能，更好的办法是，创建一个新的index，将需要保留的文档添加到新的index中**

```json
#根据count范围删除
DELETE /search_index/search_type/_delete_by_query
{
  "query": {
    "range": {
      "count": {
        "gte": 10,
        "lte": 20
      }
    }
  }
}
```

##### 5.6 复合查询

###### 	5.6.1 bool查询（重要）

​	复合过滤器，将你的多个查询条件，以一定的逻辑组合在一起

- **must: 所有的条件都必须符合，表示and的意思**
- **must_not:所有的条件都必须不符合，表示not的意思**
- **should:所有的条件，符合一个就可以，表示or的意思**

```json
#查询性别是男，年龄不是20岁，姓张或者姓杨的文档
POST /search_index/search_type/_search
{
    "query":{
        "bool":{
            "must":[
                {
                    "term":{
                        "sex":{
                            "value":1
                        }
                    }
                }
            ],
            "must_not":[
                {
                    "term":{
                        "age":{
                            "value":20
                        }
                    }
                }
            ],
            "should":[
                {
                    "wildcard":{
                        "name":{
                            "value":"张*"
                        }
                    }
                },
                {
                    "wildcard":{
                        "name":{
                            "value":"杨*"
                        }
                    }
                }
            ]
        }
    }
}
```

###### 5.6.2 boosting查询

boosting查询可以帮助我们去影响查询后的分数，从而起到影响排名的作用

boosting查询有三个参数：

- positive:只有匹配上positive的查询内容，才会被放到返回的结果集中
- nagative:如果匹配上了nagative也匹配上了positive，那么就会降低这些文档的score
- nagative_boost:指定系数，必须小于1

**关于查询分数是如何计算的：**

- **搜索的关键字在文档中出现的频次越高，分数越高**
- **搜索的关键字占文档字数百分比越大，分数越高**
- **如果指定的关键字被分词，这些被分词的内容，被分词库匹配的个数越多，分数越高**

```json
#搜索content字段中有`今天天气真好`的文档，但是content字段中包含`上海`的文档的score*0.5
#？？？？？？？？问题：如果需要多个条件怎么办，还需要实践一下！？？？？？？？？？？
POST /search_index/search_type/_search
{
    "query": {
        "boosting" : {
            "positive" : {
                "match" : {
                    "content" : "今天天气真好"
                }
            },
            "negative" : {
                 "match" : {
                     "content" : "上海"
                }
            },
            "negative_boost" : 0.5
        }
    }
}
```

#####  5.7 过滤器查询(filter)

filter查询和query查询的区别:

- query:根据你的查询条件，去计算文档的匹配度得到一个分数，并根据分数做排序，并且不会进行缓存
- filter:根据你的查询条件去查询文档，不去计算分数，而且filter会对经常被过滤的数据进行缓存，**比query速度快**

**过滤器查询一般用于不需要计算分数的查询或者，配合其他的query查询，通过filter查询的条件不影响匹配文档的分数**

```json
#如下查询中的status和publish_date不影响分数
POST /search_index/search_type/_search
{
  "query": { 
    "bool": { 
      "must": [
        { "match": { "title":   "Search"        }},
        { "match": { "content": "Elasticsearch" }}
      ],
      "filter": [ 
        { "term":  { "status": "published" }},
        { "range": { "publish_date": { "gte": "2015-01-01" }}}
      ]
    }
  }
}
```

##### 5.8 高亮查询

高亮查询时根据用户输入的关键字，以一定的特殊样式展示给用户，让用户知道为什么这个结果被检索出来。

高亮展示的数据本身就是文档的一个字段，单独将字段以高亮的形式返回

ES提供了一个highlight属性，和query同级别。

- fragment_size:指定高亮数据展示多少个字符,默认是100
- pre_tags:指定前缀标签，例如：`<font color="red">`
- post_tags:指定后缀标签，例如：`</font>`
- fields:指定哪几个field以高亮的形式返回

```json
POST /search_index/search_type/_search
{
  "query": {
    "match": {
      "content": "今天天气真好"
    }
  },
  "highlight":{
  	"fields":{
  		"content":{}
  	},
  	"pre_tags":"<font color='red'>",
  	"post_tags":"</font>",
  	"fragment_size":50
  }
}
```

##### 5.9 聚合查询（重要）

ES的聚合查询和MySQL的聚合查询类似，ES的聚合查询相比MySQL的要强大很多，ES提供的统计数据的方式多种多样

**聚合查询有很多，这里只介绍三种比较常用的，其他的可以查看官方文档**

查询语法：

```json
POST /search_index/search_type/_search
#这里可以写各种查询条件，在下面再写聚合查询
...
{
    "aggs":{
        #名字可以随便起，最好叫agg
        "名字":{
        	"agg_type":{
        		"属性":"值"
    		}
    	}
    }
}
```

###### 5.9.1 去重计数查询

及Cardinality ，第一步先将返回的文档中的一个指定的字段进行去重，然后在统计多少条

```json
#根据年龄统计
POST /search_index/search_type/_search
#这里可以写各种查询条件，在下面再写聚合查询
...
{
    "aggs":{
        #名字可以随便起，最好叫agg
        "agg":{
        	"cardinality":{
        		"field":"age"
    		}
    	}
    }
}
```

######  5.9.2 范围数值统计

统计一定范围内出现的文档个数，比如，针对某个字段，分别统计<5，5-20，>20的个数

**范围统计支持数值类型，时间类型和ip类型，分别是 range,date_range,ip_range**

```json
#数值统计
#统计年龄小于5的，大于等于5的小于20，大于等于20的文档各有多少个
POST /search_index/search_type/_search
{
    "aggs":{
        #名字可以随便起，最好叫agg
        "agg":{
        	"range":{
        		"field":"age",
        		"ranges":[
                    {
                        "to":5
                    },
                    {
                        #from的意思是大于等于
                        "from":5,
    					#to的意思是小于（不包含等于）
                        "to":20
                    },
                    {
                        "from":20
                    }
                ]
    		}
    	}
    }
}

#时间类型范围统计
POST /search_index/search_type/_search
{
    "aggs":{
        #名字可以随便起，最好叫agg
        "agg":{
        	"date_range":{
        		"field":"birthday",
        		#设置时间的类型
        		"format":"yyyy-MM-dd"
        		"ranges":[
                    {
                        "to":"2000-01-01"
                    },
                    {
                        #from的意思是大于等于
                        "from":"2000-01-01",
    					#to的意思是小于（不包含等于）
                        "to":"2020-01-01"
                    },
                    {
                        "from":"2020-01-01"
                    }
                ]
    		}
    	}
    }
}

#ip类型范围统计
POST /search_index/search_type/_search
{
    "aggs":{
        #名字可以随便起，最好叫agg
        "agg":{
        	"ip_range":{
        		"field":"ip_addr",
        		"ranges":[
                    {
                        "to":"10.1.1.1"
                    },
                    {
                        #from的意思是大于等于
                        "from":10.1.1.1,
    					#to的意思是小于（不包含等于）
                        "to":10.192.1.1
                    },
                    {
                        "from":10.192.1.1
                    }
                ]
    		}
    	}
    }
}
```

###### 5.9.3 统计聚合查询

`extended_stats` 这种查询可以返回出某个字段的最大值，最小值，平均值，总和等各种信息

```json
POST /search_index/search_type/_search
{
    "aggs":{
        #名字可以随便起，最好叫agg
        "agg":{
        	"extended_stats":{
        		"field":"age",
    		}
    	}
    }
}
```

##### 5.10 地图经纬度搜索

ES中提供了一种数据类型是geo_point,这种类型就是用来存储经纬度的

```json
#添加数据,field分别是名称name和经纬度location，创建索引步骤已省略
PUT /index_name/type_name/1
{
    "name":"陆家嘴",
    "location":{
        #lon是经度，lat是纬度
    	"lon":119.3746383,
    	"lat":138.9643873
	}
}
```

###### 5.10.1 ES地图的检索方式

- geo_distance：直线距离检索方式
- geo_bounding_box：以两个店确定一个矩形，获取矩形内所有数据
- geo_polygon：以多个点确定一个多边形，获取多边形内的全部数据

###### 5.10.2 实现地图检索

```json
#geo_distance
#获取这个坐标周围两千米的数据
POST /search_index/search_type/_search
{
    "query":{
        "geo_distance":{
            #确定坐标
            "location":{
                "lon":119.3746383,
    			"lat":138.9643873
            },
        	#确定半径
            "distance":2000,
        	#确定形状为圆形
            "distance_type":"arc"
        }
    }
}

#geo_bounding_box 通过两个点确定一个矩形
POST /search_index/search_type/_search
{
    "query":{
        "geo_bounding_box":{
            #确定坐标
            "location":{
            	#一个是左上角的坐标，一个是右下角的坐标
            	"top_left":{
            		"lon":118.3746383,
    				"lat":138.9643873
        		},
        		"bottom_left":{
            		"lon":119.3746383,
    				"lat":139.9643873
        		}
            }
        }
    }
}

#geo_polygon  通过多个坐标确定一个多边形
POST /search_index/search_type/_search
{
    "query":{
        "geo_polygon":{
            "location":{
            	"points":[
            		{
            			"lon":111.3746383,
    					"lat":131.9643873
        			},
        			{
            			"lon":119.3746383,
    					"lat":139.9643873
        			},
    				{
            			"lon":116.3746383,
    					"lat":139.9643873
        			},
					{
            			"lon":139.3746383,
    					"lat":139.9643873
        			}
            	]
            }
        }
    }
}
```

