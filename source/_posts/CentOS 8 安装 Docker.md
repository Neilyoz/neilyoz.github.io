---
title: CentOS 8 安装 Docker
date: 2020-03-27 01:07:41
categories:
  - Docker
tags:
  - Docker
---

## 简单粗暴的安装

学习任何一个新技术，我都推荐在虚拟机里面去操作，毕竟环境干净搞完蛋不影响原来的操作系统。

**依赖**

```shell
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
```

**添加 Docker 源**

```shell
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

**安装**

```shell
sudo yum install docker-ce docker-ce-cli containerd.io --nobest
```

**启动 Docker CE**

```shell
sudo systemctl enable docker
sudo systemctl start docker
```

## 建立用户组

一般我们不会直接用`root`这个大权限的用户来操作，一般需要建立一个组加入`docker`组：

```shell
sudo groupadd docker
```

将当前用户加入 `docker` 组

```shell
sudo usermod -aG docker $USER
```

忒出当前客户端重新登录，进行下面的测试

<!-- more -->

## Docker 镜像

由于某些原因，我们访问外网很慢，甚至是无法访问，好在有大厂这个镜像，所以赶紧换之。

新建一个配置文件`/etc/docker/daemon.json`

```shell
vi /etc/docker/daemon.json
```

输入以下内容：

```json
{
  "registry-mirrors": [
    "https://dockerhub.azk8s.cn",
    "https://hub-mirror.c.163.com"
  ]
}
```

改完一定要重启！改完一定要重启！改完一定要重启！

```shell
sudo systemctl daemon-reload
sudo systemctl restart docker
```

## 测试 Docker 是否安装成功

执行以下命令：

```shell
docker run hello-world
```

输出：

```
Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

看到这个就证明 `docker` 已经 OJBK 了。

## Docker 搭建 Swoole PHP 开发环境

我选择的是一个官方源 `php:7.4-cli` ，里面扩展还不是很全，但是后期可以通过 `docker-php-ext-install` 进行安装。

```shell
docker run -dit --name myswoole php:7.4-cli bash
```

这里我们来解析下相关的命令

- run 运行一个容器

- -d 让 docker 进入后台运行
- -t 终端
- -i 交互式操作
- --name 将这个容器命名为 `myswoole`，这里的命名自行定义就好了
- `php:7.4-cli` 这里是我选择的源
- bash 这里表时交互使用的 shell

输入这个命令后，你会看到一段 `hash` ，然后就是没有任何反应了。其实这里容器已经启动了，速度真的是杠杠的，我们可以通过以下命令去查看这个命令。

```shell
docker ps
```

这个命令会列出所有正在运行的容器进程：

```
CONTAINER ID        IMAGE               COMMAND                  ...NAMES
90558d9a592b        php:7.4-cli         "docker-php-entrypoi…"   ...myswoole
```

好了，既然容器有了，我们就要进入容器通过命令行操作他了。

```shell
# 进入名为 myswoole 容器的bash
docker exec -it myswoole bash

# 进入命令行后
# 升级我们的依赖
# 默认的这个源没有vim，我会选择自己安装
# 因为Swoole需要使用到libssl-dev扩展
apt update
apt install vim libssl-dev

# 通过 pecl 安装 swoole
pecl install swoole

# 具体socket openssl http2 mysqlnd 是否安装你自己选择

# 安装完通过以下命令，获取扩展配置路径
php --ini

cd /usr/local/etc/php/conf.d/
vi swoole.ini

# 输入 extension=swoole.so
# 保存退出
# 查看已经有swoole扩展了
php -m
```

此时已经安装完毕，我们退出 `bash` 。这个容器已经改变，我们需要保存这个容器的修改。这里我们需要使用`docker commit` 命令。

首先我们通过`docker ps`命令获取运行容器列表

```shell
docker ps

CONTAINER ID        IMAGE               COMMAND                  ...NAMES
90558d9a592b        php:7.4-cli         "docker-php-entrypoi…"   ...myswoole
```

这里容器的`hash`值为`90558d9a592b` ，我们要保存的就是这个容器，所以需要这个 hash

```shell
# 保存这个hash值对应的容器的修改
docker commit 90558d9a592b myswoole:php7.4-cli

# 我们的 image 里面就多了 myswoole 了
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
myswoole            php7.4-cli          44a70414dc94        6 seconds ago       489MB
myphp               7.4-cli             95e2af34d266        About an hour ago   627MB
ubuntu              18.04               4e5021d210f6        5 days ago          64.2MB
php                 7.4-cli             040c7fa6ecb6        6 days ago          405MB
```

既然镜像已经保存，那么我们就要停止他，我们需要绑定本地的文件夹到容器里面，方便容器运行`swoole`程序。

```shell
docker ps
docker stop 90558d9a592b
# 这个时候我们需要调整
docker run -dit --name myswoole -p 80:3000 -v "$PWD":/usr/src/myapp -w /usr/src/myapp myswoole:php7.4-cli php index.php
```

命令分析：

- -dit 表时后台运行交互终端
- --name 命名容器名称为 myswoole
- -p 80 端口和容器的 3000 端口绑定
- -v 映射本地当前工作目录到容器的 `/usr/src/myapp` 目录中
- -w 容器内的目录路径

执行命令，这里我们会发现报错了

```
docker: Error response from daemon: Conflict. The container name "/myswoole" is already in use by container "90558d9a592bddb6e94288b2ae4297448fb3094509696290d36fe6d1e268c4ce". You have to remove (or rename) that container to be able to reuse that name.
See 'docker run --help'.
```

提示我们这个容器已经存在相同的名称了，不慌简单粗暴干掉他们就好了。

```
docker ps -a

# 输出
CONTAINER ID        IMAGE                 COMMAND                  ...NAMES
90558d9a592b        php:7.4-cli           "docker-php-entrypoi…"   ...myswoole

# 开干
docker rm 90558d9a592b

# 再次执行命令
docker run -dit --name myswoole -p 80:3000 -v "$PWD":/usr/src/myapp -w /usr/src/myapp myswoole:php7.4-cli php index.php
```

在容器绑定的 "\$PWD" 目录下创建一个`index.php`

```php
<?php
$http = new Swoole\Http\Server("0.0.0.0", 3000);

$http->on('request', function ($request, $response) {
    $response->header("Content-Type", "text/html; charset=utf-8");
    $response->end("<h1>Hello Swoole. #".rand(1000, 9999)."</h1>");
});

$http->start();
```

在容器中执行 `php index.php`，然后我访问 本机地址的 80 端口。输出成功。这里只是单个程序的 Docker 学习，还有组合后面抽时间写。
