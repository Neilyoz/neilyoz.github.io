---
title: JavaScript设计模式——原型模式
date: 2020-03-04 18:29:53
categories:
  - 前端笔记
  - JavaScript
tags:
  - 设计模式
  - JavaScript
---

原型模式的定义

> 用原型实例指向创建对象的类，使用于创建新的对象的类共享原型对象的属性以及方法。

使用场景：

> 创建新的对象的类共享原型对象的属性以及方法，多以继承来呈现。

我们可以通过 JavaScript 特有的原型继承特性去实现原型模式，也就是创建一个对象作为另一个对象的 prototype 属性值，可以通过 Object.create(prototype, optionalDescriptorObjects)来实现原型继承。

```javascript
var someAnimal = {
  name: "喵星人",
  eat: function() {}
};

// 使用Object.create创建一个新的动物
var anotherAnimal = Object.create(someAnimal, {
  eat: {
    value: function() {
      console.log("吃骨头");
    }
  }
});
anotherAnimal.name = "汪星人";
```

如果我们不想通过`Object.create`去创建，则可以使用以下代码实现

```javascript
var someAnimal = {
  name: "喵星人",
  init: function(name) {
    this.name = name;
  },
  eat: function(thing) {
    console.log("吃" + thing);
  }
};

function anotherAnimal(name) {
  function F() {}
  F.prototype = someAnimal;
  var f = new F();
  f.init(name);
  return f;
}

var anotherAnimalObj = anotherAnimal("狗");
anotherAnimalObj.eat("骨头");
```

在 ES6 里面，就简单了。。。

```javascript
class Animal {
  constructor(name) {
    this.name = name;
  }

  eat(thing) {
    console.log("吃" + thing);
  }
}

class AnotherAnimal extends Animal {
  constructor(name) {
    super(name);
  }
}
```

得益于 ES6 的便利，这个模式不用再像之前一样，复杂的编码了，当然这个也只能在开发的时候使用，我们的 Babel 还是会把代码编译成为 ES5 的运行模式，方便代码浏览器和终端都能识别运行。
