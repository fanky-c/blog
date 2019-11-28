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
如果你以后不打算修改一个变量，使用 final 或者 const。 一个 final 变量只能赋值一次；一个 const 变量是编译时常量。 （const 变量同时也是 final 变量。） 顶级的 final 变量或者类中的 final 变量在 第一次使用的时候初始化。

1. const 只能被设一次值，在声明处赋值，且值必须为编译时常量；用于修饰常量。
```dart
const bar = 1000000;       // 定义常量值
// bar =13;   // 出现异常，const修饰的变量不能调用setter方法，即：不能设值，只能在声明处设值
const atm = 1.01325 * bar; // 值的表达式中的变量必须是编译时常量（bar）;

var c = 12;
//  atm = 1 * c;  //出错，因为c不是一个编译时常量，即：非const修饰的变量（只有const修饰的变量才是编译时常量）
```

2. final：只能被设一次值，在声明处赋值，值和普通变量的设值一样，可以是对象、字符串、数字等，用于修饰值的表达式不变的变量；
  1. 对象成员值能被修改，对于能够添加成员的类（如List、Map）则可以添加或删除成员。(类似js中const)
  2. 变量本身实例不能被修改。(类似js中const)

> 注意： 实例变量可以为 final 但是不能是 const 。
```dart
//const就不能用在表达实例对象上
final wordPair = new WordPair.random(); 
```


### 内置的类型(Built-in types)
#### 类型
##### number
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

##### string
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

##### boolean
1. 当 Dart 需要一个布尔值的时候，只有 true 对象才被认为是 true。 所有其他的值都是 false。这点和 JavaScript 不一样， 像 1、 "aString"、 以及 someObject 等值都被认为是 false。


##### list (也被称之为 array)
1. 也许 array （或者有序集合）是所有编程语言中最常见的集合类型。 在 Dart 中数组就是 List 对象。所以 通常我们都称之为 lists。
```
var list = [1, 2, 3];
assert(list.length == 3);
assert(list[1] == 2);
list[1] = 1;
assert(list[1] == 1);
```
2. 在 list 字面量之前添加 const 关键字，可以 定义一个不变的 list 对象（编译时常量）：
```
var constantList = const [1, 2, 3];
// constantList[1] = 1; // Uncommenting this causes an error.
```


#####  map
1. 通常来说，Map 是一个键值对相关的对象。 键和值可以是任何类型的对象。每个 键 只出现一次， 而一个值则可以出现多次。
```dart
var gifts = {
  'first' : 'partridge',
  'second': 'turtledoves',
  'fifth' : 'golden rings'
};

var gifts = new Map();
gifts['first'] = 'partridge';
gifts['second'] = 'turtledoves';

```

2. 同样使用 const 可以创建一个 编译时常量的 map：
```
final constantMap = const {
  2: 'helium',
  10: 'neon',
  18: 'argon',
};
// constantMap[2] = 'Helium'; // Uncommenting this causes an error.
```


##### rune (用于在字符串中表示 Unicode 字符)
##### symbol

### Function
1. Dart 是一个真正的面向对象语言，方法也是对象并且具有一种 类型， Function。 这意味着，方法可以赋值给变量，也可以当做其他方法的参数。 也可以把 Dart 类的实例当做方法来调用。
```
bool isNoble(int atomicNumber) {
  return _nobleGases[atomicNumber] != null;
}

//可以选择忽略类型定义
isNoble(atomicNumber) {
  return _nobleGases[atomicNumber] != null;
}
```
2. 必须参数和可选参数
```dart
//参数放到 [] 中就变成可选
String say(String from = 'china', String msg, [String device]) {
  var result = '$from says $msg';
  if (device != null) {
    result = '$result with a $device';
  }
  return result;
}
```
3. main() 入口函数
```
void main() {
  querySelector("#sample_text_id")
    ..text = "Click me!"
    ..onClick.listen(reverseText);
}
//.. 语法为 级联调用（cascade）。 使用级联调用语法， 你可以在一个对象上执行多个操作。
```


4. 静态作用域（scope）
```
var topLevel = true;
main() {
  var insideMain = true;

  myFunction() {
    var insideFunction = true;

    nestedFunction() {
      var insideNestedFunction = true;
      assert(topLevel);
      assert(insideMain);
      assert(insideFunction);
      assert(insideNestedFunction);
    }
  }
}
//nestedFunction() 可以访问所有的变量， 包含顶级变量
```
5. 闭包（closures）
一个 闭包 是一个方法对象，不管该对象在何处被调用， 该对象都可以访问其作用域内 的变量
```
Function makeAdder(num addBy) {
  return (num i) => addBy + i;
}
main() {
  // Create a function that adds 2.
  var add2 = makeAdder(2);

  // Create a function that adds 4.
  var add4 = makeAdder(4);

  assert(add2(3) == 5);
  assert(add4(3) == 7);
}
```

6. 返回值
所有的函数都返回一个值。如果没有指定返回值，则 默认把语句 return null; 作为函数的最后一个语句执行。


### 操作符
#### 算术操作符
```
var a, b;

a = 0;
b = ++a;        // Increment a before b gets its value.
assert(a == b); // 1 == 1

a = 0;
b = a++;        // Increment a AFTER b gets its value.
assert(a != b); // 1 != 0

a = 0;
b = --a;        // Decrement a before b gets its value.
assert(a == b); // -1 == -1

a = 0;
b = a--;        // Decrement a AFTER b gets its value.
assert(a != b); // -1 != 0
```

#### 级联操作符（..）
级联操作符 (..) 可以在同一个对象上连续调用多个函数以及访问成员变量。 使用级联操作符可以避免创建 临时变量， 并且写出来的代码看起来 更加流畅：
正确使用：
```
querySelector('#button') // Get an object.
  ..text = 'Confirm'   // Use its members.
  ..classes.add('important')
  ..onClick.listen((e) => window.alert('Confirmed!'));

//上面代码和下面一样效果
var button = querySelector('#button');
button.text = 'Confirm';
button.classes.add('important');
button.onClick.listen((e) => window.alert('Confirmed!'));
```

错误使用：
```
var sb = new StringBuffer();
sb.write('foo')..write('bar');
//sb.write() 函数返回一个 void， 无法在 void 上使用级联操作符。
```

### 异常（Exceptions）
#### 异常类型
1. Error
2. Exception

#### 异常调用
1. Throw
```
throw new FormatException('Expected'); 
throw 'Expected'; //任意对象
```

2. Catch
捕获异常可以避免异常继续传递（你重新抛出rethrow异常除外）
```dart
try{

}on Exception catch (e){  //指定exception类型错误

}catch(e){  //如果捕获语句没有指定类型，则可以捕获任意类型

}
```

3. rethrow
把捕获的异常给重新抛出。
```dart
final foo = '';
void misbehave() {
  try {
    foo = "You can't change a final variable's value.";
  } catch (e) {
    print('misbehave() partially handled ${e.runtimeType}.');
    rethrow; // Allow callers to see the exception.
  }
}

void main() {
  try {
    misbehave();
  } catch (e) {
    print('main() finished handling ${e.runtimeType}.');
  }
}
```

4. Finally
要确保某些代码执行，不管有没有出现异常都需要执行，可以使用 一个 finally 语句来实现。如果没有 catch 语句来捕获异常， 则在执行完 finally 语句后， 异常被抛出了：
```dart
try {
  breedMoreLlamas();
} catch(e) {
  print('Error: $e');  // Handle the exception first.
} finally {
  cleanLlamaStalls();  // Then clean up.
}
```

### Class
#### 介绍
Dart 是一个面向对象编程语言，同时支持基于 mixin 的继承机制。 每个对象都是一个类的实例，所有的类都继承于 Object.。 基于 Mixin 的继承 意味着每个类（Object 除外） 都只有一个超类，一个类的代码可以在其他 多个类继承中重复使用

#### 语法
##### ?. 来替代 . 
```dart
//?. 来替代 . 可以避免当左边对象为 null 时候 抛出异常
var p = new Point(2, 2);
p?.y = 4;
```
##### Constructors(构造函数)
A: 含义： 定义一个和类名字一样的方法就定义了一个构造函数
```dart
class Point {
  num x;
  num y;

  Point(num x, num y) {
    // this 指带当前的实例.
    this.x = x;
    this.y = y;
  }
}
```
B: 特点： 构造函数不会继承。



### 泛型
#### 为什么使用？
在Dart中类型是可选的，你可以选择不用泛型，使用泛型有下面几个好处：
1. 有些情况下你可能想使用类型来表明你的意图，不管是使用泛型还是具体类型。
2. 可以使用检查模式和静态分析工具提供的代码分析功能，提高代码健壮和安全性。
3. 减少重复的代码。一个方法可以实现多种不同类型的作用。
```dart
//String
abstract class StringCache {
  String getByKey(String key);
  setByKey(String key, String value);
}

//Object
abstract class ObjectCache {
  Object getByKey(String key);
  setByKey(String key, Object value);
}

//泛型实现
abstract class Cache<T> {
  T getByKey(String key);
  setByKey(String key, T value);
}
```

#### 泛型使用
1. 限制泛型类型
```dart
// T must be SomeBaseClass or one of its descendants.
class Foo<T extends SomeBaseClass> {...}

class Extender extends SomeBaseClass {...}

void main() {
  // It's OK to use SomeBaseClass or any of its subclasses inside <>.
  var someBaseClassFoo = new Foo<SomeBaseClass>();
  var extenderFoo = new Foo<Extender>();

  // It's also OK to use no <> at all.
  var foo = new Foo();

  // Specifying any non-SomeBaseClass type results in a warning and, in
  // checked mode, a runtime error.
  // var objectFoo = new Foo<Object>();
}
```

### 库
1. 内置库和包管理器提供的库pub
```dart
import 'dart:io';  //内置库
import 'package:mylib/mylib.dart'; //包管理器库
import 'package:utils/utils.dart';
```

2. 导入库的一部分
```dart
// Import only foo.
import 'package:lib1/lib1.dart' show foo;

// Import all names EXCEPT foo.
import 'package:lib2/lib2.dart' hide foo;
```

3. 延迟载入库
```dart
//deferred as
import 'package:deferred/hello.dart' deferred as hello;

//使用库标识符调用 loadLibrary() 函数来加载库
//await 关键字暂停代码执行一直到库加载完成
greet() async {
  await hello.loadLibrary();
  hello.printGreeting();
}
```

### 异步流程
#### 介绍
有两种方式可以使用 Future 对象中的 数据：
1. 使用 async 和 await
2. 使用 Future API

同样，从 Stream 中获取数据也有两种 方式：
1. 使用 async 和一个 异步 for 循环 (await for)
2. 使用 Stream API 


##### 使用
```dart
checkVersion() async {
  var version = await lookUpVersion();
  if (version == expectedVersion) {
    // Do something.
  } else {
    // Do something else.
  }
}
```
可以使用 try, catch, 和 finally 来处理使用 await 的异常


[文字来源于](http://dart.goodev.org/guides/language/language-tour)