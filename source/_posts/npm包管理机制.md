---
title: npm包管理机制
date: 2020-05-11 14:28:30
tags:
    - npm
    - package.json
---

## package.json介绍
### 目录、文件
1. 程序入口
```js
//package.json文件 
{
    "main": "./index.js"
}
```

2. 规范项目目录，一个 node.js 模块是基于 CommonJS 模块化规范实现的，严格按照 CommonJS 规范，模块目录下除了必须包含包描述文件 package.json 以外，还需要包含以下目录：
 * bin：存放可执行二进制文件的目录
 * lib：存放js代码的目录
 * doc：存放文档的目录
 * test：存放单元测试用例代码的目录
 * ...

## npm版本管理

## npm原理

### npm install 运行流程
1. npm install --> 检查配置 --> .lock文件和package是否冲突  --> 检查是否有缓存 --> 

### node_modules结构
#### 嵌套结构
1. 早期的npm处理依赖的方式就递归，形成了嵌套的结构。
2. 优点： node_modules的结构和package.json一一对应。
3. 缺点： 依赖模块非常多node_modules将非常庞大； 如果不同层级依赖相同的模块导致大量的冗余


#### 扁平化结构
1. npm3.x改用了扁平化结构；
2. 安装方法： 安装模块时，不管其是直接依赖还是子依赖的依赖，优先将其安装在 node_modules 根目录；当安装到相同模块时，判断已安装的模块版本是否符合新模块的版本范围，如果符合则跳过，不符合则在当前模块的 node_modules 下安装该模块。
3. 优点： 减少了部分重复模块冗余问题； 消除了嵌套层级过深的问题。
4. 缺点： npm3.x版本并没有根本解决模块冗余问题，

### lock文件
1. 为了解决 npm install 的不确定性问题，在 npm 5.x 版本新增了 package-lock.json 文件，而安装方式还沿用了 npm 3.x 的扁平化的方式。

### 缓存
1. 在执行 npm install 或 npm update命令下载依赖后，除了将依赖包安装在node_modules 目录下外，还会在本地的缓存目录缓存一份。

### 文件的完整性
1. 下载npm包之后本地计算文件hash值， 和下载之前拿到的文件hash对比（npm info）