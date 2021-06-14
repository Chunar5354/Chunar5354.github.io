---
layout: article
tags: middleware RabbitMQ
title: RabbitMQ进阶
article_header:
  type: overlay
  theme: dark
  background_color: '#123'
  background_image: false
---

RabbitMQ消息队列的一些特性和用法

<!--more-->

## 基础的消息收发

使用文末的golang demo可以实现简单的消息收发，当通过publisher发送一条消息后，在服务器上可以查看：

```
$ rabbitmqctl list_queues -p vChunar
Timeout: 60.0 seconds ...
Listing queues for vhost vChunar ...
name	messages
chunar_q	1
```

说明此时chunar_q队列中有一条消息等待消费

调用comsumer消费一条消息后，结果变成

```
Timeout: 60.0 seconds ...
Listing queues for vhost vChunar ...
name	messages
chunar_q	0
```

## 更高级的用法

### 消息确认

在上一节的例子中可以看到消费者取出一条消息之后，队列里就会减少一条消息，这是因为我们开启了消费者的`自动确认`模式，让rabbitmq服务器收到了消费者返回的确认，就会删除一条消息

假如关闭自动确认的话：

```go
msgs, err := r.channel.Consume(
	r.QueueName, // 队列名
	"",          // 消费者名
	false,        // ACK，是否自动应答
	...
)
```

再按照上面的流程消费一条消息，发现队列中的消息数量没有减少：

```
$ rabbitmqctl list_queues -p vChunar
Timeout: 60.0 seconds ...
Listing queues for vhost vChunar ...
name	messages
chunar_q	1
```

在rabbitmq中，默认是采取`手动消费确认`的方式，因为自动确认发生在消费者`接收到消息`之后，如果后面的处理发生了错误，那么这条消息即使没有被处理成功还是返回了确认，这就出现了异常

在demo中开启手动确认:

```go
// 处理消息
go func() {
	for d := range msgs {
		// 接收到消息后的处理
		log.Printf("Received a message: %s", d.Body)
		fmt.Println(d.Body)
		d.Ack(false) // false表示只确认当前消息
	}
}()
```

再消费一次，发现队列中已经没有了消息

```
Timeout: 60.0 seconds ...
Listing queues for vhost vChunar ...
name	messages
chunar_q	0
```

### 消息限流

有时消费者的消费速度可能跟不上生产者发送消息的速度，就会导致队列中有大量消息堆积，这时当一个消费者进行消费的时候，就会`一下接收到大量的数据`，给消费端性能带来影响

rabbitmq实现一种QOS（Quality Of Service，服务质量保证）功能，在消费者`未确认之前`限制发给消费者的消息数量

在comsumer中添加代码，设置一次最多消费2条信息：

```go
// 消费者限流
r.channel.Qos(
	2,     //当前消费者在未确认之前一次最多接受2条消息
	0,     //服务器传递的最大容量
	false, //如果为true 对整个channel可用 ，false则只对当前队列可用
)
```

同时要`关闭应答`（注释掉d.ACK那一行）


向chunar_q队列中发送5条消息：

```
name	messages
chunar_q	5
```

在命令行没有找到能够区分未确认消息的方法，但可以在图形控制台查看：

[![24rOZF.png](https://z3.ax1x.com/2021/06/12/24rOZF.png)](https://imgtu.com/i/24rOZF)

此时有5条就绪（Ready）消息

然后运行consumer进行一次消费，然后查看：

[![24sRQx.png](https://z3.ax1x.com/2021/06/12/24sRQx.png)](https://imgtu.com/i/24sRQx)

此时就绪（Ready）消息变成了3条，因为有2条消息被消费但是未被应答（Unacked）

这样就实现了消息限流的功能

此外这里还有一个有趣的现象是就是rabbitmq服务器会在消费者`运行期间`且`未返回确认时`将当前消费的消息标记为Unacked，此时其他消费者`不能够获取`到Unacked的消息，而在消费者连接断开之后又会重新将Unacked的消息标记成为`Ready`，其他消费者又可以获取到这些消息

[![25s9jP.png](https://z3.ax1x.com/2021/06/12/25s9jP.png)](https://imgtu.com/i/25s9jP)

从这里可以看出消费者一定要正确的对消息进行确认，否则会出现同一个消息被多次获取以及消息顺序的混乱等问题

### 持久化

为了保证在突发故障时数据不会丢失，rabbitmq也提供了持久化的手段

有三个元素可供用户自定义是否持久化：Exahcnge、Queue和Message，它们是`递进`的关系，也就是说要想Message实现持久化，就必须使得Exchange和Queue都实现持久化

在本文的Demo中使用的是默认Exchange，它`默认是持久化`的，而demo中的Queue和Message都使用了非持久化的方式，重启一下rabbitmq：

```
$ rabbitmqctl stop_app
$ rabbitmqctl start_app
```

发现之前的Queue都没有了：

```
$ rabbitmqctl list_queues -p vChunar
Timeout: 60.0 seconds ...
Listing queues for vhost vChunar ...
```

可以改成持久化的方式，对于Queue，要在使用`QueueDeclare()`方法的时候指定（生产者和消费者都要修改）：

```go
_, err := r.channel.QueueDeclare(
	r.QueueName, // name，队列名
	true,       // durable，是否持久化
	...
)
```

对于Message，则要在`Publish()`方法中指定：

```go
r.channel.Publish(
	...
	amqp.Publishing{
		DeliveryMode: amqp.Persistent,
		ContentType: "text/plain",
		Body:        []byte(message),
	},
)
```

再重新发布一条消息，然后重启rabbitmq，再次查看，发现消息仍然存在，并可以正常被消费：

```
$ rabbitmqctl list_queues -p vChunar
Timeout: 60.0 seconds ...
Listing queues for vhost vChunar ...
name	messages
chunar_q	1
```

### 交换机类型

rabbitmq的交换机有4种类型：direct，fanout，topic和headers，每一种都对应消息队列不同的工作模式：

- direct，消费者-生产者模式，文末的demo就是此种模式

direct类型的交换机将绑定的`路由键`按照`完全匹配`的方式路由到指定队列

[direct交换机的demo](https://chunar5354.github.io/2021/06/13/rabbit-direct.html)

- fanout，发布-订阅模式

fanout类型的交换机会将消息`广播`到交换机绑定的所有队列上，无视路由键

[fanout交换机的demo](https://chunar5354.github.io/2021/06/13/rabbit-fanout.html)

- topic，主题订阅模式

topic类型的交换机会将路由键按照`.`进行分割，然后利用分割后的元素进行匹配，同时支持模糊匹配，`*`表示任意单词，`#`匹配多个单词

- headers，根据消息的headers信息进行路由匹配，应用较少，在此不做介绍

关于更详细的rabbitmq工作模式信息，建议阅读[官方示例](https://www.rabbitmq.com/getstarted.html)

### 一些属性的解读

通过代码可以看到在创建交换机、创建队列、发布消息、接收消息等方法中都接收一大堆参数，那么这些参数都表示什么意义呢？下面对一些常用的参数进行说明：

完整的说明可以查看[官方文档](https://pkg.go.dev/github.com/streadway/amqp?utm_source=godoc#Channel)

一些通用的属性：

- `durable`，持久化，持久化的交换机、队列或消息会保存在硬盘上，在rabbitmq故障或重启后`不会丢失`

- `autoDelete`，自动删除，根据rabbitmq的架构，消息传递的顺序是`publisher -> exchange -> queue -> channel -> consumer`，自动删除属性指的是如果一个成员`后面与它相连的`成员全部清除，该成员就被自动删除。举例来说，一个autoDelete的exchange绑定的队列全部被删除后，这个exchange就会被自动删除

- `noWait`，如果设为true，在新建时会等待一个rabbitmq服务器的响应（似乎不影响使用）

交换机的属性：

- `internal`，内部交换机，内部交换机`只能绑定到另一个交换机`，而不能接收publisher的消息，有点像内部路由器

队列的属性：

- `exclusive`，排他性，具有排他性的队列`只能`在创建它的connection中使用，当这个connection断开后，队列会被`删除`

消费消息时：

- `autoAck`，自动确认，消费者会在接到消息的同时立刻返回ACK，如果消费者对消息处理失败可能会丢失信息，所以不建议使用autoAck

- `noLocal`，rabbitmq不支持

### TTL消息/队列

rabbitmq中可以为一整个队列或单独的某条消息设置`过期时间`

- 设置整个队列，创建队列时通过`x-message-ttl`参数来设置，单位是毫秒

```go
args := amqp.Table{"x-message-ttl": 5000}  // 队列中的消息将在5秒后自动删除
_, err := r.channel.QueueDeclare(
	r.QueueName, // name，队列名
	false,       // durable，是否持久化
	false,       // 是否为自动删除
	false,       // 是否具有排他性
	false,       // 是否阻塞
	args,        // 额外属性
)
```

- 设置单独的消息，在发布消息时设置`Expiration`，格式是字符串，单位是毫秒

```go
r.channel.Publish(
	...
	amqp.Publishing{
		ContentType: "text/plain",
		Body:        []byte(message),
		Expiration:  "2000",
	},
)
```

如果同时设置了队列和消息的TTL，似乎是以`短的`那个时间为准

### 死信队列+延时队列

有时消息可能会失效变成死信（Dead Letter）状态，默认会被自动删除，如果我们想要继续利用这些消息，就需要有一些处理手段

消息变成死信状态有几种情况：

- 1.消息过期

- 2.消息被消费者拒绝且未设置重回队列

- 3.队列超过最大长度（可以设置策略为`删除最早消息`和`拒绝新消息加入`）

rabbitmq的方式是为队列绑定一个`死信交换机`（实际为死信交换机和它拥有的队列），当队列中的消息失效后，会被发送到这个死信交换机上，进行后续的处理

指定死信交换机：

```go
args := amqp.Table{
	"x-dead-letter-exchange":    "dead_letter",  // 死信交换机名称
	"x-dead-letter-routing-key": "my_key",       // 死信交换机的路由键
}
_, err := r.channel.QueueDeclare(
	r.QueueName, // name，队列名
	false,       // durable，是否持久化
	false,       // 是否为自动删除
	false,       // 是否具有排他性
	false,       // 是否阻塞
	args,        // 额外属性
)
```

死信队列最常用的应用就是`延时队列`，思想是将一个设置了TTL的队列绑定到死信交换机，当消息过期后被转发到死信交换机上，而消费者可以从死信交换机的队列中消费消息，这样就实现了延时

为了实现延时功能，需要有：

- 一个正常的交换机

- 一个绑定到正常交换机的TTL队列，它指定一个死信交换机

- 一个死信交换机

- 死信交换机的队列，消费者从这些队列获取消息

死信交换机和队列的声明与普通的是一样的，不过注意绑定的时候路由键要与`x-dead-letter-routing-key`的值一致

```go
err = rabbitmq.channel.QueueBind(
	"dead_letter_q", // 队列名
	"my_key",        // 路由键
	"dead_letter",   // 交换机
	false,
	nil,
)
```

完整的[延时队列demo](https://chunar5354.github.io/2021/06/13/rabbit-delay-queue.html)

## Golang测试Demo

需要安装amqp包

```
go get github.com/streadway/amqp
```

代码：

```go
package rmq

import (
	"fmt"
	"log"

	"github.com/streadway/amqp"
)

// MQURL 格式 amqp://账号：密码@rabbitmq服务器地址：端口号/vhost
const MQURL = "amqp://chunar:456789@127.0.0.1:5672/vChunar"

type RabbitMQ struct {
	conn      *amqp.Connection // 连接
	channel   *amqp.Channel    // 信道
	QueueName string           // 队列名称
	Exchange  string           // 交换机
	Key       string           // 路由 Key
	Mqurl     string           // 连接信息
}

// NewRabbitMQ 创建结构体实例
func NewRabbitMQ(queueName, exchange, key string) RabbitMQ {
	rabbitmq := RabbitMQ{
		QueueName: queueName,
		Exchange:  exchange,
		Key:       key,
		Mqurl:     MQURL,
	}
	var err error
	// 创建rabbitmq连接，是一个socket
	rabbitmq.conn, err = amqp.Dial(rabbitmq.Mqurl)
	if err != nil {
		log.Fatalf("%s:%s", "创建连接错误！", err)
	}

	rabbitmq.channel, err = rabbitmq.conn.Channel()
	if err != nil {
		log.Fatalf("%s:%s", "获取channel失败！", err)
	}

	return rabbitmq
}

// 生产者
func (r *RabbitMQ) Publish(message string) {
	defer r.conn.Close()
	defer r.channel.Close()
	// 发送消息到队列中
	r.channel.Publish(
		r.Exchange,  // 交换器名
		r.QueueName, // 路由键填队列名
		false,       // 如果为true, 会根据exchange类型和routkey规则，如果无法找到符合条件的队列那么会把发送的消息返回给发送者
		false,       // 如果为true, 当exchange发送消息到队列后发现队列上没有绑定消费者，则会把消息发还给发送者
		amqp.Publishing{
			ContentType: "text/plain",
			Body:        []byte(message),
		},
	)
}

// 消费者
func (r *RabbitMQ) Consume() {
	defer r.conn.Close()
	defer r.channel.Close()
	// 申请队列，如果队列不存在会自动创建，如果存在则跳过创建
	// 保证队列存在，消息能发送到队列中
	_, err := r.channel.QueueDeclare(
		r.QueueName, // name，队列名
		false,       // durable，是否持久化
		false,       // 是否为自动删除
		false,       // 是否具有排他性
		false,       // 是否阻塞
		nil,         // 额外属性
	)
	if err != nil {
		fmt.Println(err)
	}

	// 接收消息
	msgs, err := r.channel.Consume(
		r.QueueName, // 队列名
		"",          // 消费者名
		true,        // ACK，是否自动应答
		false,       // 是否具有排他性
		false,       // 如果设置为true，表示不能消费同一个connection中的消息
		false,       // 是否阻塞
		nil,         // 额外属性
	)

	if err != nil {
		fmt.Println(err)
	}

	forever := make(chan bool)
	// 处理消息
	go func() {
		for d := range msgs {
			// 接收到消息后的处理
			log.Printf("Received a message: %s", d.Body)
			fmt.Println(d.Body)
			// d.Ack(false) // false表示只确认当前消息
		}
	}()
	log.Printf("等待消息到来")
	<-forever
}
```

生产者：

```go
func main() {
	rabbitmq := rmq.NewRabbitMQ("chunar_q", "", "")
	rabbitmq.Publish("Hello world!")
	fmt.Println("发送成功")
}
```

消费者：

```go
func main() {
	rabbitmq := rmq.NewRabbitMQ("chunar_q", "", "")
	rabbitmq.Consume()
}
```
