---
title: JavaIO
date: 2017-08-09 15:49:28
tags: IO
categories:
		-Java
---

1.在IO有两种数据传输格式一个是字符流还一个是字节流
但是字符流就会涉及到编码的问题
1. 一开始美国使用的自己的编码表就是ASCII表
2. 中国的字符需要被识别也需要编码表于是就有了GB2312
3. 但是由于中国的子很多，还有少数名族等等后来又进行了优化扩容就出现了GBK
4. 最后国际标准组织制定了一个包含所有国家所有地区的码表就是Unicode
5. 之后对Unicode进行了优化也就是以前是所有的字符都是两个字节表示，但是现在就可以一个字节或者三个字节，具体看情况的UTF_8
  字符流一般都包含了编码表，也就是在传输的时候你可以指定编码方式然后IO流帮你转换
<!--more-->
2.在IO流里面就是有四个抽象的顶层接口
  * 字节流的：
      InputStream  OutputStream
  * 字符流：
      Reader  Writer  

3.装饰设计模式：
    也就是基于原理已经有的对象，在此对象的基础上提供更强大的功能，也就是能达到不修源码的情况下扩展原来的类的功能
    用于增强的的类称为装饰类，这个类一般都是用构造函数来接受需要被装饰的类的对象就例如BufferedRead()内部就调用了
    FileReader的read方法
    一说到功能扩展自然就会想到继承，这里就要说一下有时候为什么还需要装饰模式。
    假如说有以下类层次结构：
    ---ReadText
    ---ReadMedia
    ---ReadData
    那么我们要用缓冲区读的话我们就会有以下的类层次结构：
        ---ReadText
            |--BufferedReadText
        ---ReadMedia
            |--BufferedReadMedia
        ---ReadData
            |--BufferedReadData
    类层级就非常臃肿
    所以我们想到就用装饰者模式：
    假如有以下装饰类：
 ```java
    class BufferedReader{
        BufferedReader(ReadText readText){

        }
        BufferedReader(ReadMedia readMedia){

        }
        BufferedReader(ReadData readData){

        }
    }
 ```
  这样虽然可以，但是我们要拓展Read的子类这个装饰类必须要修改，拓展性很差
    所以我们直接就用多态性质
   ```java
      class BufferedReader extends Read{
        private Read read;
          BufferedReader(Read read){

          }

        public int read(char cbuf[], int off, int len){
              return read.read( cbuf[],  off,  len);//这里我们必须要复写Read里面的抽象方法，但是我们不知道这个read怎么实现，所以我们直接调用传进来的子类的对象的read方法

        }

        public boolean close(){
            return read.close();  //同上
        }
      }
   ```


  这样一来类层级结构就是：
  ---Read
    |---ReadText
    |---ReadMedia
    |---ReadData
    |---BufferedRead

  注意装饰类是Read的子类，并且他和被装饰类是同一个层次，因为他是更强的被装饰类，所以他们是同一个层级


  4.字节流的处理
    InputStream
        |--FileInputStream
    OutputStream
        |--FileOutputStream

   这两个类的处理方式和上面的字符流完全一样，唯一一点的不同就是，它里面读取的时候还有一个avaliable方法可以确定文件中字节大小，还有就是他是以字节为单位的
​    
  ```java
public class FileInputStreamDemo {
    public static void main(String[] args) throws IOException{
        FileInputStream fileInputStream=new FileInputStream("a.txt");
        read_3(fileInputStream);
    }

    public static void read_1(FileInputStream fileInputStream)throws IOException{
        byte by;
        while ((by= (byte) fileInputStream.read())!=-1){
            System.out.print((char) by);
        }
    }

    public static void read_2(FileInputStream fileInputStream)throws IOException{
        byte[] buffer=new byte[1024];
        char[] buf=new char[1024];
        int num=0;
        StringBuilder builder=new StringBuilder();
        while ((num=fileInputStream.read(buffer))!=-1){
            // TODO: 2017/7/16 转数组为字符串的问题   尤其是基本类型转字符串的问题   是不是还有什么好的对象去做这件事
            for (byte b:buffer){
                builder.append((char) b);
            }
        }
        System.out.println(builder);
    }

    public static void read_3(FileInputStream fileInputStream) throws IOException{
        byte[] by=new byte[fileInputStream.available()];   //这个函数就可以确定字节个数   不过只能用于较小的文件  大文件开辟太大的缓冲区会出问题
        fileInputStream.read(by);
        StringBuilder stringBuilder=new StringBuilder();
        for (byte bys:by){
            // TODO: 2017/7/16 转数组为字符串的问题   尤其是基本类型转字符串的问题   是不是还有什么好的对象去做这件事
            stringBuilder.append((char)bys);
        }
        System.out.println(stringBuilder);

    }
}
  ```

5.当然也是有BufferedInputStream和BufferedOutputStream这两个装饰类   注意这个和前面的一模一样

6.流对象的使用，流对象的使用它们的类别太多，这里为了区分使用哪一个流对象我们就具体分析一下：
    1.明确源和目的：
        源的话我们就是用InputStream和Reader
        目的则是OutputStream和Writer
    2.之后就是确定是否为纯文本：
        是：字符流
        否：字节流
    3.确定具体的对象：
        通过设备来区分：硬盘，控制台，内存等等   分别使用合适的子类
        然后根据是否需要更强的功能，更快的速度使用Buffered包装类
