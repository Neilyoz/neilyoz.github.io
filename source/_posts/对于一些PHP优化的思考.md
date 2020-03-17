---
title: 对于一些PHP优化的思考
date: 2020-03-17 16:51:22
categories:
  - PHP
tags:
  - PHP
  - 优化
---

## 前言

这个真的是一个老生常谈了，偶尔放出来炒炒其实也挺香，PHP 我从`5.*`用到`7.*`，可以说真的是看着他辉煌看着他被其他语言（Golang、Node 捅菊花）。很多人也诟病它，但是我觉得我挺喜欢这门语言的。最最主要的是因为他简单方便。但是方便的东西，不一定能写好。

优化这个东西说实话，是技术的一些细枝末节，现在很多面试里面动不动就会问很多性能方面的东西，问的有水平的会给出具体的场景和一些相关数据，问的水的直接就是让你谈谈大数据量访问的处理，不给场景，不给任何信息，心里真的日了狗了，你怼他吧，他不爽还不一定要你。

环境：MacBook Pro 2015 款 8G 内存 I5 处理器 PHP 7.4.3

<!-- more -->

## PHP 优化规则开始

### 使用最新版本的 PHP，这个是最简单

这个毋庸置疑吧，简单粗暴。PHP 的大哥们费尽心思的让 PHP 变快，但是由于一些生产环境原因，很多时候大家不敢去进行升级，从而享受不到新版 PHP 带来的性能提升。

新项目不要想了，PHP 能有多新就用多新。这样至少在运行环境上就已经有了较老版本的质的飞跃。

### 双引号（"）和单引号（')的使用

先上代码测试来说一下：

```php
<?php

function doubleQuotes($iterations)
{
    $temp_str = "";
    $start_time = microtime(true);

    for ($i = 0; $i < $iterations; $i++) {
        $temp_str .= "Hello World!";
    }

    echo "Time for doubleQuotes(): " . (microtime(true) - $start_time) . PHP_EOL;
}

function singleQuotes($iterations)
{
    $temp_str = '';
    $start_time = microtime(true);

    for ($i = 0; $i < $iterations; $i++) {
        $temp_str .= 'Hello World!';
    }

    echo "Time for singleQuotes(): " . (microtime(true) - $start_time) . PHP_EOL;
}

doubleQuotes(50000000);
singleQuotes(50000000);

// result:
// Time for doubleQuotes(): 2.1713919639587
// Time for singleQuotes(): 2.1337790489197
```

这么一看，确实单引号比双引号快，用它用它。我是觉得现在版本的 PHP 这个上面的性能差不了多少，至于怎么用还是看大家习惯了，我喜欢用单引号纯粹的因为不用按`shift` 键，仅此而已。

### 别再 for 循环的条件中调用函数

很多时候我们为了省事儿是不是会写以下的代码：

```php
$arr = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
for ($i = 0; $i < count($arr); $i++) {
    echo $i . PHP_EOL;
}
```

这个代码可以跑，没问题的，但是你有没有想过，这个条件中的函数是会被每次都调用一次的。不信？那么我们自己定义一个函数来验证一下

```php
function customCount($arr)
{
    echo "Hello" . PHP_EOL;
    return count($arr);
}

$arr = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
for ($i = 0; $i < customCount($arr); $i++) {
    echo $i . PHP_EOL;
}
```

运行一下，结果是不是很意外，少一行代码高出了很多事儿。

```php
Hello
0
Hello
.
.
.
9
Hello
```

而正确的做法最好就是让循环的条件确定好，是一个确定的数字最好。修改这个代码的方式很简单。

```php
$arr = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
$length = count($arr); // 把确定循环的数字提前求出来
for ($i = 0; $i < $length; $i++) {
    echo $i . PHP_EOL;
}
```

### 避免在文件里面使用相对路径

这个在早期 PHP 的时候，由于还没有`composer`这个工具的时候，很多框架都会自己去写 `class_loader`，更多老的项目更加是直接就 `require`和`include`了。期间会很多会用到相对路径，代码难读就不说了，性能还不能保证。所以现在有新项目，麻烦大哥们能`composer`管理就用`composer`毕竟现在已经是包开发时代了，自己造轮子嘛，不是不可以，你老板不嫌你慢，你随便。

### 记得释放资源

这个其实真的没什么好说的，计算机角度来说内存和 CPU 是真的快的不是一星半点。但是有了提升不代表这些资源是无穷无尽的，他是有限的你申请一点，总数就少那么一点点，而且你还不归还。那总有消耗殆尽的时候嘛。代码上来说就是以下：

```php
$file = fopen('a.txt', 'rb'); // 申请
fclose($file); // 记得要归还
```

### 避免不必要的全局变量

很多老代码，像 Discuz 这种，一打开简直头大，各种 `global` 用的飞起来了。访问全局变量要比访问局部变量的开销来的大，可是为啥 Discuz 这么做，咱也不敢问。哈哈哈

### 使用 isset 代替 count()、strlen()、sizeof()

一般我们为了检测一个变量或者数组长度的时候会用到`count()`、`strlen()`、`sizeof()` 这些函数，但是这些长度操作可以用`isset`来代替：

```php
$str = "1234567";
echo strlen($str) === 7 . PHP_EOL;
echo isset($str[6]) . PHP_EOL;
```

### 能使用静态方法和静态变量就用它吧

静态方法相对于静态方法来的更加快，因为不用 new 对象，不用申请内存。当然现在有预加载机制了，这条我觉得有待商榷。

### foreach > for > while

这个就不多说了，反正多数是用 foreach。有时候数组的循环，可能会用`array_map()`来出来，毕竟现在又有什么函数式编程的概念了。

当然还有一个关键点的就是不要在 foreach 中频繁进行数据库查找等 I\O 操作，这个不是 PHP 给不给力的问题，而是数据库给不给力的问题了。频繁查找就相当于 I\O 操作，这个开销是巨大的。

### echo 和 print 输出变量用`','`取代`'.'`会得到性能提升

### 双引号拼接的东西可以用 sprintf 去拼接

不管是可读性上来说还是新能考虑来说，我更喜欢 sprintf。

```php
// 字段多了我头晕
$query = "SELECT * FROM users WHERE username='".$username"' AND AND password='" . $password . "'";

// 是不是更加舒服点
$query = sprintf( "SELECT * FROM users WHERE username='%s' AND password='%s'", $user, $password );
```

### 尽可能的使用(===)去判断

因为 `==` 还存在着一个隐形的变量转换，这个转换的过程也会消耗掉一些性能。

## 总结

其实 PHP 优化的点还有很多，这里给出的也只是语言层面的一些，以及一些平时自己喜欢的一些方法，更多的学习还是要看下大神的代码，我比较喜欢的一个框架就是 Laravel，虽然这个框架臃肿，但是里面的一些设计和实现还是值得细细品味的，我用的比较多的是这个库`Illuminate\Support\Collection`，要问我有什么缺点，缺点就是我快把循环给忘光了。。。
