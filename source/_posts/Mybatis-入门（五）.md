---
title: Mybatis-入门（五）
date: 2020-04-07 12:34:44
categories:
  - Java
tags:
  - Java
  - Mybatis
---

## 前言

我们在实际开发的时候，有没有遇到数据库字段和我们实体类中字段不一样的时候。这个时候 Mybatis 有没有提供解决方案呢？作为一个程序框架肯定是考虑到了这点。举个例子我们数据库表像下面这样。直接给出 SQL，在数据库中执行就行了，还是放在我们的 `mybatis` 数据库中：

```sql
CREATE TABLE users_test (
    `id` int(20) UNSIGNED NOT NULL AUTO_INCREMENT,
    `name` VARCHAR(64) NOT NULL,
    `pwd` VARCHAR(64) NOT NULL,
    PRIMARY KEY(`id`)
) ENGINE=INNODB DEFAULT CHARSET=utf8mb4;

INSERT INTO `users_test` (`name`, `pwd`) VALUES ('鱼不浪', '666666'), ('鲁班', '666666'), ('西施', '666666'), ('麦特凯', '666666');
```

而我们的实体类是 `com.yubulang.pojo.UserTest`，里面的 `password` 与 数据库表的 `pwd` 现在是没有办法对应匹配的。我们需要让实体类与他绑定。

```java
package com.yubulang.pojo;

public class UserTest {
    private int id;
    private String name;
    private String password;

    ...
}
```

为了复习之前的知识点，我们新建一个项目: `mybatis-demo`，一路下一步。哈哈哈

<!-- more -->

## 连接数据库配置

有了前面章节配置的学习，我们知道要在项目 `resources` 创建对应 `db.properties`, `mybatis-config.xml` 配置。

db.properties 如下：

```
driver=com.mysql.jdbc.Driver
url=jdbc:mysql://localhost:3306/mybatis?useSSL=true&useUnicode=true&characterEncoding=utf-8&serverTimezone=Hongkong
username=root
password=
```

创建完对应的属性，我们需要创建对应的 `mybatis-config.xml` 配置，`${字段名}` 就是我们 `db.properties` 对应的字段信息：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!-- 引入数据库相关属性 -->
    <properties resource="db.properties"/>

    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="${driver}"/>
                <property name="url" value="${url}"/>
                <property name="username" value="${username}"/>
                <property name="password" value="${password}"/>
            </dataSource>
        </environment>
    </environments>
</configuration>
```

有了这些基础配置，为了方便我们去使用 Mybatis，我们还需要写一个对应的 `com.yubulang.utils.MybatisUtils` 工具类：

```java
package com.yubulang.utils;

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;

import java.io.IOException;
import java.io.InputStream;

public class MybatisUtils {
    private static SqlSessionFactory sqlSessionFactory;

    static {
        String resource = "mybatis-config.xml";

        try {
            InputStream in = Resources.getResourceAsStream(resource);
            sqlSessionFactory = new SqlSessionFactoryBuilder().build(in);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public static SqlSession getSqlSession() {
        return sqlSessionFactory.openSession();
    }
}
```

在这里我们已经可以连接我们的数据库了，在连接完数据库，我们需要创建我们的实体类 `com.yubulang.pojo.UserTest`:

```java
package com.yubulang.pojo;

public class UserTest {
    private int id;
    private String name;
    private String password;

    public UserTest() {
    }

    public UserTest(int id, String name, String password) {
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

这一步完成后，我们已经有了实体类，这个时候就是创建对应的 Mapper 了，我们创建 `com.yubulang.dao.UserTestMapper` 这个实现接口类：

```java
package com.yubulang.dao;

import com.yubulang.pojo.UserTest;

import java.util.List;

public interface UserTestMapper {
    List<UserTest> getUserTestList();
}
```

有了对应的实现接口，我们也要创建对应的 `com.yubulang.dao.UserTestMapper.xml` 让我们进行对其进行配置：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<!-- 注意这里的 namespace 是我们要隐射的实现接口类 -->
<mapper namespace="com.yubulang.dao.UserTestMapper">
    <!-- resultType 返回值类型，这里我们简单的，一会儿去 mybatis-config.xml 配置 -->
    <select id="getUserTestList" resultType="UserTest">
        select * from users_test;
    </select>
</mapper>
```

有了这个隐射文件，我们需要将其对应的路径引入到我们的 `mybatis-config.xml` 中去，当然还要配置对应的实体类别名：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <properties resource="db.properties"/>

    <typeAliases>
        <typeAlias type="com.yubulang.pojo.UserTest" alias="UserTest"/>
    </typeAliases>

    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="${driver}"/>
                <property name="url" value="${url}"/>
                <property name="username" value="${username}"/>
                <property name="password" value="${password}"/>
            </dataSource>
        </environment>
    </environments>

    <mappers>
        <mapper resource="com/yubulang/dao/UserTestMapper.xml"/>
    </mappers>
</configuration>
```

实现代码这里已经是 Ok 的了。我们需要在 `test/java` 目录下创建一个 `com.yubulang.dao.UserTestMapperTest` 的测试文件来测试我们的代码：

```java
package com.yubulang.dao;

import com.yubulang.pojo.UserTest;
import com.yubulang.utils.MybatisUtils;
import org.apache.ibatis.session.SqlSession;
import org.junit.Test;

import java.util.List;

public class UserTestMapperTest {
    @Test
    public void testGetUserTestList() {
        SqlSession sqlSession = MybatisUtils.getSqlSession();

        UserTestMapper mapper = sqlSession.getMapper(UserTestMapper.class);

        List<UserTest> userTestList = mapper.getUserTestList();
        for (UserTest userTest : userTestList) {
            System.out.println(userTest);
        }

        sqlSession.close();
    }
}
```

运行测试，我们可以发现运行结果 `password` 是 `null` 而 `name` 正常输出。

```
User{id=1, name='鱼不浪', password='null'}
User{id=2, name='鲁班', password='null'}
User{id=3, name='西施', password='null'}
User{id=4, name='麦特凯', password='null'}
```

到这里，我们才是要开始解决我们的问题。其实解决这个问题的方法也简单，我们可以直接修改 `com.yubulang.dao.UserTestMapper.xml` 的 select sql 语句，改成下面这样：

```
select id, name, pwd as password from users_test;
```

这样一样能达成我们的效果，但是这显然有点笨。我们来看看科学的方法。

## resultMap 结果集映射

从字面表达来说，结果集映射故名思意就是把结果集中的字段与我们的实体类字段隐射关联一下。我们还是对 `com.yubulang.dao.UserTestMapper.xml` 进行修改。

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.yubulang.dao.UserTestMapper">
    <!-- 结果集隐射 -->
    <resultMap id="UserTestMap" type="UserTest">
        <!-- 让对应的列名，对应我们的实体类属性 -->
        <result column="id" property="id"/>
        <result column="name" property="name"/>
        <result column="pwd" property="password"/>
    </resultMap>

    <!-- 原来的返回类型resultType 替换成为 resultMap -->
    <select id="getUserTestList" resultMap="UserTestMap">
        select * from users_test;
    </select>
</mapper>
```

- resultMap 元素是 MyBatis 中最重要最强大的元素
- ResultMap 的设计思想是，对简单的语句做到零配置，对于复杂一点的语句，只需要描述语句之间的关系就行了
- ResultMap 的优秀之处——你完全可以不用显式地配置它们，可以把有需要隐射的字段来隐射

所以，我们这里只需要保留下不一样的需要隐射的字段就好了：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.yubulang.dao.UserTestMapper">
    <!-- 结果集隐射 -->
    <resultMap id="UserTestMap" type="UserTest">
        <!-- 让对应的列名，对应我们的实体类属性 -->
        <!-- <result column="id" property="id"/> -->
        <!-- <result column="name" property="name"/> -->
        <result column="pwd" property="password"/>
    </resultMap>

    <!-- 原来的返回类型resultType 替换成为 resultMap -->
    <select id="getUserTestList" resultMap="UserTestMap">
        select * from users_test;
    </select>
</mapper>
```

## 总结

这里只是开胃菜，我们在项目中会有很多更加复杂的形式，一点点深入我们会接触到。就如文档中说的那样，世界如果总是这么简单就好了。不过还是劝解大家醒醒哈哈哈
