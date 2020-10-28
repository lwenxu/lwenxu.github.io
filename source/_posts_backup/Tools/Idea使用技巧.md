---
title: Idea使用技巧
date: 2017-05-05 13:09:40
tags:
categories:
  -开发工具
---
## 【编辑】
souf:
System.out.printf("");

sout:
System.out.println();

<!--more-->

soutm:
System.out.println("demo.main");

soutp:
System.out.println("args = [" + args + "]");

soutv:
System.out.println("true = " + true);

Shift+Click，可以关闭文件

Ctrl+[ / ]，可以跑到大括号的开头与结尾

Ctrl+N，可以快速打开类

Ctrl+Shift+N，可以快速打开文件

Ctrl+Q，可以看到当前方法的声明

Ctrl+Shift+Insert，可以选择剪贴板内容并插入

Alt+Insert，可以生成构造器/Getter/Setter等

Ctrl+Enter，导入包，自动修正

Ctrl+Alt+L，格式化代码

Ctrl+Alt+I，将选中的代码进行自动缩进编排，这个功能在编辑 JSP 文件时也可以工作

Ctrl+R，替换文本

Ctrl+F，查找文本

Ctrl+Shift+Space，自动补全代码

Shift+F6，重构 – 重命名

Ctrl+X，删除行

Ctrl+D，复制行

Ctrl+/或Ctrl+Shift+/，注释（//或者）

Ctrl+J，自动代码（例如：serr）

Ctrl+Alt+J，用动态模板环绕

Ctrl+H，显示类结构图（类的继承层次）

Ctrl+Alt+left/right，返回至上次浏览的位置

Alt+Up/Down，在方法间快速移动定位

F2 或 Shift+F2，高亮错误或警告快速定位

Alt+F3，逐个往下查找相同文本，并高亮显示

Ctrl+O，重写方法

Ctrl+Shift+J，整合两行

Alt+F8，计算变量值

Shift+Esc，不仅可以把焦点移到编辑器上，而且还可以隐藏当前（或最后活动的）工具窗口

Ctrl+Y，删除当前行

Ctrl+Shift+F，全局查找

Ctrl+U，转到父类

Ctrl+G，定位行

Ctrl+Backspace，按单词删除

Ctrl+”+/-”，当前方法展开、折叠

Ctrl+Shift+”+/-”，全部展开、折叠

## 【调试部分、编译】
Ctrl+F2，停止
Alt+Shift+F9，选择 Debug
Alt+Shift+F10，选择 Run
Ctrl+Shift+F9，编译
Ctrl+Shift+F10，运行
Ctrl+Shift+F8，查看断点
F8，步过
F7，步入
Shift+F7，智能步入
Shift+F8，步出
Alt+Shift+F8，强制步过
Alt+Shift+F7，强制步入
Alt+F9，运行至光标处
Ctrl+Alt+F9，强制运行至光标处
F9，恢复程序
Alt+F10，定位到断点
Ctrl+F8，切换行断点
Ctrl+F9，生成项目
Alt+1，项目
Alt+2，收藏
Alt+6，TODO
Alt+7，结构
Ctrl+Shift+C，复制路径
Ctrl+Alt+Shift+C，复制引用，必须选择类名
Ctrl+Alt+Y，同步
Ctrl+~，快速切换方案（界面外观、代码风格、快捷键映射等菜单）
Shift+F12，还原默认布局
Ctrl+Shift+F12，隐藏/恢复所有窗口
Ctrl+F4，关闭
Ctrl+Shift+F4，关闭活动选项卡
Ctrl+Tab，转到下一个拆分器
Ctrl+Shift+Tab，转到上一个拆分器
##【重构】
Ctrl+Alt+Shift+T，弹出重构菜单
Shift+F6，重命名
F6，移动
F5，复制
Alt+Delete，安全删除
Ctrl+Alt+N，内联
ctrl+shift+alt+j 列操作
##【查找】
Ctrl+F，查找
Ctrl+R，替换
F3，查找下一个
Shift+F3，查找上一个
Ctrl+Shift+F，在路径中查找
Ctrl+Shift+R，在路径中替换
Ctrl+Shift+S，搜索结构
Ctrl+Shift+M，替换结构
Alt+F7，查找用法
Ctrl+Alt+F7，显示用法
Ctrl+F7，在文件中查找用法
Ctrl+Shift+F7，在文件中高亮显示用法



# 1. 跳转

### 1.上次编辑的位置

![CgVzPH.png](https://s1.ax1x.com/2018/05/20/CgVzPH.png)

### 2.上次查看的位置

![CgZSGd.png](https://s1.ax1x.com/2018/05/20/CgZSGd.png)

### 3.书签跳转

使用Ctrl+F11可以输入一个带有数字字母标记的书签，比如我们使用的标签名是1则我们使用Ctrl+1/N可以跳转到对应的位置。

另外我们可以在Favorite中找到对应的书签

![CgZ9xI.png](https://s1.ax1x.com/2018/05/20/CgZ9xI.png)

### 4.收藏类或者方法

把光标移动到类里面或者上面，使用add to favorite 添加到对应的收藏夹，如果放到方法里面则是添加方法。

![CgZAZ8.png](https://s1.ax1x.com/2018/05/20/CgZAZ8.png)

### 5.使用macsIdea插件进行跳转

在keymap中配置AceJumpWords的快捷键为alt+j

按下alt+j 然后输入我们需要跳转的字母则会生成字母对应位置的一个快捷键，我们就可以跳转过去。

### 6.vim分屏

:sp / :vsp

ctrl+w -> h/l 左右

ctrl+w -> j/k上下

ctrl+w -> c 关闭

ctrl+w >->+/-/= 尺寸

# 2.列操作

[![CgZuzn.md.png](https://s1.ax1x.com/2018/05/20/CgZuzn.md.png)](https://imgchr.com/i/CgZuzn)

# 3.posfix

![CgZGoF.png](https://s1.ax1x.com/2018/05/20/CgZGoF.png)

## 1. for

100.fori 生成100 的for循环

## 2. sout

“hello”.sout

## 3.field

name.field 直接创建字段并且在构造方法中赋值

## 4.return

name.return 

## 5.nn

不空  name.nn

# 4.自动修复

alt+enter

这个是一个多功能的快捷键，根据不同场景有不同效果。

- 自动创建函数，当函数没有创建的时候
- 代码优化
- string的format和builder
- 接口的实现，subclass
- 单词拼写
- 导包

# 5.重构

## 1. shift+f6 

当前的方法名或者变量名都会被选中，然后我们只需要修改一个地方就可以完成整个的修改

## 2.ctrl+f6

在函数名字上使用这个快捷键就可以进行函数签名的重构。

## 3.抽取变量

![CgZjO0.png](https://s1.ax1x.com/2018/05/20/CgZjO0.png)

# 6. 调试

## 1. ctrl+f8 添加断点 

## 2.shift+f9 调试程序

## 3.f8 单步运行

## 4.resume 调到下个断点没有断点结束程序

## 5.ctrl+shift+f8 按两次哦 ，查看所有断点

## 6.mute breakpoints 禁止所有断点

## 7.ctrl+shift+f8条件断点

## 8.alt+f8表达式求值

## 9.alt+f9 运行到执行行

## 10.f2修改值

所有的导包，以及优化 ctrl+shift+o

查看所有的实现 Ctrl+H