---
title: MySQL主从复制配置
date: 2020-03-11 13:24:47
categories:
  - MySQL
tags:
  - MySQL
---
## 实现原理:

- MySQL支持单项、异步复制，复制过程中一个服务器充当主服务器，而一个或多个其他服务器充当从服务器。
- MySQL复制基于主服务器在二进制日志中跟踪所有对数据库的更改（更新、删除等等）
- 每个从服务器从主服务器接收主服务器已经记录到其二进制日志的保存更新。



## 实现步骤：

- Master将改变记录到二进制日志（**binary log**）中
- Slave将Master的**binary log events**拷贝到它的中继日志（relay log）
- Slave重做**中继日志**的时间，将改变反映它自己的数据。

![MySQL主从复制图解](https://oscimg.oschina.net/oscnet/8f7e2a97ec247fadc24443e5f6a58c466e0.jpg)
<!-- more -->
## 场景配置：

准备两台机器，IP为

- 192.168.0.130
- 192.168.0.131

在130服务器上打开my.cnf（MySQL的配置文件），找到相关binlog配置：

```ini
log-bin = mysql-bin #binlog开启
binlog_format = mixed #binlog的格式
server-id = 130 # 架构中的服务器唯一值
expire_logs_days = 10
early-plugin-load = ""
```

配置完成，重启Master Mysql服务器。连接我们的MySQL服务器运行命令查看服务器状态

```mysql
show master status;

-- 了解一下
-- reset master;可以重新开始我们的binlog 
```

这样，我们的Master主机上配置已经是OK了。



我们配置我们的131服务器上的从MySQL服务器配置，同样进入主机，我们打开my.cnf（MySQL的配置文件）

```ini
#log-bin = mysql-bin #注释关闭binlog日志
server-id = 131
relay_log = mysql-relay-bin
expire_logs_days = 10
early-plugin-load = ""
```

重启我们的Slave上的MySQL服务器。



完成上面操作，我们需要在Master MySQL中给Slave MySQL数据库授权，在我们Master上连接MySQL服务器。

```mysql
-- 我允许slave@192.168.0.131这台机器 通过 mysql密码 访问当前的Master
grant replication slave on *.* to slave@192.168.0.131 identified by 'mysql密码'
```

 主机已经配置完成。我们切换到我们的Slave主机，配置我们的Slave监听哪一台机器。

```mysql
stop slave;

change master to
master_host='192.168.0.130',
master_port=3306,
master_user='slave',
master_password='主机mysql密码',
master_log_file='mysql-bin.000001',
master_log_pos=xxx;
-- master_log_file、master_log_pos 通过Master的show master status;查看

start slave; -- 开启slave服务器

show slave status\G

-- 查看Slave_IO_Running: Yes、Slave_SQL_Running: Yes 这两项就行
```

这里我们的主从设置已经完成，我们只需要在Master主机器上进行MySQL的操作就能检测我们的主从是否正常的进行复制了。