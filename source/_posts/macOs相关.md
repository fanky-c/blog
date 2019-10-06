---
title: macOs相关
date: 2019-10-06 16:27:38
tags:
   - mac
   - ios
---

### 线程之间通信
#### performSelector
1. - (void)performSelectorOnMainThread:(SEL)aSelector withObject:(id)arg waitUntilDone:(BOOL)wait;  //返回主线程
2. - (void)performSelector:(SEL)aSelector onThread:(NSThread *)thr withObject:(id)arg waitUntilDone:(BOOL)wait; //返回指定线程


### load和initialize方法分别在调用时机
#### + load
1. 它应该是在一个类进行装载的时候触发，就是不管这个类有没有被调用，只要它被装载，那么它就会运行这个方法，只要被添加到编译源下面就会执行。
2. 不管子类有没有写load方法，父类的load都只会执行一次。
3. load方法执行的时候，系统为脆弱状态，如果我们在load里面需要调用其它类的实例对象(或类对象)的属性或者方法，必须要确保那个依赖类的load方法执行完毕
4. 当加载资源过大会造成性能问题（用户体验、系统抖动）

#### + initialize
1. 如果没有用到该类，就算加载完毕也不会执行该方法（这点与load方法不同，load方法是只要加载就执行，initialize方法必须是第一次使用该类的时候才触发且触发一次）
2. 以懒加载的方式被调用的，不是启动程序就调用。



### runloop和线程的关系
1. 每条线程都有唯一的一个 RunLoop 对象与之对应的
2. 主线程的 RunLoop 是自动创建并启动
3. 子线程的 RunLoop 需要手动创建(懒加载,只创建一次)
```
 //启动RunLoop
[[NSRunLoop currentRunLoop] run];


//第一个参数:指定运行模式
//第二个参数:指定 RunLoop 的过期时间,即:到了这个时间后RunLoop 就失效了
[[NSRunLoop currentRunLoop] runMode:kCFRunLoopDefaultModebeforeDate:[NSDate distantFuture]];
```