---
title: Java 注解之元注解 @Target 用于描述注解的作用范围
date: 2020-04-04 00:19:06
categories:
  - Java
tags:
  - Java
---

## 基本概念

- 元注解的作用就是负责注解其他注解，Java 定义了 4 个标准的 meta-annotation 类型，他们被用来提供对其他 annotation 类型作说明。
- 这些类型和它们所支持的类在`java.lang.annotation`包中找到。（@Target，@Retention，@Documented，@Inherited）
  - @Target：用于描述注解的使用范围（即：被描述的注解可以用在什么地方）
  - @Retention：表示需要在什么级别保存该注解信息，用于描述注解的声明周期（SOURCE < CLASS < RUNTIME）
  - @Documented：说明该注解将包含在 javadoc 中
  - @Inherited：说明子类可以继承父类中的该注解

当然这里我们主要讲 @Target 这个元注解。

<!-- more -->

## 各种 @Target 用到的情况

我们查看源代码可以知道，Target 可以传入一个值 `value` 这个 value 的类型为 `ElementType`

```java
public @interface Target {
    ElementType[] value();
}
```

让我们看看这个 `ElementType` 枚举类包含的值，表示注解作用在什么上面：

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

- **FIELD：作用在属性字段，包括枚举和常量**

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

- **PARAMETER：作用在方法参数上**

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

- **LOCAL_VARIABLE：作用在局部变量上**

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

- **PACKAGE：作用在包上**

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

- **TYPE_USE：作用在泛型类型使用参数上**

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

## 总结

这里只讲解元注解 @Target 的大概使用方式，当然更多的使用方式可能会更加复杂，但是先从熟悉简单的，后面复杂的也能逐步的拆解。
