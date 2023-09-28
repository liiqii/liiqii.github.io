---
title: mongo 查询
date: 2023-09-01
# lastmod: 2023-10-08
# description: mongo 查询啊实打实大
math: true
tags: ["mongo"]
ShowBreadCrumbs: false
---

# 数据查询

## `find()`方法

**方法说明**

1.  `find()`方法没有参数时会匹配集合中的所有内容，`find({})`和`find()`功能一样
2.  `find({"_id":1})`表示查询`_id`字段为 1 的文档；`find({"_id":1,"name":"salta"})`多个筛选条件表示为`and`的关系，表示查询`_id=1 and name='salta'`的文档；增加查询条件也一样
3.  `find({},{"price":0,"_id":0})`表示从查询结果中剔除`price`和`_id`字段；`find({},{"name":1,"price":1})`表示从查询结果中只保留`name`和`price`字段。
4.  `find()`方法中的筛选条件必须是常量

\*\*注意：\*\*第二个参数中如果是保留字段，则必须都是保留字段；如果是剔除字段，则必须都是剔除字段。否则 mongodb 会报错。但`_id`字段不受此规则限制，可以随意剔除。举例如下：

```javascript
//以下代码会报错
db.game.find(
    {},
    {
        name: 1,
        price: 0,
    }
);
//以下代码能正常运行
db.game.find(
    {},
    {
        name: 1,
        _id: 0,
    }
);
```

运行效果如下：

```bash
> db.game.find({},{
...     "name":1,
...     "price":0
... })
Error: error: {
        "ok" : 0,
        "errmsg" : "Cannot do exclusion on field price in inclusion projection",
        "code" : 31254,
        "codeName" : "Location31254"
}
> db.game.find({},{
...     "name":1,
...     "_id":0
... })
{ "name" : "salta legend" }
{ "name" : "monster hunter: rise" }
{ "name" : "monster hunter: world" }
{ "name" : "bayonetta" }
{ "name" : "nier replicant" }
{ "name" : "god of war" }
{ "name" : "Nioh 2" }
```

## 条件查询

### `$lt`,`$lte`,`$gt`,`$gte`,`$ne`

1.  `$lt`小于
2.  `$lte`小于等于
3.  `$gt`大于
4.  `$gte`大于等于
5.  `$ne`不等于,`$ne`可用于任意数据类型

以下代码用来查询价格大于 20,且价格小于等于 280 的 document

```javascript
db.game.find({
    price: {
        $gt: 20,
        $lte: 280,
    },
});
```

运行效果如下:

```bash
> db.game.find({
...     "price":{
...         "$gt":20,
...         "$lte":280
...     }
... })
{ "_id" : 1, "name" : "monster hunter: world", "price" : 128, "platform" : [ "steam" ], "comments" : [ { "author" : "xilia", "content" : "hello salta", "hidden" : true }, { "author" : "salta" }, { "author" : "link" }, { "author" : "link" } ] }
{ "_id" : ObjectId("62ba596511721fa1455eb94b"), "name" : "nier replicant", "company" : "square enix", "price" : 180 }
{ "_id" : ObjectId("62bd39540a71331919b6295e"), "name" : "god of war", "price" : 30 }
```

### `$or`

`$or`运算符用于表示两个是筛选条件是或的关系,以下代码用来筛选价格大于280或者价格小于等于20的document

```javascript
db.game.find({"$or":[
    {"price":{"$gt":280}},
    {"price":{"$lte":20}}
]})
```

代码运行效果如下:

```bash
> db.game.find({"$or":[
...     {"price":{"$gt":280}},
...     {"price":{"$lte":20}}
... ]})
{ "_id" : ObjectId("62b9515ba8e0e5b80c720d94"), "name" : "salta legend", "orderDate" : ISODate("2022-06-27T06:42:35.640Z"), "price" : 327 }
{ "_id" : ObjectId("62ba565711721fa1455eb949"), "name" : "monster hunter: rise", "orderDate" : ISODate("2022-06-28T01:16:07.114Z"), "price" : 298 }
{ "_id" : ObjectId("62ba596511721fa1455eb94a"), "name" : "bayonetta", "company" : "nintendo", "price" : 318 }
```

### `$in`和`$nin`

`$in`用来筛选在某些列出的范围内的属性(和sql语句中的in关键字相同)\
`$nin`是not in的意思\
以下代码用来查询价格在\[327,30]两个数字中的结果:

```javascript
db.game.find({"price":{
    "$in":[327,30]
}})
```

代码运行效果如下:

```bash
> db.game.find({"price":{
...     "$in":[327,30]
... }})
{ "_id" : ObjectId("62b9515ba8e0e5b80c720d94"), "name" : "salta legend", "orderDate" : ISODate("2022-06-27T06:42:35.640Z"), "price" : 327 }
{ "_id" : ObjectId("62bd39540a71331919b6295e"), "name" : "god of war", "price" : 30 }
```

### `$mod`

`$mod`为[取模运算](https://so.csdn.net/so/search?q=%E5%8F%96%E6%A8%A1%E8%BF%90%E7%AE%97\&spm=1001.2101.3001.7020)符,以下代码用来筛选出 对price模3余0的document。

```javascript
db.game.find({"price":{
    "$mod":[3,0]
}})
```

代码运行效果为：

```bash
> db.game.find({"price":{
...     "$mod":[3,0]
... }})
{ "_id" : ObjectId("62b9515ba8e0e5b80c720d94"), "name" : "salta legend", "orderDate" : ISODate("2022-06-27T06:42:35.640Z"), "price" : 327 }
{ "_id" : ObjectId("62ba596511721fa1455eb94a"), "name" : "bayonetta", "company" : "nintendo", "price" : 318 }
{ "_id" : ObjectId("62ba596511721fa1455eb94b"), "name" : "nier replicant", "company" : "square enix", "price" : 180 }
{ "_id" : ObjectId("62bd39540a71331919b6295e"), "name" : "god of war", "price" : 30 }
```

### `$not`

`$not`表示取反，可用于描述其他运算符，以下代码用来筛选出 不满足对price模3余0的document

```javascript
db.game.find({"price":{
    "$not":{
        "$mod":[3,0]
    }
}})
```

代码运行效果如下：

```bash
> db.game.find({"price":{
...     "$not":{
...         "$mod":[3,0]
...     }
... }})
{ "_id" : ObjectId("62ba565711721fa1455eb949"), "name" : "monster hunter: rise", "orderDate" : ISODate("2022-06-28T01:16:07.114Z"), "price" : 298 }
{ "_id" : 1, "name" : "monster hunter: world", "price" : 128, "platform" : [ "steam" ], "comments" : [ { "author" : "xilia", "content" : "hello salta", "hidden" : true }, { "author" : "salta" }, { "author" : "link" }, { "author" : "link" } ] }
{ "_id" : ObjectId("62bd3d9f0a71331919b6297a"), "name" : "Nioh 2", "published" : ISODate("2022-06-30T06:08:30.014Z") }

```

## 类型查询

### 关于null类型的说明

当查询条件设置为`{"platform":null}`时，既可以查到`platform=null`的文档，也可以查到不存在`platform`字段的文档。如果只希望得到`platform=null`的文档，可以通过`$exits`条件确定该字段存在。

以下代码用来筛选出`platform`字段存在，并且为空的文档：

```javascript
db.game.find({"platform":{
    "$eq":null,
    "$exists":true
}})
```

代码运行效果如下：

```bash
> db.game.insertMany([{name:"it takes two",platform:null},{name:"naraka",platform:""},{name:"gta5"}])
{
        "acknowledged" : true,
        "insertedIds" : [
                ObjectId("62be8e82718b6cda80d080aa"),
                ObjectId("62be8e82718b6cda80d080ab"),
                ObjectId("62be8e82718b6cda80d080ac")
        ]
}
> db.game.find({"platform":{
...     "$eq":null,
...     "$exists":true
... }})
{ "_id" : ObjectId("62be8e82718b6cda80d080aa"), "name" : "it takes two", "platform" : null }
```

如果只进行null筛选，效果如下:

```bash
> db.game.find({"platform":null})
{ "_id" : ObjectId("62b9515ba8e0e5b80c720d94"), "name" : "salta legend", "orderDate" : ISODate("2022-06-27T06:42:35.640Z"), "price" : 327 }
{ "_id" : ObjectId("62ba565711721fa1455eb949"), "name" : "monster hunter: rise", "orderDate" : ISODate("2022-06-28T01:16:07.114Z"), "price" : 298 }
{ "_id" : ObjectId("62ba596511721fa1455eb94a"), "name" : "bayonetta", "company" : "nintendo", "price" : 318 }
{ "_id" : ObjectId("62ba596511721fa1455eb94b"), "name" : "nier replicant", "company" : "square enix", "price" : 180 }
{ "_id" : ObjectId("62bd39540a71331919b6295e"), "name" : "god of war", "price" : 30 }
{ "_id" : ObjectId("62bd3d9f0a71331919b6297a"), "name" : "Nioh 2", "published" : ISODate("2022-06-30T06:08:30.014Z") }
{ "_id" : ObjectId("62be8e82718b6cda80d080aa"), "name" : "it takes two", "platform" : null }
{ "_id" : ObjectId("62be8e82718b6cda80d080ac"), "name" : "gta5" }
```

### [正则表达](https://so.csdn.net/so/search?q=%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE\&spm=1001.2101.3001.7020)式

`$regex`运算符用来在查询中对字符串进行正则表达时匹配，正则表达式也遵循JavaScript的正则表达式语法。如，以下代码用来查询name字段以n或N开头的文档：

```javascript
db.game.find({"name":{
    "$regex":/^n/i
}})
```

查询结果如下：

```bash
> db.game.find({"name":{
...     "$regex":/^n/i
... }})
{ "_id" : ObjectId("62ba596511721fa1455eb94b"), "name" : "nier replicant", "company" : "square enix", "price" : 180 }
{ "_id" : ObjectId("62bd3d9f0a71331919b6297a"), "name" : "Nioh 2", "published" : ISODate("2022-06-30T06:08:30.014Z") }
{ "_id" : ObjectId("62be8e82718b6cda80d080ab"), "name" : "naraka", "platform" : "" }
```

### 数组查询

测试用法之前先新增数据：

```javascript
db.game.updateOne(
    {name:"naraka"},
    {$set:[
        "it is so good",
        "it is so pretty",
        "i am a old gamer",
        "it`s my like",
        "i like sanniang"
    ]}
)
```

**数组内容匹配方式：**

1.  完全匹配整个数组

    `db.game.find({"comments":["abc","def"]})`用来筛选出数组`comments`字段值为`['abc','def']`的文档，这种用法和精确匹配完全一样。

2.  匹配单个数组元素

    `db.game.find({"comments":"it is so good"})`用来筛选出数组`comments`字段值中包含`it is so good`元素的文档

3.  匹配多个数组元素，`$all`

    `db.game.find({"comments":{"all":["it is so good","i am a old gamer"]}})`用来筛选出数组元素中包含`it is so good`和`i am a old gamer`两个元素的文档。如果`$all`的数组中只有一个元素，那么使用`$all`和不适用`$all`结果一样，即：`{"author":{"$all":["apple"]}}`和`{"author":"apple"}`的筛选结果完全一样

4.  匹配数组长度

    `$size`运算符用于匹配数组长度，但`$size`只能是常数，且`$size`不能和`$lt`等比较运算符同时使用。如果需要比较数组的长度，可以单独增加一个`size`字段，每次修改数组同时修改这个`size`字段，这在mongoDB中很容易实现。

    `db.game.find({"comments":{"$size":3}})`用于筛选`comments`数组长度为3的文档。

5.  匹配数组元素对象的属性

    `db.game.find({"comments.author":"salta"})`用于筛选`comments`数组中元素的`author`属性为salta的文档

**控制返回值数组的长度**

过滤返回值中显示的数组长度可以使用`$slice`修饰符，整数n表示返回前n条数据，负数n表示返回后n条数据。以下代码用于返回`comments`字段的前3个元素：

```javascript
db.game.find(
    {"name":"naraka"},
    {"comments":{"$slice":3}}
)
```

运行效果如下：

```bash
> db.game.find(
...     {"name":"naraka"},
...     {"comments":{"$slice":3}}
... )
{ "_id" : ObjectId("62be8e82718b6cda80d080ab"), "name" : "naraka", "platform" : "", "comments" : [ "it is so good", "it is so pretty", "i am a old gamer" ] }
```

此方法也可以返回数组中的一段数据，以下代码会返回`comments`字段的第2\~第4个元素（忽略前1个元素，从第2个开始返回3个元素）

```javascript
db.game.find(
    {"name":"naraka"},
    {"comments":{"$slice":[1,3]}}
)
```

显示效果如下：

```bash
> db.game.find(
...     {"name":"naraka"},
...     {"comments":{"$slice":[1,3]}}
... )
{ "_id" : ObjectId("62be8e82718b6cda80d080ab"), "name" : "naraka", "platform" : "", "comments" : [ "it is so pretty", "i am a old gamer", "it`s my like" ] }
```

**只返回数组中匹配到的第一个元素**

可以通过`$$`运算符来返回匹配到的元素，这种方法只能返回匹配到的第一个元素，返回值的数组中只有一个元素。以下代码用于筛选`comments`数组字段中的元素对象的`author`属性，返回值中只有`_id`和`comments`字段，且`comments`字段中只有匹配到的第一个元素。

```javascript
db.game.find(
	{"comments.author":"salta"},
    {"comments.$":1}
)
```

代码运行效果如下：

```bash
> db.game.find(
... {"comments.author":"salta"},
...     {"comments.$":1}
... )
{ "_id" : 1, "comments" : [ { "author" : "salta" } ] }
{ "_id" : ObjectId("62be8e82718b6cda80d080ab"), "comments" : [ { "author" : "salta", "content" : "i am salta" } ] }
```

### 数组与范围查询

有以下数据：

```javascript
db.number.insertMany([
    {"num":6},
	{"num":66},
	{"num":666},
	{"num":[6,66]},
	{"num":[6,666]},
	{"num":[6,66,666]}
])
```

有以下查询代码：

```javascript
db.number.find({"num":{
    "$gt":10,
    "$lt":100
}})
```

此代码目的是查找 10 < n u m < 100 10\<num<100 10\<num<100的数据，期望能返回的数据为：

```json
{"num":66},
```

但，实际返回的数据为：

```json
{"num" : 66 }
{"num" : [ 6, 66 ] }
{"num" : [ 6, 666 ] }
{"num" : [ 6, 66, 666 ] }
```

因为，在数组的比较中，满足`6<100` 和`666>66>10`，所以实际返回值和期望返回值产生了差别，这就使得针对数组的范围查询失效。

这种现象可以通过`$elemMatch`表达式解决，但是`$elemMatch`表达式只能匹配数组元素，对于非数组元素回被跳过。效果如下：

```bash
> db.number.find().pretty()
{ "_id" : ObjectId("62beb40a718b6cda80d080ad"), "num" : 6 }
{ "_id" : ObjectId("62beb40a718b6cda80d080ae"), "num" : 66 }
{ "_id" : ObjectId("62beb40a718b6cda80d080af"), "num" : 666 }
{ "_id" : ObjectId("62beb40a718b6cda80d080b0"), "num" : [ 6, 66 ] }
{ "_id" : ObjectId("62beb40a718b6cda80d080b1"), "num" : [ 6, 666 ] }
{
        "_id" : ObjectId("62beb40a718b6cda80d080b2"),
        "num" : [
                6,
                66,
                666
        ]
}
> db.number.find({"num":{"$elemMatch":{"$gt":10,"$lt":100}}})
> //查询结果为空。可见，{"num":66}未参与筛选
```

对于以上现象，可以通过对`num`字段添加索引，之后使用min和max方法将筛选条件的索引范围限制在最大和最小值之间的形式予以解决。具体代码如下：

```javascript
//已对num字段创建索引
db.number.find({"num":{
    "$gt":10,
    "$lt":100
}}).min({"num":10}).max({"num":100})
```

### 内嵌文档查询

如果想要通过以下方式查询一个子文档，那么这个被查询必须精确的匹配整个文档（包括各个属性的顺序）。即，如果文档的内容为`{"author":"xilia","content":"hello salta"}`，则能匹配成功；但是如果文档中存在更多其他的属性，如`{"author":"xilia","content":"hello salta","hidden":true}`则这种匹配方式不能查询出对应的结果。

```javascript
db.game.find({"comments":{
    "author":"xilia",
    "content":"hello salta"
}})
```

更重要的一点是，除了需要匹配整个文档之外，这种查询方式还是强顺序相关的，两个条件的顺序必须符合要求才能成功匹配。

在这中情况下，需要使用**点调用**<sup><a href="https://blog.csdn.net/qq_43408971/article/details/125608311#fn1" id="fnref1" target="_self">1</a></sup>的方式

```javascript
db.game.find({
    "comments.author":"xilia",
    "comments.content":"hello salta"
})
```

效果如下：

```bash
> db.game.find({"comments":{
...     "author":"xilia",
...     "content":"hello salta"
... }})
> db.game.find({
...     "comments.author":"xilia",
...     "comments.content":"hello salta"
... })
{ "_id" : 1, "name" : "monster hunter: world", "price" : 128, "platform" : [ "steam" ], "comments" : [ { "author" : "xilia", "content" : "hello salta", "hidden" : true }, { "author" : "salta" }, { "author" : "link" }, { "author" : "link" } ] }
```

## `$where`查询

对于有些特殊的查询，无法通过以上任何一种方式表示出来，mongoDB提供了`$where子句`这种查询方式。`$where子句`允许在查询中执行JavaScript代码，这样就能在JavaScript代码中执行某些特殊的比较逻辑。但是为了安全和效率的要求，应该尽量减少或消除使用`$where`，并且禁止普通用户使用。

在`$where`查询过程中，每个文档都必须从`BSON`转换成`JavaScript对象`，而且`$where`查询也不能使用索引，所以有很大的性能问题。

具体使用方法如下：

```javascript
/**
 * 查询文档中存在两个字段满足以下条件：
 * 1.其中一个字段是日期类型，并且时间在当前日期之前；
 * 2.另一个字段是数字类型，并且这个数字小于300。
 */
db.game.find({"$where":function(){
    for(var param_1 in this){
        for(var param_2 in this){
            return this[param_1]<new Date() && this[param_2]<300;
        }
    }
    return false;
}}).pretty()
```

执行效果如下：

```bash
> db.game.find({"$where":function(){
...     for(var param_1 in this){
...         for(var param_2 in this){
...             return this[param_1]<new Date() && this[param_2]<300;
...         }
...     }
...     return false;
... }}).pretty()
{
        "_id" : 1,
        "name" : "monster hunter: world",
        "price" : 128,
        "platform" : [
                "steam"
        ],
        "comments" : [
                {
                        "author" : "xilia",
                        "content" : "hello salta",
                        "hidden" : true
                },
                {
                        "author" : "salta"
                },
                {
                        "author" : "link"
                },
                {
                        "author" : "link"
                }
        ]
}
```

## 游标

### 游标基础

`find()`方法的返回值是一个游标对象，通过对游标对象的各种操作，可以对查询结果进行输出控制。（在mongoDBshell中执行`find()`方法的打印操作是shell自动执行的结果）游标对象提供了`hasNex()`和`next()`两个方法，典型的使用方式为：

```javascript
var cursor=db.game.find()
while(cursor.hasNext()){
    var item=cursor.next()
    print(item.name)
}
```

同时游标对象也实现了迭代器接口，可以在`forEach()`方法中进行操作，使用方式如下：

```javascript
var cursor=db.game.find();
cursor.forEach(res=>{
    print(res.name)
})
```

**注意**：当调用`find()`方法时，shell并不会立即执行查询操作，而是真正开始获取结果时才执行查询操作。所以可以在查询之前进行一些额外的操作，如：排序、分页、限制长度等，这些操作的返回值仍然是一个游标对象。

因此，以下三行代码返回的数据相同：

```javascript
db.game.find().sort({"price":1}).limit(3).skip(2).pretty();
db.game.find().sort({"price":1}).skip(2).limit(3).pretty();
db.game.find().limit(3).sort({"price":1}).skip(2).pretty();
db.game.find().limit(3).skip(2).sort({"price":1}).pretty();
db.game.find().skip(2).limit(3).sort({"price":1}).pretty();
db.game.find().skip(2).sort({"price":1}).limit(3).pretty();
```

当`hasNext()`方法被调用后，查询条件会发送到服务端，之后shell会获取一段查询结果（这里的一段是100个结果或4MB数据中的较小值），在客户端遍历完这些数据后，shell会再次链接数据库，使用`getMore`获取更多数据。

### `limit()`、`skip()`、`sort()`

这三个函数的使用方法如下：

```javascript
//限制只返回三个结果
db.game.find().limit(3)

//跳过前三条数据
db.game.find().skip(3)

//按价格升序排序，-1表示降序
db.game.find().sort({"price":1})
```

以上三个方法可以结合使用。对于一个字段的数据包含多种类型时，mongoDB预设了排序顺序，从最小值到最大值顺序为：

|  序号 |   类型  |
| :-: | :---: |
|  1  |  最小值  |
|  2  |  null |
|  3  |   数字  |
|  4  |  字符串  |
|  5  | 对象、文档 |
|  6  |   数组  |
|  7  | 二进制数据 |
|  8  |  对象ID |
|  9  |   布尔  |
|  10 |   日期  |
|  11 |  时间戳  |
|  12 | 正则表达式 |
|  13 |  最大值  |

**注意**：当`skip()`方法跳过大量数据时会产生性能问题，所以对于大数据量的分页操作，通常不用`skip()`方法实现。假设有一个需求，需要使用`create_date`字段降序排列，可以通过如下方式操作：

```javascript
var page_1=db.game.find().sort({"create_date":-1}).limit(20);
var latest=null;
while(page_1.hasNext()){
    latest=page_1.next();
    //do sth
}
var page_2=db.game.find({
    "create_date":{"$lt":latest.create_date}
}).sort({"create_date":-1}).limit(100)
```

### 游标的生命周期

一个游标会占用服务器的资源，在满足一定的条件之后，服务端会释放这个游标的资源，这些条件包括：

1.  游标遍历完所有的结果
2.  客户端要求终止该游标
3.  当游标超出客户端的作用域，驱动程序会向数据库发送一条 “可以销毁该游标” 的消息
4.  一个游标如果10分钟内没有被使用，数据库会自动销毁

