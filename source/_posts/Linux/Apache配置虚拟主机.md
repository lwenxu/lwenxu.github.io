---
title: Apache配置详解
date: 2019-01-03 11:08:33
tags: Linux
categories: 
	- 服务器
	- Linux
---

# 1. 虚拟主机概念

我们要想实现一个web站点，而且能够在互联网上被访问，首先它再能运行在操作系统，而且这个操作系统还要运行在物理主机上（第一它是一个主机）。在互联网上能够被访问，那我们需要一个主机，需要一个IP地址，需要一个时时在线的服务器，这需要多少资源？对众多小型站点来讲或者说对某种需求来讲，有可能都用不到服务器，也就是每天就10个人左右访问，只是需要我们在线而已，如果我们就为这一点点的需求就投入重大的资源的话是非常浪费的。我们就期望能够像我们使用虚拟机一样，虚拟的OS一样或虚拟的PC一样，能够在一台物理主机上虚拟出来多个可以同时运行的站点或者我们把它称为主机因此就把它称为虚拟主机。

# 2. Apache 主机的类型

### 1. 中心主机

### 2. 虚拟主机

1. 基于IP: 端口相同，IP地址不同。

2. 基于端口：IP相同，端口不同。

3. 基于域名：IP地址相同，端口相同主机名不同



**注意：所有的虚拟主机的配置我们都需要取消中心主机，也就是注释掉 DocumentRoot 这是配置虚拟主机的前提**

# 3. 基于域名的虚拟主机

例如我采用的 xampp 所以配置虚拟主机就在 `C:\xampp\apache\conf\extra\httpd-vhosts.conf` 中配置，由于这个文件默认被 include 到主配置文件中了，所以在这里的修改都可以生效。

首先需要保证主配置文件中的中心主机被取消了也就是：

```bash
#DocumentRoot "C:/xampp/htdocs"
```

然后打开 `httpd-vhosts.conf` 配置文件，按照下面的格式配置虚拟主机

```
<VirtualHost *:80>
	DocumentRoot "C:/xampp/htdocs"
	ServerName localhost
	<Directory "C:/xampp/htdocs">
		Options Indexes FollowSymLinks Includes ExecCGI
		AllowOverride All
		Require all granted
	</Directory>
</VirtualHost>
```

## 1.  `<VirtualHost *:80>`

 apache监听本机的所有 IP 和 80 端口做多域名虚拟主机

## 2. DocumentRoot

表示服务器的根目录

## 3. ServerName

就是表示域名，我们采用域名方式配置虚拟主机，所以每个虚拟主机的域名应该是不一样的才行

## 4. Directory

对根目录的规则应用，其中涉及到对于目录的访问权限和其他配置问题

## 5. 对于 Directory 指令解析

### 1.  Options

配置在特定目录使用哪些特性，常用的值和基本含义如下

1. ExecCGI: 在该目录下允许执行CGI脚本。
2. FollowSymLinks: 在该目录下允许文件系统使用符号连接。
3. Indexes: 当用户访问该目录时，如果用户找不到DirectoryIndex指定的主页文件(例如index.html),则返回该目录下的文件列表给用户。
4. SymLinksIfOwnerMatch: 当使用符号连接时，只有当符号连接的文件拥有者与实际文件的拥有者相同时才可以访问。

所以我们一般在配置 PHP 的时候所配置的内容是

```
Options Indexes FollowSymLinks Includes ExecCGI
```

### 2. AllowOverride

允许存在于.htaccess文件中的指令类型(.htaccess文件名是可以改变的，其文件名由AccessFileName指令决定)：

- None: 当AllowOverride被设置为None时。不搜索该目录下的.htaccess文件（可以减小服务器开销）。
- All: 在.htaccess文件中可以使用所有的指令。

建议关闭这个选项，因为apache在文档中已经明确支持不建议使用它了，主要是会降低服务器性能，.htaccess 文件可以做到的我们都可以在 Directory 指令中做的更好。

### 3. Order

控制在访问时Allow和Deny两个访问规则哪个优先，也就是黑白名单的匹配顺序

- Allow：允许访问的主机列表(可用域名或子网，例如：Allow from 192.168.0.0/16)。
- Deny：拒绝访问的主机列表。

> 更详细的用法可参看：[order用法](https://httpd.apache.org/docs/trunk/zh-cn/mod/mod_access_compat.html#order)

匹配原则：

```
Allow,Deny
```

First, all `Allow` directives are evaluated; at least one must match, or the request is rejected. Next, all `Deny` directives are evaluated. If any matches, the request is rejected. Last, any requests which do not match an `Allow` or a `Deny` directive are denied by default.

```
Deny,Allow
```

First, all `Deny` directives are evaluated; if any match, the request is denied **unless** it also matches an `Allow` directive. Any requests which do not match any `Allow` or `Deny` directives are permitted.

### 4. DirectoryIndex

主页文件设置 一般都是 index.html  index.php

### 5. Deny /Allow 

黑白名单书写规则如下：

```config
Allow from example.org
Allow from .net example.edu
Allow from 10.1.2.3
Allow from 192.168.1.104 192.168.1.205
Allow all
```

# 4. 基于端口的虚拟主机

对于同一个 ip 来做虚拟主机，就是需要监听不同的端口，首先需要在主配置文件中添加新的监听端口：

```
Listen 80
Listen 81
```

然后再虚拟主机里面配置

```
<Virtualhost *:80>
	DocumentRoot "/var1"
</Virtualhost>
<Virtualhost *:81>
	DocumentRoot "/var2"
</Virtualhost>
```

# 5. 基于ip的虚拟主机

这个就不过多赘述了，完全和基于端口的配置方式一样。也就是修改的是ip 而非端口、

# 6.服务器的优化 (MPM: Multi-Processing Modules)

apache2主要的优势就是对多处理器的支持更好，在编译时同过使用--with-mpm选项来决定apache2的工作模式。如果知道当前的apache2使用什么工作机制，可以通过httpd -l命令列出apache的所有模块，就可以知道其工作方式：

- prefork：如果httpd -l列出prefork.c，则需要对下面的段进行配置。

  ```
  <IfModule prefork.c>
   StartServers 5 #启动apache时启动的httpd进程个数。
   MinSpareServers 5 #服务器保持的最小空闲进程数。
   MaxSpareServers 10 #服务器保持的最大空闲进程数。
   MaxClients 150 #最大并发连接数。
   MaxRequestsPerChild 1000 #每个子进程被请求服务多少次后被kill掉。0表示不限制，推荐设置为1000。
   </IfModule>
  ```

在该工作模式下，服务器启动后起动5个httpd进程(加父进程共6个，通过ps -ax|grep httpd命令可以看到)。当有用户连接时，apache会使用一个空闲进程为该连接服务，同时父进程会fork一个子进程。直到内存中的空闲进程达到MaxSpareServers。该模式是为了兼容一些旧版本的程序。我缺省编译时的选项。

- worker：如果httpd -l列出worker.c，则需要对下面的段进行配置：

```
<IfModule worker.c> 
    StartServers 2 #启动apache时启动的httpd进程个数。 
    MaxClients 150 #最大并发连接数。 
    MinSpareThreads 25 #服务器保持的最小空闲线程数。 
    MaxSpareThreads 75 #服务器保持的最大空闲线程数。 
    ThreadsPerChild 25 #每个子进程的产生的线程数。 
    MaxRequestsPerChild 0 #每个子进程被请求服务多少次后被kill掉。0表示不限制，推荐设置为1000。 
</IfModule> 
```

该模式是由线程来监听客户的连接。当有新客户连接时，由其中的一个空闲线程接受连接。服务器在启动时启动两个进程，每个进程产生的线程数是固定的(ThreadsPerChild决定)，因此启动时有50个线程。当50个线程不够用时，服务器自动fork一个进程，再产生25个线程。

- perchild：如果httpd -l列出perchild.c，则需要对下面的段进行配置：

```
<IfModule perchild.c> 
    NumServers 5 #服务器启动时启动的子进程数 
    StartThreads 5 #每个子进程启动时启动的线程数 
    MinSpareThreads 5 #内存中的最小空闲线程数 
    MaxSpareThreads 10 #最大空闲线程数 
    MaxThreadsPerChild 2000 #每个线程最多被请求多少次后退出。0不受限制。 
    MaxRequestsPerChild 10000 #每个子进程服务多少次后被重新fork。0表示不受限制。 
</IfModule> 
```

该模式下，子进程的数量是固定的，线程数不受限制。当客户端连接到服务器时，又空闲的线程提供服务。 如果空闲线程数不够，子进程自动产生线程来为新的连接服务。该模式用于多站点服务器。

# 7.别名设置

对于不在DocumentRoot指定的目录内的页面，既可以使用符号连接，也可以使用别名。别名的设置如下：

```
Alias /download/ "/var/www/download/" #访问时可以输入:http://www.custing.com/download/ 
 
<Directory "/var/www/download"> #对该目录进行访问控制设置 
    Options Indexes MultiViews 
    AllowOverride AuthConfig 
    Order allow,deny 
    Allow from all 
</Directory>
```

#### 6、CGI设置

```
# 访问时可以：http://www.clusting.com/cgi-bin/，但是该目录下的CGI脚本文件要加可执行权限
ScriptAlias /cgi-bin/ "/mnt/software/apache2/cgi-bin/" 
 
<Directory "/usr/local/apache2/cgi-bin"> #设置目录属性 
    AllowOverride None 
    Options None 
    Order allow,deny 
    Allow from all 
</Directory> 
```

# 8、日志的设置

##### (1) 错误日志的设置

- ErrorLog logs/error_log :日志的保存位置

- LogLevel warn #日志的级别

显示的格式日下：

```
[Mon Oct 10 15:54:29 2005] [error] [client 192.168.10.22] access to /download/ failed, reason: user admin not allowed access 
```

##### (2) 访问日志设置

日志的缺省格式有如下几种：

- LogFormat "%h %l %u %t "%r" %>s %b "%{Referer}i" "%{User-Agent}i"" combined
- LogFormat "%h %l %u %t "%r" %>s %b" common #common为日志格式名称
- LogFormat "%{Referer}i -> %U" referer
- LogFormat "%{User-agent}i" agent
- CustomLog logs/access_log common

格式中的各个参数如下：

```
%h --客户端的ip地址或主机名 
%l --The 这是由客户端 identd 判断的RFC 1413身份，输出中的符号 "-" 表示此处信息无效。 
%u --由HTTP认证系统得到的访问该网页的客户名。有认证时才有效，输出中的符号 "-" 表示此处信息无效。 
%t --服务器完成对请求的处理时的时间。 
"%r" --引号中是客户发出的包含了许多有用信息的请求内容。 
%>s --这个是服务器返回给客户端的状态码。 
%b --最后这项是返回给客户端的不包括响应头的字节数。 
"%{Referer}i" --此项指明了该请求是从被哪个网页提交过来的。 
"%{User-Agent}i" --此项是客户浏览器提供的浏览器识别信息。 
```

下面是一段访问日志的实例：

```
192.168.10.22 - bearzhang [10/Oct/2005:16:53:06 +0800] "GET /download/ HTTP/1.1" 200 1228 
192.168.10.22 - - [10/Oct/2005:16:53:06 +0800] "GET /icons/blank.gif HTTP/1.1" 304 - 
192.168.10.22 - - [10/Oct/2005:16:53:06 +0800] "GET /icons/back.gif HTTP/1.1" 304 – 
```

# 9、用户认证的配置

##### (1) httpd.conf用户认证配置:

```
AccessFileName .htaccess 
......... 
Alias /download/ "/var/www/download/" 
<Directory "/var/www/download"> 
    Options Indexes 
    AllowOverride AuthConfig 
</Directory> 
```

##### (2) create a password file:

```
/usr/local/apache2/bin/htpasswd -c /var/httpuser/passwords bearzhang
```



##### (3) configure the server to request a password and tell the server which users are allowed access.

```
vi /var/www/download/.htaccess: 
 
AuthType Basic 
AuthName "Restricted Files" 
AuthUserFile /var/httpuser/passwords 
Require user bearzhang 
#Require valid-user #all valid user 
```

# 10、虚拟主机的配置总结

##### (1)基于IP地址的虚拟主机配置

```
Listen 80 
 
<VirtualHost 172.20.30.40> 
    DocumentRoot /www/example1 
    ServerName www.example1.com 
</VirtualHost> 
 
<VirtualHost 172.20.30.50> 
    DocumentRoot /www/example2 
    ServerName www.example2.org 
</VirtualHost> 
```

##### (2) 基于IP和多端口的虚拟主机配置

```
Listen 172.20.30.40:80 
Listen 172.20.30.40:8080 
Listen 172.20.30.50:80 
Listen 172.20.30.50:8080 
 
<VirtualHost 172.20.30.40:80> 
    DocumentRoot /www/example1-80 
    ServerName www.example1.com 
</VirtualHost> 
 
<VirtualHost 172.20.30.40:8080> 
    DocumentRoot /www/example1-8080
    ServerName www.example1.com 
</VirtualHost> 
 
<VirtualHost 172.20.30.50:80> 
    DocumentRoot /www/example2-80 
    ServerName www.example1.org 
</VirtualHost> 
 
<VirtualHost 172.20.30.50:8080> 
    DocumentRoot /www/example2-8080 
    ServerName www.example2.org 
</VirtualHost> 
```

##### (3) 单个IP地址的服务器上基于域名的虚拟主机配置：

```
# Ensure that Apache listens on port 80 
Listen 80 
 
# Listen for virtual host requests on all IP addresses 
NameVirtualHost *:80 
 
<VirtualHost *:80> 
    DocumentRoot /www/example1 
    ServerName www.example1.com 
    ServerAlias example1.com. *.example1.com 
    # Other directives here 
</VirtualHost> 
 
<VirtualHost *:80> 
    DocumentRoot /www/example2 
    ServerName www.example2.org 
    # Other directives here 
</VirtualHost> 
```

##### (4) 在多个IP地址的服务器上配置基于域名的虚拟主机：

```
Listen 80 
 
# This is the "main" server running on 172.20.30.40 
ServerName server.domain.com 
DocumentRoot /www/mainserver 
 
# This is the other address 
NameVirtualHost 172.20.30.50 
 
<VirtualHost 172.20.30.50> 
    DocumentRoot /www/example1 
    ServerName www.example1.com 
    # Other directives here ... 
</VirtualHost> 
 
<VirtualHost 172.20.30.50> 
    DocumentRoot /www/example2 
    ServerName www.example2.org 
    # Other directives here ... 
</VirtualHost> 
```

##### (5) 在不同的端口上运行不同的站点(基于多端口的服务器上配置基于域名的虚拟主机)：

```
Listen 80 
Listen 8080 
 
NameVirtualHost 172.20.30.40:80 
NameVirtualHost 172.20.30.40:8080 

<VirtualHost 172.20.30.40:80> 
    ServerName www.example1.com 
    DocumentRoot /www/domain-80 
</VirtualHost> 
 
<VirtualHost 172.20.30.40:8080> 
    ServerName www.example1.com
    DocumentRoot /www/domain-8080 
</VirtualHost> 
 
<VirtualHost 172.20.30.40:80>
    ServerName www.example2.org 
    DocumentRoot /www/otherdomain-80 
</VirtualHost> 
 
<VirtualHost 172.20.30.40:8080> 
    ServerName www.example2.org 
    DocumentRoot /www/otherdomain-8080 
</VirtualHost> 
```

##### (6) 基于域名和基于IP的混合虚拟主机的配置:

```
Listen 80 
 
NameVirtualHost 172.20.30.40 
 
<VirtualHost 172.20.30.40> 
    DocumentRoot /www/example1 
    ServerName www.example1.com 
</VirtualHost> 
 
<VirtualHost 172.20.30.40> 
    DocumentRoot /www/example2 
    ServerName www.example2.org 
</VirtualHost> 
 
<VirtualHost 172.20.30.40> 
    DocumentRoot /www/example3 
    ServerName www.example3.net 
</VirtualHost> 
```

# 11.SSL加密的配置

首先在配置之前先来了解一些基本概念：

> a. 证书的概念：首先要有一个根证书，然后用根证书来签发服务器证书和客户证书，一般理解：服务器证书和客户证书是平级关系。SSL必须安装 服务器证书来认证。 因此：在此环境中，至少必须有三个证书：根证书，服务器证书，客户端证书。 在生成证书之前，一般会有一个私钥，同时用私钥生成证书请求，再利用证书服务器的根证来签发证书。 SSL所使用的证书可以自己生成，也可以通过一个商业性CA（如Verisign 或 Thawte）签署证书。

> b. 签发证书的问题：如果使用的是商业证书，具体的签署方法请查看相关销售商的说明；如果是知己签发的证书，可以使用openssl自带的CA.sh  脚本工具。

如果不为单独的客户端签发证书，客户端证书可以不用生成，客户端与服务器端使用相同的证书。

#### (1) conf/ssl.conf 配置文件中的主要参数配置如下：

```
Listen 443 
 
SSLPassPhraseDialog buildin 
#SSLPassPhraseDialog exec:/path/to/program 
SSLSessionCache dbm:/usr/local/apache2/logs/ssl_scache 
SSLSessionCacheTimeout 300 
SSLMutex file:/usr/local/apache2/logs/ssl_mutex 
 
<VirtualHost _default_:443> 
# General setup for the virtual host 
DocumentRoot "/usr/local/apache2/htdocs" 
ServerName www.example.com:443 
ServerAdmin you@example.com 
ErrorLog /usr/local/apache2/logs/error_log 
TransferLog /usr/local/apache2/logs/access_log 
  
SSLEngine on 
SSLCipherSuite ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP:+eNULL 
  
SSLCertificateFile /usr/local/apache2/conf/ssl.crt/server.crt 
SSLCertificateKeyFile /usr/local/apache2/conf/ssl.key/server.key 
CustomLog /usr/local/apache2/logs/ssl_request_log "%t %h %{SSL_PROTOCOL}x %{SSL_CIPHER}x "%r" %b" 
</VirtualHost> 
```

#### (2) 创建和使用自签署的证书：

a. Create a RSA private key for your Apache server

```
/usr/local/openssl/bin/openssl genrsa -des3 -out /usr/local/apache2/conf/ssl.key/server.key 1024 
```

b. Create a Certificate Signing Request (CSR)

```
/usr/local/openssl/bin/openssl req -new -key /usr/local/apache2/conf/ssl.key/server.key -out /usr/local/apache2/conf/ssl.key/server.csr 
```

c. Create a self-signed CA Certificate (X509 structure) with the RSA key of the CA

```
/usr/local/openssl/bin/openssl req -x509 -days 365 -key /usr/local/apache2/conf/ssl.key/server.key -in /usr/local/apache2/conf/ssl.key/server.csr -out /usr/local/apache2/conf/ssl.crt/server.crt 
/usr/local/openssl/bin/openssl genrsa 1024 -out server.key 
/usr/local/openssl/bin/openssl req -new -key server.key -out server.csr 
/usr/local/openssl/bin/openssl req -x509 -days 365 -key server.key -in server.csr -out server.crt 
```

#### (3) 创建自己的CA（认证证书），并使用该CA来签署服务器的证书。

```
mkdir /CA 
cd /CA 
cp openssl-0.9.7g/apps/CA.sh /CA 
./CA.sh -newca 
openssl genrsa -des3 -out server.key 1024 
openssl req -new -key server.key -out server.csr 
cp server.csr newreq.pem 
./CA.sh -sign 
cp newcert.pem /usr/local/apache2/conf/ssl.crt/server.crt 
cp server.key /usr/local/apache2/conf/ssl.key/
```

**参考文章**：

> [深入理解Apache虚拟主机](http://blog.51cto.com/yahoon/36239)
>
> [Apache配置文件httpd.conf详解](https://www.jianshu.com/p/c36dd3946e74)
>
> [Apache文档](https://httpd.apache.org/docs/trunk/zh-cn/howto/htaccess.html)