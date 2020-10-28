---
title: Composer 使用指南
date: 2019-01-05 20:08:33
tags: PHP,包管理器
categories: 
	- PHP
---

## 1. 安装依赖

````
composer install 
````

这个语句会检查当前目录下的 **composer.json** 然后检查是否有 **composer.lock** 如果有这个文件那么就按照 lock 文件中的版本号进行安装依赖，如果没有lock文件就按照 composer.json 中定义的版本进行安装。

这里就需要注意，composer.json 定义的版本号可能是一个区间也就是他可以是一个范围，随着类库的更新而更新，但是 lock文件保证了此项目的版本号是确定的，也就是项目组成员得到的版本号是一致的，就不会有很多麻烦问题了。

composer.json 中的版本的定义方式有：

| 名称         | 实例                                   | 描述                                                         |
| ------------ | -------------------------------------- | ------------------------------------------------------------ |
| 确切的版本号 | `1.0.2`                                | 你可以指定包的确切版本。                                     |
| 范围         | `>=1.0` `>=1.0,<2.0``>=1.0,<1.1|>=1.2` | 通过使用比较操作符可以指定有效的版本范围。  有效的运算符：`>`、`>=`、`<`、`<=`、`!=`。  你可以定义多个范围，用逗号隔开，这将被视为一个**逻辑AND**处理。一个管道符号`|`将作为**逻辑OR**处理。  AND 的优先级高于 OR。 |
| 通配符       | `1.0.*`                                | 你可以使用通配符`*`来指定一种模式。`1.0.*`与`>=1.0,<1.1`是等效的。 |
| 赋值运算符   | `~1.2`                                 | 一个常见的用法是标记你所依赖的最低版本，像 `~1.2` （允许1.2以上的任何版本，但不包括2.0）。`~1.2` 相当于 `>=1.2,<2.0`，而 `~1.2.3` 相当于 `>=1.2.3,<1.3`。`~1.2` 只意味着 `.2` 部分可以改变，但是 `1.` 部分是固定的。 |

对的，这样就解决了项目成员的类库版本一致的问题，但是如果我们希望能够升级类库呢？那么就执行

```
composer update
```

如果只想安装或更新一个依赖，你可以白名单它们：

```sh
php composer.phar update monolog/monolog [...]
```

## 2. Packagist

[packagist](https://packagist.org/) 是 Composer 的主要资源库。 一个 Composer 的库基本上是一个包的源：记录了可以得到包的地方。但是没必要去国外的网站下载，我们可以采用国内的镜像加速：

### **方法一：** 修改 composer 的全局配置文件**（推荐方式）**

打开命令行窗口（windows用户）或控制台（Linux、Mac 用户）并执行如下命令：

```bash
composer config -g repo.packagist composer https://packagist.phpcomposer.com
```

### **方法二：** 修改当前项目的 `composer.json` 配置文件：

打开命令行窗口（windows用户）或控制台（Linux、Mac 用户），进入你的项目的根目录（也就是 `composer.json` 文件所在目录），执行如下命令：

```bash
composer config repo.packagist composer https://packagist.phpcomposer.com
```

上述命令将会在当前项目中的 `composer.json` 文件的末尾自动添加镜像的配置信息（你也可以自己手工添加）：

```json
"repositories": {
    "packagist": {
        "type": "composer",
        "url": "https://packagist.phpcomposer.com"
    }
}
```

## 3. 类库自动加载

除了库的下载，Composer 还准备了一个自动加载文件，它可以加载 Composer 下载的库中所有的类文件。使用它，你只需要将下面这行代码添加到你项目的引导文件中：

```php
require 'vendor/autoload.php';
```

## 4. 创建项目 create-project

你可以使用 Composer 从现有的包中创建一个新的项目。这相当于执行了一个 `git clone` 或 `svn checkout` 命令后将这个包的依赖安装到它自己的 vendor 目录。

此命令有几个常见的用途：

1. 你可以快速的部署你的应用。
2. 你可以检出任何资源包，并开发它的补丁。
3. 多人开发项目，可以用它来加快应用的初始化。

要创建基于 Composer 的新项目，你可以使用 "create-project" 命令。传递一个包名，它会为你创建项目的目录。你也可以在第三个参数中指定版本号，否则将获取最新的版本。

如果该目录目前不存在，则会在安装过程中自动创建。

```sh
php composer.phar create-project doctrine/orm path 2.2.*
```

此外，你也可以无需使用这个命令，而是通过现有的 `composer.json` 文件来启动这个项目。

默认情况下，这个命令会在 packagist.org 上查找你指定的包。