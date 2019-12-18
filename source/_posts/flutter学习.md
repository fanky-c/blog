---
title: flutter学习
date: 2019-06-03 19:43:44
tags:
   - flutter
---

### Flutter使用命令
1. 创建flutter项目
   1. 运行命令行： flutter create projectName

2. 从pub下载包
   1. 打开 pubspec.yaml 文件，然后在dependencies下添加需要下载的包名
   2. 运行命令行：flutter packages get

3. 更新包的版本
   1. 运行命令行：flutter packages upgrade

### import导入文件的方式
1. import 'dart:xxx'; 引入Dart标准库
2. import 'xxx/xxx.dart';引入相对路径的Dart文件
3. import 'package:xxx/xxx.dart'; 引入Pub仓库pub.dev(或者pub.flutter-io.cn)中的第三方库
4. import 'package:project/xxx/xxx.dart';引入自定义的dart文件
5. import 'xxx' show compute1，compute2; 只导入compute1，compute2
6. import 'xxx' hide compute3; 除了compute都引入
7. import 'xxx' as compute4; 将库重命名，当有名字冲突时



### Widget框架
#### 基础的Widget包括常用
1. Text： 该 widget 可让创建一个带格式的文本。
2. Row、 Column：这些具有弹性空间的布局类Widget可让您在水平（Row）和垂直（Column）方向上创建灵活的布局。其设计是基于web开发中的Flexbox布局模型。
3. Stack:  取代线性布局 (译者语：和Android中的LinearLayout相似)，Stack允许子 widget 堆叠， 你可以使用 Positioned 来定位他们相对于Stack的上下左右四条边的位置。Stacks是基于Web开发中的绝度定位（absolute positioning )布局模型设计的
4. Container: Container 可让您创建矩形视觉元素。container 可以装饰为一个BoxDecoration, 如 background、一个边框、或者一个阴影。 Container 也可以具有边距（margins）、填充(padding)和应用于其大小的约束(constraints)。另外， Container可以使用矩阵在三维空间中对其进行变换
5. Scaffold： Material Design布局结构的基本实现。此类提供了用于显示drawer、snackbar和底部sheet的API。
6. Image： 一个显示图片的widget。
7. Icon: A Material Design icon。
8. RaisedButton: Material Design中的button， 一个凸起的材质矩形按钮。
9. Placeholder： 一个绘制了一个盒子的的widget，代表日后有widget将会被添加到该盒子中

#### Widget运行原理
1. Widget的主要工作是实现一个build函数，用以构建自身。
2. 一个Widget通常由一些较低级别widget组成。Flutter框架将依次构建这些widget，直到构建到最底层的子widget时，这些最低层的widget通常为RenderObject，它会计算并描述widget的几何形状。

#### Widget分类
1. StatelessWidget

2. StatefulWidget

#### Material组件
1. Scaffold是Material中主要的布局组件. 包含了appBar、body、title等等


### State
#### State介绍
1. State对象在多次调用build()之间保持不变，允许它们记住信息(状态)。


### Navigation & Router(导航与路由)

#### 静态路由
1. 静态路由，它不可以向下一个页面传递参数。
2. 静态路由，它可以接收下一个页面的返回值的
```dart
return new MaterialApp(
      title: 'Flutter Demo',
      theme: new ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: new MyHomePage(title: 'Flutter实例'),
      routes: <String, WidgetBuilder> {
        // 这里可以定义静态路由，不能传递参数
        '/router/second': (_) => new SecondPage(),
        '/router/home': (_) => new RouterHomePage(),
      },
    );

//push一个新页面,pushNamed方法是有一个Future的返回值的，
//所以静态路由也是可以接收下一个页面的返回值的。但是不能向下一个页面传递参数。
Navigator.of(context).pushNamed('/router/second');
Navigator.of(context).pushNamed('/router/second').then((value) {
   // dialog显示返回值
   _showDialog(context, value);
})

//pop回上一个页面
Navigator.of(context).pop('这个是要返回给上一个页面的数据')

```

#### 动态路由
1. 动态路由：自己生成页面对象，所以可以传递自己想要的参数。
2. Navigation.push();  //打开新页面
   1. Navigator.push(BuildContext context, Route route) //静态方法
   2. Navigator.of(context).push(Route route)  //实例方法，和上面的静态方法用途相同
3. Navigation.pop();   //退出当前页面
   1. Navigation.pop(BuildContext context, [ result ])
   2. Navigation.of(context).pop('传给上一个页面消息')
4. Navigator.replace()