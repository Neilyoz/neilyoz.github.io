---
title: Ubuntu更新PHP版本
date: 2020-06-06 10:08:43
categories:
  - PHP
tags:
  - PHP
---

## 前言

都 2020 年了，PHP 版本都出到了 7.4.6。可是服务器上的 PHP 还是 7.1。对于我这个喜欢最新版本的人来说简直是不能忍受。怎么办？干！

## 安装 PHP 相关

公司环境有专门的运维，编译安装无可厚非。我们自己就没有必要折腾自己，又没啥特殊的服务要求，整那么多幺蛾子也没必要。直接添加源搞起来。

```shell
sudo apt-get install software-properties-common
sudo add-apt-repository ppa:ondrej/php
# 安装
sudo apt-get install php7.4 php7.4-fpm php7.4-mysql php7.4-gd php7.4-mbstring php7.4-bcmath php7.4-sqlite3 php7.4-cli php7.4-soap
# 启动
sudo service php7.4-fpm start
# 重启
sudo service php7.4-fpm restart
# 停止
sudo service php7.4-fpm stop

# 进行默认的版本选择
update-alternatives --config php
```

基本这样就完事儿了。剩下就是调整 nginxu 对应的 `fastcgi_pass` 参数，让 PHP 处理对应的请求就好了。

## 总结

别和我说什么不是编译安装不高级，特么的技术为的是解决问题，我的需求是快速搭建，没有什么特殊的管理需要，点到为止。而且现在服务器多数都是用 Docker 了。这种直接安装的无非就是一些老项目要迁移版本升级而已。简单实用快速就是王道。
