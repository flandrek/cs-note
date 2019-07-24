# Mysql集群搭建（多实例、主从）

------

# 1 MySQL多实例

## 一 、MySQL多实例介绍

### 1、什么是MySQL多实例

MySQL多实例就是在一台机器上开启多个不同的服务端口（如：3306，3307，3308），运行多个MySQL服务进程，通过不同的socket监听不同的服务端口来提供各自的服务。

### 2、MySQL多实例的特点有以下几点

- 有效利用服务器资源，当单个服务器资源有剩余时，可以充分利用剩余的资源提供更多的服务。
- 节约服务器资源
- 资源互相抢占问题，当某个服务实例服务并发很高时或者开启慢查询时，会消耗更多的内存、CPU、磁盘IO资源，导致服务器上的其他实例提供服务的质量下降；

### 3、部署mysql多实例的两种方式

第一种是使用**多个配置文件启动**不同的进程来实现多实例，这种方式的优势逻辑简单，配置简单，缺点是管理起来不太方便；

第二种是通过官方自带的**mysqld_multi**使用单独的配置文件来实现多实例，这种方式定制每个实例的配置不太方面，优点是管理起来很方便，集中管理；

### 4、同一开发环境下安装多个数据库，必须处理以下问题

- 配置文件安装路径不能相同
- 数据库目录不能相同
- 启动脚本不能同名
- 端口不能相同
- socket文件的生成路径不能相同

# 2 mysql多实例搭建

## 一、mysqld_multi搭建

### 1、下载免编译二进制包

下载地址：http://mirrors.sohu.com/mysql/MySQL-5.7/

mysql-5.7.17-linux-glibc2.5-x86_64.tar.gz

### 2、解压和迁移

- cd /usr/local

\#将安装包拖进local文件夹下并解压

- tar -zxvf mysql-5.7.17-linux-glibc2.5-x86_64.tar.gz
- mv mysql-5.7.17-linux-glibc2.5-x86_64 mysql

### 3、关闭iptables

\#临时关闭

- service iptables stop 

\#永久关闭

- chkconfig iptables off

### 4、关闭selinux

- vi /etc/sysconfig/selinux  

\#将SELINUX修改为DISABLED，即SELINUX=DISABLED 

![img](E:\markdown笔记\图片\2018080908271838.png)

### 5、创建mysql系统用户和组

- groupadd -g 27 mysql
- useradd -u 27 -g mysql mysql
- id mysql

![img](E:\markdown笔记\图片\20180809082729150.png)

### 6、创建mysql目录

- mkdir -p /data/mysql/mysql_3306/data
- mkdir -p /data/mysql/mysql_3306/log
- mkdir -p /data/mysql/mysql_3306/tmp
- mkdir -p /data/mysql/mysql_3307/data
- mkdir -p /data/mysql/mysql_3307/log
- mkdir -p /data/mysql/mysql_3307/tmp
- mkdir -p /data/mysql/mysql_3308/data
- mkdir -p /data/mysql/mysql_3308/log
- mkdir -p /data/mysql/mysql_3308/tmp

![img](E:\markdown笔记\图片\20180809082740493.png)

### 7、更改目录权限

**#****任意目录下，输入**

- chown -R mysql:mysql /data/mysql/ 
- chown -R mysql:mysql /usr/local/mysql/

### 8、添加环境变量

- echo 'export PATH=$PATH:/usr/local/mysql/bin' >>  /etc/profile 
- source /etc/profile

![img](E:\markdown笔记\图片\20180809082746491.png)

### **9、复制my.cnf文件到etc目录**

\#将原来的my.cnf文件删除了

- cp /usr/local/mysql/support-files/my-default.cnf /etc/my.cnf

### 10、修改my.cnf（在一个文件中修改即可）

- vi  /etc/my.cnf
- [client]port=3306socket=/tmp/mysql.sock [mysqld_multi]mysqld = /usr/local/mysql/bin/mysqld_safemysqladmin = /usr/local/mysql/bin/mysqladminlog = /data/mysql/mysqld_multi.log [mysqld]basedir = /usr/local/mysqlsql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES #3306数据库[mysqld3306]mysqld=mysqldmysqladmin=mysqladmindatadir=/data/mysql/mysql_3306/dataport=3306server_id=3306socket=/tmp/mysql_3306.socklog-output=fileslow_query_log = 1long_query_time = 1slow_query_log_file = /data/mysql/mysql_3306/log/slow.loglog-error = /data/mysql/mysql_3306/log/error.logbinlog_format = mixedlog-bin = /data/mysql/mysql_3306/log/mysql3306_bin #3307数据库[mysqld3307]mysqld=mysqldmysqladmin=mysqladmindatadir=/data/mysql/mysql_3307/dataport=3307server_id=3307socket=/tmp/mysql_3307.socklog-output=fileslow_query_log = 1long_query_time = 1slow_query_log_file = /data/mysql/mysql_3307/log/slow.loglog-error = /data/mysql/mysql_3307/log/error.logbinlog_format = mixedlog-bin = /data/mysql/mysql_3307/log/mysql3307_bin #3308数据库[mysqld3308]mysqld=mysqldmysqladmin=mysqladmindatadir=/data/mysql/mysql_3308/dataport=3308server_id=3308socket=/tmp/mysql_3308.socklog-output=fileslow_query_log = 1long_query_time = 1slow_query_log_file = /data/mysql/mysql_3308/log/slow.loglog-error = /data/mysql/mysql_3308/log/error.logbinlog_format = mixedlog-bin = /data/mysql/mysql_3308/log/mysql3308_bin

### **11、初始化数据库**

 **#初始化3306数据库** 

- /usr/local/mysql/scripts/mysql_install_db --basedir=/usr/local/mysql/ --datadir=/data/mysql/mysql_3306/data --defaults-file=/etc/my.cnf  

![img](E:\markdown笔记\图片\20180809082907736.png)

![img](E:\markdown笔记\图片\20180809082918142.png)

 **#****初始化****3307****数据库** 

- /usr/local/mysql/scripts/mysql_install_db --basedir=/usr/local/mysql/ --datadir=/data/mysql/mysql_3307/data --defaults-file=/etc/my.cnf

 **#****初始化****3308****数据库** 

- /usr/local/mysql/scripts/mysql_install_db --basedir=/usr/local/mysql/ --datadir=/data/mysql/mysql_3308/data --defaults-file=/etc/my.cnf

**12****、查看数据库是否初始化成功**

查看3306、3307、3308数据库

- cd /data/mysql/mysql_3306/data/
- cd /data/mysql/mysql_3307/data/
- cd /data/mysql/mysql_3308/data/

![img](E:\markdown笔记\图片\20180809082936492.png)

### 13、**设置启动文件**

- cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysql

### 14、**mysqld_multi****进行多实例管理**

启动全部实例：/usr/local/mysql/bin/mysqld_multi start

查看全部实例状态：/usr/local/mysql/bin/mysqld_multi report 

启动单个实例：/usr/local/mysql/bin/mysqld_multi start 3306 

停止单个实例：/usr/local/mysql/bin/mysqld_multi stop 3306 

查看单个实例状态：/usr/local/mysql/bin/mysqld_multi report 3306 

\#**启动全部实例**

- /usr/local/mysql/bin/mysqld_multi start
- /usr/local/mysql/bin/mysqld_multi report

![img](E:\markdown笔记\图片\20180809082943704.png)

\# **查看启动进程**

- ps -aux | grep mysql

![img](E:\markdown笔记\图片\2018080908294889.png)

\#进入tmp目录，查看sock文件

- cd /tmp

![img](E:\markdown笔记\图片\20180809082954232.png)

### 15、**修改密码**

mysql的root用户初始密码是空，所以需要登录mysql进行修改密码，下面以3306为例：

- - mysql -S /tmp/mysql_3306.sock
  - set password for root@'localhost'=password('xxxxxx');
  - flush privileges;

![img](E:\markdown笔记\图片\20180809083629338.png)

\#修改3307数据库密码

- mysql -S /tmp/mysql_3307.sock
- set password for root@'localhost'=password('xxxxxx');
- flush privileges;

\#修改3308数据库密码

- mysql -S /tmp/mysql_3307.sock
- set password for root@'localhost'=password('xxxxxx');
- flush privileges;

### 16、**新建用户及授权**

一般新建数据库都需要新增一个用户，用于程序连接，这类用户只需要insert、update、delete、select权限。这里赋予所有权限，以3306为例，3307、3308相同。

\#新增一个用户，并授权如下： 

- grant ALL PRIVILEGES on *.* to admin@'%' identified by 'xxxxxx'; 
- flush privileges

![img](E:\markdown笔记\图片\20180809083707586.png)

### **17****、外部软件登录数据库**

![img](E:\markdown笔记\图片\20180809083021122.png)

图1

![img](E:\markdown笔记\图片\2018080908302741.png)

图2

## 二、多配置文件搭建

### 1、安装包分发

140搭建的有mysql数据库

\#将数据库安装包分发到其他两台机器

- scp -r mysql/ [root@192.168.3.141:/usr/local/](mailto:root@192.168.3.141:/usr/local/)
- scp -r mysql/ [root@192.168.3.142:/usr/local/](mailto:root@192.168.3.142:/usr/local/)

### 2、卸载冗余数据库服务

\#查找之前安装的数据库服务，若有，怎卸载数据库服务

- rpm -qa|grep -i mysql
- rpm -qa|grep -i MariaDB
- rpm -ev mariadb-libs-5.5.56-2.el7.x86_64 --nodeps
- ![img](E:\markdown笔记\图片\20180809083101817.png)
- rpm -ev mysql-community-test-5.7.19-1.el7.x86_64 mysql-community-devel-5.7.19-1.el7.x86_64 mysql-community-libs-5.7.19-1.el7.x86_64 mysql-community-embedded-compat-5.7.19-1.el7.x86_64 mysql-community-embedded-devel-5.7.19-1.el7.x86_64 mysql-community-server-5.7.19-1.el7.x86_64 mysql-community-minimal-debuginfo-5.7.19-1.el7.x86_64 mysql-community-common-5.7.19-1.el7.x86_64 mysql-community-libs-compat-5.7.19-1.el7.x86_64 mysql-community-embedded-5.7.19-1.el7.x86_64 mysql-community-client-5.7.19-1.el7.x86_64 mysql-community-server-minimal-5.7.19-1.el7.x86_64 --nodeps

### 3、安装数据库

**在141****上安装数据库**

\#安装mysql的环境

- yum -y install gcc gcc-c++ gdb cmake ncurses-devel bison bison-devel 

\#解压mysql安装包

- tar -zxvf MySQL-5.7.19-1.el7.x86_64_64.rpm-bundle.tar

![img](E:\markdown笔记\图片\20180809083108435.png)

\#安装mysql服务

- sudo rpm -ivh --force mysql-community-client-5.7.19-1.el7.x86_64.rpm mysql-community-common-5.7.19-1.el7.x86_64.rpm mysql-community-devel-5.7.19-1.el7.x86_64.rpm mysql-community-embedded-5.7.19-1.el7.x86_64.rpm mysql-community-embedded-compat-5.7.19-1.el7.x86_64.rpm mysql-community-embedded-devel-5.7.19-1.el7.x86_64.rpm mysql-community-libs-5.7.19-1.el7.x86_64.rpm mysql-community-libs-compat-5.7.19-1.el7.x86_64.rpm mysql-community-minimal-debuginfo-5.7.19-1.el7.x86_64.rpm mysql-community-server-5.7.19-1.el7.x86_64.rpm mysql-community-server-minimal-5.7.19-1.el7.x86_64.rpm mysql-community-test-5.7.19-1.el7.x86_64.rpm  --nodeps

![img](E:\markdown笔记\图片\20180809083117112.png)

\#查看安装的mysql服务

- rpm -qa|grep -i mysql

![img](E:\markdown笔记\图片\20180809083122626.png)

**在142****上安装数据库：与上面同样的操作**

### 4、启动mysql并查看状态

\#启动

- service mysqld start

\#查看数据库运行状态

- service mysqld status

![img](E:\markdown笔记\图片\20180809083129838.png)

\#查看MySQL的进程

- ps -aux|grep mysql

![img](E:\markdown笔记\图片\20180809083141674.png)

### 5、设置开机自启

- systemctl enable mysqld.service

### 6、设置初始密码

\#查看随机生成的mysql密码

- grep "password" /var/log/mysqld.log

![img](E:\markdown笔记\图片\20180809083147216.png)

\#登陆数据库客户端

- mysql -u root -p

注意：用随机生成的密码

\#设置新密码

- set password = password('xxxxxx');

### 7、新建用户和修改权限

\#修改root用户的权限

- grant ALL PRIVILEGES on *.* to root@'%' identified by 'xxxxxx';

\#创建amdin用户，并赋予限

- CREATE USER 'admin'@'%' IDENTIFIED BY 'xxxxxx';  
- grant ALL PRIVILEGES on *.* to admin@'localhost' identified by 'xxxxxx';
- FLUSH PRIVILEGES;

\#查看用户的权限

- select host,user from mysql.user;

![img](E:\markdown笔记\图片\20180809083153817.png)

**在142****上设置数据库：与上面同样的操作**

 

# 2 mysql集群搭建

## 一、mysql主从集群

### 1、建立siger用户并设置权限

登陆140的mysql数据库

- mysql -u root -p
- CREATE USER 'siger'@'%'IDENTIFIED BY 'xxxxxx';
- GRANT ALL PRIVILEGES ON *.* TO 'siger'@'%';
- grant ALL PRIVILEGES on *.* to siger@'localhost' identified by 'xxxxxx';
- FLUSH PRIVILEGES;
- select host,user from mysql.user;

![img](E:\markdown笔记\图片\20180809083206455.png)

### 2、修改配置文件

修改三台数据库的配置文件

\#140数据库的配置文件

- vi /etc/my.cnf

[mysqld]

log-bin=mysql-bin

server-id=140

![img](E:\markdown笔记\图片\20180809083213627.png)

\#141数据库的配置文件

- vi /etc/my.cnf

[mysqld]

log-bin=mysql-bin

server-id=141

\#142数据库的配置文件

- vi /etc/my.cnf

[mysqld]

log-bin=mysql-bin

server-id=142

### 3、查看master状态

- SHOW MASTER STATUS;

![img](E:\markdown笔记\图片\20180809083220150.png)

### 4、配置slave

\#登陆在140，141据库，在slave上链接master

- change master to master_host='172.8.10.140',master_user='siger',master_password='xxxxxx',master_log_file='mysql-bin.000004',master_log_pos=77461339;

\#参数配置

master_host：主节点mysql的IP地址

master_user：主节点mysql登陆用户的用户名

master_password：主节点mysql登陆用户的密码

master_log_file：主节点mysql的日志文件指定

master_log_pos：主节点mysql的Ppsition的ID

![img](E:\markdown笔记\图片\2018080908522676.png)

### 5、启动从服务

\#在141数据库上

- start slave;

\#在142数据库上

- start slave;

### 6、查看slave状态

\#在3307数据库上

- show slave status\G

![img](E:\markdown笔记\图片\20180809083235833.png)

\#在3308数据库上

- show slave status\G

![img](E:\markdown笔记\图片\20180809083240825.png)

### 7、主从测试

\#在master建立数据库siger

- create database siger;
- use siger;
- create table slave_test(id int(6),name varchar(10))
- insert into slave_test values(000001,'weiyang');
- FLUSH PRIVILEGES;
- show tables;

![img](E:\markdown笔记\图片\20180809083247998.png)

\#在141、142从节点上查看

![img](E:\markdown笔记\图片\20180809083254816.png)