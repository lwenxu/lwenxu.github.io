---
title: Lombok 常用功能
date: 2018-08-05 21:23:43
tags: Java工具
Categories:
	-Java
---



# Lombok 常用功能

## 1.导入

```
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.16.20</version>
    <scope>provided</scope>
</dependency>
```

## 2.@Getter/@Setter

这两个注解可以在类上也可以在字段上，看需要的粒度而定。

## 3.@ToString

生成一个 json 类型的字符串

## 4.@EqualsAndHashCode

生成 equals 和 hashcode

## 5.@NoArgsConstructor

生成午无参构造器

## 6.@RequiredArgsConstructor

只对标有 @NonNull 的字段生成构造器，并且初始化的时候会检查NonNull 的字段如果为空则抛出异常。

## 7.@AllArgsConstructor

所有字段的构造器

## 8.@Data

```
@Data`是一个集合体。包含`Getter`,`Setter`,`RequiredArgsConstructor`,`ToString`,`EqualsAndHashCode
```

## 9.@Value

可以帮忙生成一个不可变对象。对于所有的字段都将生成final的。同`@Data`， `@Value`是一个集合体。包含`Getter`,`AllArgsConstructor`,`ToString`,`EqualsAndHashCode`。

## 10.@Builder

```
Room room = builder().name("name").id("id").createTime(new Date()).occupation("1").occupation("2").build();
```

## 11.@Log

一个日志属性，可以直接使用的。

## 12.@UtilityClass

工具类

```
@UtilityClass
public class Utility {

    public String getName() {
        return "name";
    }
}

public static void main(String[] args) {
    // Utility utility = new Utility(); 构造函数为私有的,
    System.out.println(Utility.getName());

}
```

## 13.@Cleanup

用于流等可以不需要关闭使用流对象.

```
@Cleanup
OutputStream outStream = new FileOutputStream(new File("text.txt"));
@Cleanup
InputStream inStream = new FileInputStream(new File("text2.txt"));
byte[] b = new byte[65536];
while (true) {
   int r = inStream.read(b);
   if (r == -1) break;
   outStream.write(b, 0, r); 
}
```