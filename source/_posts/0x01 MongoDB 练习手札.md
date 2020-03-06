---
title: 0x01 MongoDB 练习手札
categories:
  - MongoDB
tags:
  - MongoDB
date: 2019/11/19 20:33:23
---

最近看 Node.js 相关的一些东西，跟着书上的教程实现了一个 Express + MongoDB + Vue.js 实验性项目，对于常年使用 LNMP 黄金搭档的我来说，不用写 SQL 语句真的是太爽了，也不用关心 MySQL 中的字段增减，真的是太幸福了。

**既然学习就肯定要动手，不动手的学习就是耍流氓！说干就干！**

**MongoDB 的基本概念存入单元**：

1. 数据库 databases
2. 集合 collection
3. 文档 document

正确的理解名词有助于我们在未来的开发和工作中方便和他人进行沟通，MongoDB 中可以存在多个数据，而数据库中可以存在多个集合，每个集合中可以存放多条文档数据。

希望大家通过练习来掌握基础 MongoDB 操作，**别光看，动手敲！** **别光看，动手敲！** **别光看，动手敲！**

<!-- more -->

1. 进入 my_practices 数据库

```
use my_practices
```

2. 查看数据库

```
show dbs
```

3. 向数据库的 user 集合插入一个文档

```javascript
// insertOne 增加语义化一些
// 在开发中，因为集合中存放了多个文档，所以集合命名我更加习惯用复数的形式
// 当然这种习惯性的东西不强制每个人，只是我的个人习惯
// 当然MongoDB查询的方法还有insert,insertMany,分别是什么可以感兴趣的可以查询文档。
db.users.insertOne({
  username: "Tom",
  age: 18,
  gender: "male"
});
```

4. 查询 user 集合中的文档

```javascript
// find中可以传递空文档{}，也可以什么都不传，表示查询全部
db.users.find({});
```

5. 向数据库的 user 集合中再插入一个文档

```javascript
db.users.insertOne({
  username: "Jerry",
  age: 17,
  gender: "male"
});
```

6. 查看当前数据库 user 集合中的文档

```
db.users.find({})
```

7. 统计数据库 user 集合中的文档数量

```
db.users.find().count()
```

8. 查询数据库 user 集合中 username 为 Tom 的文档

```javascript
// 传递要查询文档中需要满足的条件
db.users.find({
  username: "Tom"
});
```

9. 向数据库 user 集合中的 username 为 Tom 的文档，添加一个 address 属性，属性值为 Shenzhen.

```javascript
// 修改指定文档对应的属性
db.users.update(
  {
    username: "Tom"
  },
  {
    $set: { address: "Shenzhen" }
  }
);
```

10. 使用{username: 'Herry'} 替换 username 为 Jerry 的文档

```javascript
db.users.replaceOne(
  {
    username: "Jerry"
  },
  {
    username: "Herry"
  }
);
```

11. 删除 username 为 Tom 的文档的 address 属性

```javascript
// 删除指定文档对应的属性，为的是删除属性，所以属性对应的值可以随意
// 个人感觉值就是起了一个占位作用，美化语句而已
db.users.update(
  {
    username: "Tom"
  },
  {
    $unset: {
      address: ""
    }
  }
);
```

12. 向 username 为 Tom 的文档中，添加一个 hobby：{cities: ['BeiJing', 'ShangHai'], movies: ['Hero', 'King']}

```javascript
// 修改之后，查询的时候我发现在一些GUI中，hobby的属性值被标记为Document
// 也就是说我们的hobby属性保存的是一个文档，这个文档叫做内嵌文档
db.users.update(
  {
    username: "Tom"
  },
  {
    $set: {
      hobby: {
        cities: ["BeiJing", "ShangHai"],
        movies: ["Hero", "King"]
      }
    }
  }
);
```

13. 向 username 为 Herry 的文档中，添加一个 hobby: {movies: ['Ip Man 1', 'Ip Man 2']}

```javascript
db.users.update(
  {
    username: "Herry"
  },
  {
    $set: {
      hobby: {
        movies: ["Ip Man 1", "Ip Man 2"]
      }
    }
  }
);
```

14. 查看喜欢电影为 Hero 的文档

```javascript
// MongoDB 支持直接通过内嵌文档的属性查询
// 注意通过内嵌文档查询，属性名必须使用引号括起来
db.users.find({
  "hobby.movies": "Hero"
});
```

15. 向 Herry 的文档中添加新电影 Ip Man 3

```javascript
// $push 用于向数组中添加一个新的元素
// 题外话：$addToSet 和 $push 拥有相同的功能，只是前者检测到集合中有这个值就不会添加
// 而后者不管数据是否重复
db.users.update(
  {
    username: "Herry"
  },
  {
    $push: {
      "hobby.movies": "Ip Man 3"
    }
  }
);
```

16. 删除喜欢 BeiJing 的用户

```javascript
db.users.remove({
  "hobby.cities": "BeiJing"
});
```

17. 删除 user 集合

```
// 删除集合中所有的文档，效率不高
db.users.remove({})
// 更加方便的方法为
db.users.drop()
```

18. 向 numbers 集合中插入 20000 条数据

```javascript
// 这种形式，我的I7电脑都用了9.8s才完成插入
for (var i = 1; i <= 20000; i++) {
  db.numbers.insert({
    num: i
  });
}

// 这种形式则舒爽很多，仅仅使用了0.5s
var arr = [];
for (var i = 1; i <= 20000; i++) {
  arr.push({ num: i });
}
db.numbers.insert(arr);

// 项目开发中数据库的方法能少调用尽量少调用
```

19. 查询 numbers 集合中 num 为 500 的文档

```javascript
db.numbers.find({
  num: 500
});

// OR

db.numbers.find({
  num: {
    $eq: 500
  }
});

// 扩展: 不等于操作符为 $ne
```

20. 查询 numbers 中 num 大于 500 的文档

```javascript
// 大于500的
db.numbers.find({
  num: {
    $gt: 500
  }
});

// 大于等于500的
db.numbers.find({
  num: {
    $gte: 500
  }
});

// 扩展：$lt 小于 | $lte 小于等于
```

21. 查询 numbers 中 num 小于 30 的文档

```javascript
db.numbers.find({
  num: {
    $lt: 30
  }
});
```

22. 查询 numbers 中 num 大于 40 小于 50 的文档

```javascript
db.numbers.find({
  num: {
    $gt: 40,
    $lt: 50
  }
});
```

23. 查询 numbers 中 num 大于 19996 的文档

```javascript
db.numbers.find({
  num: {
    $gt: 19996
  }
});
```

24. 查看 numbers 集合中的前 10 条数据

```
db.numbers.find().limit(10)
```

25. 查看 numbers 集合中的第 11 条到 20 条数据

```
db.numbers.find().skip(10).limit(10)
```

26. 查看 numbers 集合中的第 21 条到 30 条数据

```
db.numbers.find().skip(20).limit(10)
```
