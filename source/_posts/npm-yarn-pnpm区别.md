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

### 2. 缺点
## pnpm
### 1. 原理
### 2. 缺点


<br/>
[文章开源](https://jishuin.proginn.com/p/763bfbd3bcff)