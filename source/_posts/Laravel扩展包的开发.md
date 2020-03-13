---
title: Laravel扩展包的开发
date: 2020-03-12 01:07:41
categories:
  - PHP
tags:
  - PHP
  - Laravel
---
Laravel在2020年03月02日，发布了7.0版本，从3用到7一直都是用别人开发的包，由于疫情真的但是日了狗，在家找工作都不方便，怎么办学习吖，程序员哪里有休息可言。



## 扩展包的开发

### 1. 创建一个新项目，初始化扩展包配置

首先创建一个全新的Laravel项目：

```shell
composer create-project laravel/laravel package_demo --prefer-dist
```

接下来，在项目中创建目录`package/{your_name}/{your_package_name}`

```shell
mkdir -p packages/neilyoz/plugins
```

进入到这个目录，执行`composer init`

```shell
cd packages/neilyoz/plugins
composer init
```

接下来的就是看你具体的配置包信息了。执行完成后会生成一个composer文件。

```json
{
    "name": "neilyoz/plugins",
    "type": "library",
    "authors": [
        {
            "name": "Neilyoz",
            "email": "neilyoz@foxmail.com"
        }
    ],
    "require": {},
    "description": "plugins system build with laravel."
}
```

<!-- more -->

### 2. 创建扩展包基本目录，文件

一般情况下，我会创建一下的目录

```
packages/neilyoz/plugins$ tree .
├── composer.json
├── src
└── tests
```

### 3. 修改扩展包 composer 配置

然后修改我们这个扩展包的`composer.json`文件，设置一下自动加载配置、以及扩展包的命名空间。

```json
{
    "name": "neilyoz/plugins",
    "type": "library",
    "authors": [
        {
            "name": "Neilyoz",
            "email": "neilyoz@foxmail.com"
        }
    ],
    "require": {},
    "description": "plugins system build with laravel.",
    "autoload": {
        "psr-4": {
            "Neilyoz\\": "src/"
        }
    },
    "autoload-dev": {
        "psr-4": {
            "Neilyoz\\Plugins\\Tests\\": "test/"
        }
    }
}
```

### 4. 编写扩展包信息啦~

接下来，我们来创建 `PluginsServiceProvider` 、`Plugin` 文件。目录结构如下：

```
packages/neilyoz/plugins$ tree .
├── composer.json
├── src
│   └── Plugins
│       ├── Plugin.php
│       └── PluginsServiceProvider.php
└── tests
```

```php
<?php

namespace Neilyoz\Plugins;

use Illuminate\Support\ServiceProvider;

class PluginsServiceProvider extends ServiceProvider
{
    public function boot()
    {

    }

    public function register()
    {
        $this->app->singleton('plugins', function() {
            return new Plugin();
        });
    }
}
```

```php
<?php

namespace Neilyoz\Plugins;

class Plugin
{
    public function printRunning()
    {
        return 'Provider running...';
    }
}
```

到这里已经开发了一个最简单Laravel的扩展包了。



## 扩展包本地测试

把 `PluginsServiceProvider` 添加到项目的 `config/app.php` 中的 `providers` 数组中

```
'providers' => [
    ...
    Neilyoz\Plugins\PluginsServiceProvider::class,
],
```

这个时候要修改 `package_demo` 项目下的 `composer.json`

```json
{
    "require": {
        ...,
        "neilyoz/plugins": "dev-master"
    },
    ...,
    "autoload": {
        ...,
        "psr-4": {
            ...,
            "Neilyoz\\": "packages/neilyoz/plugins/src/"
        }
    },
    ...
}
```

运行命令：

```shell
composer dumpautoload
composer update
```

修改 `routes/web.php` 文件：

```
Route::get('/', function () {
    dd(app('plugins')->printRunning());
});
```

此时，我们打开浏览器访问，我这里项目对应url为：homestead.test，显示`Provider running...` 表时你已经成功了。



## 参考文章

[如何开发、本地测试、发布Laravel扩展包](https://learnku.com/articles/7426/how-to-develop-test-and-publish-a-laravel-extension-package)

