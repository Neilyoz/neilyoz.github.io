---
title: 实现React布局路由
date: 2020-03-13 20:44:18
categories:
  - JavaScript
tags:
  - React
---

## 前言

疫情在家，本来开年就要入职的公司，入职时间是一拖再拖，这里真的是吐槽一下。在家呆着爸妈鼻子不是鼻子眼睛不是眼睛的，我心里也是日了狗，从新投简历，某些傻缺HR，一来就问你会Yii吗？我就纳闷了，现在的框架哪个不是容器搞一通。

哎，抱怨没用，动手最实际，朋友要我帮忙做个管理后台，用React帮他做，不得不说一个小白尽然也知道React了。这个前端框架的知名度不得不说。我图省事儿样式让我写不可能的用Ant Design吧。



## 搭架子过程

反正就是一堆命令什么

```shell
mkdir manager_beta && cd manager_beta
npx create-react-app .
```

漫长的等待...期间我打开了书，看了看`JavaScript 设计模式`，架子是搭建完了，开始`antd`的安装

```shell
npm i --save antd
npm i --save react-router-dom
```

默认的`create-react-app`下面不是不会暴露出配置文件的，所以还要执行一下命令，为什么还要配置，作为开发人员，配置都不自己控制，总是不安心的。好吧，其实我是要用`less`这东西得配置吖。

```shell
npm run eject
npm i --save less less-loader
```

接下来就是把下面这段配置放入`config\webpack.config.js`中了

```javascript
// line: 50行左右
const lessRegex = /\.(less)$/;
const lessModuleRegex = /\.module\.(less)$/;

// line: 487行左右
{
    test: lessRegex,
        exclude: lessModuleRegex,
            use: getStyleLoaders({ importLoaders: 3 }, 'less-loader'),
},
{
    test: lessModuleRegex,
    use: getStyleLoaders(
        {
            importLoaders: 3,
            modules: true,
            getLocalIdent: getCSSModuleLocalIdent,
            modifyVars:{},
            javascriptEnabled: true,
        },
        'less-loader'
    ),
},
```

其实上面写的对我要说的怎么 `实现React布局路由` 有什么关系呢？好吧，我告诉你，一点关系都没有哈哈哈，我就是骗骗字数。

<!-- more -->

其实这个路由实现很简单，无非就是对`react-router-dom`的一个简单封装。用代码举例子吧。

```jsx
import React from 'react';
import { Route, Switch } from 'react-router-dom';

const LayoutRoute = ({ component: Component, layout: Layout, ...rest}) => {
    return (
        <Route {...rest} render={props => (
			<Layout>
                <Component {...props} />
            </Layout>
        )} />
    );
}

const MainLyout = props => (
    <div>
        <h1>Main</h1>
        {props.children}
    </div>
);

const OtherLyout = props => (
    <div>
        <h1>Other</h1>
        {props.children}
    </div>
);

const PgOne = () => {
    return (<p>PgOne</p>);
}

const XiaoluLi = () => {
    return (<p>XiaoluLi</p>)
}

const App = () => (
    <div>
        <Switch>
        	<LayoutRoute exact path="/pgone" layout={MainLyout} component={PgOne}/>
            <LayoutRoute exact path="/xiaoluli" layout={OtherLyout} component={XiaoluLi}/>
        </Switch>
    </div>
)

ReactDOM.render(
  <App />,
  document.getElementById('root')
);
```

其实就是利用了组件props传值的方式实现了这个方法，扩展了`Route`。



## 总结

希望疫情快点过去，还有就是世界和平，别再来什么幺蛾子了。还有就是求工作吖。