---
layout: article
tags: middleware RabbitMQ
title: RabbitMQ订阅-发布模型代码实例
article_header:
  type: overlay
  theme: dark
  background_color: '#123'
  background_image: false
---

Golang RabbitMQ消息队列实现订阅-发布模型

<!--more-->

## 说明

fanout交换机会将消息广播给所有绑定到它身上的队列，所以在生产者方`不需要创建队列`，只需要创建交换机（通过`amqp.Channel.ExchangeDeclare()`方法），绑定队列的操作交由消费者来完成

由于fanout方式的特性，它的消息转发`只与交换机相关`，至于队列名与路由键统统不重要，所以在创建队列的时候，传入一个空字符串，rabbitmq会自动生成一个随机队列名：

```go
q, err := r.channel.QueueDeclare(
	"",    // name，队列名
	false, // durable，是否持久化
	false, // 是否为自动删除
	true,  // 是否具有排他性
	false, // 是否阻塞
	nil,   // 额外属性
)
```

注意这里我们将队列的`exclusive`排他性设置为了true，它会在消费者链接断开之后`删除这个队列`，这一点在fanout方法中很有必要，因为如果消费者断开链接，队列却仍然保留的话，按照fanout的转发规则，交换机仍然会将新的消息发送给这个没有消费者消费的队列，造成不必要的资源浪费

然后绑定到交换机

```go
err = r.channel.QueueBind(
	q.Name,     // 队列名
	"",         // 路由键
	r.Exchange, // 交换机
	false,
	nil,
)
```

注意由于生产者中没有创建队列，所以运行这个demo时要先开启消费者，再开启生产者，开启多个消费者时，它们会同时收到消息

## Golang Demo

核心代码：

```go
package rmq

import (
	"fmt"
	"log"

	"github.com/streadway/amqp"
)

// MQURL 格式 amqp://账号：密码@rabbitmq服务器地址：端口号/vhost
const MQURL = "amqp://chunar:456789@1.15.140.88:12056/vChunar"

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

	err := r.channel.ExchangeDeclare(
		r.Exchange, // 交换机名称
		"fanout",   // 交换机类型
		true,       // 是否持久化
		false,      // 是否为自动删除
		false,      // 是否为内部exchange，true表示这个exchange不可以被客户端用来推送消息，仅仅是用来进行exchange和exchange之间的绑定
		false,      // no-wait，fase表示是阻塞 true表示是不阻塞
		nil,        // 其他参数
	)

	if err != nil {
		fmt.Println(err)
	}
	// 发送消息到队列中
	err = r.channel.Publish(
		r.Exchange, // 交换器名
		"",         // 路由键
		false,      // 如果为true, 会根据exchange类型和routkey规则，如果无法找到符合条件的队列那么会把发送的消息返回给发送者
		false,      // 如果为true, 当exchange发送消息到队列后发现队列上没有绑定消费者，则会把消息发还给发送者
		amqp.Publishing{
			ContentType: "text/plain",
			Body:        []byte(message),
		},
	)
	if err != nil {
		fmt.Println(err)
	}
}

// 消费者
func (r *RabbitMQ) Consume() {
	defer r.conn.Close()
	defer r.channel.Close()

	err := r.channel.ExchangeDeclare(
		r.Exchange, // 交换机名称
		"fanout",   // 交换机类型
		true,       // 是否持久化
		false,      // 是否为自动删除
		false,      // 是否为内部exchange，true表示这个exchange不可以被客户端用来推送消息，仅仅是用来进行exchange和exchange之间的绑定
		false,      // no-wait，fase表示是阻塞 true表示是不阻塞
		nil,        // 其他参数
	)

	if err != nil {
		fmt.Println(err)
	}
	// 申请队列，如果队列不存在会自动创建，如果存在则跳过创建
	// 保证队列存在，消息能发送到队列中
	q, err := r.channel.QueueDeclare(
		"",    // name，队列名
		false, // durable，是否持久化
		false, // 是否为自动删除
		true,  // 是否具有排他性
		false, // 是否阻塞
		nil,   // 额外属性
	)
	if err != nil {
		fmt.Println(err)
	}

	err = r.channel.QueueBind(
		q.Name,     // 队列名
		"",         // 路由键
		r.Exchange, // 交换机
		false,
		nil,
	)

	if err != nil {
		fmt.Println(err)
	}

	// 接收消息
	msgs, err := r.channel.Consume(
		q.Name, // 队列名
		"",     // 消费者名
		true,   // ACK，是否自动确认
		false,  // 是否具有排他性
		false,  // 如果设置为true，表示不能消费同一个connection中的消息
		false,  // 是否阻塞
		nil,    // 额外属性
	)

	if err != nil {
		fmt.Println(err)
	}

	forever := make(chan bool)
	// 启用协程处理消息
	go func() {
		for d := range msgs {
			// 实现我们要实现的逻辑函数
			log.Printf("Received a message: %s", d.Body)
			fmt.Println(d.Body)
			// d.Ack(false) // false表示只确认当前消息
		}
	}()
	log.Printf("等待消息到来")
	<-forever
}
```

发布者：

```go
func main() {
	rabbitmq := rmq.NewRabbitMQ("", "fanout_exchange", "")
	rabbitmq.Publish("hello world")
	fmt.Println("发送成功")
}
```

订阅者：

```go
func main() {
	rabbitmq := rmq.NewRabbitMQ("", "fanout_exchange", "")
	rabbitmq.Consume()
}
```
