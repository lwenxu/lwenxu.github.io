---
title: 关于PHP的回调函数
date: 2017-05-07 19:49:31
tags:
categories:
  -PHP
---
![](http://upload-images.jianshu.io/upload_images/1568656-0ad89e4d6ebe34e3.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


1. 普通函数的定义及调用与js相似,这个定义方式无需返回值,哪怕是有返回值在声明的时候也无需添加
2. 再后来的PHP版本中是添加了一向很有用的功能就是可在函数定义之前进行调用

```
echo add(1,2);
echo "</br>";
function add($a,$b){
    return $a+$b;
}
function sub($a,$b){
    return $a-$b;
}
echo add(23,12);
echo "</br>";
echo sub(23,22);
echo "</br>";
```

1. 下面是一个非常有用的功能就是变量函数,顾名思义就是将函数作为一个变量。
2. 其优点在于用同一个变量可以调用不同的函数,非常类似于函数的多态调用。

```
$var="add";
echo $var(4,2);
echo "</br>";
$var="sub";
echo $var(4,2);
echo "</br>";
```




* 回调函数就是在给一个函数传入一个简单的参数无法解决问题的时候给他传入一个过程,从而达到目的

* 在函数调用时给他传入一个函数作为参数就是函数回调。
```
$arr=array(2,3,5,4,1,6,7,9,8);
var_dump($arr);
echo "</br>";
```
```
//这里是自定义回调函数,返回-1是指将两个元素交换,0和1是不发生改变。
function myrule($a,$b){
    if ($a>$b){
        return 1;
    }
    elseif ($a<$b){
        return -1;
    }
    else{
        return 0;
    }
}
//usort就是系统函数,但是他的第二个参数是回调函数,这个函数参数就是排序规则
usort($arr,"myrule");
var_dump($arr);
echo "</br>";
```


 * 自己写回调函数,使用变量函数以及回调完成一个简单的过滤条件,如果需多个条件同时满足给一个&&关系即可。
 * 其中使用的变量函数可以使用系统中的一个叫做call_user_func_array()的函数进行调用,他有两个参数分别是回调函数名称,以及参数数组
 * call_user_func_array("demo",array(1,3));其优点在于array中的参数的数量可以比原函数的少,既有默认缺省参数的时候。

```
//rule1除去数组中是三的倍数的数
function rule1($a){
    if ($a%3==0){
        return true;
    }else{
        return false;
    }
}
//rule2是除去数组中的回文数（从左边读与从右边读是一样的）
function rule2($a){
    if ($a==strrev($a)){
        return true;
    }else{
        return false;
    }
}
function demo($n,$var){
    for ($i=0;$i<$n;$i++){
        if (call_user_func_array($var,array(23)))
        //if ($var($i))
        {
            continue;
        }else{
            echo $i."<br>";
        }
    }
}
$var="rule1";
demo(100,$var);
echo "</br>";
echo "<hr>";
$var="rule2";
demo(200,$var);
echo "</br>";
```
1. 注意在调用对象里面的方法时我们需要传入一个匿名对象,后面指定函数名

2. 而在调用类中的静态方法时只需指定类名即可。
> 以上两种情况都不能完全使用变量函数只能用系统的回调call_user_func_array(),只是变量函数来传参而不调用

```
class A{
    function one(){
    }
    static function two(){
    }
}
demo(200,array(new A,"one"));
demo(200,array("A","two"));
```
