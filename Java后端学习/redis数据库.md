## Redis

1、Redis是键值对数据库，Key-Value

2、是一种NoSql，非关系数据库：非结构化、Key-Value和Document(由json封装)、Graph图数据库；容易重复，用Json封装

NoSQL：非SQL查询，数据结构不固定，对一致性、安全性要求不高，对性能要求高

#### Redis

特征：非结构化，键值型；非SQL查询，数据之间无关联

​	单线程，每个命令具备原子性；

​	低延迟速度快；从内存中存取，所以读写速度快

​	支持数据持久化

​	支持主从集群、分片集群，支持多语言客户端

​	非关系型数据库可以将数据拆分，存储在不同机器上，可以保存海量数据，解决内存大小有限的问题。称为水平扩展

#### SQL

特征：结构化，数据有关联，并且是SQL查询，具备事务ACID特性

​	数据结构固定，对数据的安全性、一致性要求较高，从磁盘中存取，垂直扩展

​	关系型数据库集群模式一般是主从，主从数据一致，起到数据备份的作用，称为垂直扩展；关系型数据库因为表之间存在关联关系，如果做水平扩展会给数据查询带来很多麻烦

linux命令

ps -ef |grep 查看后台进程资源

ps  -ef | grep redis 查看有没有运行

kill -9 进程号可以杀死进程

```
先将这个配置文件备份一份：
cd /usr/local/src/redis-6.2.6
cp redis.conf redis.conf.bak
修改redis.conf文件中的一些配置：
vim redis.conf
# 按/xxx就可以查找xxx相关的字段。然后按n查找下一个，N查找上一个。:noh取消高亮显示。

# 允许访问（监听）的地址，默认是127.0.0.1，会导致只能在本地访问。修改为0.0.0.0则可以在任意IP访问，生产环境不要设置为0.0.0.0
bind 0.0.0.0
# 守护进程，修改为yes后即可后台运行
daemonize yes
# 密码，设置后访问Redis必须输入密码
requirepass 123321
# 云服务器注意一定要设置密码！不然会被攻击成矿机，这里必须要设置密码才能远程访问。
# 监听的端口
port 6379
# 工作目录，默认是当前目录，也就是运行redis-server时的命令，日志、持久化等文件会保存在这个目录
dir .
# 数据库数量，设置为1，代表只使用1个库，默认有16个库，编号0~15
databases 1
# 设置redis能够使用的最大内存
maxmemory 512mb
# 日志文件，默认为空，不记录日志。可以指定日志文件名（写到/var/log/redis.log也比较好）
logfile "redis.log"

```

启动Redis：

```
# 利用redis-cli来执行 shutdown 命令，即可停止 Redis 服务，
# 因为之前配置了密码，因此需要通过 -u 来指定密码
redis-cli -u 123321 shutdown

```

可以使用`kill -9 PID`来强制终止进程服务

##### 开机自启

```
# 启动
systemctl start redis
# 停止
systemctl stop redis
# 重启
systemctl restart redis
# 查看状态
systemctl status redis
# 开机自启动
systemctl enable redis

```

​	支持主从集群、分片集群，支持多语言客户端

需要将Linux防火墙的`6379`端口开放出来，供外部其他主机客户端进行远程访问

```
# 方式一：关闭防火墙
systemctl stop firewalld

# 方式二：将6379端口添加到防火墙开放允许
# 查看当前防火墙状态
firewall-cmd --state
# 将6379端口添加到防火墙开放允许
firewall-cmd --zone=public --add-port=6379/tcp --permanent
# 重启防火墙，立即生效
firewall-cmd --reload
# 查看防火墙所有开放的端口号列表
firewall-cmd --zone=public --list-ports

```

#### Redis命令行客户端

Redis安装完成后就自带了命令行客户端：`redis-cli`，使用方式如下：

redis-cli [options] [commonds]

```
-h 127.0.0.1：指定要连接的redis节点的IP地址，默认是127.0.0.1
-p 6379：指定要连接的redis节点的端口，默认是6379
-a 密码：登录时显式的指定redis的访问密码。如果需要隐式的输入密码，需要先进入，再输入auth 密码进行认证。
```

其中的 commonds 就是Redis的操作命令，例如：

- `ping`：与redis服务端做**心跳测试**，服务端正常会返回`pong`
- 建议使用auth用密码，否则warning
- set、get可以返回键值对

#### 图形化桌面客户端

临时关闭linux防火墙

sudo systemctl stop firewalld

Redis默认有16个仓库，编号从0至15。通过配置文件可以设置仓库数量，但是不超过16，并且不能自定义仓库名称

## Redis常见命令

Redis是典型的`key-value`数据库，key一般是字符串，而value包含很多不同的数据类型：

String            hello world

Hash             {name:”JackMa",age:21}

List		[A->B->C]

Set		{A,B,C}

SortedSet	{A:1,B:2,C:3}

GEO		{A:(120.3,30.5)}

BitMap		011011010110101011

HyperLog		011011010110101011

Redis官方文档https://redis.io/commands

参考文档

### Redis通用命令

通用指令是部分数据类型的，都可以使用的指令，常见的有：

KEYS：查看符合模板的所有key。（keys * 效率低，redis单线程，不建议在生产环境设备上使用）
DEL：删除一个或多个指定的key。返回删除的数量
EXISTS：判断key是否存在，存在返回 1，不存在返回 0。
EXPIRE：给一个key设置有效期（单位：秒），有效期到期时该key会被自动删除。因为redis数据是在内存中的
TTL：查看一个KEY的剩余有效期，其中-1表示永久有效，-2表示key已到期。
通过 help [command] 可以查看一个命令的具体用法，例如：

```
# 查看keys命令的帮助信息：
127.0.0.1:6379> help keys

KEYS pattern
summary: Find all keys matching the given pattern
since: 1.0.0
group: generic

```

### String类型

String类型，也就是字符串类型，是Redis中最简单的存储类型。

其value是字符串，不过根据字符串的格式不同，又可以分为3类：

string：普通字符串
int：整数类型，可以做自增、自减操作
float：浮点类型，可以做自增、自减操作
不管是哪种格式，底层都是字节数组形式存储，只不过是编码方式不同。字符串类型的最大空间不能超过512m。数字会转为二进制存储，相对占用内存少一点，字符串会把字符转换成对应的字节码，相对占用内存多一点。

##### String的常见命令有：

SET：添加或者修改已经存在的一个String类型的键值对。
GET：根据key获取String类型的value。
MSET：批量添加多个String类型的键值对。
MGET：根据多个key获取多个String类型的value。
INCR：让一个整型的key自增1。
INCRBY：让一个整型的key自增并指定步长，例如：incrby num 2 让num值自增2。
DECR：让一个整型的key自减1。
DECRBY：让一个整型的key自增并指定步长，例如：decrby num 2 让num值自减2。
INCRBYFLOAT：让一个浮点类型的数字自增并指定步长。
SETNX：添加一个String类型的键值对，前提是这个key不存在，否则不执行。（等价于set key val nx）只有新增效果
SETEX：添加一个String类型的键值对，并且指定有效期。（添加并设置有效期，合二为一。等价于set key val ex）



#### （2）Key结构



Redis没有类似MySQL中的Table的概念，我们该如何区分不同类型的key呢？

例如，需要存储用户、商品信息到redis，有一个用户id是1，有一个商品id恰好也是1，此时如果使用id作为key，那就会冲突了，该怎么办？

Redis的key允许有多个单词形成层级结构，多个单词之间用`:`隔开，格式如下：

​	项目名:业务名:类型:id

这个格式并非固定，也可以根据自己的需求来删除或添加词条。这样就可以把不同类型的数据区分开了。从而避免了key的冲突问题。

例如我们的项目名称叫 heima，有user和product两种不同类型的数据，我们可以这样定义key：

- user相关的key：**heima:user:1**
- product相关的key：**heima:product:1**
- 在Redis的桌面客户端中，还会以相同前缀作为层级结构，让数据看起来层次分明，关系清晰。

如果命令行查询时碰到中文乱码，输exit退出后，再重新打开客户端，并在后面添加 --raw，如redis-cli --raw

### Hash类型

`Hash`类型，也叫散列，其value是一个无序字典，类似于Java中的`HashMap<String, HashMap<String, Object>>`结构。

String结构是将对象序列化为`JSON`字符串后存储，当需要修改对象某个字段时很不方便：

Hash结构可以将对象中的**每个字段独立存储**，可以针对单个字段做CRUD：

即value分为field和value(键值对)

Hash的常见命令有：

HSET key field value：添加或者修改hash类型key的field的值。

HGET key field：获取一个hash类型key的field的值。

HMSET：批量添加多个hash类型key的field的值。（hmset 和 hset 效果相同 ，redis4.0之后hmset弃用了，直接用hset即可）

HMGET：批量获取多个hash类型key的field的值。

HDEL key field [field ...]：删除hash类型key的一个或多个field。

HGETALL：获取一个hash类型的key中的所有的field和value。

HKEYS：获取一个hash类型的key中的所有的field。

HINCRBY：让一个hash类型key的字段值自增并指定步长。

HSETNX：添加一个hash类型的key的field值，前提是这个field不存在，否则不执行。

### List类型

Redis中的List类型与Java中的LinkedList类似，可以看做是一个双向链表结构（实际结构更为复杂）。既支持正向检索和也支持反向检索。

特征也与LinkedList类似：

有序
元素可以重复
插入和删除快
查询速度一般
常用来存储一个有序数据，例如：朋友圈点赞列表，评论列表等。

List的常见命令有：

LPUSH key element ... ：向列表左侧插入一个或多个元素。
LPOP key [count]：移除并返回列表左侧的第一个元素，没有则返回nil。count可以指定弹出元素的个数。
RPUSH key element ... ：向列表右侧插入一个或多个元素。
RPOP key [count]：移除并返回列表右侧的第一个元素。
LRANGE key start end：返回一段角标范围内的所有元素。
BLPOP和BRPOP：与LPOP和RPOP类似，只不过在没有元素时等待指定时间，而不是直接返回nil。例如BRPOP key1 [key2] timeout：移出并获取列表中的最后一个元素，需要指定超时时间（秒），如果列表中没有元素会阻塞列表直到等待超时或发现可弹出元素为止，超时结束阻塞返回nil。
LLEN：返回列表长度。



如何利用List结构模拟一个栈?

入口和出口在同一边（LPUSH和LPOP、RPUSH和RPOP）。
如何利用List结构模拟一个队列?

入口和出口在不同边（LPUSH和RPOP、RPUSH和LPOP）。
如何利用List结构模拟一个阻塞队列?

入口和出口在不同边。
出队时采用BLPOP或BRPOP。

### 6、Set类型

Redis的`Set`集合结构与Java中的`HashSet`类似，可以看做是一个value为null的HashMap。因为也是一个hash表，因此具备与HashSet类似的特征：

- **无序**
- **元素不可重复**
- **查找快**
- **支持交集、并集、差集等功能**

Set的常见命令有：

SADD key member ... ：向set中添加一个或多个元素。
SREM key member ... ：移除set中的指定元素。
SCARD key：返回set中元素的个数。
SISMEMBER key member：判断一个元素是否存在于set中。（类似于contains）
SMEMBERS：获取set中的所有元素（注意是无序的）。

SINTER key1 key2 ... ：求key1与key2的交集。
SUNION key1 key2 ... ：求key1与key2的并集。
SDIFF key1 key2 ... ：求key1与key2的差集。



#### SortedSet（ZSet）类型

Redis的SortedSet是一个可排序的set集合，与Java中的TreeSet有些类似，但底层数据结构却差别很大。SortedSet中的每一个元素都带有一个score属性，可以基于score属性对元素排序，底层的实现是一个跳表（SkipList）加 hash表。

##### SortedSet具备下列特性：

可排序
元素不重复（value不可相同，score可以相同）
查询速度快
因为SortedSet的可排序特性，经常被用来实现排行榜这样的功能。

##### SortedSet的常见命令有：

ZADD key score member：添加一个或多个元素到sorted set ，如果已经存在则更新其score值。
ZREM key member：删除sorted set中的一个指定元素。
ZSCORE key member : 获取sorted set中的指定元素的score值。
ZRANK key member：获取sorted set 中的指定元素的排名（排序后的索引位置）。

降序：Zrevrank key member

ZCARD key：获取sorted set中的所有元素个数。
ZCOUNT key min max：统计score值在给定范围内（闭区间）的所有元素的个数。
ZINCRBY key increment member：让sorted set中的指定元素自增，步长为指定的increment值。
ZRANGE key min max：按照score排序后，获取指定排名范围（从0开始）内的member值。
ZRANGEBYSCORE key min max：按照score排序后，获取指定score范围内的元素。
ZDIFF、ZINTER、ZUNION：求差集、交集、并集。

##### 注意：所有的排名默认都是升序，如果要降序则在命令的Z后面添加REV即可，例如：

升序获取 sorted set 中的指定元素的排名：ZRANK key member。
降序获取 sorted set 中的指定元素的排名：ZREVRANK key memeber。

##### 面试题：ZSet中如果score值相同，会按照member的字典序进行排序，前后比较顺序为：数字 > 大写字母 > 小写字母。

##### Jedis

标记为❤的就是Redis官方推荐使用的java客户端，包括：

Jedis：以Redis命令作为方法名称，学习成本低，简单实用但是Jedis实例是线程不安全的，多线程环境下需要用连接池为每一个线程创建独立的Jedis连接。
lettuce：是基于Netty实现的，支持同步、异步和响应式编程方式，并且是线程安全的。支持Redis的哨兵模式、集群模式和管道模式。
Redisson：是在Redis基础上实现了分布式、可伸缩的Java数据结构集合，例如Map、Queue等，而且支持跨进程的同步机制：Lock、Semaphore等待，比较适合用来实现特殊的功能需求。（后续分布式锁章节来学习Redisson）

##### 新建单元测试类

```java
private Jedis jedis;

@BeforeEach
void setUp() {
    // 建立连接
    jedis = new Jedis("192.168.8.100", 6379);
    // 设置密码
    jedis.auth("123321");
    // 选择仓库
    jedis.select(0);
}
//测试
@Test
void testString() {
    // 存入数据
    String result = jedis.set("name", "Aizen");
    System.out.println("result = " + result);
    // 获取数据
    String name = jedis.get("name");
    System.out.println("name = " + name);
}

@Test
void testHash() {
    // 插入hash数据
    jedis.hset("user:1", "name", "Jack");
    jedis.hset("user:1", "age", "21");
    // 获取hash数据
    Map<String, String> map = jedis.hgetAll("user:1");
    System.out.println(map);
}
//释放资源
@AfterEach
void tearDown() {
    // 关闭连接
    if (jedis != null) {
        jedis.close();
    }
}

```

#### Jedis连接池

Jedis本身是线程不安全的，并且频繁的创建和销毁连接会有性能损耗，因此我们推荐大家使用Jedis连接池代替Jedis的直连方式

有关池化思想，并不仅仅是这里会使用，很多地方都有，比如说我们的数据库连接池，比如我们tomcat中的线程池，这些都是池化思想的体现。

##### 创建Jedis连接池

```java
/**
 * Jedis连接池
 */
public class JedisConnectionFactory {
    private static final JedisPool jedisPool;

    static {
        // 配置连接池
        JedisPoolConfig poolConfig = new JedisPoolConfig();
        poolConfig.setMaxTotal(8); // 最大连接数
        poolConfig.setMaxIdle(8);   // 最大空闲连接数
        poolConfig.setMinIdle(0);   // 最小空闲连接，如果一直没有被访问，资源就会释放
        poolConfig.setMaxWait(Duration.ofMillis(1000)); // 等待时长，当连接池里没有连接可用，需要等待多长时间，默认为-1一直等
        // 创建连接池对象
        jedisPool = new JedisPool(poolConfig, "192.168.8.100", 6379, 1000, "123321");
    }

    public static Jedis getJedis() {
        return jedisPool.getResource();
    }
}

```

- 改造原始代码

- ```java
  @BeforeEach
  void setUp() {
      // 建立连接
      //jedis = new Jedis("192.168.8.100", 6379);
      jedis = JedisConnectionFactory.getJedis();
      // 设置密码
      jedis.auth("123321");
      // 选择仓库
      jedis.select(0);
  }
  
  @AfterEach
  void tearDown() {
      // 关闭连接
      if (jedis != null) {
          jedis.close();
      }
  }
  
  // Jedis底层的close方法
  @Override
  public void close() {
      if (dataSource != null) {
          Pool<Jedis> pool = this.dataSource;
          this.dataSource = null;
          if (isBroken()) {
              pool.returnBrokenResource(this);
          } else {
              pool.returnResource(this);
          }
      } else {
        connection.close();
      }
  }
  
  ```

  1） JedisConnectionFacotry：工厂设计模式，通过工厂来获得连接池中的Jedis对象，而不用直接去new对象，降低耦合。
  2）静态代码块：随着类的加载而加载，确保只能执行一次，我们在加载当前工厂类的时候，就可以执行static的操作完成对连接池的初始化。
  3）使用了连接池创建后，当关闭连接并不是释放资源，而是将Jedis资源归还给连接池。

### 2、SpringDataRedis

SpringData是Spring中数据操作的模块，包含对各种数据库的集成，其中对Redis的集成模块就叫做SpringDataRedis，

提供了对不同Redis客户端的整合（Lettuce和Jedis）
提供了RedisTemplate统一API来操作Redis
支持Redis的发布订阅模型
支持Redis哨兵和Redis集群
支持基于Lettuce的响应式编程
支持基于JDK.JSON.字符串.Spring对象的数据序列化及反序列化
支持基于Redis的JDKCollection实现
SpringDataRedis中提供了RedisTemplate工具类，其中封装了各种对Redis的操作。并且将不同数据类型的操作API封装到了不同的类型中：

​								返回值类型			说明

redisTemplate.opsForValue()         ValueOperations		操作string类型数据
redisTemplate.opsForHash()	  HashOperations	 	操作Hash类型数据

redisTemplate.opsForList()	     ListOperations	       操作List类型数据
redisTemplate.OpsForSet()	     SetOperations		操作Set类型数据
redisTemplate.OpsForZSet()	ZSetOperations	操作SortedSet类型数据
通用的命令redisTemplate

- 导入pom依赖

- ```java
  <dependencies>
    <!-- spring-data-redis -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-data-redis</artifactId>
    </dependency>
    <!-- commons-pool2连接池 -->
    <dependency>
      <groupId>org.apache.commons</groupId>
      <artifactId>commons-pool2</artifactId>
    </dependency>
    <!-- Jackson -->
    <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-databind</artifactId>
    </dependency>
    <dependency>
      <groupId>org.projectlombok</groupId>
      <artifactId>lombok</artifactId>
      <optional>true</optional>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
    </dependency>
  </dependencies>
  
  <build>
    <plugins>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
        <configuration>
          <excludes>
            <exclude>
              <groupId>org.projectlombok</groupId>
              <artifactId>lombok</artifactId>
            </exclude>
          </excludes>
        </configuration>
      </plugin>
    </plugins>
  </build>
  
  ```

properties配置文件

```Java
# spring-data-redis
spring:
  data:
    redis:
      host: 192.168.8.100
      port: 6379
      password: 123321
      lettuce:
        pool: # lettuce的pool必须手动配置才会生效
          max-active: 8 # 最大连接
          max-idle: 8 # 最大空闲连接
          min-idle: 0 # 最小空闲连接
          max-wait: 1000ms  # 连接等待时间

```

- 注入RedisTemplate直接使用

- ```Java
  @SpringBootTest
  class SpringDataRedisTests {
      @Autowired
      private RedisTemplate<String, Object> redisTemplate;    // 注入RedisTemplate
  
      @Test
      void testString() {
          // 写入一条String数据
          redisTemplate.opsForValue().set("name", "虎哥");
          // 获取String数据
          Object name = redisTemplate.opsForValue().get("name");
          System.out.println("name = " + name);
      }
  }
  
  ```

  

只不过写入前会把Object序列化为字节形式，默认是采用`JDK序列化`（`JdkSerializationRedisSerializer`），JDK序列化底层使用的是`ObjectOutputStream`将`Java对象转为字节`后写入Redis，得到的结果是字节序列：

\xAC\xED\x00\x05t\x00\x12\xE6\x88\x91\xE8\xB8\x8F\xE9\xA9\xAC\xE6\x9D\xA5\xE8\xBE\xA3\xEF\xBC\x81

- 可读性差
- 内存占用较大

我们可以创建一个RedisConfig配置类，自定义RedisTemplate的序列化方式。

```java
@Configuration
public class RedisConfig {
    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory connectionFactory) {
        // 创建RedisTemplate对象
        RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
        // 设置连接工厂
        redisTemplate.setConnectionFactory(connectionFactory);
        // 创建JSON序列化工具
        GenericJackson2JsonRedisSerializer jsonRedisSerializer = new GenericJackson2JsonRedisSerializer();
        // 设置Key和HashKey的序列化：String
        redisTemplate.setKeySerializer(RedisSerializer.string());
        redisTemplate.setHashKeySerializer(RedisSerializer.string());
        // 设置Value和HashValue的序列化方式为：json
        redisTemplate.setValueSerializer(jsonRedisSerializer);
        redisTemplate.setHashValueSerializer(jsonRedisSerializer);
        return redisTemplate;
    }
}

```

采用了JSON序列化来代替默认的JDK序列化方式

整体可读性有了很大提升，并且能将Java对象自动的序列化为JSON字符串，并且查询时能自动把JSON反序列化为Java对象。不过，其中记录了序列化时对应的`class全类名`，目的是为了查询时实现`自动反序列化`。这会带来额外的`内存开销`。

尽管JSON的序列化方式可以满足我们的需求，但依然存在一些问题，如图：

为了在反序列化时知道对象的类型，JSON序列化器会将类的class类型写入json结果中，存入Redis，会带来额外的内存开销。

为了减少内存的消耗，我们可以采用手动序列化的方式，换句话说，就是不借助默认的序列化器，而是我们自己来控制序列化的动作，同时，我们只采用String的序列化器，这样在存储value时，我们就不需要在内存中就不用多存储数据，从而节约我们的内存空间。

```Java
@SpringBootTest
class SpringRedisTemplateTests {
    @Autowired
    private StringRedisTemplate stringRedisTemplate;    // 注入StringRedisTemplate

    @Test
    void testString() {
        // 写入一条String数据
        stringRedisTemplate.opsForValue().set("name", "虎哥");
        // 获取String数据
        Object name = stringRedisTemplate.opsForValue().get("name");
        System.out.println("name = " + name);
    }

    private static final ObjectMapper mapper = new ObjectMapper();

    @Test
    void testSaveUser() throws JsonProcessingException {
        // 创建对象
        User u = new User("虎哥", 21);
        // 手动序列化
        String json = mapper.writeValueAsString(u);
        // 写入User对象（将Java对象序列化为json）
        stringRedisTemplate.opsForValue().set("user:200", json);
        // 获取数据（将json反序列化为Java对象）
        String jsonUserStr = stringRedisTemplate.opsForValue().get("user:200");
        // 手动反序列化
        User user = mapper.readValue(jsonUserStr, User.class);
        System.out.println("user = " + user);
    }
}

```

总结：RedisTemplate的两种序列化实践方案

方案一：
自定义RedisTemplate
修改RedisTemplate的序列化器为GenericJackson2JsonRedisSerializer
方案二：
使用StringRedisTemplate
写入Redis时，手动把对象序列化为JSON
读取Redis时，手动把读取到的JSON反序列化为对象
