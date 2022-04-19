---
title: 图解http读书笔记
date: 2022-03-13 15:39:41
tags:
- 图解http
- 读书笔记
---

## web及网络基础
### 1、IP、TCP和DNS
1. 负责传输的IP协议
IP（Internet Protocol）网际协议位于网络。 IP地址指明了节点被分配到的地址，MAC地址是指网卡所属的固定地址。IP地址可以和MAC地址进行配对。IP地址可变换，但MAC地址基本上不会更改。

2. 确保可靠性的TCP协议
TCP位于传输层，提供可靠的字节流服务。TCP协议为了更容易传送大数据才把数据分割，而且TCP协议能够确认数据最终是否送达到对方。

<img src="/img/http1.jpeg" style="max-width:95%" />
1. 发送端首先发送一个带SYN标志的数据包给对方。
2. 接收端收到后，回传一个带有SYN/ACK标志的数据包以示传达确认信息。
3. 最后，发送端再回传一个带ACK标志的数据包，代表“握手”结束。

3. 负责域名解析的DNS服务
DNS（Domain Name System）服务是和HTTP协议一样位于应用层的协议。它提供域名到IP地址之间的解析服务

### 2、 URI和URL区别
1. 与URI（统一资源标识符）相比，我们更熟悉URL（UniformResource Locator，统一资源定位符）。URL正是使用Web浏览器等访问Web页面时需要输入的网页地址
2. URI就是由某个协议方案表示的资源的定位标识符，采用HTTP协议时，协议方案就是http。除此之外，还有ftp、mailto、telnet、file等
3. URI用字符串标识某一互联网资源，而URL表示资源的地点（互联网上所处的位置）。可见URL是URI的子集


## HTTP报文
### 报文结构
<img src="/img/http2.jpeg" alt="请求报文（上）和响应报文（下）的结构" style="max-width:95%" />
上图：请求报文（上）和响应报文（下）的结构
<img src="/img/http3.jpeg" alt="请求报文（上）和响应报文（下）的实例" style="max-width:95%" />
上图：请求报文（上）和响应报文（下）的实例

### 编码提示传输速率
#### 介绍
1. 报文（message）: 是HTTP通信中的基本单位，由8位组字节流（octet sequence，其中octet为8个比特）组成，通过HTTP通信传输。
2. 实体（entity）: 作为请求或响应的有效载荷数据（补充项）被传输，其内容由实体首部和实体主体组成

*通常，报文主体等于实体主体。只有当传输中进行编码操作时，实体主体的内容发生变化，才导致它和报文主体产生差异*

#### 压缩传输的内容编码
内容编码指明应用在实体内容上的编码格式，并保持实体信息原样压缩。内容编码后的实体由客户端接收并负责解码。常用的内容编码有：gzip、deflate
<img src="/img/http4.jpeg" alt="" style="max-width:95%" />
#### 分割发送的分块传输编码
在HTTP通信过程中，请求的编码实体资源尚未全部传输完成之前，浏览器无法显示请求页面。在传输大容量数据时，通过把数据分割成多块，能够让浏览器逐步显示页面。这种把实体主体分块的功能称为分块传输编码（Chunked Transfer Coding）
<img src="/img/http5.jpeg" alt="" style="max-width:95%" />

## 状态码
