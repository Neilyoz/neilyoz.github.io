---
title: Mybatis-入门（六）
date: 2020-04-07 23:08:26
categories:
  - Java
tags:
  - Java
  - Mybatis
---

## 前言

日志作为我们开发中的辅助工具，早期我们开发只是使用 `System.out.println()` 来输出内容，但是在实际开发中我们更青睐于日志系统，Mybatis 也支持日志系统的配置，我们来看看日志系统是如何配置启动的。

## 设置日志系统配置

日志系统的配置相当简单，只需要在 `mybatis-config.xml` 文件中加入 `<settings>` 标签即可，注意这里的 `<setting>` 中 `name` 和 `value` 都不能写错，必须和官方提供的文档相同，否则就会有不知名的原因报错，我们为了减少报错，我一般的习惯是直接打开官方文档复制即可。

支持的类型有：

- SLF4J
- LOG4J 【掌握】
- LOG4J2
- JDK_LOGGING
- COMMONS_LOGGING
- STDOUT_LOGGING【掌握】
- NO_LOGGING

<!-- more -->

我们复用 `{% post_link "Mybatis-入门（五）" %}` 项目的代码，在代码的基础上，加入这个配置选项，我们先使用系统 `STDOUT_LOGGING` 标准输出。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">

<configuration>
    <properties resource="db.properties"/>

    <settings>
        <!-- 标准的日志工厂 -->
        <setting name="logImpl" value="STDOUT_LOGGING"/>
    </settings>

    <typeAliases>
        <typeAlias type="com.yubulang.pojo.User" alias="User"/>
    </typeAliases>

    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="${driver}"/>
                <property name="url" value="${url}"/>
                <property name="username" value="${username}"/>
                <property name="password" value="${password}"/>
            </dataSource>
        </environment>
    </environments>

    <mappers>
        <mapper resource="com/yubulang/dao/UserMapper.xml"/>
    </mappers>
</configuration>
```

配置之后我们运行对应的测试，这样我们就可以看到命令行中就出现了很多日志信息：

```
Logging initialized using 'class org.apache.ibatis.logging.stdout.StdOutImpl' adapter.
PooledDataSource forcefully closed/removed all connections.
PooledDataSource forcefully closed/removed all connections.
PooledDataSource forcefully closed/removed all connections.
PooledDataSource forcefully closed/removed all connections.
Opening JDBC Connection
Created connection 266437232.
Setting autocommit to false on JDBC Connection [com.mysql.jdbc.JDBC4Connection@fe18270]
==>  Preparing: select * from users where id = ?;
==> Parameters: 1(Integer)
<==    Columns: id, name, password
<==        Row: 1, 鱼不浪, 666666
<==      Total: 1
User{id=1, name='鱼不浪', password='666666'}
Resetting autocommit to true on JDBC Connection [com.mysql.jdbc.JDBC4Connection@fe18270]
Closing JDBC Connection [com.mysql.jdbc.JDBC4Connection@fe18270]
Returned connection 266437232 to pool.
```

## LOG4J 日志

通常我们使用的最多的就是 `LOG4J` 这个日志，学习一个东西的时候需要知道它是什么

- Log4j 是 Apache 的一个开源项目，通过使用 Log4j，我们可以控制日志信息输送的目的地是控制台、文件、GUI 组件
- 定义日志的输出格式
- 定义每一条的日志的级别
- 通过修改配置文件进行配置，不需要修改代码，这个才是我最喜欢的

- 先导入 LOG4J 的包，注意这里在配置文件当中必须大写且完全一样，直接 pom.xml 中添加

```xml
<!-- log4j -->
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.17</version>
</dependency>
```

配置完成后，maven 安装一下对应的包，当然我们还要配置对应的 log4j 属性，我们把它放到 `resources` 的 `log4j.properties` 中去。

```
# 设置rootLogger的level等级为DEBUG，后面跟输出目的地 console, file
log4j.rootLogger=DEBUG,console,file

# 控制台输出的相关配置
log4j.appender.console=org.apache.log4j.ConsoleAppender
log4j.appender.console.Target=System.out
log4j.appender.console.Threshold=DEBUG
log4j.appender.console.layout=org.apache.log4j.PatternLayout
log4j.appender.console.layut.ConversionPattern=[%c]-%m%n

# 文件输出的相关设置
log4j.appender.file = org.apache.log4j.RollingFileAppender
log4j.appender.file.File = ./log/yubulang.log
log4j.appender.file.MaxFileSize = 10mb
log4j.appender.file.Threshold = DEBUG
log4j.appender.file.layout = org.apache.log4j.PatternLayout
log4j.appender.file.layout.ConversionPattern = [%p][%d{yy-MM-dd}][%c]%m%n


# 日志输出级别
log4j.logger.org.mybatis=DEBUG
log4j.logger.java.sql=DEBUG
log4j.logger.java.sql.Statement=DEBUG
log4j.logger.java.sql.ResultSet=DEBUG
log4j.logger.java.sql.PreparedStatement=DEBUG
```

将 `<setting name="logImpl" value="STDOUT_LOGGING" />` 改为 `<setting name="logImpl" value="LOG4J" />` 这样运行我们的测试文件就已经可以正常运行了。输出日志如下：

```
Logging initialized using 'class org.apache.ibatis.logging.log4j.Log4jImpl' adapter.
Logging initialized using 'class org.apache.ibatis.logging.log4j.Log4jImpl' adapter.
PooledDataSource forcefully closed/removed all connections.
PooledDataSource forcefully closed/removed all connections.
PooledDataSource forcefully closed/removed all connections.
PooledDataSource forcefully closed/removed all connections.
Opening JDBC Connection
Created connection 300031246.
Setting autocommit to false on JDBC Connection [com.mysql.jdbc.JDBC4Connection@11e21d0e]
==>  Preparing: select * from users where id = ?;
==> Parameters: 1(Integer)
<==      Total: 1
User{id=1, name='鱼不浪', password='666666'}
Resetting autocommit to true on JDBC Connection [com.mysql.jdbc.JDBC4Connection@11e21d0e]
Closing JDBC Connection [com.mysql.jdbc.JDBC4Connection@11e21d0e]
Returned connection 300031246 to pool.
```

## Log4j 的简单使用

我们在测试类中 `com/yubulang/dao/UserMapperTest.java` 添加以下方法

```java
package com.yubulang.dao;

import com.yubulang.pojo.User;
import com.yubulang.utils.MybatisUtils;
import org.apache.ibatis.session.SqlSession;
import org.apache.log4j.Logger;
import org.junit.Test;

public class UserMapperTest {
    static Logger logger = Logger.getLogger(UserMapperTest.class);

    @Test
    public void testGetUserById() {
        SqlSession sqlSession = MybatisUtils.getSqlSession();

        UserMapper mapper = sqlSession.getMapper(UserMapper.class);

        User user = mapper.getUserById(1);

        System.out.println(user);

        sqlSession.close();
    }

    @Test
    public void testLog4j() {
        // 普通信息
        logger.info("info：进入了testLog4j");
        logger.debug("debug：进入了testLog4j");
        logger.error("error：进入了testLog4j");
    }
}
```

## 总结

日志在我们的项目开发中十分的重要，但是好运的是使用他并不困难，值要配置以下就能轻松的使用。
