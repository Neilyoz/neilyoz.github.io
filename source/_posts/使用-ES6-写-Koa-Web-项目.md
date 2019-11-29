---
title: 使用 ES6 写 Koa Web 项目
date: 2019-11-29 22:27:41
categories:
  - Node.JS
tags:
  - Node.js
  - ES6
  - Babel
  - koa
---

我们 `node.js` 只是实现了部分 ES6 的语法，所以为了让我们 ES6 的代码能 `100%` 在 node.js 下执行，必须使用我们的 `babel` 把 ES6 代码编译成 nodejs 可执行的代码。

- node 环境：v12.13.1

## 开发环境搭建

首先，我们建立一个文件 `my_koa`

```shell
mkdir my_koa && cd my_koa
npm init -y
```

在我们的 `my_koa` 文件夹下就会产生一个 `package.json` 文件使我们的包配置文件，内容如下：

```json
{
  "name": "my_koa",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "Neilyoz",
  "license": "ISC"
}
```

然后安装我们的依赖

```shell
npm i --save-dev @babel/cli @babel/core @babel/node @babel/preset-env @babel/register eslint
```

安装完成后 `package.json` 如下：

```json
{
  "name": "my_koa",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "Neilyoz",
  "license": "ISC",
  "devDependencies": {
    "@babel/cli": "^7.7.4",
    "@babel/core": "^7.7.4",
    "@babel/node": "^7.7.4",
    "@babel/preset-env": "^7.7.4",
    "@babel/register": "^7.7.4",
    "eslint": "^6.7.1"
  }
}
```

<!-- more -->

我的习惯是将代码放到 `my_koa\src` 目录下进行管理，所以执行以下命令创建

```shell
mkdir src
touch index.js
```

归功于 `npm` 包管理的便利性，到这里已经成功了一半了。接下来就是要安装我们的主角 `koa` 了

```shell
npm i koa koa-router
```

`npm` 完成安装后，此时我们的 `package.json` 如下：

```json
{
  "name": "my_koa",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "Neilyoz",
  "license": "ISC",
  "devDependencies": {
    "@babel/cli": "^7.7.4",
    "@babel/core": "^7.7.4",
    "@babel/node": "^7.7.4",
    "@babel/preset-env": "^7.7.4",
    "@babel/register": "^7.7.4",
    "eslint": "^6.7.1"
  },
  "dependencies": {
    "koa": "^2.11.0",
    "koa-router": "^7.4.0"
  }
}
```

到这里我们需要的基础包就已经安装完成了。可以开始我们的编码部分。

打开 `src/index.js` 文件输入一下内容：

```javascript
import Koa from "koa";
import KoaRouter from "koa-router";

const app = new Koa();
const router = new KoaRouter();

router.get("/", async ctx => {
  ctx.body = "Index";
});

app.use(router.routes()).use(router.allowedMethods());

app.listen(3000);
```

直接运行 `node ./src/index.js` ，我们会看到是报错的。因为这里的编码并没有编译成为 nodejs 认识的形式。

一般我们开发环境可以直接使用 `babel-node ./src/index.js` 来运行代码，因为有安装 `@babel/node` 为了方便，我一般会在 `package.json` 中定义一个命令来直接运行，代码如下：

```json
{
  "name": "my_koa",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "dev": "babel-node ./src/index.js"
  },
  "keywords": [],
  "author": "Neilyoz",
  "license": "ISC",
  "devDependencies": {
    "@babel/cli": "^7.7.4",
    "@babel/core": "^7.7.4",
    "@babel/node": "^7.7.4",
    "@babel/preset-env": "^7.7.4",
    "@babel/register": "^7.7.4",
    "eslint": "^6.7.1"
  },
  "dependencies": {
    "koa": "^2.11.0",
    "koa-router": "^7.4.0"
  }
}
```

要执行代码的时候只需要

```shell
npm run dev
```

可是开发环境修改完代码，总是要运行命令重新加载代码。那么怎么才能让代码保存，程序自动加载代码呢？

我使用 `nodemon` 来实现，首先我们要全局安装 `nodemon`

```shell
npm i -g nodemon
```

并且在 `my_koa` 的根目录下创建 `nodemon.json` 文件来保存 `nodemon` 运行需要的配置。

`nodemon.json` 文件内容如下：

```
{
  "exec": "npm run dev",
  "watch": ["src/*"],
  "ext": "js, html, css, json"
}
```

这时我们还需要在 `package.json` 加入一个自定义命令，如下：

```json
{
  "name": "my_koa",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "dev": "babel-node ./src/index.js",
    "watch": "nodemon"
  },
  "keywords": [],
  "author": "Neilyoz",
  "license": "ISC",
  "devDependencies": {
    "@babel/cli": "^7.7.4",
    "@babel/core": "^7.7.4",
    "@babel/node": "^7.7.4",
    "@babel/preset-env": "^7.7.4",
    "@babel/register": "^7.7.4",
    "eslint": "^6.7.1"
  },
  "dependencies": {
    "koa": "^2.11.0",
    "koa-router": "^7.4.0"
  }
}
```

这个时候我们执行 `npm run watch` 再修改 `./src/index.js` 的代码，代码保存就可以自动加载了。

## 生产环境代码生成

因为`src` 中的代码还是开发环境的代码，并没有通过我们的 babel 进行编译转换，所以直接使用 node 是可能无法启动的，我们需要配置 `.babelrc` babel 的配置文件，来告诉 `babel` 具体要做哪些编译。

`.babelrc` 如下：

```json
{
  "presets": [
    [
      "@babel/preset-env",
      {
        "targets": {
          "node": true
        }
      }
    ]
  ]
}
```

然后在 `package.json` 中定义一个 `build` 命令，内容如下：

```json
{
  "name": "my_koa",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "build": "babel src --out-dir dist",
    "dev": "babel-node ./src/index.js",
    "watch": "nodemon"
  },
  "keywords": [],
  "author": "Neilyoz",
  "license": "ISC",
  "devDependencies": {
    "@babel/cli": "^7.7.4",
    "@babel/core": "^7.7.4",
    "@babel/node": "^7.7.4",
    "@babel/preset-env": "^7.7.4",
    "@babel/register": "^7.7.4",
    "eslint": "^6.7.1"
  },
  "dependencies": {
    "koa": "^2.11.0",
    "koa-router": "^7.4.0"
  }
}
```

执行我们 `npm run build` 就会在 `my_koa` 产生 `dist` 目录，这个目录里面的代码就是我们应用于生产环境的编译代码。
