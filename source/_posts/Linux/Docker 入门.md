---
title: Docker 入门
date: 2018-04-29 12:32:34
tags: Docker
categories:
  - Docker
---

## 1. Docker 简介

直接运行于操作系统内核上的虚拟化解决方案，他是一个操作系统级别的虚拟化也就是说容器只能运行在相同或者相似的内和操作系统之上的。所以我们只能在 docker 中运行 Linux 系统而不能运行 Windows 系统。他是依赖于 Linux 的内核特性：Namespace 和 Cgroup （Control Group）。

<!--more-->

### 1. VM vs Docker

![image](https://user-images.githubusercontent.com/22151420/39402559-e8dd93d6-4b94-11e8-90ca-1377aee1a335.png)

可以看到，在虚拟机上我们需要包含到细腻华技术和操作系统，但是我们在 Docker 中只需要依赖底层的操作系统，和一些必要的库，所以在空间上有非常大的优势，另外就是速度上，由于启动的服务更少，无需启动 OS 所以开销更小。

### 2. 镜像

![image](https://user-images.githubusercontent.com/22151420/39402643-9a0cb338-4b97-11e8-9f8b-77c16fcf1b0d.png)

docker 的镜像组成如上图，最底层就启动层，然后就是 root 文件系统，接着是我们自己叠加的各种应用。注意这里的每一层都是只读的，但是在我们的 linux 中，root 一开始是只读的，加载完毕后是可写的，但是这里不同，主要就是因为可以进行镜像的叠加。然后使用联合加载技术，同时把所有的镜像层加载进去。

### 3. 容器

docker 的镜像相当于一个 class 类，然后容器则是 new 出来的对象。容器是基于镜像运行出来的，但是我们有时候还是需要对镜像或者容器进行修改，这里采用的方式就是在镜像的最上面一层添加上一层虚拟的层，写得操作都是在这一层，当我们要生成新的镜像的时候就是把这一层设置为只读。这也是 docker 的 CopyOnWrite 技术。 

### 4. Namespace 和 Cgroups

Linux 内核实现的 Namespace 就是用于文件系统，进程，网络，IPC，MNT，UTS 的隔离。而 Cgroups 是用于对上面的 namespace 的资源限制，优先级设定。

## 2. 容器基本操作

#### 1.启动容器

```bash
docker run image args cmd
```

#### 2.创建交互容器

```bash
docker run -i -t ubuntu /bin/bash 
```

#### 3. 查看所有容器/最近创建的容器

```bash
docker ps -a/-l 
```

#### 4. 检查 docker 容器，返回详细信息

```bash
docker inspect ubuntu 
```

#### 5. 设置容器名

```bash
docker run --name ubu ubuntu 
```

#### 6. 以交互方式启动已经停止的容器

```bash
docker start -i ubu 
```

#### 7. 删除停止的容器

```Bash
docker rm ubu 
```

#### 8. 守护运行

在运行docker 容器以后我们要退出的时候不要用 `ctrl+c` 或者 `exit` ，使用 `ctrl+p/q` 既可以实现以 Daemon 方式运行容器。而此时如果我们需要再次进入容器我们需要使用 `docker attach name` 来进入容器。

另外我们还可以使用 `-d` 参数在运行容器的时候让他进入 Daemon 状态。

#### 9. 查看输出

当我们在后台状态运行容器的时候有时候我们需要查看对应的输出，就可以使用 

```bash
docker logs -f -t --tail nun name
```

- -f 是实时更新输出（follow）
- -t 则是带有时间的日志（timestamp）
- —tail 指定结尾行数

#### 10. 查看运行容器中运行的进程

```bash
docker top name
```

#### 11. 在运行的容器中启动新的进程/运行新的命令

```bash
docker exec -i/-t  name #类似 run
```

#### 12. 停止容器

```bash
docker stop name #信号
docker kill name #杀死
```

## 3. 镜像操作

#### 1. 查看镜像

```bash
docker [-f --no-tunck -a -q] images 
```

- -f 使用过滤器，过滤部分镜像
- —no-tunck 不进行 id 的截断
- -a 显示所有的镜像
- -q 只显示镜像 id

#### 2. tag

表示一个镜像的不同版本。

#### 3. 删除镜像

```bash
docker rmi name:tag
```

删除镜像，当我们删除的是很多的镜像的一个 tag 则是 untag 操作。

#### 4. 查找镜像

1. dockerhub 网站

2. 使用 search 命令

   ```bash
   docker search [-s num] name
   ```

   - -s 过滤最低星级

#### 5. 获取镜像

```bash
docker [-a] pull name
```

-a 这个参数就是下载所有 tag 的镜像。

#### 6. push 镜像

```bash
docker push rep/name
```

在上传的时候我们只上传了修改的部分，而不是全部。

#### 7. 构建 docker 镜像

##### 1. docker commit

```bash
docker commit [-a -m ] name rep/name 
```

把镜像提交成一个新的镜像。

- -a 作者信息
- -m 提交信息

##### 2. docker file

创建 dockerfile 然后使用 `docker bulid -t name filepath` 构建镜像。

#### 8. Dockerfile

1. FROM 第一条非注释命令，表示采用那个镜像作为基础镜像
2. MAINTAINER 维护者信息
3. RUN cmd  每一个 run 都会创建一个新的层
4. EXPOSE port 开放端口
5. CMD 运行时运行，会被 RUN 覆盖
6. ENTERPOINT  同上，但是不会被 RUN 覆盖
7. ADD/COPY src des 复制文件，ADD 有类似解压功能，COPY 没有

## 4. Docker 容器连接

### 1. docker 之间的连接

我们可以使用 

```bash
docker run --link=name:alias 
```

接下来访问响应的主机我们只需要使用 alias 即可。因为实际上是在这台机子上做了很多的环境变量还有 host 的修改导致的。

### 2. 拒绝连接

```bash
--icc =false 
--link
--iptables=true
```

如果配置了 `—icc=false` 那么就是拒绝所有的连接。而当我们配置了 iptables 则只允许 link 指定的容器。

## 5. 数据卷

![image](https://user-images.githubusercontent.com/22151420/39403129-1024c864-4ba5-11e8-8d1a-6fb0b28b2d94.png)

数据卷说白了就是数据映射，将本机的数据映射到 docker 容器中。