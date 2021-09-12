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
// package.json文件 
// 例如： import { notification } from 'antd'; 实际上引入的就是./index.js中暴露出去的模块
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
###  SemVer规范
#### 标准版本
SemVer规范的标准版本号采用 X.Y.Z 的格式，其中 X、Y 和 Z 为非负的整数，且禁止在数字前方补零。X 是主版本号、Y 是次版本号、而 Z 为修订号
1. 主版本号(major)：当你做了不兼容的API 修改
2. 次版本号(minor)：当你做了向下兼容的功能性新增
3. 修订号(patch)：当你做了向下兼容的问题修正。

#### 先行版本
当某个版本改动比较大、并非稳定而且可能无法满足预期的兼容性需求时，你可能要先发布一个先行版本。
1. 内部版本(alpha):
2. 公测版本(beta):
3. 正式版本的候选版本rc: 即 Release candiate
```js
  npm view vue versions  //查看vue发布的所以版本记录
```


### 版本工具使用
```js
//下载
npm i semver

//判断版本号是否合法
semver.valid('1.2.3') // '1.2.3'

//强制转化成semver版本号
semver.valid(semver.coerce('v2')) // '2.0.0'
semver.valid(semver.coerce('42.6.7.9.3-alpha')) // '42.6.7'
```

### 锁定版本依赖
#### lock文件
#### 定期更新依赖
使用 npm outdated 可以帮助我们列出有哪些还没有升级到最新版本的依赖：
1. 黄色表示不符合我们指定的语意化版本范围 - 不需要升级
2. 红色表示符合指定的语意化版本范围 - 需要升级

执行 npm update 会升级所有的红色依赖。

## npm原理

### npm install 运行流程
![image](/blog/img/npm.png)
1. 检查.npmrc文件，优先级： 项目基本.npmrc文件 > 用户的 > 全局的 > npm内置
2. 检查项目中有无lock文件
3. 无lock文件
   1. 不存在缓存就去网上下载, 将下载的包复制到npm缓存目录, 依赖结构解压到 node_modules
   2. 存在缓存，将缓存依赖结构解压到node_modules
   3. 检验包的完整性
   4. 如果检验通过
   5. 构建依赖树，不管其是直接依赖还是子依赖的依赖，优先将其放置在 node_modules 根目录；当遇到相同模块时，判断已放置在依赖树的模块版本是否符合新模块的版本范围，如果符合则跳过，不符合则在当前模块的 node_modules 下放置该模块
   6. 生成lock文件
4. 有lock文件
   1. 检查lock和package中依赖是否有冲突， 如有冲突以package为准
   2. 如果没有冲突，直接跳过获取包信息、构建依赖树过程， 开始在缓存中查找包信息，后续过程和上面的一样

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


[本文参考](https://mp.weixin.qq.com/s/gtsnxFOYRZ4i-Id_9Gr6Aw)