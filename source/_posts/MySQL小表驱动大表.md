---
title: MySQL小表驱动大表
date: 2020-03-08 21:02:38
categories:
  - MySQL
tags:
  - MySQL
---
## 前言

我们查询小表的数据肯定是比查询大表的数据来的快，所以我们可以用查询小表得到的结果集，作为查询条件去查询大表，这样提前的一个结果范围，能够让我们更快的获取数据。

<!-- more -->

## 准备基础数据

我们准备一个部门表，在实际中一个公司的部门肯定是比人员少，如果部门比人员还多的公司，那么请大家看清楚再入坑，不然会有很多意想不到的惊喜。

```mysql
-- 创建部门表
-- 生产环境不要用DROP! 生产环境不要用DROP! 生产环境不要用DROP!
DROP TABLE IF EXISTS `department`; 
CREATE TABLE IF NOT EXISTS `department` (
    `id` int(11) NOT NULL AUTO_INCREMENT,
    `dept_name` varchar(200) DEFAULT NULL,
    `address` varchar(200) DEFAULT NULL,
    PRIMARY KEY(`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 插入测试数据
INSERT INTO `department` (`dept_name`, `address`) VALUES ('研发部(RD)', '2层');
INSERT INTO `department` (`dept_name`, `address`) VALUES ('人事部(HR)', '3层');
INSERT INTO `department` (`dept_name`, `address`) VALUES ('市场部(MK)', '4层');
INSERT INTO `department` (`dept_name`, `address`) VALUES ('后勤部(MIS)', '5层');
INSERT INTO `department` (`dept_name`, `address`) VALUES ('财务部(FD)', '6层');

-- 创建雇员表
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

-- 随机字符串函数
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

-- 存储过程
DROP PROCEDURE IF EXISTS `insert_emp`;
CREATE PROCEDURE `insert_emp`(in max_num int)
BEGIN
	DECLARE i INT DEFAULT 0;
	SET autocommit = 0;
	repeat
		set i = i + 1;
		insert into `testemployee` (name, dep_id, age, salary, cus_id) VALUES (rand_str(5), floor(rand()*(6-1) + 1), floor(20 + RAND()*10), floor(2000 + RAND()*10), FLOOR(1 + RAND()*10));
		UNTIL i = max_num
	end repeat;
	
	commit;
END$$

delimiter ;

-- 创建数据
CALL insert_emp(100000);
```



## 小表驱动大表

### 例子

我们先用直接操作，查询出 `testemployee` 的数据研发部和人事部的信息。

```mysql
-- Time: 0.5s
SELECT * FROM `testemployee` WHERE dep_id = 2 OR dep_id = 3;
```

这里因为使用了 `OR` 即使是创建了索引，也会导致索引失效，全表查询。速度相对而言比较慢。

既然使用到了 `OR` 去做查询，`testemployee` 表全表查询肯定是比 `department` 表全表查询慢，所以我们可以使用小的数据表查询结果驱动大的数据集。

```mysql
-- Time: 0.033s
SELECT * FROM `testemployee` WHERE dep_id IN (SELECT id FROM department WHERE dept_name='研发部(RD)' OR dept_name='人事部(HR)');
```



### in与exists

其实两个可以互相替换，但是思想却大不相同。上面的语句我们看看如何改写

```mysql
SELECT * FROM `testemployee` WHERE EXISTS (SELECT 2 FROM department WHERE dept_name='研发部(RD)' OR dept_name='人事部(HR)');
```

他和IN的区别在于，使用大表来驱动小表，子语句中是小表，至于什么时候使用，不言而喻了吧。



## 总结

这里的数据库表的例子可能不太恰当，但是主要是想说明一个观点。就是小表得到结果的数据肯定比大表快，通过小表我们缩小了确定数据的范围，而不用让大表逐条去比对数据。当然我们该做索引的地方还是要做索引。这个跑不掉。
