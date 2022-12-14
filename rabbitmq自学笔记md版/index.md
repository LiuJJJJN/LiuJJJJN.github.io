# RabbitMQ自学笔记md版




# RabbitMQ

> 大连交通大学 信息学院 刘嘉宁 2021-11-21
>
> 笔记摘自 bjpwernode 秦世国



## 什么是消息队列

- 消息队列 MQ ( Message Queue )

1. 解耦
    - 生产者负责将数据写入到队列中，谁想不想要这个数据，与生产者无关
    - 消费者直接从消息队列中取数据，即便消费者宕机超时，与生产者无关
2. 异步
    - 生产者执行完主要功能后，将后续需要处理的功能存入消息队列即可返回
    - 异步化调用其他系统接口，消费者处理速度不影响生产者性能
3. 削锋 / 限流
    - 生产者们根据自己的能力从消息队列中取数据，即便系统同时有再多请求都不至于崩溃



## RabbitMQ消息队列

- Erlang 语言开发的 AMQP 的开源实现

- AMQP：高级消息队列协议（应用层）

    - AMQP 协议本身包括三层：

        - Module Layer: 位于协议最高层，主要定义了一些供客户端调用的命令，客户端可以利用这些命令实现自己的业务逻辑。例如，客户端可以使用Queue . Declare 命令声明一个队列或者使用Basic.Consume 订阅消费一个队列中的消息。

        - Session Layer: 位于中间层，主要负责将客户端的命令发送给服务器，再将服务端的应答返回给客户端，主要为客户端与服务器之间的通信提供可靠性同步机制和错误处理。
        - Transport Layer: 位于最底层，主要传输二进制数据流，提供帧的处理、信道复用、错误检测和数据表示等。



## RabbitMQ 特点

1. 可靠性：持久化、传输确认、发布确认
2. 灵活的路由：Exchange
3. 消息集群：集群组成Broker
4. 高可用：宕机解决
5. 多种协议：支持多种协议
6. 多语言客户端：支持常用编程语言
7. 管理界面：提供可视化界面
8. 跟踪机制：消息异常时可追踪维护



## RabbitMQ常用命令

1. 启动 RabbitMQ：`rabbitmq-server start &`
2. 关闭 RabbitMQ：`rabbitmqctl stop`




