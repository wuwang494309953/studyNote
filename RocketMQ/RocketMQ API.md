#### RocketMQ生产者-消息的返回状态

| 状态码              | 解释                                               |
| ------------------- | -------------------------------------------------- |
| SEND_OK             | 发送成功                                           |
| FLUSH_DISK_TIMEOUT  | 发送成功，但是未持久化，如果此时MQ挂掉，消息会丢失 |
| FLUSH_SLAVE_TIMEOUT | 从节点同步失败                                     |
| SLAVE_NOT_AVAILABLE | 从节点不可用                                       |

##### DefaultMQProducer


| 方法名                                  | 解释                                               |
| --------------------------------------- | -------------------------------------------------- |
| DefaultMQProducer()                     | 构造方法。默认`producerGroup`为 "DEFAULT_PRODUCER" |
| DefaultMQProducer(String producerGroup) | 构造方法                                           |
| setNamesrvAddr(String namesrvAddr)      | 设置 Name Server 地址                              |

##### Message

| 方法名                                                       | 解释                                                       |
| ------------------------------------------------------------ | ---------------------------------------------------------- |
| Message(String topic, String tags, String keys, byte[] body) | 构造方法                                                   |
| setDelayTimeLevel(int level)                                 | 设置延迟消息(1s 5s 10s 30s 1m 2m 3m ... 10m 20m 30m 1h 2h) |
|                                                              |                                                            |

##### DefaultMQPushConsumer

| 方法名                       | 解释                           |
| ---------------------------- | ------------------------------ |
| setConsumeFromWhere          | 消费者启动时从哪里开始消息消息 |
| allocateMessageQueueStrategy | 消费者消费策略                 |
| subscription                 | 订阅者                         |
| offsetStore                  | 本地存储和集群存储             |
| pullInterval                 | 拉取的时间间隔                 |
| pullBatchSize                | 拉取的个数                     |
| consumeMessageBatchMaxSize   | 一个线程一次消费多少条消息     |