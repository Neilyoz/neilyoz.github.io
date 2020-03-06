---
title: 0x01 Promise理解第一个部分
date: 2020-03-04 02:35:24
categories:
  - JavaScript
tags:
  - JavaScript
---

## 基础铺垫

`event loop` 它的执行顺序：

- 一开始整个脚本作为一个宏任务执行
- 执行过程中同步代码直接执行，宏任务进入宏任务队列，微任务进入微任务队列
- 当红任务执行完出队，检查微任务列表，有则一次执行，直到全部执行完
- 执行浏览器 UI 线程渲染工作
- 检查是否有`Web Worker`任务，有则执行
- 执行完本轮的宏任务，回到 2，依次循环，直到宏任务和微任务队列都为空

**微任务：**

- MutationObserver、Promise.then()或 reject()、Promise 为基础开发的其它技术，比如 feth Api v8 的垃圾回收过程、Node 独有的 process.nextTick。

**宏任务：**

- script、setTimeout、setInterval、setImmdiate、I/O、UI rendering

注意：所有任务开始的时候，由于宏任务包括了 script，所以浏览器会先执行一个宏任务，在这个过程中你看到的延迟任务（例如：setTimeout）将放到下一轮宏任务中执行。

<!-- more -->

## 只看 Promise 部分（理解相对如容易）

**题目一：**

```javascript
const promise1 = new Promise((resolve, reject) => {
  console.log("promise1");
});
console.log("1", promise1);
```

过程分析：

- 从上至下，先遇到`new Promise`，执行该构造函数中的代码`promise1`
- 然后执行同步代码`1` ，此时`promise1`没有被`resolve`或者`reject`，因此状态还是`pending`

结果：

```
'promise1'
'1' Promise{<pending>}
```

**题目二：**

```javascript
const promise = new Promise((resolve, reject) => {
  console.log(1);
  resolve("success");
  console.log(2);
});

promise.then(() => {
  console.log(3);
});

console.log(4);
```

过程分析：

- 从上至下，先遇到`new Promise`，执行中的同步代码`1`
- 再遇到`resolve('success')`，将`promise`的状态改为了`resolved`并且将值保存下来
- 继续执行同步代码`2`
- 跳出`promise`，往下执行，碰到`promise.then`这个微任务，将其加入微任务队列
- 同步执行代码`4`
- 本轮宏任务全部执行完毕，检查微任务队列，发现`promise.then`这个微任务则状态为`resolved`，执行它。

结果：

```
1 2 4 3
```

**题目三：**

```javascript
const promise = new Pormise((resolve, reject) => {
  console.log(1);
  console.log(2);
});

promise.then(() => {
  console.log(3);
});

console.log(4);
```

过程分析：

- 和题目二相似，只不过在`promise`中并没有`resolve`或者`reject`
- 因此`promise.then`并不会执行，它只有在被改变了状态之后才会执行。

结果：

```
1 2 4
```

**题目四：**

```javascript
const pormise1 = new Promise((resolve, reject) => {
  console.log("promise1");
  resolve("resolve1");
});

const promise2 = promise1.then(res => {
  console.log(res);
});

console.log("1", promise1);
console.log("2", promise2);
```

过程分析：

- 从上至下，先遇到`new Promise`，执行该构造函数中的代码`promise1`
- 碰到`resolve`函数，将`promise1`的状态改变为`resolved`，并将结果保存下来
- 碰到`promise1.then`这个微任务，将它放入微任务队列
- `promise2`是一个新状态为`pending`的`Promise`
- 执行同步代码`1`，同时打印出`promise1`的状态是`resolved`
- 执行同步代码`2`，同时打印出`promise2`的状态是`pending`
- 宏任务执行完毕，查找微任务队列，发现`promise1.then`这个微任务且状态为`resolved`，执行它。

结果：

```
'promise1'
'1' Promise{<resolved>: 'resolve1'}
'2' Promise{<pending>}
'resolve1'
```

**题目五：**

```javascript
const fn = () =>
  new Promise((resolve, reject) => {
    console.log(1);
    resolve("success");
  });

fn().then(res => {
  console.log(res);
});

console.log("start");
```

过程分析：

- 从上至下，`new Promise`构造函数创建一个实例赋值给`fn`
- `fn()`调用执行`1`
- `fn().then`新的微任务，加入微任务队列中去
- 执行同步代码`start`
- 因为前面`resolve`，进入微任务队列执行，输出`success`

结果：

```
'1'
'start'
'success'
```

**题目六：**

`fn`调用放`start`之后

```javascript
const fn = () =>
  new Promise((resolve, reject) => {
    console.log(1);
    resolve("success");
  });

console.log("start");

fn().then(res => {
  console.log(res);
});
```

过程分析：

- `new Promise`赋值给`fn`，`fn`未发生调用
- 执行同步代码`start`
- `fn`发生调用，执行`1`，之后执行`resolve`
- `fn().then()`随后执行，输出`success`

结果：

```
'start'
'1'
'success'
```

## Promise 结合 setTimeout

**题目一：**

```javascript
console.log("start");
setTimeout(() => {
  console.log("time");
});
Promise.resolve().then(() => {
  console.log("resolve");
});
console.log("end");
```

过程分析：

- 刚开始整个脚本为一个宏任务执行，对于同步代码直接进入执行栈进行执行，因此先打印出`start`和`end`
- `setTimeout`作为一个宏任务，放入宏任务队列（下一个循环）
- `Promise.then`作为一个微任务，放入微任务队列
- 本次宏任务执行完毕，检查微任务，发现`Promise.then`，然后执行
- 接下来进入下一个宏任务，发现`setTimeout`，然后执行

结果：

```
'start'
'end'
'resolve'
'time'
```

**题目二：**

```javascript
const promise = new Promise((resolve, reject) => {
  console.log("1");
  setTimeout(() => {
    console.log("timerStart");
    resolve("success");
    console.log("timerEnd");
  }, 0);
  console.log(2);
});

promise.then(res => {
  console.log(res);
});

console.log(4);
```

过程分析：

- 从上至下，先遇到`new Promise`，执行该构造函数中的代码`1`
- 然后碰到了定时器，将这个定时器中的函数放到下一个宏任务的延迟队列中等待执行
- 执行同步代码`2`
- 跳出`promise`函数，遇到`promise.then`，但其状态还是为`pending`，这里理解为先不执行
- 执行同步代码`4`
- 一轮循环过后，进入第二次宏任务，发现延迟队列中有`setTimeout`定时器，执行它
- 首先执行`timeStart`，然后遇到了`resolve`，将`promise`的状态改为`resolved`且保存结果并将之前的`promise.then`推入微任务队列
- 继续执行同步代码`timerEnd`
- 宏任务全部执行完毕，查找微任务队列，发现`promise.then`这个微任务，执行它。

结果：

```
'1'
'2'
'4'
'timerStart'
'timerEnd'
'success'
```

**题目三：**

```javascript
setTimeout(() => {
  console.log("timer1");
  setTimeout(() => {
    console.log("timer3");
  }, 0);
});

setTimeout(() => {
  console.log("timer2");
}, 0);

console.log("start");
```

过程分析：

- `script`本来是一个宏任务，先执行`start`
- `setTimeout（timer1）、setTimeout（timer2）`宏任务放入宏任务队列
- 检测宏任务队列，执行`timer1`之后将`setTimeout(timer3)`加入宏任务队列，然后执行`timer2`
- 检测宏任务队列，执行`timer3`

结果：

```
'start'
'timer1'
'timer2'
'timer3'
```

**题目四：**

```javascript
setTimeout(() => {
  console.log("timer1");
  Promise.resolve().then(() => {
    console.log("promise");
  });
}, 0);

setTimeout(() => {
  console.log("timer2");
}, 0);

console.log("start");
```

过程分析：

- `script`本身是宏任务，优先执行`start`
- `setTimeout(timer1)`、`setTimeout(timer2)`放入宏任务队列，并开始执行
- 执行宏任务`timer1`，产生微任务`Promise.resolve().then`，微任务进入队列执行`promise`
- 执行宏任务`timer2`

**题目五：**

```javascript
Promise.resolve().then(() => {
  console.log("promise1");
  const timer2 = setTimeout(() => {
    console.log("timer2");
  }, 0);
});

const timer1 = setTimeout(() => {
  console.log("timer1");
  Promise.resolve().then(() => {
    console.log("promise2");
  });
}, 0);

console.log("start");
```

过程分析：

- `script`作为第一个宏任务来执行，这里标记为**宏任务 1**
- 遇到`Promise.resolve().then`这个微任务，将`then`中的内容加入第一次的微任务队列，标为**微任务 1**
- 遇到`setTimeout（timer1）`，加入到宏任务队列，标为**宏任务 2**，等待执行
- 执行**宏任务 1**的同步代码`start`，完成后，检查微任务队列
- 发现有一个`promise.then`这个微任务执行，打印同步代码`promise1`，发现定时器`timer2`，将它加入**宏任务 2**的后面，标记为**宏任务 3**
- 第一次微任务队列（微任务 1）执行完毕，执行第二次宏任务（宏任务 2），首先执行同步代码`timer1`
- 然后遇到了`promise2`这个微任务，将它加入此次循环的微任务队列，标为**微任务 2**
- **宏任务 2**中没有同步代码可执行，查找本次循环的微任务队列（**微任务 2**），发现`promise2`
- 第二轮执行完毕，执行**宏任务 3**，打印出`timer2`

结果：

```
'start'
'promise1'
'timer1'
'promise2'
'timer2'
```

题目六：

```javascript
const promise1 = new Promise((resolve, reject) => {
  setTimeout(() => {
    reslove("success");
  }, 0);
});

const promise2 = pormise1.then(() => {
  throw new Error("error!!!");
});

console.log("promise1", promise1);
console.log("promise2", promise2);

setTimeout(() => {
  console.log("promise1", promise1);
  console.log("promise2", promise2);
}, 2000);
```

过程分析：

- 先执行第一个`new Promise`中的函数，碰到`setTimeout`将它加入下一个宏任务列表
- 跳出`new Promise`，碰到`promise1.then`这个微任务，但其状态还是为`pending`，这里理解为先不执行
- `promise2`是一个新的状态为`pending`的`Promise`
- 执行同步代码`console.log('promise1')`，则打印出的`promise1`的状态为`pending`
- 执行同步代码`console.log('promise2')`，则打印出的`promise2`的状态为`pending`
- 碰到第二个`setTimeout`定时器，将其放入下一个宏任务列表
- 第一轮宏任务执行结束，并且没有微任务需要执行，因此执行第二轮任务
- 先执行第一个定时器的内容，将`promise1`的状态改为`resolved`且保存结果并将之前的`promise1.then`推入微任务队列
- 该定时器中没有其他的同步代码可执行，因此执行本轮的微任务队列，也就是`promise1.then`，它将抛出一个错误，且将`promise2`的状态设置为了`rejected`
- 第一个定时器执行完毕，开始执行第二个定时器中的内容
- 打印出`promise1`，且此时`promise1`的状态为`resolved`
- 打印出`promise2`，且此时`promise2`的状态为`rejected`

结果：

```
'promise1' Promise{<pending>}
'promise2' Promise{<pending>}
Uncaught (in promise) Error: error!!!
'promise1' Promise{<resolved>: "success"}
'promise2' Promise{<rejected>: Error: error!!!}
```

题目七：

```javascript
const promise1 = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve("success");
    console.log("timer1");
  }, 1000);
  console.log("promise1里面的内容");
});

const promise2 = promise1.then(() => {
  throw new Error("error!!!");
});

console.log("promise1", promise1);
console.log("promise2", promise2);

setTimeout(() => {
  console.log("timer2");
  console.log("promise1", promise1);
  console.log("promise2", promise2);
}, 2000);
```

结果：

```
'promise1里面的内容'
'promise1' Promise{<pending>}
'promise2' Promise{<pending>}
'timer1'
Uncaught (in promise) Error: error!!!
'timer2'
'promise1' Promise{<resolved>: success>}
'promise2' Promise{<reject>: error!!!>}
```
