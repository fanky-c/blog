---
title: Jenkins打造前端自动化工作流
date: 2020-05-08 20:36:21
tags:
    - Jenkis安装
    - 前端自动化工作流
---

### Jenkis安装
#### 安装Docker和 Docker Compose
#### 通过Docker安装Jenkis

### 初始化Jenkis
1. jenkins的默认端口是8080,启动成功后在浏览器打开。如果是阿里云或者腾讯云要在安全组添加对外访问端口。
2. 需要的插件选择默认即可。
3. 创建一个管理员账户。

### 前端自动化工作流
#### 创建任务
#### 实现git钩子
1. 打开刚创建的任务，选择配置，添加git远程仓库地址，配置登录名及密码及分支。
2. 安装Generic Webhook Trigger Plugin插件。
3. 添加触发器。
   1. 第2步安装的Webhook触发器插件功能很强大，可以根据不同的触发参数触发不同的构建操作，比如我向远程仓库提交的是master分支的代码，就执行代码部署工作，我向远程仓库提交的是某个feature分支，就执行单元测试，单元测试通过后合并至dev分支。灵活性很高


#### 实现自动化构建
1. jenkins添加NodeJS Plugin插件，在构建环境选择“Provide Node & npm bin/ folder to PATH”
2. 点击构建，把要执行的命令输进去，多个命令使用&&分开。
3. 此时本地修改一下代码push测试一下，就可以触发任务。


#### 实现自动化部署
1. Jenkins上装一个插件Publish Over SSH，我们将通过这个工具实现服务器部署功能。
2. 增加构建后操作步骤，选择send build artificial over SSH
3. 自己新增构建后自动化脚本



#### 实现邮件提醒
