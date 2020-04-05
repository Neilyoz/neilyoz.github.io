---
title: Mybaits-入门（二）
date: 2020-04-05 23:13:37
categories:
  - Java
tags:
  - Java
  - Mybatis
---

## 前言

有了第一篇{% post_link "Mybaits-入门（一）" "《Mybaits-入门（一）》" %}的铺垫，我们已经有一个基本的程序骨架了，虽然只是简单的查询，但是我们可以大概了解到 `Mybatis` 的相关操作了。接下来咱们一点一点的来，慢慢了解 Mybatis 的`增`、`删`、`改`、`查`操作。

## CRUD

我们把上一篇文章中`com/yubulang/dao/UserMapper.xml`的配置贴出来，看看相关的参数

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<!-- 绑定一个指定的Dao/Mapper接口 -->
<mapper namespace="com.yubulang.dao.UserMapper">
    <!-- 查询语句, id对应UserMapper接口中的方法名 -->
    <!-- resultType 返回类型：要填写全类型名 -->
    <select id="getUserList" resultType="com.yubulang.pojo.User">
        select * from users
    </select>
</mapper>
```

<!-- more -->

### namespace

我们需要注意的是 `namespace` 中的包名和 `Dao/Mapper` 接口的包名一致。

### select：选择查询语句

- id 就是对应的 namespace 中的方法名
- resultType：Sql 语句执行的返回值
- parameterType：参数类型

刚才我们已经查询全部用户的方法，现在我们实现一个查询指定用户的 id 的用户消息。还是老规矩，我们先实现接口实现类。

```java
package com.yubulang.dao;

import com.yubulang.pojo.User;

import java.util.List;

public interface UserMapper {
    // 获取全部用户
    List<User> getUserList();

    // 根据id获取用户
    User getUserById(int id);
}
```

在接口实现类中添加了`getUserById`方法，同时我们也要在`UserMapper.xml`下面添加对应的 `<select>` 标签，这个标签需要我们传递参数进入 `parameterType` 参数类型。在写 sql 语句时，我们要用 `#{id}` 来做参数占位符。

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<!-- 绑定一个指定的Dao/Mapper接口 -->
<mapper namespace="com.yubulang.dao.UserMapper">
    <!-- 查询语句, id对应UserMapper接口中的方法名 -->

    <!-- resultType 返回类型：要填写全类型名 -->
    <select id="getUserList" resultType="com.yubulang.pojo.User">
        select * from users
    </select>

    <!-- 查询指定用户id的信息 -->
    <select id="getUserById" resultType="com.yubulang.pojo.User" parameterType="int">
        select * from users where id=#{id}
    </select>
</mapper>
```

写完之后，测试代码也要跟上：

```java
package com.yubulang.dao;

import com.yubulang.pojo.User;
import com.yubulang.utils.MybatisUtils;
import org.apache.ibatis.session.SqlSession;
import org.junit.Test;

import java.util.List;

public class UserMapperTest {
    @Test
    public void testGetUserList() {
        // 第一步：获得SqlSession对象
        SqlSession sqlSession = MybatisUtils.getSqlSession();

        // 执行SQL
        // 方式一：getMapper
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        List<User> userList = mapper.getUserList();
        for (User user : userList) {
            System.out.println(user);
        }

        System.out.println("============================================");

        // 方式二：旧的一种方式不太推荐
        List<User> userList2 = sqlSession.selectList("com.yubulang.dao.UserMapper.getUserList");
        for (User user : userList2) {
            System.out.println(user);
        }

        // 关闭SqlSession
        sqlSession.close();
    }

    @Test
    public void testGetUserById() {
        SqlSession sqlSession = MybatisUtils.getSqlSession();

        UserMapper mapper = sqlSession.getMapper(UserMapper.class);

        User user = mapper.getUserById(1);

        System.out.println(user);

        sqlSession.close();
    }
}
```

### insert： 插入操作

既然有查询，就肯定有插入的操作，其实方式都差不多，需要我们耐心的去重复，直到形成本能记忆。我们还是在 `com.yubulang.dao.UserMapper` 中创建一个 `addUser` 方法。

```java
package com.yubulang.dao;

import com.yubulang.pojo.User;

import java.util.List;

public interface UserMapper {
    // 获取全部用户
    List<User> getUserList();

    // 根据id获取用户
    User getUserById(int id);

    // 创建一个用户
    int addUser(User user);
}
```

`UserMapper.xml`也要加入对应的 \<insert\> 标签，这个 insert 标签，没有 returnType 标签，这里我们的 `parameterType` 是一个 User 类型，我们要怎么传值呢？Mybatis 允许我们直接使用类的字段属性做占位符。我们来看看 xml 文件如何编写。

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<!-- 绑定一个指定的Dao/Mapper接口 -->
<mapper namespace="com.yubulang.dao.UserMapper">
    <!-- 查询语句, id对应UserMapper接口中的方法名 -->

    <!-- resultType 返回类型：要填写全类型名 -->
    <select id="getUserList" resultType="com.yubulang.pojo.User">
        select * from users
    </select>

    <!-- 查询指定用户id的信息 -->
    <select id="getUserById" resultType="com.yubulang.pojo.User" parameterType="int">
        select * from users where id=#{id}
    </select>

    <!-- 创建一个用户 -->
    <!-- insert 没有returnType -->
    <!-- User 对象中的属性，可以直接取出 -->
    <insert id="addUser" parameterType="com.yubulang.pojo.User">
        insert into users (name, password) values (#{name}, #{password})
    </insert>
</mapper>
```

测试代码这个时候要跟上，在 `test/java` 目录 `com.yubulang.dao.UserMapperTest` 添加 `testAddUser` 测试方法：

```java
@Test
public void testAddUser() {
    SqlSession sqlSession = MybatisUtils.getSqlSession();

    UserMapper mapper = sqlSession.getMapper(UserMapper.class);

    User user = new User();
    user.setName("赵云");
    user.setPassword("666666");

    int result = mapper.addUser(user);
    if (result > 0) {
        System.out.println("插入成功！");
    }

    sqlSession.close();
}
```

我们执行，会发现这条数据并未插入到数据库，这是为什么呢？因为增删改都需要提交事务才能完成入库。

所以我们需要将代码调整一下：

```java
// 增删改查需要提交事务
@Test
public void testAddUser() {
    SqlSession sqlSession = MybatisUtils.getSqlSession();

    UserMapper mapper = sqlSession.getMapper(UserMapper.class);

    User user = new User();
    user.setName("赵云");
    user.setPassword("666666");

    int result = mapper.addUser(user);
    if (result > 0) {
        System.out.println("插入成功！");
    }

    // 提交事务
    sqlSession.commit();

    sqlSession.close();
}
```

### update：修改操作

有了添加操作的铺垫，我相信 update 的操作也就是依葫芦画瓢的事儿，我还是继续将代码敲一遍。接口实现类代码如下：

```java
package com.yubulang.dao;

import com.yubulang.pojo.User;

import java.util.List;

public interface UserMapper {
    // 获取全部用户
    List<User> getUserList();

    // 根据id获取用户
    User getUserById(int id);

    // 创建一个用户
    int addUser(User user);

    // 修改一个用户
    int updateUser(User user);
}
```

`UserMapper.xml`也要加入对应的 \<update\> 标签。

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<!-- 绑定一个指定的Dao/Mapper接口 -->
<mapper namespace="com.yubulang.dao.UserMapper">
    <!-- 查询语句, id对应UserMapper接口中的方法名 -->

    <!-- resultType 返回类型：要填写全类型名 -->
    <select id="getUserList" resultType="com.yubulang.pojo.User">
        select * from users
    </select>

    <!-- 查询指定用户id的信息 -->
    <select id="getUserById" resultType="com.yubulang.pojo.User" parameterType="int">
        select * from users where id=#{id}
    </select>

    <!-- 创建一个用户 -->
    <!-- insert 没有returnType -->
    <!-- User 对象中的属性，可以直接取出 -->
    <insert id="addUser" parameterType="com.yubulang.pojo.User">
        insert into users (name, password) values (#{name}, #{password})
    </insert>

    <!-- 修改一个用户 -->
    <update id="updateUser" parameterType="com.yubulang.pojo.User">
        update users set name=#{name}, password=#{password} where id=#{id}
    </update>
</mapper>
```

测试代码走起，在 `test/java` 目录 `com.yubulang.dao.UserMapperTest` 添加 `testUpdateUser` 测试方法：

```java
@Test
public void testUpdateUser() {
    SqlSession sqlSession = MybatisUtils.getSqlSession();

    UserMapper mapper = sqlSession.getMapper(UserMapper.class);

    User user = mapper.getUserById(3);
    user.setName(user.getName() + "修改");

    int result = mapper.updateUser(user);
    if (result > 0) {
        System.out.println("修改成功");
    }

    sqlSession.commit();
    sqlSession.close();
}
```

### delete：删除操作

删除用户的操作接口实现类：

```java
package com.yubulang.dao;

import com.yubulang.pojo.User;

import java.util.List;

public interface UserMapper {
    // 获取全部用户
    List<User> getUserList();

    // 根据id获取用户
    User getUserById(int id);

    // 创建一个用户
    int addUser(User user);

    // 修改一个用户
    int updateUser(User user);

    // 删除一个用户
    int deleteUser(int id);
}
```

`UserMapper.xml`也要加入对应的 \<delete\> 标签。

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<!-- 绑定一个指定的Dao/Mapper接口 -->
<mapper namespace="com.yubulang.dao.UserMapper">
    <!-- 查询语句, id对应UserMapper接口中的方法名 -->

    <!-- resultType 返回类型：要填写全类型名 -->
    <select id="getUserList" resultType="com.yubulang.pojo.User">
        select * from users
    </select>

    <!-- 查询指定用户id的信息 -->
    <select id="getUserById" resultType="com.yubulang.pojo.User" parameterType="int">
        select * from users where id=#{id}
    </select>

    <!-- 创建一个用户 -->
    <!-- insert 没有returnType -->
    <!-- User 对象中的属性，可以直接取出 -->
    <insert id="addUser" parameterType="com.yubulang.pojo.User">
        insert into users (name, password) values (#{name}, #{password})
    </insert>

    <!-- 修改一个用户 -->
    <update id="updateUser" parameterType="com.yubulang.pojo.User">
        update users set name=#{name}, password=#{password} where id=#{id}
    </update>

    <!-- 删除一个用户 -->
    <delete id="deleteUser" parameterType="int">
        delete from users where id=#{id}
    </delete>
</mapper>
```

测试代码走起，在 `test/java` 目录 `com.yubulang.dao.UserMapperTest` 添加 `testDeleteUser` 测试方法：

```java
@Test
public void testDeleteUser() {
    SqlSession sqlSession = MybatisUtils.getSqlSession();

    UserMapper mapper = sqlSession.getMapper(UserMapper.class);

    int result = mapper.deleteUser(4);
    if (result > 0) {
        System.out.println("删除成功");
    }

    sqlSession.commit();
    sqlSession.close();
}
```

## 总结

基本的操作不是很难，不过复杂的还需要我们看看文档，才能知道一些具体的使用场景，加油！动手！动手！动手！
