---
title: 前端使用http提交数据方式
date: 2022-02-24 20:29:47
tags:
 - resful
 - query
 - form-urlencoded
 - form-data
 - application/json
---

## 支持方式
### url param
url param是一种resful风格（https://www.ruanyifeng.com/blog/2014/05/restful_api.html），把参数写在url中
```js
// 这里的 chao 就是路径中的参数（url param）, 获取chao相关信息
http://abc.com/person/chao
```

### query
通过 url 中 ？后面的用 & 分隔的字符串传递数据。 需要对数据做 url encode

```js
// 这里的 chao 就是路径中的参数（url param）, 获取chao相关信息
// 其中非英文的字符和一些特殊字符要经过编码，可以使用 encodeURLComponent 的 api，
// 或者使用封装了一层的 qeury-string 库来处理

http://abc.com/person?name=chao&age=12

```

### form-urlencoded
**用 form 表单提交数据就是这种，它和 query 字符串的方式的区别只是放在了 body 里，然后指定下 header头 content-type 是 application/x-www-form-urlencoded。 需要对数据做 url encode**

<img src="/img/form-urlencoded.png" />

通过 & 分隔的 form-urlencoded 的方式需要对内容做 url encode，如果传递大量的数据，比如上传文件的时候就不是很合适了，因为文件 encode 一遍的话太慢了，这时候就可以用 form-data

### form-data
form data 不再是通过 & 分隔数据，**而是用 --------- + 一串数字做为分隔符**。因为不是 url 的方式了，自然也不用再做 url encode。

**form-data 需要指定 content type 为 multipart/form-data，然后指定 boundary 也就是分割线。很明显，这种方式适合传输文件，而且可以传输多个文件**

<img src="/img/form-data.png" />

### application/json
**form-urlencoded 需要对内容做 url encode，而 form data 则需要加很长的 boundary，两种方式都有一些缺点。如果只是传输 json 数据的话，content-type 为 application/json**

<img src="/img/json.png" />

<br />
[文章来源于](https://mp.weixin.qq.com/s/Df3Q3IRBZlHq9RkSsHGT8g)
