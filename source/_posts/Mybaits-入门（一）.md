---
title: Mybaits-入门（一）
date: 2020-04-05 16:13:37
categories:
  - Java
tags:
  - Java
  - Mybatis
---

## 前言

学完 JDBC 后，其实很多时候我们项目开发中不会直接使用 JDBC，而是会用到一些更好的第三方封装库，Mybatis 就是第三方库中使用人数最多，口碑最好的一个存在。为了快速上手，当然是盘它吖。

## 搭建 MySQL 数据库相关的表环境

- 搭建数据库，并创建数据

```mysql
-- 创建数据库
CREATE DATABASE IF NOT EXISTS `mybatis` CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- 使用数据库
USE `mybatis`;

-- 创建users表
CREATE TABLE `users` (
  `id` int(20) unsigned NOT NULL AUTO_INCREMENT,
  `name` varchar(64) NOT NULL,
  `password` varchar(64) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 插入数据库
INSERT INTO `users` (`name`, `password`) VALUES ('鱼不浪', '666666');
INSERT INTO `users` (`name`, `password`) VALUES ('鲁班', '666666');
INSERT INTO `users` (`name`, `password`) VALUES ('西施', '666666');
INSERT INTO `users` (`name`, `password`) VALUES ('麦特凯', '666666');
```

<!-- more -->

- 新建普通的 maven 项目为`mybatis-study`，删除 `src` 目录，使他成为一个父项目
- 在父项目的 `pom.xml` 中放入以下依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>mybatis-study</artifactId>
    <version>1.0-SNAPSHOT</version>

    <!-- 导入依赖 -->
    <dependencies>
        <!-- mysql 驱动 -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.48</version>
        </dependency>

        <!-- mybatis -->
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.5.4</version>
        </dependency>

        <!-- junit -->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.13</version>
        </dependency>
    </dependencies>
</project>
```

- 创建名为 `mybatis-01` 的子项目，并进入`src/main/resources` 目录，创建一个`mybatis-config.xml`的文件，并输入以下配置

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<!-- 核心配置文件 -->
<configuration>
    <!-- 多个开发环境配置 -->
    <environments default="development">
        <environment id="development">
            <!-- 事务管理 -->
            <transactionManager type="JDBC"/>

            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url"
                          value="jdbc:mysql://localhost:3306/mybatis?useSSL=true&amp;useUnicode=true&amp;characterEncoding=utf8&amp;serverTimezone=Hongkong"/>
                <property name="username" value="root"/>
                <property name="password" value=""/>
            </dataSource>
        </environment>
    </environments>
</configuration>
```

- 编写 Mybatis 工具类

  创建`com.yubulang.utils`包，在里面创建`MybatisUtils.java`工具类，然后输入以下代码

```java
package com.yubulang.utils;

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;

import java.io.IOException;
import java.io.InputStream;

// sqlSessionFactory 用来构造 sqlSession
public class MybatisUtils {

    private static SqlSessionFactory sqlSessionFactory;

    static {
        // 这里是固定搭配写一次就OJBK了
        try {
            // 使用mybatis第一步：获取我们的 sqlSessionFactory 对象
            String resource = "mybatis-config.xml";
            InputStream in = Resources.getResourceAsStream(resource);

            sqlSessionFactory = new SqlSessionFactoryBuilder().build(in);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    /**
     * 既然有了sqlSessionFactory，我们可以通过它获取一个 SqlSession
     *
     * @return SqlSession
     */
    public static SqlSession getSqlSession() {
        return sqlSessionFactory.openSession();
    }
}
```

## 编写代码操作数据库

- 实体类：我们创建 `com.yubulang.pojo.User.java` 这个实体类

```java
package com.yubulang.pojo;

/**
 * 实体类
 */
public class User {
    private int id;
    private String name;
    private String password;

    public User() {
    }

    public User(int id, String name, String password) {
        this.id = id;
        this.name = name;
        this.password = password;
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", password='" + password + '\'' +
                '}';
    }
}
```

- Dao/Mapper 接口（其实可以理解成为 Dao 层）

```java
package com.yubulang.dao;

import com.yubulang.pojo.User;

import java.util.List;

public interface UserMapper {
    List<User> getUserList();
}
```

- 接口实现类，我们在 `Mybatis` 这里，由原来的`UserDaoImpl.java`直接用`xml`文件代替，`com.yubulang.dao`包下创建`UserMapper.xml`

```java
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

## 测试上面的代码

我们使用`junit`测试，在 maven 项目中 `main` 同级的有一个 `test` 文件夹，我们尽量让我们的测试文件，与我们的 `main` 包中的结构相同。

我们键入以下测试代码：

```java
package com.yubulang.dao;

import com.yubulang.pojo.User;
import com.yubulang.utils.MybatisUtils;
import org.apache.ibatis.session.SqlSession;
import org.junit.Test;

import java.util.List;

public class UserMapperTest {
    @Test
    public void test() {
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
}
```

注意这里的报错信息：

```
org.apache.ibatis.binding.BindingException: Type interface com.yubulang.dao.UserMapper is not known to the MapperRegistry.
```

在这里我们会接触到一个新的概念就是`MapperRegistry`。这是因为我们在当初配置`mybatis-config.xml`的时候没有配置对应的\<mappers\>，我们将对应的 mapper 配置加入到`mybatis-config.xml`中，然后启动测试`test()`。

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<!-- 核心配置文件 -->
<configuration>
    <!-- 多个开发环境配置 -->
    <environments default="development">
        <environment id="development">
            <!-- 事务管理 -->
            <transactionManager type="JDBC"/>

            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url"
                          value="jdbc:mysql://localhost:3306/mybatis?useSSL=true&amp;useUnicode=true&amp;characterEncoding=utf8&amp;serverTimezone=Hongkong"/>
                <property name="username" value="root"/>
                <property name="password" value=""/>
            </dataSource>
        </environment>
    </environments>

    <!-- 每一个Mapper.xml都需要在mybatis核心配置文件中注册-->
    <mappers>
        <mapper resource="com/yubulang/dao/UserMapper.xml"/>
    </mappers>
</configuration>
```

启动运行，我们则发现我擦，报错了。说没有办法找到我们的`Could not find resource com/yubulang/dao/UserMapper.xml`这个文件，这是因为 maven 的过滤机制导致的，我们可以修改`pom.xml`配置，让他支持我们的 xml 文件可以被编译到我们的 class 中去。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>mybatis-study</artifactId>
        <groupId>org.example</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>mybatis-01</artifactId>

    <!-- 在build中配置resource，来防止我们资源导出失败的问题 -->
    <build>
        <resources>
            <resource>
                <directory>src/main/resources</directory>
                <includes>
                    <include>**/*.properties</include>
                    <include>**/*.xml</include>
                </includes>
                <filtering>true</filtering>
            </resource>

            <resource>
                <directory>src/main/java</directory>
                <includes>
                    <include>**/*.properties</include>
                    <include>**/*.xml</include>
                </includes>
                <filtering>true</filtering>
            </resource>
        </resources>
    </build>
</project>
```

再次点击运行测试，就能正常跑起来了。

## 总结

有了 Mybatis 后，我们的数据查询操作基本就在`xml`实现，我们单独维护起来也更加方便。可以说提高了我们不少的生产力。当然这里只是入门的操作，我们更多的操作还要去实践应用。一定要多动手，不要怕出错！
