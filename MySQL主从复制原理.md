# MySQL主从复制原理

### 一、背景

```
关于mysql主从同步，相信大家都不陌生，随着系统应用访问量逐渐增大，单台数据库读写访问压力也随之增大，当读写访问达到一定瓶颈时，将数据库的读写效率骤然下降，甚至不可用；为了解决此类问题，通常会采用mysql集群，当主库宕机后，集群会自动将一个从库升级为主库，继续对外提供服务；那么主库和从库之间的数据是如何同步的呢？

为了减轻主库的压力，应该在系统应用层面做读写分离，写操作走主库，读操作走从库，下图为MySQL官网给出的主从复制的原理图，从图中可以简单的了解读写分离及主从同步的过程，分散了数据库的访问压力，提升整个系统的性能和可用性，降低了大访问量引发数据库宕机的故障率。
```

![=MySQL主从同步原理图](C:\Users\慕唐\Desktop\python\=MySQL主从同步原理图.png)

### 二、binlog简介

```python
MySQL主从同步是基于从库对主库binlog文件的增量订阅来实现，为了更好的理解主从同步过程，这里简单介绍一下binlog日志文件。binlog日志用于记录所有更新了数据或者已经潜在更新了数据（例如，没有匹配任何行的一个DELETE）的所有语句。语句以“事件”的形式保存，它描述数据更改，它是以二进制的形式保存在磁盘中。我们可以通过mysql提供的查看工具mysqlbinlog查看文件中的内容，例如 mysqlbinlog mysql-bin.00001 | more，这里注意一下binlog文件的后缀名00001，binlog文件大小和个数会不断的增加，当MySQL停止或重启时，会产生一个新的binlog文件，后缀名会按序号递增，例如mysql-bin.00002、mysql-bin.00003，并且当binlog文件大小超过 max_binlog_size系统变量配置时也会产生新的binlog文件。
```

##### 1.binlog日志格式

```python
我们来看一下binlog日志文件都支持哪些格式：

1.statement：记录每一条更改数据的sql

优点：binlog文件较小，节约I/O，性能较高。
缺点：不是所有的数据更改都会写入binlog文件中，尤其是使用MySQL中的一些特殊函数（如LOAD_FILE()、UUID()等）和一些不确定的语句操作，从而导致主从数据无法复制的问题。

2.row：不记录sql，只记录每行数据的更改细节

优点：详细的记录了每一行数据的更改细节，这也意味着不会由于使用一些特殊函数或其他情况导致不能复制的问题。
缺点：由于row格式记录了每一行数据的更改细节，会产生大量的binlog日志内容，性能不佳，并且会增大主从同步延迟出现的几率。

3.mixed：一般的语句修改使用statment格式保存binlog，如一些函数，statement无法完成主从复制的操作，则采用row格式保存binlog，MySQL会根据执行的每一条具体的sql语句来区分对待记录的日志形式，也就是在Statement和Row之间选择一种。
```

##### 2.binlog日志内容

```python
mysqlbinlog命令查看的内容如下：
```

![11111](C:\Users\慕唐\Desktop\python\11111.png)

```python
根据事件类型查看的binlog内容：
```

![22222](C:\Users\慕唐\Desktop\python\22222.png)

##### 3.binlog事件类型

```python
binlog记录的所有操作实际上都有对应的事件类型的，譬如STATEMENT格式中的DML操作对应的是QUERY_EVENT类型，ROW格式下的DML操作对应的是ROWS_EVENT类型，如果想了解更多请参考官方文档；
```

### 三、主从复制原理

```python
#同步复制: 
	指的是客户端连接到MySQL主服务器写入一段数据, MySQL主服务器同步给MySQL从服务器需要等待从服务器发出同步完成的响应才返回客户端OK, 这其中等待同步的过程是阻塞的, 如果有N台从服务器, 效率极低 

#异步复制: 
	指的是客户端连接到MySQL主服务器写入一段数据, MySQL主服务器将写入的数据发送给MySQL从服务器, 然后直接返回客户端OK, 可能从服务器的数据会和主服务不一致 

#半同步复制:
	指的是客户端连接到MySQL主服务器写入一段数据, MySQL主服务器只将数据同步复制给其中一台从服务器, 半同步复制给其他的从服务器, 来达到其中一台从服务器完全同步的效果
```



```python
mysql主从复制需要三个线程，master（binlog dump thread）、slave（I/O thread 、SQL thread）。

#master
1.binlog dump线程：当主库中有数据更新时，那么主库就会根据按照设置的binlog格式，将此次更新的事件类型写入到主库的binlog文件中，此时主库会创建log dump线程通知slave有数据更新，当I/O线程请求日志内容时，会将此时的binlog名称和当前更新的位置同时传给slave的I/O线程。

#slave
2.I/O线程：该线程会连接到master，向log dump线程请求一份指定binlog文件位置的副本，并将请求回来的binlog存到本地的relay log中，relay log和binlog日志一样也是记录了数据更新的事件，它也是按照递增后缀名的方式，产生多个relay log（ host_name-relay-bin.000001）文件，slave会使用一个index文件（ host_name-relay-bin.index）来追踪当前正在使用的relay log文件。

3.SQL线程：该线程检测到relay log有更新后，会读取并在本地做read操作，将发生在主库的事件在本地重新执行一遍，来保证主从数据同步。此外，如果一个relay log文件中的全部事件都执行完毕，那么SQL线程会自动将该relay log 文件删除掉。

#在master、slave上查看上述的线程：
	SHOW PROCESSLIST;
```

```
下面是整个复制过程的原理图：
```

![MySQL主从](C:\Users\慕唐\Desktop\python\MySQL主从.png)

### 四、主动同步延迟问题

##### 1.延迟问题的阐述

```python
mysql的主从复制都是单线程的操作，主库对所有DDL和DML产生binlog，binlog是顺序写，所以效率很高；

slave的I/O线程到主库取日志，效率也比较高；

但是，slave的SQL线程将主库的DDL和DML操作在slave实施，DML和DDL的IO操作是随即的，不是顺序的，成本高很多，还可能存在slave上的其他查询产生lock争用的情况，由于SQL也是单线程的，所以一个DDL卡住了，需要执行很长一段事件，后续的DDL线程会等待这个DDL执行完毕之后才执行，这就导致了延时。

当主库的TPS并发较高时，产生的DDL数量超过slave一个sql线程所能承受的范围，延时就产生了；

除此之外，还有可能与slave的大型query语句产生了锁等待导致。
```

##### 2.几种常见的解决方案

```python
由于主从同步延迟是客观存在的，我们只能从我们自己的架构上进行设计，尽量让主库的DDL快速执行。
下面列出几种常见的解决方案：

1. 业务的持久化层的实现采用分库架构，mysql服务可平行扩展，分散压力。

2. 服务的基础架构在业务和mysql之间加入memcache或者Redis的cache层。降低mysql的读压力；

3. 使用比主库更好的硬件设备作为slave；

4. sync_binlog在slave端设置为0；

5. –logs-slave-updates 从服务器从主服务器接收到的更新不记入它的二进制日志。

6. 禁用slave的binlog
```

### 五、自己总结

```python
#描述mysql主从复制原理
	从库的io线程会实时依据master.info信息的去主库的binlog日志里面读取更新的内容，将更新的内容取回到自己的中继日志中，同时会更新master.info信息，此时sql线程实时会从中继日志中读取并执行里面的sql语句。

#描述mysql主从同步部署
1.将主库的数据备份，备份的时候时候master-data=1指定。然后在从库上将备份数据导入

2.在主库上给开启主库的bin-log功能，以及service-id

mysql> GRANT REPLICATION SLAVE ON *.* TO 'slave_account'@'10.121.0.220' identified by '123456'

这句的意思是，允许从服务器'10.121.0.220 '使用'slave-account'及'123456'这个帐号密码对对主服务器的所有数据库（*.*)进行主从复制（'REPLICATION SLAVE').

3.在主库上执行flush  tables  with  relay  locks； 读写锁。然另开启一个窗口

4.在从库上执行change master里面指定刚才创建的用户以及密码

5.执行start slave；

6.show slave status；看看两个线程是否启动

7.在主库上解锁。
```

### 六、MySQL主从同步实战

##### 1.工具

```python
系统:ubuntu16.04
mysql:5.7.17
#2台机器：
master IP:192.168.33.22
slave  IP:192.168.33.33
```

##### 2.master机器上操作

###### 2.1 更改配置文件

```python
找到配置文件 /etc/mysql/mysql.conf.d/mysqld.cnf
配置如下：
bind-address = 192.168.33.22            #your master 
server-id = 1                           #在master-slave架构中，每台机器节点都需要有唯一的
log_bin = /var/log/mysql/mysql-bin.log  #开启binlog
```

###### 2.2 重启MySQL，使配置文件生效

```python
sudo systemctl restart mysql
```

###### 2.3 创建主从同步的mysql user

```python
mysql -u root -p
Password:

#创建slave1用户，并指定该用户只能在主机192.168.33.33上登录。
mysql> CREATE USER 'slave1'@'192.168.33.33' IDENTIFIED BY 'slavepass';

#为slave1赋予REPLICATION SLAVE权限。
mysql> GRANT REPLICATION SLAVE ON *.* TO 'slave1'@'192.168.33.33';
```

###### 2.4 为MySQL加读锁

```python
# 为了主库与从库的数据保持一致，我们先为MySQL加读锁，使其变为只读；
mysql> FLUSH TABLES WITH READ LOCK;
```

###### 2.5 记录下MASTER REPLICAATION LOG的位置

```sql
mysql> SHOW MASTER STATUS;
```

###### 2.6 将master DB中现有的数据信息导出

```python
mysqldump -u root -p --all-databases --master-data > dbdump.sql
```

###### 2.7 解除master DB的读锁

```python
mysql> UNLOCK TABLES;
```

###### 2.8 将步骤6中的dbdump.sql文件copy到slave

```python
scp dbdump.sql ubuntu@192.168.33.33:/home/ubuntu
```

##### 3.slave机器上操作

###### 3.1 更改配置文件

```python
找到配置文件 /etc/mysql/mysql.conf.d/mysqld.cnf

更改配置如下：
bind-address = 192.168.33.33                   #your slave 
server-id = 2                                  #master-slave结构中，唯一的
log_bin = /var/log/mysql/mysql-bin.log         #开启binlog
```

###### 3.2 重启MySQL，使配置文件生效

```python
sudo systemctl restart mysql;
```

###### 3.3 导入从master DB中导出的dbdump.sql文件，使得master-slave数据一致

```python
mysql -u root -p < /home/ubuntu/dbdump.sql;
```

###### 3.4 使slave与master建立连接，从而同步

```python
mysql -u root -p

Password: 

mysql> STOP SLAVE;

mysql> CHANGE MASTER TO 
						-> MASTER_HOST='192.168.33.22', 
					    -> MASTER_USER='slave1', 
    			        -> MASTER_PASSWORD='slavepass', 
                        -> MASTER_LOG_FILE='mysql-bin.000001', 
                        -> MASTER_LOG_POS=613;

mysql> START SLAVE;

# 注解
MASTER_LOG_FILE='mysql-bin.000001'与MASTER_LOG_POS=613的值，是从上面的 SHOW MASTER STATUS 得到的。

经过如此设置之后，就可以进行master-slave同步了~
```











































