---
title: Linux常用技巧
date: 2017-05-04 21:08:52
tags:
categories:
  -Linux
---
### 使用ls显示一个文件的绝路径：
```bash
ls file |sed "s:^:`pwd`/:"
```

### 创建一个程序的启动器：
使用Desktop Entery的格式，首先需要进入相应目录建立文件，在文件中写入程序相关信息：
```
cd /usr/share/applications/
sudo vim application.desktop
```
开始写入文件信息,格式如下：
```bash
[Desktop Entery]
Version=1.0
Name=AppName
Exec=/application/path
Icon=/application/icon.png
Terminal=false
Type=Application
```

### vim中选定内容：
命令模式下使用vi命令变为可视之后即可选中，使用d就可以删除

### 快速处理某个被包含的内容：
以下命令可以对标点内的内容进行操作。

> ci'、ci"、ci(、ci[、ci{、ci< - 分别更改这些配对标点符号中的文本内容
di'、di"、di(或dib、di[、di{或diB、di< - 分别删除这些配对标点符号中的文本内容
yi'、yi"、yi(、yi[、yi{、yi< - 分别复制这些配对标点符号中的文本内容
vi'、vi"、vi(、vi[、vi{、vi< - 分别选中这些配对标点符号中的文本内容

另外如果把上面的i改成a可以连配对标点一起操作。
举个例子：
比如要操作的文本如下：
111"222"333
将光标移到"222"的任何一个字符处输入命令 di" ,文本会变成： 111""333
若输入命令 da" ,文本会变成： 111333
