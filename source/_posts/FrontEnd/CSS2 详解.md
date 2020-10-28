---
title: CSS2 从入门到精通
date: 2019-01-08 11:01:28
tags: CSS
categories:
 - FrontEnd
---



## 1. 常用的选择器

### 1. 元素选择器
* 作用：通过元素选择器可以选择指定的元素
* 语法：tag{}

```css
p{
  color: red;
}
h1{
  color: red;
}
```

### 2. id 选择器
* 作用：通过元素的id属性值选中唯一的一个元素
* 用法: #id{}

```css
#p1{
	font-size: 20px;
}
```

### 3.类选择器
* 通过元素的class属性值选中一组元素
* 语法：.class{}

```css
.p2{
  color: red;
}
			
.hello{
  font-size: 50px;
}
```

### 4. 选择器分组
* 通过选择器分组可以同时选中多个选择器对应的元素,简单来说就是同时为这些选择的元素应用相同的样式。他也称作为 **并集选择器 **就是上面的每一个逗号分开的相当于一个集合，那么相当于把他们合并起来到一个集合中只要元素命中其中一条规则就会应用样式。
* 语法：选择器1,选择器2,选择器N{}

```css
#p1 , .p2 , h1{
  background-color: yellow;
}
```

### 5. 通配选择器
* _他可以用来选中页面中的所有的元素_
* 语法：*{}

```css
*{
  color: red;
}
```

### 6. 交集选择器
* 作用：选择**同时满足所有选择器规则**的元素，所以也称作为交集选择器，元素必须命中所有的选择条件才能应用样式。下面的例子就相当于必须满足元素名为span并且拥有p3这个类
* 语法：选择器1选择器2选择器N{}

```css
span.p3{
				background-color: yellow;
}
```

### 7. 后代选择器
* 作用：选中指定元素的指定后代元素，下面的例子就是选择id为d1的span后代，注意后代是不用和祖先元素相邻。
* 语法：祖先元素 后代元素{}

```css
#d1 span{
	color: greenyellow;
}
```

### 8. 子选择器
* 作用：选中指定父元素的指定子元素,两者必须是相邻的
* 语法：父元素 > 子元素

```css
div > span{
				background-color: yellow;
}
```

> Tips:
> 父元素：直接包含子元素的元素<br />子元素：直接被父元素包含的元素<br />祖先元素：直接或间接包含后代元素的元素，父元素也是祖先元素<br />后代元素：直接或间接被祖先元素包含的元素，子元素也是后代元素<br />兄弟元素：拥有相同父元素的元素叫做兄弟元素


### 9. 伪类选择器
伪类专门用来表示元素的一种的特殊的状态，比如：访问过的超链接，比如普通的超链接，比如获取焦点的文本框<br />当我们需要为处在这些特殊状态的元素设置样式时，就可以使用伪类
##### 1. :link 表示普通的链接（没访问过的链接）

```css
a:link{color: yellowgreen;}
```

##### 2. :visited 表示访问过的链接
浏览器是通过历史记录来判断一个链接是否访问过,由于涉及到用户的隐私问题，所以使用visited伪类只能设置字体的颜色
##### 3. :hover伪类表示鼠标移入
##### 4. :active表示的是超链接被点击的状态
##### 5.:focus 光标为获得焦点
##### 6.::selection选中的内容
这个伪类在火狐中需要采用另一种方式编写::-moz-selection兼容火狐的

```css
p::-moz-selection{background-color: orange;}
```

兼容大部分浏览器的

```css
p::selection{background-color: orange;}
```

### 10. 伪元素选择器
使用伪元素来表示元素中的一些特殊的位置.
##### 1. :first-letter 内容中的第一个字母
##### 2. :first-line内容的第一行
##### 3.:before表示元素最前边的部分
##### 4. :after表示元素的最后边的部分
   一般before和after都需要结合content这个样式一起使用，   通过content可以向before或after的位置添加一些内容<br />:after
### 11. 属性选择器
* 作用：可以根据元素中的属性或属性值来选取指定元素
* 语法：
  * [属性名] 选取含有指定属性的元素
  * [属性名="属性值"] 选取含有指定属性值的元素
  * [属性名^="属性值"] 选取属性值以指定内容开头的元素
  * [属性名$="属性值"] 选取属性值以指定内容结尾的元素
### 12. 子元素选择器
##### 1. :first-child 可以选中第一个子元素:
##### 2.last-child 可以选中最后一个子元素

```css
body > p:first-child{background-color: yellow;}p:last-child{background-color: yellow;}
```

##### 3. :nth-child 可以选中任意位置的子元素
该选择器后边可以指定一个参数，指定要选中第几个子元素<br />even 表示偶数位置的子元素<br />odd 表示奇数位置的子元素

```css
p:nth-child(odd){background-color: yellow;}
```
`:first-of-type`,`:last-of-type`，`:nth-of-type`,`:first-child`这些非常的类似，只不过child，是在所有的子元素中排列而type，是在当前类型的子元素中排列

```css
p:first-of-type{background-color: yellow;}
```

### 13.兄弟选择器
为挨着的兄弟元素添加样式，其中 + 表示后一个，而 ~ 则表示前一个

```css
span ~ p{
  background-color: yellow;
}
span + p{
  background-color: yellow;
}
```

## 2. 样式继承
像儿子可以继承父亲的遗产一样，在CSS中，祖先元素上的样式，也会被他的后代元素所继承,利用继承，可以将一些基本的样式设置给祖先元素，这样所有的后代元素将会自动继承这些样式。但是并不是所有的样式都会被子元素所继承，比如：`背景相关的样式`都不会被继承 `边框相关的样式` `定位相关`的

## 3. 样式优先级
优先级的规则：
1. 内联样式 ， 优先级  1000
2. id选择器，优先级   100
3. 类和伪类， 优先级   10
4. 元素选择器，优先级 1
5. 通配* ，    优先级 0
6. 继承的样式，没有优先级
7. 并集选择器的优先级是单独计算
8. 可以在样式的最后，添加一个!important，则此时该样式将会获得一个最高的优先级，将会优先于所有的样式显示甚至超过内联样式，但是在开发中尽量避免使用!important
## 4. 长度单位
### 1.像素 px
像素是我们在网页中使用的最多的一个单位，一个像素就相当于我们屏幕中的一个小点，我们的屏幕实际上就是由这些像素点构成的但是这些像素点，是不能直接看见。不同显示器一个像素的大小也不相同，显示效果越好越清晰，像素就越小，反之像素越大。
### 2. 百分比 %
也可以将单位设置为一个百分比的形式，这样浏览器将会根据其父元素的样式来计算该值使用百分比的好处是，当父元素的属性值发生变化时，子元素也会按照比例发生改变在我们创建一个自适应的页面时，经常使用百分比作为单位
### 3. em
em和百分比类似，它是相对于当前元素的字体大小来计算的 1em = 1font-size 使用em时，当字体大小发生改变时，em也会随之改变当设置字体相关的样式时，经常会使用em
### 4. 行间距
在CSS并没有为我们提供一个直接设置行间距的方式， 我们只能通过设置行高来间接的设置行间距，行高越大行间距越大 使用line-height来设置行高 行高类似于我们上学单线本，单线本是一行一行，线与线之间的距离就是行高， 网页中的文字实际上也是写在一个看不见的线中的，而文字会默认在行高中垂直居中显示。<br />行间距 = 行高 - 字体大小<br /><br />通过设置line-height可以间接的设置行高，<br />可以接收的值：<br />1.直接就收一个大小<br />2.可以指定一个百分数，则会相对于字体去计算行高<br />3.可以直接传一个数值，则行高会设置字体大小相应的倍数<br />

```css
p{
    line-height: 200%;
    line-height: 20px;
    line-height: 2;
}
```

对于单行文本来说，可以将行高设置为和父元素的高度一致， 这样可以是单行文本在父元素中垂直居中

## 5. 文本样式
### 1. text-transform可以用来设置文本的大小写
可选值：
* none 默认值，该怎么显示就怎么显示，不做任何处理
* capitalize 单词的首字母大写，通过空格来识别单词
* uppercase 所有的字母都大写
* lowercase 所有的字母都小写<br />
### 2.text-decoration可以用来设置文本的修饰
可选值：
* none：默认值，不添加任何修饰，正常显示
* underline 为文本添加下划线
* overline 为文本添加上划线
* line-through 为文本添加删除线

超链接会默认添加下划线，也就是超链接的text-decoration的默认值是underline如果需要去除超链接的下划线则需要将该样式设置为none<br />
### 3.letter-spacing可以指定字符间距
### 4. word-spacing可以设置单词之间的距离
实际上就是设置词与词之间空格的大小<br />
### 5.text-align用于设置文本的对齐方式
可选值：
* left 默认值，文本靠左对齐
* right ， 文本靠右对齐
* center ， 文本居中对齐
* justify ， 两端对齐通过调整文本之间的空格的大小，来达到一个两端对齐的目的<br />
### 6.text-indent用来设置首行缩进
当给它指定一个正值时，会自动向右侧缩进指定的像素 如果为它指定一个负值，则会向左移动指定的像素, 通过这种方式可以将一些不想显示的文字隐藏起来 这个值一般都会使用em作为单位

## 6. 盒模型
使用width来设置盒子内容区的宽度，使用height来设置盒子内容区的高度。**width和height只是设置的盒子内容区的大小，而不是盒子的整个大小，盒子可见框的大小由内容区，内边距和边框共同决定。 **内联元素不能设置 height 和 width ，如果非要设置必须修改 display 。**
### 1. 为元素设置边框
要为一个元素设置边框必须指定三个样式<br />
#### 1. border-width:边框的宽度<br />
使用border-width可以分别指定四个边框的宽度如果在border-width指定了四个值，则四个值会分别设置给 上 右 下 左，按照顺时针的方向设置的<br />    如果指定三个值，则三个值会分别设置给    上  左右 下<br />    如果指定两个值，则两个值会分别设置给 上下 左右<br />    如果指定一个值，则四边全都是该值<br />
#### 2.border-color:边框颜色<br />
和宽度一样，color也提供四个方向的样式，可以分别指定颜色
#### 3. border-style:边框的样式<br />
* none，默认值，没有边框
* solid 实线
* dotted 点状边框
* dashed 虚线
* double 双线

### 2. 内边距（padding）
指的是盒子的内容区与盒子边框之间的距离*  一共有四个方向的内边距,内边距会影响盒子的可见框的大小，元素的背景会延伸到内边距.

### 3. 外边距
外边距指的是当前盒子与其他盒子之间的距离，他不会影响可见框的大小，而是会影响到盒子的位置。**由于页面中的元素都是靠左靠上摆放的，所以注意当我们设置上和左外边距时，会导致盒子自身的位置发生改变**。_<br />margin还可以设置为auto，auto一般只设置给水平方向的margin如果只指定，左外边距或右外边距的margin为auto则会将外边距设置为最大值垂直方向外边距如果设置为auto，则外边距默认就是0如果将left和right同时设置为auto，则会将两侧的外边距设置为相同的值，就可以使元素自动在父元素中居中，所以我们经常将左右外边距设置为auto。

#### 1. 垂直外边距重叠
垂直外边距的重叠在网页中**相邻**垂直方向**的外边距会发生外边距的重叠所谓的外边距重叠指兄弟元素之间的相邻外边距会取最大值而不是取和.<br />      如果父子元素的**垂直外边  距相邻 **了，则子元素的外边距会设置给父元素.
#### 2. 如何解决外边距重叠
1. 在两个元素之家加其他元素
2. 添加 border
3. 添加 padding 

**注意：内联元素是不支持垂直方向的外边距的，其他的都和块级元素相同。**<br />
## 7. 元素展示方式
### 1. display
将一个内联元素变成块元素， 通过display样式可以修改元素的类型<br />可选值：<br />
* inline：可以将一个元素作为内联元素显示
* block: 可以将一个元素设置块元素显示
* inline-block：将一个元素转换为行内块元素可以使一个元素既有行内元素的特点又有块元素的特点既可以设置宽高，又不会独占一行
* none: 不显示元素，并且元素不会在页面中继续占有位置使用该方式隐藏的元素，不会在页面中显示，并且不再占据页面的位置<br />
### 2. visibility
可以用来设置元素的隐藏和显示的状态<br />可选值：
* visible 默认值，元素默认会在页面显示
* hidden 元素会隐藏不显示,使用 visibility:hidden;隐藏的元素虽然不会在页面中显示，但是它的位置会依然保持

### 3. _overflow_
子元素默认是存在于父元素的内容区中，<br />      理论上讲子元素的最大可以等于父元素内容区大小<br />   如果子元素的大小超过了父元素的内容区，则超过的大小会在父元素以外的位置显<br />      超出父元素的内容，我们称为溢出的内容<br /> 父元素默认是将溢出内容，在父元素外边显示，<br />   通过overflow可以设置父元素如何处理溢出内容：<br />      可选值：<br />         - visible，默认值，不会对溢出内容做处理，元素会显示<br />         - hidden, 溢出的内容，会被修剪，不会显示<br />         - scroll, 会为父元素添加滚动条，通过拖动滚动条该属性不论内容是否溢出，都会添加水平<br />         - auto，会根据需求自动添加滚动条，需要水平就添加水平需要垂直就添加垂直都不需要就都不加

## 8. 文档流
文档流文档流处在网页的最底层，它表示的是一个页面中的位置，我们所创建的元素默认都处在文档流中<br />元素在文档流中的特点<br />
### 1.块元素
1.块元素在文档流中会独占一行，块元素会自上向下排列。<br />      2.块元素在文档流中默认宽度是父元素的100%<br />      3.块元素在文档流中的高度默认被内容撑开<br />
### 2.内联元素
1.内联元素在文档流中只占自身的大小，会默认从左向右排列，如果一行中不足以容纳所有的内联元素，则换到下一行，继续自左向右。<br />      2.在文档流中，内联元素的宽度和高度默认都被内容撑开

## 9. 浮动
块元素在文档流中默认垂直排列， 如果希望块元素在页面中水平排列，可以使块元素脱离文档流使用float来使元素浮动，从而脱离文档流<br />可选值：
* none，默认值，元素默认在文档流中排列
* left，元素会立即脱离文档流，向页面的左侧浮动
* right，元素会立即脱离文档流，向页面的右侧浮动

### 1. 浮动的规则
当为一个元素设置浮动以后（float属性是一个非none的值），   元素会立即脱离文档流，元素脱离文档流以后，它下边的元素会立即向上移动   元素浮动以后，会尽量向页面的左上或右上漂浮，   直到遇到父元素的边框或者其他的浮动元素<br />如果浮动元素上边是一个没有浮动的块元素，则浮动元素不会超过块元素。<br />浮动的元素不会超过他上边的兄弟元素，最多最多一边齐。<br />浮动的元素不会盖住文字，文字会自动环绕在浮动元素的周围<br />在文档流中，子元素的宽度默认占父元素的全部当元素设置浮动以后，会完全脱离文档流.块元素脱离文档流以后，高度和宽度都被内容撑开<br />       开启span的浮动内联元素脱离文档流以后会变成块元素

### 2. 浮动后高度塌陷
在文档流中，父元素的高度默认是被子元素撑开的，也就是子元素多高，父元素就多高。但是当为子元素设置浮动以后，子元素会完全脱离文档流，此时将会导致子元素无法撑起父元素的高度，导致父元素的高度塌陷。由于父元素的高度塌陷了，则父元素下的所有元素都会向上移动，这样将会导致页面布局混乱。<br /> 所以在开发中一定要避免出现高度塌陷的问题,我们可以将父元素的高度写死，以避免塌陷的问题出现，但是一旦高度写死，父元素的高度将不能自动适应子元素的高度，所以这种方案是不推荐使用的。<br />

### 3. 解决高度塌陷问题
#### 1. 开启BFC
根据W3C的标准，在页面中元素都一个隐含的属性叫做Block Formatting Context简称BFC，该属性可以设置打开或者关闭，默认是关闭的。当开启元素的BFC以后，元素将会具有如下的特性：<br />   1.父元素的垂直外边距不会和子元素重叠    <br />   2.开启BFC的元素不会被浮动元素所覆盖<br />   3.开启BFC的元素可以包含浮动的子元素<br />如何开启元素的BFC<br />   1.设置元素浮动使用这种方式开启，虽然可以撑开父元素，但是会导致父元素的宽度丢失而且使用这种方式也会导致下边的元素上移，不能解决问题。这是因为开启浮动的会计元素宽高都是默认被撑起来的，除非自动设置宽高。<br />   2.设置元素绝对定位<br />   3.设置元素为inline-block可以解决问题，但是会导致宽度丢失，不推荐使用这种方式<br />   4.将元素的overflow设置为一个非visible的值<br />**推荐方式：将overflow设置为hidden是副作用最小的开启BFC的方式。 **<br />是在IE6及以下的浏览器中并不支持BFC，所以使用这种方式不能兼容IE6。在IE6中虽然没有BFC，但是具有另一个隐含的属性叫做hasLayout，该属性的作用和BFC类似，所在IE6浏览器可以通过开hasLayout来解决该问题开启方式很多，我们直接使用一种副作用最小的：直接将元素的zoom设置为1即可<br />
* zoom表示放大的意思，后边跟着一个数值，写几就将元素放大几倍
* zoom:1表示不放大元素，但是通过该样式可以开启hasLayout
* zoom这个样式，只在IE中支持，其他浏览器都不支持

```css
.parent{
	zoom:1;
  overflow: hidden;
}
```

#### 2. 清除浮动
可以直接在高度塌陷的父元素的最后，添加一个空白的div， 然后在对其进行清除浮动，   由于这个div并没有受到浮动元素的影响，所以他应该处在原来的元素在的时候的位置这样就相当于原来的元素还在，所以是可以撑开父元素的高度的，这样可以通过这个空白的div来撑开父元素的   基本没有副作用使用这种方式虽然可以解决问题，但是会在页面中添加多余的结构。

```css
.clear{
				clear: both;
}
```

#### 3. 伪类
通过after伪类，选中父元素的后边可以通过after伪类向元素的最后添加一个空白的块元素，然后对其清除浮动，这样做和添加一个div的原理一样，可以达到一个相同的效果，而且不会在页面中添加多余的div，这是我们最推荐使用的方式，几乎没有副作用。

```css
.clearfix:after{
				/*添加一个内容*/
				content: "";
				/*转换为一个块元素*/
				display: block;
				/*清除两侧的浮动*/
				clear: both;
			}
```

#### 4. 最佳实践

```css
.clearfix:before,
.clearfix:after{
  content: "";
  display: table;
  clear: both;
}
			
.clearfix{
  zoom: 1;
}
```
经过修改后的clearfix是一个多功能的,既可以解决高度塌陷，又可以确保父元素和子元素的垂直外边距不会重叠。子元素和父元素相邻的垂直外边距会发生重叠，子元素的外边距会传递给父元素 使用空的table标签可以隔离父子元素的外边距，阻止外边距的重叠
### 4. 清除浮动
由于受到box1浮动的影响，box2整体向上移动了100px<br />我们有时希望清除掉其他元素浮动对当前元素产生的影响，这时可以使用<br />clear可以用来清除其他浮动元素对当前元素的影响<br />可选值：<br />      none，默认值，不清除浮动<br />      left，清除左侧浮动元素对当前元素的影响<br />      right，清除右侧浮动元素对当前元素的影响<br />      both，清除两侧浮动元素对当前元素的影响清除对他影响最大的那个元素的浮动

### 5. 总结

​	在元素浮动的时候我们将元素看成两层。上面一层是文字相关的而下面一层则是和盒模型相关的。而一个元素开启浮动那么这个元素就被提升了半层。也就是两个排列的 box1 box2 ，在box1 浮动以后由于他之浮动半层，所以box2的盒模型部分会插入到box1下面，也就是box1会有三层，而box2的文字那一层是被卡在外面了，不会进去。

## 10. 定位
定位指的就是将指定的元素摆放到页面的任意位置 通过定位可以任意的摆放元素 通过position属性来设置元素的定位<br />可选值：<br />
* static：默认值，元素没有开启定位<br />
* relative：开启元素的相对定位<br />
* absolute：开启元素的绝对定位<br />
* fixed：开启元素的固定定位（也是绝对定位的一种）

当开启了元素的定位（position属性值是一个非static的值）时， 可以通过left right top bottom四个属性来设置元素的偏移量<br />
* left：元素相对于其定位位置的左侧偏移量<br />
* right：元素相对于其定位位置的右侧偏移量<br />
* top：元素相对于其定位位置的上边的偏移量<br />
* bottom：元素相对于其定位位置下边的偏移量<br />

通常偏移量只需要使用两个就可以对一个元素进行定位， 一般选择水平方向的一个偏移量和垂直方向的偏移量来为一个元素进行定位
### 1. 相对定位
当元素的position属性设置为relative时，则开启了元素的相对定位<br />
1. 当开启了元素的相对定位以后，而不设置偏移量时，元素不会发生任何变化
2. 相对定位是相对于元素在文档流中原来的位置进行定位
3. 相对定位的元素不会脱离文档流
4. 相对定位会使元素提升一个层级
5. 相对定位不会改变元素的性质，块还是块，内联还是内联

### 2. 绝对定位
当position属性值设置为absolute时，则开启了元素的绝对定位<br />绝对定位：<br />
1. 开启绝对定位，会使元素脱离文档流<br />
2. 开启绝对定位以后，如果不设置偏移量，则元素的位置不会发生变化<br />
3. 绝对定位是相对于离他最近的开启了定位的祖先元素进行定位的（一般情况，开启了子元素的绝对定位都会同时开启父元素的相对定位）如果所有的祖先元素都没有开启定位，则会相对于浏览器窗口进行定位
4. 绝对定位会使元素提升一个层级<br />
5. 绝对定位会改变元素的性质，内联元素变成块元素，块元素的宽度和高度默认都被内容撑开
### 3.  固定定位
当元素的position属性设置fixed时，则开启了元素的固定定位 固定定位也是一种绝对定位，它的大部分特点都和绝对定位一样 不同的是： 固定定位永远都会相对于浏览器窗口进行定位 固定定位会固定在浏览器窗口某个位置，不会随滚动条滚动IE6不支持固定定位.

### 4. 总结

浮动元素的包含块为最近的块级祖先元素。而定位则比较复杂：

- 初始包含块由用户浏览器创建，在html中就是html标签。但是有些浏览器也会采用body作为根元素，大多数浏览器中初始包含块都是一个视窗大小的矩形。但是视口不等于初始包含块。
- 对于非根元素如果是relative或者static则包含块是最近的块级框。（或者说是他本来应该处于的位置。）
- 对于非根元素如果是absolute则包含块是最近的开启定位的元素(非static)的内边距的边界，也就是说是由边框的边界确定的。

由于我们的行业规范是 div+css 所以我们不考虑内联元素的情况，因为div是块级元素。

**定位是提升一层，和浮动不一样。**

**注意：** 

对于一些常用的一些样式我们需要知道他们的默认值是什么比如：

left top right bottom width height 的默认值都是auto不是 0 也不是100%

margin 和 padding 默认则是 0 如果我们采用百分比百分比参照的是父级元素的 width 的而不是当前元素的大小。

## 11. 常用布局

### 1. 三列布局

#### 1.定位实现方式

​	三列的话我们可以让其中的两列分别使用绝对定位，然后由于绝对定位默认是依照浏览器的窗口（或者将body设为realtive 然后相对于 body），所以可以让两边的直接在两边，而中间的使用块元素直接让宽度充满整个空闲部分。

​	然后由于随着我们浏览器变小我们需要设置一个body的最小宽否则有可能让我们的中间部分压没了。

​	其实这个是有问题的，就是当我们压缩矿口的时候会出现滚动条如果我们设置了 min-width ，这样我们的右边以浏览器窗口对其就会有问题。

​	**所以不推荐使用。**

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
			body{
				/*2*left+right*/
				min-width: 600px;
			}
			div{
				height: 100px;
			}
			#left,#right{
				width: 200px;
				background: pink;
			}
			#middle{
				background: deeppink;
				padding: 0 200px;
			}
			#left{
				position: absolute;
				left: 0;
				top: 0;
			}
			#right{
				position: absolute;
				right: 0;
				top: 0;
			}
		</style>
	</head>
	<body style="position: relative;">
		<div id="left">left</div>
		<div id="middle">middle</div>
		<div id="right">right</div>
	</body>
</html>
```

#### 2. 使用浮动

将左右两列用浮动分别让他们在两边展示，而中间的为块级元素这样他的盒子布局的那一层就会浮上去，而文字刚好就被卡住。但是这个东西的前提就是内容区必须放在最后内容区的第二层才会自动的浮动上去。这样一来就有一个问题就是内容区是在文档流中最后被加载的。

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title></title>
		<!--
			1.两边固定  当中自适应
			2.当中列要完整显示
			3.当中列要优先加载
		-->
		<style type="text/css">
			
			*{
				margin: 0;
				padding: 0;
			}
			body{
				/*2*left+right*/
				min-width: 600px;
			}
			div{
				height: 100px;
			}
			#left,#right{
				width: 200px;
				background: pink;
			}
			#left{
				float: left;
			}
			#right{
				float: right;
			}
			#middle{
				background: deeppink;
			}
		</style>
	</head>
	<body>
		<div id="left">left</div>
		<div id="right">right</div>
		<div id="middle">middle</div>
	</body>
</html>
```

#### 3. 圣杯布局

​	首先我们还是使用 float:left 让左右两部的元素浮动起来，但是由于我们的content是块级元素没办法让下面的元素上来，那么我们只能把它也设置为浮动，这样一来他的宽度就是被内容撑开了，我们需要设置他的宽度为body的100%，现在是content占据100%但是它允许下面浮动的元素上去。如果我们使用浮动让他们上去肯定就和上面的方法一样了，我们需要的是中间的元素有限加载也就是他应该出现在文档流的最前面。

​	可以采用负的margin，可以想象，我们的left的外边界和content是重叠的，那么我们如果移动他的外边界到content的起始位置那么他是不是就上去了呢？确实如此但是注意一个问题就是我们改变了外边距元素的位置实际上是不会动的，也就是在文档流中他还是在下面那一行，right的左边还是left这个块。所以我们right需要上去的时候必须margin为 -200

​	这样就好了，但是content的内容被挡住了。我们可以设把body变小，然后通过定位完成整个布局。

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title></title>
		<!--
			1.两边固定  当中自适应
			2.当中列要完整显示
			3.当中列要优先加载
		-->
		
		<!--
			浮动:	搭建完整的布局框架
			margin 为赋值:调整旁边两列的位置(使三列布局到一行上)
			使用相对定位:调整旁边两列的位置（使两列位置调整到两头）
		-->
		
		<style type="text/css">
			*{
				margin: 0;
				padding: 0;
			}
			body{
				min-width: 600px;
			}
			#content{
				padding: 0 200px;
			}
			#header,#footer{
				height: 20px;
				text-align: center;
				border: 1px solid  deeppink;
				background: gray;
			}
			#content .middle{
				float: left;
				width: 100%;
				background: pink;
				/*padding: 0 200px;*/
			}
			#content .left{
				position: relative;
				left: -200px;
				margin-left: -100%;
				float: left;
				width: 200px;
				background: yellow;
			}
			#content .right{
				position: relative;
				right: -200px;
				margin-left: -200px;
				float: left;
				width: 200px;
				background: yellow;
			}
			
			
			
			.clearfix{
				*zoom: 1;
			}
			.clearfix:after{
				content: "";
				display: block;
				clear: both;
			}
		</style>
	</head>
	<body>
		<div id="header">header</div>
		<div id="content" class="clearfix">
			<div class="middle">
				<h4>middle</h4>
			</div>
			<div class="left">left</div>
			<div class="right">right</div>
		</div>
		<div id="footer">footer</div>
	</body>
</html>
```

#### 4. 双飞翼布局

​	在上面的圣杯布局的基础之上做的改进，其实做到两边都浮动然后只是content的内容不能居中，那么他们之间的区别就是如何让内容居中上。

​	圣杯是把padding应用在外面的body上，然后使用定位让他们分到两边。我们会考虑为什么不能用padding在conten上呢，实际上我们在content浮动以后使用padding就会把整个元素变大变宽，那么我们以前设置的left和right的margin的值需要重新设置不然就是错乱的。所以他在内部加了一层结构然后使用padding，这样的话由于left和right的margin只和外面的content的外边界有关系，所以里面的padding并不能影响整体的布局。

### 2. 伪等高布局

​	如果我们两列高度不一样，需要让两列一一样高，我们采用的套路就是让他们的padding超级大，然后把整个的元素高度很高，然后让margin为负值把边界收回来，那么也就是他们的外部的盒子的外边距收回来了，此时再用 overflow:hidden;就好。

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
			#wrap{
				width: 750px;
				border: 1px solid;
				margin: 0 auto;
				overflow: hidden;
			}
			#wrap .left{
				float: left;
				width: 200px;
				background: pink;
				padding-bottom: 1000px;
				margin-bottom: -1000px;
			}
			#wrap .right{
				float: left;
				width: 500px;
				background: deeppink;
				padding-bottom: 1000px;
				margin-bottom: -1000px;
			}
			
			
			
			.clearfix{
				*zoom: 1;
			}
			.clearfix:after{
				content: "";
				display: block;
				clear: both;
			}
		</style>
	</head>
	<body>
		<div id="wrap" class="clearfix">
			<div class="left">
				left <br />
				left <br />
				left <br />
				left <br />
				left <br />
				left <br />
				left <br />
				left <br />
				left <br />
				left <br />
				left <br />
				left <br />
				left <br />
				left <br />
				left <br />
				left <br />
				left <br />
				left <br />
				left <br />
				left <br />
				left <br />
				left <br />
				left <br />
				left <br />
			</div>
			<div class="right">
				right<br />
				right<br />
				right<br />
				right<br />
				right<br />
				right<br />
				right<br />
				right<br />
				right<br />
				right<br />
				right<br />
				right<br />
				right<br />
				right<br />
			</div>
		</div>
	</body>
</html>
```



## 12. 滚动条

​	一般的在我们内容大于浏览器视窗的时候会出现滚动条，这时滚动条是应用在文档上的不是在html也不是在body上。如果我们希望修改这个设置我们有一条规则就是：

​	当我们只设置 body 和 html 中的overflow的时候他们这个属性是应用给文档的，但是如果两者同时使用这个属性那么html的设置给文档而body的设置给body。

​	实际上我们的固定定位和绝对定位的区别就在于滚动条上面，如果我们自己设置了滚动条那么就可以采用绝对定位来模拟固定定位。在移动端一般我们都是这么做的，因为我们需要禁用掉系统默认的滚动条。

​	**注意：如果我们希望获得视窗的大小的话我们需要从html到body一层层的100%的继承下来不然height只会被内容撑开**

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
				overflow: hidden;
				height: 100%;
			}
			
			body{
				overflow: auto;
				height: 100%;
			}
			
			#test{
				position: absolute;
				left: 50px;
				top: 50px;
				width: 100px;
				height: 100px;
				background: pink;
			}
		</style>
	</head>
	<body>
		<div id="test">
			
		</div>
		<div style="height: 1000px;">
			csjkahcjk <br />
			csjkahcjk <br />
			csjkahcjk <br />
			csjkahcjk <br />
			csjkahcjk <br />
			csjkahcjk <br />
			csjkahcjk <br />
			csjkahcjk <br />
		</div>
	</body>
</html>
```

## 13. 黏贴布局

​	黏贴布局就是当内容没有达到底部的时候我们的内容在底部不动，而内容太多的时候可以把我们的底部挤到下面去。

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<meta name="viewport" content="width=device-width,initial-scale=1.0,user-scalable=no"/>
		<title></title>
		<style type="text/css">
			*{
				margin: 0;
				padding: 0;
			}
			html,body{
				height: 100%;
			}
			#wrap{
				min-height: 100%;
				background: pink;
				text-align: center;
				overflow: hidden;
			}
			#wrap .main{
				padding-bottom:50px ;
			}
			#footer{
				height: 50px;
				line-height: 50px;
				background: deeppink;
				text-align: center;
				margin-top: -50px;
			}
		</style>
	</head>
	<body>
		<div id="wrap" >
			<div class="main">
				main <br />
				main <br />
				main <br />
			</div>
		</div>
		<div id="footer">
			footer
		</div>
	</body>
</html>
```

