## RocketMQ

### 消息就是数据或者命令

##### 1、MQ，messageQueue，消息队列中间件、提供了消息生产者、存储、消费者API的软件系统

##### 2、用途：

​	限流削峰（削峰填谷）：将系统的超量请求暂存其中，以便系统后期可以慢慢进行处理，避免请求丢失或者压垮

​	异步解耦：上游系统会对下游系统的调用如果是同步调用，会大大降低系统的吞吐量和并发度，并且系统耦合度太高了，而异步调用会解决这些问题，在这两层之间加个MQ层

​        AB不再相互依赖

​	数据收集：分布式系统会产生海量业务，比如业务日志、监控数据、用户行为等，针对这些数据流进行实时或者批量采集汇总。

​	跨语言：传数据等等



常见MQ产品：

#####  	metaQ

RabbitMQ，吞吐量教Kafaka和RocketMQ要低，不是java语言开发的

Kafaka，特点是高吞吐量、常用于大数据领域实时计算、日志采集

RocketMQ，抽离内核的metaQ，自研协议

### 基本概念：

1、消息message，就是数据的载体，生成和消费的最小单位

2、主题topic：

​	每个消息必须属于一个主题，是一类消息的集合，是RocketMQ进行消息订阅的基本单位。一个生产者是可以发送很多种Topic的消息，一个消费者只能订阅一种Topic的消息，一个消息只能属于一个Topic。

3、Tag标签：

​	为消息设置的标签，用于同一个主题下的区分不同类型的消息，来自同一业务单元的消息，可以根据不同业务目的在同一主题下设置不同标签，消费者可以根据Tag对不同子主题实现不同的消费逻辑，提高扩展性。

4、队列Queue：

​	存储消息的物理实体，一个Topic可以包含多个Queue，每个Queue中存放的是该Topic的消息，一个Topic的queue也可以叫做分区partition

5、消息表示 messageID / messageKey

​	生产者在send()的时候自动生成一个MessageID(msgId)

​	消息到达Broker后，Broker也会自动生成一个MessageId(offsetMsgId)

​	msgId、offsetMsgId 与Key都称之为消息标识

msgId：由produce端生成，规则：

​	producerIp+进程pid+MessageClientSetter类中的ClassLoader中的hashcode+当前时间+AutomicInteger自增计数器

​	offsetMsgId：由broker端生成、生成规则：brokerIP+物理分区的offset

​	key：由用户生成的业务相关的唯一标识

6、消费者组和生产者组，多个消费者和生产者的集合

​	生产者producer通过MQ的负载均衡模块选择相应的Broker集群队列进行消息投递，支持快速失败和低延迟。一个生产者组发送相同Topic类型的消息

​	消费者负责消费消息，从Broker服务器中获取消息，并且对消息进行相关业务处理，实现负载均衡(是Queue的负载均衡)和容错(一个consumer挂了，consumer组里的其余的consumer可以继续消费)。

一个消费者组消费相同Topic类型的消息。

​	一个消费者可以消费多个queue，但是不能一个queue被多个消费者消费，消费者组必须消费同一个topic

## Offset

​	每个broker中的queue在收到消息的时候会记录offset，初始值为0，每记录一条消息offset会递增+1

minoffset最小值

maxoffset最大值

consumerOffset消费者消费进度/位置

diffTotal

消息积压/违背消费的消息数量

offsetStore-记录消费进度，offset由谁来维护：如果是广播，在本地记录offset，如果是集群，在消费者记录offset

## Broker

##### 			1、作用：负责接收、存储、转发消息，同时为消费者的拉取请求作准备，Broker同时也存储着消息相关的元数据，包括：消费进度偏移量offset、主题、队列等等

​	Broker面向producer和consumer接受和发送消息	

​	nameserver提交自己的信息

​	是消息中间件的消息存储、转发服务器

​	是每个Borker节点，在启动的时候都会遍历NameServer列表，与nameserver建立长连接，注册自己的信息，然后定时上报

##### 	2、Broker集群

​	Broker高可用，可以配置成Mster/slave集群的方式，Master可写可读，Slave只可以读，Master将写入的数据同步给Slave

​	一个Master可以对应多个Slave，但是一个Slave只能对应一个Master

​	Master与Slave的对应关系通过指定相同的BrokerName，不同的BrokerID来定义，BrokerID为0的表示Master，非0表示Slave。

​	Master多机负载，可以部署多个Broker，每个Broker与nameserver集群中的所有节点建立长连接，定时注册Topic信息到所有的NameServer

## Producer

​	消息的生产者

​	通过集群中的一个节点(随机选择)建立长连接，获得Topic的路由信息，包括Topic下面有哪些Queue，这些Queue分布在哪些Broker上等等。

​	接下来向提供Topic服务的Master建立长连接，并且定时向Master发送心跳

### Consumer

​	消息的消费者，通过NameServer集群获得Topic的路由信息，连接到对应的Broker上消费消息

​	注意，由于Master-Slave都可以读消息，所以Consumer会与Master/Slave都建立连接

### NameServer

​	底层由netty实现，提供了路由管理、服务器注册、服务发现的功能，是一个无状态的节点

​	nameserver是服务发现者，集群中各个角色(producer、broker、consumer等等)都需要定时向nameserver上报自己的状态，以便互相发现彼此，超时不上报，nameserver把它从列表中剔除

​	nameserver可以部署多个，当多个nameserver存在的时候，其他角色同时上报他们的信息，以保证高可用

​	Nameserver集群间互不通信，没有主备的概念

​	NameServer是内存式存储，没有持久化的概念，因为地址信息列表可能会变，nameserver中broker、topic等信息默认不会持久化

​	为什么不用Zookeeper？因为rocketmq希望提高性能，CAP定理，客户端负载均衡。

### Rocket的hello word

##### 1、同步消息：

```java
/*
*producer
*/
public class Producer{
    public static void main(String[] args){
        DefaultMQProducer producer = new DefaultMQProducer("xoxogp");
        
        //设置nameserver地址
        producer.setNameSrvAddr("192.168.150.113:9876");
        producer.start();
        
        // topic 消息将要发送到的地址
        // body 实际数据，消息是和topic绑定的，要从nameserver中去找topic在哪个broker
        Messge msg = new Message(topic,body); // topic是字符串 // body只接受字符数组
        
        SendResult sendResult = producer.send(msg);
        // sendStatus、msgID、offsetMsgID、MessageQueue(topic、brokername、queueid)
        // 正常情况下发送消息就要关闭连接--这是同步消息
        // 发完之后等broker返回状态，保证不会丢消息
        producer.shutdown();
        System.out.println("已经停机");
    }
}
```

```java
public class Consumer{
	public static void main(String[] args){
		DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("xxoocsm");
       	
        // nameserver的地址
        consumer.setNameSrvAddr("192.168.150.113:9876");
        
        // 每个consumer关注一个topic
        // topic 关注的topic   过滤器，*表示不过滤，全部消息都要
        comsumer.subscribe("myTopic001","*")
        
        consumer.registerMessageListener(new MessageListenerConcurrently()){
            public ConsumerConcurrentlyStatus consumerMessage(List<MessageExt> msgs,ConsumerConcurrentlyContext context){
                for(MessageExt msg:msgs){
                    System.out.println(new String(msg.getBody()));
                }
                // 默认情况下 这条消息只会被一个consumer消费到，点对点消息
                // message 状态修改
                // ack 如果broker长时间没有收到ack，就会让其他消费者消费消息。
                return ConsumerConcurrentlyStatus.CONSUME_SUCCESS;
            }
        };
        consumer.start();
        System.out.println("Consumer start...");
	}
}
```

#### 相关API

##### 2、一条条同步发会阻塞、批量发送消息：

```java
ArrayList<Message> list = new ArrayList<Message>();
list.add(msg1);
list.add(msg2);
// 同步发送
SendResult sendResult = producer.send(list);

producer.send(Collection c) // 接受一个集合实现批量发送
```

批量消息要求必须要具有同一个topic，相同消息配置

不支持延迟消息，

##### 3、异步可靠消息，不会阻塞等待broker的确认，而是采用事件监听的方式接收broker返回的确认

```java
        producer.setNameSrvAddr("192.168.150.113:9876");
        producer.start();
        
        // topic 消息将要发送到的地址
        // body 实际数据，消息是和topic绑定的，要从nameserver中去找topic在哪个broker
        Messge msg = new Message(topic,body); 
        // 用事件监听的方式接收broker返回的确认
		producer.setRetryTimesWhenSendAyncFailed(retryTimesWhenSendAyncFailed) // 设置超时重传时间
		producer.send(msg,new SendCallback(){
            public void onSuccess(SendResult sendResult){
                System.out.println("消息发送成功");
                System.out.println("sendResult,"+sendResult);
            }
            public void onException(Throwable e){
                // 如果发生异常case异常，尝试重投或者调整业务
                System.out.println("消息发送异常");
            }
        });
       // 异步消息不能关，因为你不知道什么时候回调ack
        System.out.println("已经停机");

```

批量消息要求必须要具有同一个topic，相同消息配置

##### 4、单向消息，不需要broker应答，只管发送，效率最高sendOneway

```java
		producer.setNameSrvAddr("192.168.150.113:9876");
		producer.start();
        
        // topic 消息将要发送到的地址
        // body 实际数据，消息是和topic绑定的，要从nameserver中去找topic在哪个broker
        Messge msg = new Message(topic,body); 
		// 只管发送，有没有发出去不管        
		producer.sendOneway(message);
```

##### 5、广播消息和集群消费，消费消息能否被广播消费，或者集群消费，是通过消费者去设置的

##### 	默认是集群，消费消息能否被广播消费，或者集群消费，是通过消费者去设置的

​	集群消费-->一群consumer，集群里面有一个消费就行，点对点的方式。

##### 	consumer.setMessageModel(MessageModel.CLUSTERING);

​	广播消费-->消息会给所有订阅了topic的消费者，每一个消费者都要消费

##### 	consumer.setMessageModel(MessageModel.BRPADCASTING);

```java
consumer.setMessageModel(MessageModel.BRPADCASTING);
consumer.start();
```

##### 6、消息过滤 TAG和Key

##### 	TAG和Keys  一般TAG用来分组，key用于标识业务

​	TAG用来过滤消息的，或者是消息分组，

```java
producer.setNameSrvAddr("192.168.150.113:9876");
producer.start();
        
// 设置TAG为TAG-A
Messge msg = new Message(topic,"TAG-A",body); 
      
// 设置TAG为TAG-A，KEY为KEY-XXX
Messge msg = new Message(topic,"TAG-A","KEY-XXX",body); 
producer.sendOneway(message);
```

​	消费的时候用过滤器，指定TAG消费

```java
// 每个consumer关注一个topic
// topic 关注的topic   过滤器，*表示不过滤，全部消息都要
comsumer.subscribe("myTopic001","TAG-A")
```

##### 7、SQL表达式过滤：消费者将收到TAGA或者TAGB的消息，但是限制是一条消息只能有一个标签，对于这种情况可以用SQL表达式筛选出消息，用公式或者条件表达式

​	1、数字比较：> ,<=,<,<=

​	2、between什么的

##### 	consumer.setMessageModel(MessageModel.CLUSTERING);

​	广播消费-->消息会给所有订阅了topic的消费者，每一个消费者都要消费

##### 	consumer.setMessageModel(MessageModel.BRPADCASTING);

```java
consumer.setMessageModel(MessageModel.BRPADCASTING);
consumer.start();
```

##### 如果消息分组，一组最好一个过滤器，不能随便乱用！！！！

DefaultMQProducer producer = new DefaultMQProducer("group--A");

##### 需要特别注意，tag selector 在一个group中的消费者，都不能随便变，要保持统一

##### 8、使用SQL去过滤

​	如果有很多消息源源不断过来

​	大数据，能用SQL拿到需要的东西，或者是做初步的筛选

​	了解即可

生产者生产的时候：

```java
for(int i=1;i<=100;i++){
    Message message = new Message("myTopic03","TAG-B","KEY-XX","XXOOXX".getBytes());
    message.putUserProperty("age",String.valueOf(i)); // 给消息设置属性列，消费的时候按照属性列值来过滤
    producer.send(messge);
}
```

消费的时候：

```java
MessageSelector messageSelector = MessageSelector.bySql("age>=18 and age<=28");
consumer.subscribe("myTopic003",messageSelector);
```

##### 9、RocketMQ事务

​	事务消费，事务commit提交之后才会MQ真正可用，consumer才能真正消费

扩展：在分布式事务解决数据一致性

RocketMQ实现2pc，两阶段提交

##### 2pc：两阶段提交，角色A和B，A第一步提交数据给B，如果本地事务执行OK，第二阶段再提交一次确认OK，数据库事务就是两阶段提交，第一次提交不会立刻生效，而是第二阶段确定OK之后状态才可用状态，如果是roll back就是不可用

##### tcc：三阶段提交，分为3步

##### 	try-confirm-cancel：try就是A先尝试写数据，B要维护锁这个数据的操作，A执行本地事务，如果commit就去B执行confirm，并且释放锁，如果失败rollback，就去B实现cancel，释放锁，数据变成不可用。

发送端-->

​	开启事务-->

​		producer端需要发送消息HFmessage给broker-->

​			broker接收到之后标记一下写入HFmessage队列-->

​				写入队列之后，同步或者异步刷盘-->

​						给producer发送确认信号-->

​							producer执行本地事务，可能会rollback，如果rollback，撤掉HFmessage里面的消息，如果commit，也撤掉，然后写入到真正的消息队列中。

​	broker为了避免长等待，开启定时任务，取事务相关信息，回调producer的方法定时检查发送端事务有没有执行完。

​	发送的消息commit之后，消息会被broker投递到队列中，消费者可以消费了。

Half Message：预处理消息，当broker收到此类消息之后，会存储到RMQ_SYS_Trans_topic的消息消费队列中

##### 如果检查的时候超时了：超过回查次数，就默认回滚消息

#### TransactionListener的两个方法：

1、executeLocalTransaction

​	半消息发送成功触发此方法来执行本地事务

2、checkLocalTransaction

​	broker将发送检查消息来检查事务状态，并将调用此方法来获取本地事务状态

##### 本地事务执行状态：

LocalTransactionState.COMMIT_MESSAGE 				执行成功，确认提交

LocalTransactionState.COMMIT_ROLLBACK_MESSAGE，	回滚消息，broker会删除半消息上

LocalTransactionState.UNKNOW 						暂时为未知状态，等待broker回查

```java
TransactionMQProducer producer = new TransactionMQProducer("xoxogroup");

producer.setNameServerAddr("192.168.150.113:9876");

// 回调
producer.setTransactionListener(new TransactionListener()){
    public LocalTransaactionState executeLocalTransaction(Message msg,Object arg){
        // 执行本地事务
        // 同步执行里面的方法业务
        // 如果有一个抛出异常，return rollback_Message
        // 如果成功 return LocalTransactionState.COMMIT_MESSAGE;
        // 如果等待 return LocalTransactionState.UNKNOW; 接着回调 checkLocalTransaction来判断
        // 半消息和业务无关，只有commit_MESSAGE才会让消息可用
        System.out.println("msg:"+msg.getTransactionId());
    }
    
    
     public LocalTransaactionState checkLocalTransaction(Message msg){
        // Broker 端回调，检查事务
        // 事务执行成功
        return LocalTransactionState.COMMIT_MESSAGE;
        // 等待
        return LocalTransactionState.UNKNOW;
        // 回滚
        return LocalTransactionState.COMMIT_ROLLBACK_MESSAGE;
        
    }
}
```



##### 10、重投机制与重新消费

##### 	Producer 默认超时时间

​	设置重发次数，如果超时没有收到ack确认信号，就重发消息

```java
// timeout for sending message
private int sendMsgTimeout = 3000;

// 异步发送时，重试次数，默认2
producer.setRetryTimesWhenSendASyncFailed(1);

// 同步发送时，重试次数，默认2
producer.setRetryTimesWhenSendFailed(1);

// 是否向其他broker发送请求，默认false
producer.setRetryAnotherBrokerWhenNotStoreOK(true);

```

##### Consumer

```java
// 消费超时，单位分钟，超过了就表示消费失败
consum.setConsumerTimeout();
// 发送ack，消费失败的时候，不去发SUCCESS，如果发送RECONSUME_LATER，producer就会重投
RECONSUME_LATER
```

##### broker投递

只有消息模式为MessageModel.CLUSTERING集群模式时，Broker才会自动进行重试，广播消息不重试，重投使用messageDelayLevel

```java

```

##### 11、如何保证消息的顺序消费

##### 	Producer 默认超时时间

​	设置重发次数，如果超时没有收到ack确认信号，就重发消息

```java
// timeout for sending message
private int sendMsgTimeout = 3000;

// 异步发送时，重试次数，默认2
producer.setRetryTimesWhenSendASyncFailed(1);

// 同步发送时，重试次数，默认2
producer.setRetryTimesWhenSendFailed(1);

// 是否向其他broker发送请求，默认false
producer.setRetryAnotherBrokerWhenNotStoreOK(true);

```

##### Consumer

### 面试QA：

#### 1、消息丢失：

SendResult

​	producer在发送同步/异步可靠消息之后，会接收到SendResult，表示消息发送成功，SendResult其中属性sendStatus表示了broker是否真正完成了消息存储

​	如果sendStatus!="ok"的时候，应该重新发送消息，避免消息丢失

​	当producer.setRetryAnotherBrokerWhenNotStoreOK

#### 2、消息重复消费

引起消息正常发送和消费的原因是网络不稳定性

#### 引起重复消费的原因：

##### 1、ACK：

​	正常情况下在consumer真正消费完消息之后应该发送ack，通知broker该消息已经正常消费，从queue中剔除，当ack因为网络原因无法发送到broker。broker会认为消息没有被消费，会开启消息的重投机制把消息再次投递到consumer

##### 2、group：

​	在CLUSTERING模式下，消息在broker中会保证相同group的consumer消费一次，但是针对不同group的consumer会推送多次

集群模式，会保证每个集群都要消费一次，可能会有业务重叠的问题

#### 解决方案

​	消息库表：处理消息之前，使用消息主键messageKey在表里带有约束的字段中，insert，messageKey唯一，写入失败就不消费了

​	Map：单机的时候，使用map，concurrentHashMap，putIfAbsent，失败就不消费

​	redis：使用MessageKey主键或者set操作，分布式幂等用redis，如果消息重投的时候ID是一样，但是messageKey是不一样的。

### 怎么保证顺序消费？

​	1、同一个Topic

​	2、同一个Queue，发的时候也不能乱，一个线程去发送一堆消息。

​	3、发消息的时候一个线程去发消息

​	4、消费的时候一个线程消费一个queue里面的消息

​	5、多个queue只能保证单个queue里面的顺序

​	在发送的时候指定queue发送，就可以保证先进先出

```java
// 保证在同一个Queue队列中
producer.send(msg , MessageQueueSelector selector)
    
producer.send(msg,new MessageQueueSelector(){
    public MessageQueue select(
        List<MessageQueue> mqs,Message msg,Object arg){
        // 当前topic里面所有的queue
        // 具体要发的那条消息
        // 对应send方法里面的arg
        msg.get((Integer)(arg)));
        // 选好的queue编号
        //固定queue
        MessageQueue queue = msg.get(0);
        return queue;
        
    }
},arg,timeout);
```

还有SelectMessageByHash 

##### producer.send(msg,new SelectMessageQueueByHash(),arg,timeout); // 按队列个数对arg 进行hash

##### producer.send(msg,new SelectMessageQueueByRandom(),arg,timeout); // 随机选queue

##### producer.send(msg,new SelectMessageQueueByMachineRoom(),arg,timeout); // 按机器

## RocketMQ的长轮询机制

​	Consumer->Borker 

​	consumer的处理能力Broker不知道

​	直接推消息broker端的压力较大，需要维护consumer的状态

​	采用长连接有可能consumer不能及时处理推送来的数据

​	pull主动权在consumer手里

短轮询：

​	client不断发送请求到server，每次都需要重新连接	

长轮询：

​	如果有数据，服务器返回报文，但是如果没有数据，服务器进入等待状态，把连接挂起，如果有消息了，服务器端会把请求response回去。但是请求一直都是客户端发起

长连接：

​	连接一旦建立就永远不断开，push方式推送。	
