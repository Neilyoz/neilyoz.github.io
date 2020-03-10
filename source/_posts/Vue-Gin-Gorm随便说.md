---
title: Vue + Gin + Gorm 随便说
date: 2020-03-10 21:18:14
categories:
  - Golang
tags:
  - Golang
  - Gin
  - Gorm
  - Vue
---

## 项目连接

😈 [传送门](https://github.com/Neilyoz/goedu) 😈

## 背景

在家无聊搭建一个 Vue + Gin + Gorm 的程序，主要也是熟悉一下看了两天的Golang语言，在做的过程中能复习Golang的语法以及一些相关的前端Vue知识。

- 熟悉了gin框架，以及中间件的使用
- 熟悉了gorm数据库ORM的一些数据库操作
- 复习Vue的一些相关知识点
- 使用jwt-go搭建jwt验证，并把验证放到Gin中间件中
- 使用yaml把配置文件独立到`config`文件夹



## 如何搭建

1. `frontend`属于前端部分，进入到这个文件夹下

```shell
npm install && npm run serve
```

2. 配置好数据库
3. `config`文件夹，配置`config.yaml`

```yaml
debug: true # 数据库调试，开启后命令行会显示对应的数据库操作
key: alexskywin # jwt加密串
mysql: # MySQL相关配置
  user: root
  pass:
  host: 127.0.0.1
  port: 3306
  database: demo
  charset: utf8mb4
```

4. 运行`main.go`文件

```shell
go run main.go
```

毕竟只是快速看看怎么回事，go run就可以了。



## 总结

这个相当于一个前后端分离的架子，但是在做这个架子的时候，还是遇到了很多困难，翻看各种文档和各种Golang语法查漏补缺的时候，还是温故知新了不少东西。

说实话，如果不是公司有特别要求，我还是喜欢Laravel这个自动挡多一些。gin + gorm目前还是半自动档的框架，没办法谁让Golang火吖，不说精通但是也得会。不过Golang上手确实很快。

2020年Golang也会作为我一个研究的方向，当然不会丢下我的PHP，毕竟是世界上最好的语言！哈哈哈