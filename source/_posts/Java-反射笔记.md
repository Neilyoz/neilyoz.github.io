---
title: Java 反射笔记
date: 2020-04-04 11:30:16
categories:
  - Java
tags:
  - Java
  - 反射
---

## 前言

有了之前的注解内容，要搭配 Java 反射就能给注解注入灵魂了。

## Java Reflection——反射

### 基础概念

- Reflecttion（反射）是 Java 被视为动态语言的关键，反射机制允许程序在执行期借助于 Reflection API 取地任何类的内部信息，并且直接操作任意对象的内部属性及方法。
- 加载完类之后，在堆内存的方法中就产生了一个 Class 类型的对象（一个类只有一个 Class 对象），这个对象就包含了完整的类的结构信息。我们可以通过这个对象看类的结构。这个对象就像是一面镜子，透过这个镜子看到类的结构，所以 Reflection 我们形象的成为反射。

正常方式：`引入需要的"包类"名称` -> `通过 new 实例化` -> `取得实例化对象`
反射方式：`实例化对象` -> `getClass()方法` -> `得到完整的"包类"名称`

### 反射机制提供的功能

- 在运行时判断任意一个对象所属的类
- 在运行时构造任意一个类的对象
- 在运行时判断任意一个类所具有的成员变量和方法
- 在运行时获取泛型信息
- 在运行时调用任意一个对象的成员变量和方法
- 在运行时处理注解
- 生成动态代理
- 等等

### 优点和缺点

**优点**

- 可以实现动态创建对象和编译，体现出很大的灵活性

**缺点**

- 对性能有影响。使用反射基本上是一种解释操作，我们可以告诉 JVM，我们希望做什么并且它满足我们的要求。这类操作总是慢于直接执行相同操作。

<!-- more -->

### 反射相关的主要 API

- java.lang.Class：代表一个类
- java.lang.reflect.Method：代表类的方法
- java.lang.reflect.Field：代表类的成员变量
- java.lang.reflect.Constructor：代表类的构造器

## Class 类

在 Object 类中定义了 getClass()方法，此方法被所有子类继承

```java
public final Class getClass()
```

getClass 方法的返回值是一个 Class 类，此类是 Java 反射的源头，实际上所谓的反射从程序的运行结果来看也很好理解，即：可以通过对象反射求出类的名称。

对象照镜子可以得到的信息：某个类的属性、方法和构造器、某个类到底实现了哪些接口。对于每个类而言，JRE 都为其保留一个不变的 Class 类型的对象。一个 Class 对象包含了特定某个结构(class/interface/enum/annotation/primitive type/void/[])的有关信息。

- Class 本身也是一个类
- Class 对象只能由系统建立对象
- 一个加载的类在 JVM 中只会有一个 Class 实例
- 一个 Class 对象对应的是一个加载到 JVM 中的一个.class 文件
- 每个类的实例都会记得自己是由哪个 Class 实例所生成
- 通过 Class 可以完整地得到一个类中的所有被加载的结构
- Class 类是 Reflection 的根源，针对任何你想动态加载、运行的类，唯有先获得相应的 Class 对象

看个实际例子：

```java
package com.yubulang.pojo.demo01;

// 什么叫反射
public class Demo01 {
    public static void main(String[] args) throws ClassNotFoundException {
        // 通过反射获取类的Class对象
        Class c1 = Class.forName("com.yubulang.pojo.demo01.User");
        Class c2 = Class.forName("com.yubulang.pojo.demo01.User");
        Class c3 = Class.forName("com.yubulang.pojo.demo01.User");
        Class c4 = Class.forName("com.yubulang.pojo.demo01.User");

        // 一个类在内存中只有一个Class对象，所以hashCode都相同
        // 一个类被加载后，类整个结构都会被封装在Class对象中。
        System.out.println(c1.hashCode());
        System.out.println(c2.hashCode());
        System.out.println(c3.hashCode());
        System.out.println(c4.hashCode());
    }
}

// 实体类：
class User {
    private String name;
    private int id;
    private int age;

    public User() {
    }

    public User(String name, int id, int age) {
        this.name = name;
        this.id = id;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                ", id=" + id +
                ", age=" + age +
                '}';
    }
}
```

### 常用方法

| 方法名                                  | 功能说明                                                         |
| --------------------------------------- | ---------------------------------------------------------------- |
| static Class forName(String name)       | 返回指定类名 name 的 Class 对象                                  |
| Object newInstance()                    | 调用默认构造函数，返回 Class 对象的一个实例                      |
| String getName()                        | 返回此 Class 对象所表示的实体（类、接口、数组类或 void）的名称。 |
| Class getSuperClass()                   | 返回当前 Class 对象的父类 Class 对象                             |
| Class[] getInterfaces()                 | 获取当前 Class 对象的接口                                        |
| ClassLoader getClassLoader()            | 返回该类的加载器                                                 |
| Constructor[] getConstructors()         | 返回一个包含某些 Constructor 对象的数组                          |
| Method getMethod(String name, Class..T) | 返回 Mthod 对象，此对象的形参型为 paramType                      |
| Field[] getDeclaredFields()             | 返回 Field 对象的一个数组                                        |

### 获取 Class 类的实例

- 若已知具体的类，通过类的 class 属性获取，该方法最为安全可靠，程序性能最高
  ```java
  Class person = Person.class;
  ```
- 已知某个类的实例，调用该实例的 getClass()方法获取 Class 对象
  ```java
  class Person {}
  Person person = new Person();
  Class person = person.getClass();
  ```
- 已知一个类的全类名，且该类在类路径下，可通过 Class 类的静态方法 forName()获取，可能抛出 `ClassNotFoundException`
  ```java
  Class person = Class.forName("demo01.Person");
  ```
  综合实例代码：

```java
package com.yubulang.pojo.demo02;

// 测试class类的创建方式有哪些
public class Demo02 {
    public static void main(String[] args) throws ClassNotFoundException {
        Person person = new Student();
        System.out.println("这个人是：" + person.getName());

        // 方式一：通过对象获得
        Class<? extends Person> c1 = person.getClass();
        System.out.println(c1.hashCode());

        // 方式二：通过forName获得
        Class<?> c2 = Class.forName("com.yubulang.pojo.demo02.Student");
        System.out.println(c2.hashCode());

        // 方式三：通过类名.class
        Class<Student> c3 = Student.class;
        System.out.println(c3.hashCode());

        // 方式四：基本内置类型的包装类都有一个Type属性
        Class<Integer> c4 = Integer.TYPE;
        System.out.println(c4);

        // 获得父类类型
        Class<?> c5 = c1.getSuperclass();
        System.out.println(c5);
    }
}

class Person {
    private String name;

    public Person() {
    }

    public Person(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                '}';
    }
}

class Student extends Person {
    public Student() {
        this.setName("学生");
    }
}

class Teacher extends Person {
    public Teacher() {
        this.setName("老师");
    }
}
```

### 哪些类型可以有 Class 对象

- class：外部类，成员（成员内部类，静态内部类），局部内部类，匿名内部类
- interface：接口
- []：数组
- enum：枚举
- annotation：注解@interface
- primitive type：基本数据类型
- void

上代码验证下：

```java
package com.yubulang.pojo.demo03;

import java.lang.annotation.ElementType;

public class Demo {
    public static void main(String[] args) {
        // 所有类型的Class

        Class<Object> c1 = Object.class; // 类
        Class<Comparable> c2 = Comparable.class;  // 接口
        Class<String[]> c3 = String[].class; // 一维数组
        Class<int[][]> c4 = int[][].class; // 二位数组
        Class<Override> c5 = Override.class; // 注解
        Class<ElementType> c6 = ElementType.class; // 枚举
        Class<Integer> c7 = Integer.class; // 基本类型
        Class<Void> c8 = void.class; // void
        Class<Class> c9 = Class.class; // Class

        System.out.println(c1);
        System.out.println(c2);
        System.out.println(c3);
        System.out.println(c4);
        System.out.println(c5);
        System.out.println(c6);
        System.out.println(c7);
        System.out.println(c8);
        System.out.println(c9);

        // 只要元素类型与维度一样，就是同一个Class
        int[] a = new int[10];
        int[] b = new int[100];
        System.out.println(a.getClass().hashCode());
        System.out.println(b.getClass().hashCode());
    }
}
```

## Java 内存分析

- 栈内存
  - 存放基本变量类型（会包含这个基本类型的具体数值）
  - 引用对象的变量（会存放这个引用在堆里面的具体地址）
- 堆内存
  - 存放 new 的对象和数组
  - 可以被所有的线程共享，不会存放别的对象引用
- 方法区
  - 可以被所有的线程共享
  - 包含了所有 class 和 static 变量

## 了解类的加载过程

当程序主动使用某个类时，如果该类还未被加载到内存中，则系统会通过如下三个步骤来对类进行初始化

- 类加载（Load）
  - 将类的 class 文件读入内存，并为之创建一个 `java.lang.Class` 对象。此过程由类加载器完成
- 类的连接（Link）
  - 将类的二进制数据合并到 JRE 中
- 类的初始化（Initialize）
  - JVM 负责对类进行初始化

### 类的加载与 ClassLoader 的理解

- 加载：将 class 文件字节码内容加载到内存中，并将这些静态数据转换成方法区的运行时数据结构，然后生成一个代表这个类的`java.lang.Class`对象。
- 链接：将 Java 类的二进制代码合并到 JVM 的运行状态之中的过程。
  - 验证：确保加载的类信息符合 JVM 规范，没有安全方面的问题
  - 准备：正式为类变量（static）分配内存并设置类变量默认初始值的阶段，这些内存都将在方法中进行分配。
  - 解析：虚拟机常量池内的符号引用（常量名）替换为直接引用（地址）的过程。
- 初始化：
  - 执行类构造器\<cIinit\>()方法的过程，类构造器\<cIinit\>()方法是由编译器自动收集类中所有类变量的赋值动作和静态代码块中的语句合并产生的。（类构造器是构造类信息的，不是构造该类对象的构造器）。
  - 当初始化一个类的时候，如果发现其父类还没有进行初始化，则需要先触发其父类的初始化。
  - 虚拟机会保证一个类的\<cIinit\>()方法在多线程环境中被正确加锁和同步。

代码演示如下

```java
package com.yubulang.pojo.demo04;

public class Demo {
    public static void main(String[] args) {
        /*
        1. 加载到内存，会产生一个类对应Class对象
        2. 链接，链接结束后 m = 0
        3. 初始化
            方法区会有类的白能量初始化信息，而赋值的动作会在下面完成
            <cInit>() {
                // 编译器自动收集类中所有类变量的赋值动作和静态代码块中的语句合并产生
                System.out.println("A类静态代码块初始化");
                m = 300;
                m = 100;
            }
         */
        A a = new A();
        System.out.println(A.m);
    }
}

class A {
    static {
        System.out.println("A类静态代码块初始化");
        m = 300;
    }

    static int m = 100;

    public A() {
        System.out.println("A类的无参构造器初始化");
    }
}

```

### 什么时候会发生类初始化？

- 类的主动引用（一定会发生类的初始化）

  - 当虚拟机启动，先初始化 main 方法所在类
  - new 一个类的对象
  - 调用类的静态成员（除了 final 常量）和静态方法
  - 使用 java.lang。reflect 包的方法和对类进行反射调用
  - 初始化一个类，如果其父类没有被初始化，则先初始化它的父类

- 类的被动引用（不发生类的初始化）
  - 当访问一个静态域时，只有真正声明这个域的类才会被初始化。如：当通过子类引用父类的静态变量，不会导致子类初始化
  - 通过数组定义类引用，不会触发此类的初始化
  - 引用常量不会触发此类的初始化（常量在链接阶段就存入调用类的常量池中了）

上面话对应的例子：

```java
package com.yubulang.pojo.demo05;

public class Demo {
    static {
        System.out.println("Main所在类被加载");
    }

    public static void main(String[] args) throws ClassNotFoundException {
        // 类的主动引用（一定会发生类的初始化）
        // 1. 主动引用，初始化一个类，如果其父类没有被初始化，则先会初始化其父类
        /*
        输出：
            Main所在类被加载
            父类被加载
            子类被加载
         */
        // Son son = new Son();

        // 2. 反射也会产生主动给引用
        /*
        输出：
            Main所在类被加载
            父类被加载
            子类被加载
         */
        // Class.forName("com.yubulang.pojo.demo05.Son");

        // 3. 调用类的静态成员（除了final常量）和静态方法
        /*
        输出：
            Main所在类被加载
            父类被加载
            子类被加载
            100
         */
        // System.out.println(Son.m);

        /*
        输出：
            Main所在类被加载
            父类被加载
            子类被加载
            Son出来秀了
         */
        // Son.show();


        // ====================================================

        // 类的被动引用（不会发生类的初始化）
        // 1. 当通过子类引用父类的静态变量，不会导致子类初始化
        /*
        输出：
            Main所在类被加载
            父类被加载
            2
         */
        // System.out.println(Son.b);

        // 2. 通过数组定义类引用，不会触发此类的初始化
        // 只有main类被加载
        // Son[] array = new Son[5];

        // 3. 引用常量不会触发此类的初始化
        // 只有main类被加载
        // System.out.println(Son.M);
    }
}

class Father {
    static int b = 2;

    static {
        System.out.println("父类被加载");
    }
}

class Son extends Father {
    static {
        System.out.println("子类被加载");
        m = 300;
    }

    static int m = 100;
    static final int M = 100;

    public static void show() {
        System.out.println("Son出来秀了");
    }
}
```

## 类加载器的作用

- 类加载的作用：将 class 文件字节码内容加载到内存中，并将这些静态数据转换成方法区的运行时数据结构，然后在堆中生成一个代表这个类的 `java.lang.Class` 对象，作为方法区中类数据的访问入口。
- 类缓存：标准的 JavaSE 类加载器可以按照要求查找类，但一旦某个类被加载到类加载器中，它将维持加载（缓存）一段时间，不过 JVM 垃圾回收机制可以回收这些 Class 对象。

### 类加载器的作用

类加载器作用是用来把类（class）装载进内存。JVM 规范定义了如下类型的类的加载器。

- 引导类加载器：用 C++编写，是 JVM 自带的类加载器，负责 Java 平台核心库，用来装载核心类库。该装载类无法直接获取
- 扩展加载器：负责 jre/lib/ext 目录下 jar 包或 -D java.ext.dirs 指定目录下的 jar 包装入工作库
- 系统类加载器：负责 java --classpath 或 -D java.class.path 所指的目录下的类与 jar 包装入工作，是常用的加载器。

我们来获取下类加载器：

```java
package com.yubulang.pojo.demo06;

public class Demo {
    public static void main(String[] args) throws ClassNotFoundException {
        // 获取系统类的加载器
        ClassLoader systemClassLoader = ClassLoader.getSystemClassLoader();
        System.out.println(systemClassLoader);

        // 获取系统类加载器的父类加载器 -> 扩展加载器
        ClassLoader parent = systemClassLoader.getParent();
        System.out.println(parent);

        // 获取扩展类加载器的父类加载器 -> 根加载器(C++)
        ClassLoader root = parent.getParent();
        System.out.println(root);

        // 测试当前类是哪个加载器加载的
        ClassLoader classLoader = Class.forName("com.yubulang.pojo.demo06.Demo").getClassLoader();
        System.out.println(classLoader);

        // 测试JDK的类，结果是根加载器（C++）那层的
        classLoader = Class.forName("java.lang.Object").getClassLoader();
        System.out.println(classLoader);

        // 如何获取系统类加载器可以加载的路径
        String classPath = System.getProperty("java.class.path");
        System.out.println(classPath);

        // 双清委派机制
        // 假设你自定义了一个 java.lang.String包，
        // 加载完成后Java会检测包的正确性，
        // 会用自身root加载的java.lang.String包，
        // 覆盖你写的包，以保证包的正确性
    }
}
```

## 创建运行类的对象

###获取类运行时类的完整结构  
通过反射获取运行时类的完整结构，我们可以获取类的 Field、Method、Constructor、Superclass、Interface、Annotation。

- 实现的全部接口
- 所继承的父类
- 全部的构造器
- 全部的方法
- 全部的字段
- 注解

上例子

```java
package com.yubulang.pojo.demo07;


import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.lang.reflect.Method;

public class Demo {
    public static void main(String[] args) throws ClassNotFoundException, NoSuchFieldException, NoSuchMethodException {
        Class<?> c1 = Class.forName("com.yubulang.pojo.demo07.User");

        // 获得类的名字
        System.out.println("=== 类的名字 ===");
        System.out.println(c1.getName()); // 获取 包和类的名字完整信息
        // 获取类的简单名字
        System.out.println(c1.getSimpleName()); // 获取 类的名字信息

        // 获取类的属性
        System.out.println("=== 类的 public 属性 ====");
        Field[] fields = c1.getFields(); // 获取类的 public 属性
        for (Field field : fields) {
            System.out.println(field);
        }

        System.out.println("=== 类的所有属性 ====");
        Field[] declaredFields = c1.getDeclaredFields(); // 获取类的所有属性
        for (Field declaredField : declaredFields) {
            System.out.println(declaredField);
        }

        // 获得指定属性的值
        System.out.println("=== 获得指定属性的值 ===");
        Field name = c1.getDeclaredField("name");
        System.out.println(name);

        // 获取所有的方法
        System.out.println("=== 获取所有的方法 ===");
        Method[] methods = c1.getMethods(); // 获取本类及其父类的全部public方法
        for (Method method : methods) {
            System.out.println("getMethods:" + method);
        }

        Method[] declaredMethods = c1.getDeclaredMethods(); // 获取本类的全部方法(包含私有方法）
        for (Method declaredMethod : declaredMethods) {
            System.out.println("getDeclaredMethods:" + declaredMethod);
        }

        // 获得指定方法
        System.out.println("=== 获得指定方法 ===");
        Method getName = c1.getMethod("getName", null);
        System.out.println(getName);
        Method setName = c1.getMethod("setName", String.class);
        System.out.println(setName);

        // 获取所有构造器
        System.out.println("=== 获取所有public构造器 ===");
        Constructor<?>[] constructors = c1.getConstructors();
        for (Constructor<?> constructor : constructors) {
            System.out.println(constructor);
        }

        System.out.println("=== 获取所有构造器 ===");
        Constructor<?>[] declaredConstructors = c1.getDeclaredConstructors();
        for (Constructor<?> constructor : declaredConstructors) {
            System.out.println(constructor);
        }

        // 获取指定构造器
        System.out.println("=== 获取指定构造器 ===");
        Constructor<?> constructor = c1.getConstructor(null);
        System.out.println(constructor);
        constructor = c1.getConstructor(String.class, int.class, int.class);
        System.out.println(constructor);
    }
}

class User {
    private String name;
    private int id;
    private int age;

    public User() {
    }

    public User(String name, int id, int age) {
        this.name = name;
        this.id = id;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                ", id=" + id +
                ", age=" + age +
                '}';
    }
}
```

## 小结

- 在实际的操作中，取得类的信息的操作代码，并不会经常开发
- 一定要熟悉 java.lang.reflect 包的作用，反射机制。
- 如何获取属性、方法、构造器的名称，修饰符等。

## 有了 Class 对象，能做什么？

- 创建类的对象：调用 Class 对象的 newInstance()方法

  - 类必须由一个无参数的构造器
  - 类的构造器的访问权限需要足够

没有无参构造器时，需要在操作的时候明确的调用类中的构造器，并将参数传递进去之后，才可以实例化操作。
步骤：

- 通过 Class 类 getDeclaredConstrutor(Class ... parameterTypes)取得本类的指定形参类型构造器
- 想构造器的形参中传递一个对象数组进去，里面包含了构造器中所需的各个参数
- 通过 Constructor 实例化对象

### 调用指定的方法

通过反射，调用类中的方法，通过 Method 类完成。

- 通过 Class 类的 getMethod(String name, Class...parameterTypes)方法取得一个 Method 对象，并设置此方法操作时所需要的参数类型。
- 之后使用 Object invoke(Object obj, Object[] args)进行调用，并向方法中传递要设置的 obj 对象的参数信息

```java
Object invoke(Object obj, Object... args)
```

- Object 对应原方法的返回值，若原方法无返回值，此时返回 null
- 若原方法若为静态方法，此时形参 Object obj 可为 null
- 若原方法形参列表为空，则 Object[] args 为 null
- 若原方法声明为 private，则需要在调用此 invoke()方法前，显式调用方法对象的 setAccessible(true)方法，将可以访问 private 方法。

show the code：

```java
package com.yubulang.pojo.demo08;

import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;

public class Demo {
    public static void main(String[] args) throws ClassNotFoundException, IllegalAccessException, InstantiationException, NoSuchMethodException, InvocationTargetException, NoSuchFieldException {
        // 获取一个Class对象
        Class<?> c1 = Class.forName("com.yubulang.pojo.demo08.User");

        // 构建一个对象
        User user = (User) c1.newInstance();

        // 通过构造器创建对象（有参的）
        Constructor<?> declaredConstructor = c1.getDeclaredConstructor(String.class, int.class, int.class);
        User user2 = (User) declaredConstructor.newInstance("鱼不浪", 1, 18);
        System.out.println(user2);

        // 通过反射调用方法
        // invoke(对象,  对象方法的参数)
        User user3 = (User) c1.newInstance();
        Method setName = c1.getMethod("setName", String.class);
        setName.invoke(user3, "鱼不浪");

        Method getName = c1.getMethod("getName", null);
        System.out.println(getName.invoke(user3, null));

        // 通过反射操作属性
        User user4 = (User) c1.newInstance();
        Field name = c1.getDeclaredField("name");
        // 不能直接操作私有属性，我们要关闭程序安全检测，属性或者方法的setAccessible
        name.setAccessible(true); // 因为是私有属性，所以要把访问类型改成可以访问
        name.set(user4, "鱼不浪");
        System.out.println(name.get(user4));
    }
}

class User {
    private String name;
    private int id;
    private int age;

    public User() {
    }

    public User(String name, int id, int age) {
        this.name = name;
        this.id = id;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                ", id=" + id +
                ", age=" + age +
                '}';
    }
}
```

### setAccessible

- Method 和 Field、Construtor 对象都有 setAccessible() 方法
- setAccessible 作用时启动和禁用访问安全检查的开关
- 参数值为 true 则表示反射的对象在使用时应该取消 Java 语法访问检查
  - 提高反射的效率。如果代码中必须用反射，该句代码需要频繁的被调用，那么请设置为 true
  - 使得原本无法访问的私有成员也可以访问
- 参数值为 false 则表示反射的对象应该实施 Java 语言访问检查

测试代码上：

```java
package com.yubulang.pojo.demo09;

import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;

public class Demo {
    // 普通方式调用
    public static void test01() {
        User user = new User();
        long startTime = System.currentTimeMillis();
        for (int i = 0; i < 1000000000; i++) {
            user.getName();
        }
        long endTime = System.currentTimeMillis();

        System.out.println("普通方式调用10亿次耗时：" + (endTime - startTime) + "ms");
    }

    // 反射方式调用
    public static void test02() throws ClassNotFoundException, NoSuchMethodException, IllegalAccessException, InstantiationException, InvocationTargetException {
        Class<?> c1 = Class.forName("com.yubulang.pojo.demo09.User");
        User user = (User) c1.newInstance();
        Method getName = c1.getMethod("getName");
        long startTime = System.currentTimeMillis();
        for (int i = 0; i < 1000000000; i++) {
            getName.invoke(user, null);
        }
        long endTime = System.currentTimeMillis();

        System.out.println("反射方式没关闭检测调用10亿次耗时：" + (endTime - startTime) + "ms");
    }

    // 反射方式调用 关闭检测
    public static void test03() throws ClassNotFoundException, IllegalAccessException, InstantiationException, NoSuchMethodException, InvocationTargetException {
        Class<?> c1 = Class.forName("com.yubulang.pojo.demo09.User");
        User user = (User) c1.newInstance();
        Method getName = c1.getMethod("getName");
        getName.setAccessible(true);
        long startTime = System.currentTimeMillis();
        for (int i = 0; i < 1000000000; i++) {
            getName.invoke(user, null);
        }
        long endTime = System.currentTimeMillis();

        System.out.println("反射方式关闭检测调用10亿次耗时：" + (endTime - startTime) + "ms");
    }

    public static void main(String[] args) {
        test01();

        try {
            test02();
            test03();
        } catch (Exception ignored) {
            System.out.println(ignored.getMessage());
        }
    }
}

class User {
    private String name;
    private int id;
    private int age;

    public User() {
    }

    public User(String name, int id, int age) {
        this.name = name;
        this.id = id;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                ", id=" + id +
                ", age=" + age +
                '}';
    }
}
```

## 反射操作泛型

- Java 采用泛型擦除的机制来引入泛型，Java 中的泛型仅仅时给编译器 javac 使用的，确保数据的安全性和免去强制类型转换问题，但是，一旦编译完成，所有和泛型有关的类型全部擦除
- 为了通过反射操作这些类型，Java 新增了 ParameterizedType，GenericArrayType，TypeVariable 和 WildcardType 几种类型来代表不能被统一的 Class 类中的类型但是又和原始类型齐名的类型。
- ParameterizedType：表示一种参数化类型，比如 Collection\<String\>
- GenericArrayType：表示一种元素类型是参数化类型或者类型变量的数组类型
- TypeVariable：是各种类型变量的公共父接口
- WildcardType：代表一种通配符类型表达式

上代码：

```java
package com.yubulang.pojo.demo10;

import java.lang.reflect.Method;
import java.lang.reflect.ParameterizedType;
import java.lang.reflect.Type;
import java.util.List;
import java.util.Map;

public class Demo {
    // 通过反射获取泛型
    public void test01(Map<String, User> map, List<User> list) {
        System.out.println("test01");
    }

    public Map<String, User> test02() {
        System.out.println("test02");
        return null;
    }

    public static void main(String[] args) throws NoSuchMethodException {
        Method method = Demo.class.getMethod("test01", Map.class, List.class);

        // 获取参数的泛型类型
        Type[] genericParameterTypes = method.getGenericParameterTypes();
        for (Type genericParameterType : genericParameterTypes) {
            System.out.println(genericParameterType);

            // 检测参数是不是结构化参数类型
            if (genericParameterType instanceof ParameterizedType) {

                // 获取泛型类型中的真实类型
                Type[] actualTypeArguments = ((ParameterizedType) genericParameterType).getActualTypeArguments();
                for (Type actualTypeArgument : actualTypeArguments) {
                    System.out.println(actualTypeArgument);
                }
            }
        }

        System.out.println("==================================");

        Method method02 = Demo.class.getMethod("test02", null);
        // 获取函数的返回值泛型类型
        Type genericReturnType = method02.getGenericReturnType();
        // 检测参数是不是结构化参数类型
        if (genericReturnType instanceof ParameterizedType) {
            System.out.println(genericReturnType);

            // 获取泛型类型中的真实类型
            Type[] actualTypeArguments = ((ParameterizedType) genericReturnType).getActualTypeArguments();
            for (Type actualTypeArgument : actualTypeArguments) {
                System.out.println(actualTypeArgument);
            }
        }
    }
}

class User {
    private String name;
    private int id;
    private int age;

    public User() {
    }

    public User(String name, int id, int age) {
        this.name = name;
        this.id = id;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                ", id=" + id +
                ", age=" + age +
                '}';
    }
}
```

## 反射操作注解

- getAnnotations
- getAnnotation

练习注解：

```java
package com.yubulang.pojo.demo11;

import java.lang.annotation.*;
import java.lang.reflect.Field;

// 练习反射操作注解
public class Demo {
    public static void main(String[] args) throws ClassNotFoundException, NoSuchFieldException {
        Class<?> c1 = Class.forName("com.yubulang.pojo.demo11.Student");

        // 通过反射获得注解
        Annotation[] annotations = c1.getAnnotations();
        for (Annotation annotation : annotations) {
            System.out.println(annotation);
        }

        // 获得注解的value的值
        TableMapper tableMapper = (TableMapper) c1.getAnnotation(TableMapper.class);
        System.out.println(tableMapper.value());

        // 获取类指定的注解
        Field name = c1.getDeclaredField("name");
        FieldMapper nameFieldAnnotation = name.getAnnotation(FieldMapper.class);
        System.out.println(nameFieldAnnotation.columnName());
        System.out.println(nameFieldAnnotation.type());
        System.out.println(nameFieldAnnotation.length());
    }
}

// 类名的注解
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@interface TableMapper {
    String value();
}

// 属性的注解
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
@interface FieldMapper {
    String columnName();

    String type();

    int length();
}

@TableMapper("db_student")
class Student {
    @FieldMapper(columnName = "db_id", type = "int", length = 10)
    private int id;

    @FieldMapper(columnName = "db_age", type = "int", length = 10)
    private int age;

    @FieldMapper(columnName = "db_name", type = "varchar", length = 64)
    private String name;

    public Student() {
    }

    public Student(int id, int age, String name) {
        this.id = id;
        this.age = age;
        this.name = name;
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "Student{" +
                "id=" + id +
                ", age=" + age +
                ", name='" + name + '\'' +
                '}';
    }
}
```

## 总结

反射和注解提供了给我们更多的操作空间，掌握它可以写出很多优雅的代码。Enjoy it！
