---
title: Mybatis-入门（八）
date: 2020-04-09 16:47:12
categories:
  - Java
tags:
  - Java
  - Mybatis
---

## 前言

使用 xml 配置开发，我们 mybatis 也有注解开发。以后我们在公司做项目的时候多数会用到的注解，但是在 Mybatis 中还是 XML 为主。

我们这里也要回顾下什么是面向接口编程。

- 真正的开发中，很多时候我们会面向接口编程
- 根本原因：解耦，可拓展，提高复用，分层开发中，上层不用管具体的实现，大家都遵守共同的标准，是的开发变得容易，规范性好
- 在一个面向对象的系统中，系统的各种共嗯那个有许许多多不同的对象写作完成，大到各个模块之间的交互，在系统设计之初都是要着重考虑的，这也是系统设计的主要工作内容。面向接口编程就是按照这种思想来编程。

**关于接口的理解**

- 接口从更深层次的理解，就是定义（规范、约束）与实现（名实分离的原则）的分离
- 接口的本身反应了系统设计人员对系统的抽象理解
- 接口应有两类：
  - 第一类是对一个个体的抽象，它可对应为一个抽象体（abstract class）
  - 第二类是对一个个体某一方面的抽象，即形成一个抽象面（interface）
- 个体可能有多个抽象面，抽象体与抽象面是由区别的

**三个面向的区别**

- 面向对象是指，我们考虑问题时，以对象为单位，考虑它的属性及方法
- 面向过程是指，我们考虑问题时，以一个具体的流程（事务过程）为单位，考虑它的实现
- 接口设计与非接口设计时针对复用技术而言的，与面向对象（过程）不是一个问题。更多的体现就是对系整体的架构

<!-- more -->

## 使用注解开发

`Mybatis` 简单的操作我们可以使用注解来查询，这时候我们可以不需要配置 xml 文件，直接在接口类中去写注解。

```java
package com.yubulang.dao;

import com.yubulang.pojo.User;
import org.apache.ibatis.annotations.Insert;
import org.apache.ibatis.annotations.Select;

import java.util.List;

public interface UserMapper {

    @Select("select * from users")
    List<User> getUserList();

    @Insert("insert into users (`name`, `password`) values (#{name}, #{password})")
    int addUser(User user);
}
```

而我们的 `mybatis-config.xml` 中的 `mapper` 也只需要修改以下即可

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">

<configuration>
    <properties resource="db.properties"/>

    <settings>
        <setting name="logImpl" value="STDOUT_LOGGING"/>
    </settings>

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
        <!-- 绑定接口 -->
        <mapper class="com.yubulang.dao.UserMapper"/>
    </mappers>
</configuration>
```

开始写测试代码搞起，有了这两个例子，我相信修改和删除是什么样的结果也不难了吧

```java
package com.yubulang.dao;

import com.yubulang.pojo.User;
import com.yubulang.utils.MybatisUtils;
import org.apache.ibatis.session.SqlSession;
import org.junit.Test;

import java.util.List;

public class UserMapperTest {
    @Test
    public void testGetUserList() {
        SqlSession sqlSession = MybatisUtils.getSqlSession();

        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        List<User> userList = mapper.getUserList();

        for (User user : userList) {
            System.out.println(user);
        }

        sqlSession.close();
    }

    @Test
    public void testAddUser() {
        SqlSession sqlSession = MybatisUtils.getSqlSession();

        UserMapper mapper = sqlSession.getMapper(UserMapper.class);

        User user = new User();
        user.setName("金毛狮王");
        user.setPassword("666666");

        int i = mapper.addUser(user);
        if (i > 0) {
            System.out.println("插入成功");
        }

        sqlSession.commit();

        sqlSession.close();
    }
}
```

## 总结

mybatis 其实在实际开发中多数是用到的 xml，复杂的基本是通过 xml 配置来完成。注解只是拓展下大家的知识面。
