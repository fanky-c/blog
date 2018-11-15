---
title: nginx
date: 2018-11-14 17:11:06
tags:
   - nginx
---

### 启动、停止、重启、配置文件校验

#### 开启
* 第一种方法  格式为： ngin地址 -c nginx配置文件位置
* 例如：/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf

#### 停止
##### A:从容停止  需要知道进程号:
* 1,查看nginx进程号命令为:ps -ef|grep nginx  只需要查看master的进程号
* 2,停止命令 kill -QUIT  进程号

##### B:强制停止
* pkill -9 nginx

#### 重启
* 进入sbin目录  命令 cd /usr/local/nginx/sbin
* ./nginx -s reload


#### 配置文件校验
* 进入sbin目录  命令 cd /usr/local/nginx/sbin
* ./nginx -t