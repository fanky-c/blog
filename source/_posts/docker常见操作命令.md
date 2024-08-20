---
title: docker常见操作命令
date: 2024-08-19 19:41:34
tags:
 - docker
 - 镜像
 - 容器
---

## 服务
### 1、登录
```js
echo "password" | docker login "xxxx.com:80" --username "username" --password-stdin
```

## 镜像

镜像（Image）：Docker 镜像是一个特殊的文件系统，除了提供容器运行时所需的程序、库、资源、配置等文件外，还包含了一些为运行时准备的一些配置参数（如匿名卷、环境变量、用户等）。镜像不包含任何动态数据，其内容在构建之后也不会被改变。

### 1、拉取镜像

```js
docker pull [选项] [Docker Registry 地址[:端口号]/]仓库名[:标签]

docker pull xxxx.com:80/library/fed_node-v18.19.0@sha256:d38b6c5ab2e9ddb335006dd32f69f3e9a2b74b53be6b0936d2ab6582db09fd12
```

### 2、列出镜像
```js
docker images
// 或者
docker image ls
```

### 3、构建镜像
```js
// 一键清理 Build Cache 缓存命令
docker builder prune

docker build -t fed_node-v18.19.0:latest ./
```

### 4、镜像运行
```js
docker run [镜像ID]
```

### 5、镜像打tag和入库
```js
docker tag myimage:1.0 myusername/myimage:1.0
docker push myusername/myimage:1.0
```

### 6、删除镜像
```js
docker rmi <镜像Id>
```

### 7、导出镜像
```js
docker save 7baf28ea91eb > nginx.tar
```

### 8、检索远程仓库镜像
```js
docker search 'node'
```

### 9、Dockerfile常见的指令

```js
 FROM：指定基础镜像

 RUN：执行命令

 COPY：复制文件

 ADD：更高级的复制文件

 CMD：容器启动命令

 ENV：设置环境变量

 EXPOSE：暴露端口
```

## 容器

容器（Container）：镜像（Image）和容器（Container）的关系，就像是面向对象程序设计中的 类 和 实例 一样，镜像是静态的定义，容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等。

### 1、查看容器
```js
// 列出本机运行的容器
docker ps

// 列出本机所有的容器（包括停止和运行）
docker ps -a
```

### 2、启动容器
```js
// 新建并启动
docker run [镜像名/镜像ID]

// 启动已终止容器
docker start [容器ID]
```

### 3、启动容器(bash交互， 退出自动删除)
```js
docker run -it --rm 27f7fe bash
```

### 4、停止容器
```js
// 停止运行的容器
docker stop [容器ID]

//杀死容器进程
docker  kill [容器ID]
```

### 5、重启容器
```js
docker restart [容器ID]
```

### 6、删除容器
```js
docker rm [容器ID]
```

### 7、进入容器
```js
// 如果从这个 stdin 中 exit，会导致容器的停止
docker attach [容器ID]

// 交互式进入容器
docker exec [容器ID]


// 进入容器通常使用第二种方式，docker exec后面跟的常见参数如下：
// －d, --detach 在容器中后台执行命令；
// －i, --interactive=true I false ：打开标准输入接受用户输入命令
```

### 8、查看容器日志
```js
docker logs [容器ID]
```


## 仓库

仓库（Repository）：仓库（Repository）类似Git的远程仓库，集中存放镜像文件。