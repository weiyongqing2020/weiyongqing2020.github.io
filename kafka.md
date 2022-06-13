# kafka概述
kafka是高吞吐，持久化的，分布式的消息队列。  
kafka借助zookeeper来实现集群元数据的管理，控制器的选举等操作。  
有三种主要的使用场景：
- 消息系统：即所谓的消息中间件，有削峰，解耦，异步通信等作用。
- 存储系统：kafka将消息持久化到磁盘，比其他将消息保存在内存的系统来说，降低了数据丢失的风险，只需将对应的数据包存储策略设置为永久保存或启用主题的日志压缩功能即可。
- 流式处理平台：kafka提供完整的流式处理类库。


# 安装与配置
先配置好zk, zk也可以配置为集群。

主要在配置文件中配置brokerid，log文件的地址，对外提供服务的地址以及zk的地址。再用启动脚本kafka- server-start . sh启动。

# 生产者API
**生产者实例是线程安全的**

**生产者api的使用步骤** 
1. 配置生产者客户端的参数，创建生产者实例
2. 构建消息
3. 发送消息
4. 关闭生产者实例

生产者发送消息有三种方式:  
1. 发后即忘
```
public void send(KafkaProducer<String, String> kp, ProducerRecord<String, String> pr, String topic, String value) {
    kp.send(pr);
}
```
2. 异步发送
```
public void send(KafkaProducer<String, String> kp, ProducerRecord<String, String> pr, String topic, String value) {
    kp.send(pr, new Callback() {
        // 回调函数
        @Override
        public void onCompletion(RecordMetadata metadata, Exception exception) {
            if (topic.equals(KafkaTopic.CAN_DATA) || topic.equals(KafkaTopic.CAN_DATA_GB32960) || topic.equals(KafkaTopic.GPS_DATA) || topic.equals(KafkaTopic.BRACELET_DATA)) {
                return;
            }
            if (null != exception) {
                //log.info_kafka("upload kafka fail! topic:"+topic + exception.getMessage());
                //入本地库
                insertlocalSqlite(topic, value);
                return;
            }
            //log.info_kafka("upload kafka success  topic:"+topic);
            return;
        }
    });
}
```
3. 同步发送
```
public boolean sendMessageSync(String topic, String value) {

    // 消息封装
    ProducerRecord<String, String> pr = new ProducerRecord<String, String>(topic, configFromSqlite.getSbbh(), value);
    try {
        // 发送数据,等待返回成功
        kp.send(pr).get();
        return true;
    } catch (ExecutionException ex) {
        log.error(ex.getMessage(), ex);
        return false;
    } catch (InterruptedException ex) {
        log.error(ex.getMessage(), ex);
        return false;
    }
}
```

## 拦截器

## 序列化器
需要将发送的消息序列化为字节数组，但是不建议根据对象自定义序列化方式，应该使用 更加通用的而方式，比如先将对象转为json字符串，再进行序列化。
## 分区器
如果没有指定分区器，则会根据key做哈希函数，再对分区数进行取模，来确定该发往哪个分区。
## 生产者的架构

如上图所示，生产者端有两个线程，主线程和sender线程，消息在主线程依次经过拦截器，序列化器，分区器后，进入消息累加器中，消息累加器为每个分区维护了一个双端队列，队列中保存了一个一个批次，批次的大小由batch.size设置，默认为16KB，但是批次的的大小跟消息的大小也有着密切的关系，每次有新的消息过来，会判断当前批次还放不放的下这条消息，如果放不下会创建新的批次，如果消息大小小于批次大小就按照参数大小来创建，大于批次大小就按照消息的大小来创建。

sender线程会将累加器中的<分区，Deque<ProducerBatch>>转换为<Node,Deque<ProducerBatch>>的形式，sender线程只关心消息往哪个节点发，而不关系消息属于哪个分区。

消息累加器的大小右buffer.memory控制，默认为32MB。当消息累加器满了后，在send方法发送消息就会阻塞或抛异常，这取决于max.block.ms参数，默认为60秒。
## 元数据的更新
kafka的各个broker节点，kafka有哪些主题，主题有哪些分区，分区的leader副本在哪个节点上，这些都属于元数据信息，生产者需要知道这些元数据信息，才能确定消息该往哪个节点发送。
如上图的生产者架构中，有InflightRequest模块，它会记录下已经发送出去但还没有收到回复的消息（个数由参数max.in.flight.requests.per.connection设置，默认为5），这种消息最少的节点，也就是负载最轻，响应最快的broker节点，生产者端向此节点发送查询元数据的请求。

## 重要的生产者参数
```
properties = new Properties();
 properties.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "101.52.126.109:9092");
 //properties.put("zookeeper.connect", zookeeper_url);//声明zk
 //properties.put("producer.type", "sync");
 properties.put(ProducerConfig.ACKS_CONFIG, "1");
 // 发送失败后,再重复发送的次数
 properties.put(ProducerConfig.RETRY_BACKOFF_MS_CONFIG, 0);
 //序列化器
 properties.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringSerializer");
 properties.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringSerializer");
 //分区器
 properties.put(ProducerConfig.PARTITIONER_CLASS_CONFIG, "kafka.producer.DefaultPartitioner");
 //properties.put("bak.key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
 //properties.put("bak.value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
 //properties.put("bak.value.serializer", "org.apache.kafka.common.serialization.StringSerializer");

/* //消息累加器中BufferPool的大小，默认16KB
 ProducerConfig.BATCH_SIZE_CONFIG;
 //消息累加器的大小,默认32MB
 ProducerConfig.BUFFER_MEMORY_CONFIG;
 //请求在sender线程发往kafka之前还会被保存在InFlightRequests中，InFlightRequests缓存已经发出去但还没有收到响应的请求，每个连接最多缓存n(默认为5)个未响应的请求。
 ProducerConfig.MAX_IN_FLIGHT_REQUESTS_PER_CONNECTION;
 //生产消息的速度大于发送到服务器的速度，就会将缓冲区占满，此时send()方法要么阻塞，要不抛异常，取决于参数max.block.ms参数的设置
 ProducerConfig.MAX_BLOCK_MS_CONFIG;
 //生产者客户端能发送消息的最大值 B为单位，默认是1048576B，即1M
 ProducerConfig.MAX_REQUEST_SIZE_CONFIG;*/
```


# 消费者API
## 消费者与消费者组的概念
## 消费的步骤
1. 配置参数，创建消费者实例
2. 订阅主题
3. 拉取消息并消费
4. 提交偏移量
5. 关闭消费者实例  

### 订阅主题
订阅主题有三种方式：  
1. 指定要订阅的topic集合
2. 用正则表达式来订阅topic
3. 订阅指定的topic与分区。
```
// 订阅主题
consumer.subscribe(Collections.singletonList(topic));

//正则表达式 consumer.subscribe(Pattern.compile("can*")) //订阅所有can开头的topic
//consumer.assign(Arrays.asList(new TopicPartition(topic,0))); //只会消费topic的第0分区的消息
```
### 拉取消息的方式
拉取消息完成后可以获取消息集中分区信息：
```
/**
 * 消息消费
 */
//处理指定分区的消息
//获取消息集中所有的分区
Set<TopicPartition> partitionSet = records.partitions();
//获取指定分区的消息
List<ConsumerRecord<String, String>> res = records.records(new TopicPartition(topic, 0));

可以指定偏移量或时间来进行拉取消息
/**
 * 设置从指定的位移开始消费
 */
//seek方法可以指定从某分区的某位移开始消费，但要先执行poll方法，因为是在poll方法中给消费者分配分区。
Set<TopicPartition> partitions = new HashSet<>();
//无法判定poll方法的分配分区需要多少时间，用循环来等待
while (partitions.size() == 0) {
    consumer.poll(Duration.ofMillis(100));
    partitions = consumer.assignment();
}
for (TopicPartition topicPartition : partitions) {
    //从位移10开始消费
    consumer.seek(topicPartition, 10);
}
//
//查找分区末尾
Map<TopicPartition, Long> offsets = consumer.endOffsets(partitions);
//查找分区起始点
offsets = consumer.beginningOffsets(partitions);
//按时间查询，查找分区一天前的偏移量
Map<TopicPartition, Long> timestampToSearch = new HashMap<>();
for (TopicPartition each : partitions) {
    timestampToSearch.put(each, System.currentTimeMillis() - 1 * 24 * 60 * 60 * 1000);
}
Map<TopicPartition, OffsetAndTimestamp> offsetAndTimestampMap = consumer.offsetsForTimes(timestampToSearch);

提交消费位移的方式
/**
 * 提交位移
 */
//手动提交-同步提交
//不带参数，提交当前poll到的最新的消费位移
consumer.commitSync();
//带参数，指定提交某分区的指定消费位移
for (TopicPartition topicPartition : records.partitions()) {
    List<ConsumerRecord<String, String>> each = records.records(topicPartition);
    //消费数据...
    consumer.commitSync(Collections.singletonMap(topicPartition, new OffsetAndMetadata(each.get(each.size() - 1).offset())));

}

//手动提交-异步提交,不会阻塞
//不带参数
consumer.commitAsync();
```

## 分区的再均衡
```
/**
 * 再均衡 分区的所属权从一个消费者转为另一个消费者的行为
 */
//再均衡监听器
consumer.subscribe(Collections.singleton(topic), new ConsumerRebalanceListener() {
    //再均衡操作之前，消费者停止读取消息之后
    @Override
    public void onPartitionsRevoked(Collection<TopicPartition> collection) {
        consumer.commitSync();
    }

    //再均衡操作之后，消费者开始读取消息之前
    @Override
    public void onPartitionsAssigned(Collection<TopicPartition> collection) {
        //
    }
});
```

## 如何提高消费的效率
```
/**
 * 多线程
 */
//多线程消费的两个主要的方法
//1.创建多个KafkaConsumer实例，一个消费者消费一个或多个分区。会需要创建多个tcp连接，分区数多的话系统开销大。
//2.因为poll方法一般很快，考虑将处理消息的模块改成多线程的方式。
```

## 关键的参数
```
properties = new Properties();
//可以配置一个或多个broker的地址，只配置一个broker也可以找到其他broker的地址，配置多个可以防止其中一个broker挂掉导致连不上kafka
properties.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, kafka_url);
//消费者组id
properties.put(ConsumerConfig.GROUP_ID_CONFIG, "reader_01");
//自动提交消费位移,默认为true，自动提交消费位移在poll方法的逻辑里完成，每次poll都会检查是否可以进行位移提交
//若要手动提交，需要将这个参数设置为false
properties.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG,true);
//自动提交的时间间隔，默认5秒
properties.put(ConsumerConfig.AUTO_COMMIT_INTERVAL_MS_CONFIG, "1000");
//当kafka找不到消费位移时，从哪里开始消费 earliest：表示从分区头开始，latest：表示从分区末尾开始 none：表示抛出NoOffsetForPartitionException异常
properties.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG,"earliest");
//key和value的反序列化器
properties.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringDeserializer");
properties.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringDeserializer");

/*//拦截器
ConsumerConfig.INTERCEPTOR_CLASSES_CONFIG;
//poll方法一次能拉取的最小数据量，对于延迟不敏感的业务可以调大这个参数来提升吞吐量
ConsumerConfig.FETCH_MIN_BYTES_CONFIG;
//poll方法一次能拉取的最大数据量，就算某条消息的大小大于这个参数，也是可以正常消费的。
ConsumerConfig.FETCH_MAX_BYTES_CONFIG;
//满足不了FETCH_MIN_BYTES_CONFIG时的最大等待时间
ConsumerConfig.FETCH_MAX_WAIT_MS_CONFIG;
//一次拉取最大的消息条数，默认500
ConsumerConfig.MAX_POLL_RECORDS_CONFIG;
//socket接收缓冲区的大小 默认65536 64kb
ConsumerConfig.RECEIVE_BUFFER_CONFIG;
//socket发送缓冲区的大小 默认128kb 如果消费者与kafka在不同的机房，可以适当调大这个值
ConsumerConfig.SEND_BUFFER_CONFIG;*/
```

# 主题与分区的管理
## 主题的管理
### 创建主题
可以使用kafka-topics.sh来创建主题，kafka-topics.sh有alter，list，describe，create等命令，不同的命令有不同的参数。
创建主题的实例如下：



也可以传入replica-assignment参数来手动智指定分区副本的分配方案：

可以通过传入config参数来覆盖配置：



注：broker端参数auto.create.topics.enable配置为true时（默认就是true），会在生产者向一个不存在的主题发送数据时自动创建主题，建议不要配置为true，减少不可控因素。
主题的名称不能重复（加上if-no-exist参数可以避免报错），主题名也尽量不要包含“.”或“_”。
分区数只能增加不能减少，减少操作的成本太高。
### 查看主题
可以通过list命令来查看所有主题：

可以通过describe命令来查看某一主题，指定主题，或所有主题的详细信息：

通过under-replicated-partitions可以查看存在失效副本的分区（即isr<ar的分区）：

通过unavailable-partitions参数可以查看没有leader副本的分区，此时分区不可用：



### 修改主题
主题创建好后，也可以通过alter命令进行修改，例如修改分区数，修改参数等。
增加分区的示例：

分区数增加后可能会引起原本的key发送的分区发生变化（生产者api在没有指定分区号的情况下，会对key做哈希函数，然后对分区数取余，操作类似于哈希表中对将key映射为数组下标，如果分区数发生了变化，那么映射的分区号也会发生变化）。

修改配置的示例：


### 删除主题
可以通过delete命令删除主题，示例如下：

删除主题其实就是在zk的/admin/delete_topics下创建与待删除的主题同名的节点，真正的删除操作是由控制器来完成的。所有我们也可以通过手动在上述路径下添加zk节点的方式来删除主题。

或者我们也可以通过手动删除zk节点，再删除相关文件的方式来删除主题。


## 分区的管理
###优先副本的选举
在kafka集群的运行过程中，可能有broker节点宕机然后又恢复，此时就会出现leader副本分布不均衡的情况，造成负载失衡，所以需要进行优先副本的选举。
优先副本就是指ar集合中排第一个的副本。
auto.leader.rebalance.enable参数设置为true时表示会自动按时进行优选副本的选举操作，但是不建议将其设置为true，这样可能在业务高峰期影响业务的正常进行。
用kafka-perferred-replica-election.sh脚本执行优先副本的选举：

leader副本的转移是一项成本较高的工作，可以通过json文件，指定分区的方式分批次的进行此操作：



### 分区的重分配
当kafka集群中有broker节点下线时，该节点上的副本会变的不可用，会导致集群的负载失衡，影响
服务的可用性。当增加新的broker节点时，新增节点只对新创建的topic有用，而对原先的topic无效。所以需要进行分区的重分配操作。
分区的重分配需要用kafka-reassign-partitions.sh脚本，分为以下三步：
1. 在json文件中写明要进行分区重分配的topic，即主题清单
2. 脚本将此文件和所要分配的broker节点列表作为参数执行generate命令，会显示当前的分区副本情况，以及自动生成的要执行的方案。，可以对执行的方案进行修改。
3. 执行方案。

执行完分区的重分配后往往需要在执行一次优先副本的选举，确保负载的均衡。

添加副本因子
可以通过分区重分配来添加副本因子。

## 如何选择合适的分区数
分区数并不是越多越好，可以通过kafka-producer-perf-test.sh生产者性能测试脚本进行测试，来确定topic合适的分区数。消费者也有性能测试脚本的 kafka-consumer-perf-test.sh。
kafka-producer-perf-test.sh的使用示例：


## KafkaAdminClient
一般使用kafka-topics.sh脚本来进行主题的管理，也可以用KafkaAdminC!ient来管理主题，以此可以将主题管理的功能集成到公司内部的系统中，打造管理，监控，告警，运维为一体的平台。

# 日志结构
kafka的日志文件结构如下图所示：

每个副本的命名方式为topic-partition，每个副本是一个文件夹，里面包含多个日志分段文件（LogSegment），每个日志分段文件起码有两个索引文件，偏移量索引文件和时间戳索引文件（还可能有事务索引文件），日志分段文件以及对应的索引文件都以起始绝对偏移量来命名。

在消费者拉取消息或者follower副本同步消息的时候，提供的都是逻辑偏移量，需要有一个逻辑偏移量到物理地址的对应关系。综合存储空间。查找效率等多方考虑，kafka最终选择稀疏索引的方式来组织索引文件，绝对偏移量是8字节的长整型值，索引文件中的偏移量是4字节的相对偏移量，定位后需要将其与文件名中的起始绝对偏移量相加。log.index.interval.bytes可以控制索引项的密度。




偏移量索引文件的索引项内容是相对偏移量与其对应的物理地址。根据消息的偏移量X来查找消息物理地址的过程：broker端维护了一个跳表，每个日志文件的baseOffset作为key，先定位小于等于X的最大值的日志分段，再计算相对偏移量，在偏移量索引文件中使用二分查找找到小于等于该相对偏移量的最大值（kafka会将偏移量索引文件映射到内存中），获取到物理地址后再向后找，直到定位到该条消息。

时间戳索引文件的索引项内容是时间戳与其对应的相对偏移量。根据时间戳定位消息的过程：根据时间戳X逐一对比日志分段中的最大时间戳，定位到X所属的时间戳索引文件，再使用二分查找定位到小于等于X的最大值，取出对应的相对偏移量，然后再根据这个相对偏移量走一遍偏移量索引文件的查询，有点类似于mysql中的二级索引。



生成新的日志分段文件的几种情形：
日志分段文件的大小达到了参数log.segme bytes设置的阈值值（默认为1GB）。
索引文件的大小达到了参数log.index.size .max. bytes设置的阈值(默认为10M)。
消息的相对偏移量达到了int.MAX_VALUE。

日志文件的清理策略：
日志删除
日志压缩:相同key的不同value值只保留一条
可以通过参数log.cleanup.policy来设置日志清理策略（delete/compact）
日志删除
可以通过配置参数log.retention.ms，log.retention.minutes，log.retention.hours来设置日志文件保留的时间，优先级由高到低，默认设置log.retention.hours为168，即7天。
也可以通过设置日志的大小（注意是所有日志文件的大小，不是单个日志分段文件）的阈值来作为日志删除的条件。相关参数为log.retention.bytes，默认为-1，即无穷大。
日志压缩

磁盘存储
顺序IO
kafka写磁盘采用顺序IO，只会不断向文件后追加数据。
页缓存
将磁盘的数据缓存在内存中，把对磁盘的访问变为对内存的访问，kafka中大量使用页缓存，即使kafka服务重启，页缓存依然有效。
零拷贝
将数据直接从磁盘发到网卡设备中，而不用经过应用程序之手。




# 深入服务端
kafka的协议





控制器：

控制器由zookeeper选出，每个broker节点在启动的时候都会去竞争成为controller节点。
控制器需要管理整个集群中所有分区和副本的状态。当leader副本出现故障时，控制器要负责选出新的leader副本；当通过kafka-topics.sh脚本为某个topic增加新的分区数时，控制器要负责分区的重新分配；当检测到某个分区的isr集合出现变化时，控制器要通知所有的broker更新其元数据信息。

控制器比普通的broker节点要多做的工作：
- 监听分区相关的变化，处理分区的重分配，优先副本的选举，isr集合的变更。
- 监听主题相关的变化。
- 监听broker相关的变化。
- 更新集群的元数据信息。



# 优雅的关闭方式：
使用脚本kafka-server-stop.sh来关闭，但是可能有一个问题，关闭脚本是通过ps ax来找到kafka的进程id，然后kill -s命令结束掉进程，但是在linux系统中，ps命令对打印的最大字符数有限制，不能超过
页大小，所以脚本可能获取不到kafka的进程id。

推荐使用kill -s或kill -15或kill 命令来结束卡夫卡进程，kill 命令不写参数的话默认参数就是-15，这种方式可以触发kafka的钩子函数，确保消息完全同步到磁盘上。

不能使用kill -9来关闭kafka进程。

# 消息的可靠性

