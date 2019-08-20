---
title: ios中定义属性atmoic
date: 2019-08-20 16:37:38
tags:
    - nonatomic
    - assign
    - strong
    - weak
---

### 定义
定义属性中的特性有atomic、nonatomic、copy、assign、strong、weak等，一般格式如下：
```js
@property (nonatomic, strong) NSString *name; 
```

### 使用

#### atomic
1. 默认属性
2. 当前进程进行到一半，其他线程来访问当前线程，可以保证先执行完毕当前线程
3. 只是保证setter/getter 完整，不是线程安全


#### nonatomic
1. 非默认属性
2. 两个线程同时访问同一个属性将会导致无法预计的结果
3. 优点是程序运行速度快


#### copy
1. 是owner，不是reference（引用）。当对象可变时，可设置为copy，用于获取此时值的副本
2. 使用copy创建的新对象也是强引用，使用完成后需要负责释放该对象


#### assign
1. 与copy相反，只是reference，不是owner。只返回指针
2. 用于float、int、BOOL等类型
3. 释放后再发送消息会导致程序崩溃


#### strong
1. 默认属性
2. strong = retain iOS引入ARC后，用strong替代了retain
3. 所有实例变量、局部变量默认都是strong
4. 创建一个强引用的指针，引用对象引用计数加1
5. 如果有多个对象同时引用一个属性，任一对象对该属性的修改都会影响其他对象获取的值



#### weak
1. 只是reference，不是owner。即引用计数不会加1
2. IBOutlet常用weak
3. 可将weak对象设为nil，向nil发送消息，什么都不会执行，程序也不会崩溃
4. 代理使用weak。delegate几乎一直own代理对象，所以代理对象应该对代理使用weak，否则会形成循环引用（retain cycle）。但也有例外，如果代理对象的生命周期比代理短，代理对象也可以使用strong


#### readonly
1. 非默认属性
2. 只有可读方法，也就是只有getter方法