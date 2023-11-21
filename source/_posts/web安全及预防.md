---
title: web安全及预防
date: 2019-05-11 18:59:04
tags:
     - web安全
     - web安全措施
---

### web常见安全漏洞分类
#### 跨站脚本攻击(XSS)
1. 介绍： 攻击者在web在页面通过表单等方式插入script代码，当用户浏览网页时候，script代码会被执行，从而达到攻击目的。
2. 分类：
   1. 反射型（服务器渲染）。用户访问带xxs脚本url请求，服务端接受数据发送给浏览器，浏览器解析造成xss漏洞。
   2. 存储型。用户输入带XSS脚本的数据，服务端把数据存进数据库，页面被其他用户正常请求时，返回带XSS的数据。
   3. DOM类型。运用恶意脚本通过DOM动态地检查和修改页面内容。
3. 危害：
   1. 修改网页内容
   2. 盗取cookie
   3. 页面重定向
   4. 收集用户浏览器信息
4. 措施：
   1. 检测输入参数，对应<、>等字符进行过滤或者编码。
   2. 对输出参数进行htmlEncode编码或转义
   3. 设置httpOnly防御js获取cookie
   4. 设置csp(content-security-policy), 就是为了页面内容安全而制定的一系列防护策略. 通过CSP所约束的的规责指定可信的内容来源（这里的内容可以指脚本、图片、iframe、font、style等等可能的远程的资源）

#### SQL注入
1. 介绍：SQL注入是利用WEB服务，把精心设计的SQL命令注入到后台数据库执行。
2. 危害：
   1. 对数据库非法读取、删除、添加等
   2. 盗取用户铭感信息
3. 措施：
   1. 不应直接用前端传过来的sql做拼接，而应该用预编译语句，绑定变量，不改变sql语义。
   2. 用户输入参数验证
   3. 对关键数据库设置只读，减少风险。

#### 跨站点请求伪造（CSRF）
1. 介绍：攻击者盗用你的身份，以你的名义发送恶意请求。

2. 满足条件：
   1. 登录受信用网站A，并在本地生成Cookie
   2. 在不登出A网站情况下，访问危险的B网站。
   3. 案例：
      1. 如果登录电商网站赠送积分或者礼物，接口：http://www.mybank.com/Transfer.php？money=1000&to=zhangsan
      2. 攻击者在评论区发表：<img src=http://www.mybank.com/Transfer.php?money=1000&who=hacker>。 然后就源源不断收到礼物。
      3. 送礼物或礼物是有记录的，然后攻击者找到删除记录接口（发现是post方式），自己构造一个页面通过form表单删除，然后同window.location = 'XXXX' 返回之前网页。

3. 措施：
   1. 验证HTTP referer字段，判断连接从哪个页面跳转过来。
   2. 不用get请求，改用post。
   3. 在http请求参数添加随机token，在服务器建立拦截器，如果请求中无token或者错误的token则拒绝请求。
   4. 添加验证码、滑块验证。
   5. chrome51版本以上 cookie设置Same-site，直接禁止第三方访问



<br >
[参考一](https://tech.meituan.com/2018/09/27/fe-security.html)
[参考二](https://www.ruanyifeng.com/blog/2019/09/cookie-samesite.html)