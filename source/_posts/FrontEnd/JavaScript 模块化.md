---
title: JavaScript 模块化
date: 2019-01-10 12:20:28
tags: JavaScript
categories:
 - FrontEnd
---

## 1. 模块化过程

### 1. 全局函数

```JavaScript
/**
 * 全局函数模式: 将不同的功能封装成不同的全局函数
 * 问题: Global被污染了, 很容易引起命名冲突
 */
//数据
let data = 'atguigu.com'
function foo() {
    console.log('foo()')
}
function bar() {
    console.log('bar()')
}
```

### 2. 对象空间

```JavaScript
/**
 * namespace模式: 简单对象封装
 * 作用: 减少了全局变量
 * 问题: 不安全(数据不是私有的, 外部可以直接修改)
 */
let myModule = {
  data: 'atguigu.com',
  foo() {
    console.log(`foo() ${this.data}`)
  },
  bar() {
    console.log(`bar() ${this.data}`)
  }
}
```

### 3. 立即执行函数

```JavaScript
/**
 * IIFE模式: 匿名函数自调用(闭包)
 * IIFE : immediately-invoked function expression(立即调用函数表达式)
 * 作用: 数据是私有的, 外部只能通过暴露的方法操作
 * 问题: 如果当前这个模块依赖另一个模块怎么办?
 */
(function (window) {
  //数据
  let data = 'atguigu.com'

  //操作数据的函数
  function foo() { //用于暴露有函数
    console.log(`foo() ${data}`)
  }

  function bar() {//用于暴露有函数
    console.log(`bar() ${data}`)
    otherFun() //内部调用
  }

  function otherFun() { //内部私有的函数
    console.log('otherFun()')
  }

  //暴露行为
  window.myModule = {foo, bar}
})(window)
```

### 4.依赖注入的自执行函数

```JavaScript
/**
 * IIFE模式增强 : 引入依赖
 * 这就是现代模块实现的基石
 */
(function (window, $) {
  //数据
  let data = 'atguigu.com'

  //操作数据的函数
  function foo() { //用于暴露有函数
    console.log(`foo() ${data}`)
    $('body').css('background', 'red')
  }

  function bar() {//用于暴露有函数
    console.log(`bar() ${data}`)
    otherFun() //内部调用
  }

  function otherFun() { //内部私有的函数
    console.log('otherFun()')
  }

  //暴露行为
  window.myModule = {foo, bar}
})(window, jQuery)
```

## 2.CommonJS

​	commonJs 是同步进行依赖引入的，并且他是相当于一个文件一个模块，对于模块的导出采用了 `module.exports={}` 或者使用 `exports={}` 来完成的，而模块的引入则是通过 `let mo=require("./module.js")` 但是注意 nodejs 采用的就是 CommonJs 所以他是直接可用这个语句的，但是浏览器不行，所以浏览器需要使用 bowserify 工具打包编译才能让浏览器识别。

### 1.在服务端应用

#### 1.下载安装node.js

#### 2.创建项目结构

  ```
  |-modules
    |-module1.js
    |-module2.js
    |-module3.js
  |-app.js
  |-package.json
    {
      "name": "commonJS-node",
      "version": "1.0.0"
    }
  ```
#### 3.下载第三方模块

  * npm install uniq --save
#### 4.模块化编码

  * module1.js
    ```
    module.exports = {
      foo() {
        console.log('moudle1 foo()')
      }
    }
    ```
  * module2.js
    ```
    module.exports = function () {
      console.log('module2()')
    }
    ```
  * module3.js
    ```
    exports.foo = function () {
      console.log('module3 foo()')
    }
    
    exports.bar = function () {
      console.log('module3 bar()')
    }
    ```
  * app.js 
    ```
    /**
      1. 定义暴露模块:
        module.exports = value;
        exports.xxx = value;
      2. 引入模块:
        var module = require(模块名或模块路径);
     */
    "use strict";
    //引用模块
    let module1 = require('./modules/module1')
    let module2 = require('./modules/module2')
    let module3 = require('./modules/module3')
    
    let uniq = require('uniq')
    let fs = require('fs')
    
    //使用模块
    module1.foo()
    module2()
    module3.foo()
    module3.bar()
    
    console.log(uniq([1, 3, 1, 4, 3]))
    
    fs.readFile('app.js', function (error, data) {
      console.log(data.toString())
    })
    ```
#### 5.通过node运行app.js

命令: node app.js

### 2.在浏览器的运行

#### 1.创建项目结构

  ```
  |-js
    |-dist //打包生成文件的目录
    |-src //源码所在的目录
      |-module1.js
      |-module2.js
      |-module3.js
      |-app.js //应用主源文件
  |-index.html
  |-package.json
    {
      "name": "browserify-test",
      "version": "1.0.0"
    }
  ```
#### 2.下载browserify

  * 全局: npm install browserify -g
  * 局部: npm install browserify --save-dev
3. 定义模块代码
  * module1.js
    ```
    module.exports = {
      foo() {
        console.log('moudle1 foo()')
      }
    }
    ```
  * module2.js
    ```
    module.exports = function () {
      console.log('module2()')
    }
    ```
  * module3.js
    ```
    exports.foo = function () {
      console.log('module3 foo()')
    }
    
    exports.bar = function () {
      console.log('module3 bar()')
    }
    ```
  * app.js (应用的主js)
    ```
    //引用模块
    let module1 = require('./module1')
    let module2 = require('./module2')
    let module3 = require('./module3')
    
    let uniq = require('uniq')
    
    //使用模块
    module1.foo()
    module2()
    module3.foo()
    module3.bar()
    
    console.log(uniq([1, 3, 1, 4, 3]))
    ```

#### 3.打包处理js:

* browserify js/src/app.js -o js/dist/bundle.js

#### 4.页面使用引入:

```
<script type="text/javascript" src="js/dist/bundle.js"></script> 
```



## 3. AMD

异步模块定义。**显示声明依赖注入** 

1. 定义模块采用 define
2. 导出模块在回调函数中 return
3. 引入模块使用 require

### 1.下载require.js, 并引入

  * 官网: http://www.requirejs.cn/
  * github : https://github.com/requirejs/requirejs
  * 将require.js导入项目: js/libs/require.js 
### 2.创建项目结构

  ```
  |-js
    |-libs
      |-require.js
    |-modules
      |-alerter.js
      |-dataService.js
    |-main.js
  |-index.html
  ```
### 3.定义require.js的模块代码

  * dataService.js
    ```
    define(function () {
      let msg = 'test'
    
      function getMsg() {
        return msg.toUpperCase()
      }
    
      return {getMsg}
    })
    ```
  * alerter.js
    ```
    define(['dataService', 'jquery'], function (dataService, $) {
      let name = 'Tom2'
    
      function showMsg() {
        $('body').css('background', 'gray')
        alert(dataService.getMsg() + ', ' + name)
      }
    
      return {showMsg}
    })
    ```
### 4.应用主(入口)js: main.js

  ```
  (function () {
    //配置
    require.config({
      //基本路径
      baseUrl: "js/",
      //模块标识名与模块路径映射
      paths: {
        "alerter": "modules/alerter",
        "dataService": "modules/dataService",
      }
    })
    //引入使用模块
    require( ['alerter'], function(alerter) {
      alerter.showMsg()
    })
  })()
  ```

### 6.页面使用模块:

```
  <script data-main="js/main" src="js/libs/require.js"></script>
```

### 7.使用第三方基于require.js

#### 1.将jquery的库文件导入到项目: 

js/libs/jquery-1.10.1.js

#### 2.在main.js中配置jquery路径

```
paths: {
          'jquery': 'libs/jquery-1.10.1'
      }
```

#### 3.在alerter.js中使用jquery

```
define(['dataService', 'jquery'], function (dataService, $) {
    var name = 'xfzhang'
    function showMsg() {
        $('body').css({background : 'red'})
        alert(name + ' '+dataService.getMsg())
    }
    return {showMsg}
})
```

### 8.第三方不基于require.js的框架

#### 1.将angular.js和angular-messages.js导入项目

* js/libs/angular.js
* js/libs/angular-messages.js

#### 2.在main.js中配置

```
(function () {
  require.config({
    //基本路径
    baseUrl: "js/",
    //模块标识名与模块路径映射
    paths: {
      //第三方库
      'jquery' : 'libs/jquery-1.10.1',
      'angular' : 'libs/angular',
      'angular-messages' : 'libs/angular-messages',
      //自定义模块
      "alerter": "modules/alerter",
      "dataService": "modules/dataService"
    },
    /*
     配置不兼容AMD的模块
     exports : 指定导出的模块名
     deps  : 指定所有依赖的模块的数组
     */
    shim: {
      'angular' : {
        exports : 'angular'
      },
      'angular-messages' : {
        exports : 'angular-messages',
        deps : ['angular']
      }
    }
  })
  //引入使用模块
  require( ['alerter', 'angular', 'angular-messages'], function(alerter, angular) {
    alerter.showMsg()
    angular.module('myApp', ['ngMessages'])
    angular.bootstrap(document,["myApp"])
  })
})()
```

#### 3.页面

```
<form name="myForm">
  用户名: <input type="text" name="username" ng-model="username" ng-required="true">
  <div style="color: red;" ng-show="myForm.username.$dirty&&myForm.username.$invalid">用户名是必须的</div>
</form>
```

## 4. ES6

​	es6这种方式就是定义模块直接向我们定义一个对象或者函数一样，而需要导出就是采用 export 导出函数或者对象或变量。在一个模块中可以多次 export 但是在引入的时候必须采用对象解构的方式导入我们 export 的内容。

```javascript
//------------------module.js
export foo=()=>{}
export boo=()=>{}
//------------------main.js
import {foo,boo} from './module.js'
```

​	如果我们使用默认导出则是可以直接用一个变量来接受我们导出的内容。

```javascript
import module from './module.js'
module.foo();
```

### 1.定义package.json文件

  ```
  {
    "name" : "es6-babel-browserify",
    "version" : "1.0.0"
  }
  ```
### 2.安装babel-cli, babel-preset-es2015和browserify

```bash
npm install babel-cli browserify -g  // babel 的命令行工具
npm install babel-preset-es2015 --save-dev  // babel的工具包，babel的工具包作用有很多比如这里 es6 转 es5 还有把jsx转html等
```

### 3.定义.babelrc文件

```javascript
{
 "presets": ["es2015"] //在babel工作之前会读取这个文件，然后表明babel调用什么模块做什么事情，这个就是 es6转es5 比如说还有 react 就是转jsx语法
 }
```

### 4.编码

  * js/src/module1.js
    ```
    export function foo() {
      console.log('module1 foo()');
    }
    export let bar = function () {
      console.log('module1 bar()');
    }
    export const DATA_ARR = [1, 3, 5, 1]
    ```
  * js/src/module2.js
    ```
    let data = 'module2 data'
    
    function fun1() {
      console.log('module2 fun1() ' + data);
    }
    
    function fun2() {
      console.log('module2 fun2() ' + data);
    }
    
    export {fun1, fun2}
    ```
  * js/src/module3.js
    ```
    export default {
      name: 'Tom',
      setName: function (name) {
        this.name = name
      }
    }
    ```
  * js/src/app.js
    ```
    import {foo, bar} from './module1'
    import {DATA_ARR} from './module1'
    import {fun1, fun2} from './module2'
    import person from './module3'
    
    import $ from 'jquery'
    
    $('body').css('background', 'red')
    
    foo()
    bar()
    console.log(DATA_ARR);
    fun1()
    fun2()
    
    person.setName('JACK')
    console.log(person.name);
    ```
### 5.编译

  1. 使用Babel将ES6编译为ES5代码(但包含CommonJS语法) : babel js/src -d js/lib
  2. 使用Browserify编译js : browserify js/lib/app.js -o js/lib/bundle.js
### 6.页面中引入测试

  ```
  <script type="text/javascript" src="js/lib/bundle.js"></script>
  ```
### 7.引入第三方模块(jQuery)

#### 1.下载jQuery模块: 

`npm install jquery@1 --save`

#### 2.在app.js中引入并使用

    ```javascript
import $ from 'jquery'
$('body').css('background', 'red')
    ```