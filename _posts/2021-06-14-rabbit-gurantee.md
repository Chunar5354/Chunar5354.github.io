---
layout: article
tags: middleware RabbitMQ
title: RabbitMQ保障消息可靠性的方法
article_header:
  type: overlay
  theme: dark
  background_color: '#123'
  background_image: false
---

RabbitMQ消息队列中保障消息的可靠性

<!--more-->

# RabbirMQ保证消息可靠性

在rabbitmq中消息的传递路径为`publisher -> exchange -> queue -> consumer`

保证可靠性就是要保证消息在每一个环节都是可靠的

## 生产者到交换机的可靠性

生产者到交换机的可靠性保障有两种手段：`事务机制`和`发送方确认机制`，由于事务机制会损失性能，所以推荐使用发送方确认机制

在默认情况下，发送方将消息发送给broker（实际是exchange）后，服务器是不返回任何响应的，这样发送方也就无从知晓消息是否被正确接收

可以将发送方的channel设置为`confirm`模式，这样该channel上的每一条消息都会被赋予一个`唯一的ID`（DeliveryTag，从1开始），当消息被成功路由到`所有匹配的队列`后，就会返回一个`ack`给生产者（可持久化的消息和队列会在`写入磁盘`后返回ack），如果消息处理错误，则返回一个`nack`

消息返回是`异步`的，且无论是否处理成功，一定会返回响应（ack或nack），在golang中返回的响应为`amqp.Confirmation`类型:

```go
type Confirmation struct {
	DeliveryTag uint64 // A 1 based counter of publishings from when the channel was put in Confirm mode
	Ack         bool   // True when the server successfully received the publishing
}
```

### 实现方法

基于[这个demo]做了修改，完整代码比较长，就不放在这里了

在生产者方法中，加入：

```go
_ = r.channel.Confirm(false)                                      // 将生产者channel设为confirm模式，noWait为false表示该方法等待响应
confirms := r.channel.NotifyPublish(make(chan amqp.Confirmation)) // 创建一个接收确信信息的通道
go func(chan amqp.Confirmation) {
	for confirmed := range confirms {
		if confirmed.Ack {
			fmt.Printf("%d号消息发送成功\n", confirmed.DeliveryTag)
		} else {
			fmt.Printf("%d号消息发送失败\n", confirmed.DeliveryTag)
		}
	}
}(confirms)
```


## 交换机到队列

除了comfirm之外，rabbitmq还提供了mandatory和immediate：

- 如果开启mandatory，当没有队列通过指定的路由键绑定到交换机上时，即`路由错误`时，会向发送方返回本次发送的消息

- 如果开启immediate，当队列上`没有正在监听的消费者`时，会向发送方返回本次发送的消息(immediate似乎被禁用了，不要将这个字段设成true)

返回的信息是`amqp.Return`类型：

```go
type Return struct {
	ReplyCode  uint16 // reason
	ReplyText  string // description
	Exchange   string // basic.publish exchange
	RoutingKey string // basic.publish routing key

	// Properties
	ContentType     string    // MIME content type
	ContentEncoding string    // MIME content encoding
	Headers         Table     // Application or header exchange table
	DeliveryMode    uint8     // queue implementation use - non-persistent (1) or persistent (2)
	Priority        uint8     // queue implementation use - 0 to 9
	CorrelationId   string    // application use - correlation identifier
	ReplyTo         string    // application use - address to to reply to (ex: RPC)
	Expiration      string    // implementation use - message expiration spec
	MessageId       string    // application use - message identifier
	Timestamp       time.Time // application use - message timestamp
	Type            string    // application use - message type name
	UserId          string    // application use - creating user id
	AppId           string    // application use - creating application

	Body []byte
}
```

### 使用方法

基于[这个demo]做了修改，完整代码比较长，就不放在这里了

在发送消息的时候指定：

```go
r.channel.Publish(
	r.Exchange, // 交换器名
	r.Key,      // 路由键
	false,      // mandatory，如果为true, 会根据exchange类型和routkey规则，如果无法找到符合条件的队列那么会把发送的消息返回给发送者
	false,      // immediate，如果为true, 当exchange发送消息到队列后发现队列上没有绑定消费者，则会把消息发还给发送者
	amqp.Publishing{
		ContentType: "text/plain",
		Body:        []byte(message),
	},
)
```

创建接收返回信息的通道：

```go
returnChan := r.channel.NotifyReturn(make(chan amqp.Return))
go func(chan amqp.Return) {
	for re := range returnChan {
		fmt.Println("---------handle  return----------")
		fmt.Printf("replyCode: %d\n", re.ReplyCode)
		fmt.Println("replyText: " + re.ReplyText)
		fmt.Println("exchange: " + re.Exchange)
		fmt.Println("routingKey: " + re.RoutingKey)
		fmt.Println("body: " + string(re.Body))
	}
}(returnChan)
```

可以自己在发送方设置一个错误的路由键，测试一下输出结果

## 消息进入队列后宕机

如果消息成功进入了队列，却出现了宕机或服务器重启，就会导致数据丢失

对于这种情况要使用`持久化`的方式将消息存储在硬盘上

有三个元素可供用户自定义是否持久化：Exahcnge、Queue和Message，它们是`递进`的关系，也就是说要想Message实现持久化，就必须使得Exchange和Queue都实现持久化

在demo中使用了持久化的Exchange，而demo中的Queue和Message都使用了非持久化的方式，重启一下rabbitmq：

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

关于改成持久化的方式：

对于Exchange，要在使用`ExchangeDeclare()`方法的时候指定:

```go
err = rabbitmq.channel.ExchangeDeclare(
	exchange, // 交换机名称
	"direct", // 交换机类型
	true,     // 是否持久化
    ...
}
```

对于Queue，要在使用`QueueDeclare()`方法的时候指定：

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

## 队列到消费者

rabbitmq的消息被消费者消费后，需要`被确认`才能从队列中删除，有手动确认和自动确认两种方式

在demo中使用了自动确认，现在关闭自动确认试一下：

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

