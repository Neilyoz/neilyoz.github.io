---
title: MySQL8 Laravel数据库连接失败
date: 2020-05-31 11:27:04
categories:
  - MySQL
  - PHP
  - Laravel
tags:
  - MySQL
  - PHP
  - Laravel
---

## 前言

娘希匹的~ 懒得装环境直接 Docker 了一个 MySQL 最新版，版本是 8 来着。结果我用 Navicat 连接成功，满心欢喜的以为 OJBK 了。结果换 Laravel 去 migrate 时结果肯定时不行吖。

## 为什么不行？

标题有点开车的感觉，真实的刺激哈哈哈。因为 MySQL8 的密码加密方式换成了 `caching_sha2_password` 而我们的 PDO 貌似还不支持这种方式。所以只能是使用下面的命令进行降级了。将 `caching_sha2_password` 降级成为 `mysql_native_password`。

```mysql
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '{此处替换新的密码}';
FLUSH PRIVILEGES;
```

为什么是 PDO 问题呢？我在使用 Golang 的数据库时候并不会出现这个问题，而 PHP PDO 连接的时候就哎。

## 总结

谁让你 PHP 是世界上最好的语言呢？这么多新语言，我还是爱你的 PHP。
