---
title: Dart语法
date: 2019-11-09 19:08:29
tags:
   - Dart
---

### 相关概念
1. 所有能够使用变量引用的都是对象， 每个对象都是一个类的实例。在 Dart 中 甚至连数字、方法和 null 都是对象。所有的对象都继承于 Object 类
2. 使用静态类型( String, int,  bool, num) 可以更清晰的表明你的意图，并且可以让静态分析工具来分析你的代码， 但这并不是强制性的。（在调试代码的时候你可能注意到 没有指定类型的变量的类型为 dynamic。）
3. Dart 在运行之前会先解析你的代码。你可以通过使用 类型或者编译时常量来帮助 Dart 去捕获异常以及 让代码运行的更高效
4. Dart 支持顶级方法 (例如 main())，同时还支持在类中定义函数。 （静态函数和实例函数）。 你还可以在方法中定义方法 （嵌套方法或者局部方法）。
5. 同样，Dart 还支持顶级变量，以及 在类中定义变量（静态变量和实例变量）。 实例变量有时候被称之为域（Fields）或者属性（Properties）。
6. 和 Java 不同的是，Dart 没有 public、 protected、 和 private 关键字。如果一个标识符以 (_) 开头，则该标识符 在库内是私有的

### 关键词(keywords)

### 变量（variables）
#### 变量赋值 
```
var name = 'chao';  //上面名字为 name 的变量引用了 一个内容为 “chao” 的 String 对象
```
#### 默认值
```
//没有初始化的变量自动获取一个默认值为 null。
//类型为数字的变量如何没有初始化其值也是 null，不要忘记了数字类型也是对象。
int age;  
```
#### 可选的类型
在声明变量的时候，你可以选择加上具体 类型：
```
String name = 'chao';
```

#### final and const
如果你以后不打算修改一个变量，使用 final 或者 const。 一个 final 变量只能赋值一次；一个 const 变量是编译时常量。 （Const 变量同时也是 final 变量。） 顶级的 final 变量或者类中的 final 变量在 第一次使用的时候初始化。
> 注意： 实例变量可以为 final 但是不能是 const 。
```dart
final wordPair = new WordPair.random(); 
```

### 内置的类型(Built-in types)
#### 类型
##### numbers
1. 分为：int、double
2. strings和numbers转换
```dart
// String -> int
var one = int.parse('1');
assert(one == 1);

// String -> double
var onePointOne = double.parse('1.1');
assert(onePointOne == 1.1);

// int -> String
String oneAsString = 1.toString();
assert(oneAsString == '1');

// double -> String
String piAsString = 3.14159.toStringAsFixed(2);
assert(piAsString == '3.14');
```

##### strings
1. 可以在字符串中使用表达式，用法是这样的： ${expression}。如果表达式是一个比赛服，可以省略 {}。 如果表达式的结果为一个对象，则 Dart 会调用对象的 toString() 函数来获取一个字符串。
```dart
var s = 'string interpolation';
assert('Dart has $s, which is very handy.' ==
       'Dart has string interpolation, ' +
       'which is very handy.');
assert('That deserves all caps. ' +
       '${s.toUpperCase()} is very handy!' ==
       'That deserves all caps. ' +
       'STRING INTERPOLATION is very handy!');
```
2. 可以使用 + 操作符来把多个字符串链接为一个，也可以把多个 字符串放到一起来实现同样的功能：
```
var s1 = 'String ' 'concatenation'
         " works even over line breaks.";
assert(s1 == 'String concatenation works even over '
             'line breaks.');
var s2 = 'The + operator '
         + 'works, as well.';
assert(s2 == 'The + operator works, as well.');
```
3. 使用三个单引号或者双引号也可以 创建多行字符串对象：
```
var s1 = '''
You can create
multi-line strings like this one.
''';
var s2 = """This is also a
multi-line string.""";
```

##### booleans
1. 当 Dart 需要一个布尔值的时候，只有 true 对象才被认为是 true。 所有其他的值都是 false。这点和 JavaScript 不一样， 像 1、 "aString"、 以及 someObject 等值都被认为是 false。


##### lists (也被称之为 arrays)



#####  maps
##### runes (用于在字符串中表示 Unicode 字符)
##### symbols



[文字来源于](http://dart.goodev.org/guides/language/language-tour)