---
title: npm|yarn|pnpm区别
date: 2022-02-07 16:54:02
tags:
    - npm、yarn和pnpm区别
    - yarn
    - pnpm
---

## npm
### 1. 原理
<img src="/img/npm4.png" />
1. 检查.npmrc文件, 优先级为：项目级的.npmrc 文件 > 用户级的.npmrc 文件> 全局级的.npmrc文件 > npm内置的.npmrc 文件
2. 检查有无lock文件
3. 无lock文件
   1. 从npm远程获取包信息
   2. 根据package.json构建依赖树
   3. 在缓存中依此查找依赖树中每个包
      1. 不存在缓存：
         1. 从npm包远程仓库下载包
         2. 检查包的完整性
         3. 检查不通过就重新下载
         4. 检查通过
            1. 将下载的包复制到npm缓存目录
            2. 将下载的包依照依赖树结构解压到node_modules
      2. 存在缓存：将缓存按照依赖结构解压到 node_modules
   4. 将包解压到 node_modules
   5. 生成lock文件
4. 有lock文件
   1. 检查package.json中依赖版本和是否和package-lock.json中的依赖有冲突
   2. 如果没有冲突，直接跳过获取包的、构建依赖树、查找缓存后续过程相同

**1. 项目代码中引用了一个模块，模块查找流程如下**
1. 在当前模块路径下搜索
2. 在当前模块 node_modules 路径下搜素
3. 在上级模块的 node_modules 路径下搜索
4. 直到搜索到全局路径中的 node_modules

**2. npm 3.x 新增扁平结构**
1. **安装模块时，不管其是直接依赖还是子依赖的依赖，优先将其安装在node_modules根目录；当安装到相同模块时，判断已安装的模块版本是否符合新模块的版本范围，如果符合则跳过，如果不符合则在当前模块的node_modules下安装该模块。** 
2. **解决了**
  1. 不同层级的依赖中， 可能引用同一个模块， 导致大量冗余
  2. 在windows系统中，文件路径最大长度为260个字符，嵌套过深可能导致不可预祝的问题   
   

### 2. 缺点
1. **没有彻底解决扁平结构**
   1. 如果依赖的不同版本的包依然会存在嵌套问题
   <img width="80%" src="/img/npm5.jpeg" />


## yarn
### 1. 原理
<img src="/img/yarn.webp" />
1. **A、检查（checking）:**主要是检查项目中是否存在一些 npm 相关的配置文件，如 package-lock.json 等。如果存在，可能会警告提示，因为它们可能会存在冲突。在这一阶段，也会检查系统 OS、CPU 等信息。
2. **B、解析包（resolving packages）:** 在经过复杂的解析算法后，我们就确定了所有依赖的具体版本信息以及下载地址
    1. 首先获取项目 package.json 中声明的首层依赖，包括 dependencies, devDependencies, optionalDependencies 声明的依赖
    2. 接着采用遍历首层依赖的方式获取依赖包的版本信息，以及递归查找每个依赖下嵌套依赖的版本信息，并将解析过和正在解析的包用一个 Set 数据结构来存储，这样就能保证同一个版本范围内的包不会被重复解析
3. **C、获取包（fetching packages）:** 这一步主要是利用系统缓存，到缓存中找到具体的包资源(查看缓存：yarn cache dir)
     1. 首先会尝试在缓存中查找依赖包，如果没有命中缓存，则将依赖包下载到缓存中。（判断系统中存在符合 "cachefolder+slug+node_modules+pkg.name" 规则的路径）
     2. 对于没有命中缓存的包，Yarn 会维护一个 fetch 队列，按照规则进行网络请求。这里也是 yarn 诞生之初解决 npm v3 安装缓慢问题的优化点，**支持并行下载**
4. **D、链接包（linking dependencies:** 这一步主要是将缓存中的依赖，复制到项目目录下，同时遵循扁平化原则  
5. **F、构建包（building fresh packages）:** 如果依赖包中存在二进制包需要进行编译，会在这一步进行。 

### 2. yarn和npm区别
1. lockfile。package-lock.json 自带版本锁定+依赖结构，你想改动一些依赖，可能影响的范围要比表面看起来的复杂的多；而 yarn.lock 自带版本锁定，并没有确定的依赖结构，使用 yarn 管理项目依赖，需要 package.json + yarn.lock 共同确定依赖的结构
2. 依赖管理策略
3. 性能， yarn性能在有缓存、重新install的情况下会比npm好不少


## pnpm
### 1. 特点
1. 包安装速度极快
2. 磁盘空间利用非常高效
3. 支持 monorepo
   之前对于多个项目的管理，我们一般都是使用多个 git 仓库，但 monorepo 的宗旨就是用一个 git 仓库来管理多个子项目，所有的子项目都存放在根目录的packages目录下，那么一个子项目就代表一个package
4. 安全性更高
   
### 2. 原理
#### 1. 速度快
**在绝多大数场景下，pnpm包安装的速度都是明显优于npm/yarn，速度会比 npm/yarn快2-3倍**
1. yarn PnP：直接去掉 node_modules，将依赖包内容写在磁盘，节省了 node 文件 I/O 的开销，这样也能提升安装速度。（具体原理见这篇文章https://loveky.github.io/2019/02/11/yarn-pnp/)

#### 2. 存储管理
**a. 基于内容寻址的文件系统来存储磁盘上所有的文件** 
1. 不会重复安装同一个包: 如果 100 个项目都依赖 lodash，那么 lodash 很可能就被安装了 100 次，磁盘中就有 100 个地方写入了这部分代码， 使用 pnpm 只会安装一次，磁盘中只有一个地方写入
2. 即使一个包的不同版本，pnpm 也会极大程度地复用之前版本的代码：比如 lodash 有 100 个文件，更新版本之后多了一个文件，那么磁盘当中并不会重新写入 101 个文件，而是保留原来的 100 个文件的 hardlink，仅仅写入那一个新增的文件

#### 3. 依赖管理
#### 4. 安全管理
### 3. 日常使用
1. pnpm install
2. pnpm update
3. pnpm uninstall
```js
// 移除 axios 
// --filter 来指定 package。
pnpm uninstall axios --filter package-a
```
4. pnpm link
```js
// 将本地项目连接到另一个项目。注意，使用的是硬链接，而不是软链
pnpm link ../../axios
```


<br/>

[文章来源](https://jishuin.proginn.com/p/763bfbd3bcff)

[文章来源](https://cloud.tencent.com/developer/article/1555982)

[文章来源](https://jishuin.proginn.com/p/763bfbd655cc)