### RabbitMQ相关概念

RabbitMQ整体上是一个生产者与消费者模型，主要负责接收、存储和转发消息。从计算机术语层面来说RabbitMQ模型更像是一种交换器模型。Producer:生产者，就是投
递消息的一方。生产者创建消息，然后发布到RabbitMQ中。消息一般可以包含2个部分：消息体和标签。消息体也可以称之为payload,在实际应用中，消息体一般是一个
带有业务逻辑结构的数据，比如JSON字符串。消息的标签用来表述这条消息，比如一个交换器的名称和一个路由键。生产者把消息交由RabbitMQ，RabbitMQ之后会根据
标签把消息发送到感兴趣的消费者。Consumer:消费者，就是接受消息的一方。消息者连接到RabbitMQ服务器，并订阅到队列上。当消费者消费一条消息时，只是消费者
的消息体(payload)。在消息路由的过程中，消息的标签会丢失，存入到队列中的消息只要消息体，消费者也只会消费到消息体，并不知道生产者是谁。Broker:消息中
间件的服务节点。对于RabbitMQ来说，一个RabbitMQ Broker可以简单地看成一个RabbitMQ服务节点，或者RabbitMQ服务实例。Queue:队列，是RabbitMQ内部对象，
用于存储消息。RabbitMQ中消息都只能存储在队列中，这一点和kafka这种消息中间件相反。Kafka将消息信息存储在topic主题这个逻辑层面，而相对应的队列逻辑只
是topic实际存储文件中的位移标识。RabbitMQ的生产者生产消息并最终投递到队列中，消费者可以从队列中获取消息并消费。多个消费者可以订阅同一个队列，这时队
列中的消息会被平均分摊(Round-Robin，即轮询)给多个消费者进行处理，而不是每个消费者都收到所有的消息并处理。**RabbitMQ不支持队列层面的广播消息，如果需
要在其上进行二次开发，处理逻辑会变得异常复杂，同时也不建议这么做。**Exchange:交换器，生产者将消息发送到Exchange，由路由器将消息路由到一个或者多个队
列中。如果路由不到，或许会返回给生产者，或许直接丢弃。RoutingKey：路由键，生产者将消息发送给交换器的时候，一般会指定一个RoutingKey，用来指定这个消
息的路由规则，而这个RoutingKey需要与交换器类型和绑定键(BingingKey)联合使用才能最终生效。Binding：绑定。RabbitMQ中通过绑定将交换器与队列关联起来，
在绑定的时候一般会指定一个绑定键(BindingKey)，这样RabbitMQ就知道如何正确地将消息路由到队列中了。在绑定多个队列到同一个交换器的时候，这些绑定允许使
用相同的BindingKey。BindingKey并不是在所有的情况下都生效，它依赖于交换器类型，比如fanout类型的交换器就会无视BindingKey，而是将消息路由到所有绑定
到该交换器的队列中。打个简单地比喻，交换器相当于投递包裹的邮箱，RoutingKey相当于填写在包裹上的地址，BindingKey相当于填写在包裹上的地址，BindingKey
相当于包裹的目的地，当填写在包裹上的地址和实际想要投递的地址相匹配时，那么这个包裹就会被正确投递到目的地，最后这个目的地的“主人”————队列可以保留这个
包裹。

###### AMQP协议
AMQP 协议中的基本概念：
* Broker: 接收和分发消息的应用，我们在介绍消息中间件的时候所说的消息系统就是Message Broker。
* Virtual host: 出于多租户和安全因素设计的，把AMQP的基本组件划分到一个虚拟的分组中，类似于网络中的namespace概念。当多个不同的用户使用同一个
RabbitMQ server提供的服务时，可以划分出多个vhost，每个用户在自己的vhost创建exchange／queue等。
* Connection: publisher／consumer和broker之间的TCP连接。断开连接的操作只会在client端进行，Broker不会断开连接，除非出现网络故障或broker服务出现
问题。
* Channel: 如果每一次访问RabbitMQ都建立一个Connection，在消息量大的时候建立TCP Connection的开销将是巨大的，效率也较低。Channel是在connection内部
建立的逻辑连接，如果应用程序支持多线程，通常每个thread创建单独的channel进行通讯，AMQP method包含了channel id帮助客户端和message broker识别
channel，所以channel之间是完全隔离的。Channel作为轻量级的Connection极大减少了操作系统建立TCP connection的开销。
* Exchange: message到达broker的第一站，根据分发规则，匹配查询表中的routing key，分发消息到queue中去。常用的类型有：direct (point-to-point), 
topic (publish-subscribe) and fanout (multicast)。
* Queue: 消息最终被送到这里等待consumer取走。一个message可以被同时拷贝到多个queue中。
* Binding: exchange和queue之间的虚拟连接，binding中可以包含routing key。Binding信息被保存到exchange中的查询表中，用于message的分发依据。

###### 交换器类型
RabbitMQ常用的交换器类型有fanout、direct、topic和headers这四种。AMQP协议里提到两个类型：System和自定义，一般不用，下面对这四种类型进行一一描述：
1. fanout：它会把所有发送到该交换器的消息路由到所有与该交换器绑定的队列中；
2. direct：它会把消息路由到那些BindingKey和RoutingKey完全匹配的队列中；
3. topic：它相对direct类型路由器在匹配规则上进行了扩展，BindingKey和RoutingKey用一个点号`.`分隔字符串，被点号分隔的每一段独立的字符串称为一个单词
，而BindingKey还可以使用特殊字符`*和#`进行模糊匹配，其中"*"用于匹配一个单词，“#”用于匹配多规则单词(可以是零个)。
4. headers：它不依赖于路由键的匹配规则来路由消息，而是根据发送消息内容中的headers属性进行匹配。在绑定队列和交换器时定制了一组键值对，当发送消息到交
换器时，RabbitMQ会获得到该消息的headers(也是一个键值对形式)，对比其中的键值对是否完全匹配队列和交换器绑定时指定的键值对，如果匹配则将该消息路由到该
队列。

###### 连接与信道
RabbitMQ中无论是生产者还是消费者都需要和RabbitMQ Broker建立连接，这个连接就是一条TCP连接，也就是Connection。一旦TCP连接建立起来，客户端紧接着可以
创建一个AMQP信道(channel)，每个信道都会被指定一个唯一的ID。信道时建立在connection基础之上的虚拟连接，RabbitMQ处理的每条AMQP指令都是通过信道完成的
。为什么我们需要一个虚拟信道的概念，试想一下，一个应用程序中很多个线程需要从RabbitMQ中消费消息，或者生产消息，那么必然需要建立很多个connection，也
就是许多个TCP连接。然而对操作系统而言，建立和销毁TCP连接时非常大的开销，如果遇到使用高峰，性能压力非常大。RabbitMQ采用类似NIO(Non-Blocking IO)的
做法，选择TCP连接复用，不仅可以减少开销而且便于管理。

###### RabbitMQ安装
1. 第一步：因为RabbitMQ是使用erlang开发的，所以安装RabbitMQ前需要安装erlang语言包，添加erlang安装源，在centOS7上安装最新版
```
# In /etc/yum.repos.d/rabbitmq_erlang.repo
[rabbitmq_erlang]
name=rabbitmq_erlang
baseurl=https://packagecloud.io/rabbitmq/erlang/el/7/$basearch
repo_gpgcheck=1
gpgcheck=1
enabled=1
# PackageCloud's repository key and RabbitMQ package signing key
gpgkey=https://packagecloud.io/rabbitmq/erlang/gpgkey
       https://dl.bintray.com/rabbitmq/Keys/rabbitmq-release-signing-key.asc
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
metadata_expire=300

[rabbitmq_erlang-source]
name=rabbitmq_erlang-source
baseurl=https://packagecloud.io/rabbitmq/erlang/el/7/SRPMS
repo_gpgcheck=1
gpgcheck=0
enabled=1
# PackageCloud's repository key and RabbitMQ package signing key
gpgkey=https://packagecloud.io/rabbitmq/erlang/gpgkey
       https://dl.bintray.com/rabbitmq/Keys/rabbitmq-release-signing-key.asc
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
metadata_expire=300
```
安装命令：`yum install erlang`
2. 第二步：安装RabiitMQ.添加RabbitMQ安装源
```
[bintray-rabbitmq-server]
name=bintray-rabbitmq-rpm
baseurl=https://dl.bintray.com/rabbitmq/rpm/rabbitmq-server/v3.8.x/el/7/
gpgcheck=0
repo_gpgcheck=0
enabled=1
```
安装命令：
```
#导入公共数据签名
rpm --import https://www.rabbitmq.com/rabbitmq-release-signing-key.asc
#安装RabbitMQ server
yum install rabbitmq-server.noarch
```
