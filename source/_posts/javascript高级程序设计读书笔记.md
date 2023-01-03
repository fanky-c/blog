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
1. constructor: 保存着用于创建当前对象函数， 对应当前的例子，构造函数（constructor）就是Object();
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
2.  filter()：对数组中的每一项运行给定函数，返回该函数会返回true的项组成的数组。
3.  forEach()：对数组中的每一项运行给定函数。这个方法没有返回值
4.  map()：对数组中的每一项运行给定函数，返回每次函数调用的结果组成的数组。
5.  some()：对数组中的每一项运行给定函数，如果该函数对任一项返回true，则返回true。 


**归并方法**

reduce()和reduceRight()。这两个方法都会迭代数组的所有项，然后构建一个最终返回的值。


### 3、Date类型

### 4、RegExp类型

### 5、Function类型

### 6、基本包装类型

## 面向对象程序设计

## 函数表达式

## DOM

## 事件