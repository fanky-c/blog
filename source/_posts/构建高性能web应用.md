---
title: 构建高性能web应用
date: 2019-02-17 14:55:53
tags:
    - 性能
---

####  什么是高性能web前端应用？
1. 打开慢
   1. 白屏、无样式时间长 -> css、图片、字体等加载和页面渲染时间太长
   2. 可用时间晚 -> js加载时间长
2. 反应慢
   1. 响应时间慢 -> 网络延迟、不稳定
   2. 操作没反应 -> 
   3. 动画卡顿、不顺畅 -> 内存占用高、内存泄露
3. 耗电

#### 在浏览器输入url到显示网页的过程:
DNS查询->tcp连接->http请求->服务器响应->客户端加载渲染

#### 浏览器工作过程(客户端加载渲染)
##### A:浏览器加载
1. 加载过程：
   1. 浏览器接受完整的html或者一个chunk资源。
   2. 当遇到请求外链css、js、image等资源，浏览器发送请求来获取相应的资源。这是个异步的请求过程，但是当js请求会阻塞后面的请求、解析、渲染。
    > 原因是js当初设计就是简单修改网页内容，所以会改变DOM、CSS，也就是说当js执行前后面所有资源的下载都是不必要的。当js加载、解析、执行完成后才进行html后面的渲染。
   3. css文件虽然不会阻塞js文件下载，但是会阻塞js文件执行。
    > var height = $('#id').height(); 浏览器必须保证css下载、解析完成后才执行这段代码。 
   4. css会阻塞DOM树解析吗？css会阻塞DOM树渲染吗？
2. 了解浏览器如何进行加载，我们可以在引用外部样式文件，外部css、js时，将他们放到合适的位置，使浏览器以最快的速度将文件加载完毕。

##### B:浏览器解析
1. 解析过程：
   1. DOM解析(DOM树)：html文档解析生成解析树即dom树，是由dom元素及属性节点组成，树的根是document对象。![image](https://image-static.segmentfault.com/323/594/3235942180-5960e0d629942_articlex)
   2. CSS解析：将css文件解析为样式表对象。该对象包含css规则，该规则包含选择器和声明对象。![image](http://upload-images.jianshu.io/upload_images/94752-a88c7ccbb30a132c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/630/format/webp)
2. 了解浏览器如何进行解析，我们可以在构建DOM结构，组织css选择器时，选择最优的写法，提高浏览器的解析速率。
   

##### C:浏览器渲染
1. 渲染过程：
   1. 根据CSS树和DOM树，生成布局（flow），即将所有渲染树的所有节点进行平面合成
   2. 将布局绘制（paint）在屏幕上
2. 了解浏览器如何进行渲染，明白渲染的过程，我们在设置元素属性，编写js文件时，可以减少”重绘“”重排“的消耗。
> 1. 重绘: 当一个元素的外观发生改变，但没有改变布局,重新把元素外观绘制出来的过程。
> 2. 重排: 当DOM的变化影响了元素的几何信息(DOM对象的位置和尺寸大小)，浏览器需要重新计算元素的几何属性，将其安放在界面中的正确位置。由于浏览器渲染界面是基于流式布局的，所以影响范围有全局和局部之分。

  
##### 网页生成的过程总结：
![image](/blog/img/brower.webp)


#### 性能优化方案
1. DNS查询:
   1. 拆分域名增加并行下载量，可以按照静态和动态资源进行域名划分
   2. 首页提前预获取DNS(dns-prefetch)，提前解析页面所需的域名：
   ![image](http://user-gold-cdn.xitu.io/2018/5/28/163a4d01ff02c0c5?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)
2. http连接和服务器响应：
   1. 减少和合并请求(http2可以不用这一步)
      1. 稍微大一点的图标可以用css雪碧图
      2. 小的图标用Base64编码，可以节约请求数
      3. 如果服务器支持（nginx-http-concat插件）可以把多个请求合并成一个：http://localhost/my-web/static/css??a.css,b.css，
   2. 减少传输资源的大小
      1. 服务器开启GZIP压缩
      2. js混淆压缩、图片压缩或者用更小的WebP图片格式以及css压缩  
      3. 减少不必要的cookie传输，禁用静态资源域名下的cookie(和上面的拆分域相配合)
   3. 充分利用浏览器缓存，减少重新请求服务器数据
      1. 缓存存储：Cache-Control: max-age=31536000
      2. 缓存过期：过期头 (Expires)
      3. 缓存对比：Last-Modified、E-Tag
      <img src="/blog/img/expires" width="100%" alt="缓存">
      ![image](/blog/img/expires1.png)
      >只有当ctrl+f5才会强制刷新，开发的时候一般建议用文件hash值避免这种修改的文件也被强缓存
   4. 单页面应用可以在webpack打包的时候使用code split和bundle split将代码分离动态加载
   > Bundle splitting: 实际上就是创建多个更小的文件，并行加载，以获得更好的缓存效果；主要的作用就是使浏览器并行下载，提高下载速度。并且运用浏览器缓存，只有代码被修改，文件名中的哈希值改变了才会去再次加载。
   > 
   > Code splitting: 只加载用户最需要的部分，其余的代码都遵从懒加载的策略；主要的作用就是加快页面加载速度，不加载不必要加载的东西。(require.ensure / Dynamic Imports)
   > [参考](https://mp.weixin.qq.com/s/LCqnIoRzIiy8zTyw6_avqA)
   5. nginx搭建反向代理，进行有效的负载均衡。nginx配置截取：
   ```
    http {
      upstream video {
        ip_hash;
        server 127.0.0.1:3000 weight = 2; //权重的值，越大被命中几率越大
        server 127.0.0.1:3001 weight = 1;
        server 127.0.0.1:3002 weight = 0.5;
      }
      server {
        listen: 8080;
        location / {
            proxy_pass: http://video
       }
     }
    }
   ```
3. 客户端加载渲染
   1. js阻塞浏览器其他资源加载，所以不是必须尽量放在页面底部或者async/defer等异步加载
   2. 优化关键渲染路径就是要优先显示和用户先关内容，可以把关键的css样式内联在页面，其他完成的样式可以异步加载
   3. DOM和样式结构嵌套不用太深，减少解析耗时
   4. 减少重排(回流)
      1. 在js中尽量减少DOM操作，尽量缓存访问DOM的样式信息，可以用操作class方式改变样式或者渲染动画，避免多次回流
      2. 如果一个文档区域要大量操作DOM，可以让当前区域脱离文档流(absolute、fixed、float)或者固定当前区域大小、位置。
      3. 由于隐藏的元素不会影响重新渲染，如果对一个元素进行复杂的操作，可以先隐藏再显示，这样只会重排二次。
      4. 现在流行的框架vue、reactjs都引入Virtual DOM。
         > 1，innerHTML: render html string O(template size) + 重新创建所有 DOM 元素 O(DOM size)
         > 
         > 2，Virtual : render Virtual DOM + diff O(template size) + 必要的 DOM 更新 O(DOM change)
   5. 减少重绘
      1. 利用缓存把多次的操作合并在一起，不要在循环中直接调用。
   6. 涉及到动画尽量开启GPU，js做动画会阻塞代码执行。transform: translateZ(0);
   7. 防止内存泄露（如果一个值不再需要了，引用数却不为0，垃圾回收机制无法释放这块内存）
      1. 全局变量。有时候业务需求要在模块定义全局变量，模块销毁的时候让变量等于null
      2. 定时器没被销毁。
      3. 脱离 DOM 的引用。
      4. 闭包，匿名函数可以访问父级作用域。
   8. 在开发的过程， 对于连续行为及短时间会触发很多次的浏览器事件,例如：scroll,resize, mousemove,我们急需对其进行性能优化。两种方式可以去优化我们的性能 函数节流(throttle) 和 函数去抖(debounce)[参考](http://demo.nimius.net/debounce_throttle/)
      >throttle策略的电梯。保证如果电梯第一个人进来后，15秒后准时运送一次，不等待。如果没有人，则待机

      ```
         var throttle = (function(){
             var previous = 0,
             before = +new Date;
             return function(func, wait){
                var now = +new Date;
               if(previous == 0){
                   before = +new Date;
                   previous++;
                   func();
                   return; 
                }

                if(now - before > wait){
                  previous++;
                  func();
                  before = now;
                 return;
                }
            }
         })()
      ```

      >debounce策略的电梯。如果电梯里有人进来，等待15秒。如果又人进来，15秒等待重新计时，直到15秒超时，开始运送。

      ```
         var dubounce = (function(){
           var timer = 0;
           return function(func, wait){
             clearTimeout(timer);

             timer = setTimeout(function(){
                func()
             }, wait)
           }   
        })()
       ```
  4. 耗电
     1. 主要是网络模块耗电。
     2. 整个UI可用用黑白色调节省电。