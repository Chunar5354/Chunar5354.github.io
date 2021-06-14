---
layout: article
tags: middleware RabbitMQ
title: RabbitMQ延时队列代码实例
article_header:
  type: overlay
  theme: dark
  background_color: '#123'
  background_image: false
---

Golang RabbitMQ消息队列实现延时队列

<!--more-->

## 说明

使用了默认交换机作为连接publisher的交换机

## Golang Demo

核心代码:

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

	// 死信交换机
	err = rabbitmq.channel.ExchangeDeclare(
		"dead_letter", // 交换机名称
		"direct",      // 交换机类型
		false,         // 是否持久化
		false,         // 是否为自动删除
		false,         // 是否为内部exchange，true表示这个exchange不可以被客户端用来推送消息，仅仅是用来进行exchange和exchange之间的绑定
		false,         // no-wait，fase表示是阻塞 true表示是不阻塞
		nil,           // 其他参数
	)
	// 死信队列
	_, err = rabbitmq.channel.QueueDeclare(
		"dead_letter_q", // name，队列名
		false,           // durable，是否持久化
		false,           // 是否为自动删除
		false,           // 是否具有排他性
		false,           // 是否阻塞
		nil,             // 额外属性
	)
	// 绑定
	err = rabbitmq.channel.QueueBind(
		"dead_letter_q", // 队列名
		"my_key",        // 路由键
		"dead_letter",   // 交换机
		false,
		nil,
	)

	return rabbitmq
}

// 生产者
func (r *RabbitMQ) Publish(message string) {
	defer r.conn.Close()
	defer r.channel.Close()
	// 申请队列，如果队列不存在会自动创建，如果存在则跳过创建
	// 保证队列存在，消息能发送到队列中
	args := amqp.Table{
		"x-message-ttl":             5000,
		"x-dead-letter-exchange":    "dead_letter",
		"x-dead-letter-routing-key": "my_key",
	}
	_, err := r.channel.QueueDeclare(
		r.QueueName, // name，队列名
		false,       // durable，是否持久化
		false,       // 是否为自动删除
		false,       // 是否具有排他性
		false,       // 是否阻塞
		args,        // 额外属性
	)
	if err != nil {
		fmt.Println(err)
	}
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
	// 接收消息
	msgs, err := r.channel.Consume(
		"dead_letter_q", // 队列名
		"",              // 消费者名
		true,            // ACK，是否自动应答
		false,           // 是否具有排他性
		false,           // 如果设置为true，表示不能消费同一个connection中的消息
		false,           // 是否阻塞
		nil,             // 额外属性
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
	rabbitmq := rmq.NewRabbitMQ("delay_q", "", "")
	rabbitmq.Publish("hello world")
	fmt.Println("发送成功")
}
```

消费者：

```go
func main() {
	rabbitmq := rmq.NewRabbitMQ("", "", "")
	rabbitmq.Consume()
}
```
