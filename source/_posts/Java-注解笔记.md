---
title: Java 注解笔记
date: 2020-04-04 00:19:06
categories:
  - Java
tags:
  - Java
---

## 前言

之前我常用的语言是 PHP，受限于语言特性，说实话 PHP 被很多公司边缘化了，感觉后面发展空间和路子并不是很大，加上很多解决方案还是 Java 的比较多，且资料更加健全，我一头扎入了 Java 的学习中。

注解和反射在我们在现在的框架中常常看到，查看框架的底层也会发现反射的身影，如果我们不了解这些通用的方法，显然有点说不过去。边学边记录，记录自己的学习路线。

## 注解

### 基本概念

- Annotation 是从 JDK5.0 开始引入的新技术
- 作用：
  - 不是程序本身，可以对程序做出解释（这一点和注释没有什么区别）
  - 可以被其他程序（比如：编译器等）读取
- Annotation 的格式：
  - 注解以“@注解名”在代码中存在，还可以添加一些参数值，例如：@SuppressWarnings(value="unchecked")
- Annotation 在哪里使用？
  - 可以附加在 package、class、method、field 等上面，相当于给爱他们添加了额外的辅助信息，我们可以通过反射机制编程实现对这些元数据的访问。

### 内置注解

- `@Override`：定义在`java.lang.Override`中，此注解只适用于修饰方法，表示一个方法声明打算重写超类中的一个方法声明。
- `@Deprecated`：定义在`java.lang.Deprecated`中，此注解可用于修饰方法，属性，类，表示不鼓励大家使用这个元素，通常是因为它很危险或者有更好的选择。
- `@SuppressWarning`：定义在`java.lang.SuppressWarning`中，用来抑制编译时的警告信息
  - 与前面两个注解有所不同，你需要添加一个参数才能正确使用，这些参数都是已经定义好了的，我们选择性使用就好了。
    - @SuppressWarning("all")
    - @SuppressWarning("unchecked")
    - @SuppressWarning(value={"unchecked", "deprecation"})
    - 等等......

<!-- more -->

## 元注解

### 基本概念

- 元注解的作用就是负责注解其他注解，Java 定义了 4 个标准的 meta-annotation 类型，他们被用来提供对其他 annotation 类型作说明。
- 这些类型和它们所支持的类在`java.lang.annotation`包中找到。（@Target，@Retention，@Documented，@Inherited）
  - @Target：用于描述注解的使用范围（即：被描述的注解可以用在什么地方）
  - @Retention：表示需要在什么级别保存该注解信息，用于描述注解的声明周期（SOURCE < CLASS < RUNTIME）
  - @Documented：说明该注解将包含在 JavaDoc 中
  - @Inherited：说明子类可以继承父类中的该注解

### @Target 用于描述注解的作用范围

我们查看源代码可以知道，Target 可以传入一个值 `value` 这个 value 的类型为 `ElementType`

```java
public @interface Target {
    ElementType[] value();
}
```

让我们看看这个`ElementType`枚举类包含的值，表示注解作用在什么上面：

- **TYPE：作用在类、接口，包括注解、枚举**

  ```java
  // 我们先定义一个注解，文件为 MyAnnotationDemo01.java
  package com.yubulang.demo01;

  import java.lang.annotation.ElementType;
  import java.lang.annotation.Target;

  @Target(value = {ElementType.TYPE})
  public @interface MyAnnotationDemo01 {
  }
  ```

  然后我们定义这个注解可以使用的对象，我是用的 idea，如果注解不支持这个类型，注解是会有红线提示。

  ```java
  package com.yubulang.demo01;

  // 作用在类上
  @MyAnnotationDemo01
  public class AnnotationDemo {

  }

  // 作用在接口上
  @MyAnnotationDemo01
  interface TestInterface {

  }

  // 作用在注解上
  @MyAnnotationDemo01
  @interface TestAnnotation {

  }

  // 作用在枚举类上
  @MyAnnotationDemo01
  enum TestEnum {
      HELLO,
      WORLD,
  }
  ```

* **FIELD：作用在属性字段，包括枚举和常量**

  ```java
  // 定义一个注解
  package com.yubulang.demo02;

  import java.lang.annotation.ElementType;
  import java.lang.annotation.Target;

  @Target(ElementType.FIELD)
  public @interface MyAnnotationDemo01 {
  }
  ```

  然后我们定义这个注解可以使用的对象：

  ```java
  package com.yubulang.demo02;

  public class AnnotationDemo01 {
      // 作用在name属性
      @MyAnnotationDemo01
      protected String name;

      // 作用在常量上
      @MyAnnotationDemo01
      protected final String id = "0001";

      enum HelloEnum {
          THING01,
          THING02
      }

      // 作用在枚举类型
      @MyAnnotationDemo01
      protected HelloEnum helloEnum = HelloEnum.THING01;
  }

  ```

- **METHOD：作用在方法**

  ```java
  // 定义一个注解
  package com.yubulang.demo03;

  import java.lang.annotation.ElementType;
  import java.lang.annotation.Target;

  @Target(ElementType.METHOD)
  public @interface MyAnnotationDemo01 {
  }
  ```

  然后我们定义这个注解可以使用的对象：

  ```java
  package com.yubulang.demo03;

  public class AnnotationDemo01 {
      // 作用于方法
      @MyAnnotationDemo01
      public void test() {

      }
  }
  ```

* **PARAMETER：作用在方法参数上**

  ```java
  // 定义一个注解
  package com.yubulang.demo04;

  import java.lang.annotation.ElementType;
  import java.lang.annotation.Target;

  @Target(ElementType.PARAMETER)
  public @interface MyAnnotationDemo01 {
  }
  ```

  然后我们定义这个注解可以使用的对象：

  ```java
  package com.yubulang.demo04;

  public class AnnotationDemo {
      // 作用在方法的参数上
      public void test(@MyAnnotationDemo01 String name) {

      }
  }
  ```

- **CONSTRUCTOR：作用在构造方法上**

  ```java
  // 定义一个注解
  package com.yubulang.demo05;

  import java.lang.annotation.ElementType;
  import java.lang.annotation.Target;

  @Target(ElementType.CONSTRUCTOR)
  public @interface MyAnnotationDemo01 {
  }
  ```

  然后我们定义这个注解可以使用的对象：

  ```java
  package com.yubulang.demo05;

  public class AnnotationDemo {
      // 作用在构造函数上
      @MyAnnotationDemo01
      public AnnotationDemo() {

      }

      // 这里会报错，因为不是构造方法
      // @MyAnnotationDemo01
      // public void test() {
      //
      // }
  }
  ```

* **LOCAL_VARIABLE：作用在局部变量上**

  ```java
  // 定义一个注解
  package com.yubulang.demo06;

  import java.lang.annotation.ElementType;
  import java.lang.annotation.Target;

  @Target(ElementType.LOCAL_VARIABLE)
  public @interface MyAnnotationDemo01 {
  }
  ```

  然后我们定义这个注解可以使用的对象：

  ```java
  package com.yubulang.demo06;

  public class AnnotationDemo {
      public void test() {
          // 作用在局部变量上
          @MyAnnotationDemo01
          String name = "李雷与韩梅梅";
      }
  }
  ```

- **ANNOTATION_TYPE：作用在注解上**

  ```java
  // 定义一个注解
  package com.yubulang.demo07;

  import java.lang.annotation.ElementType;
  import java.lang.annotation.Target;

  @Target(ElementType.ANNOTATION_TYPE)
  public @interface MyAnnotationDemo {
  }
  ```

  然后我们定义这个注解可以使用的对象：

  ```java
  package com.yubulang.demo07;

  // 作用在注解上
  @MyAnnotationDemo
  public @interface AnnotationDemo {

  }
  ```

* **PACKAGE：作用在包上**

  ```java
  // 定义一个注解
  package com.yubulang.demo08;

  import java.lang.annotation.ElementType;
  import java.lang.annotation.Target;

  @Target(ElementType.PACKAGE)
  public @interface MyAnnotationDemo {
  }
  ```

  然后我们定义这个注解可以使用的对象，这里有个特殊我们需要创建一个 `package-info.java` 这个是包的描述文件。

  ```java
  // package-info.java
  // 作用在包上
  @MyAnnotationDemo
  package com.yubulang.demo08;
  ```

- **TYPE_PARAMETER：作用在定义泛型的类型参数上**

  ```java
  // 定义一个注解
  package com.yubulang.demo09;

  import java.lang.annotation.ElementType;
  import java.lang.annotation.Target;

  @Target(ElementType.TYPE_PARAMETER)
  public @interface MyAnnotationDemo {
  }
  ```

  然后我们定义这个注解可以使用的对象：

  ```java
  package com.yubulang.demo09;

  // 作用在定义泛型类型参数上
  class Demo<@MyAnnotationDemo T> {
      private T data;
  }

  public class AnnotationDemo {
      public void test() {
          Demo<String> demo = new Demo<>();
      }
  }
  ```

* **TYPE_USE：作用在泛型类型使用参数上**

  ```java
  // 定义一个注解
  package com.yubulang.demo10;

  import java.lang.annotation.ElementType;
  import java.lang.annotation.Target;

  @Target(ElementType.TYPE_USE)
  public @interface MyAnnotationDemo {
  }
  ```

  然后我们定义这个注解可以使用的对象：

  ```java
  package com.yubulang.demo10;

  class Demo<T> {
      private T data;
  }

  public class AnnotationDemo {
      // 作用在泛型类型使用时
      Demo<@MyAnnotationDemo String> demo = new Demo<>();
  }
  ```

### @Retention 用于表示注解的生命周期

我们查看源代码可以知道，Retention 可以传入 `value` 这个值的类型为`RetentionPolicy` 枚举类

```java
public @interface Retention {
    /**
     * Returns the retention policy.
     * @return the retention policy
     */
    RetentionPolicy value();
}
```

我们一般写代码的时候是源代码阶段（SOURCE），编译成 CLASS 的时候就是（CLASS），运行的时候就是（RUNTIME）。作用域由大到小应该是：RUNTIME > CLASS > SOURCE。

这个理解起来想容易，我们来使用一下：

```java
// 例子一=============================
package com.yubulang.demo11;

import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;

// 只有在源代码时
@Retention(RetentionPolicy.SOURCE)
public @interface MyAnnotationDemo01 {
}

// 例子二=============================
package com.yubulang.demo11;

import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;

// 注解在java文件编译成为class依然有效
@Retention(RetentionPolicy.CLASS)
public @interface MyAnnotationDemo02 {
}

// 例子三=============================
package com.yubulang.demo11;

import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;

// 注解在运行时依然有效
@Retention(RetentionPolicy.RUNTIME)
public @interface MyAnnotationDemo03 {
}
```

### @Documented 说明该注解将包含在 javadoc 中

默认的`javadoc`工具，是不包括注解的。但是如果声明注解时指定了 `@Documented` 这个注解就会被 `javadoc` 之类的工具处理，所以注解类型信息也会被包括在生成文档中，是一个标记注解，没有成员。

### @Inherited 子类会继承父类的注解

```java
package com.yubulang.demo12;

import java.lang.annotation.*;

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@interface InheritedTest {
    String value();
}

@InheritedTest("拥有Inherited")
class Person {
    public void methodOne() {
    }

    public void methodTwo() {
    }
}

class Student extends Person {

}

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@interface IsNotInherited {
    String value();
}

@IsNotInherited("未拥有Inherited")
class NotInheritedAnnotationPerson {
    public void methodOne() {
    }

    public void methodTwo() {
    }
}

class NotInheritedAnnotationStudent extends NotInheritedAnnotationPerson {

}

public class Test02 {
    public static void main(String[] args) {
        Class<Student> studentClass = Student.class;

        if (studentClass.isAnnotationPresent(InheritedTest.class)) {
            System.out.println(studentClass.getAnnotation(InheritedTest.class).value());
        }

        Class<NotInheritedAnnotationStudent> nStudentClass = NotInheritedAnnotationStudent.class;

        if (nStudentClass.isAnnotationPresent(IsNotInherited.class)) {
            System.out.println(nStudentClass.getAnnotation(IsNotInherited.class).value());
        }
    }
}
```

### 自定义注解

- 使用@interface 自定义注解时，自动继承了 `java.lang.annotation.Annotation` 接口
- 分析：
  - @interface 用来声明一个注解，格式：`public @interface 注解名 {定义内容}`
  - 其中的每一个方法实际上是声明了一个配置参数
  - 方法的名称就是参数的名称
  - 返回值类型就是参数类型（返回值只能是基本类型，Class，String，enum）
  - 可以通过 default 来声明参数的默认值
  - 如果只有一个参数成员，一般命名为 value
  - 注解元素必须要有值，我们定义注解元素时，经常使用空字符串，0 作为默认值

知道了这些基础的概念，我觉得还是要通过一些代码，才能巩固。废话就不多说我们看代码。

```java
package com.yubulang.demo12;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

public class Test01 {
    // 没有 default 默认值，我们必须给注解赋值
    // 有 default "" 可以显式赋值，也可以不赋值
    @MyAnnotation01(name = "大哥", schools = {"哈理工", "哈工大"})
    public void test() {
    }

    @MyAnnotation02("大哥")
    public void test02() {
    }
}

@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@interface MyAnnotation01 {
    // 注解的参数：参数类型 + 参数名()
    String name() default "";

    int age() default 0;

    int id() default -1; // 如果默认值为 -1，代表不存在

    String[] schools() default {};
}

@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@interface MyAnnotation02 {
    String value(); // 方法名为value，才能省略，其他的不行
}
```

## 总结

知道了怎么定义注解只是一个开始，我们想要注解去处理一些具体的事儿，还需要反射的配合才能实现。万丈高楼平地起，今天就到这里。
