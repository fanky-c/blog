---
title: javascript高级程序设计读书笔记
date: 2022-06-21 21:34:31
tags:
 - javascript高级程序设计
 - 读书笔记
---

## 基本概念
任何语言的核心都必然会描述这门语言最基本的工作原理。而描述的内容通常都要涉及这门语言的**语法、操作符、数据类型、内置功能等用于构建复杂解决方案的基本概念**。

### 1、语法
####  1.1 区分大小写
ECMAScript中的一切（变量、函数名和操作符）都区分大小写。这也就意味着，变量名test和变量名Test分别表示两个不同的变量。

####  1.2  标识符
第一个字符必须是一个字母、下划线（_）或一个美元符号（$）；

#### 1.3 语句
ECMAScript中的语句以一个分号结尾；如果省略分号，则由解析器确定语句的结尾。**加上分号也会在某些情况下增进代码的性能，因为这样解析器就不必再花时间推测应该在哪里插入分号了。**


### 2、数据类型
**JS基本数据类型：String、Number、Boolean、Undefined、Null、Symbol、BigInt；Object本质上是由一组无序的名值对组成的**

**ECMAScript中也有一种复杂的数据类型，即Object类型，该类型是这门语言中所有对象的基础类型。**

#### 2.1 typeof操作符
对一个值使用typeof操作符可能返回下列某个字符串：

1. "undefined"——如果这个值未定义；
2. "boolean"——如果这个值是布尔值；
3. "string"——如果这个值是字符串；
4. "symbol"——如果这个值是symbol；
5. "number"——如果这个值是数值；
6. "object"——如果这个值是对象或null；
7. "function"——如果这个值是函数。

#### 2.2  Undefined类型
Undefined类型只有一个值，即特殊的undefined。在使用var声明变量但未对其加以初始化时，这个变量的值就是undefined。 

对于尚未声明过的变量，只能执行一项操作，即使用typeof操作符检测其数据类型。 


#### 2.3  Null类型
从逻辑角度来看，null值表示一个空对象指针，而这也正是使用typeof操作符检测null值时会返回"object"的原因。

```js
typeof null // 'object'
```

尽管null和undefined有这样的关系，但它们的用途完全不同。如前所述，无论在什么情况下都没有必要把一个变量的值显式地设置为undefined，可是同样的规则对null却不适用。**换句话说，只要意在保存对象的变量还没有真正保存对象，就应该明确地让该变量保存null值。这样做不仅可以体现null作为空对象指针的惯例，而且也有助于进一步区分null和undefined。**


#### 2.4 Number类型

##### 2.4.1 NaN
NaN，即非数值（Not a Number）是一个特殊的数值，这个数值用于表示一个本来要返回数值的操作数未返回数值的情况（这样就不会抛出错误了）。例如，在其他编程语言中，任何数值除以非数值都会导致错误，从而停止代码执行。但在ECMAScript中，任何数值除以非数值会返回NaN，因此不会影响其他代码的执行。

NaN与任何值都不相等，包括NaN本身


##### 2.4.2 数值转换
Number()、parseInt()和parseFloat()

**Number()函数的转换规则如下:**
1. 如果是Boolean值，true和false将分别被转换为1和0。
2. 如果是数字值，只是简单的传入和返回
3. 如果是null值，返回0
4. 如果是undefined，返回NaN。
5. 如果是对象，则调用对象的valueOf()方法，然后依照前面的规则转换返回的值。如果转换的结果是NaN，则调用对象的toString()方法，然后再次依照前面的规则转换返回的字符串值
6. 如果是字符串，遵循下列规则：
   1. 如果字符串中只包含数字（包括前面带正号或负号的情况），则将其转换为十进制数值，即"1"会变成1, "123"会变成123，而"011"会变成11（注意：前导的零被忽略了）；
   2.  如果字符串中包含有效的浮点格式，如"1.1"，则将其转换为对应的浮点数值（同样，也会忽略前导零)
   3.  如果字符串中包含有效的十六进制格式，例如"0xf"，则将其转换为相同大小的十进制整数值。


**parseInt()函数**
parseInt()函数在转换字符串时，更多的是看其是否符合数值模式。它会忽略字符串前面的空格，直至找到第一个非空格字符。

1. 如果第一个字符不是数字字符或者负号，parseInt()就会返回NaN；也就是说，用parseInt()转换空字符串会返回NaN（Number()对空字符返回0）
2. 如果第一个字符是数字字符，parseInt()会继续解析第二个字符，直到解析完所有后续字符或者遇到了一个非数字字符


**不指定基数意味着让parseInt()决定如何解析输入的字符串，因此为了避免错误的解析，我们建议无论在什么情况下都明确指定基数。**
```js
let num1 = parseInt("10", 2) // 2 按二进制解析
let num1 = parseInt("10", 8) // 8 按八进制解析
let num1 = parseInt("10", 10) // 10 按十进制解析
let num1 = parseInt("10", 16) // 16 按十六进制解析
```


#### 2.5 String类型

##### 2.5.1 转换为字符串

要把一个值转换为一个字符串有三种方式：
 1. 第一种是使用几乎每个值都有的toString()方法；
 2. 在不知道要转换的值是不是null或undefined的情况下，还可以使用转型函数String()，这个函数能够将任何类型的值转换为字符串。String()函数遵循下列转换规则
    1. 如果值有toString()方法，则调用该方法（没有参数）并返回相应的结果；
    2. 如果值是null，则返回"null"；
    3. 如果值是undefined，则返回"undefined"。
 3. 可以使用加号操作符把它与一个字符串（""）加在一起


#### 2.6 Object类型
对象其实就是一组数据和功能的集合。

**Object类型是所有它的实例的基础。换句话说，Object类型所具有的任何属性和方法也同样存在于更具体的对象中。**

##### 2.6.1 创建
```js
let o = new Object();

let o = new Object; // 有效，但是不推荐
```

##### 2.6.2 Object每个实例具有以下属性和方法
1. constructor: 保存着用于创建当前对象函数， 对应当前的例子，构造函数（constructor）就是Object;
2. hasOwnProperty(propertyName)： 用于检查给定的属性在当前对象实例中（而不是在实例原型中）是否存在；
3. isPrototypeOf(object): 用于检查传入的对象是否是当前对象的原型；
4. propertyIsEnumerable(propertyName): 用于检查给定的属性是否能够使用for-in语句来枚举;
5. toString()：返回对象的字符串表示;
6. valueOf()：返回对象的字符串、数值或布尔值表示。通常与toString()方法的返回值相同。


### 3、操作符
#### 3.1 一元操作符
##### 3.1.1 递增和递减操作符
**前置递增和递减操作时，变量的值都是在语句被求值以前改变的；后置递增和递减与前置递增和递减有一个非常重要的区别，即递增和递减操作是在包含它们的语句被求值之后才执行的**

```js
// 前置
let num1 = 2;
let num2 = 20;
let num3 = --num1 + num2; // 21
let num4 = num1 + num2; // 21

// 后置
let num1 = 2;
let num2 = 20;
let num3 = num1-- + num2; // 22
let num4 = num1 + num2; // 21
```

它们不仅适用于整数，还可以用于字符串、布尔值、浮点数值和对象
1. 在应用于一个包含有效数字字符的字符串时，先将其转换为数字值，再执行加减1的操作。字符串变量变成数值变量。
2.  在应用于对象时，先调用对象的valueOf()方法以取得一个可供操作的值。然后对该值应用前述规则。如果结果是NaN，则在调用toString()方法后再应用前述规则。对象变量变成数值变量

#### 3.2 位操作符

#### 3.3 乘性操作符
##### 3.3.1  乘法
1. 如果有一个操作数是NaN，则结果是NaN；
2. 如果是Infinity与0相乘，则结果是NaN； 
3. 如果是Infinity与非0数值相乘，则结果是Infinity或-Infinity，取决于有符号操作数的符号；
4. 如果是Infinity与Infinity相乘，则结果是Infinity；
5. 如果有一个操作数不是数值，则在后台调用Number()将其转换为数值，然后再应用上面的规则。


##### 3.3.2 求模

```js
let result = 26 % 5; // 1
```

求模相关规则：
1.  如果被除数是零，则结果是零
2.  如果有一个操作数不是数值，则在后台调用Number()将其转换为数值，然后再应用上面的规则


#### 3.4 加性操作符
##### 3.4.1 加法

相关规则：
1. 如果有一个操作数是对象、数值或布尔值，则调用它们的toString()方法取得相应的字符串值，然后再应用前面关于字符串的规则。对于undefined和null，则分别调用String()函数并取得字符串"undefined"和"null"。


##### 3.4.2 减法

相关规则：
1. 如果有一个操作数是字符串、布尔值、null或undefined，则先在后台调用Number()函数将其转换为数值，然后再根据前面的规则执行减法计算。如果转换的结果是NaN，则减法的结果就是NaN；
2. 如果有一个操作数是对象，则调用对象的valueOf()方法以取得表示该对象的数值。如果得到的值是NaN，则减法的结果就是NaN。如果对象没有valueOf()方法，则调用其toString()方法并将得到的字符串转换为数值


### 4、流程控制语句
#### 4.1 do-while语句
do-while语句是一种后测试循环语句，即只有在循环体中的代码执行之后，才会测试出口条件。换句话说，在对条件表达式求值之前，循环体内的代码至少会被执行一次。

```js
var i = 0;

do {
  i += 2;
} while (i < 10)
```

#### 4.2 for语句
由于ECMAScript中不存在块级作用域，因此在循环内部定义的变量也可以在外部访问到。

```js
for (var i=0; i<10; i++){
   alert(i);
}
alert(i) // 9
```

for语句中的初始化表达式、控制表达式和循环后表达式都是可选的。将这三个表达式全部省略，就会创建一个无限循环

```js
for (; ;){
   // 无限循环
}
```

#### 4.3 for-in语句
ECMAScript对象的属性没有顺序。因此，通过for-in循环输出的属性名的顺序是不可预测的。具体来讲，所有属性都会被返回一次，但返回的先后次序可能会因浏览器而异


#### 4.4 break和continue语句
break语句会立即退出循环，强制继续执行循环后面的语句。而continue语句虽然也是立即退出循环，但退出循环后会从循环的顶部继续执行


#### 4.5 with语句
with语句的作用是将代码的作用域设置到一个特定的对象中；定义with语句的目的主要是为了简化多次编写同一个对象的工作

```js
var qs=location.search.substring(1);
var hostName=location.hostname;
var url=location.href;

// with语句
with(location){
   var qs=search.substring(1);
   var hostName=hostname;
   var url=href;
}
```

with语句的代码块内部，每个变量首先被认为是一个局部变量，而如果在局部环境中找不到该变量的定义，就会查询location对象中是否有同名的属性。如果发现了同名属性，则以location对象属性的值作为变量的值。 如果locaion没有在window全局对象找。


#### 4.6 switch语句
虽然ECMAScript中的switch语句借鉴自其他语言，但这个语句也有自己的特色。首先，可以在switch语句中使用任何数据类型（在很多其他语言中只能使用数值），无论是字符串，还是对象都没有问题。其次，每个case的值不一定是常量，可以是变量，甚至是表达式。


switch语句在比较值时使用的是全等操作符，因此不会发生类型转换（例如，字符串"10"不等于数值10）。

```js
switch ("hello"){
  case "hello":
   alert('hello');
   break;
  case "good":
   alert("good");
   break;
   default:
      alert("wtf");  
}

// 另外
let num = 1;
switch (true){
  case num < 0:
   alert('less than 0');
   break;
  case num >= 0 && num <= 10:
   alert("between 0 and 10");
   break;
   default:
      alert("more than 10");  
}
```

### 5、函数

#### 5.1 参数
ECMAScript函数不介意传递进来多少个参数，也不在乎传进来参数是什么数据类型。也就是说，即便你定义的函数只接收两个参数，在调用这个函数时也未必一定要传递两个参数。可以传递一个、三个甚至不传递参数，而解析器永远不会有什么怨言。之所以会这样，原因是ECMAScript中的参数在内部是用一个数组来表示的。函数接收到的始终都是这个数组，而不关心数组中包含哪些参数（如果有参数的话）。如果这个数组中不包含任何元素，无所谓；如果包含多个元素，也没有问题。实际上，在函数体内可以通过arguments对象来访问这个参数数组，从而获取传递给函数的每一个参数。

**arguments对象只是与数组类似（它并不是Array的实例），因为可以使用方括号语法访问它的每一个元素（即第一个元素是arguments[0]，第二个元素是arguments[1]，以此类推），使用length属性来确定传递进来多少个参数。**


```js
function doNum(num1, num2){
   arguments[1] = 10;
   return arguments[0] + num2;
}

doNum(1, 2)  // 11
```
每次执行这个doAdd()函数都会重写第二个参数，将第二个参数的值修改为10。因为arguments对象中的值会自动反映到对应的命名参数，所以修改arguments[1]，也就修改了num2，结果它们的值都会变成10。不过，这并不是说读取这两个值会访问相同的内存空间；它们的内存空间是独立的，但它们的值会同步。


没有传递值的命名参数将自动被赋予undefined值。这就跟定义了变量但又没有初始化一样。例如，如果只给doAdd()函数传递了一个参数，则num2中就会保存undefined值。

**重点：ECMAScript中的所有参数传递的都是值，不可能通过引用传递参数。**

#### 5.2 没有重载
ECMAScript函数不能像传统意义上那样实现重载。而在其他语言（如Java）中，可以为一个函数编写两个定义，只要这两个定义的签名（接受的参数的类型和数量）不同即可。如前所述，ECMAScirpt函数没有签名，因为其参数是由包含零或多个值的数组来表示的。而没有函数签名，真正的重载是不可能做到的。


## 变量、作用域和内存问题
JavaScript的变量与其他语言的变量有很大区别。JavaScript变量松散类型的本质，决定了它只是在特定时间用于保存特定值的一个名字而已。由于不存在定义某个变量必须要保存何种数据类型值的规则，变量的值及其数据类型可以在脚本的生命周期内改变。尽管从某种角度看，这可能是一个既有趣又强大，同时又容易出问题的特性，


### 1、基本类型和引用类型的值
#### 1.1 基本类型和引用类型的值

基本类型值指的是简单的数据段，而引用类型值指那些可能由多个值构成的对象。

引用类型的值是保存在内存中的对象。与其他语言不同，JavaScript不允许直接访问内存中的位置，也就是说不能直接操作对象的内存空间。在操作对象时，实际上是在操作对象的引用而不是实际的对象。为此，引用类型的值是按引用访问的。

##### 1.1.1 动态的属性

我们不能给基本类型的值添加属性，尽管这样做不会导致任何错误； 

```js
let name = 'cha';
name.age = 12;
console.log(name.age) // undefined
```

##### 1.1.2 复制变量值
当从一个变量向另一个变量复制引用类型的值时，同样也会将存储在变量对象中的值复制一份放到为新变量分配的空间中。不同的是，这个值的副本实际上是一个指针，而这个指针指向存储在堆中的一个对象。复制操作结束后，两个变量实际上将引用同一个对象。因此，改变其中一个变量，就会影响另一个变量，

```js
var obj1=new Object();
var obj2=obj1;
obj1.name="Nicholas";
alert(obj2.name);   //"Nicholas"
```

<img src="/img/heap.jpeg" width="95%" height="auto">


##### 1.1.3 传递参数
ECMAScript中所有函数的参数都是按值传递的。也就是说，把函数外部的值复制给函数内部的参数，就和把值从一个变量复制到另一个变量一样。

```js
// 基本类型
function addTen(num){
  num += 10;
  return num;
}
var count = 20;
var result = addTen(count);
console.log(count); // 20
console.log(result); // 30

/**
 * 在这个函数内部，obj和person引用的是同一个对象
 */

// 引用类型
function setName(obj) {
   obj.name = 'nick';
}
let person = new Object();
setName(person);
console.log(person.name) // nick



/**
 *  这说明即使在函数内部修改了参数的值，但原始的引用仍然保持未变。
 *  实际上，当在函数内部重写obj时，这个变量引用的就是一个局部对象了。
 *  而这个局部对象会在函数执行完毕后立即被销毁。
 **/

// 引用类型
function setName(obj){
  obj.name = 'nick';
  obj = new Object();
  obj.name = 'greg';
}

let person = new Object();
setName(person);
console.log(person.name); // nick
```

关于引用类型错误的理解：

因为person指向的对象在堆内存中只有一个，而且是全局对象。有很多开发人员错误地认为：在局部作用域中修改的对象会在全局作用域中反映出来，就说明参数是按引用传递的。

关于引用类型正确理解：

在把person传递给setName()后，其name属性被设置为"Nicholas"。然后，又将一个新对象赋给变量obj，同时将其name属性设置为"Greg"。如果person是按引用传递的，那么person就会自动被修改为指向其name属性值为"Greg"的新对象。但是，当接下来再访问person.name时，显示的值仍然是"Nicholas"


##### 1.1.4 检测类型

typeof操作符是确定一个变量是字符串、数值、布尔值，还是undefined的最佳工具。如果变量的值是一个对象或null，则typeof操作符会像下面例子中所示的那样返回"object"。

所有引用类型的值都是Object的实例。因此，在检测一个引用类型值和Object构造函数时，instanceof操作符始终会返回true。

```js
// 无法区分Object 和 其他类型
{} instanceof Object // true
[] instanceof Object // true
[] instanceof Array // true
new RegExp() instanceof Object // true
new RegExp() instanceof RegExp // true
```


### 2、执行环境和作用域
执行环境（execution context，为简单起见，有时也称为“环境”）是JavaScript中最为重要的一个概念。执行环境定义了变量或函数有权访问的其他数据，决定了它们各自的行为。

内部环境可以通过作用域链访问所有的外部环境，但外部环境不能访问内部环境中的任何变量和函数。

#### 2.1 延长作用域链


#### 2.2 没有块级作用域（es6 let实现）
这里是在一个if语句中定义了变量color。如果是在C、C++或Java中，color会在if语句执行完毕后被销毁。但在JavaScript中，if语句中的变量声明会将变量添加到当前的执行环境（在这里是全局环境）中。在使用for语句时尤其要牢记这一差异

```js
for (var i=0; i < 10; i++){
   doSomething(i);
}
alert(i);        //10
```

### 3、垃圾回收
这种垃圾收集机制的原理其实很简单：找出那些不再继续使用的变量，然后释放其占用的内存。为此，垃圾收集器会按照固定的时间间隔（或代码执行中预定的收集时间），周期性地执行这一操作。

#### 3.1 标记清除

#### 3.2 引用计数

#### 3.3 性能问题

#### 3.4  管理内存

解除一个值的引用并不意味着自动回收该值所占用的内存。解除引用的真正作用是让值脱离执行环境，以便垃圾收集器下次运行时将其回收。

```js
function createPerson(name){
   var localPerson=new Object();
   localPerson.name=name;
   return localPerson;
}


var globalPerson=createPerson("Nicholas");


// 手工解除globalPerson的引用
globalPerson=null;
```
## 引用类型
### 1、Object类型

### 2、Array类型
#### 2.1 创建

```js
let color = new Array(3); // 创建包含3项数组
let color = new Array("red"); // 创建一个包含1项， 即字符串：red

// 与对象一样，在使用数组字面量表示法时，也不会调用Array构造函数
let color = ["red"];

// 数组length可读可写， 可以从数组的末尾移除项或向数组中添加新项
let colors = ['red', 'green', 'yellow'];
colors.length = 2; // console.log(colors[2]) undefined

colors[colors.length] = 'yellow'; // 添加数组
colors.length = 4; // console.log(colors[3]) undefined
```

#### 2.2 检测数组

```js
// 方法一
arr instanceof Array

// 方法二
Array.isArray(arr)

// 方法三
Object.prototype.toString.call(arr) === '[object Array]'
```

#### 2.3 数组api

**栈方法**

具体说来，数组可以表现得就像栈一样，后者是一种可以限制插入和删除项的数据结构。栈是一种LIFO（Last-In-First-Out，后进先出）的数据结构，也就是最新添加的项最早被移除。push(推入最后一项) 和 pop (获取最后一项) 

**队列方法**

队列数据结构的访问规则是FIFO（First-In-First-Out，先进先出） unshift(推入第一项) 和 pop(获取最后一项)

**重排序方法**

reverse()方法会反转数组项的顺序

**sort()方法按升序排列数组项——即最小的值位于最前面，最大的值排在最后面。为了实现排序，sort()方法会调用每个数组项的toString()转型方法，然后比较得到的字符串，以确定如何排序。即使数组中的每一项都是数值，sort()方法比较的也是字符串**

```js
[0, 1, 5, 10, 15].sort() // [0, 1, 10, 15, 5]

[0, 1, 5, 10, 15].sort(function(a,b){
   if(a < b){
     return -1;
   }else if(a > b){
     return 1;
   }else{
      return 0;
   }
})

```

**操作方法**

concat()方法可以基于当前数组中的所有项创建一个新数组,具体来说，这个方法会先创建当前数组一个副本，然后将接收到的参数添加到这个副本的末尾，最后返回新构建的数组。


**slice()：它能够基于当前数组中的一或多个项创建一个新数组。slice()方法可以接受一或两个参数，即要返回项的起始和结束位置。在只有一个参数的情况下，slice()方法返回从该参数指定位置开始到当前数组末尾的所有项。如果有两个参数，该方法返回起始和结束位置之间的项——但不包括结束位置的项。注意，slice()方法不会影响原始数组。**

```js
let color = [1, 2, 3];
let color2 = color.slice(1); // color不变， color2:[2, 3]
```


**splice()：这个方法恐怕要算是最强大的数组方法了，它有很多种用法。splice()的主要用途是向数组的中部插入项。splice()方法始终都会返回一个数组，该数组中包含从原始数组中删除的项（如果没有删除任何项，则返回一个空数组）, 且会改变原数组**

1.  删除：可以删除任意数量的项，只需指定2个参数：要删除的第一项的位置和要删除的项数。例如，splice(0,2)会删除数组中的前两项。
2.  插入：可以向指定位置插入任意数量的项，只需提供3个参数：起始位置、0（要删除的项数）和要插入的项。如果要插入多个项，可以再传入第四、第五，以至任意多个项。例如，splice(2,0, "red", "green")会从当前数组的位置2开始插入字符串"red"和"green"
3.  替换：可以向指定位置插入任意数量的项，且同时删除任意数量的项，只需指定3个参数：起始位置、要删除的项数和要插入的任意数量的项。插入的项数不必与删除的项数相等。例如，splice (2,1, "red", "green")会删除当前数组位置2的项，然后再从位置2开始插入字符串"red"和"green"

```js
// 删除
let color = [1, 2, 3, 4];
let color1 = color.splice(0, 1); // color: [2, 3, 4] color1: [1]

// 插入
let color3 = [1,2,3,4]; 
let color4 = color3.splice(1,0,'a', 'b'); // color3:[ 1, 'a', 'b', 2, 3, 4 ]  color4: []


//替换
let color5 = [1,2,3,4]; 
let color6 = color5.splice(1,1,'a'); // color5:[ 1, 'a', 3, 4 ] color6: [2]
```


**迭代方法**

1.  every()：对数组中的每一项运行给定函数，如果该函数对每一项都返回true，则返回true。
2.  some()： 对数组中的每一项运行给定函数，如果该函数对任一项返回true，则返回true。 
3.  filter()：对数组中的每一项运行给定函数，返回该函数会返回true的项组成的数组。
4.  forEach()：对数组中的每一项运行给定函数。这个方法没有返回值
5.  map()：对数组中的每一项运行给定函数，返回每次函数调用的结果组成的数组。


```js
// filter
let num = [1, 2, 3, 4, 5];
num.filter((item. index, array)=>{
   return item > 2;
});
console.log(num) // [3, 4, 5]

// map
let num1 = [1, 2, 3, 4, 5];
num1.map((item. index, array)=>{
   return item *2;
});
console.log(num1) // [2, 4, 6, 8, 10]
```


**归并方法**

reduce()和reduceRight()。这两个方法都会迭代数组的所有项，然后构建一个最终返回的值。

```js
// 数组之和
let values = [1, 2, 3, 4, 5];
// 参数：前一个值、当前值、项的索引和数组对象
let sum = values.reduce((prev, cur, index, array)=>{
   console.log(prev, cur, index, array); // prev ==> 1 3 6 10
   return prev += cur;
});

console.log(sum) // 15
```

### 3、Date类型
ECMAScript中的Date类型是在早期Java中的java.util.Date类基础上构建的。为此，Date类型使用自UTC（Coordinated Universal Time，国际协调时间）1970年1月1日午夜（零时）开始经过的毫秒数来保存日期。在使用这种数据存储格式的条件下，Date类型保存的日期能够精确到1970年1月1日之前或之后的100000000年。

#### 3.1 Date.now()
ECMAScript 5添加了Date.now()方法，返回表示调用这个方法时的日期和时间的毫秒数。

```js
//取得开始时间
var start=Date.now();


//调用函数
doSomething();


//取得停止时间
var stop=Date.now(),
    result=stop - start;
```

支持Date.now()方法的浏览器包括IE9+、Firefox 3+、Safari 3+、Opera 10.5和Chrome。在不支持它的浏览器中，使用+操作符获取Date对象的时间戳，也可以达到同样的目的。

```js
//取得开始时间
var start=+new Date();


//调用函数
doSomething();
//取得停止时间
var stop=+new Date(),
    result=stop - start;
```

#### 3.2 继承
与其他引用类型一样，Date类型也重写了**toLocaleString()、toString()和valueOf()**方法；但这些方法返回的值与其他类型中的方法不同。Date类型的toLocaleString()方法会按照与浏览器设置的地区相适应的格式返回日期和时间。这大致意味着时间格式中会包含AM或PM，但不会包含时区信息（当然，具体的格式会因浏览器而异）。而toString()方法则通常返回带有时区信息的日期和时间，其中时间一般以军用时间（即小时的范围是0到23）表示。下面给出了在不同浏览器中调用toLocaleString()和toString()方法，输出PST（Pacific Standard Time，太平洋标准时间）时间2007年2月1日午夜零时的结果。

### 4、RegExp类型
#### 4.1 RegExp类型

```js
let expression = /pattern/flags;
```

其中的模式（pattern）部分可以是任何简单或复杂的正则表达式，可以包含字符类、限定符、分组、向前查找以及反向引用。每个正则表达式都可带有一或多个标志（flags），用以标明正则表达式的行为。正则表达式的匹配模式支持下列3个标志。

g：表示全局（global）模式，即模式将被应用于所有字符串，而非在发现第一个匹配项时立即停止；

i：表示不区分大小写（case-insensitive）模式，即在确定匹配项时忽略模式与字符串的大小写；

m：表示多行（multiline）模式，即在到达一行文本末尾时还会继续查找下一行中是否存在与模式匹配的项。


**与其他语言中的正则表达式类似，模式中使用的所有元字符都必须转义。正则表达式中的元字符包括：**

```js
( [ { \ ^ $ | ) ? * + . ] } 
```

#### 4.2 RegExp实例属性

global：布尔值，表示是否设置了g标志。

ignoreCase：布尔值，表示是否设置了i标志。

lastIndex：整数，表示开始搜索下一个匹配项的字符位置，从0算起。

multiline：布尔值，表示是否设置了m标志。

source：正则表达式的字符串表示，按照字面量形式而非传入构造函数中的字符串模式返回。

```js
let pattern1 = /\[bc\]at/i
console.log(pattern1.global); // false
console.log(pattern1.ignoreCase); // true


let pattern2 = new RegExp("\\[bc\\]at", "i");
console.log(pattern2.global); // false
console.log(pattern2.ignoreCase); // true
```

#### 4.3 RegExp实例方法

exec()

```js
// 待补充
```

test()

```js
let text = "000-00-0000";
let pattern = /\d{3}-\d{2}-\d{4}/;
pattern.test(text) // true
```


### 5、Function类型
函数实际上是对象。每个函数都是Function类型的实例，而且都与其他引用类型一样具有属性和方法。由于函数是对象，因此函数名实际上也是一个指向函数对象的指针，不会与某个函数绑定。

```js
// 创建函数方法一
function sum (num1, num2){
   return num1 + num2;
}

// 创建函数方法二
let sum = function (num1, num2) {
    return num1 + num2;  
}

/**
 *
 * 从技术角度讲，这是一个函数表达式。但是，我们不推荐读者使用这种方法定义函数，
 * 因为这种语法会导致解析两次代码（第一次是解析常规ECMAScript代码，第二次是解析传入构造函数中的字符串），
 * 从而影响性能。不过，这种语法对于理解“函数是对象，函数名是指针”的概念倒是非常直观的。
 * */

// 创建函数方法三，不推荐
let sum = new Function("num1", "num2", "return num1 + num2"); 
```

#### 5.1 没有重载
这个例子中声明了两个同名函数，而结果则是后面的函数覆盖了前面的函数。**在创建第二个函数时，实际上覆盖了引用第一个函数的变量addSomeNumber。**

```js
function addSomeNumber(num){
    return num+100;
}

function addSomeNumber(num) {
    return num+200;
}

var result=addSomeNumber(100); //300
```

#### 5.2 函数声明和函数表达式
解析器在向执行环境中加载数据时，对函数声明和函数表达式并非一视同仁。**解析器会率先读取函数声明，并使其在执行任何代码之前可用（可以访问）；至于函数表达式，则必须等到解析器执行到它所在的代码行，才会真正被解释执行。**

```js
/**
 * 因为在代码开始执行之前，解析器就已经通过一个名为函数声明提升
 * （function declaration hoisting）的过程，读取并将函数声明添加到执行环境中。
 */

alert(sum(10, 10));
function sum (num1, num2){
   return num1 + num2;
}
```

```js
/**
 * 在执行到函数所在的语句之前，变量sum中不会保存有对函数的引用；
 */ 
alert(sum(10, 10));
let sum = function (num1, num2){
   return num1 + num2;
}
```

#### 5.3 作为值的函数

#### 5.4 函数内部属性
在函数内部，有两个特殊的对象：arguments和this。其中，arguments在第3章曾经介绍过，它是一个类数组对象，包含着传入函数中的所有参数。虽然arguments的主要用途是保存函数参数，但这个对象还有一个名叫callee的属性，该属性是一个指针，指向拥有这个arguments对象的函数。请看下面这个非常经典的阶乘函数。

```js
function factorial(num){
  if(num <= 1){
   return 1;
  }else {
   return num * arguments.callee(num - 1);
  }
}
```

this的指向：由于在调用函数之前，this的值并不确定，因此this可能会在代码执行过程中引用不同的对象。

```js
window.color = 'red';
let o = {color: 'blue'};

function sayColor(){
   console.log(this.color);
}
sayColor(); // red


o.sayColor = sayColor;
o.sayColor(); // blue
```

#### 5.5 函数的属性和方法
ECMAScript中的函数是对象，因此函数也有属性和方法。每个函数都包含两个属性：length和prototype。其中，length属性表示函数希望接收的命名参数的个数。

**在ECMAScript核心所定义的全部属性中，最耐人寻味的就要数prototype属性了。对于ECMAScript中的引用类型而言，prototype是保存它们所有实例方法的真正所在。换句话说，诸如toString()和valueOf()等方法实际上都保存在prototype名下，只不过是通过各自对象的实例访问罢了。在创建自定义引用类型以及实现继承时，prototype属性的作用是极为重要的（第6章将详细介绍）。在ECMAScript 5中，prototype属性是不可枚举的，因此使用for-in无法发现。**


##### 5.1.1 call 和 apply

每个函数都包含两个非继承而来的方法：apply()和call()。这两个方法的用途都是在特定的作用域中调用函数，实际上等于设置函数体内this对象的值。

```js
// apply
function sum(num1, num2){
   return num1 + num2;
}

/**
 *  callSum1()在执行sum()函数时传入了this作为this值
   （因为是在全局作用域中调用的，所以传入的就是window对象）和arguments对象
 */ 
function callSum1(num1, num2){
   return sum.apply(this, arguments);
}

function callSum2(num1, num2){
   return sum.apply(this, [num1, num2]);
}

callSum1(10, 10);  // 20
callSum2(10, 10);  // 20


/**
 * 除了参数传递，最主要的是对象冒充（扩充函数作用域）
 */ 
window.color = 'red';
let o = {
   color: 'blue'
}

function sayColor(){
   console.log(this.color);
}

sayColor(); // red

sayColor.call(window) // red
sayColor.call(o) // blue
```

##### 5.1.2 bind
ECMAScript 5还定义了一个方法：bind()。这个方法会创建一个函数的实例，其this值会被绑定到传给bind()函数的值。

```js
window.color = 'red';
let o = {
   color: 'blue'
}
function sayColor(){
   console.log(this.color);
}
let a = sayColor.bind(o);
a() // blue
```

### 6、基本包装类型
为了便于操作基本类型值，ECMAScript还提供了3个特殊的引用类型：Boolean、Number和String。这些类型与本章介绍的其他引用类型相似，但同时也具有与各自的基本类型相应的特殊行为。实际上，每当读取一个基本类型值的时候，后台就会创建一个对应的基本包装类型的对象，从而让我们能够调用一些方法来操作这些数据。

```js
let s1 = 'some text';
let s2 = s1.subString(2);
```
我们知道，基本类型值不是对象，因而从逻辑上讲它们不应该有方法（尽管如我们所愿，它们确实有方法）。
其实，为了让我们实现这种直观的操作，后台已经自动完成了一系列的处理。**当第二行代码访问s1时，访问过程处于一种读取模式，也就是要从内存中读取这个字符串的值。而在读取模式中访问字符串时，后台都会自动完成下列处理：**

(1) 创建String类型的一个实例；

(2) 在实例上调用指定的方法；

(3) 销毁这个实例。

```js
var s1=new String("some text");
var s2=s1.substring(2);
s1=null;
```

**引用类型与基本包装类型的主要区别就是对象的生存期。使用new操作符创建的引用类型的实例，在执行流离开当前作用域之前都一直保存在内存中。而自动创建的基本包装类型的对象，则只存在于一行代码的执行瞬间，然后立即被销毁。这意味着我们不能在运行时为基本类型值添加属性和方法。**

```js
var s1="some text";
s1.color="red";
alert(s1.color);    //undefined
```

Object构造函数也会像工厂方法一样，根据传入值的类型返回相应基本包装类型的实例。

```js
var obj=new Object("some text");
alert(obj instanceof String);    //true
alert(obj instanceof Object);    //true
```

使用new调用基本包装类型的构造函数，与直接调用同名的转型函数是不一样的。 
在下面例子中，变量number中保存的是基本类型的值25，而变量obj中保存的是Number的实例。

```js
var value="25";
var number=Number(value);   //转型函数
alert(typeof number);          //"number"


var obj=new Number(value); //构造函数
alert(typeof obj);              //"object"
```
#### 6.1 Boolean类型

#### 6.2 Number类型
具体来讲，就是在使用typeof和instanceof操作符测试基本类型数值与引用类型数值时，得到的结果完全不同。

```js
var numberObject=new Number(10);
var numberValue=10;
alert(typeof numberObject);    //"object"
alert(typeof numberValue);     //"number"
alert(numberObject instanceof Number);   //true
alert(numberValue instanceof Number);    //false
```

#### 6.3 String类型

### 7、单体内置对象
由ECMAScript实现提供的、不依赖于宿主环境的对象，这些对象在ECMAScript程序执行之前就已经存在了。
意思就是说，开发人员不必显式地实例化内置对象，因为它们已经实例化了。前面我们已经介绍了大多数内置对象，例如Object、Array和String。ECMA-262还定义了两个单体内置对象：Global和Math。

#### 7.1 Global对象
所有在全局作用域中定义的属性和函数，都是Global对象的属性。本书前面介绍过的那些函数，诸如isNaN()、isFinite()、parseInt()以及parseFloat()，实际上全都是Global对象的方法。除此之外，Global对象还包含其他一些方法：

1. URI编码方法：encodeURI()和encodeURIComponent()

**encodeURI()不会对本身属于URI的特殊字符进行编码，例如冒号、正斜杠、问号和井字号；而encodeURIComponent()则会对它发现的任何非标准字符进行编码。**

**一般来说，我们使用encodeURIComponent()方法的时候要比使用encodeURI()更多，因为在实践中更常见的是对查询字符串参数而不是对基础URI进行编码。**

1. eval()方法

2. Global对象的属性
   
3. window对象
ECMAScript虽然没有指出如何直接访问Global对象，但Web浏览器都是将这个全局对象作为window对象的一部分加以实现的。因此，在全局作用域中声明的所有变量和函数，就都成为了window对象的属性

JavaScript中的window对象除了扮演ECMAScript规定的Global对象的角色外，还承担了很多别的任务。第8章在讨论浏览器对象模型时将详细介绍window对象.

#### 7.2 Math对象
1. min()和max()方法

```js
var values=[1, 2, 3, 4, 5, 6, 7, 8];
var max=Math.max.apply(Math, values);
```

2.  Math.ceil()执行向上舍入
3.  Math.floor()执行向下舍入
4.  Math.round()执行标准舍入


### 8、总结
**Object是一个基础类型，其他所有类型都从Object继承了基本的行为**

**函数实际上是Function类型的实例，因此函数也是对象；而这一点正是JavaScript最有特色的地方。由于函数是对象，所以函数也拥有方法，可以用来增强其行为。**

在所有代码执行之前，作用域中就已经存在两个内置对象：Global和Math。在大多数ECMAScript实现中都不能直接访问Global对象；不过，Web浏览器实现了承担该角色的window对象。



## 面向对象程序设计（重要知识点）
### 1、理解对象
早期的JavaScript开发人员经常使用这个模式创建新对象。几年后，对象字面量成为创建这种对象的首选模式。

```js
 let person = new Object();
 person.name = 'fc';
 person.sayName = function (){};

// 字面量
let person={
    name: "fc",
    sayName: function(){}
};
```

#### 1.1 属性类型

1. 数据属性
要修改属性默认的特性，必须使用ECMAScript 5的Object.defineProperty()方法。这个方法接收三个参数：属性所在的对象、属性的名字和一个描述符对象。

```js
let person = {};
Object.defineProperty(person, 'name', { 
    configurable: false, // 无法删除对象属性
    value: 'fc',
    writable: false, // 无法修改对象属性的值
    enumerable: true, // 表示能通过for-in循环返回属性
})
```

2. 访问器属性
访问器属性不包含数据值；它们包含一对儿getter和setter函数（不过，这两个函数都不是必需的）。在读取访问器属性时，会调用getter函数，这个函数负责返回有效的值；在写入访问器属性时，会调用setter函数并传入新值，这个函数负责决定如何处理数据。

不一定非要同时指定getter和setter。只指定getter意味着属性是不能写，尝试写入属性会被忽略。在严格模式下，尝试写入只指定了getter函数的属性会抛出错误。类似地，只指定setter函数的属性也不能读，否则在非严格模式下会返回undefined，而在严格模式下会抛出错误。


#### 1.2 Object.definePropert
```js
let person = {
   name: 'fc',
   say: 1
};
Object.defineProperty(person, 'year', { 
    get: function() {
       return this.name;
    },
    set: function(newValue) {
        if(newValue > 38){
          this.say = 2;
          this.name = 'fc1';
        }
    }
})
person.year = 39;
```

#### 1.3 Object.getOwnPropertyDescriptor
这个方法接收两个参数：属性所在的对象和要读取其描述符的属性名称。返回值是一个对象，如果是访问器属性，这个对象的属性有configurable、enumerable、get和set；如果是数据属性，这个对象的属性有configurable、enumerable、writable和value。



### 2、创建对象
虽然Object构造函数或对象字面量都可以用来创建单个对象，**但这些方式有个明显的缺点：使用同一个接口创建很多对象，会产生大量的重复代码。**为解决这个问题，人们开始使用工厂模式的一种变体。

#### 2.1 工厂模式

```js
function createPerson(name, age, job){
   let o = new Object();
   o.name = name;
   o.age = age;
   o.job = job;
   o.sayName = function (){
      return this.name;
   }
   return o;
}

let p1 = createPerson('f', 12, 'sf');
let p2 = createPerson('a', 13, 'sf');
```

可以无数次地调用这个函数，而每次它都会返回一个包含三个属性一个方法的对象。**工厂模式虽然解决了创建多个相似对象的问题，但却没有解决对象识别的问题（即怎样知道一个对象的类型）**


#### 2.2 构造函数模式

```js
// 按照惯例，构造函数始终都应该以一个大写字母开头
function Person(name, age, job){
   this.name = name;
   this.age = age;
   this.job = job;
   this.sayName = function (){
      console.log(this.name);
   }
}

let p1 = new Person('f', 12, 'sf');
let p2 = new Person('fc', 13, 'sf')
```

##### 2.2.1 构造函数模式和工厂模式区别

1. 没显示地创建对象
2. 直接将属性和方法赋给this对象
3. 没有return语句


**创建Person新实例，必须使用new操作符。调用构造函数会经历一下4个步骤：**

1. 创建一个新对象
2. 将构造函数的作用域赋给新对象（因此this就指向了这个新对象）
3. 执行构造函数的代码（为这个新对象添加属性）
4. 返回新对象

##### 2.2.2 constructor

```js
// p1和p2分别保存着Person的一个不同的实例。这两个对象都有一个constructor（构造函数）属性，该属性指向Person
p1.constructor == Person // true
p2.constructor == Person // true
```

##### 2.2.3 instanceof
```js
alert(p1 instanceof Object);   //true
alert(p2 instanceof Person);   //true
alert(p1 instanceof Object);   //true
alert(p2 instanceof Person);   //true
```

创建自定义的构造函数意味着将来可以将它的实例标识为一种特定的类型；而这正是构造函数模式胜过工厂模式的地方。在这个例子中，p1和p2之所以同时是Object的实例，是因为所有对象均继承自Object。


##### 2.2.4 将构造函数当作函数
构造函数与其他函数的唯一区别，就在于调用它们的方式不同。不过，构造函数毕竟也是函数，不存在定义构造函数的特殊语法。任何函数，只要通过new操作符来调用，那它就可以作为构造函数；而任何函数，如果不通过new操作符来调用，那它跟普通函数也不会有什么两样。

```js
// 当做构造函数使用
let p1 = new Person('f', 12, 'sf');
p1.sayName(); // f

// 当做普通函数使用
Person('Greg', 12, 'sf'); // 添加到window
window.sayName() // Greg

// 在另一个对象作用域调用(call/apply)
let o = {};
Person.call(o, 'frany', 24, 'sf');
o.sayName(); // frany
```

##### 2.2.5 构造函数的问题
1. 每个方法都要在每个实例上重新创建一遍。在前面的例子中，p1和p2都有一个名为sayName()方法。

```js
function Person(name, age, job){
   this.name=name;
   this.age=age;
   this.job=job;
   this.sayName=new Function("alert(this.name)"); // 与声明函数在逻辑上是等价的
}

console.log(p1.sayName == p2.sayName) // false, 不同实例上的同名函数是不相等的
```

```js
function Person(name, age, job){
   this.name=name;
   this.age=age;
   this.job=job;
   this.sayName=sayName;
}

function sayName(){
  this.name = name;
}

console.log(p1.sayName == p2.sayName) // true, 但是多了全局对象，没有封装性
```
#### 2.3 原型模式
我们创建的每个函数都有一个prototype（原型）属性，这个属性是一个指针，指向一个对象，而这个对象的用途是**包含可以由特定类型的所有实例共享的属性和方法。使用原型对象的好处是可以让所有对象实例共享它所包含的属性和方法。**

```js
function Person() {

}
Person.prototype.name = 'Greg';
Person.prototype.age = 24;
Person.prototype.job = 'software engineer';
Person.prototype.sayName = function (){
   console.log(this.name);
}

let p1 = new Person();
p1.sayName(); // Greg 

let p2 = new Person();
p2.sayName(); //Greg

// 但与构造函数模式不同的是，新对象的这些属性和方法是由所有实例共享的。
// 换句话说，p1和p2访问的都是同一组属性和同一个sayName()函数。
console.log(p1.sayName == p2.sayName) //true
```

##### 2.3.1 理解原型对象
在默认情况下，所有原型对象都会自动获得一个constructor（构造函数）属性，这个属性是一个指向prototype属性所在函数的指针。就拿前面的例子来说，Person.prototype.constructor指向Person。而通过这个构造函数，我们还可继续为原型对象添加其他属性和方法。

使用hasOwnProperty()方法可以检测一个属性是存在于实例中，还是存在于原型中。

```js
function Person() {

}
Person.prototype.name = 'Greg';
Person.prototype.age = 24;
Person.prototype.job = 'software engineer';
Person.prototype.sayName = function (){
   console.log(this.name);
}

let p1 = new Person();

console.log(p1.hasOwnProperty('name')); // false

p1.name = 'fr'; 
console.log(p1.hasOwnProperty('name')); // true

delete p1.name;
console.log(p1.name); // Greg
console.log(p1.hasOwnProperty('name')); // false
```

<img src="/img/prototype.jpeg" width="95%">


##### 2.3.2 原型与in操作符
有两种方式使用in操作符：单独使用和在for-in循环中使用。在单独使用时，in操作符会在通过对象能够访问给定属性时返回true，**无论该属性存在于实例中还是原型中。**

```js
// 判断属性是否在原型中
function hasPrototypeProperty(object, name) {
   return !(object.hasOwnProperty(name)) && (name in object);
}
```

要取得对象上**所有可枚举的实例属性**，可以使用ECMAScript 5的Object.keys()方法。

```js
function Person() {

}
Person.prototype.name = 'Greg';
Person.prototype.age = 24;
Person.prototype.job = 'software engineer';
Person.prototype.sayName = function (){
   console.log(this.name);
}

Object.keys(Person.prototype); // name, age, job, sayName

// 如果你想要得到所有实例属性，无论它是否可枚举，都可以使用Object.getOwnPropertyNames()方法。
Object.getOwnPropertyNames(Person.prototype); // constructor, name, age, job, sayName

let p1 = new Person(); 
p1.name = 'f';
p1.age = 12;

Object.keys(p1); // 只显示实例属性 name, age 
```
##### 2.3.3 更简单的原型语法

constructor属性不再指向Person了。前面曾经介绍过，每创建一个函数，就会同时创建它的prototype对象，这个对象也会自动获得constructor属性。而我们在这里使用的语法，**本质上完全重写了默认的prototype对象，因此constructor属性也就变成了新对象的constructor属性（指向Object构造函数），不再指向Person函数。** 此时，尽管instanceof操作符还能返回正确的结果，但通过constructor已经无法确定对象的类型了。

```js
function Person(name, age, job){
    this.name = name;
    this.age = age;
    this.job = job;
}
// 注意：重写了prototype对象
Person.prototype = {
   // constructor: Person,  重新设置constructor指向，但是有个问题？
   say: function(){
      console.log(`name:${this.name}; age:${this.age}; job: ${this.job}`);
   }
}

let p1 = new Person('fc', 12, 'SE');

p1 instanceof Person // true
p1 instanceof Object // true
p1.constructor == Person // false
p1.constructor == Object // true
```

以这种方式重设constructor属性会导致它的[[Enumerable]]特性被设置为true。默认情况下，原生的constructor属性是不可枚举的，因此如果你使用兼容ECMAScript 5的JavaScript引擎，可以试一试Object.defineProperty()。

```js
//重设构造函数，只适用于ECMAScript 5兼容的浏览器
Object.defineProperty(Person.prototype, "constructor", {
   enumerable: false,
   value: Person
});
```

##### 2.3.4 原型的动态性
由于在原型中查找值的过程是一次搜索，因此我们对原型对象所做的任何修改都能够立即从实例上反映出来——即使是先创建了实例后修改原型也照样如此。

原因可以归结为实例与原型之间的松散连接关系。当我们调用p1.say()时，首先会在实例中搜索名为say的属性，在没找到的情况下，会继续搜索原型。因为实例与原型之间的连接只不过是一个指针，而非一个副本，因此就可以在原型中找到新的say属性并返回保存在那里的函数。

```js
function Person(){}

let p1 = new Person();

Person.prototype.say = function(){
   console.log('hi');
}

p1.say(); // hi
```

如果重写原型对象呢？

```js
function Person(){

}

let p1 = new Person();

// 注意， 这里是重写了Person.prototype对象!!
Person.prototype = {
   constructor: Person,
   name: 'fc',
   age: 12,
   job: 'se',
   sayName: function(){
      console.log(this.name);
   }
}
/**
 * Error原因是重写了原型对象。 
 * 实例中的指针仅指向原型(Person.prototype)，
 * 而不在指向构造函数（Person）
 */
p1.sayName() // Error
```

调用构造函数时会为实例添加一个指向最初原型的[[Prototype]]指针，而把原型修改为另外一个对象就等于切断了构造函数与最初原型之间的联系。请记住：实例中的指针仅指向原型，而不指向构造函数。

<img src="/img/prototype1.jpeg" width="95%">

从上图可见，重写原型对象切断了现有原型与任何之前已经存在的对象实例之间的联系；它们引用的仍然是最初的原型。



##### 2.3.5 原生对象的原型
原型模式的重要性不仅体现在创建自定义类型方面，就连所有原生的引用类型，都是采用这种模式创建的。所有原生引用类型（Object、Array、String，等等）都在其构造函数的原型上定义了方法。例如，在Array.prototype中可以找到sort()方法，而在String.prototype中可以找到substring()方法。

通过原生对象的原型，不仅可以取得所有默认方法的引用，而且也可以定义新方法。可以像修改自定义对象的原型一样修改原生对象的原型，因此可以随时添加方法。

尽管可以这样做，但我们不推荐在产品化的程序中修改原生对象的原型。如果因某个实现中缺少某个方法，就在原生对象的原型中添加这个方法，那么当在另一个支持该方法的实现中运行代码时，就可能会导致命名冲突。而且，这样做也可能会意外地重写原生方法。

##### 2.3.6 原型对象的问题
原型模式的最大问题是由其共享的本性所导致的。

原型中所有属性是被很多实例共享的，这种共享对于函数非常合适。对于那些包含基本值的属性倒也说得过去，毕竟（如前面的例子所示），通过在实例上添加一个同名属性，可以隐藏原型中的对应属性。然而，对于包含引用类型值的属性来说，问题就比较突出了。

```js
function Person(){

}

Person.prototype = {
   constructor: Person,
   name: 'fc',
   age: 12,
   friends: ['Greg', 'simon'],
   sayName: function(){
      console.log(this.name);
   }
}

let p1 = new Person();
let p2 = new Person();

p1.friends.push('Franky');

p1.friends // ['Greg', 'simon', 'Franky']
p2.friends // ['Greg', 'simon', 'Franky']
p1.friends === p2.friends // true
```

**由于friends数组存在于Person.prototype而非p1中，所以刚刚提到的修改也会通过p2.friends（与p1.friends指向同一个数组）反映出来。**

### 3、继承
许多OO语言都支持两种继承方式：接口继承和实现继承。接口继承只继承方法签名，而实现继承则继承实际的方法。如前所述，由于函数没有签名，在ECMAScript中无法实现接口继承。**ECMAScript只支持实现继承，而且其实现继承主要是依靠原型链来实现的。**

#### 3.1 原型链继承
ECMAScript中描述了原型链的概念，并将原型链作为实现继承的主要方法。**其基本思想是利用原型让一个引用类型继承另一个引用类型的属性和方法。**

构造函数、原型、实例的关系：每个构造函数都有一个原型对象，原型对象都包含一个指向构造函数的指针，而实例都包含一个指向原型对象的内部指针

```js
// 原型链继承模型如下：

// 父类
function SuperType(){
   this.property = true;
}

SuperType.prototype.getSuperValue = function (){
   return this.property;
}

// 子类
function SubType(){
  this.subproperty = false;
}

/**
 * 继承是通过创建SuperType的实例,
 * 并将该实例赋给SubType.prototype实现的
 */
SubType.prototype = new SuperType(); 

SubType.prototype.getSubValue = function (){
   return this.subproperty;
}

// 实例
let instance = new SubType();
console.log(instance.getSuperValue());  // true  继承父类方法
console.log(instance.getSubValue());   // false
```

**原型链继承实现的本质是重写原型对象，代之以一个新类型的实例。**换句话说，原来存在于SuperType的实例中的所有属性和方法，现在也存在于SubType.prototype中了。

原型链继承实现的本质是重写原型对象，代之以一个新类型的实例。换句话说，原来存在于SuperType的实例中的所有属性和方法，现在也存在于SubType.prototype中了。

<img src="/img/prototype2.jpeg" width="95%" height="auto">

调用instance.getSuperValue()会经历三个搜索步骤：1）搜索实例；2）搜索SubType.prototype;3）搜索SuperType.prototype，最后一步才会找到该方法。在找不到属性或方法的情况下，搜索过程总是要一环一环地前行到原型链末端才会停下来。


##### 3.1.1 对象默认的原型
**我们知道，所有引用类型默认都继承了Object，而这个继承也是通过原型链实现的。**
所有函数的默认原型都是Object的实例，因此默认原型都会包含一个内部指针，指向Object.prototype。这也正是所有自定义类型都会继承toString()、valueOf()等默认方法的根本原因。

<img src="/img/prototype3.jpeg" width="95%" height="auto">

SubType继承了SuperType，而SuperType继承了Object。当调用instance.toString()时，实际上调用的是保存在Object.prototype中的那个方法

##### 3.1.2 原型和实例的关系
```js
// 1、instanceof
instance instanceof Object // true
instance instanceof SuperType // true
instance instanceof SubType // true


// 2、isPrototypeOf()
Object.prototype.isPrototypeOf(instance) // true
SuperType.prototype.isPrototypeOf(instance) // true
SubType.prototype.isPrototypeOf(instance) // true
```

##### 3.1.3 谨慎地定义方法
```js
// 1、覆盖父类的方法和属性
// todo


// 2、使用字面量穿件原型链，会重写原型链
function SuperType(){
   this.property = true;
}
SuperType.prototype.getSuperValue = function (){
   return this.property;
}


function SubType(){
  this.subproperty = false;
}
SubType.prototype = {
   getSubValue: function (){
      return this.subproperty;
   },
   otherMethod: function(){
      return false;
   }
}

let instance = new SubType();
console.log(instance.getSuperValue()); // error
```

以上代码展示了刚刚把SuperType的实例赋值给原型，紧接着又将原型替换成一个对象字面量而导致的问题。由于现在的原型包含的是一个Object的实例，而非SuperType的实例，因此我们设想中的原型链已经被切断——SubType和SuperType之间已经没有关系了。

##### 3.1.4 原型链的问题
问题一：原型中包含引用类型值（数组）的原型属性会被所有实例共享。想必大家还记得，我们前面介绍过包含引用类型值的原型属性会被所有实例共享；而这也正是为什么要在构造函数中，而不是在原型对象中定义属性的原因。在通过原型来实现继承时，原型实际上会变成另一个类型的实例。**于是，原先的实例属性也就顺理成章地变成了现在的原型属性了。** （这里需要仔细思考下）

问题二：在创建子类型的实例时，不能向超类型的构造函数中传递参数。实际上，应该说是没有办法在不影响所有对象实例的情况下，给超类型的构造函数传递参数。

```js
function SuperType(){
  this.colors = ['red', 'green'];
}

function SubType(){

}

// SubType原型属性 继承 SuperType实例属性 （这是重点)
SubType.prototype = new SuperType();

let instance1 = new SubType();
instance1.colors.push('blue');
console.log(instance1.colors); // ['red', 'green', 'blue']

let instance2 = new SubType();
console.log(instance2.colors); // ['red', 'green', 'blue']
```

#### 3.2 借用构造函数继承（call/apply）
**在解决原型中包含引用类型值所带来问题的过程中**，开发人员开始使用一种叫做借用构造函数的技术（有时候也叫做伪造对象或经典继承）。这种技术的基本思想相当简单，即在子类型构造函数的内部调用超类型构造函数。

```js
function SuperType(){
   this.colors = ['red', 'green', 'yellow'];
}

function SubType(){
   // SubType 继承 SuperType
   SuperType.call(this);
}

let instance1 = new SubType();
instance1.colors.push('black'); //['red', 'green', 'yellow', 'black']

let instance2 = new SubType();
instance1.colors // ['red', 'green', 'yellow']
```

我们实际上是在（未来将要）新创建的SubType实例的环境下调用了SuperType构造函数。这样一来，就会在新SubType对象上执行SuperType()函数中定义的所有对象初始化代码。结果，SubType的每个实例就都会具有自己的colors属性的副本了。

###### 3.2.1 好处（相对于原型支持传参）
```js
function SuperType(colors){
   this.colors = colors;
}

function SubType(){
   // SubType 继承 SuperType, 同时还传参
   SuperType.call(this, ['red', 'green']);

   // 实例属性
   this.name = 'fc';
}
```

###### 3.2.2 问题
如果仅仅是借用构造函数，那么也将无法避免构造函数模式存在的问题——方法都在构造函数中定义，因此函数复用就无从谈起了。而且，在超类型的原型中定义的方法，对子类型而言也是不可见的，结果所有类型都只能使用构造函数模式

#### 3.3 组合继承 (最常见)
组合继承，有时候也叫做伪经典继承，指的是将原型链和借用构造函数的技术组合到一块，从而发挥二者之长的一种继承模式。其背后的思路是使用原型链实现对原型属性和方法的继承，而通过借用构造函数来实现对实例属性的继承。这样，**既通过在原型上定义方法实现了函数复用，又能够保证每个实例都有它自己的属性。**

```js
function SuperType(name, colors){
   this.name = name;
   this.colors = colors;
}
SuperType.prototype.sayName = function (){
   console.log(this.name);
}

function SubType(name, colors, age){
   SuperType.call(this, name, colors);
   this.age = age;
}
SubType.prototype = new SuperType();
SubType.prototype.constructor = SubType;
SubType.prototype.sayAge = function () {
  console.log(this.age);
}

let instance1 = new SubType('f1', ['red', 'yellow'], 12);
instance1.colors.push('black');
console.log(instance1.colors); //  ['red', 'yellow', 'black']
instance1.sayName(); // f1
instance1.sayAge(); // 12


let instance2 = new SubType('f2', ['red', 'yellow'], 13);
console.log(instance2.colors);   // ['red', 'yellow']
instance2.sayName(); // f2
instance2.sayAge(); // 13
```

组合继承避免了原型链和借用构造函数的缺陷，融合了它们的优点，成为JavaScript中最常用的继承模式。而且，instanceof和isPrototypeOf()也能够用于识别基于组合继承创建的对象。

#### 3.4 原型式继承（es6实现Object.create()）

```js
function myObject(o){
   function F(){};
   F.prototype = o;
   return new F();
}

let person = {
   name: 'fc',
   friends: ['A', 'B', 'C']
}

// myObject(person) 可以用Object.create(person)代替
let aPerson = myObject(person);
aPerson.name = 'fr';
aPerson.friends.push('D');

let bPerson = myObject(person);
bPerson.name = 'fd';
bPerson.friends.push('E');

console.log(person.friends); // ['A', 'B', 'C', 'D', 'E']
```

在没有必要兴师动众地创建构造函数，而只想让一个对象与另一个对象保持类似的情况下，原型式继承是完全可以胜任的。**不过别忘了，包含引用类型值的属性始终都会共享相应的值，就像使用原型模式一样。**

#### 3.4 寄生式继承
待补充...

#### 3.4 寄生组合继承
前面说过，组合继承是JavaScript最常用的继承模式；不过，它也有自己的不足。组合继承最大的问题就是无论什么情况下，都会调用两次超类型构造函数：一次是在创建子类型原型的时候，另一次是在子类型构造函数内部。没错，子类型最终会包含超类型对象的全部实例属性，但我们不得不在调用子类型构造函数时重写这些属性。

```js
function SuperType(name){
   this.name = name;
   this.colors = ['red', 'green', 'yellow'];
}
SuperType.prototype.sayName = function (){
   console.log(this.name);
}


function SubType(name, age){
   SuperType.call(this, name); // 第一次调用构造函数
   this.age = age;
}
SubType.prototype = new SuperType(); // 第二次调用构造函数
SubType.prototype.constructor = SubType;
SubType.prototype.sayAge = function (){
   console.log(this.name);
}
```
寄生组合继承:

```js
function inheritPrototype(subType, superType){
   let prototype = Object.create(superType.prototype); // 创建对象
   prototype.constructor = subType;  // 增强对象
   subType.prototype = prototype;   // 制定对象
}

function SuperType(name){
   this.name = name;
   this.colors = ['red', 'green', 'yellow'];
}
SuperType.prototype.sayName = function (){
   console.log(this.name);
}

function subType(name, age){
   SuperType.call(this, name);
   this.age = age; 
}
inheritPrototype(subType, SuperType);
SubType.prototype.sayAge = function (){
   console.log(this.name);
}
```

这个例子的高效率体现在它只调用了一次SuperType构造函数，并且因此避免了在SubType.prototype上面创建不必要的、多余的属性。与此同时，原型链还能保持不变；因此，还能够正常使用instanceof和isPrototypeOf()。开发人员普遍认为寄生组合式继承是引用类型最理想的继承范式。

### 4、总结
ECMAScript支持面向对象（OO）编程，但不使用类或者接口。对象可以在代码执行过程中创建和增强，因此具有动态性而非严格定义的实体。

1. 工厂模式，使用简单的函数创建对象，为对象添加属性和方法，然后返回对象。这个模式后来被构造函数模式所取代。
2. 构造函数模式，可以创建自定义引用类型，可以像创建内置对象实例一样使用new操作符。不过，构造函数模式也有缺点，即它的每个成员都无法得到复用，包括函数。由于函数可以不局限于任何对象（即与对象具有松散耦合的特点），因此没有理由不在多个对象间共享函数。
3. 原型模式，使用构造函数的prototype属性来指定那些应该共享的属性和方法。组合使用构造函数模式和原型模式时，使用构造函数定义实例属性，而使用原型定义共享的属性和方法。

**JavaScript主要通过原型链实现继承。原型链的构建是通过将一个类型的实例赋值给另一个构造函数的原型实现的。这样，子类型就能够访问超类型的所有属性和方法，这一点与基于类的继承很相似。**

原型链的问题是对象实例共享所有继承的属性和方法，因此不适宜单独使用。解决这个问题的技术是借用构造函数，即在子类型构造函数的内部调用超类型构造函数。这样就可以做到每个实例都具有自己的属性，同时还能保证只使用构造函数模式来定义类型。

## 函数表达式
### 1、函数创建
定义函数的方式有两种：一种是函数声明，另一种就是函数表达式。

**关于函数声明，它的一个重要特征就是函数声明提升（function declaration hoisting），意思是在执行代码之前会先读取函数声明。**这就意味着可以把函数声明放在调用它的语句后面。

理解函数提升的关键，就是理解函数声明与函数表达式之间的区别。

```js
// 1、函数声明
function functionName(arg0, arg1, arg2) {
   //函数体
}

// 2、函数表达式
let functionName = function (){

}
```

### 2、递归

arguments.callee是一个指向正在执行的函数的指针

```js
function factorial(num){
   if(num <= 1){
      return 1;
   }else{
      return num * arguments.callee(num-1);
   }
}
```
但在严格模式下，不能通过脚本访问arguments.callee，访问这个属性会导致错误。

```js
var factorial=(function f(num){
   if (num <=1){
       return 1;
   } else {
       return num ＊ f(num-1);
   }
});
```

### 3、闭包
**有不少开发人员总是搞不清匿名函数和闭包这两个概念，因此经常混用。闭包是指有权访问另一个函数作用域中的变量的函数。创建闭包的常见方式，就是在一个函数内部创建另一个函数。**

当createComparisonFunction()函数返回后，其执行环境的作用域链会被销毁，但它的活动对象仍然会留在内存中；直到匿名函数被销毁后，createComparisonFunction()的活动对象才会被销毁。 如下：

```js
function createComparisonFunction(propertyName) {
   return function(object1, object2){
       var value1=object1[propertyName];
       var value2=object2[propertyName];
       if (value1 < value2){
           return -1;
       } else if (value1 > value2){
           return 1;
       } else {
           return 0;
       }
   };
}

var compare = createComparisonFunction("name");
var result = compare({ name: "Nicholas" }, { name: "Greg" });

compare = null; // 解除匿名函数引用，释放内存
```

由于闭包会携带包含它的函数的作用域，因此会比其他函数占用更多的内存。过度使用闭包可能会导致内存占用过多

#### 3.1 闭包与变量
作用域链的这种配置机制引出了一个值得注意的副作用，**即闭包只能取得包含函数中任何变量的最后一个值**

```js
function createFunctions(){
   let result = [];
   for (var i=0; i<10; i++){
      result[i] = function(){
         return i;
      } 
   }
   return result;
}

createFunctions()[0](); // 10
createFunctions()[9](); // 10
```
上面函数实际上，每个函数都返回10。因为每个函数的作用域链中都保存着createFunctions()函数的活动对象，所以它们引用的都是同一个变量i。当createFunctions()函数返回后，变量i的值是10，此时每个函数都引用着保存变量i的同一个变量对象，所以在每个函数内部i的值都是10

```js
function createFunctions(){
   let result = [];
   for (var i=0; i<10; i++){
      result[i] = function(num){
         return function(){
             return num;
         };
      }(i) 
   }
   return result;
}
```
在这个版本中，我们没有直接把闭包赋值给数组，而是定义了一个匿名函数，并将立即执行该匿名函数的结果赋给数组。这里的匿名函数有一个参数num，也就是最终的函数要返回的值。在调用每个匿名函数时，我们传入了变量i。**由于函数参数是按值传递的，所以就会将变量i的当前值复制给参数num。而在这个匿名函数内部，又创建并返回了一个访问num的闭包。这样一来，result数组中的每个函数都有自己num变量的一个副本，因此就可以返回各自不同的数值了**

#### 3.2 this对象
，this对象是在运行时基于函数的执行环境绑定的：在全局函数中，this等于window，而当函数被作为某个对象的方法调用时，this等于那个对象。不过，匿名函数的执行环境具有全局性，因此其this对象通常指向window（但call/apply指向其他对象）。但有时候由于编写闭包的方式不同，这一点可能不会那么明显。

```js
let name = 'this window';
let object = {
   name: 'my object',
   getNameFunc: function(){
      // let that = this; 解决方法一：保存副本
      return function() {
         return this.name;
      }
   }
}
object.getNameFunc()(); // this window  非严格模式下
```

#### 3.3 内存泄露
如果闭包的作用域链中保存着一个HTML元素，那么就意味着该元素将无法被销毁。

```js
function assignHandler(){
   var element=document.getElementById("someElement");
   element.onclick=function(){
       alert(element.id);
   };
}
```
以上代码创建了一个作为element元素事件处理程序的闭包，而这个闭包则又创建了一个循环引用。由于匿名函数保存了一个对assignHandler()的活动对象的引用，因此就会导致无法减少element的引用数。只要匿名函数存在，element的引用数至少也是1，因此它所占用的内存就永远不会被回收

```js
function assignHandler(){
   var element=document.getElementById("someElement");
   var id=element.id;
   element.onclick=function(){
       alert(id);
   };
   element=null;
}
```
把element.id的一个副本保存在一个变量中，并且在闭包中引用该变量消除了循环引用。但仅仅做到这一步，还是不能解决内存泄漏的问题。**必须要记住：闭包会引用包含函数的整个活动对象，而其中包含着element。即使闭包不直接引用element，包含函数的活动对象中也仍然会保存一个引用。因此，有必要把element变量设置为null**

### 4、模仿块级作用域
匿名函数可以用来模仿块级作用域并避免这个问题。

```js
/**
 * 下面这种做法可以减少闭包占用的内存问题，因为没有指向匿名函数的引用。
   只要函数执行完毕，就可以立即销毁其作用域链了。 
 */

(function(){
   //这里是块级作用域
})();
```

```js
function(){
   //这里是块级作用域
}();     //出错！
```
这段代码会导致语法错误，是因为JavaScript将function关键字当作一个函数声明的开始，**而函数声明后面不能跟圆括号。然而，函数表达式的后面可以跟圆括号。**要将函数声明转换成函数表达式，只要像下面这样给它加上一对圆括号即可。


### 5、私有变量
严格来讲，JavaScript中没有私有成员的概念；所有对象属性都是公有的。不过，倒是有一个私有变量的概念。任何在函数中定义的变量，都可以认为是私有变量，因为不能在函数的外部访问这些变量。**私有变量包括函数的参数、局部变量和在函数内部定义的其他函数。**

我们把有权访问私有变量和私有函数的公有方法称为特权方法（privileged method）

```js
function MyObject(){
    //私有变量和私有函数
    var privateVariable=10;

    function privateFunction(){
        return false;
    }


    //特权方法
    this.publicMethod=function (){
        privateVariable++;
        return privateFunction();
    };
}
```

#### 5.1 静态私有变量

初始化未经声明的变量，总是会创建一个全局变量。因此，MyObject就成了一个全局变量，能够在私有作用域之外被访问到。但也要知道，在严格模式下给未经声明的变量赋值会导致错误。

```js
(function(){
    //私有变量和私有函数
    var privateVariable=10;

    function privateFunction(){
        return false;
    }
    //构造函数 
    MyObject=function(){

    };


    //公有/特权方法
    MyObject.prototype.publicMethod=function(){
        privateVariable++;
        return privateFunction();
    };
})();

let a = new MyObject();
a.publicMethod();
```
#### 5.2 模块模式
前面的模式是用于为自定义类型创建私有变量和特权方法的。而道格拉斯所说的模块模式（module pattern）则是为单例创建私有变量和特权方法。所谓单例（singleton），指的就是只有一个实例的对象。

由于这个对象是在匿名函数内部定义的，因此它的公有方法有权访问私有变量和函数。从本质上来讲，这个对象字面量定义的是单例的公共接口。这种模式在需要对单例进行某些初始化，同时又需要维护其私有变量时是非常有用的。

```js
var singleton=function(){
    //私有变量和私有函数
    var privateVariable=10;
    function privateFunction(){
        return false;
    }
    
    //特权/公有方法和属性
      return {
          publicProperty: true,
          publicMethod : function(){
              privateVariable++;
              return privateFunction();
          }
      };
  }(); // 函数表达式后面支持带（）。 如果函数声明function a(){}()就报错 ---> (function a(){})()


// 使用
let applicaton = function(){
    let components = new Array();
    components.push(new BaseComponent());

    return {
      getComponents: function(){
         return components;
      },
      registerComponents: function(components){
          if(typeof components === 'object'){
             components.push(new BaseComponent());
          }
      }
    }
}()  
```

#### 5.3 增强的模块模式

有人进一步改进了模块模式，即在返回对象之前加入对其增强的代码。这种增强的模块模式适合那些单例必须是某种类型的实例，同时还必须添加某些属性和（或）方法对其加以增强的情况。

```js
var singleton=function(){

    //私有变量和私有函数
    var privateVariable=10;
    function privateFunction(){
        return false;
    }

    //创建对象
    var object=new CustomType();

    //添加特权/公有属性和方法
    object.publicProperty=true;
    object.publicMethod=function(){
        privateVariable++;
        return privateFunction();
    };

    //返回这个对象
    return object;
}();

// 使用
let applicaton = function(){
    let components = new Array();
    components.push(new BaseComponent());
    
    // 创建applicaton副本
    let app = new BaseComponent(); 

    app.getComponents=function(){
       return components;
    }
    app.registerComponents=function(components){
        if(typeof components === 'object'){
           components.push(new BaseComponent());
        }
    }

    return app;
}() 
```

### 6、总结
**匿名函数，也称为拉姆达函数，是一种使用JavaScript函数的强大方式。** 以下总结了函数表达式的特点：
1. 函数表达式不同于函数声明。函数声明要求有名字，但函数表达式不需要。没有名字的函数表达式也叫做匿名函数。
2. 递归函数应该始终使用arguments.callee来递归地调用自身，不要使用函数名——函数名可能会发生变化。

**当在函数内部定义了其他函数时，就创建了闭包。闭包有权访问包含函数内部的所有变量，**原理如下：
1. 在后台执行环境中，闭包的作用域链包含着它自己的作用域、包含函数的作用域和全局作用域。
2. 通常，函数的作用域及其所有变量都会在函数执行结束后被销毁。
3. 但是，当函数返回了一个闭包时，这个函数的作用域将会一直在内存中保存到闭包不存在为止。


**使用闭包可以在JavaScript中模仿块级作用域**（JavaScript本身没有块级作用域的概念），要点如下:
1. 创建并立即调用一个函数，这样既可以执行其中的代码，又不会在内存中留下对该函数的引用。
2. 结果就是函数内部的所有变量都会被立即销毁——除非将某些变量赋值给了包含作用域（即外部作用域）中的变量。

**因为创建闭包必须维护额外的作用域，所以过度使用它们可能会占用大量内存。**


## DOM
### 1、DOM节点和操作
#### 1.1 节点类型
每个节点都有一个nodeType属性，用于表明节点的类型。

```js
if (someNode.nodeType==1){     //适用于所有浏览器
   alert("Node is an element.");
}
```

#### 1.2 节点关系
每个节点都有一个childNodes属性，其中保存着一个NodeList对象。NodeList是一种类数组对象，用于保存一组有序的节点，可以通过位置来访问这些节点。

```js
var firstChild=someNode.childNodes[0];
var secondChild=someNode.childNodes.item(1);
var count=someNode.childNodes.length;
```

我们在本书前面介绍过，对arguments对象使用Array.prototype.slice()方法可以将其转换为数组。而采用同样的方法，也可以将NodeList对象转换为数组。

```js
//在IE8及之前版本中无效
var arrayOfNodes=Array.prototype.slice.call(someNode.childNodes,0);
```

DOM节点关系图：

<img src="/img/dom_node.jpeg" width="95%" height="auto">

```js
if(someNode.nextSibling === null){
   console.log('我是最后一个节点');
}else if(someNode.previousSibling === null){
   console.log('我是第一个节点');
}
```

#### 1.3 操作节点 

如果传入到appendChild()中的节点已经是文档的一部分了，那结果就是将该节点从原来的位置转移到新位置。

```js
var returnedNode=someNode.appendChild(newNode);
alert(returnedNode==newNode);           //true
alert(someNode.lastChild==newNode);   //true
```

#### 1.4 其他方法

### 2、DOM扩展

### 3、DOM2和DOM3

## 事件