# Redis学习笔记

### 1. Nosql概述

##### 1.1 为什么要用Nosql

随着时代的发展，MySQL已经不太适合现在的一些场景，热榜，大数据等，数据量很大而且变化很大，MySQL已经不够用了

用户的个人信息，社交网络，地理位置，用户产生的关系，用户日志等爆发式增长！这时候就需要用到Nosql了

##### 1.2 什么是NoSQL

NoSQL(NoSQL = Not Only SQL )，意即"不仅仅是SQL"。

最初的目的是为了大规模web 应用。NoSQL 的拥护者们提倡运用非关系型的数据存储，通常的应用如下特点：模式自由、支持简易复制、简单的API、最终的一致性（非ACID）、大容量数据等。

Redis是发展最快的，是我们当下必须要掌握的一项技术！！！

很多数据类型，比如用户的信息，社交网络，地理位置，这些数据类型的存储不需要一个固定的格式，

NoSQL特点：

- 方便扩展

- 大数据量高性能(一秒读8万次，读取11万次)

- 数据类型是多样的（不需要事先设计数据库，随取随用）

- 和关系型数据库的区别

  ```
  传统的关系型数据库
  1.结构化组织
  2.SQL
  3.数据和关系都存在单独的表中
  4.严格的一致性
  5.基础的事务
  NoSQL 
  1.不仅仅是数据
  2.没有固定的查询语言
  3.键值对存储，列存储，文档存储，图形数据库
  4.最终一致性
  5.CAP定理，BASE(异地多活)
  6.高性能，高可用，高扩展 
  ```

##### 1.3 NoSQL的四大分类

- kv键值对
  - 新浪：redis
  - 美团：redis + tair
  - 阿里，百度：redis + memcache
- 文档型数据库（bson格式）
  - MongoDB（一般必须要掌握）：是一个基于分布式文件存储的数据库，C++编写，主要用来处理大量的文档，MongoDB是非关系型数据库中功能最丰富的，最像关系型数据库的
  - ConthDB
- 列存储数据库
  - HBase
  - 分布式文件系统
- 图关系数据库（不是存图片的，存放的是关系，比如朋友圈社交网络，广告推荐）
  - Neo4j,InfoGrid

### 2. redis入门

##### 2.1 概述

 Redis (Remote DIctionary Server) 是一个由Salvatore Sanfilippo写的key-value存储系统。

Redis是一个**开源**的使用ANSI C语言编写、遵守BSD协议、支持网络、可基于内存亦可持久化的日志型、Key-Value数据库，并提供多种语言的API。它通常被称为数据结构服务器，因为值（value）可以是 字符串(String), 哈希(Hash), 列表(list), 集合(sets) 和 有序集合(sorted sets)等类型。

也被人称为结构化数据库

>Redis能干嘛？

1. 内存存储、持久化，内存是断电即刻消失，所以持久化很重要
2. 效率高，可以用于告诉缓存
3. 发布订阅系统
4. 地图信息分析
5. 计时器，计数器
6. ...  

> 特性

1. 多样的数据类型
2. 持久化
3. 集群
4. 事务

官网： https://redis.io/ 

中文官网： http://www.redis.cn/ 

Redis推荐在Linux服务器上搭建

##### 2.2 安装

- windows下安装

  ```
  1. 下载安装包，地址：https://github.com/tporadowski/redis/releases
  2. 下载完毕得到压缩包
  3. 解压
  4. 进入解压目录，双击redis-server.exe
  5. 双击redis-cli.exe连接redis客户端 
  ```

- Linux下安装

  ```
  1. 下载安装包 wget http://download.redis.io/releases/redis-5.0.8.tar.gz
  2. 放到/opt目录下,
  3. 解压 tar -xzvf redis-5.0.8.tar.gz
  4. 进入解压后的文件，可以看到redis.conf
  5. yum install gcc-c++
  6. make && make install
  7. redis的默认安装路径在/usr/local/bin
  8. 在该目录下创建 myconf文件
  9. 把/opt/redis-5.0.8/redis.conf复制到该目录下
  10. vim该文件，修改daemonize no 改为 daemonize yes(以守护进程启动)
  11. 启动redis服务 cd /usr/local/bin , redis-server /myconf/redis.conf（以该配置文件运行）
  12. 启动redis客户端， redis-cli -h 127.0.0.1 -p 6379 (本地运行可以不加 -h 和-p)
  ```

##### 2.3 性能测试

在`/usr/local/bin/`下有一个`redis-benchmark`的软件可以用来测试redis性能

redis 性能测试工具可选参数如下所示：

| 序号 | 选项      | 描述                                       | 默认值    |
| :--- | :-------- | :----------------------------------------- | :-------- |
| 1    | **-h**    | 指定服务器主机名                           | 127.0.0.1 |
| 2    | **-p**    | 指定服务器端口                             | 6379      |
| 3    | **-s**    | 指定服务器 socket                          |           |
| 4    | **-c**    | 指定并发连接数                             | 50        |
| 5    | **-n**    | 指定请求数                                 | 10000     |
| 6    | **-d**    | 以字节的形式指定 SET/GET 值的数据大小      | 2         |
| 7    | **-k**    | 1=keep alive 0=reconnect                   | 1         |
| 8    | **-r**    | SET/GET/INCR 使用随机 key, SADD 使用随机值 |           |
| 9    | **-P**    | 通过管道传输 <numreq> 请求                 | 1         |
| 10   | **-q**    | 强制退出 redis。仅显示 query/sec 值        |           |
| 11   | **--csv** | 以 CSV 格式输出                            |           |
| 12   | **-l**    | 生成循环，永久执行测试                     |           |
| 13   | **-t**    | 仅运行以逗号分隔的测试命令列表。           |           |
| 14   | **-I**    | Idle 模式。仅打开 N 个 idle 连接并等待。   |           |

我们来做一个测试

```shell
#测试：100个并发，100000个请求
redis-benchmark -h 127.0.0.1 -p 6379 -c 100 -n 100000
====== SET ======
  100000 requests completed in 1.96 seconds #10万个请求在1.96秒内完成
  100 parallel clients  #100个并发客户端
  3 bytes payload # 每次写入3个字节
  keep alive: 1 #只有一台服务器处理

31.85% <= 1 milliseconds
89.83% <= 2 milliseconds
99.63% <= 3 milliseconds
99.87% <= 4 milliseconds
99.90% <= 187 milliseconds
100.00% <= 187 milliseconds #所有请求在187毫秒处理完成
51072.52 requests per second #每秒处理51072.52个
```

##### 2.4 redis的基础知识

redis默认有16个数据库， 默认使用第0个数据库

- 可以使用`select`切换数据库

  ```shell
  #select 切换数据库
  127.0.0.1:6379> select 3
  OK
  #dbsize 查看数据库大小
  127.0.0.1:6379[3]> dbsize
  (integer) 0
  127.0.0.1:6379[3]> 
  ```

- 看看所有的key `keys *`

- 清空当前数据库: `flushdb`, 清空全部数据库: `flushall`

- **redis是单线程的！**redis是基于内存操作的，cpu不是redis的性能瓶颈，redis的性能瓶颈是机器的内存和网络的带宽，采用单线程，避免了不必要的上下文切换和竞争条件，也不存在多进程或者多线程导致的切换而消耗CPU，不用去考虑各种锁的问题，不存在加锁释放锁操作

### 3. 五大基础数据类型

**各数据类型能做什么**

**String：缓存、计数器、分布式锁等。**
**List：链表、队列、微博关注人时间轴列表等。**
**Hash：用户信息、Hash 表等。**
**Set（集合）：去重、赞、踩、共同好友等。**
**Zset（有序集合）：访问量排行榜、点击量排行榜等。**

基本命令

```shell
#查看所有key
keys *
#创建key
set key value
#获得一个key的值
get key
#判断key是否存在
exists key
#移除key 1表示当前数据库
move key 1
#设置key的过期时间 单位是秒
expire key seconds
#查看当前key的剩余时间
ttl key
#查看key的类型
type key
```

##### 3.1 string类型

```shell
#设置值
set key value
#获得一个key的值
get key
#给一个key追加数据,如果key不存在，相当于set 如:原有key值为yang，append之后变为yangguang
append name guang
#获取该key的长度
strlen key
#########################################################################
#值自动加1，一般用于文章的阅读数等
incr key
#值自动减1
decr key
#设置该key增加10
incrby key 10
#设置该key减少10
decrby key 10
#######################################################################
#截取一个字符串区间,从0开始，-1是到最后,
getrange key start end  #如;getrange key 0 3
#替换字符串,从key值中的第二个开始替换两个字符为xx
setrange key 1 xx
#########################################################################
#设置过期时间,设置key的值为xxxx,过期时间为60秒
setex key 60 xxxx
#如果不存在的时候设置，设置成功返回1，设置失败返回0，分布式锁中会经常使用
setnx name yangguang
#########################################################################
#批量设置值,批量设置name,age,phone
mset name yangguang age 18 phone 17633373393
#批量获取值
mget name age phone
#如果不存在的时候批量设置,具有原子性，要么一起成功，要么一起失败 
msetnx name yangguang age 18 phone 17633373393
#########################################################################
#设置对象
set user:1 {name:zhangsan,age:3} #设置一个user:1对象，值为json
#也可以用这种格式来保存对象信息,这只不过是两种保存对象的方式，具体怎么设置还要看实际用途
mset user:1:name zhangsan user:1:age 3
#########################################################################
#先get再set,返回的是get的值,如果不存在就相当于设置,如果存在就相当于更新
getset name zhangsan
```

##### 3.2 List类型

在redis中，list类型实际上是一个链表，可以用list实现消息队列(lpush,rpop)，栈(lpush,lpop)，如果链表很长，两边插入或者改动效率很高，中间插入或者修改，效率很低

```shell
lpush
lrange
#将一个值或者多个值，放在列表的头部(左）
lpush list_1 1 2 3
#lrange 获取一个队列的一段范围内的信息 lrang list start end
127.0.0.1:6379> lrange list_1 0 -1
1) "3"
2) "2"
3) "1"
#将一个值或者多个值，放在列表的尾部(右)
127.0.0.1:6379> rpush list_1 4 5 6
#可以看到 从尾部插入4，5，6
127.0.0.1:6379> lrange list_1 0 -1
1) "3"
2) "2"
3) "1"
4) "4"
5) "5"
6) "6"
#########################################################################
# lpop 从一个队列中从头部弹出一个值
127.0.0.1:6379> lpop list_1
"3"
# rpop 从一个队列中从尾部弹出一个值
127.0.0.1:6379> rpop list_1
"6"
127.0.0.1:6379> lrange list_1 0 -1
1) "2"
2) "1"
3) "4"
4) "5"
#########################################################################
lindex 获取某个队列中某个下标的值
#获取某个队列中某个下标的值
127.0.0.1:6379> lindex list_1 0
"2"
#########################################################################
llen 获取某个队列中某个下标的值
#返回队列list_1队列的值
127.0.0.1:6379> llen list_1
(integer) 4
#########################################################################
lrem 移除队列中指定个数的值 lrem key count value
#获取队列中所有的值
127.0.0.1:6379> lrange list_1 0 -1
1) "2"
2) "1"
3) "4"
4) "5"
#移除队列中一个1
127.0.0.1:6379> lrem list_1 1 1
(integer) 1
127.0.0.1:6379> lrange list_1 0 -1
1) "2"
2) "4"
3) "5"
#########################################################################
ltrim 从队列中的一个下标截取到另一个下标（其他的删除） ltrim key start end

127.0.0.1:6379> lpush num1 1 2 3 4 5
(integer) 5
#从num1队列中从下标2截取到下标5(下标5不存在，就直接截取到最后)
127.0.0.1:6379> ltrim num1 2 5
OK
#下标0的5和下标1的四已经被截取掉了
127.0.0.1:6379> lrange num1 0 -1
1) "3"
2) "2"
3) "1"
#########################################################################
rpoplpush 从一个队列移除，从另一个队列推入 rpoplpush source destination

127.0.0.1:6379> lrange num1 0 -1
1) "3"
2) "2"
3) "1"
#从num1移除一个值，移入num
127.0.0.1:6379> rpoplpush num1 num
"1"
127.0.0.1:6379> lrange num 0 -1
1) "1"
2) "3"
127.0.0.1:6379> lrange num1 0 -1
1) "3"
2) "2"
#########################################################################
lset 更新队列中某一个下标的值（必须要队列存在，否则报错）lset key index value
#更新num1队列中下标为0的值为333
lset num1 0 333
#########################################################################
linsert 在队列中的某个值的前面或者后面插入值 linsert key before|after value1 value2
#在num1队列中在2值之前插入一个1
127.0.0.1:6379> linsert num1 before 2 1
(integer) 3
127.0.0.1:6379> lrange num1 0 -1
1) "1"
2) "2"
3) "3"

```

##### 3.3 Set类型

无序集合，**set中的值是不能重复的**

```shell
sadd  往集合中添加数据(如果集合不存在创建集合，如果值存在返回0) set key value
smembers 查看集合中的所有值  smembers key
sismember 查看集合中是否有某一个值(存在返回1，不存在返回0) sismember key value
#往集合中添加数据(如果集合不存在创建集合)
127.0.0.1:6379> sadd myset hello
(integer) 1
#查看集合中的所有值
127.0.0.1:6379> smembers myset
1) "hello"
#查看集合中是否有某一个值
127.0.0.1:6379> sismember myset hello
(integer) 1
#########################################################################
scard 获取集合中的元素个数   scard key
srem 移除集合中的某个元素或者某几个元素(成功返回1，失败返回0) srem key value1 value2
127.0.0.1:6379> scard myset
(integer) 3
127.0.0.1:6379> srem myset !
(integer) 1
#########################################################################
srandmember 随机抽取一个或者多个元素 srandmember key [count]
127.0.0.1:6379> SRANDMEMBER myset
"world"
#########################################################################
spop 随机弹出集合中的一个元素或者多个元素 spop key [count]
127.0.0.1:6379> spop myset 2
1) "aaa"
2) "fff"
#########################################################################
smove 从一个集合中移除指定值到另一个集合（这个集合不存在会创建） smove source destination value
#从myset集合中移除ccc到youset集合中
127.0.0.1:6379> SMOVE myset youset ccc
(integer) 1
127.0.0.1:6379> SMEMBERS youset
1) "ccc"
#########################################################################
sdiff 第一个队列和其他队列的差集 sdiff key1 key2 [key3...]
sinter 第一个队列和其他队列的交集(共同关注，共同好友) sinter key1 key2 [key3...]
sunion 第一个队列和其他队列的差集 sunion key1 key2 [key3...]
127.0.0.1:6379> sdiff youset myset
1) "ccc"
127.0.0.1:6379> SINTER myset youset
1) "ddd"
2) "bbb"
127.0.0.1:6379> SUNION myset youset
1) "ddd"
2) "hello"
3) "bbb"
4) "ccc"
5) "eee"
6) "world"
```

##### 3.4 Hash 类型

Map集合，key-map(如：key-{key-value,key-value...}),本质上和string类型没太大的区别，还是一个简单的key-value，hash更适合对象的存储，比如存储用户的信息 user:1->{name:xxxx,age:xx,phone:xxx}

```shell
hset 设置一个hash类型的key  hset key field value
hget 获取一个hash类型key的field hget key field
hmset 设置多个hash类型的key hmset key field value [field value]
hmget 获取一个hash类型key的多个field hget key field [field]
hdel 删除一个hash类型的key的一个或多个field hdel key field [field]

#设置一个名称为myhash的key，其中field为name value为zhangsan
127.0.0.1:6379> hset myhash name zhangsan
(integer) 1
#获取field
127.0.0.1:6379> hget myhash name
"zhangsan"
#设置多个field
127.0.0.1:6379> hmset myhash name lisi age 18
OK
#获取多个field
127.0.0.1:6379> hmget myhash name age
1) "lisi"
2) "18"
#获取myhash中的所有field和value，其格式跟string差不多
127.0.0.1:6379> hgetall myhash
1) "name"
2) "lisi"
3) "age"
4) "18"
#删除myhash中的年龄field
127.0.0.1:6379> hdel myhash age
(integer) 1
#########################################################################
hlen 获取hash的长度  hlen key
hexists 获取某个field是否存在 hexists key field
hkeys 获取所有field名  hkeys key
hvals 获取所有的value值 hvals key

127.0.0.1:6379> hlen myhash
(integer) 1
127.0.0.1:6379> hkeys myhash
1) "name"
127.0.0.1:6379> hexists myhash age
(integer) 0
127.0.0.1:6379> hvals myhash
1) "lisi"
#########################################################################
hincrby 设置某个field增加多少（可以为负数）hincrby key field number
hsetnx 如果field不存在设置(成功返回1，失败返回0) hsetnx key field value

127.0.0.1:6379> hincrby myhash count 2
(integer) 3
127.0.0.1:6379> hsetnx myhash age 18
(integer) 1

```

##### 3.5 Zset 类型

Redis 有序集合和集合一样也是string类型元素的集合,且不允许重复的成员。

不同的是每个元素都会关联一个double类型的分数。redis正是通过分数来为集合中的成员进行从小到大的排序。

有序集合的成员是唯一的,但分数(score)却可以重复。

```shell
zadd 创建一个有序集合       zadd key score value [score value] 
zrange  获取一定范围(第几个到第几个)的有序集合(0 -1 表示所有,withscores表示显示分数,可以limit)  zrange key min max [withscores]
zrangebyscore 通过分数获取一定范围内的(正序的)有序集合（-inf +inf 表示负无穷到正无穷,withscores表示显示分数,可以limit） zrangebyscore key min_score max_score [withscores]
zrevrangebyscore 通过分数获取一定范围内的(倒叙的)有序集合（-inf +inf 表示负无穷到正无穷,withscores表示显示分数,可以limit） zrevrangebyscore key min_score max_score [withscores]

#获取下标为1到下标为3中间的数据,显示分数
127.0.0.1:6379> zrange myzset 1 3 withscores
1) "wangwu"
2) "22"
3) "chenliu"
4) "23"
5) "lisi"
6) "25"
#正序获取分数在20-40之间的数据，显示分数
127.0.0.1:6379> zrangebyscore myzset 20 40 withscores
 1) "wangwu"
 2) "22"
 3) "chenliu"
 4) "23"
 5) "lisi"
 6) "25"
 7) "zhanger"
 8) "37"
 9) "liwu"
10) "40"
#倒叙获取分数在正无穷-30之间的数据，显示分数
127.0.0.1:6379> ZREVRANGEBYSCORE myzset +inf 30 withscores
1) "liwu"
2) "40"
3) "zhanger"
4) "37"
#########################################################################
zrem 删除有序集合中的某个值  zrem key value
zcard 获取有序集合总数  zcard key
zcount 根据分数获取区间内的数量 zcount key min_score max_score

127.0.0.1:6379> zrem myzset liuwu
(integer) 0
127.0.0.1:6379> zcard myzset
(integer) 6
127.0.0.1:6379> zcount myzset 1 50
(integer) 6
#########################################################################
zscore 显示某个成员的分数 zscore key member
zrank 显示某个成员的排名(从小到大,返回的数字加一才是真实的排名) zrank key member
zrevrank 显示某个成员的排名(从大到小，返回的数字加一才是真实的排名) zrevrank key member
zincrby 给某个成员的分数加几(可以加负数)  zincrby key number member

127.0.0.1:6379> zscore myzset lisi
"43"
#5+1 从小到大第六名
127.0.0.1:6379> zrank myzset lisi
(integer) 5
#0+1 从大到小第一名
127.0.0.1:6379> zrevrank myzset lisi
(integer) 0
#lisi分数减5
127.0.0.1:6379> zincrby myzset -5 lisi
"38"

```

### 4. 三种特殊数据类型

##### 4.1 geospatial 类型

 朋友的定位，附近的人，打车距离计算？

redis的Geo（地理空间类型）在3.2版本就推出了，这个功能可以推算地理位置的信息，两地之间的距离，方圆几里之内的人

**geo类型其底层数据类型是zset，所以可以用zset的操作命令来做一些操作**

```shell
geoadd 添加地理位置(两极无法添加，有效的经度从-180度到180度。有效的纬度从-85.05112878度到85.05112878度)  geoadd key 经度 纬度 member [经度 纬度 member]
geopos 获取经纬度信息 geopos key member
geodist 获取两个坐标之间的直线距离（单位：m：米,km：千米,mi:英里，ft:英尺） geodist key member1 member2 [单位]
georadius 坐标且给定半径之内信息查询 georadius key 经度 纬度 radius 单位(m,km,mi,ft) [withcoord(显示坐标)] [withdist(显示距离)] [count(显示数量) number]
georadiusbymember 同georadius一样(不过中心点是key中的一个元素)
#添加经纬度城市信息
127.0.0.1:6379> geoadd city 116.405285 39.904989  beijing 121.472644 31.231706 shanghai  113.280637 23.125178 guangzhou 114.085947 22.547 shenzhen 113.665412 34.757975  zhengzhou
(integer) 5
#查询城市经纬度
127.0.0.1:6379> geopos city zhengzhou
1) 1) "113.66541177034378052"
   2) "34.75797603259534441"
#查询城市之间的距离
127.0.0.1:6379> geodist city zhengzhou shanghai km
"826.8356"
#查询坐标1000km之内的城市信息，显示坐标，显示距离，显示10个
127.0.0.1:6379> georadius city 110 30 1000 km withcoord withdist count 10
1) 1) "zhengzhou"
   2) "631.2281"
   3) 1) "113.66541177034378052"
      2) "34.75797603259534441"
2) 1) "guangzhou"
   2) "831.2636"
   3) 1) "113.28063815832138062"
      2) "23.12517743834835215"
3) 1) "shenzhen"
   2) "923.4929"
   3) 1) "114.08594459295272827"
      2) "22.54699993773966327"
#georadiusbymember
127.0.0.1:6379> GEORADIUSBYMEMBER city zhengzhou 1500 km
1) "shenzhen"
2) "guangzhou"
3) "shanghai"
4) "zhengzhou"
5) "beijing"

```

##### 4.2 Hyperloglog 类型

Redis 在 2.8.9 版本添加了 HyperLogLog 结构。

Redis HyperLogLog 是用来做基数统计的算法，HyperLogLog 的优点是，在输入元素的数量或者体积非常非常大时，计算基数所需的空间总是固定 的、并且是很小的。

在 Redis 里面，每个 HyperLogLog 键只需要花费 12 KB 内存，就可以计算接近 2^64 个不同元素的基 数。这和计算基数时，元素越多耗费内存就越多的集合形成鲜明对比。

但是，因为 HyperLogLog 只会根据输入元素来计算基数，而不会储存输入元素本身，所以 HyperLogLog 不能像集合那样，返回输入的各个元素。

HyperLogLog有0.81%的错误率，但是作为统计这些错误率是完全可以接受的，所以很适合统计网站UV这些应用场景

| 序号 | 命令及描述                                                   |
| :--- | :----------------------------------------------------------- |
| 1    | [PFADD key element [element ...\]](https://www.runoob.com/redis/hyperloglog-pfadd.html) 添加指定元素到 HyperLogLog 中。 |
| 2    | [PFCOUNT key [key ...\]](https://www.runoob.com/redis/hyperloglog-pfcount.html) 返回给定 HyperLogLog 的基数估算值。 |
| 3    | [PFMERGE destkey sourcekey [sourcekey ...\]](https://www.runoob.com/redis/hyperloglog-pfmerge.html) 将多个 HyperLogLog 合并为一个 HyperLogLog |

##### 4.3 bitmap 类型

统计 用户信息，活跃，不活跃， 登录，未登录，打卡，未打卡等，这些两种状态的都可以使用bitmaps

Bitmap 位图 ，数据结构！ 都是操作二进制位来进行记录，也就只有0和1两种状态

比如统计365天的打卡情况  365天= 365bit  1字节 = 8bit 46字节左右

BitMap是什么？

就是通过一个bit位来表示某个元素对应的值或者状态,其中的key就是对应元素本身。我们知道8个bit可以组成一个Byte，**所以bitmap本身会极大的节省储存空间。**

Redis从2.2.0版本开始新增了`setbit`,`getbit`,`bitcount`等几个bitmap相关命令。虽然是新命令，但是并没有新增新的数据类型，因为`setbit`等命令只不过是在`set`上的扩展。

其他相关应用场景详见: https://segmentfault.com/a/1190000008188655 

```shell
setbit 设置 setbit key offset value(0或者1)
getbit 获取 getbit key offset
bitcount 统计1的数量 bitcount key
bitop 对一个或多个保存二进制位的字符串 key 进行位元操作（ AND 、 OR 、 NOT 、 XOR ），并将结果保存到 destkey 上。 bitop operation destkey key [key]

#设置从开始有打卡功能开始，第x天的打卡情况，0未打卡，1打卡
127.0.0.1:6379> setbit sign 1 0
(integer) 0
127.0.0.1:6379> setbit sign 2 0
(integer) 0
127.0.0.1:6379> setbit sign 3 1
(integer) 0
127.0.0.1:6379> setbit sign 4 1
(integer) 0
127.0.0.1:6379> setbit sign 5 1
(integer) 0
127.0.0.1:6379> setbit sign 6 1
(integer) 0
127.0.0.1:6379> setbit sign 7 1
(integer) 0
127.0.0.1:6379> type sign
string
127.0.0.1:6379> get sign
"\x1f"
#获取第七天的打卡情况
127.0.0.1:6379> getbit sign 7
(integer) 1
#获取总的打卡次数
127.0.0.1:6379> bitcount sign
(integer) 5

```

### 5. redis事务

##### 5.1 redis事务简介

事务的本质：一组命令的集合，一个事务中所有的命令都是被序列化的，在事务执行过程中，会按照顺序执行

一次性、顺序性、排他性

redis事务没有隔离级别的概念，所有命令在事务中，并没有被执行，只有在发起执行命令(Exec)之后才能执行

redis同一事务中的命令中有编译性异常时，所有命令都不会被执行，但是这些命令中只有运行时异常，那么只有异常的命令不能执行，其他的命令还是可以正常执行的，所以**redis单条命令保证原子性，但是事务不保证原子性**

**编译性异常：**

命令写错，比如参数少写了，这些命令在事务中，exec时也会报错，所有命令都不会执行

**运行时异常：**

命令没有语法性的错误，比如, 自增(incr)一个字符串类型的value，这个时候，事务中其他命令可以正常执行，只有这条自增命令不能执行

**redis的事务**：

- 开启事务（multi）
- 命令入队（...）
- 执行事务（exec）

```shell
#开启事务
127.0.0.1:6379> multi
OK
#命令入队
127.0.0.1:6379> set name zhangsan
QUEUED
127.0.0.1:6379> set age 18
QUEUED
127.0.0.1:6379> get name
QUEUED
127.0.0.1:6379> set name lisi
QUEUED
#执行事务
127.0.0.1:6379> exec
1) OK
2) OK
3) "zhangsan"
4) OK

```

> 放弃事务

```shell
127.0.0.1:6379> multi
OK
127.0.0.1:6379> set phone 1111
QUEUED
#取消事务
127.0.0.1:6379> DISCARD
OK
#事务中的所有命令都不会被执行
127.0.0.1:6379> get phone
(nil)

```

##### 5.2 redis乐观锁

在redis中乐观锁的功能是由watch命令实现的，如果watch的key在其他线程中有被修改，那么这个线程中已经开启的事务中对这个key的修改就会失败，整个事务都不会执行

watch命令：监视一个事务中的某个key或者某几个key,`watch key [key]`

unwatch命令：解除监视，如果事务执行了`exec`或者`discard`，那么不需要手动解除监视 

例子如下：

```shell
#该例子需要两个终端来模拟并发场景
#终端1
#设置余额100
127.0.0.1:6379> set money 100
OK
#设置消费0
127.0.0.1:6379> set out 0
OK
#监控余额
127.0.0.1:6379> watch money
OK
#开启事务
127.0.0.1:6379> MULTI
OK
#在这个时候 终端2对money做了修改
127.0.0.1:6379> get money
"100"
127.0.0.1:6379> incrby money 900
OK
#终端2对money的修改结束

#在终端1中设置余额未80
127.0.0.1:6379> decrby money 20
QUEUED
#在终端1中设置消费20
127.0.0.1:6379> incrby out 20
QUEUED
#执行，发现没有执行成功
127.0.0.1:6379> exec
(nil)
#余额没有变成80，且消费没有变成20
127.0.0.1:6379> get money
"1000"
127.0.0.1:6379> get out
"0"
#这时就需要重新监视money，然后继续在事务中执行相关操作
#再次监视money
127.0.0.1:6379> watch money
#开启事务
127.0.0.1:6379> MULTI
#在终端1中设置余额未80
127.0.0.1:6379> decrby money 20
QUEUED
#在终端1中设置消费20
127.0.0.1:6379> incrby out 20
QUEUED
#执行，这次执行成功！！！！(前提是其他线程没有修改money)
127.0.0.1:6379> exec
1) 980
2) 20
```

### 6. redis.conf详解

常用的配置，从上到下依次是

1. 单位：unit单位对大小写不敏感

   ```
   # 1k => 1000 bytes
   # 1kb => 1024 bytes
   # 1m => 1000000 bytes
   # 1mb => 1024*1024 bytes
   # 1g => 1000000000 bytes
   # 1gb => 1024*1024*1024 bytes
   #
   # units are case insensitive so 1GB 1Gb 1gB are all the same.
   ```

2. 包含：用来加载一些单独设置配置文件

   ```
   ################################## INCLUDES ###################################
   
   # Include one or more other config files here.  This is useful if you
   # have a standard template that goes to all Redis servers but also need
   # to customize a few per-server settings.  Include files can include
   # other files, so use this wisely.
   #
   # Notice option "include" won't be rewritten by command "CONFIG REWRITE"
   # from admin or Redis Sentinel. Since Redis always uses the last processed
   # line as value of a configuration directive, you'd better put includes
   # at the beginning of this file to avoid overwriting config change at runtime.
   #
   # If instead you are interested in using includes to override configuration
   # options, it is better to use include as the last line.
   #
   # include /path/to/local.conf
   # include /path/to/other.conf
   
   ```

3. 网络(NETWORK)：

   ```bash
   # 这里设置的是本机访问，如果远程想要访问可以改成*
   bind 127.0.0.1   #绑定的ip
   protected-mode yes #保护模式，一般都是开启
   port 6379  #端口
   
   ```

4. 通用(GENERAL):

   ```shell
   daemonize yes #以守护进程的方式运行，默认是no，要改成yes
   pidfile /var/run/redis_6379.pid #如果以后台守护进程运行，我们就需要指定一个pid文件
   #日志
   # Specify the server verbosity level.
   # This can be one of:
   # debug (a lot of information, useful for development/testing)
   # verbose (many rarely useful info, but not a mess like the debug level)
   # notice (moderately verbose, what you want in production probably) 生产环境
   # warning (only very important / critical messages are logged)
   loglevel notice
   logfile ""   #日志文件的位置
   
   databases 16 #默认的数据库是16个
   
   always-show-logo yes #是否总是显示logo
   
   ```

5. RDB快照(SNAPSHOTTING):

   redis是内存数据库，如果没有持久化，那么断电就会丢失所有数据

   ```shell
   #如果900秒内，如果至少有一个key进行了修改，我们就进行持久化操作
   save 900 1
   #如果300秒内，如果至少有十个key进行了修改，我们就进行持久化操作
   save 300 10
   #如果60秒内，如果至少有一万个key进行了修改，我们就进行持久化操作
   save 60 10000
   #持久化如果出错，redis是否继续工作
   stop-writes-on-bgsave-error yes
   #是否压缩rdb文件，需要消耗一定的cpu资源
   rdbcompression yes 
   #保存rdb文件的时候，进行错误的检查校验
   rdbchecksum yes
   #rdb文件的保存目录 默认时当前目录
   dir ./
   ```

6. 复制(REPLICATION):

7. 安全(SECURITY):

   ```shell
   #设置密码 requirepass 123456 ，也可以通过命令设置 config set requirepass,设置文成后，通过 auth **** 输入密码登录
   # requirepass foobared
   ```

8. 客户端(CLIENTS):

   ```shell
   #设置最大客户端数量
   # maxclients 10000
   ```

9. 内存(MEMORY MANAGEMENT):

   ```shell
   #redis的最大内存限制
   # maxmemory <bytes>
   #内存满了之后的处理策略，
   1.noeviction：内存不足，写操作失败（2.xx版本默认的）
   2.allkeys-lru：移除最近最少使用的key（最常用的）
   3.allkeys-random：随机移除
   4.volatile-lru：在设置了过去时间的key中移除最近最少使用的key(3.2版本默认的)
   5.volatile-random：在设置了过期时间的key中随机移除
   6.volatile-ttl：在设置了过期时间的key中 有更早过期时间的被移除
   # maxmemory-policy noeviction
   ```

10. AOF设置(APPEND ONLY MODE):

    ```shell
    #默认不开启AOF模式持久化，默认是RDB方式
    appendonly no
    #持久化aof文件的名称
    appendfilename "appendonly.aof"
    #AOF持久化策略设置，
    # appendfsync always #每次执行都会sync,
    appendfsync everysec #每秒执行一次 sync,可能丢失一秒数据
    # appendfsync no #由操作系统同步数据
    
    ```

### 7. redis持久化

##### 7.1 RDB（Redis DataBase）持久化

redis会单独fork一个子进程来进行持久化，会先将数据写到一个临时文件中，等到持久化结束了，再用这个临时文件替换上次持久化的文件，整个过程中，主进程不进行任何IO操作，这就确保了极高的性能，如果需要进行大规模的数据恢复，且对恢复的数据完整性不是非常的敏感，RDB模式比AOF模式更高效，RDB的缺点是最后一次持久化的时候宕机了那么数据可能会丢失，redis默认使用RDB模式

再主从架构中，一般只在从机上开启RDB持久化

RDB保存的文件默认是; dump.rdb（可以再配置中设置）

> 触发规则

1. save的规则满足的情况下，会自动触发rdb规则
2. 执行flushall命令，也会出发rdb规则
3. 退出redis，也会产生rdb文件

> 恢复RDB文件

**只需要将RDB文件放在redis的启动目录下就可以，redis启动的时候会自动检查dump.db恢复其中的数据**

查看需要存放的位置

```shell
config get dir
1)"dir"
2)"/usr/local/bin" #再这个目录下存放RDB文件就会自动恢复其中的数据
```

> RDB优点

1. 适合大规模的数据恢复，因为速度更快
2. 如果对数据的完整性要求不高，可以使用RDB

> RDB缺点

1. 需要一定时间间隔进行操作，如果redis意外宕机了，上一次持久化之后的数据修改就会丢失
2. 需要fork进程来执行持久化，所以会消耗一定的内存

##### 7.2 AOF持久化

将我们的所有命令都记录下来，恢复的时候就把这个文件全部执行一遍。

以日志的形式记录每一个写操作，只允许追加文件不允许改写文件，redis启动之初会读取该文件重新构建数据，换言之，redis重启的话就根据日志文件的内容将写指令从前往后执行一遍

AOF保存的文件是appendonly.aof，可以通过配置文件修改

默认是不开启的，我们需要修改配置文件手动开启，redis重启后，就可以生效

如果aof文件有错误，redis是启动不起来的，可以用`redis-check-aof`工具来修复

```shell
redis-check-aof --fix appendonly.aof
```

> AOF的持久化规则

1.一种是写入AOF文件由系统决定，缓存区满就写入，效率高，

2.还有一种是一秒写入AOF文件一次，最多丢失一秒钟的缓存数据

3.还有一种是 每次写操作 都写入AOF文件,效率最低，但是最安全

> AOF的恢复

- 如果只配置 AOF ，重启时加载 AOF 文件恢复数据；
- 如果同时配置了 RDB 和 AOF ，启动是只加载 AOF 文件恢复数据；
- 如果只配置 RDB，启动是将加载 dump 文件恢复数据。

> AOF的优点

1. 通过设置，可以比RDB丢失更少的数据，或者是不丢失数据

> AOF缺点

1. aof的文件远远大于rdb文件，修复的速度也会比rdb慢
2. aof运行效率也要比rdb慢，因为由磁盘IO

### 8. redis发布订阅

redis的发布订阅(pub/sub)是一种消息通信模式，发送者发送消息，订阅者(sub)接收消息，redis的客户端可以订阅任意数量的频道

例如：有三个客户端  client2 、 client5 和 client1  订阅了channel1, 当有新消息通过 PUBLISH 命令发送给频道 channel1 时， 这个消息就会被发送给订阅它的三个客户端 

> 相关命令

| 序号 | 命令及描述                                                   |
| :--- | :----------------------------------------------------------- |
| 1    | [PSUBSCRIBE pattern [pattern ...\]](https://www.runoob.com/redis/pub-sub-psubscribe.html) 订阅一个或多个符合给定模式的频道。 |
| 2    | [PUBSUB subcommand [argument [argument ...\]]](https://www.runoob.com/redis/pub-sub-pubsub.html) 查看订阅与发布系统状态。 |
| 3    | [PUBLISH channel message](https://www.runoob.com/redis/pub-sub-publish.html) 将信息发送到指定的频道。 |
| 4    | [PUNSUBSCRIBE [pattern [pattern ...\]]](https://www.runoob.com/redis/pub-sub-punsubscribe.html) 退订所有给定模式的频道。 |
| 5    | [SUBSCRIBE channel [channel ...\]](https://www.runoob.com/redis/pub-sub-subscribe.html) 订阅给定的一个或多个频道的信息。 |
| 6    | [UNSUBSCRIBE [channel [channel ...\]]](https://www.runoob.com/redis/pub-sub-unsubscribe.html) 指退订给定的频道。 |

订阅端：

```shell
redis 127.0.0.1:6379> SUBSCRIBE redisChat

Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "redisChat"
3) (integer) 1
```

发布端：

```shell
redis 127.0.0.1:6379> PUBLISH redisChat "Redis is a great caching technique"

(integer) 1

redis 127.0.0.1:6379> PUBLISH redisChat "Learn redis by runoob.com"

(integer) 1

# 订阅者的客户端会显示如下消息
1) "message"
2) "redisChat"
3) "Redis is a great caching technique"
1) "message"
2) "redisChat"
3) "Learn redis by runoob.com"
```

> 原理

通过`SUBSCRIBE`命令订阅了某个频道之后，redis-server里维护了一个字典，字典的键就是一个个的频道，字典的值是一个链表，链表中保存着所有订阅这个频道的客户端，`SUBSCRIBE`命令的关键就是将客户端添加到指定的链表中。

通过`publish`命令向订阅者发送消息，redis-server会使用给定的频道作为键，再它维护的频道字典中找到这个频道下的订阅客户端的链表，便利链表，发送消息给订阅者

发布订阅可以用于实时消息系统中，比如及时聊天，群聊等

### 9. redis主从复制

主从复制：是指一台redis服务器的数据，复制到其他的redis服务器。前者是主节点(master)，后者是从节点(slave)，数据的复制是单向的，只能从主节点到从节点，主节点以写为主，从节点以读为主。

> 主从复制的作用

- 数据冗余：主从复制实现了数据的热备份，是持久化之外的一种数据冗余方式
- 故障恢复：当主节点出现问题时，可以由从节点提供数据，实现快速的故障恢复，实际上是一种服务的冗余
- 负载均衡：再主从复制的基础上，配合读写分离，可以由主节点提供写服务，从节点提供读服务，分担服务器负载，尤其是再读多写少的场景下，从多个从节点分担读压力，可以大大增加redis服务的并发量
- 高可用基石：主从复制还是哨兵和集群能够实现的基础，因此说主从复制是redis高可用的基础

一般来说，要将redis应用到工程项目中，只使用一台redis服务器是万万不能的，原因:

- 从结构上，单个redis服务器可能会发生故障，而且一台服务器需要处理所有的请求负载，压力太大
- 从容量上，单服务器内存容量有限。**单台redis服务器内存最好不要大于20G**

**所有一般互联网项目中，都是一主多从的架构**

以下测试，会在一台服务器上配置3个redis服务，分别监听79，80，81端口，79为主机，80，81为从机

##### 9.1 基础环境配置

只配置从库，不用配置主库

```shell
#查看当前库的信息
127.0.0.1:6379> info replication
# Replication
role:master #角色
connected_slaves:0 #从机数量
master_replid:a25c8550faa2e7ddaf3cc1184416b46fa4f3c556
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:0
second_repl_offset:-1
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0
```

复制3个配置文件，修改对应信息

1. 端口
2. pid文件名称
3. log文件名称
4. rdb文件名称

修改完成后，依次启动redis服务 `redis-server myconf/redis79.conf`,`redis-server myconf/redis80.conf`,`redis-server myconf/redis81.conf`,然后用`ps`命令查看是否启动成功

##### 9.2 一主二从

默认情况下，每台redis服务都是主服务，只需要配置从节点就可以了

- 通过命令配置（只是用来测试，服务一关闭，主从结构就不存在了）

  ```shell
  #配置80和81为从机
  
  #分别在80和81客户端中设置 slaveof host port
  127.0.0.1:6380> SLAVEOF 127.0.0.1 6379
  OK
  127.0.0.1:6380> info replication
  # Replication
  role:slave  #角色 从节点
  master_host:127.0.0.1 #主节点ip
  master_port:6379 #主节点端口
  master_link_status:up
  master_last_io_seconds_ago:6
  master_sync_in_progress:0
  slave_repl_offset:28
  slave_priority:100
  slave_read_only:1
  connected_slaves:0
  master_replid:76bd4019b1524e3c6fe3715f955e0e31f257ca2d
  master_replid2:0000000000000000000000000000000000000000
  master_repl_offset:28
  second_repl_offset:-1
  repl_backlog_active:1
  repl_backlog_size:1048576
  repl_backlog_first_byte_offset:1
  repl_backlog_histlen:28
  
  #在主机上查看info
  127.0.0.1:6379> info replication
  # Replication
  role:master
  connected_slaves:2
  slave0:ip=127.0.0.1,port=6380,state=online,offset=238,lag=1   #有两个从机
  slave1:ip=127.0.0.1,port=6381,state=online,offset=238,lag=1
  master_replid:76bd4019b1524e3c6fe3715f955e0e31f257ca2d
  master_replid2:0000000000000000000000000000000000000000
  master_repl_offset:238
  second_repl_offset:-1
  repl_backlog_active:1
  repl_backlog_size:1048576
  repl_backlog_first_byte_offset:1
  repl_backlog_histlen:238
  
  ```

- 通过配置文件配置(真实场景都是通过这种配置的)

  1. 修改配置文件

     ```shell
     #只需要修改80和81的配置文件就可以
     #在配置文件中找到 REPLICATION 区域，这里是用来配置主从的
     # replicaof <masterip> <masterport> #这里是配置主节点的ip和port的
     replicaof 127.0.01 6379
     
     # If the master is password protected (using the "requirepass" configuration
     # directive below) it is possible to tell the replica to authenticate before
     # starting the replication synchronization process, otherwise the master will
     # refuse the replica request.
     #
     # masterauth <master-password>  #如果主节点有密码，这里是配置主节点密码
     ```

  2. 保存并退出

  3. 启动各自服务，这样这三台redis服务就会自动配置主从结构

     ```shell
     redis-server myconf/redis79.conf
     redis-server myconf/redis80.conf
     redis-server myconf/redis81.conf
     ```

  4. 验证

     ```shell
     #登录主节点客户端，查看信息
     127.0.0.1:6379> info replication
     # Replication
     role:master
     connected_slaves:2  #有两个从节点
     slave0:ip=127.0.0.1,port=6380,state=online,offset=70,lag=1
     slave1:ip=127.0.0.1,port=6381,state=online,offset=70,lag=1
     master_replid:9de68f1b0da3c221854469330a854ecb6ddc53e3
     master_replid2:0000000000000000000000000000000000000000
     master_repl_offset:70
     second_repl_offset:-1
     repl_backlog_active:1
     repl_backlog_size:1048576
     repl_backlog_first_byte_offset:1
     repl_backlog_histlen:70
     ```

##### 9.3 主从复制注意点

- 配置主从之后，数据会从主机同步到从机，从机无法进行写操作。
- 主机断开后，从机还是从机，==不会变成主机(配置哨兵模式之后，会变成主机)，主机回来之后，从机依旧能从主机获取信息
- 如果是通过命令行配置的主从，从机(81那台)断开后再启动，这时候就不能获取到主机后来新增的信息了，因为，从机从新启动后又变回一个主机了，然后再把这个81加入原来的主从结构中，断开之后的信息同样会同步到81上

> 主从复制原理

从机成功连接主机后，会发送一个sync同步命令

主机接收到命令，启动后台存盘进程，同时收集所有接收到的用于修改数据集命令，在后台进程执行完成后，主机将传送整个数据文件到从机，完成一次完全同步（全量复制）

**全量复制**：从机接收到数据库文件后，将其存盘并加载到内存中

**增量复制**：主机继续将新的收集到的命令依次传给从机，完成同步

只要从机重新连上主机，依次完全同步(全量复制)就会自动执行！

### 10. 哨兵模式

当一台服务器宕机后，需要手动把一台从机设置成主机，这需要人力干预，费时费力，还会造成一段时间内服务器不可用，不是一种好方法，更多的时候，我们优先考虑使用哨兵模式。

redis从2.8开始正式提供Sentinel(哨兵)架构来解决这个问题，**该模式能够后台监控主机是否故障，如果故障了根据投票输自动将从库转变成主库**

哨兵模式是一种特殊的模式，首先redis提供了哨兵的命令，哨兵是一个独立的进程，**其原理是同通过发送命令，等待redis服务器的响应，从而监控多个redis实例**

哨兵有两个作用：

- 通过发送命令，让redis服务器返回其运行状态，包括主服务器和从服务器
- 当哨兵检测到主服务器宕机了，会自动将从服务器切换成主服务器，然后通过**发布订阅模式**通知其他的从服务器，修改配置文件，让它们切换主机

然而一个哨兵进程进行监控也可能出现问题，所以我们可以同时使用多个哨兵进行监控，各个哨兵之间也会进行监控，这样就形成了**多哨兵模式**

假设主服务器宕机，哨兵1检查到这个结果，系统并不会马上进行failover（故障转移）过程，仅仅是哨兵1主观认为主服务器不可用，这个现象叫做**主观下线**，当后面的哨兵也检测到主服务器不可用，并且达到一定的数量后，哨兵之间会进行依次投票，投票结果由一个哨兵发起，进行failover（故障转移）操作，切换成功后，就会通过发布订阅系统，让各个哨兵把自己监控的从服务器切换主机，这个过程就是**客观下线**

> 测试

我们目前时一主二从

1. 配置哨兵配置文件

   ```shell
   #sentinel monitor 被监控的名称 host port 1(这数字表示主机挂了，从机投票看哪个变成主机)
   sentinel monitor myredis 127.0.0.1 6379 1
   # 守护进程模式
   daemonize yes
   ```

2. 启动哨兵模式

   ```shell
   #启动哨兵模式
   redis-sentinel myconf/sentinel.conf 
   28397:X 30 Jun 2020 17:50:37.561 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
   28397:X 30 Jun 2020 17:50:37.561 # Redis version=5.0.8, bits=64, commit=00000000, modified=0, pid=28397, just started
   28397:X 30 Jun 2020 17:50:37.561 # Configuration loaded
   ```

如果主机下线，通过选举选择一个从机作为主机继续提供服务，其本质就是修改各redis服务器的配置文件来实现的，主机再次上线，那么现在它已经不是主机了，而变成了一个从机

> 哨兵模式优点

1. 哨兵集群，基于主从复制模型，有所主从复制模型有的优点，它都有
2. 主从可以切换，故障可以转移，系统的可用性提高
3. 哨兵是主从模式的升级，手动变成自动，更加健壮

> 哨兵模式缺点

1. redis不好在线扩容，集群容量一旦达到上限，在线扩容十分麻烦
2. 实现多哨兵模式配置很麻烦(前面的是实例，而且是单哨兵)，里面由很多选择，可参控教程 https://www.jianshu.com/p/911935fe723e 

### 11. redis缓存穿透、击穿和雪崩

##### 11.1 缓存穿透（查不对）

缓存穿透的概念很简单，用户向查询一个数据，发现redis中没有，就会向数据库发送请求查询，发现也没有，本次请求失败，当用户很多时，缓存都没有命中，都去请求数据库，这就会造成数据库压力很大，也就是**缓存穿透**

> 解决方案

- 布隆过滤器

- 缓存空对象， 当数据库不命中的时候，及返回的空数据缓存起来，同时设置一个过期时间，这样就保护了后端数据库

  不过这种方案也有一些问题：

  - 因为要保存很多空值，所以就需要更多的内存空间去保存跟多的键
  - 即便设置了过期时间，还是会造成缓存层和数据层的数据有一定时间的不一致，比如，在缓存有效期内，数据库有数据，但是缓存的却是空（个人想法：可以在数据库存储该数据的时候，去redis中查找是否存在这个数据对应的key，如果存在，判断是否一致，不一致更新缓存）

##### 11.2 缓存击穿（量太大，缓存过期）

缓存击穿是指，一个热点key在不停的扛着大并发，当这个key设置了失效时间，且失效瞬间，持续大量的并发访问数据库，会造成数据库压力瞬间增大很多倍

> 解决方案

- 热点数据永不过期

- 分布式锁，就是在缓存过期时，只有一个请求能够去数据库访问，其他请求等待

  个人思路：当redis中热点key 不存在的时候，通过`setnx`命令这只这个key的锁(记得设过期时间)，如果返回1，则去数据库中查询,查询到后，重新设置这个key，其他请求到这一步返回的肯定是0，等待一定时间（sleep什么的）再去redis中请求这个key

##### 11.3 缓存雪崩

缓存雪崩就是redis中数据集体过期，或者服务器宕机或者断电，数据过期的压力，数据库还可能顶的主，但是如果宕机，所有数据都没了，对数据库的压力时不可估量的

> 解决方案

- redis高可用（keepalived等,）:设置异地多活
- 限流降速：解决方法就是缓存击穿的分布式锁方法
- 数据预热：提前先把可能访问的数据预先访问一遍，这样大量的数据就会存在缓存中

### 12. redis集群

##### 12.1 集群简介

redis的ops可以达到10万/秒，但是如果业务的OPS是20万或者30万/秒呢？单机redis的内存上限是256G，业务需要内存容量是1T呢？这里就需要redis集群来解决这些问题

集群就是使用网络将若干台计算机联通起来，提供统一的管理方式，使其对外呈现单机服务的效果

> 集群的作用

- 分摊单台服务器的访问压力，实现负载均衡
- 分摊单台服务器的存储压力，实现可扩展性
- 降低单台服务器宕机带来的业务灾难

##### 12.2 redis集群结构设计

> 数据存储设计

1. 通过算法设计，计算出key应该保存的位置，具体是通过对key通过CRC16后再对16384取模得到一个数值
2. 将所有的存储空间分割成16384份，也就是槽(slot)的概念，每台服务器保存一部分，
3. 将key按照计算出的数值，放入对应的槽中
4. 当需要增加服务器的时候，redis就是把其他服务器的一部分槽给了这个新的服务器，减少服务器就是把要减少的那个服务器的槽分给其他服务器，其实**所谓的增减节点就是改变了槽的存储位置**

> 集群内部通讯设计

- 各个节点的数据库相互通信，保存各个库中槽的编号
- 当一个请求过来，通过算法知道这个key在哪个槽，又因为各个节点都知道所有的槽在哪个节点中，如果这个槽在当前节点中，那么一次命中直接返回，如果不在当前节点中，告知其具体位置

##### 12.3 cluster集群结构搭建

下面要搭建的3主3从的集群架构

1. 首先把`redis.conf`复制成`redis-6379.conf`,然后修改`redis-6379.conf`其中的`port`,`pidfile`,`dbfilename`,`appendfilename`，然后在文件中添加如下配置并保存

   ```shell
   vim redis-6379.conf
   #在配置文件中添加
   cluster-enabled yes
   cluster-config-file node-6379.conf #节点配置文件
   cluster-node-timeout 10000  #判断节点下线时间 10秒
   ```

   然后通过命令复制多个并且修改其中的6379变成6380等

   ```shell
   sed "s/6379/6380/g" redis-6379.conf > redis-6380.conf
   sed "s/6379/6381/g" redis-6379.conf > redis-6381.conf
   sed "s/6379/6382/g" redis-6379.conf > redis-6382.conf
   sed "s/6379/6383/g" redis-6379.conf > redis-6383.conf
   sed "s/6379/6384/g" redis-6379.conf > redis-6384.conf
   ```

   然后启动这六个redis服务

   ```shell
   redis-server myconf/redis-cluster/redis-6379.conf
   redis-server myconf/redis-cluster/redis-6380.conf
   redis-server myconf/redis-cluster/redis-6381.conf
   redis-server myconf/redis-cluster/redis-6382.conf
   redis-server myconf/redis-cluster/redis-6383.conf
   redis-server myconf/redis-cluster/redis-6384.conf
   #然后通过ps命令查看6个服务是否启动
   ps -aux|grep redis
   ```

2. 创建cluster集群

   ```shell
   #redis5.0之后通过redis-cli --cluster命令启动集群，不需要redis-trib.rb这样启动了
   redis-cli --cluster create 127.0.0.1:6379 127.0.0.1:6380 127.0.0.1:6381 127.0.0.1:6382 127.0.0.1:6383 127.0.0.1:6384 --cluster-replicas 1
   [OK] All nodes agree about slots configuration.
   >>> Check for open slots...
   >>> Check slots coverage...
   [OK] All 16384 slots covered.
   #启动成功！！！！
   #如果想要关闭直接关各个redis服务即可
   ```

##### 12.4 设置和获取数据

客户端连接redis集群不能用之前的`redis-cli`要用`redi-cli -c`，这样表示用集群的连接方式