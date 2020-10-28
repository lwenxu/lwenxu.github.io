---
title: ECMAScript 从入门到精通
date: 2019-01-10 08:01:28
tags: JavaScript
categories:
 - FrontEnd
---

## 1. es5

### 1. 严格模式

#### 1.理解:

  * 除了正常运行模式(混杂模式)，ES5添加了第二种运行模式："严格模式"（strict mode）。
  * 顾名思义，这种模式使得Javascript在更严格的语法条件下运行
#### 2.目的/作用

   * 消除Javascript语法的一些不合理、不严谨之处，减少一些怪异行为
   * 消除代码运行的一些不安全之处，为代码的安全运行保驾护航
   * 为未来新版本的Javascript做好铺垫
#### 3.使用

  * 在全局或函数的第一条语句定义为: 'use strict';
  * 如果浏览器不支持, 只解析为一条简单的语句, 没有任何副作用
#### 4.语法和行为改变

* 必须用var声明变量
* 禁止自定义的函数中的this指向window，也就是不允许使用全局函数
* 创建eval作用域，本来的eval中定义的变量直接做变量提升为全局的作用域。可能会被攻击。
* 对象不能有重名的属性

### 2.object的拓展方法

ES5给Object扩展了一些静态方法, 常用的2个:
#### 1.Object.create(prototype, [descriptors])

  * 作用: 以指定对象为原型创建新的对象

  * 为新的对象指定新的属性, 并对属性进行描述
    - value : 指定值
    - writable : 标识当前属性值是否是可修改的, 默认为false
    - configurable: 标识当前属性是否可以被删除 默认为false
    - enumerable： 标识当前属性是否能用for in 枚举 默认为false

    ```javascript
    var obj = {name : 'curry', age : 29}
    obj1 = Object.create(obj, {
            sex : {
                value : '男',
                writable : true
            }
        });
    ```

    ![](http://images.heniankj.com/20190110082907.png)

    obj1 的原型对象就是 obj，并企里面有一个 sex 值，但是要注意设置 sex的一些属性，比如现在 sex 就是只能够读写，不能被删除，也不能采用 for in 遍历。
#### 2.Object.defineProperties(object, descriptors)

  * 作用: 为指定对象定义扩展多个属性
    * get ：用来获取当前属性值得回调函数
    * set ：修改当前属性值得触发的回调函数，并且实参即为修改后的值
   * 存取器属性：setter,getter一个用来存值，一个用来取值

```javascript
var obj2 = {
    firstName : 'curry',
    lastName : 'stephen'
};
Object.defineProperties(obj2, {
    fullName : {
        get : function () {
            return this.firstName + '-' + this.lastName
        },
        set : function (data) {
            var names = data.split('-');
            this.firstName = names[0];
            this.lastName = names[1];
        }
    }
});
```

![](http://images.heniankj.com/20190110083053.png)

可以看到新设置的 fullName 属性和其他的是不一样的，因为我们设置了 get 和 set 那么他就会惰性求值，只有我们在调用这个属性的时候会调用 get 方法求出元素的值，set 也是在设置值的时候才会调用。如果没有 get 就获取不到值，没有set则不能设置值。

除了上面这种设置 get 和 set 的方式之外还可以使用对象本身的方法：

1. get propertyName(){} 用来得到当前属性值的回调函数
2. set propertyName(){} 用来监视当前属性值变化的回调函数

```javascript
var obj = {
    firstName : 'kobe',
    lastName : 'bryant',
    get fullName(){
        return this.firstName + ' ' + this.lastName
    },
    set fullName(data){
        var names = data.split(' ');
        this.firstName = names[0];
        this.lastName = names[1];
    }
};
```

代码同理。

### 3. Array 拓展

1. Array.prototype.indexOf(value) : 得到值在数组中的第一个下标
2. Array.prototype.lastIndexOf(value) : 得到值在数组中的最后一个下标
3. Array.prototype.forEach(function(item, index){}) : 遍历数组
4. Array.prototype.map(function(item, index){}) : 遍历数组返回一个新的数组，返回加工之后的值
5. Array.prototype.filter(function(item, index){}) : 遍历过滤出一个新的子数组， 返回条件为true的值

在原型上定义了这么多方法，意味着 array 的实例都可以调用这些方法。

### 4. 函数拓展

#### 1.Function.prototype.bind(obj) 

作用: 将函数内的this绑定为obj, 并将函数返回

#### 2.区别bind()与call()和apply()?

都能指定函数中的this，但是call()/apply()是立即调用函数而bind()是将函数返回。所以我们的 bind 一般是用来指定回调函数的 this。

## 2. es6

### 1. let

#### 1.作用:

  * 与var类似, 用于声明一个变量

#### 2.特点:

  * 在块作用域内有效，以前只有文件和函数作用域，这个东西有块级作用域
  * 不能重复声明，否则会报错
  * 不会预处理, 不存在提升

#### 3.应用:

  * 循环遍历加监听
  * 使用let取代var是趋势

### 2. const

#### 1.作用:

定义一个常量

#### 2.特点:

不能修改其它特点同let

#### 3.应用:

保存不用改变的数据

### 3. 结构赋值

#### 1.理解:

从对象或数组中提取数据, 并赋值给变量(多个)

#### 2.对象的解构赋值

```
let {n, a} = {n:'tom', a:12}
```

#### 3.数组的解构赋值

```
let [a,b] = [1, 'atguigu'];
```

#### 4.用途

给多个形参赋值

#### 5. 例子

对象

```javascript
    let obj = {name : 'kobe', age : 39,site:{github:'aaa',weibo:'bbb'}};
    //对象的解构赋值
    let {age,site:{github}} = obj;
    console.log(age,github);
```

数组

```javascript
let arr = ['abc', 23, true];
let [, , c, d] = arr;
console.log(c, d); //按照下标寻找
```

参数解构

```javascript
function person1({name, age}) {
    console.log(name, age);
}
person1(obj);
```

### 4.模板字符串

 简化字符串的拼接,模板字符串必须用 `` 包含, 变化的部分使用${xxx}定义

```javascript
console.log(`我叫:${obj.name}, 我的年龄是：${obj.age}`);
```

### 5.对象的简写方法

#### 1.同名属性省略名字

#### 2.省略function关键字

```javascript
let x = 1;
let y = 2;
let point = {
  x,
  y,
  setX (x) {this.x = x}
};
```

### 6.箭头函数

#### 1.作用: 定义匿名函数

#### 2.基本语法:

* 没有参数: () => console.log('xxxx')
* 一个参数: i => i+2
* 大于一个参数: (i,j) => i+j
* 函数体不用大括号: 默认返回结果
* 函数体如果有多个语句, 需要用{}包围，若有需要返回的内容，需要手动返回

#### 3.使用场景: 多用来定义回调函数

#### 4.箭头函数的特点：

1. 简洁
2. 箭头函数没有自己的this，箭头函数的this不是调用的时候决定的，而是在定义的时候处在的对象就是它的this
3. 扩展理解： 箭头函数的this看外层的是否有函数，如果有，外层函数的this就是内部箭头函数的this，如果没有，则this是window。

```javascript
// 这个this是window因为内层的箭头函数往外找找到有函数，则就看他的this，而他往外找找不到说明就是window
let obj = {
    name : 'kobe',
    age : 39,
    getName : () => {
        btn2.onclick = () => {
            console.log(this);//obj
        };
    }
};
// 这个this是window因为内层的箭头函数往外找找到有函数，则就看他的this，而他是一个普通函数就看是谁调用他了。如果是obj调用的话 this 就是 obj
let obj = {
    name : 'kobe',
    age : 39,
    getName : () => {
        btn2.onclick = () => {
            console.log(this);//obj
        };
    }
};
```

### 7. 打包解包运算符

#### 1.rest(可变)参数

用来取代arguments 但比arguments灵活,他只能作为最后部分形参参数，也就是打包最后一部分形参

```javascript
function add(...values) {
    let sum = 0;
    for(value of values) {
    sum += value;
    }
    return sum;
}
```

#### 2.扩展运算符

```javascript
let arr1 = [1,3,5];
let arr2 = [2,...arr1,6];
arr2.push(...arr1);
```

### 8. 形参默认值

```javascript
function(a=1){
    
}
```

### 9. promise

#### 1.理解:

​	Promise对象: 代表了未来某个将要发生的事件(通常是一个异步操作)，有了promise对象, 可以将异步操作以同步的流程表达出来, 避免了层层嵌套的回调函数(俗称'回调地狱')，ES6的Promise是一个构造函数, 用来生成promise实例。

#### 2.使用promise基本步骤(2步):

  * 创建promise对象

    ```
    let promise = new Promise((resolve, reject) => {
        //初始化promise状态为 pending
      //执行异步操作
      if(异步操作成功) {
        resolve(value);//自动修改promise的状态为 fullfilled ，设置了promise的状态以后然后调用then或者catch，根据promise的状态执行对应的then或者catch内容。或者说在then里面有两个函数，分别是成功和失败，他会地总调用。 里面的value 就就会传递给then里面的回调成功的入参。
      } else {
        reject(errMsg);//自动修改promise的状态为 rejected
      }
    })
    ```

  * 调用promise的then()

    ```
    promise.then(function(
      result => console.log(result), //这个result就是resolve里面的参数
      errorMsg => alert(errorMsg)
    ))
    ```

#### 3.promise对象的3个状态

  * pending: 初始化状态
  * fullfilled: 成功状态
  * rejected: 失败状态

#### 4.应用:

  * 使用promise实现超时处理

  * 使用promise封装处理ajax请求
    let request = new XMLHttpRequest();
    request.onreadystatechange = function () {
    }
    request.responseType = 'json';
    request.open("GET", url);
    request.send();

#### 5. 流程

![](http://images.heniankj.com/20190110093914.png)

### 10. symbol

ES5中对象的属性名都是字符串，容易造成重名，污染环境

#### 1.概念：

ES6中的添加了一种原始数据类型symbol(已有的原始数据类型：String, Number, boolean, null, undefined, 对象)

#### 2.特点：

​    1、Symbol属性对应的值是唯一的，解决命名冲突问题
​    2、Symbol值不能与其他数据进行计算，包括同字符串拼串
​    3、for in, for of遍历时不会遍历symbol属性

#### 3.使用：

​    1、调用Symbol函数得到symbol值

```javascript
let symbol = Symbol();
let obj = {};
obj[symbol] = 'hello';
```

​    2、传参标识

​    2、传参标识

```javascript
let symbol = Symbol('one');
let symbol2 = Symbol('two');
console.log(symbol);// Symbol('one')
console.log(symbol2);// Symbol('two')
```

​    3、内置Symbol值

​    3、内置Symbol值

除了定义自己使用的Symbol值以外，ES6还提供了11个内置的Symbol值，指向语言内部使用的方法。对象的Symbol.iterator属性，指向该对象的默认遍历器方法

### 11. iterator

#### 1.概念： 

iterator是一种接口机制，为各种不同的数据结构提供统一的访问机制

#### 2.作用：

为各种数据结构，提供一个统一的、简便的访问接口；
使得数据结构的成员能够按某种次序排列
ES6创造了一种新的遍历命令for...of循环，Iterator接口主要供for...of消费。

#### 3.工作原理：

  - 创建一个指针对象，指向数据结构的起始位置。
  - 第一次调用next方法，指针自动指向数据结构的第一个成员
  - 接下来不断调用next方法，指针会一直往后移动，直到指向最后一个成员
  - 每调用next方法返回的是一个包含value和done的对象，{value: 当前成员的值,done: 布尔值}
    * value表示当前成员的值，done对应的布尔值表示当前的数据的结构是否遍历结束。
    * 当遍历结束的时候返回的value值是undefined，done值为false

原生具备iterator接口的数据(可用for of遍历)
1、Array
2、arguments
3、set容器
4、map容器
5、String

#### 4. 模拟为对象部署 iterator 接口

```javascript
let iteratorableObj={
    [Symbol.iteraor]:function(){
        return {
            let nextIndex=0;
            next:function(){
                return {
                    value:this[nextIndex++],
                    done:this.length==nextIndex
                }
            }
        }
    }
}
```

### 12. generator

#### 1.概念：

1. ES6提供的解决异步编程的方案之一
2. Generator函数是一个状态机，内部封装了不同状态的数据，
3. 用来生成遍历器对象
4. 可暂停函数(惰性求值), yield可暂停，next方法可启动。每次返回的是yield后的表达式结果

####  2.特点：

1. function 与函数名之间有一个星号
2. 内部用yield表达式来定义不同的状态

   例如：
​     function* generatorExample(){
​       let result = yield 'hello';  // 状态值为hello
​       yield 'generator'; // 状态值为generator
​     }

1. generator函数返回的是指针对象({next:function{….}})，而不会执行函数内部逻辑
2. 调用next方法函数内部逻辑开始执行，遇到yield表达式停止，返回{value: yield后的表达式结果/undefined, done: false/true}
3. 再次调用next方法会从上一次停止时的yield处开始，直到最后
4. yield语句返回结果通常为undefined， 当调用next方法时传参内容会作为启动时yield语句的返回值。

```javascript
function* generatorTest() {
  console.log('函数开始执行');
  let ret=yield 'hello';
  console.log(ret); // 这个值就是 aaa 就是这次启动的时候next传入的值
  console.log('函数暂停后再次启动');
  yield 'generator';
}
// 生成遍历器对象
let Gt = generatorTest();
// 执行函数，遇到yield后即暂停
console.log(Gt); // 遍历器对象
let result = Gt.next(); // 函数执行,遇到yield暂停
console.log(result); // {value: "hello", done: false}
result = Gt.next('aaa'); // 函数再次启动
console.log(result); // {value: 'generator', done: false}
result = Gt.next();
console.log(result); // {value: undefined, done: true}表示函数内部状态已经遍历完毕

// 真正的为对象部署接口采用Symbol.iterator属性;
let myIterable = {};
myIterable[Symbol.iterator] = function* () {
  yield 1;
  yield 2;
  yield 4;
};
for(let i of myIterable){
  console.log(i);
}
let obj = [...myIterable];
console.log(obj);

console.log('-------------------------------');
// 案例练习
/*
* 需求：
* 1、发送ajax请求获取新闻内容
* 2、新闻内容获取成功后再次发送请求，获取对应的新闻评论内容
* 3、新闻内容获取失败则不需要再次发送请求。
* 
* */ 
function* sendXml() {
  // url为next传参进来的数据
 let url = yield getNews('http://localhost:3000/news?newsId=2');
  yield getNews(url);
}
function getNews(url) {
  $.get(url, function (data) {
    console.log(data);
    let commentsUrl = data.commentsUrl;
    let url = 'http://localhost:3000' + commentsUrl;
    // 当获取新闻内容成功，发送请求获取对应的评论内容
    // 调用next传参会作为上次暂停是yield的返回值
    sx.next(url);
  })
}

let sx = sendXml();
// 发送请求获取新闻内容
sx.next();
```

### 13. async 

这个其实是generator的语法糖，这个是 es2017 的内容，但是已经被广泛的用开了。

#### 1.概念：

 真正意义上去解决异步回调的问题，同步流程表达异步操作

#### 2.语法：

```javascript
async function foo(){
    await 异步操作;
    await 异步操作；
}
```

#### 3.特点：

1. 不需要像Generator去调用next方法，遇到await等待，当前的异步操作成功就往下执行
2. 返回的总是Promise对象，可以用then方法进行下一步操作
3. async取代Generator函数的星号*，await取代Generator的yield 。await 的返回值就是 reslove 和 reject 的参数
4. 语意上更为明确，使用简单，经临床验证，暂时没有任何副作用

```javascript
async function timeout(ms) {
  return new Promise(resolve => {
    setTimeout(resolve, ms);
  })
}

async function asyncPrint(value, ms) {
  console.log('函数执行', new Date().toTimeString());
  await timeout(ms);
  console.log('延时时间', new Date().toTimeString());
  console.log(value);
}

console.log(asyncPrint('hello async', 2000));

// await 
async function awaitTest() {
  let result = await Promise.resolve('执行成功'); // 这个直接用来创建并设定 Promise 的状态
  console.log(result);
  let result2 = await Promise.reject('执行失败');  // 这个直接用来创建并设定 Promise 的状态
  console.log(result2);
  let result3 = await Promise.resolve('还想执行一次');// 执行不了，失败了就不会继续执行下去
  console.log(result3);
}
awaitTest();


// 案例演示
async function sendXml(url) {
    return new Promise((resolve, reject) => {
      $.ajax({
        url,
        type: 'GET',
        success: data =>  resolve(data),
        error: error => reject(error)
      })
    })
}

async function getNews(url) {
  let result = await sendXml(url); //如果这里报错了就会执行 reject 导致控制台抛异常，但是为了用户体验我们可以在错误的时候使用resolve但是要不让下面的函数继续执行。
  let result2 = await sendXml(result.commentsUrl);
  console.log(result, result2);
}
getNews('http://localhost:3000/news?id=2')
```

### 14. class

1. 通过class定义类/实现类的继承
2. 在类中通过constructor定义构造方法，因为原型中的 constructor 就是我们定义的工厂方法，在我们使用工厂方法的时候。
3. 通过new来创建类的实例
4. 通过extends来实现类的继承
5. 通过super调用父类的构造方法
6. 重写从父类中继承的一般方法
7. 在class中写方法只能通过省略 function 的简写方式或者箭头函数书写，否则会报错

### 15. 数组扩展

1. Array.from(v) 将伪数组转成真数组
2. Array.of(v1,v2,v3….) 将一系列的元素变成数组
3. Array.find((value,index,arr)=>{return true/false}) 找出第一个满足条件的元素
4. Array.findIndex((value,index,arr)=>{return true/false})  同上只是返回的不是元素而是下标

### 16.Obj拓展

1. Object.is(v1, v2)

   判断2个数据是否完全相等

2. Object.assign(target, source1, source2..)

   将源对象的属性复制到目标对象上

3. 直接操作 __proto__ 属性

     ```javascript
     let obj2 = {};
     obj2.__proto__ = obj1;
     ```

### 17. 深度克隆

#### 1、数据类型：

  * 数据分为基本的数据类型(String, Number, boolean, Null, Undefined)和对象数据类型
  - 基本数据类型：
    特点： 存储的是该对象的实际数据
  - 对象数据类型：
    特点： 存储的是该对象在栈中引用，真实的数据存放在堆内存里

#### 2、复制数据

  - 基本数据类型存放的就是实际的数据，可直接复制
    let number2 = 2;
    let number1 = number2;
  - 克隆数据：对象/数组
    1、区别： 浅拷贝/深度拷贝
       判断： 拷贝是否产生了新的数据还是拷贝的是数据的引用
       知识点：对象数据存放的是对象在栈内存的引用，直接复制的是对象的引用
       let obj = {username: 'kobe'}
       let obj1 = obj; // obj1 复制了obj在栈内存的引用
    2、常用的拷贝技术
      1). arr.concat(): 数组浅拷贝
      2). arr.slice(): 数组浅拷贝
      3). JSON.parse(JSON.stringify(arr/obj)): 数组或对象深拷贝, 但不能处理函数数据
      4). 浅拷贝包含函数数据的对象/数组
      5). 深拷贝包含函数数据的对象/数组

```javascript
// 复制的对象的方式
// 浅度复制
let obj = {username: 'kobe', age: 39, sex: {option1: '男', option2: '女'}};
let obj1 = obj;
console.log(obj1);
obj1.sex.option1 = '不男不女'; // 修改复制的对象会影响原对象
console.log(obj1, obj);

console.log('-----------');
// Object.assign();  浅复制
let obj2 = {};
Object.assign(obj2, obj);
console.log(obj2);
obj2.sex.option1 = '男'; // 修改复制的对象会影响原对象
console.log(obj2, obj);

// 深度克隆(复制)

function getObjClass(obj) {
  let result = Object.prototype.toString.call(obj).slice(8, -1);
  if(result === 'Null'){
    return 'Null';
  }else if(result === 'Undefined'){
    return 'Undefined';
  }else {
    return result;
  }
}
// for in 遍历数组的时候遍历的是下标
let testArr = [1,2,3,4];
for(let i in testArr){
  console.log(i); // 对应的下标索引
}

// 深度克隆
function deepClone(obj) {
  let result, objClass = getObjClass(obj);
  if(objClass === 'Object'){
    result = {};
  }else if(objClass === 'Array'){
    result = [];
  }else {
    return obj; // 如果是其他数据类型不复制，直接将数据返回
  }
  // 遍历目标对象
  for(let key in obj){
    let value = obj[key];
    if(getObjClass(value) === "Object" || 'Array'){
      result[key] = deepClone(value);
    }else {
      result[key] = obj[key];
    }
  }
  return result;
}


let obj3 = {username: 'kobe',age: 39, sex: {option1: '男', option2: '女'}};
let obj4 = deepClone(obj3);
console.log(obj4);
obj4.sex.option1 = '不男不女'; // 修改复制后的对象不会影响原对象
console.log(obj4, obj3);
```

## 3.es7

1. 指数运算符(幂): **
2. Array.prototype.includes(value) : 判断数组中是否包含指定value

