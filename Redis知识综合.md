## Redis知识综合

### 超哥博客：

### https://www.cnblogs.com/pyyu/p/9843950.html

### 一、redis基础

##### 1.redis的特性

```python
Redis是一个开源（BSD许可）的、内存中的数据结构存储系统，它可以用作数据库、缓存和消息中间件；
redis是c语言编写的，支持数据持久化，是key-value类型数据库；
redis支持数据备份，也就是master-slave模式；
```

##### 2.redis的优势

```
性能高，读取速度10万次每秒
写入速度8万次每秒
所有操作支持原子性

用作缓存数据库，数据放在内存中
替代某些场景下的mysql，如社交类app
大型系统中，可以存储session信息，购物车订单
```

##### 3.redis的安装

###### 3.1yum安装redis

```python
#前提得配置好阿里云yum源，epel源
#查看是否有redis包
yum list redis
#安装redis
yum install redis -y
#安装好，启动redis
systemctl start redis
```

###### 3.2检查redis是否工作

```python
#在redis客户端执行
redis-cli    
#进入交互式环境后，执行ping，返回pong表示安装成功
127.0.0.1:6379> ping
PONG
```

###### 3.3源码编译安装redis

```python
# 编译安装的优势
	-编译安装时可以指定扩展的module（模块）；
	-编译安装可以统一安装路径，linux软件约定安装目录在/opt/下面；
	-软件仓库版本一般比较低，编译源码安装可以根据需求，安装最新的版本；
    
# 安装步骤
1.下载redis源码
wget http://download.redis.io/releases/redis-4.0.10.tar.gz
2.解压缩
tar -zxf redis-4.0.10.tar.gz
3.切换redis源码目录
cd redis-4.0.10.tar.gz
4.编译源文件
make 
5.编译好后，src/目录下有编译好的redis指令
6.make install 安装到指定目录，默认在/usr/local/bin
```

##### 4.redis可执行文件

```python
./redis-benchmark        #用于进行redis性能测试的工具
./redis-check-dump       #用于修复出问题的dump.rdb文件
./redis-cli              #redis的客户端
./redis-server           #redis的服务端
./redis-check-aof        #用于修复出问题的AOF文件
./redis-sentinel         #用于集群管理
```

##### 5.redis配置文件

```python
redis配置文件名为 redis.conf，这个文件可以自定义

# redis.conf核心配置项
#绑定ip，如需要远程访问，需要填写服务器ip
bind 127.0.0.1  

#端口，redis启动端口
port 

#守护进程方式运行
daemonize yes

#rdb数据文件
dbfilename dump.rdb

#数据文件存放路径
dir /var/lib/redis/

#日志文件
logfile /var/log/redis/redis-server.log

#主从复制
slaveof 
```

##### 6.redis数据结构

```python
redis是一种高级的key：value存储系统，其中value支持五种数据类型
字符串（strings）
哈希 （hashes）
列表（lists）
集合（sets）
有序集合（sorted sets）

# 基本命令
keys *         查看所有key
type key       查看key类型
expire key seconds    过期时间
ttl key        查看key过期剩余时间        -2表示key已经不存在了
persist        取消key的过期时间         -1表示key存在，没有过期时间
exists key     判断key存在           存在返回1    否则0
del keys       删除key    可以删除多个
dbsize         计算key的数量
```

###### 6.1string类型

```python
# 方法
set 　　设置key
get   获取key
append  追加string
mset   设置多个键值对
mget   获取多个键值对
del  删除key
incr  递增+1
decr  递减-1

# 示例
127.0.0.1:6379> set name 'yu'   #设置key
OK
127.0.0.1:6379> get name　　　　#获取value
"yu"
127.0.0.1:6379> set name 'yuchao'　　#覆盖key
OK
127.0.0.1:6379> get name　　　　#获取value
"yuchao"
127.0.0.1:6379> append name ' dsb' 　　#追加key的string
(integer) 10
127.0.0.1:6379> get name　　#获取value
"yuchao dsb"


127.0.0.1:6379> mset user1 'alex' user2 'xiaopeiqi'　　　　#设置多个键值对
OK
127.0.0.1:6379> get user1　　　　#获取value
"alex"
127.0.0.1:6379> get user2　　　　#获取value
"xiaopeiqi"
127.0.0.1:6379> keys *　　　　　　#找到所有key
1) "user2"
2) "name"
3) "user1"

127.0.0.1:6379> mget user1 user2 name   #获取多个value
1) "alex"
2) "xiaopeiqi"
3) "yuchao dsb"
127.0.0.1:6379> del name　　　　　　　　#删除key
(integer) 1
127.0.0.1:6379> get name　　　　　　　　#获取不存在的value，为nil
(nil)
127.0.0.1:6379> set num 10　　　　#string类型实际上不仅仅包括字符串类型，还包括整型，浮点型。redis可对整个字符串或字符串一部分进行操作，而对于整型/浮点型可进行自增、自减操作。
OK　　　　
127.0.0.1:6379> get num
"10"
127.0.0.1:6379> incr num　　　　#给num string 加一 INCR 命令将字符串值解析成整型，将其加一，最后将结果保存为新的字符串值，可以用作计数器
(integer) 11
127.0.0.1:6379> get num　　
"11"

127.0.0.1:6379> decr num　　　　　　#递减1  
(integer) 10
127.0.0.1:6379> decr num　　　　#递减1
(integer) 9
127.0.0.1:6379> get num
"9"
```

###### 6.2list类型

```python
# 方法
lpush          从列表左边插
rpush          从列表右边插
lrange         获取一定长度的元素  lrange key  start stop
ltrim          截取一定长度列表
lpop           删除最左边一个元素
rpop           删除最右边一个元素
lpushx/rpushx  key存在则添加值，不存在不处理

# 示例
lpush duilie 'alex' 'peiqi' 'ritian'  #新建一个duilie，从左边放入三个元素

llen duilie  #查看duilie长度

lrange duilie 0 -1  #查看duilie所有元素

rpush duilie 'chaoge'   #从右边插入chaoge

lpushx duilie2  'dsb'  #key存在则添加 dsb元素，key不存在则不作处理

ltrim duilie 0 2  #截取队列的值，从索引0取到2，删除其余的元素

lpop #删除左边的第一个

rpop #删除右边的第一个
```

###### 6.3sets集合类型

```python
# 方法
sadd/srem   添加/删除 元素
sismember   判断是否为set的一个元素
smembers    返回集合所有的成员
sdiff       返回一个集合和其他集合的差异
sinter      返回几个集合的交集
sunion      返回几个集合的并集

# 示例
sadd zoo  wupeiqi yuanhao  #添加集合，有三个元素，不加引号就当做字符串处理

smembers zoo  #查看集合zoo成员

srem zoo  wupeiqi #删除zoo里面的alex

sismember zoo wupeiqi  #返回改是否是zoo的成员信息，不存在返回0，存在返回1

sadd zoo wupeiqi   #再把wupeiqi加入zoo

smembers zoo  #查看zoo成员

sadd zoo2 wupeiqi mjj #添加新集合zoo2

sdiff zoo zoo2 #找出集合zoo中有的，而zoo2中没有的元素

sdiff zoo2  zoo  #找出zoo2中有，而zoo没有的元素

sinter zoo zoo1   #找出zoo和zoo1的交集，都有的元素

sunion  zoo zoo1  #找出zoo和zoo1的并集，所有的不重复的元素
```

###### 6.4有序集合

```python
#序号	#命令及描述
1	ZADD key score1 member1 [score2 member2] 
	向有序集合添加一个或多个成员，或者更新已存在成员的分数
2	ZCARD key 
	获取有序集合的成员数
3	ZCOUNT key min max 
	计算在有序集合中指定区间分数的成员数
4	ZINCRBY key increment member 
	有序集合中对指定成员的分数加上增量 increment
5	ZINTERSTORE destination numkeys key [key ...] 
	计算给定的一个或多个有序集的交集并将结果集存储在新的有序集合 key 中
6	ZLEXCOUNT key min max 
	在有序集合中计算指定字典区间内成员数量
7	ZRANGE key start stop [WITHSCORES] 
	通过索引区间返回有序集合成指定区间内的成员
8	ZRANGEBYLEX key min max [LIMIT offset count] 
	通过字典区间返回有序集合的成员
9	ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT] 
	通过分数返回有序集合指定区间内的成员
10	ZRANK key member 
	返回有序集合中指定成员的索引
11	ZREM key member [member ...] 
	移除有序集合中的一个或多个成员
12	ZREMRANGEBYLEX key min max 
	移除有序集合中给定的字典区间的所有成员
13	ZREMRANGEBYRANK key start stop 
	移除有序集合中给定的排名区间的所有成员
14	ZREMRANGEBYSCORE key min max 
	移除有序集合中给定的分数区间的所有成员
15	ZREVRANGE key start stop [WITHSCORES] 
	返回有序集中指定区间内的成员，通过索引，分数从高到底
16	ZREVRANGEBYSCORE key max min [WITHSCORES] 
	返回有序集中指定分数区间内的成员，分数从高到低排序
17	ZREVRANK key member 
	返回有序集合中指定成员的排名，有序集成员按分数值递减(从大到小)排序
18	ZSCORE key member 
	返回有序集中，成员的分数值
19	ZUNIONSTORE destination numkeys key [key ...] 
	计算给定的一个或多个有序集的并集，并存储在新的 key 中
20	ZSCAN key cursor [MATCH pattern] [COUNT count] 
	迭代有序集合中的元素（包括元素成员和元素分值）
```

```python
都是以z开头的命令！
# 示例
利用有序集合的排序，排序学生的成绩；
    127.0.0.1:6379> ZADD mid_test 70 "alex"
    (integer) 1
    127.0.0.1:6379> ZADD mid_test 80 "wusir"
    (integer) 1
    127.0.0.1:6379> ZADD mid_test 99 "yuyu"
zreverange 倒叙   zrange正序；
    127.0.0.1:6379> ZREVRANGE mid_test 0 -1 withscores
    1) "yuyu"
    2) "99"
    3) "wusir"
    4) "80"
    5) "xiaofneg"
    6) "75"
    7) "alex"
    8) "70"
    127.0.0.1:6379> ZRANGE mid_test 0 -1 withscores
    1) "alex"
    2) "70"
    3) "xiaofneg"
    4) "75"
    5) "wusir"
    6) "80"
    7) "yuyu"
    8) "99"
移除有序集合mid_test中的成员，xiaofeng给移除掉；
    127.0.0.1:6379> ZREM mid_test xiaofneg
    (integer) 1
    127.0.0.1:6379> ZRANGE mid_test 0 -1 withscores
    1) "alex"
    2) "70"
    3) "wusir"
    4) "80"
    5) "yuyu"
    6) "99"
返回有序集合mid_test的基数；
    127.0.0.1:6379> ZCARD mid_test
    (integer) 3	
返回成员的score值；
    127.0.0.1:6379> ZSCORE mid_test alex
    "70"
zrank返回有序集合中成员的排名。默认按score，从小到大排序；
    127.0.0.1:6379> ZRANGE mid_test 0 -1 withscores
    1) "alex"
    2) "70"
    3) "wusir"
    4) "80"
    5) "yuyu"
    6) "99"
    127.0.0.1:6379>
    127.0.0.1:6379>
    127.0.0.1:6379> ZRANK mid_test wusir
    (integer) 1
    127.0.0.1:6379> ZRANK mid_test yuyu
    (integer) 2
```

###### 6.5哈希数据结构

```python
哈希结构就是k1 ---> k1 : v1  如同字典套字典 { k1 : { k2: v2 }  }，取出v2必须取出k1，取出k2
hashes即哈希，哈希是从redis-2.0.0版本之后才有的数据结构。
hashes存的是字符串和字符串值之间的映射，比如一个用户要存储其全名、姓氏、年龄等等，就很适合使用哈希。
# 方法
hset     设置散列值
hget     获取散列值
hmset    设置多对散列值
hmget    获取多对散列值
hsetnx   如果散列已经存在，则不设置（防止覆盖key）
hkeys    返回所有keys
hvals    返回所有values
hlen     返回散列包含域（field）的数量
hdel     删除散列指定的域（field）
hexists  判断是否存在


redis hash是一个string类型的field和value的映射表

语法  hset key field value  

hset news1   title "first news title" #设置第一条新闻 news的id为1，添加数据title的值是"first news title"

hset news1 content "news content"    #添加一个conntent内容

hget news1 title   #获取news:1的标题

hget news1  content  #获取news的内容

hmget news1  title content   #获取多对news:1的 值

hmset news2 title "second news title" content "second Contents2"   #设置第二条新闻news:2 多个field

hmget news2 title  content #获取news:2的多个值

hkeys news1   #获取新闻news:1的所有key

hvals news1   #获取新闻news:1的所有值

hlen news1    #获取新闻news:1的长度

hdel news1 title   #删除新闻news:1的title

hlen news1     #看下新闻news:1的长度

hexists news1 title    #判断新闻1中是否有title，不存在返回0，存在返回1
```

### 二、redis的发布与订阅

##### 1.命令简介

```python
PSUBSCRIBE
    PSUBSCRIBE pattern [pattern ...]
        订阅一个或多个符合给定模式的频道，每个模式以*作为匹配符，比如it*匹配所有以it开头的频道
        
PUBLISH
    PUBLISH channel msg
        将信息 message 发送到指定的频道 channel
        
PUBSUB
    PUBSUB subcommand [argument [argument ...]]
        查看订阅与发布系统状态
        
PUNSUBSCRIBE
    PUNSUBSCRIBE [pattern [pattern ...]]
        退订指定的规则, 如果没有参数则会退订所有规则
        
SUBSCRIBE
    SUBSCRIBE channel [channel ...]
        订阅频道，可以同时订阅多个频道
        
UNSUBSCRIBE
    UNSUBSCRIBE [channel ...]
        取消订阅指定的频道, 如果不指定频道，则会取消订阅所有频道
        
#注意：
	使用发布订阅模式实现的消息队列，当有客户端订阅channel后只能收到后续发布到该频道的消息，之前发送的不会缓存，必须Provider和Consumer同时在线。
```

##### 2.发布订阅机制

```
从pub/sub的机制来看，它更像是一个广播系统：多个subscriber可以订阅多个channel，多个publisher可以往多个channel中发布消息；
可以简单理解：
Subscriber：收音机，可以收到多个频道，并以队列的方式显示；
Pulisher：电台，可以往不同的频道中发消息；
Channel：不同频率的频道
```

### 三、redis持久化RDB与AOF

```
Redis是一种内存型数据库，一旦服务器进程退出，数据库的数据就会丢失；
为了解决这个问题，Redis提供了两种持久化的方案，将内存中的数据保存到磁盘中，避免数据的丢失。
```

##### 1.RDB持久化

```
redis提供了RDB持久化的功能，这个功能可以将redis在内存中的的状态保存到硬盘中，它可以手动执行。
也可以再redis.conf中配置，定期执行。
RDB持久化产生的RDB文件是一个经过压缩的二进制文件，这个文件被保存在硬盘中，redis可以通过这个文件还原数据库当时的状态。
```

```python
#RDB(持久化)
内存数据保存到磁盘
在指定的时间间隔内生成数据集的时间点快照（point-in-time snapshot）
优点：速度快，适合做备份，主从复制就是基于RDB持久化功能实现
通过在redis中使用save命令触发RDB

#RDB配置参数：

dir /data/6379/
dbfilename  dbmp.rdb

每过900秒 有1个操作就进行持久化

save 900秒  1个修改类的操作
save 300秒  10个操作
save 60秒  10000个操作

save 900 1
save 300 10
save 60  10000
```

###### 1.1redis持久化之RDB实战

```python
1.启动redis服务端,准备配置文件
daemonize yes
port 6379
logfile /data/6379/redis.log
dir /data/6379              #定义持久化文件存储位置
dbfilename  dbmp.rdb        #rdb持久化文件
bind 10.0.0.10  127.0.0.1   #redis绑定地址
requirepass redhat          #redis登录密码
save 900 1                  #rdb机制 每900秒 有1个修改记录
save 300 10                 #       每300秒  10个修改记录
save 60  10000              #       每60秒内 10000修改记录

2.启动redis服务端

3.登录redis设置一个key
	redis-cli -a redhat
    
4.此时检查目录，/data/6379底下没有dbmp.rdb文件

5.通过save触发持久化，将数据写入RDB文件
    127.0.0.1:6379> set age 18
    OK
    127.0.0.1:6379> save
    OK
```

##### 2.AOF持久化

```python
AOF（append-only log file）
记录服务器执行的所有变更操作命令（例如set del等），并在服务器启动时，通过重新执行这些命令来还原数据集
AOF文件中的命令全部以redis协议的格式保存，新命令追加到文件末尾。

优点：最大程度保证数据不丢
缺点：日志记录非常大

# 配置参数
AOF持久化配置，两条参数：

appendonly yes
appendfsync  always    总是修改类的操作
             everysec  每秒做一次持久化
             no        依赖于系统自带的缓存大小机制
```

###### 2.1redis持久化之AOF实战

```python
1.准备AOF配置文件redis.conf
daemonize yes
port 6379
logfile /data/6379/redis.log
dir /data/6379
dbfilename  dbmp.rdb
requirepass redhat
save 900 1
save 300 10
save 60  10000
appendonly yes
appendfsync everysec

2.启动redis服务
	redis-server /etc/redis.conf
	
3.检查redis数据目录/data/6379/是否产生了aof文件
    [root@web02 6379]# ls
    appendonly.aof  dbmp.rdb  redis.log
    
4.登录redis-cli，写入数据，实时检查aof文件信息
	[root@web02 6379]# tail -f appendonly.aof
	
5.设置新key，检查aof信息，然后关闭redis，检查数据是否持久化
    redis-cli -a redhat shutdown
    redis-server /etc/redis.conf
    redis-cli -a redhat
```

##### 3.redis不重启，切换RDB备份到AOF备份

```
1.确保redis版本在2.2以上
[root@pyyuc /data 22:23:30]#redis-server -v
Redis server v=4.0.10 sha=00000000:0 malloc=jemalloc-4.0.3 bits=64 build=64cb6afcf41664c

2.通过config set命令，实现不重启redis服务，从RDB持久化切换为AOF持久化
```

###### 3.1实战

```python
1.redis.conf服务端配置文件
daemonize yes
port 6379
logfile /data/6379/redis.log
dir /data/6379
dbfilename  dbmp.rdb
save 900 1                    #rdb机制 每900秒 有1个修改记录
save 300 10                   #每300秒        10个修改记录
save 60  10000                #每60秒内        10000修改记录

2.启动redis服务端
	redis-server redis.conf
    
3.登录redis客户端，插入数据，手动持久化
    127.0.0.1:6379> set name chaoge
    OK
    127.0.0.1:6379> set age 18
    OK
    127.0.0.1:6379> set addr shahe
    OK
    127.0.0.1:6379> save
    OK
    
4.检查RDB文件
    [root@pyyuc /data 22:34:16]#ls 6379/
    dbmp.rdb  redis.log
    
5.备份这个rdb文件，保证数据安全
	[root@pyyuc /data/6379 22:35:38]#cp dbmp.rdb /opt/
    
6.执行命令，开启AOF持久化
    127.0.0.1:6379> CONFIG set appendonly yes   #开启AOF功能
    OK
    127.0.0.1:6379> CONFIG SET save ""          #关闭RDB功能
    OK
    
7.确保数据库的key数量正确
    127.0.0.1:6379> keys *
    1) "addr"
    2) "age"
    3) "name"
    
8.确保插入新的key，AOF文件会记录
    127.0.0.1:6379> set title golang
    OK
```

### 四、redis安全设置

##### 1.redis安全特征

```
1.redis没有用户概念，redis只有密码
2.redis默认在保护模式下，不允许远程任何用户登录（protected-mode）
```

##### 2.redis安全设置

```python
1.redis.conf设置
    protected-mode yes   #打开保护模式
    port 6380            #更改默认启动端口
    requirepass xxxxxx   #设置redis启动密码，xxxx是自定义的密码

2.启动redis服务端
    redis-server /opt/redis-4.0.10/redis.conf &     #指定配置文件启动redis，且后台启动
    
3.启动redis客户端
[root@oldboy_python ~ 09:48:41]#redis-cli -p 6380
127.0.0.1:6380> auth xxxx
OK

# 补充
1.检查redis是否设置了密码
    127.0.0.1:6380> CONFIG get requirepass
    1) "requirepass"
    2) "xxxxxx"

2.如果没有，也可以给redis设置密码（命令方式）
	CONFIG set requirepass "xxxxxx"
```

### 五、redis主从同步

##### 1.redis主从同步原理

```python
1.Redis集群中的数据库复制是通过主从同步来实现的
2.主节点（Master）把数据分发给从节点（Slave）
3.主从同步的好处是在于高可用，redis节点有冗余设计

# 原理
1. 从服务器向主服务器发送SYNC命令。
2. 接到SYNC命令的主服务器会调用BGSAVE命令，创建一个RDB文件，并使用缓冲区记录接下来执行的所有写命令。
3. 当主服务器执行完BGSAVE命令时，它会向从服务器发送RDB文件，而从服务器则会接收并载入这个文件。
4. 主服务器将缓冲区储存的所有写命令发送给从服务器执行。
-------------------
1、在开启主从复制的时候，使用的是RDB方式来同步主从数据的；
2、同步开始之后，通过主库命令传播的方式，主动的复制方式实现；
3、2.8以后实现PSYNC的机制，实现断线重连；
```

##### 2.redis主从同步实战

###### 2.1环境准备

```python
#环境：
准备两个或两个以上redis实例
mkdir /data/638{0..2}  #创建6380 6381 6382文件夹

#6380.conf
vim  /data/6380/redis.conf
port 6380
daemonize yes
pidfile /data/6380/redis.pid
loglevel notice
logfile "/data/6380/redis.log"
dbfilename dump.rdb
dir /data/6380
protected-mode no

#6381.conf
vim  /data/6381/redis.conf
port 6381
daemonize yes
pidfile /data/6381/redis.pid
loglevel notice
logfile "/data/6381/redis.log"
dbfilename dump.rdb
dir /data/6381
protected-mode no

#6382.conf
vim  /data/6382/redis.conf
port 6382
daemonize yes
pidfile /data/6382/redis.pid
loglevel notice
logfile "/data/6382/redis.log"
dbfilename dump.rdb
dir /data/6382
protected-mode no

# 启动三个redis实例
    redis-server /data/6380/redis.conf
    redis-server /data/6381/redis.conf
    redis-server /data/6382/redis.conf
```

###### 2.2配置主从同步

```python
#6381/6382命令行
    redis-cli -p 6381
    SLAVEOF 127.0.0.1 6380  #指明主的地址

    redis-cli -p 6382
    SLAVEOF 127.0.0.1 6380  #指明主的地址

#检查主从状态
    从库：
    127.0.0.1:6382> info replication
    127.0.0.1:6381> info replication

    主库：
    127.0.0.1:6380> info replication
```

###### 2.3测试，主库写入数据，检查从库数据

```python
#主
127.0.0.1:6380> set name chaoge

#从
127.0.0.1:6381> get name 
```

###### 2.4手动进行主从复制故障切换

```python
#关闭主库6380
    redis-cli -p 6380
    shutdown
#检查从库主从信息，此时master_link_status:down 
    redis-cli -p 6381
    info replication

    redis-cli -p 6382
    info replication
```

###### 2.5重新选择主库

```python
1.关闭6381的从库身份
    redis-cli -p 6381
    info replication
    slaveof no one

2.将6382设为6381的从库
    6382连接到6381：
    [root@db03 ~]# redis-cli -p 6382
    127.0.0.1:6382> SLAVEOF no one
    127.0.0.1:6382> SLAVEOF 127.0.0.1 6381
3.检查6382，6381的主从信息
```

### 六、redis哨兵集群

##### 1.Redis-Sentinel

```python
Redis-Sentinel是redis官方推荐的高可用性解决方案，
当用redis作master-slave的高可用时，如果master本身宕机，redis本身或者客户端都没有实现主从切换的功能。
redis-sentinel就是一个独立运行的进程，用于监控多个master-slave集群，
自动发现master宕机，进行自动切换slave > master。

#sentinel主要功能如下：
- 时时的监控redis是否良好运行，如果节点不可达就会对节点进行下线标识；
- 如果被标识的是主节点，sentinel就会和其他的sentinel节点“协商”，如果其他节点也认为主节点不可达，就会选举一个sentinel节点来完成自动故障转义；
- 在master-slave进行切换后，master_redis.conf、slave_redis.conf和sentinel.conf的内容都会发生改变，即master_redis.conf中会多一行slaveof的配置，sentinel.conf的监控目标会随之调换
```

##### 2.Redis-Sentinel的工作原理

```python
每个Sentinel以每秒钟一次的频率向它所知的Master，Slave以及其他 Sentinel 实例发送一个 PING 命令；
 
如果一个实例（instance）距离最后一次有效回复 PING 命令的时间超过 down-after-milliseconds 选项所指定的值，则这个实例会被 Sentinel 标记为主观下线。

如果一个Master被标记为主观下线，则正在监视这个Master的所有 Sentinel 要以每秒一次的频率确认Master的确进入了主观下线状态。

当有足够数量的 Sentinel（大于等于配置文件指定的值）在指定的时间范围内确认Master的确进入了主观下线状态， 则Master会被标记为客观下线

在一般情况下，每个 Sentinel 会以每 10 秒一次的频率向它已知的所有Master，Slave发送 INFO 命令

当Master被 Sentinel 标记为客观下线时，Sentinel 向下线的 Master 的所有 Slave 发送 INFO 命令的频率会从 10 秒一次改为每秒一次

若没有足够数量的 Sentinel 同意 Master 已经下线， Master 的客观下线状态就会被移除。

若 Master 重新向 Sentinel 的 PING 命令返回有效回复， Master 的主观下线状态就会被移除。

#主观下线和客观下线
主观下线：Subjectively Down，简称 SDOWN，指的是当前 Sentinel 实例对某个redis服务器做出的下线判断。
客观下线：Objectively Down， 简称 ODOWN，指的是多个 Sentinel 实例在对Master Server做出 SDOWN 判断，并且通过 SENTINEL is-master-down-by-addr 命令互相交流之后，得出的Master Server下线判断，然后开启failover.

SDOWN适合于Master和Slave，只要一个 Sentinel 发现Master进入了ODOWN， 这个 Sentinel 就可能会被其他 Sentinel 推选出， 并对下线的主服务器执行自动故障迁移操作。

ODOWN只适用于Master，对于Slave的 Redis 实例，Sentinel 在将它们判断为下线前不需要进行协商， 所以Slave的 Sentinel 永远不会达到ODOWN。
```

##### 3.redis主从复制背景问题

```python
Redis主从复制可将主节点数据同步给从节点，从节点此时有两个作用：
    1.一旦主节点宕机，从节点作为主节点的备份可以随时顶上来。
    2.扩展主节点的读能力，分担主节点读压力。
但是问题是：
	一旦主节点宕机，从节点上位，那么需要人为修改所有应用方的主节点地址（改为新的master地址），还需要命令所有从节点复制新的主节点；
那么这个问题，redis-sentinel就可以解决了
```

##### 4.主从复制结构

![主从复制1](C:\Users\慕唐\Desktop\python\主从复制1.png)

![主从复制2](C:\Users\慕唐\Desktop\python\主从复制2.png)

##### 5.Redis-Sentinel架构

```python
redis的一个进程，但是不存储数据，只是监控redis
```

![Redis-Sentinel0](C:\Users\慕唐\Desktop\python\Redis-Sentinel0.png)

![Redis-Sentinel1](C:\Users\慕唐\Desktop\python\Redis-Sentinel1.png)

![Redis-Sentinel2](C:\Users\慕唐\Desktop\python\Redis-Sentinel2.png)

![Redis-Sentinel3](C:\Users\慕唐\Desktop\python\Redis-Sentinel3.png)

##### 6.redis命令整理

```python
官网地址：http://redisdoc.com/

redis-cli info             #查看redis数据库信息

redis-cli info replication #查看redis的复制授权信息

redis-cli info sentinel    #查看redis的哨兵信息
```

##### 7.安装与配置

###### 7.1所有配置文件如下

![哨兵配置](C:\Users\慕唐\Desktop\python\哨兵配置.png)

###### 7.2主节点master的redis-6379.conf

```python
port 6379
daemonize yes
logfile "6379.log"
dbfilename "dump-6379.rdb"
dir "/var/redis/data/"
```

###### 7.3从节点slave的redis-6380.conf

```python
port 6380
daemonize yes
logfile "6380.log"
dbfilename "dump-6380.rdb"
dir "/var/redis/data/" 
slaveof 127.0.0.1 6379      // 从属主节点
```

###### 7.4从节点slave的redis-6381.conf

```python
port 6381
daemonize yes
logfile "6380.log"
dbfilename "dump-6380.rdb"
dir "/var/redis/data/" 
slaveof 127.0.0.1 6379      // 从属主节点
```

###### 7.5启动redis主节点

```python
redis-server /etc/redis-6379.conf
```

###### 7.6测试redis主节点是否通信

```python
redis-cli  
ping
```

###### 7.7启动两个slave节点的redis服务

```
[root@master 192.168.119.10 ~]$redis-server /etc/redis-6380.conf
[root@master 192.168.119.10 ~]$redis-server /etc/redis-6381.conf
```

###### 7.8验证slave节点的redis服务

```python
[root@master ~]$redis-cli -p 6380 ping
PONG
[root@master ~]$redis-cli -p 6381 ping
PONG
```

###### 7.9确定主从关系

```python
# 在主节点上查看主从通信关系
[root@master ~]# redis-cli  -p 6379 info replication

# 在从节点上查看主从关系
[root@slave 192.168.119.11 ~]$redis-cli  -p 6380 info replication

此时可以在master上写入数据，在slave上查看数据，此时主从复制配置完成；
```

###### 7.10开始配置Redis-Sentinel

```python
#redis-sentinel-26379.conf配置文件写入如下信息：
#Sentinel节点的端口
port 26379  
dir /var/redis/data/
logfile "26379.log"

#当前Sentinel节点监控 192.168.119.10:6379 这个主节点
#2代表判断主节点失败至少需要2个Sentinel节点节点同意
#mymaster是主节点的别名
sentinel monitor mymaster 192.168.119.10 6379 2

#每个Sentinel节点都要定期PING命令来判断Redis数据节点和其余Sentinel节点是否可达，如果超过30000毫秒30s且没有回复，则判定不可达
sentinel down-after-milliseconds mymaster 30000

#当Sentinel节点集合对主节点故障判定达成一致时，Sentinel领导者节点会做故障转移操作，选出新的主节点，
#原来的从节点会向新的主节点发起复制操作，限制每次向新的主节点发起复制操作的从节点个数为1
sentinel parallel-syncs mymaster 1

#故障转移超时时间为180000毫秒
sentinel failover-timeout mymaster 180000

-----------------------------------------------------------------------------------
redis-sentinel-26380.conf和redis-sentinel-26381.conf的配置仅仅差异是port(端口)的不同。
然后启动三个sentinel哨兵
redis-sentinel /etc/redis-sentinel-26379.conf
redis-sentinel /etc/redis-sentinel-26380.conf
redis-sentinel /etc/redis-sentinel-26381.conf
```

```python
#此时查看哨兵是否成功通信
[root@master ~]# redis-cli -p 26379  info sentinel
sentinel_masters:1
sentinel_tilt:0
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
sentinel_simulate_failure_flags:0
master0:name=mymaster,status=ok,address=192.168.119.10:6379,slaves=2,sentinels=3
#看到最后一条信息正确即成功了哨兵，
#哨兵主节点名字叫做mymaster，状态ok，监控地址是192.168.119.10:6379，有两个从节点，3个哨兵
```

###### 7.11redis高可用故障实验

```python
#大致思路：
杀掉主节点的redis进程6379端口，观察从节点是否会进行新的master选举，进行切换
重新恢复旧的“master”节点，查看此时的redis身份
 
#首先查看三个redis的进程状态
ps -ef|grep redis

#检查三个节点的复制身份状态
第一个
[root@master tmp]# redis-cli -p 6381 info replication
# Replication
role:slave
master_host:127.0.0.1
master_port:6380
第二个
[root@master tmp]# redis-cli -p 6380 info replication
# Replication
role:master
connected_slaves:2
slave0:ip=127.0.0.1,port=6381,state=online,offset=54386,lag=0
slave1:ip=127.0.0.1,port=6379,state=online,offset=54253,lag=0
第三个
[root@master tmp]# redis-cli -p 6379 info replication
# Replication
role:slave
master_host:127.0.0.1
master_port:6380

此时，干掉master！！！然后等待其他两个节点是否能自动被哨兵sentienl，切换为master节点

ps -ef|grep 6380   #干掉master进程
 
此时查看两个slave的状态

精髓就是查看一个参数：
master_link_down_since_seconds:13
 
稍等片刻之后，发现slave节点成为master节点！！
[root@master tmp]# redis-cli -p 6379 info replication
# Replication
role:master
connected_slaves:1
slave0:ip=127.0.0.1,port=6381,state=online,offset=41814,lag=1
大功告成！！开心！！！
```

### 七、redis-cluster配置

##### 1.客户端分片

```python
redis3.0集群采用P2P模式，完全去中心化，将redis所有的key分成了16384个槽位，每个redis实例负责一部分slot，集群中的所有信息通过节点数据交换而更新。

#redis实例集群主要思想
	将redis数据的key进行散列，通过hash函数特定的key会映射到指定的redis节点上
```

##### 2.数据分布原理图

![数据分布](C:\Users\慕唐\Desktop\python\数据分布.png)

###### 2.1数据分布理论

```python
分布式数据库首要解决把整个数据集按照分区规则映射到多个节点的问题，即把数据集划分到多个节点上，每个节点负责整个数据的一个子集。

常见的分区规则有哈希分区和顺序分区。Redis Cluster采用哈希分区规则，因此接下来会讨论哈希分区规则。
    -节点取余分区
    -一致性哈希分区
    -虚拟槽分区(redis-cluster采用的方式)
```

###### 2.2顺序分区

![顺序分区](C:\Users\慕唐\Desktop\python\顺序分区.png)

###### 2.3哈希分区

![哈希分区](C:\Users\慕唐\Desktop\python\哈希分区.png)

```python
例如按照节点取余的方式，分三个节点；
1~100的数据对3取余，可以分为三类：
    余数为0
    余数为1
    余数为2
 
那么同样的分4个节点就是hash(key)%4；
节点取余的优点是简单，客户端分片直接是哈希+取余；
```

###### 2.4虚拟槽分区

```python
Redis Cluster采用虚拟槽分区；

虚拟槽分区巧妙地使用了哈希空间，
使用分散度良好的哈希函数把所有的数据映射到一个固定范围内的整数集合，整数定义为槽（slot）。

Redis Cluster槽的范围是0 ～ 16383。
槽是集群内数据管理和迁移的基本单位。
采用大范围的槽的主要目的是为了方便数据的拆分和集群的扩展，每个节点负责一定数量的槽。
```

![虚拟槽分配](C:\Users\慕唐\Desktop\python\虚拟槽分配.png)

##### 3.搭建redis-cluster

```python
搭建集群分为几步：
    - 准备节点（几匹马儿）
    - 节点通信（几匹马儿分配主从）
    - 分配槽位给节点（slot分配给马儿）
    
redis-cluster集群架构
    多个服务端，负责读写，彼此通信，redis指定了16384个槽。
    多匹马儿，负责运输数据，马儿分配16384个槽位，管理数据。
	ruby的脚本自动就把分配槽位这事做了（需要安装ruby）
```

##### 4.安装

```
官方提供通过ruby语言的脚本一键安装
```

###### 4.1环境准备

```python
通过配置，开启redis-cluster

port 7000
daemonize yes
dir "/opt/redis/data"
logfile "7000.log"
dbfilename "dump-7000.rdb"
cluster-enabled yes   #开启集群模式
cluster-config-file nodes-7000.conf　　#集群内部的配置文件
cluster-require-full-coverage no　　#redis cluster需要16384个slot都正常的时候才能对外提供服务，换句话说，只要任何一个slot异常那么整个cluster不对外提供服务。 因此生产环境一般为no

#redis支持多实例的功能，我们在单机演示集群搭建，需要6个实例，三个是主节点，三个是从节点，数量为6个节点才能保证高可用的集群。

每个节点仅仅是端口运行的不同！
[root@yugo /opt/redis/config 17:12:30]#ls
redis-7000.conf  redis-7002.conf  redis-7004.conf
redis-7001.conf  redis-7003.conf  redis-7005.conf

#确保每个配置文件中的端口修改！！
```

###### 4.2运行redis实例

```python
#创建6个节点的redis实例
1855  2018-10-24 15:46:01 redis-server redis-7000.conf
1856  2018-10-24 15:46:13 redis-server redis-7001.conf
1857  2018-10-24 15:46:16 redis-server redis-7002.conf
1858  2018-10-24 15:46:18 redis-server redis-7003.conf
1859  2018-10-24 15:46:20 redis-server redis-7004.conf
1860  2018-10-24 15:46:23 redis-server redis-7005.conf

#检查日志文件
cat 7000.log

#检查redis服务的端口、进程
netstat -tunlp|grep redis
ps -ef|grep redis
```

###### 4.3创建开启redis-cluster

```python
# 安装ruby流程
    1.下载、编译、安装Ruby
    2.安装rubygem redis
    3.安装redis-trib.rb命令
    

#下载ruby
wget https://cache.ruby-lang.org/pub/ruby/2.3/ruby-2.3.1.tar.gz

#安装ruby
tar -xvf ruby-2.3.1.tar.gz
./configure --prefix=/opt/ruby/
make && make install

#准备一个ruby命令
#准备一个gem软件包管理命令
#拷贝ruby命令到path下/usr/local/ruby
cp /opt/ruby/bin/ruby /usr/local/
cp bin/gem /usr/local/bin

#安装ruby gem包管理工具
wget http://rubygems.org/downloads/redis-3.3.0.gem
gem install -l redis-3.3.0.gem

#查看gem有哪些包
gem list -- check redis gem

#安装redis-trib.rb命令
[root@yugo /opt/redis/src 18:38:13]#cp /opt/redis/src/redis-trib.rb /usr/local/bin/
```

###### 4.4一键开启redis-cluster集群

```python
#每个主节点，有一个从节点，代表--replicas 1
redis-trib.rb create --replicas 1 127.0.0.1:7000 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005

#集群自动分配主从关系：7000、7001、7002为主 7003、7004、7005为从 
```

##### 5.查看集群状态

```python
redis-cli -p 7000 cluster info  
redis-cli -p 7000 cluster nodes  #等同于查看nodes-7000.conf文件节点信息

#集群主节点状态
redis-cli -p 7000 cluster nodes | grep master
#集群从节点状态
redis-cli -p 7000 cluster nodes | grep slave
```

```python
# 测试写入集群数据，登录集群必须使用redis-cli -c -p 7000必须加上-c参数

[root@yugo /opt/redis/src 18:46:07]#redis-cli -c -p 7000
127.0.0.1:7000> ping
PONG
127.0.0.1:7000> keys *
(empty list or set)
127.0.0.1:7000> get name
-> Redirected to slot [5798] located at 127.0.0.1:7001
"chao"

#工作原理：
	redis客户端任意访问一个redis实例，如果数据不在该实例中，通过重定向引导客户端访问所需要的redis实例
```

### 八、python操作redis









