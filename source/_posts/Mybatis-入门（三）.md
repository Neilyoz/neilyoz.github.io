---
title: Mybatis-入门（三）
date: 2020-04-06 18:21:36
categories:
  - Java
tags:
  - Java
  - Mybatis
---

## 前言

经过了增删改查的编写，我们对 Mybatis 的使用有了一个大致的了解，但是仅仅知道增删改查的使用还不够，就好像我们使用 Nginx 去作为服务器，我们作为程序员不仅是要用，更多的是去配置他们一样。所以我们也要来学习一下 Mybatis 的配置，也是我们真正要掌握的能力。

## 核心配置文件

官方建议把配置文件命名为：`mybatis-config.xml`，配置文档中的层级机构如下：

- <span id="config">configuration（配置）</span>
  - properties（属性）**[完全掌握]**
  - settings（设置）**[部分掌握]**
  - typeAliases（类型别名）**[完全掌握]**
  - typeHandlers（类型处理器）[了解]
  - objectFactory（对象工厂）[了解]
  - plugins（插件）[了解]
  - environments（环境配置）**[完全掌握]**
    - environment（环境变量）
      - transactionManager（事务管理器）
      - dataSource（数据源）
  - databaseIdProvider（数据库厂商标识）[了解]
  - mappers（映射器）**[完全掌握]**

看着多，我们一个一个的看。我们进入配置的时候最先配置的肯定是连接数据库的环境配置。所以我们先看环境配置。

<!-- more -->

## environments（环境配置）

Mybatis 是可以配置多个环境的，这种机制有助于 SQL 隐射应用于多个数据库之中。例如：开发的时候我们用开发数据库、生产的时候我们使用生产数据库。

**不过要记住：尽管可以配置多个环境，但每个 SqlSessionFactory 实例只能选择一种环境。**

我们来看看：配置中的一些描述,这个是我配置的环境

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<!-- 核心配置文件 -->
<configuration>
    <!-- 多个开发环境配置 -->
    <!-- 这里可以配置多个环境，但是default是指定我们默认使用的环境是哪个 -->
    <!-- 修改值，只需要对应 <environment> 的id即可，这里默认的是development -->
    <environments default="development">
        <!-- id 是对环境的命名 -->
        <environment id="development">
            <!-- 事务管理器 -->
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url"
                          value="jdbc:mysql://localhost:3306/mybatis?useSSL=true&amp;useUnicode=true&amp;characterEncoding=utf8&amp;serverTimezone=Hongkong"/>
                <property name="username" value="root"/>
                <property name="password" value=""/>
            </dataSource>
        </environment>

        <environment id="test">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url"
                          value=""/>
                <property name="username" value=""/>
                <property name="password" value=""/>
            </dataSource>
        </environment>
    </environments>
</configuration>
```

通过上面例子，我们可以学会使用配置多套运行环境。Mybatis 默认的事务管理器就是 JDBC，默认连接池是 POOLED。

## 属性（properties）

我们可以通过 properties 属性来实现引用配置文件。这些属性是可以外部配置并且替换的，既可以在典型的 Java 属性文件中配置，亦可以通过 properties 元素的子元素来传递。

这里我们在 `resources` 中创建 `db.properties` 文件，输入以下内容

```
driver=com.mysql.jdbc.Driver
url=jdbc:mysql://localhost:3306/mybatis?useSSL=true&useUnicode=true&characterEncoding=utf8&serverTimezone=Hongkong
username=root
password=
```

输入完成后，这个文件可以在我们的核心配置文件中引入，引入时，我们需要注意顺序，引入顺序如上面，核心配置文件下面给出的顺序[跳转到配置处](#config)

修改后的配置文件如下：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<!-- 核心配置文件 -->
<configuration>
    <!-- 引入外部配置文件 -->
    <!-- 因为 db.properties 和 mybatis-config.xml 在同一个文件下，我们直接输入文件名就好了。 -->
    <properties resource="db.properties"/>

    <!-- 多个开发环境配置 -->
    <environments default="development">
        <environment id="development">
            <!-- 事务管理 -->
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <!-- 原来手写的文件，我们这里就可以直接使用 db.properties 中的定义的变量所代替 -->
                <property name="driver" value="${driver}"/>
                <property name="url" value="${url}"/>
                <property name="username" value="${username}"/>
                <property name="password" value="${password}"/>
            </dataSource>
        </environment>
    </environments>

    <!-- 每一个Mapper.xml都需要在mybatis核心配置文件中注册-->
    <mappers>
        <mapper resource="com/yubulang/dao/UserMapper.xml"/>
    </mappers>
</configuration>
```

当然，我们的\<properties\>标签之中还能写入单独的属性\<property\>，我们也可以把对应的属性单独定义在这里。具体的代码大家可以自己尝试一下。如果定义的属性同时存在于 `resource` 外部配置文件中，同时使用了\<property\>，他会有限使用外部文件中定义的变量。

## 别名（typeAliases）

- 类型别名可以为 Java 类设置一个缩写的名字。
- 它仅用于 XML 配置，意在降低冗余的全限定类名书写。

之前我们在 `UserMapper.xml` 中的类型上都需要使用全限定类名，这样写起来真的是又臭又长，就像下面这样

```xml
<select id="getUserList" resultType="com.yubulang.pojo.User">
    select * from users;
</select>
```

我们更加希望的是 `resultType="User"` 要达到这个目的，我们需要在 `mybatis-config.xml` 中设置\<typeAliases\>当然还是要提醒大家注意配置的顺序，必须按顺序配置，这里我给出了两个方式：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<!-- 核心配置文件 -->
<configuration>
    <!-- 引入外部配置文件 -->
    <properties resource="db.properties"/>

    <!-- 第一个方式：挨个给实体类起别名 -->
    <typeAliases>
        <typeAlias type="com.yubulang.pojo.User" alias="User"/>
    </typeAliases>

    <!-- 第二个方式：直接扫描指定包 -->
    <!-- 这里需要注意，我们扫描包的时候，它的默认别名是类名，首字母小写！ -->
    <!-- <typeAliases>
        <package name="com.yubulang.pojo"/>
    </typeAliases> -->

    <!-- 多个开发环境配置 -->
    <environments default="development">
        <environment id="development">
            <!-- 事务管理 -->
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="${driver}"/>
                <property name="url" value="${url}"/>
                <property name="username" value="${username}"/>
                <property name="password" value="${password}"/>
            </dataSource>
        </environment>
    </environments>

    <!-- 每一个Mapper.xml都需要在mybatis核心配置文件中注册-->
    <mappers>
        <mapper resource="com/yubulang/dao/UserMapper.xml"/>
    </mappers>
</configuration>
```

这个时候，我们的 `com/yubulang/dao/UserMapper.xml` 就可以使用指定的别名了，不需要再写常常的全限定类名。

### 何时使用别名（typeAliases），用哪种？

- 在实体类比较少的时候，使用第一种
- 在实体类比较多的时候，使用第二种

第一种可以自定义别名，第二种则不行，如果非要修改，可以使用 `@Alias()` 注解给对应的实体类起别名。像下面这样：

```java
package com.yubulang.pojo;

import org.apache.ibatis.type.Alias;

@Alias("CustomUser")
class User {
    ...
}
```

Mybatis 为常见的 Java 类型内建的类型别名。它们都是不区分大小写的，注意，为了应对原始类型的命名重复，采取了特殊的命名风格。
[❤【连接传送门】❤](https://mybatis.org/mybatis-3/zh/configuration.html#typeAliases)

## 设置（settings）

这是 MyBatis 中极为重要的调整设置，它们会改变 MyBatis 的运行时行为。目前记住对应的日志处理器怎么设置就好。具体的日志选项。

[❤【连接传送门】❤](https://mybatis.org/mybatis-3/zh/configuration.html#settings)

## 其他配置，用到再查，因为很少用。

- typeHandlers（类型处理器）
- objectFactory（对象工厂）
- plugins（插件）
  - mybatis-generator-core
  - mybatis-plus
  - 通用 mapper

## 映射器（mappers）

MapperRegistry：注册绑定我们的 Mapper 文件；
方式一：

```xml
<mappers>
    <mapper resource="com/yubulang/dao/UserMapper.xml"/>
</mappers>
```

方式二：使用 class 文件绑定注册

```xml
<mappers>
    <mapper class="com.yubulang.dao.UserMapper"/>
</mappers>
```

方式三：使用扫描包进行注入绑定

```xml
<mappers>
    <package class="com.yubulang.dao"/>
</mappers>
```

方式二和三注意点：

- 接口和他的 Mapper 配置文件必须同名
- 接口和它的 Mapper 配置文件必须在同一包下

## 总结

通过学习这里的配置案例，我们应该已经达到以下的目的：

- 将数据库配置文件外部引入
- 实体类别名
- 保证 UserMapper 接口和 UserMapper.xml 改为一致，并且放在同一个包下
