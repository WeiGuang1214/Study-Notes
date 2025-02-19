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
DEL：删除一个或多个指定的key。
EXISTS：判断key是否存在，存在返回 1，不存在返回 0。
EXPIRE：给一个key设置有效期（单位：秒），有效期到期时该key会被自动删除。
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


