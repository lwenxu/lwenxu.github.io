---
title: React 实战
date: 2019-01-17 11:20:28
tags: React
categories:
 - FrontEnd
---

## 1.目录

![](http://images.heniankj.com/20190117112438.png)



## 2.依赖安装

```bash
cnpm install antd-mobile -S 
//组件按需加载  第二个是用来做react脚手架的二次配置的，因为我们看不到Webpack的配置文件了 https://www.cnblogs.com/xiaohuochai/p/8491055.html
cnpm install --save-dev babel-plugin-import react-app-rewired@2.0.2-next.0 //注意指定版本号否则不兼容
cnpm install --save-dev less@2.7.3 less-loader

```



## 3.配置项目

### 1.修改index添加触摸事件

```html
<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, minimum-scale=1, user-scalable=no" />
<script src="https://as.alipayobjects.com/g/component/fastclick/1.0.6/fastclick.js"></script>
    <script>
      if ('addEventListener' in document) {
        document.addEventListener('DOMContentLoaded', function() {
          FastClick.attach(document.body);
        }, false);
      }
      if(!window.Promise) {
        document.writeln('<script src="https://as.alipayobjects.com/g/component/es6-promise/3.2.2/es6-promise.min.js"'+'>'+'<'+'/'+'script>');
      }
</script>
```

### 2.添加 antd 配置文件 config-overrides.js 

注意这个文件不是在 src 下面而是在整个项目的与 package,json 同级的目录。定义加载配置的 js 模块 !

```javascript
const {injectBabelPlugin} = require('react-app-rewired');
module.exports = function override(config, env) {
config = injectBabelPlugin(['import', {libraryName: 'antd-mobile', style: 'css'}],
config);
return config;
}
```

修改配置: package.json 

```json
"scripts": {
    "start": "react-app-rewired start",
    "build": "react-app-rewired build",
    "test": "react-app-rewired test --env=jsdom",
    "eject": "react-scripts eject"
} 
```

这是关于自定义配置的内容  [Antd配置](https://ant.design/docs/react/use-with-create-react-app-cn)

