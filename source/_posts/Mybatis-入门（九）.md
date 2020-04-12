---
title: Mybatis-入门（九）
date: 2020-04-09 21:47:12
categories:
  - Java
tags:
  - Java
  - Mybatis
---

## 前言

我们在实际开发中是不仅仅针对一张表，而是两张或者多张表联动的操作，这里我们经常面对的关系就是一对一，一对多，多对一，多对多等等。

这里我们搭建环境，我们看看最常见的数据关系，学生和老师的关系

- 对于老师来说，一个老师拥有多个学生
- 对于学生而言，多个学生都有一个老师

```mysql
create table teachers (
    `id` int(20) not null auto_increment,
    `name` varchar(64) not null,
    PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

insert into teachers (`name`) values ('燕子李三');

create table students (
    `id` int(20) not null auto_increment,
    `name` varchar(64) not null,
    `tid` int(20) not null,
    PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

insert into students (`name`, `tid`) values ('张三', 1), ('李四', 1), ('王五', 1), ('赵六', 1);
```

我们导入数据库，然后开始创建项目。

<!-- more -->

## 搭建新项目

到目前为之，我觉得学习就是一个重复的过程，不停的创建新的实验性项目，当然我心里也是很烦的。但是没有办法，发现只有动手之后我才能知道到底哪里有问题，我才能有针对性的记忆。这里我们建立一个 `mybatis-06` 创建好之后，我们在需要在 `pom.xml` 的 `project` 标签中加入以下代码：

### 第一步，`pom.xml` 加入相应的配置

```xml
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
```

这个配置主要是为了相关的 XML 文件在编译之后，能被打包到对应的位置。

### 第二步：创建 `db.properties` `mybatis-config.xml` 配置类

db.properties 配置如下：

```properties
driver=com.mysql.jdbc.Driver
url=jdbc:mysql://localhost:3306/mybatis?useSSL=true&useUnicode=true&characterEncoding=utf-8&serverTimezone=Hongkong
username=root
password=
```

mybatis-config.xml 配置如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">

<configuration>
    <properties resource="db.properties"/>

    <settings>
        <setting name="logImpl" value="STDOUT_LOGGING"/>
    </settings>

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

### 第三步：创建 MybatisUtils 工具类

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
        try {
            String resource = "mybatis-config.xml";
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

到这里我们已经有了一个基础的架子。

## 一对多

对于老师而言，一个老师负责多个学生，我们来创建两个实体类：

### 创建实体类

`com.yubulang.pojo.Teacher.java`：

```java
package com.yubulang.pojo;

public class Teacher {
    private int id;
    private String name;

    public Teacher() {
    }

    public Teacher(int id, String name) {
        this.id = id;
        this.name = name;
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

    @Override
    public String toString() {
        return "Teacher{" +
                "id=" + id +
                ", name='" + name + '\'' +
                '}';
    }
}
```

`com.yubulang.pojo.Student`，注意这里我们要关联 Teacher，所以在 Student 的属性中 teacher 应该对应 Teacher 类：

```java
package com.yubulang.pojo;

public class Student {
    private int id;
    private String name;
    private Teacher teacher;

    public Student() {
    }

    public Student(int id, String name, Teacher teacher) {
        this.id = id;
        this.name = name;
        this.teacher = teacher;
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

    public Teacher getTeacher() {
        return teacher;
    }

    public void setTeacher(Teacher teacher) {
        this.teacher = teacher;
    }

    @Override
    public String toString() {
        return "Student{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", teacher=" + teacher +
                '}';
    }
}
```

### 创建实现接口

`com.yubulang.dao.StudentMapper.java`, 因为关联方式有两种:

```java
package com.yubulang.dao;

import com.yubulang.pojo.Student;
import com.yubulang.pojo.Teacher;

import java.util.List;

public interface StudentMapper {

    // 第一种方式 根据结果描述隐射关系
    List<Student> getStudent();

    // 第二种方式
    List<Student> getStudentList();
}
```

有了实现接口，我们需要对应的 xml 与之匹配它的实现。我们把 xml 放到 `resources/com/yubulang/dao` 中，命名为 `StudentMapper.xml`：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.yubulang.dao.StudentMapper">
    <!-- 第一种方法，编写关联关系去查询 -->
    <select id="getStudent" resultMap="StudentTeacher">
        select * from students;
    </select>

    <resultMap id="StudentTeacher" type="Student">
        <association property="teacher" column="tid" javaType="Teacher" select="getTeacherByTid"/>
    </resultMap>

    <select id="getTeacherByTid" resultType="Teacher">
        select * from teachers where id=#{id}
    </select>

    <!-- 第二种方法（相对简单）：根据查询结果来关联 -->
    <select id="getStudentList" resultMap="StudentTeacher2">
        select s.id sid, s.name sname, t.id tid, t.name tname from students s, teachers t where s.tid = t.id;
    </select>

    <resultMap id="StudentTeacher2" type="Student">
        <result property="id" column="sid"/>
        <result property="name" column="sname"/>
        <association property="teacher" javaType="Teacher">
            <result property="id" column="tid"/>
            <result property="name" column="tname"/>
        </association>
    </resultMap>
</mapper>
```

### 测试类的干活

```java
package com.yubulang.dao;

import com.yubulang.pojo.Student;
import com.yubulang.utils.MybatisUtils;
import org.apache.ibatis.session.SqlSession;
import org.junit.Test;

import java.util.List;

public class StudentMapperTest {
    @Test
    public void testGetStudent() {
        SqlSession sqlSession = MybatisUtils.getSqlSession();
        StudentMapper mapper = sqlSession.getMapper(StudentMapper.class);
        List<Student> studentList = mapper.getStudent();
        for (Student student : studentList) {
            System.out.println(student);
        }

        sqlSession.close();
    }

    @Test
    public void testGetStudentList() {
        SqlSession sqlSession = MybatisUtils.getSqlSession();
        StudentMapper mapper = sqlSession.getMapper(StudentMapper.class);
        List<Student> studentList = mapper.getStudentList();
        for (Student student : studentList) {
            System.out.println(student);
        }

        sqlSession.close();
    }
}
```

## 多对一

对于学生而言，多个学生跟着一个老师，我们来创建两个实体类：

### 创建实体类

`com.yubulang.pojo.Teacher`:

```java
package com.yubulang.pojo;

import java.util.List;

public class Teacher {
    private int id;
    private String name;

    // 一个老师有多个学生
    private List<Student> students;

    public Teacher() {
    }

    public Teacher(int id, String name, List<Student> students) {
        this.id = id;
        this.name = name;
        this.students = students;
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

    public List<Student> getStudents() {
        return students;
    }

    public void setStudents(List<Student> students) {
        this.students = students;
    }

    @Override
    public String toString() {
        return "Teacher{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", students=" + students +
                '}';
    }
}
```

`com.yubulang.pojo.Student`:

```java
package com.yubulang.pojo;

public class Student {
    private int id;
    private String name;
    private int tid;

    public Student() {
    }

    public Student(int id, String name, int tid) {
        this.id = id;
        this.name = name;
        this.tid = tid;
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

    public int getTid() {
        return tid;
    }

    public void setTid(int tid) {
        this.tid = tid;
    }

    @Override
    public String toString() {
        return "Student{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", tid=" + tid +
                '}';
    }
}
```

### 创建实现接口

`com.yubulang.dao.TeacherMapper`:

```java
package com.yubulang.dao;

import com.yubulang.pojo.Teacher;
import org.apache.ibatis.annotations.Param;

public interface TeacherMapper {
    Teacher getTeacherById(@Param("tid") int id);

    Teacher getTeacherByIdAnother(@Param("tid") int id);
}
```

有了实现接口，我们需要对应的 xml 与之匹配它的实现。我们把 xml 放到 `resources/com/yubulang/dao` 中，命名为 `TeacherMapper.xml`：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.yubulang.dao.TeacherMapper">
    <!--第一种方式（复杂度在sql）-->
    <select id="getTeacherById" resultMap="TeacherStudent">
        select t.id tid, t.name tname, s.id sid, s.name sname
        from teachers t,
             students s
        where t.id = s.tid
          and t.id = #{tid};
    </select>

    <resultMap id="TeacherStudent" type="Teacher">
        <result property="id" column="tid"/>
        <result property="name" column="tname"/>
        <collection property="students" ofType="Student">
            <result property="id" column="sid"/>
            <result property="name" column="sname"/>
            <result property="tid" column="tid"/>
        </collection>
    </resultMap>

    <!--第二种方式复杂度在凭借-->
    <select id="getTeacherByIdAnother" resultMap="TeacherStudentAnother">
        select id, name
        from teachers
        where id = #{tid}
    </select>

    <resultMap id="TeacherStudentAnother" type="Teacher">
        <result property="id" column="id"/>
        <collection property="students" javaType="ArrayList" ofType="Student" select="getStudentByTid" column="id">
            <result property="id" column="id"/>
            <result property="name" column="name"/>
            <result property="tid" column="tid"/>
        </collection>
    </resultMap>

    <select id="getStudentByTid" resultType="Student">
        select *
        from students
        where tid = #{tid}
    </select>
</mapper>
```

### 测试代码的干货

```java
package com.yubulang.dao;

import com.yubulang.pojo.Teacher;
import com.yubulang.utils.MybatisUtils;
import org.apache.ibatis.session.SqlSession;
import org.junit.Test;

public class TeacherMapperTest {
    @Test
    public void testGetTeacherById() {
        SqlSession sqlSession = MybatisUtils.getSqlSession();

        TeacherMapper mapper = sqlSession.getMapper(TeacherMapper.class);
        Teacher teacherById = mapper.getTeacherById(1);

        System.out.println(teacherById);

        sqlSession.close();
    }

    @Test
    public void testGetTeacherByIdAnother() {
        SqlSession sqlSession = MybatisUtils.getSqlSession();

        TeacherMapper mapper = sqlSession.getMapper(TeacherMapper.class);
        Teacher teacherById = mapper.getTeacherByIdAnother(1);

        System.out.println(teacherById);

        sqlSession.close();
    }
}
```

## 总结

比之前更发杂了点，但是这些关系是我们做项目时常用的一些数组组成方式。有个基础，后面复杂的都不在话下。
