---
title: Mybatis-入门（七）
date: 2020-04-08 21:26:18
categories:
  - Java
tags:
  - Java
  - Mybatis
---

## 前言

我们的数据库会被请求很多数据，但是如果数据是很多的时候，我们肯定是分批返回而不是全部返回。所以在实际开发中，我们会对数据进行分页。

## 分页

分页操作的本质就是`MySQL`中的 `limit` 关键字，之前我们检索数据的做法是：

```mysql
select * from users;
```

实际工作中，我们并不会这么用，因为数据库的数据随着时间的增加肯定是越来越多的，有可能是 1W、10W、100W，但是作为人我们没有办法一次性看这么多，而且性能上也是不允许。一般我们会使用 limit 来对其进行限制

```mysql
-- 这句话就是从头部开始十五条
select * from users limit 0, 15;
```

<!-- more -->

有了这个基础我们就知道如何写 mybatis 了。无非就是组装这个 sql 嘛。还是将就上一篇文章的项目

```java
package com.yubulang.dao;

import com.yubulang.pojo.User;

import java.util.List;
import java.util.Map;

public interface UserMapper {
    User getUserById(int id);

    List<User> getUserByPage(Map<String, Integer> map);
}
```

UserMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.yubulang.dao.UserMapper">
    <select id="getUserById" parameterType="int" resultType="User">
        select *
        from users
        where id = #{id};
    </select>

    <select id="getUserByPage" parameterType="map" resultType="User">
        select * from users limit #{offset}, #{pageSize}
    </select>
</mapper>
```

测试代码：

```java
@Test
public void testGetUserByPage() {
    SqlSession sqlSession = MybatisUtils.getSqlSession();

    UserMapper mapper = sqlSession.getMapper(UserMapper.class);

    Map<String, Integer> map = new HashMap<String, Integer>();
    map.put("offset", 0);
    map.put("pageSize", 2);

    List<User> userList = mapper.getUserByPage(map);

    for (User user : userList) {
        System.out.println(user);
    }

    sqlSession.close();
}
```

## 总结

其实分页还有 RowBounds 和分页插件，这个感兴趣的可以自行查询，其实本质都是不变的，就是 SQL 语句 Limit 的控制。
