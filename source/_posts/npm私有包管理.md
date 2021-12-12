---
title: npm私有包管理
date: 2020-05-25 21:34:01
tags:
    - verdaccio私有npm服务器
    - nrm
    - lerna
    - npx
    - nvm
---
## verdaccio私有npm服务器
### 介绍
1. verdaccio（读：韦尔搭桥）是一个 Node.js创建的轻量的私有 npm proxy registry

### 背景
1. 希望有个统一管理公司公共包平台，且有相应的权限访问和配置
2. 希望下载公共包走公共仓库， 私有包走内部私有仓库
3. 与npm、yarn兼容


### 安装
1. npm全局安装： npm install -g verdaccio
2. docker安装: docker pull verdaccio/verdaccio （官方的docker镜像）


### 使用
```js
//1. 添加用户
npm adduser --registry http://localhost:4873


//2. 发布本地包
进入包目录  ==> npm publish --registry http://localhost:4873

//3. 安装已发布的包
npm install <包名>  npm publish --registry http://localhost:4873

//4. 删除已发布的包
npm unpublish <包名> --force --registry http://fankyc.cn:4873
```


### 配置
1. 全局配置
```js
npm config set @youdao:registry "http://111.11.11.1:4100/"
```

2. 项目根目录配置
```js
//项目根目录中新建文件.npmrc来指定@youdao/xxxx包安装源
@youdao:registry "http://111.11.11.1:4100/
```

### 部署
1. pm2进程守护
2. nginx配置

## Nrm管理镜像源
### 背景
1. npm包有很多的镜像源，有的是私有包、有的源有的时候访问失败、有的源可能没有最新的包等等，所以有时需要切换npm的源，nrm包就是解决快速切换问题的。

### 安装
npm install -g nrm

### 使用
```js
// 列出可选择的源
nrm ls

//切换正在使用的源
nrm use npm


//添加一个源
//下面以本地verdaccio私有仓库为例
nrm add verdaccio http://127.0.0.1:4873


//删除一个源
nrm del verdaccio


//测试本地所有源的速度
nrm test

```

## lerna多包管理

### 背景

## npx
## 介绍
### npx和npm的区别
1. npm是永久存在，npx是临时安装，用完后删除
2. npx会帮你执行依赖包里面的二进制文件，侧重于执行命令； npm会帮你下载包到项目中，侧重于包的管理

### 原理
1. 执行node-modules包
```js
// 方法一：在项目脚本或 package.json 的 scripts 字段中对其调用进行定义
{
 "scripts": {
    "mocha": 'mocha --version'
  },
}
npm run mocha

// 方法二: 必须要要定位到 node_modules 中用繁琐的命令才能实现在命令行中对其进行调用
./node-modules/.bin/mocha --version
```
2. npx执行原理
```js
npx mocha --version
// 1. npx 会自动查找当前依赖包中的可执行文件 
// 2. 如果找不到，就会去 PATH 里找 
// 3. 如果依然找不到，就会帮你安装
```

### 使用场景
#### 1. 避免全局安装，执行一次性命令
```js
// npx 将 create-react-app 下载到一个临时目录，使用以后再删除
npx create-react-app my-react-app
```

#### 2. 指定 Node 执行版本
```js
// npx -p 参数，我们可以指定要安装的模块，后面继续跟着要执行的命令
npx -p node@0.12.8 node -v
```
#### 3. 执行gitHub源码
```js
// 执行仓库代码
npx github:piuccio/cowsay helloyo

// 一个标准模块定义至少应该包含一个 package.json 文件，并指定 bin 脚本入口，例如下面这样
{
    "name": "cowsay",
    "version": "0.0.0",
    "bin": "./index.js"
}
```

## nvm
#### 介绍
nvm: 多版本nodejs管理工具