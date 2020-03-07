---
title: MySQL最大值Max的优化
date: 2020-03-08 02:16:04
categories:
  - MySQL
tags:
  - MySQL
---
## 前言

一个一个例子来，获取最大值max的优化。求数据的最大值，说实在话在我们的实际开发中还是挺好用的。可能数据表一万两万的时候不是很看的出来性能，但是任何东西都顶不住数量多啊。为了模拟这个数量多的环境，不废话，贴出代码，方便大家。

<!-- more -->

## 创建所需要的数据表以及数据

先来 `300W` 数据压压惊，我们来找 `age` 来找大哥，看谁才是最牛逼，最大的人。

```mysql
-- 创建数据表
DROP TABLE IF EXISTS `testemployee`;
CREATE TABLE `testemployee` ( 
	`id` int(11) NOT NULL AUTO_INCREMENT,
	`name` VARCHAR(20) DEFAULT NULL,
	`dep_id` int(11) DEFAULT NULL,
	`age` int(11) DEFAULT NULL,
	`salary` DECIMAL(10, 2) DEFAULT NULL,
	`cus_id` int(11) DEFAULT NULL,
	PRIMARY key(`id`)
) ENGINE = INNODB DEFAULT CHARSET=utf8mb4;

-- 随机生成字符串
delimiter $$
DROP FUNCTION IF EXISTS `rand_str`;
CREATE FUNCTION rand_str(n int) RETURNS varchar(255)
BEGIN
	declare str varchar(100) default 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ';
	declare i int default 0;
	declare res_str varchar(255) default 'php_';

	while i < n do
		SET res_str = CONCAT(res_str, SUBSTR(str, FLOOR(1 + RAND()*52), 1));
		SET i = i + 1;
	end while;

	return res_str;
END$$
delimiter ;

-- 存储过程
CREATE DEFINER=`root`@`localhost` PROCEDURE `insert_emp`(in max_num int)
BEGIN
	DECLARE i INT DEFAULT 0;
	SET autocommit = 0;
	repeat
		set i = i + 1;
		insert into `testemployee` (name, dep_id, age, salary, cus_id) VALUES (rand_str(5), floor(1+rand()*10), floor(20 + RAND()*10), floor(2000 + RAND()*10), FLOOR(1 + RAND()*10));
		UNTIL i = max_num
	end repeat;

	commit;
END

-- 记得CALL，300W个数据先玩玩
CALL insert_emp(3000000)
```

## 常规max操作看看性能

```mysql
-- Time: 5.501s
SELECT max(age) FROM testemployee;
```

怀疑人生的时常吖，五秒多出现在服务器上简直就是一场灾难。这里我们发现 `age` 并没有索引对吧，那就干索引吧。

```mysql
-- 索引搞起来
CREATE INDEX idx_age ON testemployee(age);

-- Time: 0.024s
SELECT max(age) FROM testemployee;
```

## 总结

其实这个优化不是很难，主要的还是要自己去动手把东西都做一遍，代码都给出来，欢迎大家自己动手去写写。