---
title: PHP Closure闭包类详解
date: 2020-08-23 17:30:43
categories:
  - PHP
tags:
  - PHP
---

## 前言

最近很久没有更新 Blog 了，上班之后确实回家已经是瘫软状态，没办法搬砖要负责任，有奇葩需求要让奇葩需求不奇葩。

作为一个现代的 PHP 程序员很少有不接触 Laravel 的吧，接触了 Laravel 或多或少要接触 Collection 类，接触了 Collection 类肯定会接触到 Closure。

Collection 类是一个很好的工具类，用起来各种优雅，这里我们先卖个关子后面再源码解析 Collection 类。今天我们讲的是 Closure 闭包类的使用。

## Closure 闭包类

Closure 类是用于代表匿名函数的类，匿名函数会产生这个类型的对象，Closure 类摘要如下：

```php
Closure {
    /* 方法 */
    __construct ( void )
    public static Closure bind(
        Closure $closure ,
        object $newthis [, mixed $newscope = 'static']
    ): Closure
	public Closure bindTo(object $newthis [, mixed $newscope = 'static'] ): Closure
}
```

方法说明：

- `Closure::__construct` 用于禁止实例化的构造函数

- `Closure::bind` 复制一个闭包，绑定指定的 `$this` 对象和类作用域
- `Closure::bindTo` 复制当前闭包对象，绑定指定的 `$this` 对象和类作用域

`Closure::bind` 是 `Closure::bindTo` 的静态版本，其说明如下：

```
public static Closure bind(
        Closure $closure ,
        object $newthis [, mixed $newscope = 'static']
    ): Closure
```

参数说明：

- `$closure` 表示需要绑定的闭包对象
- `$newthis` 表示需要绑定到闭包对象的对象，或者 NULL 创建未绑定的闭包
- `$newscope` 表示想要绑定给闭包的类作用域，可以传入类名或类的实例，默认值是'static'，表示不改变。

```
public Closure bindTo(object $newthis [, mixed $newscope = 'static'] ): Closure
```

参数说明：

- `$newthis` 表示需要绑定到闭包对象的对象，或者 NULL 创建未绑定的闭包
- `$newscope` 表示想要绑定给闭包的类作用域，可以传入类名或类的实例，默认值是'static'，表示不改变。

<!-- more -->

## 相关实例

### `Closure::bind` 例子 1：

```php
<?php
/**
 * 复制一个闭包，绑定类作用域。
 */
class Person
{
    private static string $nickname = '小名';
}

/**
 * 回调函数
 * 获取Person类静态私有成员属性
 * @return mixed
 */
$person = function () {
    return static::$nickname;
};

// 给闭包绑定了 Person 实例的作用域，但未给闭包绑定 $this 对象
// 可以理解为：给回调函数中的static绑定对应的运行环境
$bindPerson = Closure::bind($person, null, new Person());

echo $bindPerson() . PHP_EOL;
```

### `Closure::bind` 例子 2：

```php
<?php
/**
 * 复制一个闭包，绑定 $this 和 类所有作用域。
 */
class Person
{
    private string $name = '张某';
}

/**
 * 回调函数
 * 获取Person类私有成员属性
 * @return mixed
 */
$person = function () {
    return $this->name;
};

// 给闭包绑定了 Person 类的作用域，
// 同时将 Person 实例对象作为$this对象绑定给闭包
$bindPerson = Closure::bind($person, new Person(), 'Person');

echo $bindPerson() . PHP_EOL;
```

### `Closure::bind` 例子 3：

```php
<?php
/**
 * 复制一个闭包，绑定 $this 和 类公有作用域。
 */
class Person
{
    public string $name = '孙悟空';
    private string $nickname = '斗战胜佛';
}

/**
 * 回调函数
 * 获取Person类公有成员属性
 * @return mixed
 */
$person = function () {
    // echo $this->nickname . PHP_EOL;
    return $this->name;
};

// 将 Person 实例对象作为 $this 对象绑定给闭包, 保留闭包原有作用域
$bindPerson = Closure::bind($person, new Person());

echo $bindPerson() . PHP_EOL;
```

### `Closure::bindTo` 例子：

```php
<?php
/**
 * 复制一个闭包，绑定 $this 和 类公有作用域。
 */
class Person
{
    public string $name = '孙悟空';
    private string $nickname = '斗战胜佛';
}

// 回调函数输出绑定对象的属性
$callback = function () {
    echo $this->name . PHP_EOL;
    echo $this->nickname . PHP_EOL;
};

// 将Person的上下文绑定到回调函数上
$bindPerson = $callback->bindTo(new Person(), new Person());

$bindPerson();
```

## 总结

其实 PHP 的 Closure 类是声明匿名函数时就会产生一个 Closure 对象，这个对象可以通过`bindTo`方法来指定\$this 绑定的对象以及对应的作用域。
