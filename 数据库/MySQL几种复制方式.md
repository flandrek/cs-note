# MySQL几种复制方式

## 主从复制

MySQL主从复制包括异步模式、半同步模式、GTID模式以及多源复制模式，默认是异步模式 (如之前详细介绍的[mysql主从复制](https://www.cnblogs.com/kevingrace/p/6256603.html))。

所谓异步模式指的是 MySQL 主服务器上 I/O thread 线程将二进制日志写入 binlog 文件之后就返回客户端结果，不会考虑二进制日志是否完整传输到从服务器以及是否完整存放到从服务器上的relay日志中，这种模式一旦主服务(器)宕机，数据就可能会发生丢失。

异步模式是一种**基于偏移量**的主从复制

#### **实现原理**

主库开启 binlog 功能并授权从库连接主库，从库通过 change master 得到主库的相关同步信息然后连接主库进行验证，主库 IO 线程根据从库 slave 线程的请求，从 master.info 开始记录的位置点向下开始取信息，同时把取到的位置点和最新的位置与 binlog 信息一同发给从库 IO 线程，从库将相关的 sql 语句存放在 relay-log 里面，最终从库的 sql 线程将 relay-log 里的 sql 语句应用到从库上，至此整个同步过程完成，之后将是无限重复上述过程。

#### mysql开启主从复制的步骤 

1. 在主库与从库都安装mysql数据库;
2. 在主库的 my.cnf 配置文件中配置 server-id 和 log-bin；
3. 在登陆主库后创建认证用户并做授权; 
4. 在从库的 my.cnf 配置文件中配置 server-id;
5. 登陆从库后，指定 master 并开启同步开关。

需要注意的是server-id主从库的配置是不一样的。

**server-id存在作用:** mysql同步的数据中是包含server-id的，而server-id用于标识该语句最初是从哪个server写入的。因此server-id一定要有的

**server-id不能相同的原因：**每一个同步中的slave在master上都对应一个master线程，该线程就是通过slave的server-id来标识的；每个slave在master端最多有一个master线程，如果两个slave的server-id 相同，则后一个连接成功时，slave主动连接master之后，如果slave上面执行了slave stop；则连接断开，但是master上对应的线程并没有退出；当slave start之后，master不能再创建一个线程而保留原来的线程，那样同步就可能有问题；

在mysql做主主同步时，多个主需要构成一个环状，但是同步的时候又要保证一条数据不会陷入死循环，这里就是靠server-id来实现的；

#### 主从复制过程                                                                                                                                                                                               

MySQL的复制（replication）是一个**异步**的复制，从一个MySQL instace（称之为Master）复制到另一个MySQL instance（称之Slave）。实现整个复制操作主要由三个进程完成的，其中两个进程在Slave（Sql进程和IO进程），另外一个进程在Master（IO进程）上。简单来说，Mysql复制就是一种基于binlog的单线程异步复制过程！！

MySQL Replication复制的基本过程如下：

1. Slave上面的IO进程连接上Master，并请求从指定日志文件的指定位置（或者从最开始的日志）之后的日志内容；

```
`mysql> CHANGE MASTER TO``    ``-> MASTER_HOST=``'master_host_name'``,``    ``-> MASTER_USER=``'replication_user_name'``,``    ``-> MASTER_PASSWORD=``'replication_password'``,``    ``-> MASTER_LOG_FILE=``'recorded_log_file_name'``,``    ``-> MASTER_LOG_POS=recorded_log_position;``    ``-> MASTER_CONNECT_RETRY=10;                           ``#连接主库失败时，每隔10s钟就重新连一下! 一般省略这一行配置，不用配置这一行。`
```

2. Master接收到来自Slave的IO进程的请求后，通过负责复制的IO进程根据请求信息读取制定日志指定位置之后的日志信息，返回给Slave的IO进程。返回信息中除了日志所包含的信息之外，还包括本次返回的信息已经到Master端的bin-log文件的名称以及bin-log的位置；

3. Slave的IO进程接收到信息后，将接收到的日志内容依次添加到Slave端的relay-log文件的最末端，并将读取到的Master端的bin-log的文件名和位置记录到master-info文件中，以便在下一次读取的时候能够清楚的高速Maste "我需要从某个bin-log的哪个位置开始往后的日志内容，请发给我"!

4. Slave的Sql进程检测到relay-log中新增加了内容后，会马上解析relay-log的内容成为在Master端真实执行时候的那些可执行的内容，并在自身执行。                                                                                                                                                                          



## 半同步复制

### 几种复制方式的区别

**异步复制（Asynchronous replication）**
MySQL默认的复制即是异步的，主库在执行完客户端提交的事务后会立即将结果返给给客户端，并不关心从库是否已经接收并处理，这样就会有一个问题，主如果crash掉了，此时主上已经提交的事务可能并没有传到从上，如果此时，强行将从提升为主，可能导致新主上的数据不完整。

**全同步复制（Fully synchronous replication）**
指当主库执行完一个事务，所有的从库都执行了该事务才返回给客户端。因为需要等待所有从库执行完该事务才能返回，所以全同步复制的性能必然会收到严重的影响。

**半同步复制（Semisynchronous replication）**
介于异步复制和全同步复制之间，主库在执行完客户端提交的事务后不是立刻返回给客户端，而是等待至少一个从库接收到并写到 relay log 中才返回给客户端。相对于异步复制，半同步复制提高了数据的安全性，同时它也造成了一定程度的延迟，这个延迟最少是一个TCP/IP往返的时间。所以，**半同步复制最好在低延时的网络中使用**。

![img](E:\markdown笔记\图片\907596-20190107180749552-36551725.gif)



-  **对于异步复制**，主库将事务Binlog事件写入到Binlog文件中，此时主库只会通知一下Dump线程发送这些新的Binlog，然后主库就会继续处理提交操作，而此时不会保证这些Binlog传到任何一个从库节点上。

- **对于全同步复制**，当主库提交事务之后，所有的从库节点必须收到，APPLY并且提交这些事务，然后主库线程才能继续做后续操作。这里面有一个很明显的缺点就是，主库完成一个事务的时间被拉长，性能降低。

- **对于半同步复制**，是介于全同步复制和异步复制之间的一种，主库只需要等待至少一个从库节点收到并且Flush Binlog到Relay Log文件即可，主库不需要等待所有从库给主库反馈。同时，这里只是一个收到的反馈，而不是已经完全执行并且提交的反馈，这样就节省了很多时间。

### **Mysql半同步复制技术**                                                                

一般而言，普通的replication，即MySQL的异步复制，依靠MySQL二进制日志也即binary log进行数据复制。比如两台机器，一台主机（master），另外一台是从机（slave）。
**正常的复制为**：事务一（t1）写入binlog buffer；dumper线程通知slave有新的事务t1；binlog buffer进行checkpoint；slave的io线程接收到t1并写入到自己的的relay log；slave的sql线程写入到本地数据库。 这时，master和slave都能看到这条新的事务，即使master挂了，slave可以提升为新的master。
**异常的复制为**：事务一（t1）写入binlog buffer；dumper线程通知slave有新的事务t1；binlog buffer进行checkpoint；slave因为网络不稳定，一直没有收到t1；master挂掉，slave提升为新的master，t1丢失。
**很大的问题是**：主机和从机事务更新的不同步，就算是没有网络或者其他系统的异常，当业务并发上来时，slave因为要顺序执行master批量事务，导致很大的延迟。

为了弥补以上几种场景的不足，MySQL从5.5 开始推出了半同步复制。相比异步复制，半同步复制提高了数据完整性，因为很明确知道，在一个事务提交成功之后，这个事务就**至少会存在于两个地方**。即在 master 的 dumper 线程通知 slave 后，增加了一个 ack（消息确认），即是否成功收到 t1 的标志码，也就是 dumper 线程除了发送t1 到 slave ，还承担了接收 slave 的 ack 工作。如果出现异常，没有收到 ack，那么将自动降级为普通的复制，直到异常修复后又会自动变为半同步复制。 

#### 半同步复制具体特性

-  从库会在连接到主库时告诉主库，它是不是配置了半同步。
-  如果半同步复制在主库端是开启了的，并且至少有一个半同步复制的从库节点，那么此时主库的事务线程在提交时会被阻塞并等待，结果有两种可能，要么至少一个从库节点通知它已经收到了所有这个事务的 Binlog 事件，要么一直等待直到超过配置的某一个时间点为止，而此时，半同步复制将自动关闭，转换为异步复制。
-  从库节点只有在接收到某一个事务的所有Binlog，将其写入并 Flush 到 Relay Log 文件之后，才会通知对应主库上面的等待线程。
-  如果在等待过程中，等待时间已经超过了配置的超时时间，没有任何一个从节点通知当前事务，那么此时主库会自动转换为异步复制，当至少一个半同步从节点赶上来时，主库便会自动转换为半同步方式的复制。
-  半同步复制必须是在主库和从库两端都开启时才行，如果在主库上没打开，或者在主库上开启了而在从库上没有开启，主库都会使用异步方式复制。

下面来看看半同步复制的原理图：

![img](E:\markdown笔记\图片\907596-20190106123814456-577474442.jpg)

半同步复制的意思表示 MASTER 只需要接收到其中一台 SLAVE 的返回信息，就会 commit；否则需等待直至达到超时时间然后切换成异步再提交。这个做可以使主从库的数据的延迟较小，可以在损失很小的性能的前提下提高数据的安全性。

主库产生 binlog 到主库的 binlog file，传到从库中继日志，然后从库应用；也就是说传输是异步的，应用也是异步的。半同步复制指的是传输同步，应用还是异步的！
**好处：**保证数据不丢失（本机和远端都有binlog）
**坏处：**不能保证应用的同步。

mysql半同步复制模式的流程图

![img](E:\markdown笔记\图片\907596-20190107002522997-1755785661.png)

即主库忽然崩了时，从库虽然说有延迟，但是延迟过后，可以把从库提升为主库继续服务，事后恢复到主库即可

mysql异步复制模式的流程图

**![img](E:\markdown笔记\图片\907596-20190107002617968-1212781875.png)**



#### **半同步复制的潜在问题**

客户端事务在存储引擎层提交后，在得到从库确认的过程中，主库宕机了，此时，可能的情况有两种
\-  **事务还没发送到从库上**
此时，客户端会收到事务提交失败的信息，客户端会重新提交该事务到新的主上，当宕机的主库重新启动后，以从库的身份重新加入到该主从结构中，会发现，该事务在从库中被提交了两次，一次是之前作为主的时候，一次是被新主同步过来的。
\-  **事务已经发送到从库上**
此时，从库已经收到并应用了该事务，但是客户端仍然会收到事务提交失败的信息，重新提交该事务到新的主上。

#### **无数据丢失的半同步复制**

针对上述潜在问题，MySQL 5.7引入了一种新的半同步方案：Loss-Less 半同步复制。针对上面这个图，"Waiting Slave dump" 被调整到 "Storage Commit" 之前。当然，之前的半同步方案同样支持，MySQL 5.7.2引入了一个新的参数进行控制: rpl_semi_sync_master_wait_point, 这个参数有两种取值：

1) AFTER_SYNC , 这个是新的半同步方案，Waiting Slave dump 在 Storage Commit 之前。

2) AFTER_COMMIT, 这个是老的半同步方案。

**来看下面半同步复制原理图，分析下半同步复制潜在问题**

**![img](E:\markdown笔记\图片\907596-20190107181333872-1479031548.jpg)**

master将每个事务写入binlog（sync_binlog=1），传递到slave刷新到磁盘(sync_relay=1)，同时主库提交事务（commit）。master等待slave反馈收到relay log，只有收到ACK后master才将commit OK结果反馈给客户端。

在MySQL 5.5-5.6使用after_commit的模式下，客户端事务在存储引擎层提交后，在得到从库确认的过程中，主库宕机了。此时，即主库在等待Slave ACK的时候，虽然没有返回当前客户端，但事务已经提交，其他客户端会读取到已提交事务。如果Slave端还没有读到该事务的events，同时主库发生了crash，然后切换到备库。那么之前读到的事务就不见了，出现了幻读。

![img](E:\markdown笔记\图片\907596-20190107181414963-464559410.jpg)

如果主库永远启动不了，那么实际上在主库已经成功提交的事务，在从库上是找不到的，也就是数据丢失了，这是MySQL不愿意看到的。所以在MySQL 5.7版本中增加了after_sync（无损复制）参数，并将其设置为默认半同步方式，解决了数据丢失的问题。

#### **半同步复制的安装部署**

要想使用半同步复制，必须满足以下几个条件
1）MySQL 5.5及以上版本
2）变量have_dynamic_loading为YES （查看命令：show variables like "have_dynamic_loading";）
3）主从复制已经存在 (即提前部署mysql主从复制环境,主从同步要配置基于整个数据库的，不要配置基于某个库的同步，即同步时不要过滤库)

**-  首先加载插件**
因用户需执行INSTALL PLUGIN, SET GLOBAL, STOP SLAVE和START SLAVE操作，所以用户需有SUPER权限。

半同步复制是一个功能模块，库要能支持动态加载才能实现半同步复制! (安装的模块存放路径为/usr/local/mysql/lib/plugin）
主数据库执行:

`mysql> INSTALL PLUGIN rpl_semi_sync_master SONAME ``'semisync_master.so'``;` `[要保证``/usr/local/mysql/lib/plugin/``目录下有semisync_master.so文件 (默认编译安装后就有)]``---------------------------------------------------------------------------------------``如果要卸载(前提是要关闭半同步复制功能)，就执行``mysql> UNINSTALL PLUGIN rpl_semi_sync_master;`

从数据库执行:

`mysql> INSTALL PLUGIN rpl_semi_sync_slave SONAME ``'semisync_slave.so'``;` `[要保证``/usr/local/mysql/lib/plugin/``目录下有semisync_slave.so文件 (默认编译安装后就有)]``---------------------------------------------------------------------------------------``如果要卸载(前提是要关闭半同步复制功能)，就执行``mysql> UNINSTALL PLUGIN rpl_semi_sync_slave;`

**-  查看插件是否加载成功的两种方式：**
1) 方式一

`mysql> show plugins;``........``| rpl_semi_sync_master       | ACTIVE   | REPLICATION        | semisync_master.so | GPL     |`

2) 方式二

`mysql> SELECT PLUGIN_NAME, PLUGIN_STATUS FROM INFORMATION_SCHEMA.PLUGINS  WHERE PLUGIN_NAME LIKE ``'%semi%'``;``+----------------------+---------------+``| PLUGIN_NAME          | PLUGIN_STATUS |``+----------------------+---------------+``| rpl_semi_sync_master | ACTIVE        |``+----------------------+---------------+``1 row ``in` `set` `(0.00 sec)`

**-  启动半同步复制**
在安装完插件后，半同步复制默认是关闭的，这时需设置参数来开启半同步
主数据库执行:

`mysql> SET GLOBAL rpl_semi_sync_master_enabled = 1;`

从数据库执行:

`mysql> SET GLOBAL rpl_semi_sync_slave_enabled = 1;`

以上的启动方式是在登录mysql后的命令行操作，也可写在my.cnf配置文件中（推荐这种启动方式）。
主数据库的my.cnf配置文件中添加:

`plugin-load=rpl_semi_sync_master=semisync_master.so``rpl_semi_sync_master_enabled=1`

从数据库的my.cnf配置文件中添加:

`plugin-load=rpl_semi_sync_slave=semisync_slave.so``rpl_semi_sync_slave_enabled=1`

在个别高可用架构下，master和slave需同时启动，以便在切换后能继续使用半同步复制！即在主从数据库的my.cnf配置文件中都要添加:

`plugin-load = ``"rpl_semi_sync_master=semisync_master.so;rpl_semi_sync_slave=semisync_slave.so"``rpl-semi-``sync``-master-enabled = 1``rpl-semi-``sync``-slave-enabled = 1`

**-  重启从数据库上的IO线程**

`mysql> STOP SLAVE IO_THREAD;``mysql> START SLAVE IO_THREAD;`

**特别注意:** 如果没有重启，则默认的还是异步复制模式！，重启后，slave会在master上注册为半同步复制的slave角色。这时候，主的error.log中会打印如下信息:

`2019-01-05T10:03:40.104327Z 5 [Note] While initializing dump thread ``for` `slave with UUID <ce9aaf22-5af6-11e6-850b-000c2988bad2>, found a zombie dump thread with the same UUID. Master is killing the zombie dump thread(4).``2019-01-05T10:03:40.111175Z 4 [Note] Stop asynchronous binlog_dump to slave (server_id: 2)``2019-01-05T10:03:40.119037Z 5 [Note] Start binlog_dump to master_thread_id(5) slave_server(2), pos(mysql-bin.000003, 621)``2019-01-05T10:03:40.119099Z 5 [Note] Start semi-``sync` `binlog_dump to slave (server_id: 2), pos(mysql-bin.000003, 621)`

**-  查看半同步是否在运行**
主数据库:

`mysql> show status like ``'Rpl_semi_sync_master_status'``;``+-----------------------------+-------+``| Variable_name               | Value |``+-----------------------------+-------+``| Rpl_semi_sync_master_status | ON    |``+-----------------------------+-------+``1 row ``in` `set` `(0.00 sec)`

从数据库:

`mysql> show status like ``'Rpl_semi_sync_slave_status'``;``+----------------------------+-------+``| Variable_name              | Value |``+----------------------------+-------+``| Rpl_semi_sync_slave_status | ON    |``+----------------------------+-------+``1 row ``in` `set` `(0.20 sec)`

这两个变量常用来监控主从是否运行在半同步复制模式下。至此，MySQL半同步复制环境就部署完成了！

​                                                                                                                                                                      

需要注意下，其实Mysql半同步复制并不是严格意义上的半同步复制。当半同步复制发生超时时（由rpl_semi_sync_master_timeout参数控制，单位是毫秒，默认为10000，即10s），会暂时关闭半同步复制，转而使用异步复制。当master dump线程发送完一个事务的所有事件之后，如果在rpl_semi_sync_master_timeout内，收到了从库的响应，则主从又重新恢复为半同步复制。**[一旦有一次超时自动降级为异步].**

`mysql> show variables like ``"rpl_semi_sync_master_timeout"``;``+------------------------------+-------+``| Variable_name                | Value |``+------------------------------+-------+``| rpl_semi_sync_master_timeout | 10000 |``+------------------------------+-------+``1 row ``in` `set` `(0.01 sec)`

 接下来可以测试下:

1) 主数据库 （从数据库在执行"stop slave"之前）

`mysql> create database bobo;``Query OK, 1 row affected (0.05 sec)` `mysql> create table bobo.ceshi(``id` `int);``Query OK, 0 row affected (0.28 sec)` `mysql> insert into bobo.ceshi values(1);``Query OK, 1 row affected (0.09 sec)`

2) 从数据执行"stop slave"

`mysql> stop slave;`

再观察主数据库

`mysql> insert into bobo.ceshi values(2);``Query OK, 1 row affected (10.01 sec)` `mysql> show status like ``'Rpl_semi_sync_master_status'``;``+-----------------------------+-------+``| Variable_name               | Value |``+-----------------------------+-------+``| Rpl_semi_sync_master_status | OFF   |``+-----------------------------+-------+``1 row ``in` `set` `(0.00 sec)`

查看从数据库

`mysql> show status like ``'Rpl_semi_sync_slave_status'``;``+-----------------------------+-------+``| Variable_name               | Value |``+-----------------------------+-------+``| Rpl_semi_sync_slave_status  | OFF   |``+-----------------------------+-------+``1 row ``in` `set` `(0.01 sec)`

3) 接着再在从数据库执行"start slave"

`mysql> start slave;`

再观察主数据

`mysql> insert into bobo.ceshi values(3);``Query OK, 1 row affected (0.00 sec)` `mysql> show status like ``'Rpl_semi_sync_master_status'``;``+-----------------------------+-------+``| Variable_name               | Value |``+-----------------------------+-------+``| Rpl_semi_sync_master_status | ON    |``+-----------------------------+-------+``1 row ``in` `set` `(0.00 sec)`

查看从数据库

`mysql> show status like ``'Rpl_semi_sync_slave_status'``;``+-----------------------------+-------+``| Variable_name               | Value |``+-----------------------------+-------+``| Rpl_semi_sync_slave_status  | ON   |``+-----------------------------+-------+``1 row ``in` `set` `(0.00 sec)`

**以上验证分为三个阶段:**
1) 在Slave执行stop slave之前，主的insert操作很快就能返回。
2) 在Slave执行stop slave后，主的insert操作需要10.01s才返回，而这与rpl_semi_sync_master_timeout参数的时间相吻合。这时，查看两个状态的值，均为“OFF”了。同时，主的error.log中打印如下信息:

`2019-01-05T11:51:49.855452Z 6 [Warning] Timeout waiting ``for` `reply of binlog (``file``: mysql-bin.000003, pos: 1447), semi-``sync` `up to ``file` `mysql-bin.000003, position 1196.``2019-01-05T11:51:49.855742Z 6 [Note] Semi-``sync` `replication switched OFF.`

3) 在Slave执行start slave后，主的insert操作很快就能返回，此时，两个状态的值也变为“ON”了。同时，主的error.log中会打印如下信息:

`2019-01-05T11:52:40.477098Z 7 [Note] Start binlog_dump to master_thread_id(7) slave_server(2), pos(mysql-bin.000003, 1196)``2019-01-05T11:52:40.477168Z 7 [Note] Start semi-``sync` `binlog_dump to slave (server_id: 2), pos(mysql-bin.000003, 1196)``2019-01-05T11:52:40.523475Z 0 [Note] Semi-``sync` `replication switched ON at (mysql-bin.000003, 1447)`

**其他变量说明**                                                  

**环境变量（show variables like '%Rpl%';）**

`mysql> show variables like ``'%Rpl%'``;``+-------------------------------------------+------------+``| Variable_name                             | Value      |``+-------------------------------------------+------------+``| rpl_semi_sync_master_enabled              | ON         |``| rpl_semi_sync_master_timeout              | 10000      |``| rpl_semi_sync_master_trace_level          | 32         |``| rpl_semi_sync_master_wait_for_slave_count | 1          |``| rpl_semi_sync_master_wait_no_slave        | ON         |``| rpl_semi_sync_master_wait_point           | AFTER_SYNC |``| rpl_stop_slave_timeout                    | 31536000   |``+-------------------------------------------+------------+``7 rows ``in` `set` `(0.30 sec)`

rpl_semi_sync_master_wait_for_slave_count
MySQL 5.7.3引入的，该变量设置主需要等待多少个slave应答，才能返回给客户端，默认为1。

rpl_semi_sync_master_wait_no_slave
ON
默认值，当状态变量Rpl_semi_sync_master_clients中的值小于rpl_semi_sync_master_wait_for_slave_count时，Rpl_semi_sync_master_status依旧显示为ON。

OFF
当状态变量Rpl_semi_sync_master_clients中的值于rpl_semi_sync_master_wait_for_slave_count时，Rpl_semi_sync_master_status立即显示为OFF，即异步复制。

简单来说，如果mysql架构是1主2从，2个从都采用了半同步复制，且设置的是rpl_semi_sync_master_wait_for_slave_count=2，如果其中一个挂掉了，对于rpl_semi_sync_master_wait_no_slave设置为ON的情况，此时显示的仍然是半同步复制，如果rpl_semi_sync_master_wait_no_slave设置为OFF，则会立刻变成异步复制。

**状态变量（show status like '%Rpl_semi%';）**

`mysql> show status like ``'%Rpl_semi%'``;``+--------------------------------------------+-------+``| Variable_name                              | Value |``+--------------------------------------------+-------+``| Rpl_semi_sync_master_clients               | 1     |``| Rpl_semi_sync_master_net_avg_wait_time     | 0     |``| Rpl_semi_sync_master_net_wait_time         | 0     |``| Rpl_semi_sync_master_net_waits             | 6     |``| Rpl_semi_sync_master_no_times              | 1     |``| Rpl_semi_sync_master_no_tx                 | 1     |``| Rpl_semi_sync_master_status                | ON    |``| Rpl_semi_sync_master_timefunc_failures     | 0     |``| Rpl_semi_sync_master_tx_avg_wait_time      | 1120  |``| Rpl_semi_sync_master_tx_wait_time          | 4483  |``| Rpl_semi_sync_master_tx_waits              | 4     |``| Rpl_semi_sync_master_wait_pos_backtraverse | 0     |``| Rpl_semi_sync_master_wait_sessions         | 0     |``| Rpl_semi_sync_master_yes_tx                | 4     |``+--------------------------------------------+-------+``14 rows ``in` `set` `(0.00 sec)`

Rpl_semi_sync_master_clients
当前半同步复制从的个数，如果是一主多从的架构，并不包含异步复制从的个数。

Rpl_semi_sync_master_no_tx
The number of commits that were not acknowledged successfully by a slave.
具体到上面的测试中，指的是insert into bobo.ceshi values(2)这个事务。

Rpl_semi_sync_master_yes_tx
The number of commits that were acknowledged successfully by a slave.
具体到上面的测试中，指的是以下四个事务:
mysql> create database bobo;
mysql> create table bobo.ceshi(id int);
mysql> insert into bobo.ceshi values(1);
mysql> insert into bobo.ceshi values(3); 

#### **简单总结**

1) 在一主多从的架构中，如果要开启半同步复制，并不要求所有的从都是半同步复制。
2) MySQL 5.7极大的提升了半同步复制的性能。
    5.6版本的半同步复制，dump thread 承担了两份不同且又十分频繁的任务：传送binlog 给slave ，还需要等待slave反馈信息，而且这两个任务是串行的，dump thread 必须等待 slave 返回之后才会传送下一个 events 事务。dump thread 已然成为整个半同步提高性能的瓶颈。在高并发业务场景下，这样的机制会影响数据库整体的TPS 。
    5.7版本的半同步复制中，独立出一个 ack collector thread ，专门用于接收slave 的反馈信息。这样master 上有两个线程独立工作，可以同时发送binlog 到slave ，和接收slave的反馈。

**Mysql半同步模式配置示例**                                                                           

`mysql主数据库： 172.16.60.205``mysql从数据库： 172.16.60.206``mysql5.6.39 安装部署，参考：https:``//www``.cnblogs.com``/kevingrace/p/6109679``.html` `主数据库172.16.60.205配置：``[root@mysql-master ~]``# cat /usr/local/mysql/my.cnf``.......``server-``id``=1``log-bin=mysql-bin``sync_binlog = 1``binlog_checksum = none``binlog_format = mixed` `从数据库172.16.60.206配置：``[root@mysql-slave ~]``# cat /usr/local/mysql/my.cnf``.........``server-``id``=2``log-bin=mysql-bin``slave-skip-errors = all` `然后从库同步操作，不需要跟master_log_file 和 master_log_pos=120``mysql> change master to master_host = ``'172.16.60.205'``, master_port = 3306, master_user =``'slave'``, master_password =``'slave@123'``;` `其他的配置，参考https:``//www``.cnblogs.com``/kevingrace/p/6256603``.html``即主从同步配置不过滤库，是基于整个数据库的同步。其他的操作和这个文档中记录的差不多` `========================================================``主数据库``[root@mysql-master ~]``# mysql -p123456``......``mysql> use kevin;``mysql> show tables;``+-----------------+``| Tables_in_kevin |``+-----------------+``| haha            |``+-----------------+``1 row ``in` `set` `(0.00 sec)``  ` `mysql> ``select` `* from haha;``+----+--------+``| ``id` `| name   |``+----+--------+``|  2 | anxun  |``|  3 | huoqiu |``|  4 | xihuju |``+----+--------+``3 rows ``in` `set` `(0.00 sec)``  ` `从数据库：``[root@mysql-slave ~]``# mysql -p123456``..........``..........``mysql> show slave status \G;``*************************** 1. row ***************************``               ``Slave_IO_State: Waiting ``for` `master to send event``                  ``Master_Host: 172.16.60.205``                  ``Master_User: slave``                  ``Master_Port: 3306``                ``Connect_Retry: 60``.........``.........``             ``Slave_IO_Running: Yes``            ``Slave_SQL_Running: Yes``  ` `mysql> ``select` `* from kevin.haha;``+----+--------+``| ``id` `| name   |``+----+--------+``|  2 | anxun  |``|  3 | huoqiu |``|  4 | xihuju |``+----+--------+``3 rows ``in` `set` `(0.00 sec)``  ` `主数据库插入数据``mysql> insert into haha values(1,``"linux"``);``Query OK, 1 row affected (0.03 sec)``mysql> insert into haha values(11,``"linux-ops"``);``Query OK, 1 row affected (0.04 sec)``  ` `从数据库查看``mysql> ``select` `* from kevin.haha;``+----+-----------+``| ``id` `| name      |``+----+-----------+``|  1 | linux     |``|  2 | anxun     |``|  3 | huoqiu    |``|  4 | xihuju    |``| 11 | linux-ops |``+----+-----------+``5 rows ``in` `set` `(0.00 sec)``  ` `mysql> ``select` `version();``+------------+``| version()  |``+------------+``| 5.6.39-log |``+------------+``1 row ``in` `set` `(0.00 sec)``  ` `mysql> show variables like ``"have_dynamic_loading"``;``+----------------------+-------+``| Variable_name        | Value |``+----------------------+-------+``| have_dynamic_loading | YES   |``+----------------------+-------+``1 row ``in` `set` `(0.00 sec)``  ` `从上面可知，已满足mysql半同步复制功能部署的条件：``1）MySQL 5.5及以上版本！``2）变量have_dynamic_loading为YES``3）主从复制已经存在！``  ` `===================================================``接下来进行mysql半同步复制环境部署：``  ` `1）加载mysql半同步复制的插件``主数据库执行:``mysql> INSTALL PLUGIN rpl_semi_sync_master SONAME ``'semisync_master.so'``;``Query OK, 0 row affected (0.04 sec)``  ` `从数据库执行：``mysql> INSTALL PLUGIN rpl_semi_sync_slave SONAME ``'semisync_slave.so'``;``Query OK, 0 row affected (0.04 sec)``  ` `2）查看插件是否加载成功的两种方式：``主数据库执行：``mysql> SELECT PLUGIN_NAME, PLUGIN_STATUS FROM INFORMATION_SCHEMA.PLUGINS  WHERE PLUGIN_NAME LIKE ``'%semi%'``;``+----------------------+---------------+``| PLUGIN_NAME          | PLUGIN_STATUS |``+----------------------+---------------+``| rpl_semi_sync_master | ACTIVE        |``+----------------------+---------------+``1 row ``in` `set` `(0.00 sec)``  ` `从数据库执行：``mysql> SELECT PLUGIN_NAME, PLUGIN_STATUS FROM INFORMATION_SCHEMA.PLUGINS  WHERE PLUGIN_NAME LIKE ``'%semi%'``;``+----------------------+---------------+``| PLUGIN_NAME          | PLUGIN_STATUS |``+----------------------+---------------+``| rpl_semi_sync_slave  | ACTIVE        |``+----------------------+---------------+``2 rows ``in` `set` `(0.01 sec)``  ` `3） 启动半同步复制``主数据库执行：``mysql> SET GLOBAL rpl_semi_sync_master_enabled = 1;``Query OK, 0 rows affected (0.00 sec)``  ` `从数据库执行：``mysql> SET GLOBAL rpl_semi_sync_slave_enabled = 1;``Query OK, 0 rows affected (0.00 sec)``  ` `----------------------------------------------------------------------------------------------------------------------------------------------``温馨提示：``除了上面的设置方法以外， 还可以以下面方式启动半同步复制，即在my.cnf文件中添加启动配置（推荐这种方式）：``主数据库``[root@mysql-master ~]``# vim /usr/local/mysql/my.cnf      #在[mysqld]区域添加下面内容``........``plugin-load=rpl_semi_sync_master=semisync_master.so``rpl_semi_sync_master_enabled=ON         ``#或者设置为"1"，即开启半同步复制功能``rpl-semi-``sync``-master-timeout=1000       ``#超时时间为1000ms，即1s``  ` `[root@mysql-master ~]``# /etc/init.d/mysqld restart``  ` `主数据库``[root@mysql-slave ~]``# vim /usr/local/mysql/my.cnf``........``plugin-load=rpl_semi_sync_slave=semisync_slave.so``rpl_semi_sync_slave_enabled=ON``  ` `[root@mysql-slave ~]``# /etc/init.d/mysqld restart``----------------------------------------------------------------------------------------------------------------------------------------------``  ` `4）查看半同步是否在运行``主数据库执行：``mysql> show status like ``'Rpl_semi_sync_master_status'``;``+-----------------------------+-------+``| Variable_name               | Value |``+-----------------------------+-------+``| Rpl_semi_sync_master_status | ON    |``+-----------------------------+-------+``1 row ``in` `set` `(0.00 sec)``  ` `从数据库执行(此时可能还是OFF状态，需要在下一步重启IO线程后，从库半同步状态才会为ON)：``mysql> show status like ``'Rpl_semi_sync_slave_status'``;``+----------------------------+-------+``| Variable_name              | Value |``+----------------------------+-------+``| Rpl_semi_sync_slave_status | ON    |``+----------------------------+-------+``1 row ``in` `set` `(0.00 sec)`` ` `5）重启从数据库上的IO线程``mysql> STOP SLAVE IO_THREAD;``Query OK, 0 rows affected (0.04 sec)``  ` `mysql> START SLAVE IO_THREAD;``Query OK, 0 rows affected (0.00 sec)``  ` `重启从数据的IO线程之后，  slave会在master上注册为半同步复制的slave角色``查看主数据库上的error日志，就会发现下面信息：``[root@mysql-master ~]``# tail -f /data/mysql/data/mysql-error.log``2019-01-06 23:23:34 10436 [Note] Semi-``sync` `replication initialized ``for` `transactions.``2019-01-06 23:23:34 10436 [Note] Semi-``sync` `replication enabled on the master.``2019-01-06 23:27:28 10436 [Note] Stop asynchronous binlog_dump to slave (server_id: 2)``2019-01-06 23:27:28 10436 [Note] Start semi-``sync` `binlog_dump to slave (server_id: 2), pos(mysql-bin.000004, 2944)``2019-01-06 23:32:03 10436 [Note] Stop semi-``sync` `binlog_dump to slave (server_id: 2)``2019-01-06 23:32:03 10436 [Note] Start semi-``sync` `binlog_dump to slave (server_id: 2), pos(mysql-bin.000004, 3357)``  ` `如上操作，就已经部署了mysql的半同步复制环境``  ` `现在往主数据库插入新数据``mysql> insert into kevin.haha values(5,``"grace"``);``Query OK, 1 row affected (0.13 sec)``  ` `mysql> insert into kevin.haha values(6,``"huihui"``);``Query OK, 1 row affected (0.11 sec)``  ` `到从数据库上查看，正常复制过来了``  ` `mysql> show slave status \G;``.........``.........``             ``Slave_IO_Running: Yes``            ``Slave_SQL_Running: Yes``  ` `mysql> ``select` `* from kevin.haha;``+----+-----------+``| ``id` `| name      |``+----+-----------+``|  1 | linux     |``|  2 | anxun     |``|  3 | huoqiu    |``|  4 | xihuju    |``|  5 | grace     |``|  6 | huihui    |``| 11 | linux-ops |``+----+-----------+``7 rows ``in` `set` `(0.00 sec)``  ` `查看主数据库``mysql> show status like ``'%Rpl_semi%'``;``+--------------------------------------------+-------+``| Variable_name                              | Value |``+--------------------------------------------+-------+``| Rpl_semi_sync_master_clients               | 1     |``| Rpl_semi_sync_master_net_avg_wait_time     | 475   |               ``#网络等待的平均时间``| Rpl_semi_sync_master_net_wait_time         | 1427  |               ``#网络等待时间   ``| Rpl_semi_sync_master_net_waits             | 3     |``| Rpl_semi_sync_master_no_times              | 0     |``| Rpl_semi_sync_master_no_tx                 | 0     |               ``#大于0就是异步。半同步是应为0 ``| Rpl_semi_sync_master_status                | ON    |``| Rpl_semi_sync_master_timefunc_failures     | 0     |``| Rpl_semi_sync_master_tx_avg_wait_time      | 622   |               ``# 平均等待时间``| Rpl_semi_sync_master_tx_wait_time          | 1868  |               ``#总的等待时间``| Rpl_semi_sync_master_tx_waits              | 3     |``| Rpl_semi_sync_master_wait_pos_backtraverse | 0     |``| Rpl_semi_sync_master_wait_sessions         | 0     |``| Rpl_semi_sync_master_yes_tx                | 3     |               ``#大于0就是半同步模式``+--------------------------------------------+-------+``14 rows ``in` `set` `(0.00 sec)`` ` `==========================================================``现在做下测试：``当半同步复制发生超时（由rpl_semi_sync_master_timeout参数控制，单位是毫秒，默认为10000，即10s），会暂时关闭半同步复制，转而使用异步复制。``当master dump线程发送完一个事务的所有事件之后，如果在rpl_semi_sync_master_timeout内，收到了从库的响应，则主从又重新恢复为半同步复制。`` ` `mysql> show variables like ``"rpl_semi_sync_master_timeout"``;``+------------------------------+-------+``| Variable_name                | Value |``+------------------------------+-------+``| rpl_semi_sync_master_timeout | 10000 |``+------------------------------+-------+``1 row ``in` `set` `(0.00 sec)`` ` `关闭从数据库的slave``mysql> stop slave;``Query OK, 0 rows affected (0.14 sec)`` ` `然后在主数据库插入一条数据，发现半同步复制会发生超时``发生超时后，暂时关闭半同步复制，转而使用异步复制``mysql> insert into kevin.haha values(55,``"laolao"``);``Query OK, 1 row affected (10.13 sec)`` ` `mysql> show status like ``'Rpl_semi_sync_master_status'``;``+-----------------------------+-------+``| Variable_name               | Value |``+-----------------------------+-------+``| Rpl_semi_sync_master_status | OFF   |``+-----------------------------+-------+``1 row ``in` `set` `(0.00 sec)`` ` `从数据库也会关闭半同步``mysql> show status like ``'Rpl_semi_sync_slave_status'``;``+----------------------------+-------+``| Variable_name              | Value |``+----------------------------+-------+``| Rpl_semi_sync_slave_status | OFF   |``+----------------------------+-------+``1 row ``in` `set` `(0.00 sec)`` ` `接着再打开从数据库的slave``mysql> start slave;``Query OK, 0 rows affected (0.04 sec)`` ` `然后主数据再插入一条数据，就会发现半同步复制不会超时，半同步复制功能打开也打开了``mysql> insert into kevin.haha values(555,``"laolaolao"``);``Query OK, 1 row affected (0.04 sec)`` ` `mysql> ``select` `* from haha;``+-----+-----------+``| ``id`  `| name      |``+-----+-----------+``|   1 | linux     |``|   2 | anhui     |``|   3 | huoqiu    |``|   4 | xihuju    |``|   5 | grace     |``|   6 | huihui    |``|  11 | linux-ops |``|  55 | laolao    |``| 555 | laolaolao |``+-----+-----------+``9 rows ``in` `set` `(0.00 sec)`` ` `mysql> show status like ``'Rpl_semi_sync_master_status'``;``+-----------------------------+-------+``| Variable_name               | Value |``+-----------------------------+-------+``| Rpl_semi_sync_master_status | ON    |``+-----------------------------+-------+``1 row ``in` `set` `(0.00 sec)`` ` `查看从数据库``mysql> ``select` `* from haha;``+-----+-----------+``| ``id`  `| name      |``+-----+-----------+``|   1 | linux     |``|   2 | anhui     |``|   3 | huoqiu    |``|   4 | xihuju    |``|   5 | grace     |``|   6 | huihui    |``|  11 | linux-ops |``|  55 | laolao    |``| 555 | laolaolao |``+-----+-----------+``9 rows ``in` `set` `(0.00 sec)`` ` `mysql> show status like ``'Rpl_semi_sync_slave_status'``;``+----------------------------+-------+``| Variable_name              | Value |``+----------------------------+-------+``| Rpl_semi_sync_slave_status | ON    |``+----------------------------+-------+``1 row ``in` `set` `(0.00 sec)`

通过上面的验证可知，遇到半同步复制超时情况，就会自动降为异步工作。可以在Slave上停掉半同步协议，然后在Master上创建数据库看一下能不能复制到Slave上。**需要注意一点的是，当Slave开启半同步后，或者当主从之间网络延迟恢复正常的时候，半同步复制会自动从异步复制又转为半同步复制，还是相当智能的。**

**删除"半同步复制", 恢复"异步复制"模式**                                                  

`先在从数据库上关闭半同步复制功能，然后卸载半同步插件``mysql> stop slave;``Query OK, 0 rows affected (0.07 sec)` `mysql> SET GLOBAL rpl_semi_sync_slave_enabled = 0;``Query OK, 0 rows affected (0.00 sec)` `mysql> show status like ``'Rpl_semi_sync_slave_status'``;``+----------------------------+-------+``| Variable_name              | Value |``+----------------------------+-------+``| Rpl_semi_sync_slave_status | OFF   |``+----------------------------+-------+``1 row ``in` `set` `(0.00 sec)` `mysql> UNINSTALL PLUGIN rpl_semi_sync_slave;``Query OK, 0 rows affected (0.00 sec)` `mysql> show status like ``'Rpl_semi_sync_slave_status'``;``Empty ``set` `(0.00 sec)` `mysql> SELECT PLUGIN_NAME, PLUGIN_STATUS FROM INFORMATION_SCHEMA.PLUGINS  WHERE PLUGIN_NAME LIKE ``'%semi%'``;``Empty ``set` `(0.01 sec)` `删除my.cnf里面的半同步配置``[root@mysql-slave mysql]``# vim /usr/local/mysql/my.cnf``#plugin-load=rpl_semi_sync_slave=semisync_slave.so``#rpl_semi_sync_slave_enabled=ON` `一定要重启mysql服务``[root@mysql-slave mysql]``# /etc/init.d/mysql restart``Shutting down MySQL..                                      [  OK  ]``Starting MySQL..                                           [  OK  ]` `=========================================================``接着再主数据库关闭半同步复制功能，卸载半同步插件``mysql> show status like ``'Rpl_semi_sync_master_status'``;``+-----------------------------+-------+``| Variable_name               | Value |``+-----------------------------+-------+``| Rpl_semi_sync_master_status | ON    |``+-----------------------------+-------+``1 row ``in` `set` `(0.00 sec)` `mysql> SET GLOBAL rpl_semi_sync_master_enabled = 0;``Query OK, 0 rows affected (0.00 sec)` `mysql> show status like ``'Rpl_semi_sync_master_status'``;``+-----------------------------+-------+``| Variable_name               | Value |``+-----------------------------+-------+``| Rpl_semi_sync_master_status | OFF   |``+-----------------------------+-------+``1 row ``in` `set` `(0.00 sec)` `mysql> show status like ``'%Rpl_semi%'``;``+--------------------------------------------+--------+``| Variable_name                              | Value  |``+--------------------------------------------+--------+``| Rpl_semi_sync_master_clients               | 0      |                               ``#确保这一行的数值为0，即没有半同步复制的客户端``| Rpl_semi_sync_master_net_avg_wait_time     | 40186  |``| Rpl_semi_sync_master_net_wait_time         | 401860 |``| Rpl_semi_sync_master_net_waits             | 10     |``| Rpl_semi_sync_master_no_times              | 2      |``| Rpl_semi_sync_master_no_tx                 | 1      |``| Rpl_semi_sync_master_status                | OFF    |``| Rpl_semi_sync_master_timefunc_failures     | 0      |``| Rpl_semi_sync_master_tx_avg_wait_time      | 592    |``| Rpl_semi_sync_master_tx_wait_time          | 4737   |``| Rpl_semi_sync_master_tx_waits              | 8      |``| Rpl_semi_sync_master_wait_pos_backtraverse | 0      |``| Rpl_semi_sync_master_wait_sessions         | 0      |``| Rpl_semi_sync_master_yes_tx                | 8      |``+--------------------------------------------+--------+``14 rows ``in` `set` `(0.00 sec)` `mysql> UNINSTALL PLUGIN rpl_semi_sync_master;``Query OK, 0 rows affected (0.00 sec)` `mysql> SELECT PLUGIN_NAME, PLUGIN_STATUS FROM INFORMATION_SCHEMA.PLUGINS  WHERE PLUGIN_NAME LIKE ``'%semi%'``;``Empty ``set` `(0.00 sec)` `mysql> show status like ``'Rpl_semi_sync_master_status'``;``Empty ``set` `(0.00 sec)` `删除my.cnf里面的半同步配置``[root@mysql-master ~]``# vim /usr/local/mysql/my.cnf``#plugin-load=rpl_semi_sync_master=semisync_master.so``#rpl_semi_sync_master_enabled=ON``#rpl-semi-sync-master-timeout=1000` `一定要重启mysql服务``[root@mysql-master ~]``# /etc/init.d/mysql restart``Shutting down MySQL...                                     [  OK  ]``Starting MySQL..                                           [  OK  ]` `=========================================================``再接着重启从数据库上的IO线程``mysql> STOP SLAVE IO_THREAD;``Query OK, 0 rows affected (0.04 sec)` `mysql> START SLAVE IO_THREAD; ``Query OK, 0 rows affected (0.00 sec)` `=========================================================``通过上面操作，就完全删掉了mysql半同步复制模式，恢复异步复制模式。``在关闭半同步复制模式的过程中，可以查看``/data/mysql/data/mysql-error``.log日志信息，从中观察到半同步关闭的信息。` `在从机重新start slave``mysql> start slave;``Query OK, 0 rows affected, 1 warning (0.00 sec)` `mysql> show slave status \G;``.........``.........``             ``Slave_IO_Running: Yes``            ``Slave_SQL_Running: Yes` `接着在主数据插入新数据``mysql> ``select` `* from kevin.haha;``+-----+-----------+``| ``id`  `| name      |``+-----+-----------+``|   1 | linux     |``|   2 | anhui     |``|   3 | huoqiu    |``|   4 | xihuju    |``|   5 | grace     |``|   6 | huihui    |``|  11 | linux-ops |``|  55 | laolao    |``| 555 | laolaolao |``+-----+-----------+``9 rows ``in` `set` `(0.00 sec)` `mysql> insert into kevin.haha values(12,``"wangjuan"``);``Query OK, 1 row affected (0.04 sec)` `mysql> insert into kevin.haha values(13,``"congcong"``);``Query OK, 1 row affected (0.04 sec)` `去从库上查看，发现已经同步过来了``mysql> ``select` `* from kevin.haha;``+-----+-----------+``| ``id`  `| name      |``+-----+-----------+``|   1 | linux     |``|   2 | anhui     |``|   3 | huoqiu    |``|   4 | xihuju    |``|   5 | grace     |``|   6 | huihui    |``|  11 | linux-ops |``|  12 | wangjuan |``|  13 | congcong |``|  55 | laolao    |``| 555 | laolaolao |``+-----+-----------+``9 rows ``in` `set` `(0.00 sec)` `======================================================``温馨提示：``经过几次实验，发现了一个坑，就是在做mysql半同步复制模式下，主从同步配置要是基于整个数据库的同步，而不要使用``"binlog_do_db"``和``"binlog_ingore_db"``进行过滤库的同步配置，否则会造成半同步复制模式下的数据同步失败。` `之前做过一次实验，mysql主从关系是针对某个具体库进行配置同步的，即：``主数据库的my.cnf配置：``#主从同步配置``server-``id``=1       ``log-bin=mysql-bin  ``binlog-``do``-db=kevin``binlog-ignore-db=mysql ``sync_binlog = 1   ``binlog_checksum = none``binlog_format = mixed` `# 半同步复制功能开启``plugin-load=rpl_semi_sync_master=semisync_master.so``rpl_semi_sync_master_enabled=ON   ``rpl-semi-``sync``-master-timeout=1000 ` `从数据库的my.cnf配置：``#主从同步配置``server-``id``=2  ``log-bin=mysql-bin  ``replicate-``do``-db=kevin``replicate-ignore-db=mysql ``slave-skip-errors = all` `# 半同步复制功能开启``plugin-load=rpl_semi_sync_slave=semisync_slave.so``rpl_semi_sync_slave_enabled=ON` `然后从库通过下面方式跟主数据库同步``mysql> change  master to master_host=``'172.16.60.205,master_user='``slave``',master_password='``slave@123``',master_log_file='``mysql-bin.000007',master_log_pos=120;` `以上配置的mysql主从同步是针对kevin这个具体的库的，在后续半同步复制模式下，主从数据同步失败。``而且在这种情况下，删除半同步复制模式配置，恢复到异步同步模式，主从数据同步还是失败。` `-----------------------------------------------------------``最后修改主从数据库的my.cnf文件，按照下面方式进行主从同步，则半同步复制模式下，主从数据同步正常。``并且删除半同步复制模式配置，恢复到异步同步模式，主从数据同步一样正常。` `主数据库的my.cnf文件调整后的配置：``#主从同步配置``server-``id``=1``log-bin=mysql-bin``sync_binlog = 1``binlog_checksum = none``binlog_format = mixed` `# 半同步复制功能开启``plugin-load=rpl_semi_sync_master=semisync_master.so``rpl_semi_sync_master_enabled=1``rpl-semi-``sync``-master-timeout=1000` `从数据库的my.cnf文件调整后的配置：``#主从同步配置``server-``id``=2``log-bin=mysql-bin``slave-skip-errors = all` `# 半同步复制功能开启``plugin-load=rpl_semi_sync_slave=semisync_slave.so``rpl_semi_sync_slave_enabled=1`` ` `然后从库通过下面方式跟主数据库同步``mysql> change master to master_host = ``'172.16.60.205'``, master_port = 3306, master_user =``'slave'``, master_password =``'slave@123'``;` `============================================================``特别注意下：``在做mysq主从同步时最好别过滤库了，即最好进行基于整个数据库的同步配置，同步命令为：``"change master to master_host = '主数据库ip', master_port = 主数据库mysql端口, master_user ='同步的用户', master_password ='同步的密码';"` `使用过滤库或过滤表的方式进行主从同步配置，后续会带来一些比较麻烦的坑。``如果业务数比较多的情况下，就使用mysql多实例方式进行同步，一个业务一个mysql实例，主从同步配置中不要进行过滤库或表的配置，即基于整个数据库同步。`

另外，在实际使用中还碰到一种情况从库IO线程有延迟时，主库会自动把半同步复制降为异步复制；当从库IO延迟没有时，主库又会把异步复制升级为半同步复制。可以进行压测模拟，但是此时查看Master的状态跟上面直接关闭Slave半同步有些不同，会发现Rpl_semi_sync_master_clients仍然等于1，而Rpl_semi_sync_master_status等于OFF。随着MySQL 5.7版本的发布，半同步复制技术升级为全新的Loss-less Semi-Synchronous Replication架构，其成熟度、数据一致性与执行效率得到显著的提升。

**MySQL 5.7半同步复制的改进**                                                          
现在我们已经知道，在半同步环境下，主库是在事务提交之后等待Slave ACK，所以才会有数据不一致问题。所以这个Slave ACK在什么时间去等待，也是一个很关键的问题了。因此MySQL针对半同步复制的问题，在5.7.2引入了Loss-less Semi-Synchronous，在调用binlog sync之后，engine层commit之前等待Slave ACK。这样只有在确认Slave收到事务events后，事务才会提交。在commit之前等待Slave ACK，同时可以堆积事务，利于group commit，有利于提升性能。

MySQL 5.7安装半同步模块，命令如下:

`mysql> ``install` `plugin rpl_semi_sync_master soname ``'semisync_master.so'``;``Query OK, 0 rows affected (0.00 sec)`

看一下相关状态信息:

`mysql> show global variables like ``'%semi%'``;``+-------------------------------------------+------------+``| Variable_name                             | Value      |``+-------------------------------------------+------------+``| rpl_semi_sync_master_enabled              | OFF        |``| rpl_semi_sync_master_timeout              | 10000      |``| rpl_semi_sync_master_trace_level          | 32         |``| rpl_semi_sync_master_wait_for_slave_count | 1          |``| rpl_semi_sync_master_wait_no_slave        | ON         |``| rpl_semi_sync_master_wait_point           | AFTER_SYNC |``+-------------------------------------------+------------+``6 rows ``in` `set` `(0.00 sec)`

**支持无损复制（Loss-less Semi-Synchronous）**
在Loss-less Semi-Synchronous模式下，master在调用binlog sync之后，engine层commit之前等待Slave ACK（需要收到至少一个Slave节点回复的ACK后）。这样只有在确认Slave收到事务events后，master事务才会提交，然后把结果返回给客户端。此时此事务才对其他事务可见。在这种模式下解决了after_commit模式带来的幻读和数据丢失问题，因为主库没有提交事务。但也会有个问题，假设主库在存储引擎提交之前挂了，那么很明显这个事务是不成功的，但由于对应的Binlog已经做了Sync操作，从库已经收到了这些Binlog，并且执行成功，相当于在从库上多了数据，也算是有问题的，但多了数据，问题一般不算严重。这个问题可以这样理解，作为MySQL，在没办法解决分布式数据一致性问题的情况下，它能保证的是不丢数据，多了数据总比丢数据要好。

无损复制其实就是对semi sync增加了rpl_semi_sync_master_wait_point参数，来控制半同步模式下主库在返回给会话事务成功之前提交事务的方式。rpl_semi_sync_master_wait_point该参数有两个值：AFTER_COMMIT和AFTER_SYNC

第一个值：AFTER_COMMIT（5.6默认值）

master将每个事务写入binlog（sync_binlog=1），传递到slave刷新到磁盘(sync_relay=1)，同时主库提交事务。master等待slave反馈收到relay log，只有收到ACK后master才将commit OK结果反馈给客户端。

![img](E:\markdown笔记\图片\907596-20190107182002244-1722224147.jpg)

第二个值：AFTER_SYNC（5.7默认值，但5.6中无此模式）

master将每个事务写入binlog , 传递到slave刷新到磁盘(relay log)。master等待slave反馈接收到relay log的ack之后，再提交事务并且返回commit OK结果给客户端。 即使主库crash，所有在主库上已经提交的事务都能保证已经同步到slave的relay log中。

![img](E:\markdown笔记\图片\907596-20190107182024513-1339398492.jpg)

**半同步复制与无损复制的对比**
1) ACK的时间点不同
**-**  半同步复制在InnoDB层的Commit Log后等待ACK，主从切换会有数据丢失风险。
**-**  无损复制在MySQL Server层的Write binlog后等待ACK，主从切换会有数据变多风险。

2) 主从数据一致性
**-**  半同步复制意味着在Master节点上，这个刚刚提交的事物对数据库的修改，对其他事物是可见的。因此，如果在等待Slave ACK的时候crash了，那么会对其他事务出现幻读，数据丢失。
**-**  无损复制在write binlog完成后就传输binlog，但还没有去写commit log，意味着当前这个事物对数据库的修改，其他事物也是不可见的。因此，不会出现幻读，数据丢失风险。

因此5.7引入了无损复制（after_sync）模式，带来的主要收益是解决after_commit导致的master crash后数据丢失问题，因此在引入after_sync模式后，所有提交的数据已经都被复制，故障切换时数据一致性将得到提升。

**-**  性能提升，支持发送binlog和接受ack的异步化

旧版本的semi sync受限于dump thread ，原因是dump thread承担了两份不同且又十分频繁的任务：传送binlog给slave ，还需要等待slave反馈信息，而且这两个任务是串行的，dump thread必须等待slave返回之后才会传送下一个events事务。dump thread已然成为整个半同步提高性能的瓶颈。在高并发业务场景下，这样的机制会影响数据库整体TPS 。



为了解决上述问题，在5.7版本的semi sync框架中，独立出一个Ack Receiver线程 ，专门用于接收slave返回的ack请求，这将之前dump线程的发送和接受工作分为了两个线程来处理。这样master上有两个线程独立工作，可以同时发送binlog到slave，和接收slave的ack信息。因此半同步复制得到了极大的性能提升。这也是MySQL 5.7发布时号称的Faster semi-sync replication。

![img](E:\markdown笔记\图片\907596-20190107182306864-1967536061.jpg)

但是在MySQL 5.7.17之前，这个Ack Receiver线程采用了select机制来监听slave返回的结果，然而select机制监控的文件句柄只能是0-1024，当超过1024时，用户在MySQL的错误日志中或许会收到类似如下的报错，更有甚者会导致MySQL发生宕机。

semi-sync master failed on net_flush() before waiting for slave reply.

MySQL 5.7.17版本开始，官方修复了这个bug，开始使用poll机制来替换原来的select机制，从而可以避免上面的问题。其实poll调用本质上和select没有区别，只是在I/O句柄数理论上没有上限了，原因是它是基于链表来存储的。但是同样有缺点：比如大量的fd的数组被整体复制于用户态和内核地址空间之间，而不管这样的复制是不是有意义。poll还有一个特点是“水平触发”，如果报告了fd后，没有被处理，那么下次poll时会再次报告该fd。

其实在高性能软件中都是用另外一种调用机制，名为epoll，高性能的代表，比如Nginx，haproxy等都是使用epoll。可能poll的复杂性比epoll低，另外对于ack receiver线程来说可能poll足矣。

**-**  性能提升，控制主库接收slave写事务成功反馈数量
MySQL 5.7新增了rpl_semi_sync_master_wait_slave_count参数，可以用来控制主库接受多少个slave写事务成功反馈，给高可用架构切换提供了灵活性。如图所示，当count值为2时，master需等待两个slave的ack。

![img](E:\markdown笔记\图片\907596-20190107182431634-198416297.jpg)

**-**  性能提升, Binlog互斥锁改进

旧版本半同步复制在主提交binlog的写会话和dump thread读binlog的操作都会对binlog添加互斥锁，导致binlog文件的读写是串行化的，存在并发度的问题。

![img](E:\markdown笔记\图片\907596-20190107182502927-837806755.jpg)

**MySQL 5.7对binlog lock进行了以下两方面优化:**
\-  移除了dump thread对binlog的互斥锁。
\-  加入了安全边际保证binlog的读安全。

 ![img](E:\markdown笔记\图片\907596-20190107182621759-532855673.jpg)

从replication功能引入后，官方MySQL一直在不停完善，但也可以发现当前原生的MySQL主备复制的实现，实际上很难在满足数据一致性前提下做到高可用高性能。

​                                                                 **参数sync_binlog/sync_relay与半同步复制**                                                                                       

sync_binlog的配置
其实无损复制流程中也会存在着会导致主备数据不一致的情况，使主备同步失败的情形。

当sync_binlog为0的时候，binlog sync磁盘由操作系统负责。当不为0的时候，其数值为定期sync磁盘的binlog commit group数。sync_binlog值不等于1的时候事务在FLUSH阶段就传输binlog到从库了，而值为1时，binlog同步操作是在SYNC阶段后。当sync_binlog值大于1的时候，sync binlog操作可能并没有使binlog落盘。如果没有落盘，事务在提交前，Master掉电，然后恢复，那么这个时候该事务被回滚。但是Slave上可能已经收到了该事务的events并且执行，这个时候就会出现Slave事务比Master多的情况，主备同步会失败。**所以如果要保持主备一致，需要设置sync_binlog为1。**

WAIT_AFTER_SYNC和WAIT_AFTER_COMMIT两图中Send Events的位置，也可能导致主备数据不一致，出现同步失败的情形。实际在rpl_semi_sync_master_wait_point分析的图中是sync binlog大于1的情况。流程如下图所示。Master依次执行flush binlog， update binlog position， sync binlog。如果Master在update binlog position后，sync binlog前掉电，Master再次启动后原事务就会被回滚。但可能出现Slave获取到Events，这也会导致Slave数据比Master多，主备同步失败。

![img](E:\markdown笔记\图片\907596-20190107183347837-1385844683.jpg)

由于上面的原因，sync_binlog设置为1的时候，MySQL会update binlog end pos after sync。流程如下图所示。这时候，对于每一个事务都需要sync binlog，同时sync binlog和网络发送events会是一个串行的过程，性能下降明显。

![img](E:\markdown笔记\图片\907596-20190107183411389-1585090122.jpg)

在Slave的IO线程中get_sync_period获得的是sync_relay_log的值，与sync_binlog对sync控制一样。当sync_relay_log不是1的时候，semisync返回给Master的position可能没有sync到磁盘。在gtid_mode下，在保证前面两个配置正确的情况下，sync_relay_log不是1的时候，仅发生Master或Slave的一次Crash并不会发生数据丢失或者主备同步失败情况。如果发生Slave没有sync relay log，Master端事务提交，客户端观察到事务提交，然后Slave端Crash。这样Slave端就会丢失掉已经回复Master ACK的事务events。

![img](E:\markdown笔记\图片\907596-20190107183437823-1202453790.jpg)

但当Slave再次启动，如果没有来得及从Master端同步丢失的事务Events，Master就Crash。这个时候，用户访问Slave就会发现数据丢失。

![img](E:\markdown笔记\图片\907596-20190107183512894-492663295.jpg)

通过上面这个Case，MySQL semisync如果要保证任意时刻发生一台机器宕机都不丢失数据，需要同时设置sync_relay_log为1。对relay log的sync操作是在queue_event中，对每个event都要sync，所以sync_relay_log设置为1的时候，事务响应时间会受到影响，对于涉及数据比较多的事务延迟会增加很多。

**MySQL三节点**
在一主一从的主备semisync的数据一致性分析中放弃了高可用，当主备之间网络抖动或者一台宕机的情况下停止提供服务。要做到高可用，很自然我们可以想到一主两从，这样解决某一网络抖动或一台宕机时候的可用性问题。但是，前文叙述要保证数据一致性配置要求依然存在，即正常情况下的性能不会有改善。同时需要解决Master宕机时候，如何选取新主机的问题，如何避免多主的情形。

![img](E:\markdown笔记\图片\907596-20190107183547698-1410068144.jpg)

选取新主机时一定要读取两个从机，看哪一个从机有最新的日志，否则可能导致数据丢失。这样的三节点方案就类似分布式Quorum机制，写的时候需要保证写成功三节点中的法定集合，确定新主的时候需要读取法定集合。利用分布式一致性协议Paxos/Raft可以解决数据一致性问题，选主问题和多主问题，因此近些年，国内数据库团队大多实现了基于Paxos/Raft的三节点方案。MySQL官方后续也以插件形式引入了支持多主集群的Group Replication方案。