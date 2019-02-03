Kafka系列-3-Kafka架构介绍
------------------------------------------

# Kafka系列文章地址
- [Kafka系列-1-Kafka基本介绍](https://github.com/liukgg/study_notes/blob/master/kafka/kafka_intro_01.md)
- [Kafka系列-2-常用消息系统简单对比](https://github.com/liukgg/study_notes/blob/master/kafka/kafka_intro_02.md)

# Kafka相关术语
- Broker 实现数据存储的服务器，Kafka集群包含一个或多个服务器
- Topic 用来对消息进行分类，每个进入到Kafka的信息都会被放到一个Topic下
- Partition 每个Topic中的消息会被分为若干个Partition，以提高消息的处理效率
- Producer 消息的生产者，负责发布消息到Kafka broker
- Consumer 消息的消费者，向Kafka broker读取消息的客户端
- Consumer Group 消息的消费群组

# Topic 与 Partition，Consumer 与 Consumer Group
- 物理上不同Topic的消息分开存储
- 逻辑上一个Topic的消息虽然保存于一个或多个broker上但用户只需指定消息的Topic即可生产或消费数据而不必关心数据存于何处
- Parition是物理上的概念，每个Topic包含一个或多个Partition
- 每个Consumer属于一个特定的Consumer Group（可为每个Consumer指定group name，若不指定group name则属于默认的group）

# Kafka拓扑结构
一个典型的Kafka集群中包含：若干Producer，若干broker，若干Consumer Group，以及一个Zookeeper集群。

### 从Kafka本身的角度看 Kafka 拓扑结构
![kakfa-architerure-01](https://mmbiz.qpic.cn/mmbiz_jpg/KoEickpDsyE4b72b6icEtCXtwicsjiaBicwUk6T9iacic6cdLzHIbmHiaTzRhxPtOmTrD8NLxAVUcoI9m3Gric0VHKCwAYA/0?wx_fmt=jpeg)

### 从业务场景、业务系统的角度看 Kafka 拓扑结构
![kakfa-architerure-02](https://mmbiz.qpic.cn/mmbiz_png/KoEickpDsyE4b72b6icEtCXtwicsjiaBicwUkibrZbM4H67R9iaxXjIRa342urxpQNaLibWtCCx8giagL5I5ibID1YdM31ow/0?wx_fmt=png)

### Kafka架构剖析
- 一个典型的Kafka集群中包含：若干Producer（可以是web前端产生的Page View，或者是服务器日志，系统CPU、Memory等），若干broker（Kafka支持水平扩展，一般broker数量越多，集群吞吐率越高），若干Consumer Group，以及一个Zookeeper集群。
- Kafka通过Zookeeper管理集群配置，选举leader，以及在Consumer Group发生变化时进行rebalance。
- Producer使用push模式将消息发布到broker，Consumer使用pull模式从broker订阅并消费消息。

### Kafka的核心API介绍
- Kafka高效地处理实时流式数据，可以实现与Storm、HBase和Spark的集成。
- 作为群集部署到多台服务器上，Kafka处理它所有的发布和订阅消息系统使用了四个API，即生产者API、消费者API、Stream API和Connector API。
- 它能够传递大规模流式消息，自带容错功能。
- Kafka有四个主要API：
  * 生产者API：支持应用程序发布Record流。
  * 消费者API：支持应用程序订阅Topic和处理Record流。
  * Stream API：将输入流转换为输出流，并产生结果。
  * Connector API：执行可重用的生产者和消费者API，可将Topic链接到现有应用程序。

### Kafka为何选择Pull模式而不是push模式
- push模式很难适应消费速率不同的消费者，因为消息发送速率是由broker决定的。
- 而pull模式则可以根据Consumer的消费能力以适当的速率消费消息。
- 对于Kafka而言，pull模式更合适。
- pull模式可简化broker的设计，Consumer可自主控制消费消息的速率，同时Consumer可以自己控制消费方式——即可批量消费也可逐条消费，同时还能选择不同的提交方式从而实现不同的传输语义（三种传输语义：至少一次提交，至多一次提交，严格一次提交）。

# 参考材料
- http://www.infoq.com/cn/articles/depth-interpretation-of-kafka-data-reliability?utm_source=articles_about_Kafka&utm_medium=link&utm_campaign=Kafka
- http://www.infoq.com/cn/articles/kafka-analysis-part-1
