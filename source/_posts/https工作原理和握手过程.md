---
title: https工作原理和握手过程
date: 2020-03-29 19:34:53
tags:
    - https工作原理
    - https握手过程
---

### 协议
1. https 相对于http多了一个SSL/TLS一层（ip -> tcp -> ssl/tls -> http）

### 加密算法
#### 对称算法
1. 加密和解密都是使用的同一个密钥。

#### 非对称算法
1. 公钥和私钥。 公钥和公钥算法是公开， 私钥是保密的。 安全性好但是性能差。

### https工作原理



### https握手过程
#### TCP三次握手
1. client -> server  (syn)

2. server -> client  (syn + ack)

3. client -> server  (ack + 内容) 

#### SSL握手
1. clinet -> server 
   1. 客户端版本号、32字节随机值、客户端支持的加密算法

2. server -> client 
   1. 客户端版本号、32字节随机值、客户端支持的加密算法
   2. 服务端公钥、证书颁发机构

3. client -> server
   1. 根据之前服务器发送来的公钥进行加密
   2. 告诉服务器以后信息都是用协商好的密钥和算法

4. server -> client
  1. 告诉客户端会使用协商好的密钥来加密