---
title: JavaScript设计模式——单例模式
date: 2020-03-04 13:14:15
categories:
  - JavaScript
tags:
  - 设计模式
  - JavaScript
---

众所周知，程序开发有很多语言，它们拥有各种不同的语法和规则，但是通用的思想部分就是——数据结构、设计模式，业内俗称内功。我是 PHP 入行，用 PHP 来实现一个单例我觉得不是什么难题。突然有一天 Node.js 进入了我的视线，心想 JavaScript 这几年皮呀，但是我喜欢。

单例的定义：

> 保证一个类仅有一个实例，并提供一个访问它的全局访问点。实现的方法为先判断实例是否存在，如果存在则直接返回，否则就创建实例再返回，这就保证了一个类只实例化一次。

使用场景：

> 一个单一对象。比如：数据库连接、弹窗，一次运行无论操作多少次数据库，连接只存在一个，弹窗不管点击多少次，弹窗只能创建一次。

代码实现：
<!-- more -->
```javascript
class Singleton {
  constructor() {}
}

Singleton.getInstance = (() => {
  let instance;

  return () => {
    if (!instance) {
      instance = new Singleton();
    }
    return instance;
  };
})();

let s1 = Singleton.getInstance();
let s2 = Singleton.getInstance();
console.log(s1 === s2);
```

可以看出来，这个实现并不是很复杂，只是通过一个变量把实例缓存起来，判断这个变量缓存存在则返回`instance`这个实例。
