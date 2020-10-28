---
title: CSS3 从入门到精通
date: 2019-01-08 21:01:28
tags: CSS
categories:
 - FrontEnd
---
## 1. 选择器

### 1. 一些注意的点

#### 1. love & hate (lvht)

​	这是什么意思呢？其实是我们在写 a 表的伪类的时候会有一些优先级的问题，如果说我们没有按照这个优先级写css 的话很可能会导致我们样式不生效或者说被覆盖的问题。具体来说就是 `link` , `visited`，`hover`，`target` 。

#### 2. nth-child(n)

​	注意这个的 n 是从 1 开始的而不是 0 开始的，另外如果我们写了 `#wrap p:nth-child(1)` 的含义是指 id 为 wrap 下面的第一个子元素，并且这个字元素必须是 `p` 才会应用样式。

​	同样的还有一个类似的 就是 `nth-of-type(n)`  这个其实和上面的很像只是他的要求稍微低一些。`#wrap p:nth-of-type(1)` 比如这个就是表示 `wrap` 下面的第一个 `p` 标签。

​	上面两个的重要的区别(坑！)。就是 `nth-of-type` 是以元素标签为中心的，下面看个例子来理解这句话。

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title></title>
		<style type="text/css">
			*{
				margin: 0;
				padding: 0;
			}
             #wrap .inner:nth-of-type(1){
				border: 1px solid;
			}
			/*#wrap div:nth-of-type(1){
				border: 1px solid;
			}
			#wrap p:nth-of-type(1){
				border: 1px solid;
			}
			#wrap span:nth-of-type(1){
				border: 1px solid;
			}
			#wrap h1:nth-of-type(1){
				border: 1px solid;
			}
			#wrap h2:nth-of-type(1){
				border: 1px solid;
			}*/
			
			/*nth-of-type以元素为中心*/
		</style>
	</head>
	<body>
		<div id="wrap">
			<div class="inner">div</div>
			<p class="inner">p</p>
			<span class="inner">span</span>
			<h1 class="inner">h1</h1>
			<h2 class="inner">h2</h2>
		</div>
	</body>
</html>
```

​	注意上面的代码看起来就是第一个被选中，但是由于它是以元素为中心的那么，他会被展开成下面的语法结构。所以说所有的都会被选中。而 `nth-child` 则是正常的选中第一个。

## 2. 字体图标

​	字体图标减少了页面的请求，而且不失真非常可靠，推荐使用。如果我们需要 使用一些特殊的紫图需要自定义字体因为为了兼容不同的用户，但是这样会增加网络负担。

```csss
@font-face{
    font-family:"test";
    src:url('');/*字体文件的位置*/
}
```

​	使用字体图标，主要就是ttf，wof还有一个样式表，直接引入样式表就可以使用了。

## 3. 新的UI方案

### 1. 文本新增样式

#### 1. opactiy

​	他不是继承属性。但是子元素也会被透明掉。

#### 2. rgba

​	这个前三个就是 rgb 最后一个是透明度。正式有了这个现在的文字透明背景不透明或者背景透明文字不透明就好做了，直接给某一方设置rgba而另外一方设置正常的颜色就好。而不用opacity导致前后后透明。

```css
a{
    color:rgba(1,1,1,.5);
}
```

​	创建一个模糊背景的图片。首先需要有一个图片背景(让图片做背景一般都是图片有多大背景就能看到多少图片，为了让图片完全的铺满背景需要把 background-size:100% 100%)，然后需要对他做一个模糊使用了一个函数 filter:blur(10px) 就是把它模糊掉。然后需要一个外面的黑色遮罩，让他出现半透明的状态就可以看到后面的内容了。

​	**小技巧：如果希望我们内层的元素铺满外层的内容区可以使用绝对定位然后让他的四个方向都为 0 **

```html
<!DOCTYPE html>
<html>
   <head>
      <meta charset="UTF-8">
      <meta name="viewport" content="width=device-width,initial-scale=1.0,user-scalable=no" />
      <title></title>
      <style type="text/css">
         *{
            margin: 0;
            padding: 0;
         }
         #wrap{
            height: 100px;
            background: rgba(0,0,0,.5);
            position: relative;
         }
         #wrap #bg{
            position: absolute;
            left: 0;
            right: 0;
            top: 0;
            bottom: 0;
            background: url(img/avatar.jpg) no-repeat;
            background-size:100% 100% ;
            z-index: -1;
            filter: blur(10px);
         }
         img{
            margin: 24px 0 0 24px;
         }
      </style>
   </head>
   <body>
      <div id="wrap">
         <img src="img/avatar.jpg" width="64px" height="64px"/>
         <div id="bg"></div>
      </div>
   </body>
</html>
```

#### 3. 文字阴影

text-shadow用来为文字添加阴影，而且可以添加多层，阴影值之间用逗号隔开。（多个阴影时，第一个阴影在最上边）

默认值：none        不可继承 

值
​    <color>
​       可选。可以在偏移量之前或之后指定。如果没有指定颜色，则使用UA（用户代理）选择的颜色。
​    <offset-x> <offset-y>
​       必选。这些长度值指定阴影相对文字的偏移量。
​       <offset-x> 指定水平偏移量，若是负值则阴影位于文字左边。        
​       <offset-y> 指定垂直偏移量，若是负值则阴影位于文字上面。
​       如果两者均为0，则阴影位于文字正后方(如果设置了<blur-  radius> 则会产生模糊效果)。
​    <blur-radius>
​       可选。这是 <length> 值。如果没有指定，则默认为0。
​       值越大，模糊半径越大，阴影也就越大越淡	

```css
h1{
    text-shadow:gray 10px 10px 10px,pink 10px 10px 10px;
}
```

####  4. 文字排版

​	溢出显示省略号，最常用的！但是使用这个东西盒子需要是一个 block

```html
<!DOCTYPE HTML>
<html>
<head>
<meta charset="utf-8">
<title>无标题文档</title>
<style>
	div{
		width: 200px;
		height: 200px;
		border: 1px solid;
		margin: 0 auto;
        /*这三样式是一直在一起的不能去掉，并且盒子不能靠内容撑开*/
		white-space: nowrap;
		overflow: hidden;
		text-overflow: ellipsis;
	}
</style>
</head>
<body>
	<div>aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa</div>
</body>
</html>
```

### 2. 盒模型阴影

​	首先是需要进行一个盒子的居中：

```css
div{
    position: absolute;
    left: 0;
    right: 0;
    bottom: 0;
    top: 0;
    margin: auto;
    width: 100px;
    height: 100px;
    background: pink;
    text-align: center;
    line-height: 100px;
}
```

​	注意需要margin为auto而不能为0，盒子唏嘘也要有长宽。

box-shadow 
​    以逗号分割列表来描述一个或多个阴影效果，可以用到几乎任何元素上。 如果元素同时设置了 border-radius ，阴影也会有圆角效果。多个阴影时和多个 text shadows 规则相同(第一个阴影在最上面)。

默认值:  none    不可继承

值：
​    inset
​       默认阴影在边框外。
​       使用inset后，阴影在边框内。
​    <offset-x> <offset-y>
​       这是头两个 <length> 值，用来设置阴影偏移量。
​        <offset-x> 设置水平偏移量，如果是负值则阴影位于元素左边。
​        <offset-y> 设置垂直偏移量，如果是负值则阴影位于元素上面。
​       如果两者都是0，那么阴影位于元素后面。
​       这时如果设置了<blur-radius> 或<spread-radius> 则有模糊效果。
​    <blur-radius>
​       这是第三个 <length> 值。值越大，模糊面积越大，阴影就越大越淡。 
​       不能为负值。默认为0，此时阴影边缘锐利。
​    <spread-radius>
​       这是第四个 <length> 值。取正值时，阴影扩大；取负值时，阴影.收缩。默认为0，此时阴影与元素同样大。
​    <color>
​       阴影颜色，如果没有指定，则由浏览器决定

 ```css
div{
    box-shadow: -10px -10px 10px 0px black , 20px 20px 10px -10px deeppink;
}
 ```

### 3. box-sizing

​	由于我们一般的额 height 和 width 的值指的是内容区的大小，所以如果我们用了 padding的话会导致整个盒子的变化，那么为了不让这种计算出现，可以使用 `box-sizing:border-box` 就可以了。此时采用padding的话就是从内容区扣掉的。 

### 4. border-radius

​	圆角

### 5. 背景

- background-color 默认为transparent 也就是透明的

- background-image 指定图片做背景，有的图片在z轴上面绘制，先写得图片会在最后绘制也就是层叠图片的时候是按照写得顺序从上往下绘制的。

- no-repeat

- background-position 如果图片的大小大于框的大小，这个属性就可以决定图片是从哪开始显示的。

- background-origin 表示图片是从哪开始渲染的，它有三个值：border-box ,padding-box ,content-box 分别就是从边框，内边距开始的地方，内容区开始的地方。

- background-clip:从那个位置开始裁剪，属性值也是和origin一样的。

- background-size：百分比：  指定背景图片相对背景区（background positioning area）的百分比。背景区由background-origin设置，默认为盒模型的内容区与内边距 auto：  以背景图片的比例缩放背景图片。

  注意：
    单值时，这个值指定图片的宽度，图片的高度隐式的为auto
    两个值: 第一个值指定图片的宽度，第二个值指定图片的高度     

### 6. 线性渐变

​	**注意：渐变是针对于图片的而不是颜色。**
 	  为了创建一个线性渐变，你需要设置一个起始点和一个方向（指定为一个角度）。你还要定义终止色。终止色就是你想让浏览器去平滑的过渡过去，并且你必须指定至少两种，当然也会可以指定更多的颜色去创建更复杂的渐变效果。

​	如果在里面设置了透明度他也会跟着渐变的，就是 rgba

-默认从上到下发生渐变
​              linear-gradient(red,blue);

-改变渐变方向：（top bottom left right）
​              linear-gradient(to 结束的方向,red,blue);

-使用角度
​              linear-gradient(角度,red,blue);


-颜色节点的分布（第一个不写为0%，最后一个不写为100%）
​            linear-gradient(red 长度或者百分比,blue 长度或者百分比);
-重复渐变
​           repeating-linear-gradient(60deg,red 0,blue 30%);

 ```css
p{
    background:repeating-linear-gradient(135deg,black 0px,black 10px,white 10px,white 20px);
}
 ```

注意这个写法他的意思就是1-10是黑到黑的渐变就是一个完整的黑色，而10-20 是白到白的渐变。为什么要这么做，主要就是我们使用了 `repeating-linear-gradient`  这个 repeat 就是重复的渐变，如果我们不是设置的渐变而是直接的纯色的话他就不会重复了。所以这种方式巧妙的让这些黑白重复了。发廊灯的例子：

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title></title>
		<style type="text/css">
			*{
				margin: 0;
				padding: 0;
			}
			
			html,body{
				height: 100%;
				overflow: hidden;
			}
			
			#wrap{
				width: 40px;
				height: 300px;
				border: 1px solid;
				margin: 100px auto;
				overflow: hidden;
			}
			#wrap > .inner{
				height: 600px;
				background:repeating-linear-gradient(135deg,black 0px,black 10px,white 10px,white 20px);
			}
			
		</style>
	</head>
	<body>
		<div id="wrap">
			<div class="inner"></div>
		</div>
	</body>
	<script type="text/javascript">
		var inner = document.querySelector("#wrap > .inner");
		var flag =0;
		
		setInterval(function(){
			flag++;
			if(flag==300){
				flag=0;
			}
			inner.style.marginTop = -flag+"px";
		},1000/60)
		
	</script>
</html>
```



光板动画，iPhone的开机动画：

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title></title>
		<style type="text/css">
			*{
				margin: 0;
				padding: 0;
			}
			
			html,body{
				height: 100%;
				overflow: hidden;
				background: black;
				text-align: center;
			}
			
			h1{
				/*transition: 3s;*/
				margin-top: 50px;
				display: inline-block;
				color: rgba(255, 255, 255,.3);
				font:bold 80px "微软雅黑";
				background: linear-gradient(120deg,rgba(255,255,255,0) 100px ,rgba(255,255,255,1) 180px ,rgba(255,255,255,0) 260px);
				background-repeat:no-repeat ;
				-webkit-background-clip: text ;
			}
			
			/*h1:hover{
				background-position: 500px 0;
			}*/
		</style>
	</head>
	<body>
		<h1>atguigu尚硅谷</h1>
	</body>
	<script type="text/javascript">
		var h1 = document.querySelector("h1");
		var flag =-160;
		
		setInterval(function(){
			flag+=10;
			if(flag==600){
				flag=-160;
			}
			h1.style.backgroundPosition = flag+"px";
		},30)
		
	</script>
</html>
```

### 7. 径向渐变

```html
<!DOCTYPE html>
<html>
   <head>
      <meta charset="UTF-8">
      <title></title>
      <style type="text/css">
         *{
            margin: 0;
            padding: 0;
         }
         #test{
            width: 400px;
            height: 300px;
            border: 1px solid;
            margin: 0 auto;
            background-image:radial-gradient( closest-corner circle at 20px 20px,yellow, green 50%,pink) ;
         }
      </style>
   </head>
   <body>
      <div id="test">
         
      </div>
   </body>
</html>
```

## 4.过渡

### 1. transition-property

​	这个属性是用来指定哪些属性需要做渐变的，而且并不是所有的属性都是可以做渐变的，可以在mdn上查询到。多个属性可以采用逗号分隔。默认值就是 all

### 2. transition-duration

​	这个用来规定动画间隔的时间，注意如果这个列表不满足上面的property的个数，他默认采用复制来匹配，比如上面有三个属性，而duration只有两个时间 3s,5s 那么最后的结果应该是 3s.5s.3s,5s 这样上面三个属性是 3s,5s ,3s 注意点就是他们需要带上单位。

### 3. transition-timing-function

​	这个就是用来指定整个动画的运行的过程或者速率：

属性值：
​         1、ease：（加速然后减速）默认值，ease函数等同于贝塞尔曲线(0.25, 0.1, 0.25, 1.0).
​         2、linear：（匀速），linear 函数等同于贝塞尔曲线(0.0, 0.0, 1.0, 1.0).
​         3、ease-in：(加速)，ease-in 函数等同于贝塞尔曲线(0.42, 0, 1.0, 1.0).
​         4、ease-out：（减速），ease-out 函数等同于贝塞尔曲线(0, 0, 0.58, 1.0).
​         5、ease-in-out：（加速然后减速），ease-in-out 函数等同于贝塞尔曲线(0.42, 0, 0.58, 1.0)
​         6、cubic-bezier： 贝塞尔曲线，这是一个函数里面传入对应的参数就可以呈现出不同的曲线	

​		http://cubic-bezier.com 在这个地址里面就能找到对应的被萨尔曲线对应的函数值和参数了。

比如说下面的这个曲线的意思就是横轴代表的是时间而纵轴代表的就是距离或者位置了，或者说速度。在这给对方代表的就是速度。

![](http://images.heniankj.com/20190109144545.png)

​         7、step-start：等同于steps(1,start)
​              step-end：等同于steps(1,end)	

​			start表示整个时间的开始，而end表示整个时间的结束。

​	      steps(<integer>,[,[start|end]]?)
​                      第一个参数：必须为正整数，指定函数的步数
​                      第二个参数：指定每一步的值发生变化的时间点（默认值end）在每步的时间开始还是结束的时候执行。

​		多个列表会使用默认值，如果不够的话。

### 4.  transition-delay

​	表示在多久之后才开始动画。这个如果缺省了参数不够列表的长度的话也是会重复列表的。

### 5. transitionend

​	表示动画执行结束，这是一个js事件可以使用js事件来监听，他是基于属性的一个事件，也就是等一个属性变化完成以后他就会调用一次而不是针对于元素的。

```javascript
node.addEventListener("transitionend",function(){}); //这种添加事件的方式是 dom2 事件
node.ontransitionend(function(){}) //这种是 dom0 事件
```

### 6. 过渡的一些坑

#### 1. 浏览器解析

```html
<!DOCTYPE html>
<html>
   <head>
      <meta charset="UTF-8">
      <title></title>
      <style type="text/css">
         *{
            margin: 0;
            padding: 0;
         }
      
         html{
            height: 100%;
         }
         body{
            width: 60%;
            height: 60%;
            border: 1px solid;
            margin: 100px auto 0;
         }
      
         #test{
            width: 100px;
            height: 100px;
            background: pink;
            text-align: center;
            position: absolute;
            left: 0;
            right: 0;
            bottom: 0;
            top: 0;
            margin: auto;
            
            transition-property: width;
            transition-duration: 2s;
            transition-timing-function: linear;
         }
         
         body:hover #test{
            transition-property: height;
            width: 200px;
            height: 200px;
         }
         
      </style>
   </head>
   <body>
      <div id="test">
      </div>
   </body>
</html>
```

可以看到上面是鼠标画上去的时候回选中 test 这个元素，然后将它动画设置为 height 变化，由于浏览器解析 css 太快了，在划上去的瞬间内存被修改了也就是变换的元素被修改了所以一开始变化的应该是height。而移除的时候同理变化的是 width

#### 2. transition在元素首次渲染还没有结束的情况下是不会触发的

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title></title>
		<style type="text/css">
			*{
				margin: 0;
				padding: 0;
			}
		
			html{
				height: 100%;
			}
			body{
				width: 60%;
				height: 60%;
				border: 1px solid;
				margin: 100px auto 0;
			}
		
			#test{
				width: 100px;
				height: 100px;
				background: pink;
				text-align: center;
				position: absolute;
				left: 0;
				right: 0;
				bottom: 0;
				top: 0;
				margin: auto;
				
				transition-property: width;
				transition-duration: 2s;
				transition-timing-function: linear;
			}
		</style>
	</head>
	<body>
		<div id="test">
		</div>
	</body>
	<script type="text/javascript">
		//transition在元素首次渲染还没有结束的情况下是不会被触发的	
//		window.onload=function(){
			var test = document.querySelector("#test");
			test.style.width="300px";
//		}
	</script>
</html>
```

​	如果没有打开注释部分的内容我们的动画是不会执行的，而是直接跳到300的样子，如果我们打开了注释意外这我们所有的元素已经渲染完了，这个时候再去做上面的额动画是会被执行的。

​	那么这么想的话我们如果加上一个 settimeout 也是没问题的。

#### 3.坑

**1. 在绝大部分变换样式切换时,如果变换函数的位置 个数不相同也不会触发过渡**

**2. transition必须在transform之后声明才有效！！！！**

## 5. 2D变幻

**注意所有的变幻都是和transition配合使用的。**

###  1. transform

#### 1.rotate(angle)  旋转

   正值:顺时针旋转  rotate(360deg)
   负值:逆时针旋转  rotate(-360deg)
   只能设单值。正数表示顺时针旋转，负数表示逆时针旋转

#### 2. translate 移动

X方向平移:transform:  translateX(tx)
Y方向平移:transform:  translateY(ty) 
二维平移：transform:  translate(tx[, ty])； 如果ty没有指定，它的值默认为0。

可设单值，也可设双值。
​       正数表示XY轴正向位移，负数为反向位移。设单值表示只X轴位移，Y轴坐标不变，
​       例如transform: translate(100px);等价于transform: translate(100px,0);

#### 3. skew 倾斜

transform:skewX(45deg);
​    X方向倾斜:transform:  skewX(angle)
​               skewX(45deg):参数值以deg为单位 代表与y轴之间的角度
​    Y方向倾斜:transform:  skewY(angle)
​               skewY(45deg):参数值以deg为单位 代表与x轴之间的角度
​     二维倾斜:transform:  skew(ax[, ay]);  如果ay未提供，在Y轴上没有倾斜
​               skew(45deg,15deg):参数值以deg为单位 第一个参数代表与y轴之间的角度
​                                                                        第二个参数代表与x轴之间的角度
单值时表示只X轴扭曲，Y轴不变，如transform: skew(30deg);等价于 transform: skew(30deg, 0);
考虑到可读性，不推荐用单值，应该用transform: skewX(30deg);。skewY表示只Y轴扭曲，X轴不变 
 正值:拉正斜杠方向的两个角
 负值:拉反斜杠方向的两个角

#### 4. scale 缩放

transform:scale(2);
  X方向缩放:transform:  scaleX(sx); 
  Y方向缩放:transform:  scaleY(sy);
  二维缩放 :transform:  scale(sx[, sy]);  (如果sy 未指定，默认认为和sx的值相同)  

  要缩小请设0.01～0.99之间的值，要放大请设超过1的值。
  例如缩小一倍可以transform: scale(.5);放大一倍可以transform: scale(2);

 如果只想X轴缩放，可以用scaleX(.5)相当于scale(.5, 1)。
 同理只想Y轴缩放，可以用scaleY(.5)相当于scale(1, .5)

 正值:缩放的程度
 负值:不推荐使用（有旋转效果）
 单值时表示只X轴,Y轴上缩放粒度一样，如transform: scale(2);等价于transform: scale(2,2);

### 2. transform-origin

参考点 transform-origin 这个值可以是：

top left 左上

10px 10px 参照左上角的向右向下10px

10% 10% 同px参照的是自身的宽高。



## 6. 3D 旋转

### 1. transform 

里面的一些函数和2D的一模一样只是多了 X,Y,Z的后缀例如 `rotateY(360deg)`

### 2. perspective

​	景深，让3D物体看起来有近大远小的感觉。但是这个属性只能用在舞台上，也就是如果要过三D动画，必须有两层外面一层叫做舞台，里面才是真实的物体。

### 3. transform-style

建立有层级的 3d 舞台，他作用于舞台。

### 

## 7.动画

### 1. 常用属性

#### 1. animation-name

​	指定所使用的关键帧的名字，比如 `animation-name: move;` 关键帧的样子：

```css
@keyframes move{
   from{
      transform: translateY(-100px);
   }
   to{
      transform: translateY(100px);
   }
}
```

#### 2. animation-duration

​	设置动画持续的时间。需要带上单位  `animation-duration:3s ;`

#### 3.animation-direction

​	动画翻转，他变化的是关键帧的 from 到 to 的两个顺序，并且变化了`animation-timing-function`  动画的速度变化曲线。

​	他有四个值：

1. normal 就是正常的每一次执行动画都是从 from 到 to 这样的执行。
2. reverse 这样关键帧执行的顺序从 to 到 from ，并且会让 animation-timing-function`  动画的速度变化曲线翻转。
3. alternate 如果有多次启动关键帧，表示从 from到to再从to到from 这样一直往返。
4. alternate-reverse 同上，只是方向是反的。

#### 4.animation-delay

​	设置动画还有多久才开始，需要设置单位 `animation-delay:1s;`

#### 5. animation-iteration-count

​	设置关键帧执行的次数，也就是动画执行的次数。 `animation-iteration-count: 3;`

#### 6. animation-fill-mode

​	设置动画之外的状态，就是规定from和to在动画开始之前的位置和状态。

1. backwards：from之前的状态与form的状态保持一致
2. forwards：to之后的状态与to的状态保持一致
3. both：backwards+forwards

#### 7.animation-play-state

​	表示当前的动画的状态，可以通过他来暂停动画和开始动画

1. running 启动动画
2. paused 暂停动画

#### 8. animation-timing-function

​	规定了动画的速率，具体和 transition 是一样的。

### 2. 关键帧

语法：

```css
   @keyframes animiationName{
       keyframes-selector{
           css-style;
       }
    }
```



1. animiationName:必写项，定义动画的名称
2.  keyframes-selector：必写项，动画所在的时间点 from等价于 0% 而 to等价于100%，我么也可以定义其他的一些百分比。
3.  css-style：css声明

## 8. flex

​	flex分为两个版本，分别为老版本的和新版本的，新版本的功能更强大但是他不支持很多的老的移动端。但是我们可以使用css后置处理器来处理这种兼容性问题。

### 1. 基本概念

容器：就是flex布局的容器，也是一个包裹层然后里面样式设置为 flex

项目：item 就是在容器中排列的内容就是项目，**项目永远在主轴的正方向排列**

主轴：默认是x轴，正方向右

侧轴：默认y轴，正方向向下

### 2. 容器管理

#### 1. 设置容器

老版本：`display：-webkit-box`

新版本：`display：flex`

#### 2. 设置主轴

老版本：`-webkit-box-orient：vertical/horizontal`

新版本：`flex-direction:row/column`

#### 3. 设置排列方向，控制主轴方向

老版本：`-webkit-box-direction：normal/reverse`

新版本：`flex-direction:row/column/row-reverse/column-reverse`  用一个属性设置主轴以及主轴的方向

#### 4. 富裕空间管理

主侧轴的空白的空间管理。他管理的是富裕空间的位置并不影响项目的空间大小  

##### 1.主轴

老版本：`-webkit-box-pack：start(右，下)/end(左，上)/center(两边)/jusity(项目之间)`  与主轴无关，只和x，y轴有关，前面指的是x轴的情况，后面是y轴的情况	

新版本：`justify-content:flex-start(主轴正方向)/flex-end(主轴负方向)/center(主轴两边)/space-between(项目之间)/space-around(项目两边)`  

##### 2. 侧轴

老版本：`-webkit-box-align：start(右，下)/end(左，上)/center(两边)/jusity(项目之间)`  与主轴无关，只和x，y轴有关，前面指的是x轴的情况，后面是y轴的情况	

新版本：`align-items:flex-start(侧轴正方向)/flex-end(侧轴负方向)/content(侧轴两边)/base-line(按照基线)/strech(没有的，当没有高度的时候就是等高布局)`  

### 3.  项目管理

弹性空间管理就是把富裕空间匀到每一个项目之上。

老版本：`-webkit-box-flex：n`

新版本：`flex-grow:n`  

相当于每一个项目的弹性因子就是 n 那么我们的富裕空间的分配就是 (nx/n1+n2+n3+…+nx) * 富裕空间大小 简单来说这个弹性因子就表示将来会分到的富裕空间的比例。

### 4. 新版flex特性

#### 1.容器新增特性

在新版的flex中使用 `flex-wrap` 来控制侧轴的方向 规定了侧轴的方向那么在元素在主轴的方向排列不下的时候不会对元素进行压缩而是采用向侧轴排列的方式进行，这样就会产生多行多列，此时我们侧轴的富裕空间 `align-items` 就失效了，必须采用 `align-content` 来解决。

##### 1. flex-wrap 

flex-wrap 属性控制了容器为单行/列还是多行/列。并且定义了侧轴的方向，新行/列将沿侧轴方向堆砌。

默认值：nowrap     不可继承

值：nowrap | wrap | wrap-reverse 

##### 2.align-content 

align-content 属性定义弹性容器的侧轴方向上有额外空间时，如何排布每一行/列。当弹性容器只有一行/列时无作用
默认值：stretch    不可继承  

值：
flex-start
​    所有行/列从侧轴起点开始填充。第一行/列的侧轴起点边和容器的侧轴起点边对齐。
​    接下来的每一行/列紧跟前一行/列。
flex-end
​    所有弹性元素从侧轴末尾开始填充。最后一个弹性元素的侧轴终点和容器的侧轴终点对齐。
​    同时所有后续元素与前一个对齐。
center
​    所有行/列朝向容器的中心填充。每行/列互相紧挨，相对于容器居中对齐。
​    容器的侧轴起点边和第一行/列的距离相等于容器的侧轴终点边和最后一行/列的距离。
space-between
​    所有行/列在容器中平均分布。相邻两行/列间距相等。
​    容器的侧轴起点边和终点边分别与第一行/列和最后一行/列的边对齐。
space-around
​    所有行/列在容器中平均分布，相邻两行/列间距相等。
​    容器的侧轴起点边和终点边分别与第一行/列和最后一行/列的距离是相邻两行/列间距的一半。
stretch
​    拉伸所有行/列来填满剩余空间。剩余空间平均的分配给每一行/列       

##### 3. flex-flow

flex-flow 属性是设置“flex-direction”和“flex-wrap”的简写

默认值：row nowrap    不可继承

控制主轴和侧轴的位置以及方向

#### 2. 项目新增特性

#####  1.order 

设置项目在主轴的排列优先级，数字越大优先级越高。

##### 2.align-self

项目自己管理自己的侧轴的富裕空间，而不归 `align-items` 管了。

##### 3. flex-basis 

flex-basis 在弹性空间管理的时候非常有用的，我们在上面的弹性空间管理的时候也就是 `flex-grows` 我们直接是把整个容器大小减去所有项目的大小然后除以增长因子的大小来计算的，实际上我们应该减去的不是容器的总大小而是`flex-basis` 的值，默认值就是容器的大小。所以他们的计算公式应该是：

>    可用空间 = (容器大小 - 所有相邻项目flex-basis的总和)
>    可扩展空间 = (可用空间/所有相邻项目flex-grow的总和)
>    每项获得的伸缩大小 = (伸缩基准值 + (可扩展空间 x flex-grow值))

##### 4.flex-shrink

收缩因子，实际上就是我们所有的item的总大小大于容器的大小他就会进行收缩但是收缩也会有一个收缩因子，但是他的收缩因子的默认值不是 0 而是 1，计算公式：

>    1.计算收缩因子与基准值乘的总和  
>    2.计算收缩因数
> ​              收缩因数=（项目的收缩因子*项目基准值）/第一步计算总和    
>    3.移除空间的计算
> ​              移除空间= 项目收缩因数 x 负溢出的空间   

### 5. flex应用

#### 1.等分布局

根据规则我们就需要将赋予空间全部分配给每一个项目，而我们需要的全部的富裕空间就是容器的所有的宽度。

```css
.item{
    flex-grows:1;
    flex-shrink:1;
    flex-basis:1;
}
```

上面的这一堆代码为了等分布局，所以我们有一个简写的方式就是

```css
.item{
    flex:1;
}
```

