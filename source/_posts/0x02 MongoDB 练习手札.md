---
title: 0x02 MongoDB 练习手札
categories:
  - MongoDB
tags:
  - MongoDB
date: 2019/12/02 20:33:23
---

## 导入数据

为了方便我们的练习节省时间，最好的方法就是使用官方提供的数据。

数据地址：[传送门](https://media.mongodb.org/zips.json)

先下载下来，然后通过以下方法导入数据

```shell
mongoimport --db=zipcodes --collection=zipcodes --file=zips.json
```

没个 item 字段：

```json
{
  "_id": "10280",
  "city": "NEW YORK",
  "state": "NY",
  "pop": 5574,
  "loc": [-74.016323, 40.710537]
}
```

- \_id 表示邮政编码
- city 表示城市名称
- state 表示城市所属州
- pop 表示人口
- loc 表示所在地的精度和维度

<!-- more -->

## 查询州人口大于 1000 万

```javascript
db.zipcodes.aggregate([
  { $group: { _id: "$state", totalPop: { $sum: "$pop" } } },
  { $match: { totalPop: { $gte: 10 * 1000 * 1000 } } }
]);
```

聚合管道会至上而下的将处理结果传递给下一个阶段处理。上面例子就是先将文档进行以 `state` 为单位进行分组处理，计算每个州的人口总和，将分组后的统计记过通过聚合管道传递给下一个\$match 进行筛选处理。

## 查询州下城市的平均人口

```javascript
db.zipcodes.aggregate([
  {
    $group: { _id: { state: "$state", city: "$city" }, pop: { $sum: "$pop" } }
  },
  { $group: { _id: "$_id.state", avgCityPop: { $avg: "$pop" } } }
]);
```

解释下：这里我们先按照州和城市进行分组并计算人口总数，在将统计的结果，通过管道传递给下一个处理条件。

1. 分解开来理解就是，第一次分组后的单条数据结果为：

```json
// { $group: { _id: { state: "$state", city: "$city" }, pop: { $sum: "$pop" } } },
[{
	"_id" : {
    "state" : "CO",
    "city" : "EDGEWATER"
  },
  "pop" : 13154
},...]
```

2. 在这个数据结果的基础上，我们在进行分组聚合

```json
// 以这个条件 { $group: { _id: "$_id.state", avgCityPop: { $avg: "$pop" } } }
[{
  "_id" : "MN",
  "avgCityPop" : 5335
}, ...]
```

可以看出来，我们的查询结果是按照传入数组至上而下的进行管道式处理。

## 查询一个州中最大和最小城市人口数据

有了上面两个案例的思路，我们可以借鉴的思路就是：

- 先把数据按州、城市进行分组，并计算人口和
- 对分组后的数据进行排序
- 再讲分组信息按州进行分组，找出人口最大城市和人口最小城市

```javascript
db.zipcodes.aggregate([
  {
    $group: {
      _id: { state: "$state", city: "$city" },
      pop: { sum: "$pop" }
    }
  },
  { $sort: { pop: 1 } }, // 1升序，-1降序
  {
    $group: {
      _id: "$_id.state",
      biggestCity: { $last: "$_id.city" },
      biggestPop: { $last: "$pop" }, // 自定义字段，取最后一条数据
      smallestCity: { $first: "$_id.city" },
      smallestPop: { $first: "$pop" } // 自定义字段，取第一条数据
    }
  },

  // 自定义返回字段，相当于我们MySQL中的 select
  {
    $project: {
      _id: 0, // 隐藏默认_id
      state: "$_id",
      biggestCity: { name: "$biggestCity", pop: "$biggestPop" },
      smallestCity: { name: "$smallestCity", pop: "$smallestPop" }
    }
  }
]);
```

只要理解聚合管道，我们就可以逐步的对数据记性逐级处理，来获取我们想要的数据
