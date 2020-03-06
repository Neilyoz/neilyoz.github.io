---
title: MySQL 轻松创建千万级测试数据
date: 2020-03-06 17:55:46
categories:
  - PHP
  - MySQL
tags:
  - MySQL
---

## 前言

疫情期间，无奈新公司迟迟不能报道，心里真的是B了狗了。对于优化数据库的文章网络上很多，但是我觉得很不负责的是都不成体系，要么就是讲的片面，讲的也是千篇一律，索引、分析表、优化sql语句、分库分表什么的。

说了很多理论，看着一堆都头大，而且时间久远，为了复习还是重新写写博文吧。我保证这些代码都是我实实在在测试后才会贴上来的。保证能粘贴就使用。

不着急数据库优化一步一步来，我们既然要优化数据库，那么第一步就是要有数据。这里会介绍两种快速生成大量测试数据的方法，分别使用存储过程和临时数据表。

<!-- more -->

## 创建数据表

无论我们使用哪种方式，都需要使用一个数据表

```mysql
CREATE TABLE `employee` (
    `id` int(11) NOT NULL AUTO_INCREMENT,
    `name` varchar(100) NOT NULL,
    `dep_id` int(11) NOT NULL,
    `age` int(11) NOT NULL,
    PRIMARY KEY (`id`)
) ENGINE = InnoDB DEFAULT CHARSET = utf8mb4;
```



## 使用存储过程

### 创建内存表

利用MySQL内存表插入速度快的特点，我们先利用函数和存储过程在内存表中生成数据，然后再从内存表插入普通表中。

```mysql
CREATE TABLE `employee_memory` (
    `id` int(11) NOT NULL AUTO_INCREMENT,
    `name` varchar(100) NOT NULL,
    `dep_id` int(11) NOT NULL,
    `age` int(11) NOT NULL,
    PRIMARY KEY (`id`)
) ENGINE = MEMORY DEFAULT CHARSET = utf8mb4;
```



### 创建函数及存储过程

创建随机字符串，MySQL的随机数常用的方式：`FLOOR( RAND() * (max - min) + min )` 生成[min, max]之间的数字包含min和max。

```mysql
DELIMITER $$
DROP FUNCTION IF EXISTS rand_string;
CREATE FUNCTION `rand_string`(n INT) RETURNS varchar(255) CHARSET utf8mb4
    DETERMINISTIC
BEGIN
    DECLARE chars_str varchar(100) DEFAULT 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789';
    DECLARE return_str varchar(255) DEFAULT '' ;
    DECLARE i INT DEFAULT 0;
    WHILE i < n DO
        SET return_str = concat(return_str, substring(chars_str, FLOOR(1 + RAND() * 62), 1));
        SET i = i + 1;
    END WHILE;
    RETURN return_str;
END;
$$

DELIMITER ;
```

创建插入数据存储过程

```mysql
DELIMITER $$
DROP PROCEDURE IF EXISTS add_employee_memory;
CREATE PROCEDURE add_employee_memory(IN n INT)
BEGIN
	DECLARE i INT DEFAULT 1;
	WHILE (i <= n) DO
        INSERT INTO `employee_memory` (name, dep_id, age) VALUES (rand_string(10), FLOOR(RAND() * (6-1) + 1), FLOOR(RAND() * (30 - 20) + 20));
        SET i = i + 1;
    END WHILE;
END
$$

DELIMITER ;
```



### 调用存储过程

```mysql
CALL add_employee_memory(1000000);
-- 从内存表插入普通表
INSERT INTO employee SELECT NULL, `name`, `dep_id`, `age` FROM employee_memory;
DELETE FROM employee_memory;
```

>  执行肯定时没有问题的，但是报错了！`SQL Error(1114)：The table 'employee_memory' is full`警告我们内存满。别慌，我们可以通过修改`max_heap_table_size=1024M`这个参数调整，我测试能装个200W数据，不够？要啥自行车，执行几次就好了。。

数据生成完了，当然要记得把函数和过程给干掉哟。

```mysql
DROP FUNCTION IF EXISTS rand_string;
DROP PROCEDURE IF EXISTS add_employee_memory;
```

如果使用脚本去逐条插入，大哥造千万数据的话，你就挂机睡觉吧，要不然LOL通宵也行啊，保证一个晚上你惊奇的发现也就能生成百万条数据。



## 使用临时数据表

### 创建临时表数据

```mysql
CREATE TABLE temp_user_info(
    `id` int(11) NOT NULL AUTO_INCREMENT,
    `name` varchar(100) NOT NULL,
    `age` int(11) NOT NULL,
    PRIMARY KEY (`id`)
) ENGINE = InnoDB DEFAULT CHARSET=utf8mb4
```

### 用你熟悉的语言生成数据文件(文件名为：data.txt)

```php
<?php
$userInfo = [];
$userStr = '';

for ($i=1; $i<=10000000; $i++) {
    $userInfo['id'] = $i;
    $userInfo['name'] = sprintf('php_user_%d', $i);
    $userInfo['age'] = rand(20, 30);
    // 1,php_user_1,22
    $userStr .= sprintf("%s\n", join(',', $userInfo));

    // 千万不要生成一条就入库一条！
    // 千万不要生成一条就入库一条！
    // 千万不要生成一条就入库一条！
    // 10w条再调用一次IO操作，一条调用一次操作，性能叫一个酸爽
    if ($i % 100000 === 0) {
        file_put_contents('./data.txt', $userStr, FILE_APPEND);
        echo $i . ' is Ok' . PHP_EOL;
        $userStr = '';
    }
}
```

文件生成的很快，我大PHP不愧是全世界最好的语言，哈哈哈。不说笑开始导入数据到临时表

```mysql
load data LOCAL infile '[文件完整路径]/data.txt' replace into table `temp_user_info` fields terminated BY ',' lines terminated by'\n';
```

## 总结

本文总结了两个我创建大量数据的方法，我个人更喜欢用第二种，毕竟我不喜欢写SQL，我们工作中绝大多数都是使用PHP，而SQL只是在需要的时候才会去看看文档怎么写，而且SQL排查错误真的是难受，而且第二种方法我个人觉得更加效率。


