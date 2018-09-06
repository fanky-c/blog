---
title: http/1.x和http/2
date: 2018-09-05 16:49:43
tags:
    - http
---
### http/1.x

#### 基本介绍

##### A:报文
  1. 起始⾏ 
  2. ⾸部（header）
  3. 主体

![报文](/blog/img/http_content.png)  

##### B:方法
  1. get 
     * 从服务器获取文档
     * 可以被缓存
     * 通过url请求，有长度限制
     * url明文请求，安全得不到保障
     * 对服务器无副作用，多次操作不会改变服务器状态
  2. post
     * 向服务器发送所需处理的数据
     * 不能被缓存
     * 没有长度限制
     * 请求放在主体，不会被记录下来，安全性相对来说得到保障
     * 有副作用，即非幂等
  3. head
     * 从服务器获取文档头部
  4. put
     * 请求的主体存到服务器中
  5. options
     * 查词可以服务器上执行那些方法,检查服务器性能（跨域请求用的多）

##### C：状态码
  1. 1xx 信息性状态
  2. 2xx 成功
     * 206：Partial Content 分块内容
  3. 3xx 重定向
  4. 4xx 客户端错误
     * 400: Bad Request 常⻅于请求时提交了错误的参数
     * 401：Unauthorized 需要验证
     * 403：Forbidden 禁⽌访问
     * 404：Not Found
  5. 5xx 服务端错误
     * 500：Server Error 服务器出错
     * 502：Bad Gateway 通常由代理服务器返回，表示上游服务器出错
     * 503：Service Unavailable 服务暂不可⽤，但将来可⽤（例如服务当前负载太⾼） 
     * 504：Gateway Timeout 通常由代理服务器返回，表示上游服务器超时


#### 常用header
  
#### cookie机制
* HTTP是⼀个匿名、⽆状态的协议，但很多情况下服务器需要识别客户端的身份
* 识别身份的⽅法:
  1. Authentication⾸部 + WWW-Authentication⾸部
  2. 胖URL，在URL⾥带上身份信息
  3. Cookie ⽬前识别⽤户，实现持久会话的最好⽅式
* cookie本质上是⼀些key-value数据，key是cookie名字，value是cookie的值
  1. domain和path: 指定了哪些域名和路径会带上该cookie
  2. HttpOnly属性阻⽌了javascript来访问cookie，可以防⽌⼀些XSS攻击
* 会话期Cookie: 浏览器关闭之后它会被⾃动删除
* 持久性Cookie: 指定了cookie过期时间

![cookie](/blog/img/http_cookie.png) 

#### 连接机制
##### 介绍
* HTTP使⽤TCP来传输数据，HTTP/1.0每个请求都新开⼀个连接，效率低下。
  1. TCP握⼿导致⽐较⻓的延时
  2. TCP拥塞控制算法在慢启动阶段带宽较⼩
* HTTP/1.1默认使⽤⻓连接，即⼀个连接会保持⼀段时间，⽤于发送⼀系列的请求
* 管道化请求：不等待前⼀个请求返回就在同⼀个连接发送下⼀个请求（⽬前都没实现）
* 相关⾸部：Connection （客户端和服务端都使⽤）
  1. Connection: keep-alive 在当前请求完成后，保持连接
  2. Connection: close 在当前请求完成后，关闭连接

##### HTTP/1.x 三种连接⽅式
![三种连接⽅式](/blog/img/http_live.png) 

#### 缓存

#### HTTP 1.x 缺点
* Head Of Line Blocking 队头阻塞
  1. HTTP/1.1实际上没实现pipelining，⼀条连接同时只能进⾏⼀个HTTP请求，若该请求的响应很慢，则后续使⽤该连接的HTTP请求都被阻塞
* ⾸部数据冗余：发往同⼀server的多个请求的Header重复发送
  1. gzip只会压缩实体数据
* 客户端请求-服务端响应模型（缺少服务端推送）  

### http/2

#### 连接复用
#### 首部压缩
#### 服务器推送



*** 资料来源于我的同事java大神郎哥的ppt,非常感谢他。 ***