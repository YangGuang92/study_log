### 1.消息中间件介绍

##### 1.1 消息中间件用处

- 任务异步处理
- 应用程序解耦
- 消峰去谷

##### 1.2 实现消息中间件的协议

- AMQP:是一种链接协议，和JMS的本质区别是不从API层进行限制（也就是说可以用任何语言去实现），而是直接定义网络交换的数据格式
- JMS:是一种JAVA平台关于面向消息中间件的API

##### 1.3 AMQP和JMS的区别

- jms定义了统一的接口，来对消息操作进行统一，AMQP只是定义了数据交换的格式
- JMS限定了java语言，AMQP只是一种协议，不限制语言
- JMS规定了两种消息模式，AMQP消息模式更丰富

##### 1.4 常见的消息队列产品

1. ActiveMQ:基于JMS
2. ZeroMQ:基于C语言开发
3. RabbitMQ:基于AMQP协议，erlang语言开发，稳定性好
4. RocketMQ:基于JMS,阿里开发
5. kafka:类似MQ的产品：分布式消息系统，高吞吐量，常用于大数据领域

##### 1.5 RabbitMQ六种模式

简单模式，work模式，发布\订阅模式，路由模式，主题模式，RPC模式（rpc模式并不是MQ）

### 2. RabbitMQ安装





### 3.RabbitMQ入门（简单模式）

首先需要添加`php-amqplib`的composer包依赖管理

```json
{
     "require": {
      "php-amqplib/php-amqplib": "2.6.1"
     }
 }
```

然后`composer install`

##### 3.1 发布消息

```php
<?php

require_once __DIR__ . '/vendor/autoload.php';
use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Message\AMQPMessage;

//创建连接，参数分别为 ip或域名，端口号，用户名，密码
//感觉少了设置虚拟机的过程？难道有默认的虚拟机？
$connection = new AMQPStreamConnection('localhost', 5672, 'guest', 'guest');
//创建频道（或者叫管道）
$channel = $connection->channel();
//创建队列 参数分别为（队列已创建的，可以省略这一步）
$channel->queue_declare('hello', false, false, false, false);

$msg = new AMQPMessage('Hello World!');
$channel->basic_publish($msg, '', 'hello');

echo " [x] Sent 'Hello World!'\n";

$channel->close();
$connection->close();
?>
```

##### 3.2 接受消息

```php
$callback = function($msg) {
  echo " [x] Received ", $msg->body, "\n";
};

$channel->basic_consume('hello', '', false, true, false, false, $callback);

while(count($channel->callbacks)) {
    $channel->wait();
}
```



### 4.工作队列(work queue)

理解：**消息生产者把消息发布到默认的交换机(exchange)中，然后交换机把消息发送到指定工作队列中，然后多个消费者以轮询的方式消费这些信息**

##### 4.1 发布消息（包含消息持久化）

```php
<?php

require_once __DIR__ . '/vendor/autoload.php';
use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Message\AMQPMessage;

//创建连接
$connection = new AMQPStreamConnection('localhost', 5672, 'guest', 'guest');
//创建信道
$channel = $connection->channel();
//创建队列，这是就是用的是默认的交换机
//第三个参数true表示队列持久化
$channel->queue_declare('task_queue', false, true, false, false);

$data = implode(' ', array_slice($argv, 1));
if(empty($data)) $data = "Hello World!";
//生成信息
//第二个参数的意思表示消息持久化
$msg = new AMQPMessage($data,
                        array('delivery_mode' => AMQPMessage::DELIVERY_MODE_PERSISTENT)
                      );
//消息发布
$channel->basic_publish($msg, '', 'task_queue');

echo " [x] Sent ", $data, "\n";
//关闭信道，释放资源
$channel->close();
//关闭连接，释放资源
$connection->close();

?>
```

##### 4.2 接收消息（包含公平调度）

```php
<?php

require_once __DIR__ . '/vendor/autoload.php';
use PhpAmqpLib\Connection\AMQPStreamConnection;

//创建连接
$connection = new AMQPStreamConnection('localhost', 5672, 'guest', 'guest');
//创建信道
$channel = $connection->channel();
//创建队列，这是就是用的是默认的交换机
//第三个参数true表示队列持久化
$channel->queue_declare('task_queue', false, true, false, false);

echo ' [*] Waiting for messages. To exit press CTRL+C', "\n";

$callback = function($msg){
  echo " [x] Received ", $msg->body, "\n";
  sleep(substr_count($msg->body, '.'));
  echo " [x] Done", "\n";
  $msg->delivery_info['channel']->basic_ack($msg->delivery_info['delivery_tag']);
};
//表示公平调度，这表示同一时间不能发送超过一条消息给消费者
$channel->basic_qos(null, 1, null);
//使用默认交换机接受消息
$channel->basic_consume('task_queue', '', false, false, false, false, $callback);

while(count($channel->callbacks)) {
    $channel->wait();
}
//关闭信道和关闭连接
$channel->close();
$connection->close();

?>
```

##### 4.3 消息确认

理解：**当消费者在没有确认之前，队列是不会删除信息的，消息确认是默认开启的，如需关闭则需要在basic_consume函数中的第四个参数设置为true**

```php
$callback = function($msg){
  echo " [x] Received ", $msg->body, "\n";
  sleep(substr_count($msg->body, '.'));
  echo " [x] Done", "\n";
  //消息确认
  $msg->delivery_info['channel']->basic_ack($msg->delivery_info['delivery_tag']);
};
//第四个参数是是否开启消息确认，false是需要开启消息确认(默认的)
$channel->basic_consume('task_queue', '', false, false, false, false, $callback);
```

### 5.发布/订阅模式（扇形交换机）

理解：发布者需要把消息发送到交换机，而不是队列中，交换机在推送给指定的队列

<!--tips 如果交换机没有绑定队列，那么发送者发出的消息就会丢失-->

##### 5.1 发布消息

```php
<?php

require_once __DIR__ . '/vendor/autoload.php';
use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Message\AMQPMessage;
//创建连接
$connection = new AMQPStreamConnection('localhost', 5672, 'guest', 'guest');
//创建信道
$channel = $connection->channel();
//创建交换机(如果交换机已手动创建，可以不用在这里创建)
//第一个参数是交换机的名称，第二个参数是交换机的类型，该类型为扇形交换机
$channel->exchange_declare('logs', 'fanout', false, false, false);

$data = implode(' ', array_slice($argv, 1));
if(empty($data)) $data = "info: Hello World!";
//创建信息
$msg = new AMQPMessage($data);
//发布信息到指定的交换机，第二个参数是交换机的名称
$channel->basic_publish($msg, 'logs');

echo " [x] Sent ", $data, "\n";

$channel->close();
$connection->close();

?>
```



##### 5.2 接收消息(绑定，临时队列)

<!--tips 接受的消息的php文件先执行，所以绑定可以写在接收的文件里-->

```php
<?php

require_once __DIR__ . '/vendor/autoload.php';
use PhpAmqpLib\Connection\AMQPStreamConnection;
//创建连接
$connection = new AMQPStreamConnection('localhost', 5672, 'guest', 'guest');
//创建信道
$channel = $connection->channel();
//创建交换机(如果交换机已手动创建，可以不用在这里创建)
//第一个参数是交换机的名称，第二个参数是交换机的类型，该类型为扇形交换机
$channel->exchange_declare('logs', 'fanout', false, false, false);
//创建队列，如果第一个参数不传，那么创建的就是一个临时队列，且断开连接时，队列删除
//该方法会返回一个随机生成的队列名称，类似：amq.gen-jzty20brgko-hjmujj0wlg
list($queue_name, ,) = $channel->queue_declare("", false, false, true, false);
//绑定队列和交换机，第一个参数是队列的名称，第二个参数是交换机的名称
//如果交换机没有绑定队列，那么发送者发出的消息就会丢失
$channel->queue_bind($queue_name, 'logs');

echo ' [*] Waiting for logs. To exit press CTRL+C', "\n";

$callback = function($msg){
  echo ' [x] ', $msg->body, "\n";
};
//接收消息
$channel->basic_consume($queue_name, '', false, true, false, false, $callback);

while(count($channel->callbacks)) {
    $channel->wait();
}
//关闭通信和连接
$channel->close();
$connection->close();

?>
```

### 6. 路由模式(直连交换机)

理解：**根据发布者传入不同的routing_key,交换机把消息发送给绑定了这个routing_key的队列**

<!--tips 如果多个队列使用相同的routing key 绑定到交换机，那么其功能跟发布/订阅相同-->

##### 6.1 发布消息

```php
<?php

require_once __DIR__ . '/vendor/autoload.php';
use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Message\AMQPMessage;
//创建连接
$connection = new AMQPStreamConnection('localhost', 5672, 'guest', 'guest');
//创建信道
$channel = $connection->channel();
//创建交换机，第二个参数是交换机类型，设置为direct 直连交换机
$channel->exchange_declare('direct_logs', 'direct', false, false, false);

$severity = isset($argv[1]) && !empty($argv[1]) ? $argv[1] : 'info';

$data = implode(' ', array_slice($argv, 2));
if(empty($data)) $data = "Hello World!";

$msg = new AMQPMessage($data);
//发布消息，第二个参数是发布给哪个交换机，第三个参数是routing key
$channel->basic_publish($msg, 'direct_logs', $severity);

echo " [x] Sent ",$severity,':',$data," \n";
//关闭信道和连接
$channel->close();
$connection->close();

?>
```

##### 6.2 接收消息

```php
<?php

require_once __DIR__ . '/vendor/autoload.php';
use PhpAmqpLib\Connection\AMQPStreamConnection;
//创建连接
$connection = new AMQPStreamConnection('localhost', 5672, 'guest', 'guest');
//创建信道
$channel = $connection->channel();
//创建交换机， 第二个参数是交换机的类型，direct 直连交换机
$channel->exchange_declare('direct_logs', 'direct', false, false, false);
//生成临时队列
list($queue_name, ,) = $channel->queue_declare("", false, false, true, false);

$severities = array_slice($argv, 1);
if(empty($severities )) {
    file_put_contents('php://stderr', "Usage: $argv[0] [info] [warning] [error]\n");
    exit(1);
}
//绑定队列和交换机，第三个参数是routing key 
//该绑定关系也可以写在发布消息的代码中，写在这里的原因是接收消息的代码会先执行
foreach($severities as $severity) {
    $channel->queue_bind($queue_name, 'direct_logs', $severity);
}

echo ' [*] Waiting for logs. To exit press CTRL+C', "\n";

$callback = function($msg){
  echo ' [x] ',$msg->delivery_info['routing_key'], ':', $msg->body, "\n";
};
//接收消息
$channel->basic_consume($queue_name, '', false, true, false, false, $callback);

while(count($channel->callbacks)) {
    $channel->wait();
}
//关闭信道和连接
$channel->close();
$connection->close();

?>
```

### 7. 主题模式(主题交换机)

理解：**主题模式和路由模式其实是类似的，只不过是routing key 从一个单词变成了一段规则（如： \*.orange.\* ，或者 lazy.# ）,多个单词用`.`连接，符合这些规则的都会放入对应的队列中**

规则：

- `*`表示一个单词
- `#`表示任意数量的单词(零个或者多个)

##### 7.1 发布消息

```php
<?php

require_once __DIR__ . '/vendor/autoload.php';
use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Message\AMQPMessage;
//创建连接
$connection = new AMQPStreamConnection('localhost', 5672, 'guest', 'guest');
//创建信道
$channel = $connection->channel();
//创建交换机，第二个参数是交换机的类型，topic为主题交换机
$channel->exchange_declare('topic_logs', 'topic', false, false, false);

$routing_key = isset($argv[1]) && !empty($argv[1]) ? $argv[1] : 'anonymous.info';
$data = implode(' ', array_slice($argv, 2));
if(empty($data)) $data = "Hello World!";

$msg = new AMQPMessage($data);
//发布消息到topic_logs交换机
$channel->basic_publish($msg, 'topic_logs', $routing_key);

echo " [x] Sent ",$routing_key,':',$data," \n";

$channel->close();
$connection->close();

?>
```

##### 7.2 接收消息

```php
<?php

require_once __DIR__ . '/vendor/autoload.php';
use PhpAmqpLib\Connection\AMQPStreamConnection;
//创建连接
$connection = new AMQPStreamConnection('localhost', 5672, 'guest', 'guest');
//创建信道
$channel = $connection->channel();
//创建交换机，第二个参数是交换机的类型，topic为主题交换机
$channel->exchange_declare('topic_logs', 'topic', false, false, false);
//创建临时队列
list($queue_name, ,) = $channel->queue_declare("", false, false, true, false);

$binding_keys = array_slice($argv, 1);
if( empty($binding_keys )) {
    file_put_contents('php://stderr', "Usage: $argv[0] [binding_key]\n");
    exit(1);
}
//绑定队列和交换机，第三个参数是routing key（类似于 a.* 或者 a.#） 
//该绑定关系也可以写在发布消息的代码中，写在这里的原因是接收消息的代码会先执行
foreach($binding_keys as $binding_key) {
    $channel->queue_bind($queue_name, 'topic_logs', $binding_key);
}

echo ' [*] Waiting for logs. To exit press CTRL+C', "\n";

$callback = function($msg){
  echo ' [x] ',$msg->delivery_info['routing_key'], ':', $msg->body, "\n";
};
//接收消息
$channel->basic_consume($queue_name, '', false, true, false, false, $callback);

while(count($channel->callbacks)) {
    $channel->wait();
}
//关闭信道和连接
$channel->close();
$connection->close();

?>
```

### 8. 延时队列

理解：**延时队列是通过`过期时间`和`死信队列`来实现的，当一个队列设置了过期时间(或者队列中的某条消息设置了过期时间)，那么这条消息就会被发送给设置好的死信交换机，然后再通过死信交换机发送给绑定的死信队列，消费者只需消费对应的死信队列就可以实现延时队列**

用处：例如订单超过半小时未支付，自动取消订单，或者其他需要延时处理的任务

##### 8.1 发送消息

```php
//创建连接，创建信道
$connection = new AMQPStreamConnection('127.0.0.1', 5672, 'guest', 'guest');
$channel = $connection->channel();
//创建死信队列，名字为delay_exchange 设置的是直连交换机
$channel->exchange_declare('delay_exchange', 'direct',false,false,false);
//创建普通队列，名称为cache_exchange 设置为直连交换机
$channel->exchange_declare('cache_exchange', 'direct',false,false,false);
//预设值过期之后，发送给哪个死信交换机
$tale = new AMQPTable();
//队列 可以配置 x-dead-letter-exchange 和 x-dead-letter-routing(可选)，分别为设置的哪个死信交换机和哪个死信交换机的routing key
$tale->set('x-dead-letter-exchange', 'delay_exchange');
$tale->set('x-dead-letter-routing-key','delay_exchange');
//给队列设置过期时间，单位为毫秒，1000ms = 1s，所有队列中的消息都是10秒的过期时间
$tale->set('x-message-ttl',10000);
//创建队列，设置为持久化，并且把预设值的参数以最后一个参数传入
$channel->queue_declare('cache_queue',false,true,false,false,false,$tale);
//绑定队列和交换机，设置routing key 为 cache_exchange
$channel->queue_bind('cache_queue', 'cache_exchange','cache_exchange');
//设置死信队列，设置为持久化
$channel->queue_declare('delay_queue',false,true,false,false,false);
//绑定队列和交换机，设置routing key 为 delay_exchange
$channel->queue_bind('delay_queue', 'delay_exchange','delay_exchange');


$msg = new AMQPMessage('Hello World'.$argv[1],array(
    //单独设置消息的过期时间,单位为毫秒，当队列和消息同时设置了过期时间，以最小时间为准
    'expiration' => intval($argv[1]),
    'delivery_mode' => AMQPMessage::DELIVERY_MODE_PERSISTENT

));
//向cache_exchange交换机发布消息
$channel->basic_publish($msg,'cache_exchange','cache_exchange');
echo date('Y-m-d H:i:s')." [x] Sent 'Hello World!' ".PHP_EOL;
//关闭信道和连接
$channel->close();
$connection->close();

```

##### 8.2 接收消息

```php
use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Wire\AMQPTable;
//创建连接和信道
$connection = new AMQPStreamConnection('127.0.0.1', 5672, 'guest', 'guest');
$channel = $connection->channel();
//创建死信交换机和普通交换机，都是直连类型
$channel->exchange_declare('delay_exchange', 'direct',false,false,false);
$channel->exchange_declare('cache_exchange', 'direct',false,false,false);

//创建死信队列，然后绑定
$channel->queue_declare('delay_queue',false,true,false,false,false);
$channel->queue_bind('delay_queue', 'delay_exchange','delay_exchange');

echo ' [*] Waiting for message. To exit press CTRL+C '.PHP_EOL;

$callback = function ($msg){
    echo date('Y-m-d H:i:s')." [x] Received",$msg->body,PHP_EOL;

     $msg->delivery_info['channel']->basic_ack($msg->delivery_info['delivery_tag']);

};

//只有consumer已经处理并确认了上一条message时queue才分派新的message给它
$channel->basic_qos(null, 1, null);
//接收死信队列里的信息
$channel->basic_consume('delay_queue','',false,false,false,false,$callback);


while (count($channel->callbacks)) {
    $channel->wait();
}
//关闭信道和连接
$channel->close();
$connection->close();
```

### 9. 消息确认机制

理解：确认并保证消息被送达，提供了两种方式：发布确认和事务，两者不可同时使用

##### 9.1 发布确认

<!--如果消息发布失败，那么这样写，消息就会丢失，少了一步失败回调，失败回调的目的是把消息返回，然后做二次处理，后期需要补充-->

```php
	//连接rq
    $conn = new AMQPStreamConnection('192.168.2.184',5672,'guest','guest');
    //建立通道
    $channel = $conn->channel();
    //确认投放队列，并将队列持久化
    $channel->queue_declare('hello', false, true, false, false);
    //异步回调消息确认
    $channel->set_ack_handler(
        function (AMQPMessage $message) {
            echo "Message acked with content " . $message->body . PHP_EOL;
        }
    );
    $channel->set_nack_handler(
        function (AMQPMessage $message) {
            echo "Message nacked with content " . $message->body . PHP_EOL;
        }
    );
    //开启消息确认
    $channel->confirm_select();
    //建立消息，并消息持久化
    $msg = new AMQPMessage('kingblanc!',array('delivery_mode' => AMQPMessage::DELIVERY_MODE_PERSISTENT));
    $channel->basic_publish($msg, '', 'hello');

    echo " [x] Sent 'Hello World!'\n";
    //阻塞等待消息确认
    $channel->wait_for_pending_acks();

    $channel->close();
    $conn->close();
```

##### 9.2 事务

理解：当发布消息之后还有业务代码，但是业务代码执行失败，需要事务回滚

```php
//开启事务
$channel->tx_select();
$channel->basic_publish($msg, '', 'task_queue');
//事务提交
$channel->tx_commit();
//事务回滚
$channel->tx_rollback()
```

### 10.消息追踪(日志功能)

消息中心的消息追踪是需要开启Trace来实现的，Trace会记录每一次发送的消息，可通过插件形式提供可视界面，Trace开启后会自动创建交换机`amq.rabbitmq.trace`，每个队列会自动绑定该交换机，绑定后发送到队列的消息就会记录到Trace日志

<!--以下消息需要在命令行执行-->

|                   命令集                    |                 描述                  |
| :-----------------------------------------: | :-----------------------------------: |
|            rabbitmq-plugins list            |             查看插件列表              |
|  rabbitmq-plugins enable rabbitmq_tracing   |             开启trace插件             |
|            rabbitmqctl trace_on             |             打开trace开关             |
|         rabbitmqctl trace_on -p xxx         | 打开trace开关（xxx为日志追踪的vhost） |
|            rabbitmqctl trace_off            |               关闭开关                |
|  rabbitmq-plugins disable rabbitmq_tracing  |         rabbitmq关闭trace插件         |
| rabbitmqctl set_user_tags xxx administrator |   只有administrator才能查看日志结面   |

开启之后，需要在界面上重新登录，页面右上角会出现`tracing`标签，点进去，然后点击`add a new trace`添加一个trace_log文件，之后这个虚拟机中的发送消息的日志信息就会记录在该文件中，点击这个文件，输入用户名密码，就可以查看日志信息

### 11. RabbitMQ集群

##### 11.1 集群方案原理

**在整个集群中，交换机的元数据信息在所有节点中是一致的，而队列的完整数据只会存在它所创建的哪个节点上，其他节点只知道这个队列的metadata信息和一个指向队列的所属节点(owner node)的指针**

集群会同步四种类型的元数据

- 队列元数据：队列名称和属性
- 交换机元数据：交换机的名称类型和属性
- 绑定元数据： 一张简单的表格展示了如何将消息路由到队列
- vhost元数据： vhost内的队列、交换器和绑定提供命名空间和安全属性 

rabbitmq采用这种方式，而不是把队列中的完整数据保存到每个节点，原因还是基于性能和存储空间考虑的，同步完整数据会造成每个节点的已用存储空间都很大，集群的消息积压能力就会很弱，而且对持久化消息、网络和磁盘同步复制的开销就会变大

用户访问某个节点，当发布的队列在这个节点上，就可以直接发布或者消费，当不在这个节点上时，这个节点就会起到一个路由转发的作用，转发到这个队列所在的节点，从而实现发布和消费

##### 11.2 准备工作

首先要确保所在服务器的rabbitmq没有运行，可以通过`ps aux |grep rabbitmq`来查看进程或者通过`rabbitmqctl status`是否有运行的节点

##### 11.3 单机多实例搭建

假如要搭建两个rabbitmq节点，分别为rabbitmq-1,rabbitmq-2,1为主节点，2为从节点

- 启动第一个节点

  ```shell
  //后台启动
  sudo RABBITMQ_NODE_PORT=5672 RABBITMQ_NODENAME=rabbitmq-1 rabbitmq-server start &
  ```

- 启动第二个节点

  ```shell
  //因为是在一个服务器上启动，所以端口号要改一下，而且web管理插件的端口号也要改一下，如果是不同服务器就可以不用该端口号
  sudo RABBITMQ_NODE_PORT=5673 RABBITMQ_START_ARGS="rabbitmq_management listener [{port,15673}]" RABBITMQ_NODENAME=rabbitmq-2 rabbitmq-server start &
  ```

- 通过`ps aux | grep rabbitmq`来查看两个节点是否正常启动

- 以rabbitmq-1为主节点，命令如下

  ```shell
  //停止应用
  sudo rabbitmqctl -n rabbit-1 stop_app
  //重置节点，为了是清理节点上的历史数据，不清理无法加入节点
  sudo rabbitmqctl -n rabbit-1 reset
  //启动应用
  sudo rabbitmqctl -n rabbit-1 start_app
  ```

- rabbit-2作为从节点，命令如下

  ```shell
  //停止应用
  sudo rabbitmqctl -n rabbit-2 stop_app
  //重置节点，为了是清理节点上的历史数据，不清理无法加入节点
  sudo rabbitmqctl -n rabbit-2 reset
  //将rabbitmq-2以从节点的身份加入集群，xxx为主节点所在服务器的主机名
  sudo rabbitmqctl -n rabbit-2 join_cluster rabbit-1@'xxx'
  //启动应用
  sudo rabbitmqctl -n rabbit-2 start_app
  ```

- 查看集群状态

  ```shell
  sudo rabbitmqctl cluster_status -n rabbit-1
  ```

##### 11.4 多机多节点部署

- 需要读取其中一个节点的cookie,并复制到其他节点(节点之间通过cookie确定互相之间是否可通信),  文件位置在 /var/lib/rabbitmq/.erlang.cookie 可以将这个文件使用 scp传到各个节点的相同位置，也可以修改其中的值

- 配置个节点所在服务器的hosts文件(/etc/hosts)

  ```
  //类似这样的
  192.168.1.116    rabbitmq-1
  192.168.1.117    rabbitmq-2
  192.168.1.118    rabbitmq-3
  ```

- 其他步骤跟单机多节点雷同

##### 11.5 集群监控

- 管理结面监控：管理结面监控需要开启对应插件,命令:`rabbitmq-plugins enable rabbitmq_management`,然后在浏览器输入http://ip:15672进入，在管理控制台就可以看到每个节点是否正常，绿色的是正常，红色的是不正常，还可以看各节点的内存、磁盘信息、进程数、文件数量等，但是该功能比较简陋，没有告警等一系列个性化设置，所以扩展性不强，一边小公司的小型集群使用

- 通过tracing插件来实现对消息的监控，详见第10章消息追踪

- 采用RabbitMQ的 http API接口来进行监控： 这些接口是Restful风格的， HTTP API接口来获取各种业务监控需要的实时数据 ， 当然，这个接口的作用远不止于获取一些监控数据，也可以通过这些HTTP API来操作RabbitMQ进行各种集群元数据的添加/删除/更新的操作。 

  > 可以自己写相关代码来实现，或者在网上找，我暂时没找到...

   下面列举了可以利用RabbitMQ的HTTP API接口实现的各种操作（更多接口可以访问http://ip:15672/api/ 查看）： 

  |             HTTP API URL              | HTTP 请求类型  |                           接口含义                           |
  | :-----------------------------------: | :------------: | :----------------------------------------------------------: |
  |           /api/connections            |      GET       |             获取当前RabbitMQ集群下所有打开的连接             |
  |              /api/nodes               |      GET       |         获取当前RabbitMQ集群下所有节点实例的状态信息         |
  |    /api/vhosts/{vhost}/connections    |      GET       |       获取某一个虚拟机主机下的所有打开的connection连接       |
  |   /api/connections/{name}/channels    |      GET       |                获取某一个连接下所有的管道信息                |
  |     /api/vhosts/{vhost}/channels      |      GET       |               获取某一个虚拟机主机下的管道信息               |
  |        /api/consumers/{vhost}         |      GET       |            获取某一个虚拟机主机下的所有消费者信息            |
  |        /api/exchanges/{vhost}         |      GET       |           获取某一个虚拟机主机下面的所有交换器信息           |
  |          /api/queues/{vhost}          |      GET       |             获取某一个虚拟机主机下的所有队列信息             |
  |              /api/users               |      GET       |                   获取集群中所有的用户信息                   |
  |           /api/users/{name}           | GET/PUT/DELETE |                  获取/更新/删除指定用户信息                  |
  |     /api/users/{user}/permissions     |      GET       |                获取当前指定用户的所有权限信息                |
  |    /api/permissions/{vhost}/{user}    | GET/PUT/DELETE |          获取/更新/删除指定虚拟主机下特定用户的权限          |
  | /api/exchanges/{vhost}/{name}/publish |      POST      |           在指定的虚拟机主机和交换器上发布一个消息           |
  |    /api/queues/{vhost}/{name}/get     |      POST      | 在指定虚拟机主机和队列名中获取消息，同时该动作会修改队列状态 |
  |     /api/healthchecks/node/{node}     |      GET       |                  获取指定节点的健康检查状态                  |

- Zabbix 监控RabbitMQ：搭建配置要求比较高，一般都是由运维人员负责搭建，有兴趣的可以访问官网了解学习

### 12. RabbitMQ高可用集群

##### 12.1 常见的高可用集群方案

- 主备模式：一般在并发和数据量不多的情况下使用，当主节点挂掉以后从备节点中选择一个作为主节点对外提供服务，缺点是：主备切换的时候会造成业务不可用，尤其是在高并发情况下
- **镜像队列模式：又称Mirror队列，主要保证mq消息的可靠性，它通过消息复制的方式保证消息100%不丢失，该模式也是企业中使用最多的模式**
- 多活模式：该模式主要是用来实现异地数据复制，使用多活模式需要借助federation插件来实现集群和集群之间或者节点和节点之间的数据复制，该模式广泛应用于饿了么、美团、滴滴等企业（一般都是多活配合镜像）

##### 12.2 镜像队列集群搭建

- 集群节点规划

  |     ip地址     |       用途        | 主机名  |
  | :------------: | :---------------: | :-----: |
  | 192.168.13.101 |     mq主节点      | server1 |
  | 192.168.13.102 |      从节点       | server2 |
  | 192.168.13.103 |      从节点       | server3 |
  | 192.168.13.104 | HAProxy KeepAlive | server4 |
  | 192.168.13.105 | HAProxy KeepAlive | server5 |

- 复制主节点的cookie信息到从节点

  ```shell
  #首先需要配置hosts文件，在各个服务器ping其他服务器的名称都能通
  #首先要停止mq的运行 centos7 可以用systemctl stop rabbitmq-server
  /etc/init.d/rabbitmq-server stop 
  #复制cookie文件
  scp /var/lib/rabbitmq/.erlang.cookie 192.168.13.102 /var/lib/rabbitmq
  scp /var/lib/rabbitmq/.erlang.cookie 192.168.13.102 /var/lib/rabbitmq
  #然后要记得修改权限
  chmod 400 /var/lib/rabbitmq/.erlang.cookie
  ```

- 启动主节点

  ```shell
  #centos7 
  systemctl stop rabbitmq-server
  systemctl start rabbitmq-server
  #centos6
  rabbitmqctl stop
  rabbitmqctl start_app
  #注意！！记得重置节点，否则加不进去集群
  rabbitmqctl reset
  ```

- 将从节点添加到主节点的集群中

  ```shell
  #如果从节点没有启动mq服务，需要先启动服务,因为不启动服务，rabbitmqctl命令是不能用的
  systemctl start rabbitmq-server
  #然后停掉app
  rabbitmqctl stop_app
  #需要重置节点
  rabbitmqctl reset
  #server2以从节点的身份加入集群，--ram 指定消息以内存方式存储，不指定为存储到磁盘,因为内存性能更好，但是为了保证高可用性，必须保证两个节点以上存储在磁盘
  rabbitmqctl join_cluster --ram rabbit@server1
  #启动应用
  rabbitmqctl start_app
  #server3以同样的方法加入集群
  #然后查看集群状态
  rabbitmqctl cluster_status
  ```

- 设置镜像队列策略

  ```shell
  #将所有队列设置为镜像队列，主节点执行
  rabbitmqctl set_policy ha-all "^" '{"ha-model":"all"}'
  ```

  这样就完成了！

  如需修改集群信息

  ```shell
  #修改集群名称，在任何一个节点执行都可以
  rabbitmqctl set_cluster_name xxx
  #在非server2的节点上可以移除server2节点
  rabbitmqctl forget_cluster_node rabbit@server2
  ```

##### 12.3 HAProxy 实现队列的负载均衡

> HAProxy简介
>
> HAProxy提供高可用性、负载均衡以及基于TCP和HTTP应用的代理，支持虚拟主机，它是免费、快速并且可靠的一种解决方案。根据官方数据，其最高极限支持10G的并发。HAProxy支持从4层至7层的网络交换，即覆盖所有的TCP协议。就是说，Haproxy 甚至还支持 Mysql 的均衡负载。为了实现RabbitMQ集群的软负载均衡，这里可以选择HAProxy

HAProxy的规划(2台的原因是为后面高可用做准备)

| ip             | 用途    | 主机名  |
| -------------- | ------- | ------- |
| 192.168.13.104 | HAProxy | server4 |
| 192.168.13.105 | HAProxy | server5 |

安装

```shell
#下载依赖包，已安装的可以跳过
yum install gcc vim wget
#下载HAProxy
wget http://www.haproxy.org/download/1.7/src/haproxy-1.7.5.tar.gz
#解压到/usr/local目录
tar -zxvf haproxy-1.7.5.tar.gz -C /usr/local
#进入目录，编译，安装
cd /usr/local/haproxy-1.7.5
#参数说明
#TARGET=linux26 #内核版本，使用uname -r查看内核，如：2.6.18-371.el5，此时该参数就为linux26；kernel 大于2.6.28的用：TARGET=linux2628
#ARCH=x86_64 #系统位数
#PREFIX=/usr/local/haprpxy  #/usr/local/haprpxy为haprpxy安装路径
make TARGET=linux2628 ARCH=x86_64 PREFIX=/usr/local/haproxy
make install PREFIX=/usr/local/haproxy
#用来存放配置文件
mkdir /etc/haproxy
#赋权
groupadd -r -g 149 haproxy
useradd -g haproxy -r -s /sbin/nologin -u 149 haproxy
#创建haproxy配置文件
touch /etc/haproxy/haproxy.cfg
```

配置haproxy.cfg

```shell
#全局配置
global
        #日志输出配置，所有日志都记录在本机，通过local0输出
        log 127.0.0.1 local0 info
        #最大连接数
        maxconn 4096
        #改变当前的工作目录
        chroot /usr/local/haproxy
        #以指定的UID运行haproxy进程
        uid 99
        #以指定的GID运行haproxy进程
        gid 99
        #以守护进程方式运行haproxy #debug #quiet
        daemon
        #进程数量 5
        nbproc 5
        #debug
        #当前进程pid文件
        pidfile var/run/haproxy.pid

#默认配置
defaults
        #应用全局的日志配置
        log global
        #默认的模式mode{tcp|http|health}
        #tcp是4层，http是7层，health只返回OK
        mode tcp
        #日志类别tcplog
        option tcplog
        #不记录健康检查日志信息
        option dontlognull
        #3次失败则认为服务不可用
        retries 3
        #每个进程可用的最大连接数
        maxconn 2000
        #连接超时
        timeout connect 5s
        #客户端超时
        timeout client 30s
        #服务端超时
        timeout server 30s

#绑定配置
listen rabbitmq_cluster
		#注意此处的ip!!!!!!!!!!
        bind 192.168.13.104:5672
        #配置TCP模式
        mode tcp
        #加权轮询
        balance roundrobin
        #RabbitMQ集群节点配置, inter 每隔5秒对集群做一次健康检查，2次正确证明服务器可用，2次失败证明服务器不可用 
        server server1 192.168.13.101:5672 check inter 5000 rise 2 fall 3 weight 1
        server server2 192.168.13.102:5672 check inter 5000 rise 2 fall 3 weight 1
        server server3 192.168.13.103:5672 check inter 5000 rise 2 fall 3 weight 1

#haproxy监控页面地址
listen monitor
		#注意此处的ip!!!!!!!!!!
        bind 192.168.13.104:8100
        mode http
        option httplog
        stats enable
        #设置haproxy的监控地址http://192.168.13.104:8100/rabbitmq-status
        stats uri /rabbitmq-stats
        stats refresh 5s
```

启动HAProxy

```shell
/usr/local/haproxy/sbin/haproxy -f /etc/haproxy/haproxy.cfg
#查看进程
ps -aux | grep haproxy
```

然后就可以在网页上查看haproxy状态(如果不能访问，看下防火墙是否关闭)

然后另一台105的服务器也这样设置，**注意修改配置文件中的ip**

**注意：代码中的rabbitmq配置的host要是192.168.13.104！！！！！！！**

##### 12.4  KeepAlived搭建高可用的HAProxy集群

> 简介
>
> KeepAlived,它是一个高性能的服务器高可用或者热备的解决方案，KeepAlived主要用来防止服务器单点故障的发生，通过和Nginx、HAproxy等反向代理的负载均衡服务器配合实现web服务器的高可用。Keepalived采用VRRP（Virtual Router Redundancy Protocol，虚拟路由冗余协议），以软件的形式实现服务器热备功能。通常情况下是将两台Linux服务器组成一个热备组（Master和Backup），同一时间热备组内只有一台主服务器Master提供服务，同时Master会虚拟出一个公用的虚拟IP地址，简称VIP。这个VIP只存在在Master上并对外提供服务。如果Keepalived检测到Master宕机或者服务故障，备份服务器Backup会自动接管VIP称为Master，Keepalived并将原Master从热备组中移除。当原Master恢复后，会自动加入到热备组，默认再抢占称为Master，起到故障转移的功能。

KeepAlived的安装

```shell
#安装索需软件包
yum install -y openssl openssl-devel
#下载安装包
wget http://www.keepalived.org/software/keepalived-1.2.18.tar.gz
#解压编译安装
tar -zxvf keepalived-1.2.18.tar.gz -C /usr/local
cd /usr/local/keepalived-1.2.18/ && ./configure --prefix=/usr/local/keepalived
make && make install
#将keepalived安装成linux系统服务，因为没有使用keepalived默认的安装路径(/usr/local)，安装完成后需要做一些修改工作
#首先创建文件夹，将keepalived配置文件复制
mkdir /etc/keepalived
cp /usr/local/keepalived/etc/keepalived/keepalived.conf /etc/keepalived
#然后复制keepalived的脚本文件
cp /usr/local/keepalived/etc/rc.d/init.d/keepalived /etc/init.d/
cp /usr/local/keepalived/etc/sysconfig/keepalived /etc/sysconfig/
#设置的过程中如果出现已经存在则删除原有文件,重新创建
ln -s -f /usr/local/sbin/keepalived /usr/sbin/
ln -s -f /usr/local/keepalived/sbin/keepalived /sbin/
#可以设置开机启动
chkconfig keepalived on 
#安装完成
```

keepalived的配置

vim /etc/keepalived/keepalived.conf

```shell
global_defs {
   # 标识节点的字符串，通常为本机主机名
   router_id server4
}

# 自定义监控脚本
vrrp_script chk_haproxy {
    # 脚本位置
    script "/etc/keepalived/haproxy_check.sh" 
    # 脚本执行的时间间隔
    interval 2
    #如果条件成立则权重-20
    weight -20
}

vrrp_instance VI_1 {
    # Keepalived的角色，MASTER 表示主节点，BACKUP 表示备份节点(重要)
    state MASTER  
    # 指定监测的网卡，可以使用 ifconfig/ip addr 进行查看
    interface ens32
    # 虚拟路由的id，主备节点需要设置为相同（重要）
    virtual_router_id 110
    #本机的ip地址
    mcast_scp_ip 192.168.104
    # 优先级，主节点的优先级需要设置比备份节点高
    priority 100 
    # 设置主备之间的检查时间，单位为秒 
    advert_int 1 
    # 定义验证类型和密码
    authentication { 
        auth_type PASS
        auth_pass 123456
    }

    # 调用上面自定义的监控脚本
    track_script {
        chk_haproxy
    }

    virtual_ipaddress {
        # 虚拟IP地址，可以设置多个
        192.168.13.110
    }
}

```

编写执行脚本(一定要赋权否则不能执行)

```shell
#添加文件位置为/etc/keepalived/haproxy_check.sh
#! /bin/bash
COUNT= `ps -C haproxy --no-header |wc -l`
if [COUNT -eq 0];then
	/usr/local/haproxy/sbin/haproxy -f /etc/haproxy/haproxy.cfg
	sleep 2
	if [ `ps -C haproxy --no-header |wc -l` -eq 0];then
		killall keepalived
	fi
fi
```

赋权给脚本

```shell
chmod +x /etc/keepalived/haproxy_check.sh
```

启动keepalived

```shell
#启动
service keepalived start | stop | restart | status
#查看状态
ps -aux |grep haproxy
ps -aux |grep keepalived
```

**105服务器也按着这个步骤配置一遍，注意keepalived.conf里面的设置要适当修改**

在代码中，我们连rabbitmq 的host 就要连哪个虚拟的 ip `192.168.13.110`



### 13. 常见面试问题

##### 13.1 消息堆积

- 排查消费者的消费性能瓶颈，比如数据库卡了等
- 部署多个消费者

##### 13.2 消息丢失

消息丢失可能在三个地方丢失：`生产者`、`MQ`、`消费者`，一般是网络不稳定造成的

生产者：通过发布确认机制来确保消息发送成功（确认机制有三种，普通确认机制，批量确认机制，异步监听确认机制），如果失败还可以继续发（confirm_select()）

MQ: 一般是服务器宕机或者重启，就需要持久化交换机、队列、消息 （declare的时候一般第三个参数就是设置是否持久化）

消费者：一般是消费者接受完消息，然后处理业务逻辑，这时候业务服务器宕机或者重启，那么消息就会丢失(因为MQ已经删除了这条消息)，这就需要手动回复MQ服务器，而且是在尽量业务逻辑执行完成之后(basic_ack)

##### 13.3 有序消费

- 场景1：三个消息1,2,3依次发送到队列，而且有三个消费者，希望三个消费者依次消费，但是由于每个消费者消费时间差异，会造成消费者乱序的问题

  这种问题，可以创建三个队列，每个消费者对应一个队列，然后对依次传入的消息标识(如商品id)算一个hash 然后对队列个数取余，这样就可以让相同id的消息放在对应的队列里，然后对应的消费者消费

- 场景2：三个消息1,2,3依次发送到队列，只有一个消费者，但是消费者开了三个线程来处理，这样也会造成乱序消费的问题

  这个问题，消费者拉出消息之后根据id算一个hash然后对线程个数取余，然后压入同一个内存队列中，让同一个线程去处理

##### 13.4  重复消费

场景：当消费者接受完消息，而且处理成功，但是在未确认之前，服务器宕机或者重启造成消费者挂掉，那么，MQ再次发送消息给消费者，这样就有可能造成重复消费

解决：可以考虑接口幂等性的操作，每次消费成功后都要保存该消息id到数据库，然后每次消费之前都要查询该id是否消费过，更好的方案是通过redis 存储消息id,因为redis是单线程的，性能好，提供了很多原子性的操作，可以使用 `setnx`命令存储消息id

> ​	setnx(key,value) :如果key不存在则插入成功且返回1，如果存在，不进行任何操作，返回0

