---
title: Mybatis-入门（四）
date: 2020-04-07 11:04:53
categories:
  - Java
tags:
  - Java
  - Mybatis
---

## 前言

在快速搭建 mybatis 环境，实现增删改查的操作，配置的相关操作之后，我们也需要了解一下 Mybatis 的生命周期以及作用域，之所以会提到肯定是在某些场景下容易出现问题。为了避免这些不必要的问题。

## 生命周期与作用域

<img src="/images/mybatis-rumen/生命周期与作用域.png" alt="右连接" style="zoom:100%;" />

不同作用域和生命周期类别是至关重要的，因为错误的使用会导致非常严重的 **并发问题**。

<!-- more -->

**SqlSessionFactoryBuilder:**

- 一旦创建了 SqlSessionFactory，就不需要它了
- 局部变量

**SqlSessionFactory：**

- 说白了可以把它认为是：数据库连接池
- SqlSessionFactory 一旦被创建就应该在应用的运行期间一直存在，**没有任何理由丢弃它或重新创建另一个实例**
- 因此 SqlSessionFactory 的最佳作用域是应用作用域（全局的）
- 最简单的就是使用单例模式或者静态单例模式

**SqlSession：**

- 连接到连接池的一个请求
- SqlSession 的实例不是线程安全的，因此是不能被共享的，所以它的最佳的作用域是请求或方法作用域。
- 有连接使用完也必须立马关闭，防止资源的占用

<img src="/images/mybatis-rumen/SqlSession相关流程.png" alt="右连接" style="zoom:100%;" />

这里的每个 Mapper，就代表一个具体的业务主要就是负责 SQL 语句的执行。

## 总结

了解 Mybatis 的生命周期与作用域，可以帮助我们理解在项目实际开发的过程中，遇到的一些资源占用问题，防止错误的创建数据库资源造成不必要的资源浪费的现象。
