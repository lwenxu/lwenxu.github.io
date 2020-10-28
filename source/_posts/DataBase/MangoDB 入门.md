---
title: MangoDB 入门
date: 2019-01-16 09:53:28
tags: DB MangoDB
categories:
 - 数据库
---

## 1. 设置为windows启动项

```bash
mongod -dbpath "C:\Program Files\MongoDB\Server\4.0\data" -logpath "C:\Program Files\MongoDB\Server\4.0\log\MongoDB.log" -install -serviceName "MongoDB"
```

## 2.基本概念

1. 数据库（database）:数据的仓库可以在仓库中存放集合
2. 集合（collection）：集合是数组可以在集合中存放文档
3. 文档（document）：存储操作的都是文档，文档是数据库中最小的单位。

在MongoDB中，数据库和集合都不需要手动创建，当我们创建文档时，如果文档所在的集合或数据库不存在会自动创建数据库和集合.MongoDB的文档的属性值也可以是一个文档，当一个文档的属性值是一个文档时，我们称这个文档叫做 内嵌文档

```
db.users.update({username:"sunwukong"},{$set:{hobby:{cities:["beijing","shanghai","shenzhen"] , movies:["sanguo","hero"]}}});
```



## 3.基本指令

### 1. 显示数据库

```
show dbs
show databases
```



### 2.进入到指定的数据库中

```
use 数据库名
```

​	如果我们use了一个不存在的数据库他也不会报错而是在我们第一次创建文档的时候创建这个数据库，在MongoDB中所有的数据库和集合都不需要我们去创建的 。

### 3.当前所处的数据库

```
db
```

注意一下 db 就类似于 this 只想当前的上下文，我们在后面的 crud 操作可以直接使用数据库名也可以在进入到数据库之后直接使用 db

### 4.显示数据库中所有的集合

```
show collections
```

### 5.数据库的CRUD（增删改查）的操作

#### 1.向数据库中插入文档

```
    db.<collection>.insert()  向集合中插入一个或多个文档
```

当我们向集合中插入文档时，如果没有给文档指定_id属性，则数据库会自动为文档添加  ` _id` 该属性用来作为文档的唯一标识 , ` _id`我们可以自己指定，如果我们指定了数据库就不会在添加了，如果自己指定_id 也必须确保它的唯一性

```
db.collection.insertOne()  插入一个文档对象
db.collection.insertMany()  插入多个文档对象
```

上面的是在3.2 版本中新加的操作，为了语义更加的明确。

例子：向test数据库中的，stus集合中插入一个新的学生对象
{name:"孙悟空",age:18,gender:"男"}

```
db.stus.insert({name:"孙悟空",age:18,gender:"男"})
test.stus.insert({name:“lwne”,age:12,gender:“女”})
use test;
db.stus.insert({name:"lwen",age:12})
db.stus.insert([
{name:"lwen",age:12},
{name:"lwen",age:12},
{name:"lwen",age:12}
])
db.stus.find()
```

#### 2.查询

```
    db.collection.find()
```

find()用来查询集合中所有符合条件的文档，find()可以接收一个对象作为条件参数
{} 空对象或者{ }表示查询集合中所有的文档
{属性:值} 查询属性是指定值的文档

find()返回的是一个数组

```
db.collection.findOne()
```

用来查询集合中符合条件的第一个文档  ，findOne()返回的是一个文档对象 

```
db.collection.find({}).count() 
```

查询所有结果的数量

MongoDB支持直接通过内嵌文档的属性进行查询，如果要查询内嵌文档则可以通过.的形式来匹配
如果要通过内嵌文档来对文档进行查询，此时属性名必须使用引号 

```
db.users.find({'hobby.movies':"hero"});
```

逻辑条件查询：

19.查询numbers中num大于5000的文档

```
db.numbers.find({num:{$gt:500}}); 大于500
db.numbers.find({num:{$eq:500}}); 等于500
db.numbers.find().limit(10); 查看numbers集合中的前10条数据
db.numbers.find(); 开发时，我们绝对不会执行不带条件的查询
db.numbers.find().skip(10).limit(10);
db.emp.find({sal:{$lt:2000 , $gt:1000}}); 查询工资在1000-2000之间的员工
db.emp.find({$or:[{sal:{$lt:1000}} , {sal:{$gt:2500}}]}); 查询工资小于1000或大于2500的员工
```

skip()用于跳过指定数量的数据  

```
skip((页码-1) * 每页显示的条数).limit(每页显示的条数);  数据分页
```



```
db.stus.find({_id:"hello"});
db.stus.find({age:16 , name:"白骨精"});
db.stus.find({age:28});
db.stus.findOne({age:28});
db.stus.find({}).count();
```

#### 3.修改

 db.collection.update(查询条件,新对象)

update()默认情况下会使用新对象来替换旧的对象

如果需要修改指定的属性，而不是替换需要使用“修改操作符”来完成修改
`$set` 可以用来修改文档中的指定属性
`$unset` 可以用来删除文档的指定属性

`$push` 用于向数组中添加一个新的元素
`$addToSet` 向数组中添加一个新元素 ， 如果数组中已经存在了该元素，则不会添加

```
db.users.update({username:"tangseng"},{$push:{"hobby.movies":"Interstellar"}});
db.users.update({username:"tangseng"},{$addToSet:{"hobby.movies":"Interstellar"}});
```

update()默认只会修改一个

```
  db.collection.updateMany()
```

同时修改多个符合条件的文档

```
    db.collection.updateOne()
```

修改一个符合条件的文档    

```
 db.collection.replaceOne()
```

替换一个文档

```
//替换
db.stus.update({name:"沙和尚"},{age:28});

db.stus.update(
    {"_id" : ObjectId("59c219689410bc1dbecc0709")},
    {$set:{
        gender:"男",
        address:"流沙河"
    }}    
)

db.stus.update(
    {"_id" : ObjectId("59c219689410bc1dbecc0709")},
    {$unset:{
        address:1
    }}    
)

db.stus.updateMany(
    {"name" : "猪八戒"},
    {
        $set:{
            address:"猪老庄"
        }
    }    
);
    
db.stus.update(
    {"name" : "猪八戒"},
    
    {
        $set:{
        address:"呵呵呵"
        }
    }  ,
    {
        multi:true
    }    
)
```

#### 4.删除

```
db.collection.remove()
```

删除一个或多个，可以第二个参数传递一个true，则只会删除一个

如果传递一个空对象作为参数，则会删除所有的

```
db.collection.deleteOne()
db.collection.deleteMany()
db.collection.drop() 删除集合
db.dropDatabase() 删除数据库
```

一般数据库中的数据都不会删除，所以删除的方法很少调用，一般会在数据中添加一个字段，来表示数据是否被删除

### 6.排序投影

查询文档时，默认情况是按照_id的值进行排列（升序），sort()可以用来指定文档的排序的规则,sort()需要传递一个对象来指定排序规则 1表示升序 -1表示降序

```
//limit skip sort 可以以任意的顺序进行调用
db.emp.find({}).sort({sal:1,empno:-1});
```

在查询时，可以在第二个参数的位置来设置查询结果的 投影

```
db.emp.find({},{ename:1 , _id:0 , sal:1});
```



## 4. mongoose

### 1. 连接数据库

1. 下载安装Mongoose

​        npm i mongoose --save

2. 在项目中引入mongoose

​        var mongoose = require("mongoose");

3. 连接MongoDB数据库

    mongoose.connect('mongodb://数据库的ip地址:端口号/数据库名', { useMongoClient: true});如果端口号是默认端口号（27017） 则可以省略不写

4. 断开数据库连接(一般不需要调用)

   MongoDB数据库，一般情况下，只需要连接一次，连接一次以后，除非项目停止服务器关闭，否则连接一般不会断开mongoose.disconnect()

5. 监听MongoDB数据库的连接状态

   在mongoose对象中，有一个属性叫做connection，该对象表示的就是数据库连接.通过监视该对象的状态，可以来监听数据库的连接与断开

   1. 数据库连接成功的事件

​        mongoose.connection.once("open",function(){})}

​	2. 数据库断开的事件

​        mongoose.connection.once("close",function(){});

```\
//引入
var mongoose = require("mongoose");
//连接数据库
mongoose.connect("mongodb://127.0.0.1/mongoose_test" , { useMongoClient: true});

mongoose.connection.once("open",function(){
	console.log("数据库连接成功~~~");
});

mongoose.connection.once("close",function(){
	console.log("数据库连接已经断开~~~");
});

//断开数据库连接
mongoose.disconnect();
```

### 2. Schema

这个东西可以看做我们的关系数据库的表的约束。比如哪些字段必须有，而且他的值是什么类型，默认值是什么，这样我们在语言中就不会出现类型转换错误的情况，并且在mongoose中也会帮我们去做转换。

```
var mongoose = require("mongoose");
mongoose.connect("mongodb://127.0.0.1/mongoose_test",{useMongoClient:true});
mongoose.connection.once("open",function () {
	console.log("数据库连接成功~~~");
});


//将mongoose.Schema 赋值给一个变量
var Schema = mongoose.Schema;

//创建Schema（模式）对象
var stuSchema = new Schema({

	name:String,
	age:Number,
	gender:{
		type:String,
		default:"female"
	},
	address:String

});

//通过Schema来创建Model
//Model代表的是数据库中的集合，通过Model才能对数据库进行操作
//mongoose.model(modelName, schema):
//modelName 就是要映射的集合名 mongoose会自动将集合名变成复数
var StuModel = mongoose.model("student" , stuSchema);

//向数据库中插入一个文档
//StuModel.create(doc, function(err){});
StuModel.create({
	name:"白骨精",
	age:16,
	address:"白骨洞"
},function (err) {
	if(!err){
		console.log("插入成功~~~");
	}
});
```

### 3. Model

有了Model，我们就可以来对数据库进行增删改查的操作了

​    Model.create(doc(s), [callback])  用来创建一个或多个文档并添加到数据库中

参数：

​        doc(s) 可以是一个文档对象，也可以是一个文档对象的数组

​        callback 当操作完成以后调用的回调函数

查询的：

```
Model.find(conditions, [projection], [options], [callback])  查询所有符合条件的文档 总会返回一个数组
Model.findById(id, [projection], [options], [callback])  根据文档的id属性查询文档
Model.findOne([conditions], [projection], [options], [callback])  查询符合条件的第一个文档 总和返回一个具体的文档对象
```

conditions 查询的条件

projection 投影 需要获取到的字段，两种方式

1. {name:1,_id:0}
2. "name -_id"

options  查询选项（skip limit）

​	skip:3 , limit:1}

callback 回调函数，查询结果会通过回调函数返回

​         回调函数必须传，如果不传回调函数，压根不会查询

#### 使用Model进行增删改查

```javascript
StuModel.find({name:"唐僧"},function (err , docs) {
	if(!err){
		console.log(docs);
	}
});
StuModel.find({},{name:1 , _id:0},function (err , docs) {
	if(!err){
		console.log(docs);
	}
});
StuModel.find({},"name age -_id", {skip:3 , limit:1} , function (err , docs) {
	if(!err){
		console.log(docs);
	}
});

StuModel.findOne({} , function (err , doc) {
	if(!err){
		console.log(doc);
	}
});
StuModel.findById("59c4c3cf4e5483191467d392" , function (err , doc) {
	if(!err){
		//console.log(doc);
		//通过find()查询的结果，返回的对象，就是Document，文档对象
		//Document对象是Model的实例
		console.log(doc instanceof StuModel);
	}
});StuModel.create([
	{
		name:"沙和尚",
		age:38,
		gender:"male",
		address:"流沙河"
	}

],function (err) {
	if(!err){
		console.log(arguments);
	}
});


/*
	修改
 Model.update(conditions, doc, [options], [callback])
 Model.updateMany(conditions, doc, [options], [callback])
 Model.updateOne(conditions, doc, [options], [callback])
 	- 用来修改一个或多个文档
 	- 参数：
 		conditions 查询条件
 		doc 修改后的对象
 		options 配置参数
 		callback 回调函数
 Model.replaceOne(conditions, doc, [options], [callback])
* */

StuModel.updateOne({name:"唐僧"},{$set:{age:20}},function (err) {
	if(!err){
		console.log("修改成功");
	}
});


/*
删除：
 Model.remove(conditions, [callback])
 Model.deleteOne(conditions, [callback])
 Model.deleteMany(conditions, [callback])
*/
StuModel.remove({name:"白骨精"},function (err) {
	if(!err){
		console.log("删除成功~~");
	}
});


 Model.count(conditions, [callback])
 	- 统计文档的数量的

StuModel.count({},function (err , count) {
	if(!err){
		console.log(count);
	}
});

```

### 4.document

​    Document 和 集合中的文档一一对应 ， Document是Model的实例，通过Model查询到结果都是Document

```javascript
var mongoose = require("mongoose");
mongoose.connect("mongodb://127.0.0.1/mongoose_test",{useMongoClient:true});
mongoose.connection.once("open",function () {
	console.log("数据库连接成功~~~");
});

var Schema = mongoose.Schema;

var stuSchema = new Schema({

	name:String,
	age:Number,
	gender:{
		type:String,
		default:"female"
	},
	address:String

});

var StuModel = mongoose.model("student" , stuSchema);
/*
	Document 和 集合中的文档一一对应 ， Document是Model的实例
		通过Model查询到结果都是Document
 */

//创建一个Document
var stu = new StuModel({
	name:"奔波霸",
	age:48,
	gender:"male",
	address:"碧波潭"
});

/*
	document的方法
 		Model#save([options], [fn])
 */
/*
stu.save(function (err) {
	if(!err){
		console.log("保存成功~~~");
	}
});*/

StuModel.findOne({},function (err , doc) {
	if(!err){
		/*
		 	update(update,[options],[callback])
		 		- 修改对象
		 	remove([callback])
		 		- 删除对象

		 */
		//console.log(doc);
		/*doc.update({$set:{age:28}},function (err) {
			if(!err){
				console.log("修改成功~~~");
			}
		});*/

		/*doc.age = 18;
		doc.save();*/

		/*doc.remove(function (err) {
			if(!err){
				console.log("大师兄再见~~~");
			}
		});*/


		/*
			get(name)
				- 获取文档中的指定属性值
			set(name , value)
				- 设置文档的指定的属性值
			id
				- 获取文档的_id属性值
			 toJSON() ******
			 	- 转换为一个JSON对象

			 toObject()
			 	- 将Document对象转换为一个普通的JS对象
			 		转换为普通的js对象以后，注意所有的Document对象的方法或属性都不能使用了

		 */
		//console.log(doc.get("age"));
		//console.log(doc.age);

		//doc.set("name","猪小小");
		//doc.name = "hahaha";

		//console.log(doc._id);
		//var j = doc.toJSON();
		//console.log(j);

		//var o = doc.toObject();

		//console.log(o);

		doc = doc.toObject();

		delete doc.address;

		console.log(doc._id);

	}
});
```

