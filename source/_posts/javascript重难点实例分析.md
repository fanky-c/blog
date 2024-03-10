---
title: javascript重难点实例分析
date: 2023-10-30 20:09:46
tags:
  - js难点
  - js重点
---

## 1、javascript重点概念

### 1.1 javascript的基本数据类型介绍

**基本数据类型：Undefined、Null、Boolean、Number、String、Symbol**

**引用数据类型：Object、Function、Array、Date等类型**

#### 1.1.1 Undefined类型

Undefined类型只有一个唯一的字面值undefined，表示的是一个变量不存在。

下面是4种常见的出现undefined的场景：

1. 使用只声明而未初始化的变量时，会返回“undefined”。
2. 获取一个对象的某个不存在的属性（自身属性和原型链继承属性）时，会返回“undefined”。
3. 函数没有明确的返回值时，却在其他地方使用了返回值，会返回“undefined”。
4. 函数定义时使用了多个形式参数（后文简称为形参），而在调用时传递的参数的数量少于形参数量，那么未匹配上的参数就为“undefined”

#### 1.1.2 Null类型

Null类型只有一个唯一的字面值null，表示一个空指针对象，这也是在使用typeof运算符检测null值时会返回“object”的原因.

下面是3种常见的出现null的场景:

1. 一般情况下，如果声明的变量是为了以后保存某个值，则应该在声明时就将其赋值为“null”。
2. JavaScript在获取DOM元素时，如果没有获取到指定的元素对象，就会返回“null”。
3. 在使用正则表达式进行捕获时，如果没有捕获结果，就会返回“null”。

#### 1.1.3  Undefined和Null两种类型的异同

相同点：

1.  Undefined和Null两种数据类型都只有一个字面值，分别是undefined和null。
2. Undefined类型和Null类型在转换为Boolean类型的值时，都会转换为false。所以通过非运算符（！）获取结果为true的变量时，无法判断其值为undefined还是null。
3. 在需要将两者转换成对象时，都会抛出一个TypeError的异常，也就是平时最常见的引用异常。
4.  Undefined类型派生自Null类型，所以在非严格相等的情况下，两者是相等的，如下面代码所示。


不同点：

1. null是JavaScript中的关键字，而undefined是JavaScript中的一个全局变量，即挂载在window对象上的一个变量，并不是关键字。
2.  在使用typeof运算符检测时，Undefined类型的值会返回“undefined”，而Null类型的值会返回“object”。
3. 在通过call调用toString()函数时，Undefined类型的值会返回“[object Undefined]”，而Null类型的值会返回“[object Null]”。
4. 在需要进行数值类型的转换时，undefined会转换为NaN，无法参与计算；null会转换为0，可以参与计算。
5. 无论在什么情况下都没有必要将一个变量显式设置为undefined。如果需要定义某个变量来保存将来要使用的对象，应该将其初始化为null。这样不仅能将null作为空对象指针的惯例，还有助于区分null和undefined。

#### 1.1.4 Boolean类型

（1）String类型转换为Boolean类型

1.  空字符串""或者''都会转换为false。
2. 任何非空字符串都会转换为true，包括只有空格的字符串" "。

（2）Object类型转换为Boolean类型

1. 当object为null时，会转换为false。
2. 如果object不为null，则都会转换为true，包括空对象{}。

### 1.2 Number类型
在JavaScript中，Number类型的数据既包括了整型数据，也包括了浮点型数据。

① 八进制：如果想要用八进制表示一个数值，那么首位必须是0，其他位必须是0～7的八进制序列。如果后面位数的字面值大于7，则破坏了八进制数据表示规则，前面的0会被忽略，当作十进制数据处理。

```js
var num1 = 024; // 20  2*8+4*1
var num2 = 079; // 79  最后一位9超出了八进制字面值，所以不属于八进制数据，最终按照十进制处理，结果为79。
```

② 十六进制: 如果想要用十六进制表示一个数值，那么前两位必须是0x，其他位必须是十六进制序列（0～9，a～f或者A～F）。如果超过了十六进制序列，则会抛出异常。

```js
var num3 = 0x3f;  // 63  3×16+15
var num4 = 0x2g;  // SyntaxError: Invalid or unexpected token 最后一位g超出了十六进制所能表示的字面值区间，所以不满足十六进制数据表示规则
```

**Null类型转换为Number类型**

Null类型只有一个字面值null，直接转换为0。

**Undefined类型转换为Number类型**

Undefined类型只有一个字面值undefined，直接转换为NaN。

**Object类型转换为Number类型**

Object类型在转换为Number类型时，会优先调用valueOf()函数，然后通过valueOf()函数的返回值按照上述规则进行转换。如果转换的结果是NaN，则调用toString()函数，通过toString()函数的返回值重新按照上述规则进行转换；如果有确定的Number类型返回值，则结束，否则返回“NaN”。

#### 1.2.1  Number类型转换

##### 1.2.1.1 Number()函数

Number()函数可以用于将任何类型转换为Number类型，它在转换时遵循下列规则。

① 如果是数字，会按照对应的进制数据格式，统一转换为十进制并返回。

```js
Number(10);    // 10
Number(010);   // 8，010是八进制的数据，转换成十进制是8
Number(0x10);  // 16，0x10是十六进制数据，转换成十进制是16
```

② 如果是Boolean类型的值，true将返回为“1”，false将返回为“0”。

③ 如果值为null，则返回“0”。

```js
Number(null);  // 0
```

④ 如果值为undefined，则返回“NaN”。

```js
Number(undeﬁned); // NaN
```

⑤ 如果值为字符串类型，则遵循下列规则。

· 如果该字符串只包含数字，则会直接转换成十进制数；如果数字前面有0，则会直接忽略这个0。

· 如果字符串是有效的浮点数形式，则会直接转换成对应的浮点数，前置的多个重复的0会被清空，只保留一个。

· 如果字符串是有效的十六进制形式，则会转换为对应的十进制数值。

```js
Number('0x12'); // 18
Number('0x21'); // 33
```

· 如果字符串是有效的八进制形式，则不会按照八进制转换，而是直接按照十进制转换并输出，因为前置的0会被直接忽略。

```js
Number('010');   // 10
Number('0020');  // 20
```

⑥ 如果值为对象类型，则会先调用对象的valueOf()函数获取返回值，并将返回值按照上述步骤重新判断能否转换为Number类型。如果都不满足，则会调用对象的toString()函数获取返回值，并将返回值重新按照步骤判断能否转换成Number类型。如果也不满足，则返回“NaN”。

**以下是通过valueOf()函数将对象正确转换成Number类型的示例。**

```js
var obj = {
   age: 21,
   valueOf: function () {
      return this.age;
   },
   toString: function () {
      return 'good';
   }
};

Number(obj);  // 21
```

**以下是通过toString()函数将对象正确转换成Number类型的示例。**

```js
ar obj = {
   age: '21',
   valueOf: function () {
       return [];
   },
   toString: function () {
       return this.age;
   }
};

Number(obj);  // 21
```

**以下示例是通过valueOf()函数和toString()函数都无法将对象转换成Number类型的示例（最后返回“NaN”）。**

```js
var obj = {
   age: '21',
   valueOf: function () {
       return 'a';
   },
   toString: function () {
       return 'b';
   }
}

Number(obj);  // NaN
```

**如果toString()函数和valueOf()函数返回的都是对象类型而无法转换成基本数据类型，则会抛出类型转换的异常。**

```js
var obj = {
   age: '21',
   valueOf: function () {
       return [];
   },
   toString: function () {
       return [];
   }
};

Number(obj);  // 抛出异常TypeError: Cannot convert object to primitive value
```

##### 1.2.1.2 parseInt()函数

在使用parseInt()函数将字符串转换成整数时，需要注意以下5点。

* （1）非字符串类型转换为字符串类型

如果遇到传入的参数是非字符串类型的情况，则需要将其优先转换成字符串类型，即使传入的是整型数据。

```js
parseInt('0x12', 16);  // 18
parseInt(0x12, 16);    // 24
```
第一条语句直接将字符串"0x12"转换为十六进制数，得到的结果为1×16+2=18；

第二条语句由于传入的是十六进制数，所以会先转换成十进制数18，然后转换成字符串"18"，再将字符串"18"转换成十六进制数，得到的结果为1×16+8=24。


* （2）数据截取的前置匹配原则

parseInt()函数在做转换时，对于传入的字符串会采用前置匹配的原则。即从字符串的第一个字符开始匹配，如果处于基数指定的范围，则保留并继续往后匹配满足条件的字符，直到某个字符不满足基数指定的数据范围，则从该字符开始，舍弃后面的全部字符。在获取到满足条件的字符后，将这些字符转换为整数。

```js
parseInt("fg123", 16);  // 15
```

对于字符串'fg123'，首先从第一个字符开始，'f'是满足十六进制的数据，因为十六进制数据范围是0～9，a～f(A～F)，所以保留'f'；然后是第二个字符'g'，它不满足十六进制数据范围，因此从第二个字符至最后一个字符全部舍弃，最终字符串只保留字符'f'；然后将字符'f'转换成十六进制的数据，为15，因此最后返回的结果为“15”。

如果遇到的字符串是以"0x"开头的，那么在按照十六进制处理时，会计算后面满足条件的字符串；如果按照十进制处理，则会直接返回“0”。

```js
parseInt('0x12',16);   // 18 = 16 + 2
parseInt('0x12',10);   // 0
```

需要注意的一点是，如果传入的字符串中涉及算术运算，则不执行，算术符号会被当作字符处理；如果传入的参数是算术运算表达式，则会先运算完成得到结果，再参与parseInt()函数的计算。

```js
parseInt(15 * 3, 10);   // 45，先运算完成得到45，再进行parseInt(45, 10)的运算
parseInt('15 * 3', 10); // 15，直接当作字符串处理，并不会进行乘法运算
```

* （3）对包含字符e的不同数据的处理差异

```js
parseInt(6e3, 10);     // 6000
parseInt(6e3, 16);      // 24576
parseInt('6e3', 10);    // 6
parseInt('6e3', 16);     // 1763
```

第一条语句parseInt(6e3, 10)，首先会执行6e3=6000，然后转换为字符串"6000"，实际执行的语句是parseInt('6000', 10)，表示的是将字符串"6000"转换为十进制的整数，得到的结果为6000。

第二条语句parseInt(6e3, 16)，首先会执行6e3=6000，然后转换为字符串"6000"，实际执行的语句是parseInt('6000', 16)，表示的是将字符串"6000"转换为十六进制的数，得到的结果是6×163 = 24576。

第三条语句parseInt('6e3', 10)，表示的是将字符串'6e3'转换为十进制的整数，因为字符'e'不在十进制所能表达的范围内，所以会直接省略，实际处理的字符串只有"6"，得到的结果为6。

第四条语句parseInt('6e3', 16)，表示的是将字符串'6e3'转换为十六进制的整数，因为字符'e'在十六进制所能表达的范围内，所以会转换为14进行计算，最后得到的结果为6×162 +14×16 + 3 = 1763。

* （4）对浮点型数的处理

如果传入的值是浮点型数，则会忽略小数点及后面的数，直接取整。

```js
parseInt('6.01', 10); // 6
parseInt('6.99', 10); // 6
```

经过上面的详细分析，我们再来看看以下语句的执行结果。以下语句都会返回“15”，这是为什么呢？

```js
parseInt("0xF", 16);    // 十六进制的F为15，返回“15”
parseInt("F", 16);      // 十六进制的F为15，返回“15”
parseInt("17", 8);      // 八进制的"17"，返回结果为1×8 + 7 = 15
parseInt(021, 8);      // 021先转换成十进制得到17，然后转换成字符串"17"，再转换成八进制，返回结果为1×8 + 7 = 15

parseInt("015", 10);   // 前面的0忽略，返回“15”
parseInt(15.99, 10);   // 直接取整，返回“15”
parseInt("15,123", 10); // 字符串"15,123"一一匹配，得到"15"，转换成十进制后返回“15”
parseInt("FXX123", 16); // 字符串"FXX123"一一匹配，得到"F"，转换成十六进制后返回“15”
parseInt("1111", 2);    // 1×23 + 1×22 + 1×2 + 1 = 15
parseInt("15 * 3", 10); // 字符串中并不会进行算术运算，实际按照"15"进行计算，返回“15”
parseInt("15e2", 10);   // 实际按照字符串"15"运算，返回“15”
parseInt("15px", 10);   // 实际按照字符串"15"运算，返回“15”
parseInt("12", 13);     // 按照十三进制计算，返回结果为1×13 + 2 = 15
```

* （5）map()函数与parseInt()函数的隐形坑

设想这样一个场景，存在一个数组，数组中的每个元素都是Number类型的字符串['1','2', '3', '4']，如果我们想要将数组中的元素全部转换为整数，我们该怎么做呢？

我们可能会想到在Array的map()函数中调用parseInt()函数，代码如下。

```js
var arr = ['1', '2', '3', '4'];

var result = arr.map(parseInt);

console.log(result);
```
但是在运行后，得到的结果是[1, NaN, NaN, NaN]，与我们期望的结果[1, 2, 3, 4]差别很大，这是为什么呢？

上面的代码实际与下面的代码等效。

```js
arr.map(function (val, index) {
  return parseInt(val, index);
});

parseInt('1', 0);  // 1
parseInt('2', 1);  // NaN
parseInt('3', 2);  // NaN
parseInt('4', 3);  // NaN
```
任何整数以0为基数取整时，都会返回本身，所以第一行代码会返回“1”。

##### 1.2.1.3 parseFloat()函数

① 如果在解析过程中遇到了正负号（+ / -）、数字0～9、小数点或者科学计数法（e / E）以外的字符，则会忽略从该字符开始至结束的所有字符，然后返回当前已经解析的字符的浮点数形式。

```js
parseFloat('+1.2');   // 1.2
parseFloat('-1.2');   // -1.2
parseFloat('++1.2');  // NaN，符号不能连续出现
parseFloat('--1.2');  // NaN，符号不能连续出现
parseFloat('1+1.2');  // 1，'+'出现在第二位，不会当作符号位处理
```

② 字符串前面的空白符会直接忽略，如果第一个字符就无法解析，则会直接返回“NaN”。

```js
parseFloat('  1.2'); // 1.2
parseFloat('f1.2');  // NaN
```

③ 对于字符串中出现的合法科学运算符e，进行运算处理后会转换成浮点型数，这点与parseInt()函数的处理有很大的不同。

```js
parseFloat('4e3');   // 4000
parseInt('4e3', 10); // 4
```

④ 对于小数点，只能正确匹配第一个，第二个小数点是无效的，它后面的字符也都将被忽略。

```js
parseFloat('11.20');  // 11.2
parseFloat('11.2.1'); // 11.2
```

下面是使用parseFloat()函数的综合实例。

```js
parseFloat("123AF");   // 123，匹配字符串'123'
parseFloat("0xA");     // 0，匹配字符串'0'
parseFloat("22.5");    // 22.5，匹配字符串'22.5'
parseFloat("22.3.56"); // 22.3，匹配字符串'22.3'
parseFloat("0908.5");  // 908.5，匹配字符串'908.5'
```

##### 1.2.1.4  结论

· Number()函数转换的是传入的整个值，并不是像parseInt()函数和parseFloat()函数一样会从首位开始匹配符合条件的值。如果整个值不能被完整转换，则会返回“NaN”。

· parseFloat()函数在解析小数点时，会将第一个小数点当作有效字符，而parseInt()函数在解析时如果遇到小数点会直接停止，因为小数点不是整数的一部分。

· parseFloat()函数在解析时没有进制的概念，而parseInt()函数在解析时会依赖于传入的基数做数值转换。


#### 1.2.2 isNaN()函数与Number.isNaN()函数对比
Number类型数据中存在一个比较特殊的数值NaN（Not a Number），它表示应该返回数值却并未返回数值的情况。

NaN存在的目的是在某些异常情况下保证程序的正常执行。例如0/0，在其他语言中，程序会直接抛出异常，而在JavaScript中会返回“NaN”，程序可以正常执行。

**NaN有两个很明显的特点，第一个是任何涉及NaN的操作都会返回“NaN”，第二个是NaN与任何值都不相等，即使是与NaN本身相比。**

```js
NaN == NaN;  // false
```
在判断NaN时，ES5提供了isNaN()函数，ECMAScript 6（后续简称ES6）为Number类型增加了静态函数isNaN()。

##### 1.2.2.1 isNaN()函数

```js
isNaN(NaN);            // true
isNaN(undeﬁned);       // true
isNaN({});             // true

isNaN(true);           // false，Number(true)会转换成数字1
isNaN(null);           // false，Number(null)会转换成数字0
isNaN(1);              // false
isNaN('');             // false，Number('')会转换为成数字0
isNaN("1");            // false，字符串"1"可以转换成数字1
isNaN("JavaScript");   // true，字符串"JavaScript"无法转换成数字Date类型
isNaN(new Date());     // false
isNaN(new Date().toString());  // true
```

##### 1.2.2.2 Number.isNaN()函数
既然在全局环境中有isNaN()函数，为什么在ES6中会专门针对Number类型增加一个isNaN()函数呢？

这是因为isNaN()函数本身存在误导性，而ES6中的Number.isNaN()函数会在真正意义上去判断变量是否为NaN，不会做数据类型转换。只有在传入的值为NaN时，才会返回“true”，传入其他任何类型的值时会返回“false”。

```js
Number.isNaN(NaN);        // true
Number.isNaN(undeﬁned);   // false
Number.isNaN(null);       // false
Number.isNaN(true);       // false
Number.isNaN('');         // false
Number.isNaN(123);        // false
```

上面代码运行后，除了传入NaN会返回“true”以外，传入其他的值都会返回“false”。如果在非ES6环境中想用ES6中的isNaN()函数，该怎么办呢？我们有以下兼容性处理方案。

```js
// 兼容性处理
if(!Number.isNaN) {
    Number.isNaN = function (n) {
       return n !== n;
    }
}
```

##### 1.2.2.3 总结
· isNaN()函数在判断是否为NaN时，需要先进行数据类型转换，只有在无法转换为数字时才会返回“true”；

· Number.isNaN()函数在判断是否为NaN时，只需要判断传入的值是否为NaN，并不会进行数据类型转换。

#### 1.2.3 浮点型运算
**在JavaScript中，整数和浮点数都属于Number类型，它们都统一采用64位浮点数进行存储**

虽然它们存储数据的方式是一致的，但是在进行数值运算时，却会表现出明显的差异性。整数参与运算时，得到的结果往往会和我们所想的一样，而对于浮点型运算，有时却会出现一些意想不到的结果，如下面的代码所示。

```js
// 加法
0.1 + 0.2 = 0.30000000000000004
0.7 + 0.1 = 0.7999999999999999

// 减法
1.5 - 1.2 = 0.30000000000000004
0.3 - 0.2 = 0.09999999999999998

// 乘法
0.7 * 180 = 125.99999999999999
9.7 * 100 = 969.9999999999999

// 除法
0.3 / 0.1 = 2.9999999999999996
0.69 / 10 = 0.06899999999999999
```
得到这样的结果，大家是不是觉得很奇怪呢？0.1 + 0.2为什么不是等于0.3，而是等于0.30000000000000004呢？接下来我们一探究竟。

##### 1.2.3.1 浮点运算不准确原因
首先我们来看看一个浮点型数在计算机中的表示，它总共长度是64位，其中最高位为符号位，接下来的11位为指数位，最后的52位为小数位，即有效数字的部分。

· 第0位：符号位sign表示数的正负，0表示正数，1表示负数。

· 第1位到第11位：存储指数部分，用e表示。

· 第12位到第63位：存储小数部分（即有效数字），用f表示，如图1-1所示。
<br>
  <img src="/img/float.jpeg"  alt="浮点运算" height = "auto"/>
<br>
**因为浮点型数使用64位存储时，最多只能存储52位的小数位，对于一些存在无限循环的小数位浮点数，会截取前52位，从而丢失精度，所以会出现上面实例中的结果。**


##### 1.2.3.2 浮点运算计算过程
接下来以0.1 + 0.2 = 0.30000000000000004的运算为例，看看为什么会得到这个计算结果。

首先将各个浮点数的小数位按照“乘2取整，顺序排列”的方法转换成二进制表示。

具体做法是用2乘以十进制小数，得到积，将积的整数部分取出；然后再用2乘以余下的小数部分，又得到一个积；再将积的整数部分取出，如此推进，直到积中的小数部分为零为止。

然后把取出的整数部分按顺序排列起来，先取的整数作为二进制小数的高位有效位，后取的整数作为低位有效位，得到最终结果。

0.1转换为二进制表示的计算过程如下。

```js
0.1 * 2 = 0.2 //取出整数部分0

0.2 * 2 = 0.4 //取出整数部分0

0.4 * 2 = 0.8 //取出整数部分0

0.8 * 2 = 1.6 //取出整数部分1

0.6 * 2 = 1.2 //取出整数部分1

0.2 * 2 = 0.4 //取出整数部分0

0.4 * 2 = 0.8 //取出整数部分0

0.8 * 2 = 1.6 //取出整数部分1

0.6 * 2 = 1.2 //取出整数部分1
```

1.2取出整数部分1后，剩余小数为0.2，与这一轮运算的第一位相同，表示这将是一个无限循环的计算过程。

```js
0.2 * 2 = 0.4 //取出整数部分0

0.4 * 2 = 0.8 //取出整数部分0

0.8 * 2 = 1.6 //取出整数部分1

0.6 * 2 = 1.2 //取出整数部分1
...
```
因此0.1转换成二进制表示为0.0 0011 0011 0011 0011 0011 0011……（无限循环）。

同理对0.2进行二进制的转换，计算过程与上面类似，直接从0.2开始，相比于0.1，少了第一位的0，其余位数完全相同，结果为0.0011 0011 0011 0011 0011 0011……（无限循环）。

将0.1与0.2相加，然后转换成52位精度的浮点型表示。

```js
 0.0001 1001 1001 1001 1001 1001  1001 1001 1001 1001 1001 1001 1001   (0.1)
+ 0.0011 0011 0011 0011 0011 0011 0011 0011 0011 0011 0011 0011 0011   (0.2)
= 0.0100 1100 1100 1100 1100 1100 1100 1100 1100 1100 1100 1100 1100
```

##### 1.2.3.3  解决方案

这里提供一种方法，主要思路是将浮点数先乘以一定的数值转换为整数，通过整数进行运算，然后将结果除以相同的数值转换成浮点数后返回。

下面提供一套用于做浮点数加减乘除运算的代码。

```js
const operationObj = {
   /**
    * 处理传入的参数，不管传入的是数组还是以逗号分隔的参数都处理为数组
    * @param args
    * @returns {*}
    */
   getParam(args) {
      return Array.prototype.concat.apply([], args);
   },

   /**
    * 获取每个数的乘数因子，根据小数位数计算
    * 1.首先判断是否有小数点，如果没有，则返回1；
    * 2.有小数点时，将小数位数的长度作为Math.pow()函数的参数进行计算
    * 例如2的乘数因子为1，2.01的乘数因子为100
    * @param x
    * @returns {number}
    */
   multiplier(x) {
      let parts = x.toString().split('.');
      return parts.length < 2 ? 1 : Math.pow(10, parts[1].length);
   },

   /**
    * 获取多个数据中最大的乘数因子
    * 例如1.3的乘数因子为10，2.13的乘数因子为100
    * 则1.3和2.13的最大乘数因子为100
    * @returns {*}
    */
   correctionFactor() {
       let args = Array.prototype.slice.call(arguments);
       let argArr = this.getParam(args);
       return argArr.reduce((accum, next) => {
           let num = this.multiplier(next);
           return Math.max(accum, num);
       }, 1);
   },

   /**
    * 加法运算
    * @param args
    * @returns {number}
    */
   add(...args) {
       let calArr = this.getParam(args);
       // 获取参与运算值的最大乘数因子
       let corrFactor = this.correctionFactor(calArr);
       let sum = calArr.reduce((accum, curr) => {
           // 将浮点数乘以最大乘数因子，转换为整数参与运算
           return accum + Math.round(curr * corrFactor);
       }, 0);
       // 除以最大乘数因子
       return sum / corrFactor;
   },

   /**
    * 减法运算
    * @param args
    * @returns {number}
    */
   subtract(...args) {
       let calArr = this.getParam(args);
       let corrFactor = this.correctionFactor(calArr);
       let diﬀ = calArr.reduce((accum, curr, curIndex) => {
          // reduce()函数在未传入初始值时，curIndex从1开始，第一位参与运算的值需要
          // 乘以最大乘数因子
          if (curIndex === 1) {
              return Math.round(accum * corrFactor) - Math.round(curr * corrFactor);
          }
          // accum作为上一次运算的结果，就无须再乘以最大因子
          return Math.round(accum) - Math.round(curr * corrFactor);
       });
     // 除以最大乘数因子
       return diﬀ / corrFactor;
   },

   /**
    * 乘法运算
    * @param args
    * @returns {*}
    */
   multiply(...args) {
      let calArr = this.getParam(args);
      let corrFactor = this.correctionFactor(calArr);
      calArr = calArr.map((item) => {
          // 乘以最大乘数因子
          return item * corrFactor;
      });
      let multi = calArr.reduce((accum, curr) => {
          return Math.round(accum) * Math.round(curr);
      }, 1);
      // 除以最大乘数因子
      return multi / Math.pow(corrFactor, calArr.length);
   },

   /**
    * 除法运算
    * @param args
    * @returns {*}
    */
   divide(...args) {
       let calArr = this.getParam(args);
       let quotient = calArr.reduce((accum, curr) => {
           let corrFactor = this.correctionFactor(accum, curr);
           // 同时转换为整数参与运算
           return Math.round(accum * corrFactor) / Math.round(curr * corrFactor);
       });
       return quotient;
   }
};
```

### 1.3 String类型
JavaScript中的String类型（字符串类型）既可以通过双引号""表示，也可以通过单引号''表示，而且是完全等效的，这点与Java、PHP等语言在字符串的处理上是不同的。

**如果是引用类型的数据，则在转换时总是会调用toString()函数，得到不同类型值的字符串表示；如果是基本数据类型，则会直接将字面值转换为字符串表示形式。例如null值和undefined值转换为字符串时，会直接返回字面值，分别是"null"和"undefined"。**

#### 1.3.1 String类型的定义与调用

在JavaScript中，有3种定义字符串的方式，分别是字符串字面量，直接调用String()函数与new String()构造函数。

1. 字符串字面量

```js
var str = 'hello JavaScript';  // 正确写法
var str2 = "hello html";       // 正确写法
var str = 'hello css";         // 错误写法，首尾符号不一样
```

2.  直接调用String()函数

① 如果是Number类型的值，则直接转换成对应的字符串。

```js
String(123);     // '123'
String(123.12);  // '123.12'
```

② 如果是Boolean类型的值，则直接转换成'true'或者'false'。

```js
String(true);  // 'true'
String(false); // 'false'
```

③ 如果值为null，则返回字符串'null'；

```js
String(null);  // 'null'
```

④ 如果值为undefined，则返回字符串'undefined'；

```js
String(undeﬁned); // 'undeﬁned'
```

⑤ 如果值为字符串，则直接返回字符串本身；

```js
String('this is a string');  // 'this is a string'
```

⑥ 如果值为引用类型，则会先调用toString()函数获取返回值，将返回值按照上述步骤①～⑤判断能否转换字符串类型，如果都不满足，则会调用对象的valueOf()函数获取返回值，并将返回值重新按照步骤①～⑤判断能否转换成字符串类型，如果也不满足，则会抛出类型转换的异常。

以下是通过toString()函数将对象正确转换成String类型的示例:

```js
var obj = {
   age: 21,
   valueOf: function () {
       return this.age;
   },
   toString: function () {
       return 'good';
   }
};
String(obj);  // 'good'
```

以下是通过valueOf()函数将对象正确转换成String类型的示例:

```js
var obj = {
   age: '21',
   valueOf: function () {
       return this.age;
   },
   toString: function () {
       return [];
   }
};
String(obj);  // '21'
```

如果toString()函数和valueOf()函数返回的都是对象类型而无法转换成原生类型时，则会抛出类型转换的异常:

```js
var obj = {
   age: '21',
   valueOf: function () {
       return [];
   },
   toString: function () {
       return [];
   }
};
String(obj);  // 抛出异常TypeError: Cannot convert object to primitive value
```

3. new String()构造函数

new String()构造函数使用new运算符生成String类型的实例，对于传入的参数同样采用和上述String()函数一样的类型转换策略，最后的返回值是一个String类型对象的实例。

```js
new String('hello JavaScript'); // String {"hello JavaScript"}
```

4. 三者在作比较时的区别

基本字符串在作比较时，只需要比较字符串的值即可；而在比较字符串对象时，比较的是对象所在的地址。

```js
var str = 'hello';
var str2 = String(str);
var str3 = String('hello');
var str4 = new String(str);
var str5 = new String(str);
var str6 = new String('hello');

str === str2;   // true
str2 === str3;  // true
str3 === str4;  // false
str4 === str5;  // false
str5 === str6;  // false
```

对于str4、str5和str6，因为是使用new运算符生成的String类型的实例，所以在比较时需要判断变量是否指向同一个对象，即内存地址是否相同，很明显str4、str5、str6都是在内存中新生成的地址，彼此各不相同。


5. 函数的调用

在String对象的原型链上有一系列的函数，例如indexOf()函数、substring()函数、slice()函数等，通过String对象的实例可以调用这些函数做字符串的处理。

但是我们发现，采用字面量方式定义的字符串没有通过new运算符生成String对象的实例也能够直接调用原型链上的函数。

这是为什么呢？

**实际上基本字符串本身是没有字符串对象的函数，而在基本字符串调用字符串对象才有的函数时，JavaScript会自动将基本字符串转换为字符串对象，形成一种包装类型，这样基本字符串就可以正常调用字符串对象的方法了。**


#### 1.3.2 String类型常见算法

##### 1.3.2.1  字符串逆序输出

给定一个字符串'abcdefg'，执行一定的算法后，输出的结果为'gfedcba'。

1. 算法1

算法1的主要思想是借助数组的reverse()函数。

首先将字符串转换为字符数组，然后通过调用数组原生的reverse()函数进行逆序，得到逆序数组后再通过调用join()函数得到逆序字符串。

```js
// 算法1：借助数组的reverse()函数
function reverseString1(str) {
   return str.split('').reverse().join('');
}
```

2. 算法2

算法2的主要思想是利用字符串本身的charAt()函数。

从尾部开始遍历字符串，然后利用charAt()函数获取字符并逐个拼接，得到最终的结果。charAt()函数接收一个索引数字，返回该索引位置对应的字符。

```js
// 算法2：利用charAt()函数
function reverseString2(str) {
   var result = '';
   for(var i = str.length - 1; i >= 0; i--){
       result += str.charAt(i);
   }
   return result;
}
```

3. 算法3

算法3的主要思想是通过递归实现逆序输出，与算法2的处理类似。

递归从字符串最后一个位置索引开始，通过charAt()函数获取一个字符，并拼接到结果字符串中，递归结束的条件是位置索引小于0。

```js
// 算法3：递归实现
function reverseString3(strIn,pos,strOut){
   if(pos<0)
      return strOut;
   strOut += strIn.charAt(pos--);
   return reverseString3(strIn,pos,strOut);
}


var str = 'abcdefg';
var result = '';
console.log(reverseString3(str, str.length - 1, result));
```

4. 算法4

算法4的主要思想是通过call()函数来改变slice()函数的执行主体。

调用call()函数后，可以让字符串具有数组的特性，在调用未传入参数的slice()函数后，得到的是一个与自身相等的数组，从而可以直接调用reverse()函数，最后再通过调用join()函数，得到逆序字符串。

```js
// 算法4: 利用call()函数
function reverseString4(str) {
   // 改变slice()函数的执行主体，得到一个数组
   var arr = Array.prototype.slice.call(str);
   // 调用reverse()函数逆序数组
   return arr.reverse().join('');
}
```

5. 算法5

算法5的主要思想是借助栈的先进后出原则

由于JavaScript并未提供栈的实现，我们首先需要实现一个栈的数据结构，然后在栈中添加插入和弹出的函数，利用插入和弹出方法的函数字符串逆序。

首先，我们来看下基本数据结构——栈的实现。通过一个数组进行数据存储，通过一个top变量记录栈顶的位置，随着数据的插入和弹出，栈顶位置动态变化。

栈的操作包括两种，分别是出栈和入栈。出栈时，返回栈顶元素，即数组中索引值最大的元素，然后top变量减1；入栈时，往栈顶追加元素，然后top变量加1。

```js
// 栈
function Stack() {
   this.data = []; // 保存栈内元素
   this.top = 0;   // 记录栈顶位置
}

// 原型链增加出栈、入栈方法
Stack.prototype = {
   // 入栈:先在栈顶添加元素，然后元素个数加1
   push: function push(element) {
       this.data[this.top++] = element;
   },
   // 出栈：先返回栈顶元素，然后元素个数减1
   pop: function pop() {
       return this.data[--this.top];
   },
   // 返回栈内的元素个数，即长度
   length: function () {
       return this.top;
   }
};
```

```js
// 算法5：自定义栈实现
function reverseString5(str) {
   //创建一个栈的实例
   var s = new Stack();
   //将字符串转成数组
   var arr = str.split('');
   var len = arr.length;
   var result = '';
   //将元素压入栈内
   for(var i = 0; i < len; i++){
      s.push(arr[i]);
   }
   //输出栈内元素
   for(var j = 0; j < len; j++){
      result += s.pop(j);
   }
   return result;
}

// 使用
var str = 'abcdefg';
console.log(reverseString5(str));
```

##### 1.3.2.2 统计字符串中出现次数最多的字符及出现的次数

假如存在一个字符串'helloJavascripthellohtmlhellocss'，其中出现次数最多的字符是l，出现的次数是7次。

1. 算法1

算法1的主要思想是通过key-value形式的对象来存储字符串以及字符串出现的次数，然后逐个判断出现次数最大值，同时获取对应的字符，具体实现如下。

· 首先通过key-value形式的对象来存储数据，key表示不重复出现的字符，value表示该字符出现的次数。

· 然后遍历字符串的每个字符，判断是否出现在key中。如果在，直接将对应的value值加1；如果不在，则直接新增一组key-value，value值为1

· 得到key-value对象后，遍历该对象，逐个比较value值的大小，找出其中最大的值并记录key-value，即获得最终想要的结果。

```js
// 算法1
function getMaxCount(str) {
   var json = {};
   // 遍历str的每一个字符得到key-value形式的对象
   for (var i = 0; i < str.length; i++) {
       // 判断json中是否有当前str的值
       if (!json[str.charAt(i)]) {
           // 如果不存在，就将当前值添加到json中去
           json[str.charAt(i)] = 1;
       } else {
           // 如果存在，则让value值加1
           json[str.charAt(i)]++;
       }
   }
   // 存储出现次数最多的值和出现次数
   var maxCountChar = '';
   var maxCount = 0;
   // 遍历json对象，找出出现次数最大的值
  for (var key in json) {
      // 如果当前项大于下一项
      if (json[key] > maxCount) {
          // 就让当前值更改为出现最多次数的值
          maxCount = json[key];
          maxCountChar = key;
      }
   }
   //最终返回出现最多的值以及出现次数
   return '出现最多的值是' + maxCountChar + '，出现次数为' + maxCount;
}
var str = 'helloJavaScripthellohtmlhellocss';
getMaxCount(str); // '出现最多的值是l，出现次数为7'
```

2. 算法2

算法2同样会借助于key-value形式的对象来存储字符与字符出现的次数，但是在运算上有所差别。

· 首先通过key-value形式的对象来存储数据，key表示不重复出现的字符，value表示该字符出现的次数。

· 然后将字符串处理成数组，通过forEach()函数遍历每个字符。在处理之前需要先判断当前处理的字符是否已经在key-value对象中，如果已经存在则表示已经处理过相同的字符，则无须处理；如果不存在，则会处理该字符item。

· 通过split()函数传入待处理字符，可以得到一个数组，该数组长度减1即为该字符出现的次数。

· 获取字符出现的次数后，立即与表示出现最大次数和最大次数对应的字符变量maxCount和maxCountChar相比，如果比maxCount大，则将值写入key-value对象中，并动态更新maxCount和maxCountChar的值，直到最后一个字符处理完成。

· 最后得到的结果即maxCount和maxCountChar两个值。

```js
// 算法2
function getMaxCount2(str) {
   var json = {};
   var maxCount = 0, maxCountChar = '';
   str.split('').forEach(function (item) {
       // 判断json对象中是否有对应的key
       if (!json.hasOwnProperty(item)) {
           // 当前字符出现的次数
           var number = str.split(item).length - 1;
           // 直接与出现次数最大值比较，并进行更新
           if(number > maxCount) {
               // 写入json对象
               json[item] = number;
               // 更新maxCount与maxCountChar的值
               maxCount = number;
               maxCountChar = item;
           }
       }
   });

   return '出现最多的值是' + maxCountChar + '，出现次数为' + maxCount;
}

var str = 'helloJavaScripthellohtmlhellocss';
getMaxCount2(str); // '出现最多的值是l，出现次数为7'
```

3. 算法3

算法3的主要思想是对字符串进行排序，然后通过lastIndexOf()函数获取索引值后，判断索引值的大小以获取出现的最大次数。

· 首先将字符串处理成数组，调用sort()函数进行排序，处理成字符串。

· 然后遍历每个字符，通过调用lastIndexOf()函数，确定每个字符出现的最后位置，然后减去当前遍历的索引，就可以确定该字符出现的次数。

· 确定字符出现的次数后，直接与次数最大值变量maxCount进行比较，如果比maxCount大，则直接更新maxCount的值，并同步更新maxCountChar的值；如果比maxCount小，则不做任何处理。

· 计算完成后，将索引值设置为字符串出现的最后位置，进行下一轮计算，直到处理完所有字符。

```js
// 算法3
function getMaxCount3(str) {
   // 定义两个变量，分别表示出现最大次数和对应的字符
   var maxCount = 0, maxCountChar = '';
   // 先处理成数组，调用sort()函数排序,再处理成字符串
   str = str.split('').sort().join('');
   for (var i = 0, j = str.length; i < j; i++) {
       var char = str[i];
       // 计算每个字符串出现的次数
       var charCount = str.lastIndexOf(char) - i + 1;
       // 与次数最大值作比较
       if (charCount > maxCount) {
           // 更新maxCount和maxCountChar的值
           maxCount = charCount;
           maxCountChar = char;
       }
       // 变更索引为字符出现的最后位置
       i = str.lastIndexOf(char);
   }
   return '出现最多的值是' + maxCountChar + '，出现次数为' + maxCount;
}

var str = 'helloJavaScripthellohtmlhellocss';
getMaxCount3(str);  // '出现最多的值是l，出现次数为7'
```

4. 算法4

算法4的主要思想是将字符串进行排序，然后通过正则表达式将字符串进行匹配拆分，将相同字符组合在一起，最后判断字符出现的次数。

· 首先将字符串处理成数组，调用sort()函数进行排序，处理成字符串。

· 然后设置正则表达式reg，对字符串使用match()函数进行匹配，得到一个数组，数组中的每个成员是相同的字符构成的字符串。

· 遍历数组，依次将成员字符串长度值与maxCount值进行比较，动态更新maxCount与maxCountChar的值，直到数组所有元素处理完成。

```js
// 算法4
function getMaxCount4(str) {
   // 定义两个变量，分别表示出现最大次数和对应的字符
   var maxCount = 0, maxCountChar = '';
   // 先处理成数组，调用sort()函数排序,再处理成字符串
   str = str.split('').sort().join('');
   // 通过正则表达式将字符串处理成数组(数组每个元素为相同字符构成的字符串)
   var arr = str.match(/(\w)\1+/g);
   for (var i = 0; i < arr.length; i++) {
       // length表示字符串出现的次数
       var length = arr[i].length;
       // 与次数最大值作比较
       if (length > maxCount) {
           // 更新maxCount和maxCountChar
           maxCount = length;
           maxCountChar = arr[i][0];
       }
   }
   return '出现最多的值是' + maxCountChar + '，出现次数为' + maxCount;
}

var str = 'helloJavaScripthellohtmlhellocss';
getMaxCount4(str);  // '出现最多的值是l，出现次数为7'
```

5. 算法5

算法5的主要思想是借助replace()函数，主要实现方式如下。

· 通过while循环处理，跳出while循环的条件是字符串长度为0。

· 在while循环中，记录原始字符串的长度originCount，用于后面做长度计算处理。

· 获取字符串第一个字符char，通过replace()函数将char替换为空字符串''，得到一个新的字符串，它的长度remainCount相比于originCount会小，其中的差值originCount - remainCount即为该字符出现的次数。

· 确定字符出现的次数后，直接与maxCount进行比较，如果比maxCount大，则直接更新maxCount的值，并同步更新maxCountChar的值；如果比maxCount小，则不做任何处理。

· 处理至跳出while循环，得到最终结果。

```js
// 算法5
function getMaxCount5(str) {
   // 定义两个变量，分别表示出现最大次数和对应的字符
   var maxCount = 0, maxCountChar = '';
   while (str) {
       // 记录原始字符串的长度
       var originCount = str.length;
       // 当前处理的字符
       var char = str[0];
       var reg = new RegExp(char, 'g');
       // 使用replace()函数替换处理的字符为空字符串
       str = str.replace(reg, '');
       var remainCount = str.length;
       // 当前字符出现的次数
       var charCount = originCount - remainCount;
       // 与次数最大值作比较
       if (charCount > maxCount) {
          // 更新maxCount和maxCountChar的值
          maxCount = charCount;
          maxCountChar = char;
       }
   }
   return '出现最多的值是' + maxCountChar + '，出现次数为' + maxCount;
}

var str = 'helloJavaScripthellohtmlhellocss';
getMaxCount5(str);  // '出现最多的值是l，出现次数为7'
```


##### 1.3.2.3 去除字符串中重复的字符

假如存在一个字符串'helloJavaScripthellohtmlhellocss'，其中存在大量的重复字符，例如h、e、l等，去除重复的字符，只保留一个，得到的结果应该是'heloJavscriptm'。

1. 算法1

算法1的主要思想是使用key-value类型的对象存储，key表示唯一的字符，处理完后将所有的key拼接在一起即可得到去重后的结果。

· 首先通过key-value形式的对象来存储数据，key表示不重复出现的字符，value为boolean类型的值，为true则表示字符出现过。

· 然后遍历字符串，判断当前处理的字符是否在对象中，如果在，则不处理；如果不在，则将该字符添加到结果数组中。

· 处理完字符串后，得到一个数组，转换为字符串后即可获得最终需要的结果。

```js
// 算法1
function removeDuplicateChar1(str) {
   // 结果数组
   var result = [];
   // key-value形式的对象
   var json = {};
   for (var i = 0; i < str.length; i++) {
       // 当前处理的字符
       var char = str[i];
       // 判断是否在对象中
       if(!json[char]) {
           // value值设置为false
           json[char] = true;
           // 添加至结果数组中
           result.push(char);
       }
   }
   return result.join('');
}

var str = 'helloJavaScripthellohtmlhellocss';
removeDuplicateChar1(str);  // 'heloJavscriptm'
```

2. 算法2

算法2的主要思想是借助数组的filter()函数，然后在filter()函数中使用indexOf()函数判断。

· 通过call()函数改变filter()函数的执行体，让字符串可以直接执行filter()函数。

· 在自定义的filter()函数回调中，通过indexOf()函数判断其第一次出现的索引位置，如果与filter()函数中的index一样，则表示第一次出现，符合条件则return出去。这就表示只有第一次出现的字符会被成功过滤出来，而其他重复出现的字符会被忽略掉。

· filter()函数返回的结果便是已经去重的字符数组，将其转换为字符串输出即为最终需要的结果。

```js
// 算法2
function removeDuplicateChar2(str) {
   // 使用call()函数改变ﬁlter函数的执行主体
   let result = Array.prototype.ﬁlter.call(str, function (char, index, arr) {
      // 通过indexOf()函数与index的比较，判断是否是第一次出现的字符
      return arr.indexOf(char) === index;
   });
   return result.join('');
}

var str = 'helloJavaScripthellohtmlhellocss';
removeDuplicateChar2(str);  // 'heloJavscriptm'
```

借助于ES6的语法，以上方法体的执行代码还可以简写成一行的形式。

```js
return Array.prototype.filter.call(str, (char, index, arr) => arr.indexOf
(char) === index).join('');
```


3. 算法3

算法3的主要思想是借助ES6中的Set数据结构，Set具有自动去重的特性，可以直接将数组元素去重。

· 将字符串处理成数组，然后作为参数传递给Set的构造函数，通过new运算符生成一个Set的实例。

· 将Set通过扩展运算符（...）转换成数组形式，最终转换成字符串获得需要的结果。

```js
// 算法3
function removeDuplicateChar3(str) {
   // 字符串转换的数组作为参数，生成Set的实例
   let set = new Set(str.split(''));
    // 将set重新处理为数组，然后转换成字符串
   return [...set].join('');
}

var str = 'helloJavaScripthellohtmlhellocss';
removeDuplicateChar3(str);  // 'heloJavscriptm'
```

##### 1.3.2.4  判断一个字符串是否为回文字符串

回文字符串是指一个字符串正序和倒序是相同的，例如字符串'abcdcba'是一个回文字符串，而字符串'abcedba'则不是一个回文字符串。

需要注意的是，这里不区分字符大小写，即a与A在判断时是相等的。

给定两个字符串'abcdcba'和'abcedba'，经过一定的算法处理，分别会返回“true”和“false”。


1. 算法1

算法1的主要思想是将字符串按从前往后顺序的字符与按从后往前顺序的字符逐个进行比较，如果遇到不一样的值则直接返回“false”，否则返回“true”。

```js
// 算法1
function isPalindromicStr1(str) {
   // 空字符则直接返回“true”
   if (!str.length) {
       return true;
   }
   // 统一转换成小写，同时转换成数组
   str = str.toLowerCase().split('');
   var start = 0, end = str.length - 1;
   // 通过while循环判断正序和倒序的字母
   while(start < end) {
      // 如果相等则更改比较的索引
      if(str[start] === str[end]) {
          start++;
          end--;
      } else {
          return false;
      }
   }
   return true;
}

var str1 = 'abcdcba';
var str2 = 'abcedba';

isPalindromicStr1(str1);  // true
isPalindromicStr1(str2);  // false
```

2. 算法2

算法2与算法1的主要思想相同，将正序和倒序的字符逐个进行比较，与算法1不同的是，算法2采用递归的形式实现。

递归结束的条件有两种情况，一个是当字符串全部处理完成，此时返回“true”；另一个是当遇到首字符与尾字符不同，此时返回“false”。而其他情况会依次进行递归处理。

```js
// 算法2
function isPalindromicStr2(str) {
   // 字符串处理完成，则返回“true”
   if(!str.length) {
      return true;
   }
   // 字符串统一转换成小写
   str = str.toLowerCase();
   let end = str.length - 1;
   // 当首字符和尾字符不同，直接返回“false”
   if(str[0] !== str[end]) {
      return false;
   }
   // 删掉字符串首尾字符，进行递归处理
   return isPalindromicStr2(str.slice(1, end));
}

var str1 = 'abcdcba';
var str2 = 'abcedba';

isPalindromicStr2(str1);  // true
isPalindromicStr2(str2);  // false
```

3. 算法3

算法3的主要思想是将字符串进行逆序处理，然后与原来的字符串进行比较，如果相等则表示是回文字符串，否则不是回文字符串。

```js
// 算法3
function isPalindromicStr3(str) {
   // 字符串统一转换成小写
   str = str.toLowerCase();
   // 将字符串转换成数组
   var arr = str.split('');
   // 将数组逆序并转换成字符串
    var reverseStr = arr.reverse().join('');
    return str === reverseStr;
}

var str1 = 'abcdcba';
var str2 = 'abcedba';

isPalindromicStr3(str1);  // true
isPalindromicStr3(str2);  // false
```

### 1.4 运算符

#### 1.4.1 等于运算符
不同于其他编程语言，JavaScript中相等的比较分为双等于（==）比较和三等于（===）比较。这是因为在Java、C等强类型语言中，一个变量在使用前必须声明变量类型，所以在比较的时候就无须判断变量类型，只需要有双等于即可。

· 双等于运算符在比较时，会将两端的变量进行隐式类型转换，然后比较值的大小。

· 三等于运算符在比较时，会优先比较数据类型，数据类型相同才去判断值的大小，如果类型不同则直接返回“false”。

##### 1.4.1.1 三等于运算符

① 如果比较的值类型不相同，则直接返回“false”。

```js
1 === '1'; // false
true === 'true';  // false
```

需要注意的是，基本类型数据存在包装类型。在未使用new操作符时，简单类型的比较实际为值的比较，而使用了new操作符后，实际得到的是引用类型的值，在判断时会因为类型不同而直接返回“false”。

```js
1 === Number(1);  // true
1 === new Number(1);  // false
'hello' === String('hello');  // true
'hello' === new String('hello'); // false
```

② 如果比较的值都是数值类型，则直接比较值的大小，相等则返回“true”，否则返回“false”。需要注意的是，如果参与比较的值中有任何一方为NaN，则返回“false”。

```js
23 === 23;   // true
34 === NaN;  // false
NaN === NaN; // false
```

③ 如果比较的值都是字符串类型，则判断每个位置的字符是否一样，如果一样则返回“true”，否则返回“false”。

```js
'kingx' === 'kingx';   // true
'kingx' === 'kingx2';  // false
```

④ 如果比较的值都是Boolean类型，则两者同时为true或者false时，返回“true”，否则返回“false”。

```js
false === false;  // true
true === false;   // false
```

⑤ 如果比较的值都是null或者undefined，则返回“true”；如果只有一方为null或者undefined，则返回“false”。

```js
null === null;   // true
undeﬁned === undeﬁned;   // true
null === undeﬁned;   // false
```

⑥ 如果比较的值都是引用类型，则比较的是引用类型的地址，当两个引用指向同一个地址时，则返回“true”，否则返回“false”。

```js
var a = [];
var b = a;
var c = [];
console.log(a === b); // true
console.log(a === c); // false
console.log({} === {}); // false
```

实际上，如果不是通过赋值运算符（=）将定义的引用类型的值赋予变量，那么引用类型的值在比较后都会返回“false”，所以我们会发现空数组或者空对象的直接比较返回的是“false”。

```js
[] === [];  // false
{} === {};  // false
```

引用类型变量的比较还有一个很明显的特点，即只要有一个变量是通过new操作符得到的，都会返回“false”，包括基本类型的包装类型。

```js
'hello' === new String('hello');  // false
new String('hello') === new String('hello');  // false

// 函数对象类型
function Person(name) {
   this.name = name;
}
var p1 = new Person('zhangsan');
var p2 = new Person('zhangsan');
console.log(p1 === p2);  // false
```

##### 1.4.1.2  双等于运算符
相比于三等于运算符，双等于运算符在进行相等比较时，要略微复杂，因为它不区分数据类型，而且会做隐式类型转换。

① 如果比较的值类型相同，则采用与三等于运算符一样的规则。

② 如果比较的值类型不同，则会按照下面的规则进行转换后再进行比较。

· 如果比较的一方是null或者undefined，只有在另一方是null或者undefined的情况下才返回“true”，否则返回“false”。

```js
null == undeﬁned;      // true
null == 1;             // false
null == false;         // false
undeﬁned == 0;         // false
undeﬁned == false;      // false
```

· 如果比较的是字符串和数值类型数据，则会将字符串转换为数值后再进行比较，如果转换后的数值相等则返回“true”，否则返回“false”。

```js
1 == '1';     // true
123 == '123'; // true
```

需要注意的是，如果字符串是十六进制的数据，会转换为十进制后再进行比较。

```js
'0x15' == 21;  // true
```
字符串'0x15'实际为十六进制数，转换为十进制后为1×16 + 5 = 21，与21比较后返回“true”。

字符串并不支持八进制的数据，如果字符串以0开头，则0会直接省略，后面的值当作十进制返回。

```js
'020' == 16;  // false
'020' == 20;  // true
```
· 如果任一类型是boolean值，则会将boolean类型的值进行转换，true转换为1，false转换为0，然后进行比较。

```js
'1' == true;    // true
'0' == false;   // true
'0.0' == false; // true
'true' == true; // false
```

· 如果其中一个值是对象类型，另一个值是基本数据类型或者对象类型，则会调用对象的valueOf()函数或者toString()函数，将其转换成基本数据类型后再作比较，

#### 1.4.2 typeof运算符
typeof运算符在处理不同数据类型时会得到不同的结果。

<img src="/img/typeof.png"  alt="typeof" height = "auto"/>

##### 1.4.2.1 处理Undefined类型的值
虽然Undefined类型的值只有一个undefined，但是typeof运算符在处理以下3种值时都会返回“undefined”。

· undefined本身。

· 未声明的变量。

· 已声明未初始化的变量。

```js
var declaredButUndeﬁnedVariable;
typeof undeﬁned === 'undeﬁned';    // true
typeof declaredButUndeﬁnedVariable === 'undeﬁned';  // true，已声明未初始化的变量
typeof undeclaredVariable === 'undeﬁned';  // true，未声明的变量
```

##### 1.4.2.2 处理Boolean类型的值
Boolean类型的值只有两个，分别是true和false。typeof运算符在处理这两个值以及它们的包装类型时都会返回“boolean”，但是不推荐使用包装类型的写法。

```js
typeof true === 'boolean';          // true
typeof false === 'boolean';         // true
typeof Boolean(true) === 'boolean'; // true，不推荐这么写
```

##### 1.4.2.3 处理Number类型的值

```js
typeof 37 === 'number';        // true
typeof 3.14 === 'number';      // true
typeof Math.LN2 === 'number';  // true
typeof Inﬁnity === 'number';   // true
typeof NaN === 'number';       // true
typeof Number(1) === 'number'; // true，不推荐这么写
```

##### 1.4.2.4 处理String类型的值

 · 任何类型的字符串，包括空字符串和非空字符串。

 · 返回值为字符串类型的表达式。

 · 字符串类型的包装类型，例如String('hello')、String('hello' + 'world')，虽然它们也会返回“String”，但是并不推荐这么写。

```js
typeof "" === 'string';            // true
typeof "bla" === 'string';         // true
typeof (typeof 1) === 'string';    // true，因为typeof会返回一个字符串
typeof String("abc") === 'string'; // true，不推荐这么写
```

##### 1.4.2.5 处理Symbol类型的值

##### 1.4.2.6 处理Function类型的值

· 函数的定义，包括函数声明或者函数表达式两种形式。

· 使用class关键字定义的类，class是在ES6中新增的关键字，它不是一个全新的概念，原理依旧是原型继承，本质上仍然是一个Function。

· 某些内置对象的特定函数，例如Math.sin()函数、Number.isNaN()函数等。

· Function类型对象的实例，一般通过new关键字得到。

```js
var foo = function () {};
function foo2() {}

typeof foo === 'function';       // true，函数表达式
typeof foo2 === 'function';      // true，函数声明
typeof class C{} === 'function'; // true
typeof Math.sin === 'function';  // true
typeof new Function() === 'function';  // true，new操作符得到Function类型的实例
```

##### 1.4.2.7 处理Object类型的值

· 对象字面量形式，例如{name: 'kingx'}。

· 数组，例如[1, 2, 3]和Array(1, 2, 3)。

· 所有构造函数通过new操作符实例化后得到的对象，例如new Date()、new function(){}，但是new Function(){}除外。

· 通过new操作符得到的基本数据类型的包装类型对象，如new Boolean(true)、new Number(1)，但不推荐这么写。

**细心的读者可能发现了，与基本数据类型的包装类型相关的部分，我们都有写“不推荐这么写”，这是为什么呢？因为涉及包装类型时，使用了new操作符与没有使用new操作符得到的值在通过typeof运算符处理后得到的结果是不一样的，很容易让人混淆。**

```js
typeof {a:1} === 'object';      // true，对象字面量
typeof [1, 2, 4] === 'object';  // true，数组
typeof new Date() === 'object'; // true，Date对象的实例
// 下面的代码容易令人迷惑，不要使用！
typeof new Boolean(true) === 'object';  // true
typeof new Number(1) === 'object';      // true
typeof new String("abc") === 'object';  // true
```

##### 1.4.2.8 使用typeof运算符注意事项

**1、typeof运算符区分对待Object类型和Function类型**
在实际使用过程中，有必要区分Object类型和Function类型，而typeof运算符就能帮我们实现。

**2、typeof运算符对null的处理**
使用typeof运算符对null进行处理，返回的是“object”，这是一个让大家都感到惊讶的结果。因为null是一个原生类型的数据，为什么typeof运算符会返回“object”呢？

这是一个在JavaScript设计之初就存在的问题，这里简单介绍下。

在JavaScript中，每种数据类型都会使用3bit表示。

· 000表示Object类型的数据。

· 001表示Int类型的数据。

· 010表示Double类型的数据。

· 100表示String类型的数据。

· 110表示Boolean类型的数据。

由于null代表的是空指针，大多数平台中值为0x00，因此null的类型标签就成了0，所以使用typeof运算符时会判断为object类型，返回“object”。

**3、typeof运算符相关语法的括号**

```js
var number = 123;
typeof (number + ' hello');  // "string"
typeof number + ' hello';    // "number hello"
typeof 1 / 0;     // "NaN"
typeof (1 / 0);   // "number"
```

#### 1.4.3 逗号运算符
小小的逗号在JavaScript中有很大的用处，一方面它是基本的分隔符，例如，函数传递多个参数时，使用逗号分隔。

```js
console.log('我喜欢去%s上学习%s', '面试厅', 'JavaScript');
```
另一方面它可以作为一个运算符，作用是将多个表达式连接起来，从左至右依次执行。

##### 1.4.3.1 在for循环中批量执行表达式

```js
for (var i = 0, j = 10; i < 10, j < 20; i++, j++) {
   console.log(i, j);
}
```
一般在for循环的末尾处，只允许执行单个表达式。在这里我们通过逗号运算符，将i++和j++两个表达式视为同一个表达式，因此可以一次执行，处理i与j两个变量的递增。

##### 1.4.3.2 用于简化代码

```js
if (x) {
   foo();
   return bar();
} else {
   return 1;
}

// 使用逗号运算符简写后
x ? (foo(), bar()) : 1;
```

##### 1.4.3.3 用小括号保证逗号运算符的优先级
在所有的运算符中，逗号运算符的优先级是最低的，因此对于某些涉及优先级的问题，我们需要使用到小括号，将含有逗号运算符的表达式括起来。

```js
var a = 20;
var b = ++a, 10;
console.log(b);  // Uncaught SyntaxError: Unexpected number
```
对于上面的语句，首先定义一个变量a，然后使用逗号运算符对变量a执行自增操作，同时返回“10”，并将其赋值给变量b。

我们可能会认为最后输出b的值为10，但是运行后却抛出了异常，这是为什么呢？

**在上面的代码中，同时出现了赋值运算符与逗号运算符，因为逗号运算符的优先级比较低，实际会先执行赋值运算符，即先执行var b = ++a语句，再去执行后面的10，它不是一个合法的语句，所以会抛出异常。**

那么我们该怎么解决这个问题呢？

那就是使用小括号，保证逗号运算符的优先级，将赋值语句后面的内容括起来，执行完含有逗号运算符的表达式后，再执行赋值语句。

```js
var a = 20;
var b = (++a, 10);
console.log(b);  // 10
```

#### 1.4.4 运算符优先级

```js
// 语句1
a = b = c;  // a = b = 10;

// 语句2
a > b > c; // 6 > 4 > 3
```
在语句1中，将运算符OP1与OP2同时设置为赋值运算符，因为优先级相同，所以会从右到左依次运行

```js
b = 10;
a = 10;
```

在语句2中，将运算符OP1与OP2同时设置为比较运算符，因为优先级相同，所以从左至右依次执行，

```js
6 > 4;    // true
true > 3 // false
```

### 1.5 toString()函数 与 valueOf()函数

#### 1.5.1 toString()函数
toString()函数的作用是把一个逻辑值转换为字符串，并返回结果。Object类型数据的toString()函数默认的返回结果是"[object Object]"，当我们自定义新的类时，可以重写toString()函数，返回可读性更高的结果。

#### 1.5.2 valueOf()函数
valueOf()函数的作用是返回最适合引用类型的原始值，如果没有原始值，则会返回引用类型自身。Object类型数据的valueOf()函数默认的返回结果是"{}"，即一个空的对象字面量。

如果一个引用类型的值既存在toString()函数又存在valueOf()函数，那么在做隐式转换时，会调用哪个函数呢？

这里我们可以概括成两种场景，分别是引用类型转换为String类型，以及引用类型转换为Number类型。

**1、 引用类型转换为String类型**

一个引用类型的数据在转换为String类型时，一般是用于数据展示，转换时遵循以下规则。

· 如果对象具有toString()函数，则会优先调用toString()函数。如果它返回的是一个原始值，则会直接将这个原始值转换为字符串表示，并返回该字符串。

· 如果对象没有toString()函数，或者toString()函数返回的不是一个原始值，则会再去调用valueOf()函数，如果valueOf()函数返回的结果是一个原始值，则会将这个结果转换为字符串表示，并返回该字符串。

· 如果通过toString()函数或者valueOf()函数都无法获得一个原始值，则会直接抛出类型转换异常。

我们通过以下代码进行测试。

```js
var arr = [];

arr.toString = function () {
     console.log('执行了toString()函数');
     return [];
};

arr.valueOf = function () {
     console.log('执行了valueOf()函数');
     return [];
};

console.log(String(arr));
```

上面代码执行后的结果如下所示。

```js
执行了toString()函数
执行了valueOf()函数
TypeError: Cannot convert Object to primitive value
```
执行String(arr)代码时，需要将arr转换为字符串，则会优先执行toString()函数，但是其返回值为空数组[]，并不能转换为原生数据；然后调用valueOf()函数，其返回值同样为空数组[]；那么在调用完toString()函数和valueOf()函数后，均无法获取到原生数据类型表示，则抛出异常TypeError，表示无法将对象类型转换为原生数据类型。

**2、 引用类型转换为Number类型**

一个引用类型的数据在转换为Number类型时，一般是用于数据运算，转换时遵循以下规则。

· 如果对象具有valueOf()函数，则会优先调用valueOf()函数，如果valueOf()函数返回一个原始值，则会直接将这个原始值转换为数字表示，并返回该数字。

· 如果对象没有valueOf()函数，或者valueOf()函数返回的不是原生数据类型，则会再去调用toString()函数，如果toString()函数返回的结果是一个原始值，则会将这个结果转换为数字表示，并返回该数字。

· 如果通过toString()函数或者valueOf()函数都无法获得一个原始值，则会直接抛出类型转换异常。

我们通过以下代码进行测试。

```js
var arr = [];

arr.toString = function () {
   console.log('执行了toString()函数');
   return [];
};

arr.valueOf = function () {
   console.log('执行了valueOf()函数');
   return [];
};

console.log(Number(arr));
// 执行了valueOf()函数
// 执行了toString()函数
// TypeError: Cannot convert Object to primitive value
```

了解了valueOf()函数和toString()函数的关系后，我们再用下面两组代码深入拓展一下其他相关知识。

**拓展1**

```js
[] == 0; // true
[1] == 1; // true
[2] == 2; // true
```
在第一行中，空数组可以转换为数字0；在第二行和第三行中，只有一个数字元素的数组可以转换为该数字。这是为什么呢？

因为数组继承了Object类型默认的valueOf()函数，这个函数返回的是数组自身，而不是原生数据类型，所以会继续调用toString()函数。数组调用toString()函数时会返回数组元素以逗号作为分隔符构成的字符串，那么空数组就转换为空字符串，而空字符串与数字0在非严格相等的情况下是相等的，即'' == 0，返回“true”。

同样，只包含一个数字的数组[1]，转换后为字符串"1"，后判断"1" == 1，返回“true”。

**拓展2**

以下是另外一组Object类型的数据，请观察结果有什么不同。

```js
var obj = {
     i: 10,
     toString: function () {
         console.log('toString');
         return this.i;
     },
     valueOf: function () {
         console.log('valueOf');
         return this.i;
     }
};

+obj;  // valueOf
'' + obj;  // valueOf
String(obj);  // toString
Number(obj);  // valueOf
obj == '10';  // valueOf，true
obj === '10'; // false
```

第一行执行代码为+obj，将对象obj转换为原始值，用于数据运算，优先调用valueOf()函数，获得原始值，结果为数字“10”。

第二行执行代码为'' + obj，将对象obj转换为原始值，用于数据运算，优先调用valueOf()函数，获取原始值，并与字符串进行拼接，结果为字符串"10"。

第五行执行代码为obj == '10'，将对象obj转换为原始值，用于数据运算，优先调用valueOf()函数，即将10与'10'进行比较，两者是相等的，结果为“true”；

第六行执行代码为obj === '10'，因为两者数据类型不一致，直接返回“false”，并不会执行toString()函数或者valueOf()函数。

### 1.6 javascript中常见的判空方法
#### 1.6.1 判断变量为空对象

（1）判断变量为null或者undefined

判断一个变量是否为空时，可以直接将变量与null或者undefined相比较，需要注意双等于（==）和三等于（===）的区别。

```js
if(obj == null) {} // 可以判断null或者undeﬁned

if(obj === undeﬁned) {} // 只能判断undeﬁned
```

（2）判断变量为空对象{}

判断一个变量是否为空对象时，可以通过for...in语句遍历变量的属性，然后调用hasOwnProperty()函数，判断是否有自身存在的属性，如果存在则不为空对象，如果不存在自身的属性（不包括继承的属性），那么变量为空对象。

```js
// 判断变量为空
function isEmpty(obj) {
  for(let key in obj) {
      if(obj.hasOwnProperty(key)) {
         return false;
      }
  }
  return true;
}
```

我们通过以下语句来做测试。

```js
// 定义空的对象字面量
var o = {};

function Person() {}
Person.prototype.name = 'kingx';
// 通过new操作符获取对象
var p = new Person();

console.log(isEmpty(o));  // true
console.log(isEmpty(p));  // true
```

#### 1.6.2 判断变量为空数组

判断变量是否为空数组时，首先需要判断变量是否为数组，然后通过数组的length属性确定。

```js
arr instanceof Array && arr.length === 0
```
当以上两个条件都满足时，变量是一个空数组。

#### 1.6.3 判断变量为空字符串

```js
str == '' || str.trim().length == 0;
```

#### 1.6.4 判断变量为0或者NaN

当一个变量为Number类型时，判空即判断变量是否为0或者NaN，因为NaN与任何值比较都为false，所以我们可以通过取非运算符完成。

```js
!(Number(num) && num) == true;
```

#### 1.6.5 !x == true的所有情况

本小节一开始就讲到!x为true时，会包含很多种情况，这里我们一起来总结下:

· 变量为null。

· 变量为undefined。

· 变量为空字符串' '。

· 变量为数字0，包括+0、-0。

· 变量为NaN。

### 1.7 javascript中switch语句

```js
switch(expression) {
   case value1:
      statement1;
      break;
   case value2:
      statement2;
      break;
   default:
      statement;
}
```

因为在JavaScript中对于case的比较是采用严格相等(===)的。

## 2、引用数据类型

引用数据类型主要用于区别基本数据类型，描述的是具有属性和函数的对象。JavaScript中常用的引用数据类型包括Object类型、Array类型、Date类型、RegExp类型、Math类型、Function类型以及基本数据类型的包装类型，如Number类型、String类型、Boolean类型等。

引用数据类型有不同于基本数据类型的特点，具体如下所示。

· 引用数据类型的实例需要通过new操作符生成，有的是显式调用，有的是隐式调用。

· 引用数据类型的值是可变的，基本数据类型的值是不可变的。

· 引用数据类型变量赋值传递的是内存地址。

· 引用数据类型的比较是对内存地址的比较，而基本数据类型的比较是对值的比较。

### 2.1 Object类型及其实例和静态函数

#### 2.1.1 深入了解JavaScript中的new操作符

**new操作符在执行过程中会改变this的指向，**所以在了解new操作符之前，我们先解释一下this的用法。

```js
function Cat(name, age) {
   this.name = name;
   this.age = age;
}

// Cat {name: "miaomiao", age: 18}
console.log(new Cat('miaomiao',18));
```

事实上我们并未通过return返回任何值，为什么输出的信息中会包含name和age属性呢？其中起作用的就是this这个关键字了。

```js
function Cat(name,age) {
   console.log(this);  // Cat {}
   this.name = name;
   this.age = age;
}
new Cat('miaomiao',18);
```

**在JavaScript中，如果函数没有return值，则默认return this**

```js
function Cat(name, age) {
   var Cat = {};
   Cat.name = name;
   Cat.age = age;
   return Cat;
}
console.log(new Cat('miaomiao', 18));  // {name: "miaomiao", age: 18}
```

通过以上的分析，我们了解了构造函数中this的用法，那么它与new操作符之间有什么关系呢？

```js
let cat = new Cat();
```

从表面上看这行代码的主要作用是创建一个Cat对象的实例，并将这个实例值赋予cat变量，cat变量就会包含Cat对象的属性和函数。

**其实，new操作符做了3件事情，如下代码所示:**

```js
1. var cat = {};
2. cat.__proto__ = Cat.prototype;
3. Cat.call(cat);
```

第一件事： 创建cat空对象；

第二件事： 将空对象的\__proto__指向Cat对象的prototype的属性

第三件事： 将Cat()函数中的this指向cat对象


我们自定义一个类似new功能的函数，来具体讲解上面的3行代码。

```js
function Cat(name, age) {
   this.name = name;
   this.age = age;
}
function New() {
   var obj = {};
   var res = Cat.apply(obj, arguments);
   return typeof res === 'object' ? res : obj;
}
console.log(New('mimi', 18));  //Object {name: "mimi", age: 18}
```

返回的结果中也包含name和age属性，这就证明了new运算符对this指向的改变。Cat.apply(obj, arguments)调用后Cat对象中的this就指向了obj对象，这样obj对象就具有了name和age属性。

因此，不仅要关注new操作符的函数本身，也要关注它的原型属性。

我们对上面的代码进行改动，在Cat对象的原型上增加一个sayHi()函数，然后通过New()函数返回的对象，去调用sayHi()函数，看看执行情况如何。

```js
function Cat(name, age) {
   this.name = name;
   this.age = age;
}

Cat.prototype.sayHi = function () {
   console.log('hi')
};

function New() {
   var obj = {};
   var res = Cat.apply(obj, arguments);
   return typeof res === 'object' ? res : obj;
}
console.log(New('mimi', 18).sayHi()); // Uncaught TypeError: New(...).sayHi is not a function
```

我们发现执行报错了，New()函数返回的对象并没有调用sayHi()函数，这是因为sayHi()函数是属于Cat原型的函数，只有Cat原型链上的对象才能继承sayHi()函数，那么我们该怎么做呢？

这里需要用到的就是\__proto__属性，实例的\__proto__属性指向的是创建实例对象时，对应的函数的原型。设置obj对象的\__proto__值为Cat对象的prototype属性，那么obj对象就继承了Cat原型上的sayHi()函数，这样就可以调用sayHi()函数了。

```js
function Cat(name, age) {
   this.name = name;
   this.age = age;
}

Cat.prototype.sayHi = function () {
   console.log('hi')
};

function New() {
   var obj = {};
   obj._ _proto_ _ = Cat.prototype;  // 核心代码，用于继承
   var res = Cat.apply(obj, arguments);
   return typeof res === 'object' ? res : obj;
}
console.log(New('mimi', 18).sayHi());
```

#### 2.1.2  Object类型的实例函数

实例函数是指函数的调用是基于Object类型的实例的。代码如下所示。

```js
var obj = new Object();
```

所有实例函数的调用都是基于obj这个实例。

Object类型中有几个很重要的实例函数，这里分别进行详细的讲解。

**1、 hasOwnProperty(propertyName)函数**

该函数的作用是判断对象自身是否拥有指定名称的实例属性，此函数不会检查实例对象原型链上的属性。


**2、propertyIsEnumerable(propertyName)函数**

该函数的作用是判断指定名称的属性是否为实例属性并且是否是可枚举的，如果是原型链上的属性或者不可枚举都将返回“false”。

```js
// 1.数组
var array = [1, 2, 3];
array.name = 'Array';
console.log(array.propertyIsEnumerable('name')); // true ：name属性为实例属性
console.log(array.propertyIsEnumerable('join')); // false ：join()函数继承自Array类型
console.log(array.propertyIsEnumerable('length')); // false ：length属性继承自Array类型
console.log(array.propertyIsEnumerable('toString')); // false ：toString()函数
                                                    // 继承自Object

// 2.自定义对象
var Student = function (name) {
   this.name = name;
};
// 定义一个原型函数
Student.prototype.sayHello = function () {
   alert('Hello' + this.name);
};

var a = new Student('tom');
console.log(a.propertyIsEnumerable('name')); // true ：name为自身定义的实例属性
console.log(a.propertyIsEnumerable('age'));  // false ：age属性不存在，返回false
console.log(a.propertyIsEnumerable('sayHello')); // false ：sayHello属于原型函数

// 设置name属性为不可枚举的
Object.deﬁneProperty(a, 'name', {
   enumerable: false
});
console.log(a.propertyIsEnumerable('name')); // false ：name设置为不可枚举
```

#### 2.1.3  Object类型的静态函数

静态函数指的是方法的调用基于Object类型自身，不需要通过Object类型的实例。

**1、 Object.create()函数**

该函数的主要作用是创建并返回一个指定原型和指定属性的对象。语法格式如下所示。

```js
Object.create(prototype, propertyDescriptor)。
```
其中prototype属性为对象的原型，可以为null。若为null，则对象的原型为undefined。属性描述符的格式如下所示。

```js
// 建立一个自定义对象，设置name和age属性
var obj = Object.create(null, {
   name: {
       value: 'tom',
       writable: true,
       enumerable: true,
       conﬁgurable: true
   },
   age: {
       value: 22
   }
});
console.log(obj.name); // tom
console.log(obj.age); // 22
obj.age = 28;
console.log(obj.age); // 22 ：age属性的writable默认为false，此属性为只读
for (var p in obj) {
   console.log(p); // name ：只输出name属性；age属性的enumerable默认为false，不能 通过for...in 枚举
}
```

我们尝试用polyfill版本实现Object.create()函数，通过polyfill我们可以更清楚明白Object.create()函数的实现原理。

```js
Object.create = function (proto, propertiesObject) {
   // 省去中间的很多判断
   function F() {}
   F.prototype = proto;
   return new F();
};
```

在create()函数中，首先声明一个函数为F()函数，然后将F()函数的prototype属性指向传入的proto参数，通过new操作符生成F()函数的实例。

假如var f = new F()，f.\__proto__ === F.prototype。实际上生成的对象实例会把属性继承到其\__proto__属性上。

我们再通过下面的实例来验证。

```js
var test = Object.create({x:123, y:345});
console.log(test); // {}，实际生成的对象为一个空对象
console.log(test.x); // 123
console.log(test._ _proto_ _.x);  //  123
console.log(test._ _proto_ _.x === test.x); // true
```
实际生成的test为一个空对象{}。但是我们可以访问其x属性，这是因为我们可以通过其\__proto__属性访问到x属性，所以我们通过test访问到x属性和y属性，实际是通过其\__proto__属性访问到的。


**2、 Object.defineProperties()函数**

该函数的主要作用是添加或修改对象的属性值，语法格式如下所示。

```js
Object.deﬁneProperties(obj, propertyDescriptor)
```

其中的属性描述符propertyDescriptor同Object.create()函数一样。例如，给一个空对象{}添加name和age属性，其代码如下所示。

```js
var obj = {};
// 为对象添加name和age属性
Object.deﬁneProperties(obj, {
   name: {
       value: 'tom',
       enumerable: true
   },
   age: {
       value: 22,
       enumerable: true
   }
});
for (var p in obj) {
   console.log(p); // name age ：输出name和age属性
}
obj.age = 23;
console.log(obj.age); // 22 ：age属性的writable默认为false，此属性为只读
```

**3、Object.getOwnPropertyNames()函数**

该函数的主要作用是获取对象的所有实例属性和函数，不包含原型链继承的属性和函数，数据格式为数组。

```js
function Person(name, age, gender) {
   this.name = name;
   this.age = age;
   this.gender = gender;
   this.getName = function () {
       return this.name;
   }
}

Person.prototype.eat = function () {
   return '吃饭';
};

var p = new Person();
console.log(Object.getOwnPropertyNames(p)); //  ["name", "age", "gender", "getName"]
```

**4、 Object.keys()函数**

该函数的主要作用是获取对象可枚举的实例属性，不包含原型链继承的属性，数据格式为数组。keys()函数区别于getOwnPropertyNames()函数的地方在于，keys()函数只获取可枚举类型的属性。

```js
var obj = {
   name: 'tom',
   age: 22,
   sayHello: function () {
       alert('Hello' + this.name);
   }
};
// (1) getOwnPropertyNames()函数与keys()函数返回的内容都相同
// ["name", "age", "sayHello"] ：返回对象的所有实例成员
console.log(Object.getOwnPropertyNames(obj));
// ["name", "age", "sayHello"] ：返回对象的所有可枚举成员
console.log(Object.keys(obj));
// 设置对象的name属性不可枚举
Object.deﬁneProperty(obj, 'name', {
   enumerable: false
});
// (2)keys()函数，只包含可枚举成员
// ["name", "age", "sayHello"] ：返回对象的所有实例成员
console.log(Object.getOwnPropertyNames(obj));

// ["age", "sayHello"] ：返回对象的所有可枚举成员
console.log(Object.keys(obj));
```

### 2.2 Array类型

#### 2.2.1  判断一个变量是数组还是对象

```js
var a = [1, 2, 3];
console.log(typeof a); // object
```

**所以使用typeof运算符并不能直接判断一个变量是对象还是数组类型。实际上，typeof运算符在判断基本数据类型时会很有用，但是在判断引用数据类型时，却显得很吃力。**

**1、instanceof运算符**

**instanceof运算符用于通过查找原型链来检测某个变量是否为某个类型数据的实例，使用instanceof运算符可以判断一个变量是数组还是对象。**

```js
var a =  [1, 2, 3];
console.log(a instanceof Array);  // true
console.log(a instanceof Object); // true

var b = {name: 'kingx'};
console.log(b instanceof Array);  // false
console.log(b instanceof Object); // true
```

封装

```js
// 判断变量是数组还是对象
function getDataType(o) {
   if (o instanceof Array) {
       return 'Array'
   } else if (o instanceof Object) {
       return 'Object';
   } else {
       return 'param is not object type';
   }
}
```

**2、判断构造函数**

判断一个变量是否是数组或者对象，**从另一个层面讲，就是判断变量的构造函数是Array类型还是Object类型。因为一个对象的实例都是通过构造函数生成的，所以，我们可以直接判断一个变量的constructor属性。**

```js
var a = [1, 2, 3];
console.log(a.constructor === Array);  // true
console.log(a.constructor === Object); // false

var b = {name: 'kingx'};
console.log(b.constructor === Array);  // false
console.log(b.constructor === Object); // true
```

那么一个变量为什么会有constructor属性呢？这就要涉及原型链的知识了。

每个变量都会有一个\__proto__属性，表示的是隐式原型。一个对象的隐式原型指向的是构造该对象的构造函数的原型，这里用数组来举例，代码如下所示。

```js
[]._ _proto_ _ === [].constructor.prototype;  // true
[]._ _proto_ _ === Array.prototype;  // true
```

上面直接通过constructor属性判断的语句也可以改写成下面的形式。

```js
var a = [1, 2, 3];
console.log(a._ _proto_ _.constructor === Array);  // true
console.log(a._ _proto_ _.constructor === Object); // false
```

封装

```js
// 判断变量是数组还是对象
function getDataType(o) {
   // 获取构造函数
   var constructor = o._ _proto_ _.constructor || o.constructor;
   if (constructor === Array) {
       return 'Array';
   } else if (constructor === Object) {
       return 'Object';
   } else {
       return 'param is not object type';
   }
}
```

**3、toString()函数**

每种引用数据类型都会直接或间接继承自Object类型，因此它们都包含toString()函数。不同数据类型的toString()函数返回值也不一样，所以通过toString()函数就可以判断一个变量是数组还是对象。

这里我们会借助call()函数，直接调用Object原型上的toString()函数，把主体设置为需要传入的变量，然后通过返回值进行判断。

```js
var a = [1, 2, 3];
var b = {name: 'kingx'};

console.log(Object.prototype.toString.call(a)); // [object Array]
console.log(Object.prototype.toString.call(b)); // [object Object]
```

**4、Array.isArray()函数**

```js
// 下面的函数调用都返回“true”
Array.isArray([]);
Array.isArray([1]);
Array.isArray(new Array());
// 鲜为人知的事实：其实 Array.prototype 也是一个数组。
Array.isArray(Array.prototype);

// 下面的函数调用都返回“false”
Array.isArray();
Array.isArray({});
Array.isArray(null);
Array.isArray(undeﬁned);
Array.isArray(17);
Array.isArray('Array');
Array.isArray(true);
```

#### 2.2.2  filter()函数过滤满足条件的数据
**filter()函数用于过滤出满足条件的数据，返回一个新的数组，不会改变原来的数组。**它不仅可以过滤简单类型的数组，而且可以通过自定义方法过滤复杂类型的数组。

filter()函数接收一个函数作为其参数，返回值为“true”的元素会被添加至新的数组中，返回值为“false”的元素则不会被添加至新的数组中，最后返回这个新的数组。如果没有符合条件的值则返回空数组。

接下来我们可以具体看看filter()函数的使用场景。

**1、针对简单类型的数组，找出数组中所有为奇数的数字**

```js
var ﬁlterFn = function (x) {
   return x % 2;
};
```

**2、针对复杂类型的数组，找出所有年龄大于18岁的男生**

```js
var ﬁlterFn = function (obj) {
    return obj.age > 18 && obj.gender === '男';
};
```

#### 2.2.3  reduce()函数累加器处理数组元素
reduce()函数最主要的作用是做累加处理，即接收一个函数作为累加器，将数组中的每一个元素从左到右依次执行累加器，返回最终的处理结果。

```js
arr.reduce(callback[, initialValue]);
```

initialValue用作callback的第一个参数值，如果没有设置，则会使用数组的第一个元素值。callback会接收4个参数（accumulator、currentValue、currentIndex、array）。

· accumulator表示上一次调用累加器的返回值，或设置的initialValue值。如果设置了initialValue，则accumulator=initialValue；否则accumulator=数组的第一个元素值。

· currentValue表示数组正在处理的值。

· currentIndex表示当前正在处理值的索引。如果设置了initialValue，则currentIndex从0开始，否则从1开始。

· array表示数组本身。

在掌握了reduce()函数的语法后，我们列举出了几种灵活运用reduce()函数的场景，看看它是如何解决对应问题的。

**1、求数组每个元素相加的和**

```js
var arr = [1, 2, 3, 4, 5];
var sum = arr.reduce(function (accumulator, currentValue) {
   return accumulator + currentValue;
}, 0);

console.log(sum);
```
设置initialValue为0，在进行第一轮运算时，accumulator为0，currentValue从1开始，第一轮计算完成累加的值为0+1=1；在进入第二轮计算时，accumulator为1，currentValue为2，第二轮计算完成累加的值为1+2=3；以此类推，在进行5轮计算后最终的输出结果为“15”。

**2、 统计数组中每个元素出现的次数**

假如存在一个数组为[1, 2, 3, 2, 2, 5, 1]，通过一定的算法，统计出其中数字1出现的次数为2，2出现的次数为3，3出现的次数为1，5出现的次数为1。

```js
var countOccurrences = function(arr) {
   return arr.reduce(function(accumulator, currentValue) {
       accumulator[currentValue] ? accumulator[currentValue]++ :
                              accumulator[currentValue] = 1;
       return accumulator;
   }, {});
};

// 测试代码
countOccurrences([1, 2, 3, 2, 2, 5, 1]);
```

**3、多维度统计数据**
我们在知道不同币值汇率的情况下，将一组人民币的值分别换算成美元和欧元的等量值。

首先我们需要有一组人民币值，假设如下。

```js
var items = [{price: 10}, {price: 50}, {price: 100}];
```

此时查到人民币：美元汇率为1:0.1478，人民币：欧元汇率为1:0.1265。通过一定的算法，需要计算出这些人民币值对应的美元是23.792，对应的欧元是20.240。

解决问题的思路如下。

因为涉及不同汇率的计算，reduce()函数的第一个callback参数可以封装为一个reducers数组。数组中的每个元素实际为一个函数，利用reduce()函数单独完成一个汇率的计算。

```js
var reducers = {
   totalInEuros : function(state, item) {
       return state.euros += item.price * 0.1265;
   },
   totalInDollars : function(state, item) {
       return state.dollars += item.price * 0.1487;
   }
};
```

上面的reducers通过一个manager()函数，利用object.keys()函数同时执行多个函数，每个函数完成各自的汇率计算。

```js
var manageReducers = function(reducers) {
   return function(state, item) {
       return Object.keys(reducers).reduce(
               function(nextState, key) {
                   reducers[key](state, item);
                   return state;
               },
               {}
       );
   }
};

var bigTotalPriceReducer = manageReducers(reducers);
var initialState = {euros: 0, dollars: 0};
var totals = items.reduce(bigTotalPriceReducer, initialState);
console.log(totals);
```
运行结束后得到的结果为“{euros: 20.240, dollars: 23.792}”，符合预期。


#### 2.2.4 求数组的最大值和最小值

**1、通过prototype属性扩展min()函数和max()函数**

```js
// 最小值
Array.prototype.min = function() {
   var min = this[0];
   var len = this.length;
   for (var i = 1; i < len; i++){
        if (this[i] < min){
            min = this[i];
        }
   }
   return min;
};
//最大值
Array.prototype.max = function() {
   var max = this[0];
   var len = this.length;
   for (var i = 1; i < len; i++){
       if (this[i] > max) {
           max = this[i];
       }
   }
   return max;
};
```
**2、 借助Math对象的min()函数和max()函数**
算法2的主要思想是通过apply()函数改变函数的执行体，将数组作为参数传递给apply()函数。这样数组就可以直接调用Math对象的min()函数和max()函数来获取返回值。

```js
// 最大值
Array.max = function(array) {
   return Math.max.apply(Math, array);
};
// 最小值
Array.min = function(array) {
   return Math.min.apply(Math, array);
};
```

**3、算法二的优化**

```js
// 最大值
Array.prototype.max = function() {
   return Math.max.apply({}, this);
};
// 最小值
Array.prototype.min = function() {
   return Math.min.apply({}, this);
};
```

**4、借助Array类型的reduce()函数**

```js
// 最大值
Array.prototype.max = function () {
   return this.reduce(function (preValue, curValue) {
       // 比较后，返回大的值
       return preValue > curValue ? preValue : curValue;
   });
};

// 最小值
Array.prototype.min = function () {
   return this.reduce(function (preValue, curValue) {
      // 比较后，返回小的值
      return preValue > curValue ? curValue : preValue;
   });
};
```

**5、借助Array类型的sort()函数**
算法5的主要思想是借助数组原生的sort()函数对数组进行排序，排序完成后首尾元素即是数组的最小、最大元素。

**默认的sort()函数在排序时是按照字母顺序排序的，数字都会按照字符串处理，例如数字11会被当作"11"处理，数字8会被当作"8"处理。**在排序时是按照字符串的每一位进行比较的，因为"1"比"8"要小，所以"11"在排序时要比"8"小。对于数值类型的数组来说，这显然是不合理的，所以需要我们自定义排序函数。

```js
var sortFn = function (a, b) {
   return a - b;
 };
var arr5 = [2, 4, 10, 7, 5, 8, 6];
var sortArr = arr5.sort(sortFn);

// 最小值
console.log(sortArr[0]); // 2
// 最大值
console.log(sortArr[sortArr.length - 1]); // 10
```

**6、借助ES6的扩展运算符**
算法6的主要思想是借助于ES6中增加的扩展运算符（...），将数组直接通过Math.min()函数与Math.max()函数的调用，找出数组中的最大值和最小值。

```js
var arr6 = [2, 4, 10, 7, 5, 8, 6]
// 最小值
console.log(Math.min(...arr6));
// 最大值
console.log(Math.max(...arr6));
```

#### 2.2.5 数组遍历的7种方法及兼容性处理（polyfill）

**1、最原始的for循环**

**2、基于forEach()函数的方法**

```js
var arr2 = [11, 22, 33];
arr2.forEach(function (element, index, array) {
   console.log(element);
});

// polyfill
// forEach()函数兼容性处理
Array.prototype.forEach = Array.prototype.forEach ||
   function (fn, context) {
       for (var k = 0, length = this.length; k < length; k++) {
            if (typeof fn === "function"
              && Object.prototype.hasOwnProperty.call(this, k)) {
                fn.call(context, this[k], k, this);
            }
       }
   };
```

**3、 基于map()函数的方法**

map()函数在用于在数组遍历的过程中，将数组中的每个元素做处理，得到新的元素，并返回一个新的数组。map()函数并不会改变原数组，其接收的参数和forEach()函数一样。

**需要注意的一点是，在map()函数的回调函数中需要通过return返回处理后的值，否则会返回“undefined”。例如下面：在没有通过return返回处理后的值的情况下，最终的结果为[undefined,undefined, undefined]。**

```js
var arr3 = [1, 2, 3];
var arrayOfSquares = arr3.map(function (element) {
   return element * element;
});
console.log(arrayOfSquares);

// polyfill
// map()函数兼容性处理
Array.prototype.map = Array.prototype.map ||
   function (fn, context) {
       var arr = [];
       if (typeof fn === "function") {
           for (var k = 0, length = this.length; k < length; k++) {
               if(typeof fn === "function"
                 &&Object.prototype.hasOwnProperty.call(this, k)){
                   arr.push(fn.call(context, this[k], k, this));
               }
           }
       }
       return arr;
   };
```

**4、 基于filter()函数的方法**

filter()函数是通过判断返回值是否为“true”来决定是否将返回值push至新的数组中。

```js
// ﬁlter()函数兼容性处理
Array.prototype.ﬁlter = Array.prototype.ﬁlter ||
   function (fn, context) {
       var arr = [];
       if (typeof fn === "function") {
           for (var k = 0, length = this.length; k < length; k++) {
                if(typeof fn === "function" && Object.prototype.hasOwnProperty.call(this, k)) {
                   fn.call(context, this[k], k, this) && arr.push(this[k]);
                }
           }
       }
       return arr;
   };
```

**5、 基于some()函数与every()函数的方法**

some()函数与every()函数的相似之处在于都用于数组遍历的过程中，判断数组是否有满足条件的元素，满足条件则返回“true”，否则返回“false”。some()函数与every()函数的区别在于some()函数只要数组中某个元素满足条件就返回“true”，不会对后续元素进行判断；而every()函数是数组中每个元素都要满足条件时才返回“true”。

```js
// 定义判断的函数
function isBigEnough(element, index, array) {
   return element > 4;
}
// 测试some()函数
var passed1 = [1, 2, 3, 4].some(isBigEnough);
var passed2 = [1, 2, 3, 4, 5].some(isBigEnough);
console.log(passed1); // false
console.log(passed2); // true

// 测试every()函数
var passed3 = [2, 3, 4].every(isBigEnough);
var passed4 = [5, 6].every(isBigEnough);
console.log(passed3); // false
console.log(passed4); // true

// some()函数兼容性处理
Array.prototype.some = Array.prototype.some ||
   function (fn, context) {
       var passed = false;
       if (typeof fn === "function"
         &&Object.prototype.hasOwnProperty.call(this, k)) {
          for (var k = 0, length = this.length; k < length; k++) {
               if (passed === true) break; // 如果有返回值为“true”，直接跳出循环
               passed = !!fn.call(context, this[k], k, this);
          }
       }
       return passed;
   };


// every()函数兼容性处理
Array.prototype.every = Array.prototype.every ||
   function (fn, context) {
       var passed = true;
        if (typeof fn === "function"
         &&Object.prototype.hasOwnProperty.call(this, k)) {
          for (var k = 0, length = this.length; k < length; k++) {
              if (passed === false) break; // 如果有返回值为“false”，直接跳出循环
              passed = !!fn.call(context, this[k], k, this);
          }
       }
       return passed;
   };
```

**6、 基于reduce()函数的方法**

```js
// reduce()函数兼容性处理
Array.prototype.reduce = Array.prototype.reduce ||
   function (callback, initialValue) {
       var previous = initialValue, k = 0, length = this.length;
       if (typeof initialValue === "undeﬁned") {
           previous = this[0];
           k = 1;
       }
       if (typeof callback === "function") {
           for (k; k &lt; length; k++) {
               //每轮计算完后，需要将计算后的返回值重新赋给累加函数的第一个参数
               this.hasOwnProperty(k)
               && (previous = callback(previous, this[k], k, this));
           }
       }
       return previous;
   };
```

**7、基于find()函数的方法**

find()函数用于数组遍历的过程中，找到第一个满足条件的元素值时，则直接返回该元素值；如果都不满足条件，则返回“undefined”。

```js
var value = [1, 5, 10, 15].ﬁnd(function (element, index, array) {
   return element > 9;
});
var value2 = [1, 5, 10, 15].ﬁnd(function (element, index, array) {
   return element > 20;
});

console.log(value); // 10
console.log(value2); // undeﬁned

// polyfill
Array.prototype.ﬁnd = Array.prototype.ﬁnd ||
   function (fn, context) {
       if (typeof fn === "function") {
          for (var k = 0, length = this.length; k < length; k++) {
               if (fn.call(context, this[k], k, this)) {
                   return this[k];
               }
          }
       }
       return undeﬁned;
   };
```

#### 2.2.6 数组去重的7种算法
例如存在一个数组[1, 4, 5, 7, 4, 8, 1, 10, 4]，通过一定的算法，需要得到的数组为[1,4, 5, 7, 8, 10]。

**1、遍历数组 / 利用对象键值对**

```js
// 遍历数组
function arrayUnique(array) {
   var result = [];
   for (var i = 0; i < array.length; i++) {
       if(result.indexOf(array[i]) === -1) {
           result.push(array[i]);
       }
   }
   return result;
}

// 对象键值对
function arrayUnique2(array) {
   var obj = {}, result = [], val, type;
   for (var i = 0; i < array.length; i++) {
       val = array[i];
       type = typeof val;
       if (!obj[val]) {
           obj[val] = [type];
           result.push(val);
       } else if (obj[val].indexOf(type) < 0) {   // 判断数据类型是否存在
           obj[val].push(type);
           result.push(val);
       }
   }
   return result;
}
var array2 = [1, 4, 5, 7, 4, 8, 1, 10, 4, '1'];
console.log(arrayUnique2(array2));
```

**2、先排序，再去重**

主要思想是借助原生的sort()函数对数组进行排序，然后对排序后的数组进行相邻元素的去重，将去重后的元素添加至新的数组中，返回这个新数组。

```js
function arrayUnique3(array) {
   var result = [array[0]];
   array.sort(function(a,b){return a-b});
   for (var i = 0; i < array.length; i++) {
       if (array[i] !== result[result.length - 1]) {
           result.push(array[i]);
       }
   }
   return result;
}
```
**3、优先遍历数组**

主要思想是利用双层循环，分别指定循环的索引i与j，j的初始值为i+1。在每层循环中，比较索引i和j的值是否相等，如果相等则表示数组中出现了相同的值，则需要更新索引i与j，操作为++i；同时将其赋值给j，再对新的索引i与j的值进行比较。循环结束后会得到一个索引值i，表示的是右侧没有出现相同的值，将其push到结果数组中，最后返回结果数组。

```js
function arrayUnique4(array) {
   var result = [];
   for (var i = 0, l = array.length; i < array.length; i++) {
       for (var j = i + 1; j < l; j++) {
            // 依次与后面的值进行比较，如果出现相同的值，则更改索引值
            if (array[i] === array[j]) {
                j = ++i; // i自身先加，再赋值给j
            }
       }
       // 每轮比较完毕后，索引为i的值为数组中只出现一次的值
       result.push(array[i]);
   }
   return result;
 }
```

**4、基于reduce()函数**

需要借助一个key-value对象。在reduce()函数的循环中判断key是否重复，如果为是，则将当前元素push至结果数组中。实际做法是设置initialValue为一个空数组[]，同时将initialValue作为最终的结果进行返回。在reduce()函数的每一轮循环中都会判断数据类型，如果数据类型不同，将表示为不同的值，如1和"1"，将作为不重复的值。

```js
function arrayUnique5(array) {
   var obj = {}, type;
   return array.reduce(function (preValue, curValue) {
       type = typeof curValue;
       if (!obj[curValue]) {
           obj[curValue] = [type];
           preValue.push(curValue);
       }
        // 判断数据类型是否存在
       else if (obj[curValue].indexOf(type) < 0) {
           obj[curValue].push(type);
           preValue.push(curValue);
       }
       return preValue;
   }, []);
}
var array5 = [1, 4, 5, 7, 4, 8, 1, 10, 4, '1'];
console.log(arrayUnique5(array4)); // [1, 4, 5, 7, 8, 10, "1"]
```

**5、借助ES6的Set数据结构**

主要思想是借助于ES6中新增的Set数据结构，它类似于数组，但是有一个特点，即成员都是唯一的，所以Set具有自动去重的功能。

在ES6中，Array类型增加了一个from()函数，用于将类数组对象转化为数组，然后再结合Set可以完美实现数组的去重。

```js
function arrayUnique6(array) {
   return Array.from(new Set(array));
}
var arr6 = [1, 4, 5, 7, 4, 8, 1, 10, 4, '1'];
console.log(arrayUnique6(arr6)); // [1, 4, 5, 7, 8, 10, "1"]
```

**6、借助ES6的Map数据结构**

主要思想是借助于ES6中新增的Map数据结构，它是一种基于key-value存储数据的结构，每个key都只对应唯一的value。如果将数组元素作为Map的key，那么判断Map中是否有相同的key，就可以判断出元素的重复性。

Map还有一个特点是key会识别不同数据类型的数据，即1与"1"在Map中会作为不同的key处理，不需要通过额外的函数来判断数据类型。

基于Map数据结构，通过filter()函数过滤，即可获得去重后的结果。

```js
function arrayUnique7(array) {
   var map = new Map();
   return array.ﬁlter((item) => !map.has(item) && map.set(item, 1));
}
var arr7 = [1, 4, 5, 7, 4, 8, 1, 10, 4, '1'];
console.log(arrayUnique7(arr7)); //[1, 4, 5, 7, 8, 10, "1"]
```

#### 2.2.7 找出数组中出现次数最多的元素

存在一个数组为[3, 5, 6, 5, 9, 8, 10, 5, 7, 7, 10, 7, 7, 10, 10, 10, 10, 10]，通过一定的算法，找出次数最多的元素为10，其出现次数为7次。

**1、利用键值对**

```js
function ﬁndMost1(arr) {
   if (!arr.length) return;
   if (arr.length === 1) return 1;
   var res = {};
   // 遍历数组
   for (var i = 0, l = arr.length; i < l; i++) {
       if (!res[arr[i]]) {
           res[arr[i]] = 1;
       } else {
          res[arr[i]]++;
       }
   }
   // 遍历 res
   var keys = Object.keys(res);
   var maxNum = 0, maxEle;
   for (var i = 0, l = keys.length; i < l; i++) {
       if (res[keys[i]] > maxNum) {
           maxNum = res[keys[i]];
           maxEle = keys[i];
       }
   }
   return '出现次数最多的元素为:' + maxEle + '，出现次数为:' + maxNum;
}
```

**2、利用键值对优化版**

```js
function ﬁndMost2(arr) {
   var h = {};
   var maxNum = 0;
   var maxEle = null;
   for (var i = 0; i < arr.length; i++) {
       var a = arr[i];
       h[a] === undeﬁned ? h[a] = 1 : (h[a]++);
       // 在当前循环中直接比较出现次数最大值
       if (h[a] > maxNum) {
           maxEle = a;
           maxNum = h[a];
       }
   }
   return '出现次数最多的元素为:' + maxEle + '，出现次数为:' + maxNum;
}
```

**3、借助Array类型的reduce()函数**

主要思想是使用Array类型的reduce()函数，优先设置初始的出现次数最大值maxNum为1，设置initialValue为一个空对象{}，每次处理中优先计算当前元素出现的次数，在每次执行完后与maxNum进行比较，动态更新maxNum与maxEle的值，最后获得返回的结果

```js
function ﬁndMost3(arr) {
   var maxEle;
   var maxNum = 1;
   var obj = arr.reduce(function (p, k) {
       p[k] ? p[k]++ : p[k] = 1;
       if (p[k] > maxNum) {
          maxEle = k;
          maxNum++;
       }
       return p;
   }, {});
   return '次数最多的元素为:' + maxEle + '，次数为:' + obj[maxEle];
}
```

### 2.3 Date类型

#### 2.3.1 日期格式化
##### 2.3.1.1  基于严格的时间格式解析

```js
/**
  * 方法2
  * @description 对Date的扩展，将 Date 转换为指定格式的String
  *  月(M)、日(d)、小时(H)、分(m)、秒(s)、季度(q) 可以用 1~2 个占位符，
  *  年(y)可以用 1~4 个占位符，毫秒(S)只能用 1 个占位符(是 1~3 位的数字)
  * @param fmt
  * @example    * (new Date()).format("yyyy-MM-dd HH:mm:ss") // 2018-07-31 20:09:04
  * (new Date()).format("yyyy-M-d H:m")  // 2018-07-31 20:09
  * @returns {*}
  */
Date.prototype.format = function (fmt) {
   var o = {
       "M+": this.getMonth() + 1, //月份
       "d+": this.getDate(), //日
       "H+": this.getHours(), //小时
       "m+": this.getMinutes(), //分
       "s+": this.getSeconds(), //秒
       "q+": Math.ﬂoor((this.getMonth() + 3) / 3), //季度
       "S": this.getMilliseconds() //毫秒
   };
   if (/(y+)/.test(fmt))
  fmt = fmt.replace(RegExp.$1, (this.getFullYear() + "").substr(4 -
RegExp.$1.length));
   for (var k in o)
       if (new RegExp("(" + k + ")").test(fmt))
         fmt = fmt.replace(RegExp.$1, (RegExp.$1.length == 1) ? (o[k]) :
(("00" + o[k]).substr(("" + o[k]).length)));
   return fmt;
};


var d = new Date();
console.log(d.format('yyyy-MM-dd HH:mm:ss.S')); // 2017-11-26 14:46:13.894
console.log(d.format('yyyy-MM-dd')); // 2017-11-26
console.log(d.format('yyyy-MM-dd q HH:mm:ss')); // 2017-11-26 4 14:46:13
```

#### 2.3.2 日期合法性校验

**校验日期合法性的主要思想是利用正则表达式，将正则表达式按分组处理，匹配到不同位置的数据后，得到一个数组。利用数组的数据构造一个Date对象，获得Date对象的年、月、日的值，再去与数组中表示年、月、日的值比较。如果都相等的话则为合法的日期，如果不相等的话则为不合法的日期。**

```js
function validateDate(str) {
   var reg = /^(\d+)-(\d{1,2})-(\d{1,2})$/;
   var r = str.match(reg);
   if (r == null) return false;
   r[2] = r[2] - 1;
   var d = new Date(r[1], r[2], r[3]);
   if (d.getFullYear() != r[1]) return false;
   if (d.getMonth() != r[2]) return false;
   if (d.getDate() != r[3]) return false;
   return true;
}

console.log(validateDate ('2018-08-20'));  // true
console.log(validateDate ('2018-08-40'));  // false
```

#### 2.3.3 日期计算

##### 2.3.3.1 比较日期大小

```js
function CompareDate(dateStr1, dateStr2) {
   var date1 = dateStr1.replace(/-/g, "\/");
   var date2 = dateStr2.replace(/-/g, "\/");
   return new Date(date1) > new Date(date2);
}

var dateStr1 = "2018-07-30 7:31";
var dateStr2 = "2018-07-31 7:30";
var dateStr3 = "2018-08-01 17:31";
var dateStr4 = "2018-08-01 17:30";
CompareDate(dateStr1, dateStr2);  // false
CompareDate(dateStr3, dateStr4);  // true
```

##### 2.3.3.2 计算当前日期前后N天的日期

```js
function GetDateStr(AddDayCount) {
   var dd = new Date();
   dd.setDate(dd.getDate() + AddDayCount);  //获取AddDayCount天后的日期
     var y = dd.getFullYear();
    //获取当前月份的日期，不足10补0
   var m = (dd.getMonth() + 1) < 10 ? "0" + (dd.getMonth() + 1) : (dd.getMonth() + 1);
   var d = dd.getDate() < 10 ? "0" + dd.getDate() : dd.getDate();  //获取当前几号，//不足10补0
   return y + "-" + m + "-" + d;
}

console.log("半年前："+GetDateStr(-180)); // 半年前：2018-02-02
console.log("三月前："+GetDateStr(-90));  // 三月前：2018-05-03
console.log("一月前："+GetDateStr(-30));  // 一月前：2018-07-02
console.log("昨天："+GetDateStr(-1));     // 昨天：2018-07-31
console.log("今天："+GetDateStr(0));      // 今天：2018-08-01
console.log("明天："+GetDateStr(1));      // 明天：2018-08-02
console.log("后天："+GetDateStr(2));      // 后天：2018-08-03
console.log("一月后："+GetDateStr(30));   // 一月后：2018-08-31
console.log("三月后："+GetDateStr(90));   // 三月后：2018-10-30
console.log("半年后："+GetDateStr(180));  // 半年后：2019-01-28
```

##### 2.3.3.3 计算两个日期的事件差

计算两个日期的时间差的主要思路如下。

· 将传入的时间字符串中的“-”分隔符转换为“/”。

· 将转换后的字符串构造成新的Date对象。

· 以毫秒作为最小的处理单位，然后根据处理维度，进行相应的描述计算。例如天换算成毫秒，就为“1000 * 3600 * 24”。

· 两个时间都换算成秒后，进行减法运算，与维度值相除即可得到两个时间的差值。

```js
function GetDateDiﬀ(startTime, endTime, diﬀType) {
   // 将yyyy-MM-dd的时间格式转换为yyyy/MM/dd的时间格式
   startTime = startTime.replace(/\-/g, "/");
   endTime = endTime.replace(/\-/g, "/");
   // 将计算间隔类性字符转换为小写
   diﬀType = diﬀType.toLowerCase();
   var sTime = new Date(startTime);  // 开始时间
   var eTime = new Date(endTime);  // 结束时间
   //作为除数的数字
   var divNum = 1;
   switch (diﬀType) {
       case "second":
          divNum = 1000;
          break;
       case "minute":
          divNum = 1000 * 60;
          break;
       case "hour":
          divNum = 1000 * 3600;
          break;
                 case "day":
          divNum = 1000 * 3600 * 24;
          break;
       default:
          break;
   }
   return parseInt((eTime.getTime() - sTime.getTime()) / parseInt(divNum));
}

var result1 = GetDateDiﬀ("2018-07-30 18:12:34", '2018-08-01 9:17:30', "day");
var result2 = GetDateDiﬀ("2018-07-29 20:56:34", '2018-08-01 9:17:30', "hour");
console.log("两者时间差为：" + result1 + "天。");  // 1天
console.log("两者时间差为：" + result2 + "小时。"); // 60小时
```

## 3、函数
在JavaScript中，要说理解起来，难度最大的就是函数了。函数中包括作用域、原型链、闭包等核心知识点

· 函数的定义与调用。

· 函数参数。

· 构造函数。

· 变量提升与函数提升。

· 闭包。

· this使用详解。

· call()函数、apply()函数、bind()函数的使用与区别。

### 3.1 函数的定义和调用

#### 3.1.1 函数的定义

**1、 函数声明**

```js
function sum (num1, num2){
  return num1 + num2;
}
```

**2、函数表达式**

```js
// 函数表达式
var sum = function (num1, num2) {
   return num1 + num2;
};
```

 或者

```js
// 具有函数名的函数表达式
var sum = function foo(num1, num2) {
   return num1 + num2;
};
```

其中foo是函数名称，它实际是函数内部的一个局部变量，在函数外部是无法直接调用的，示例如下。

```js
console.log(foo(1, 3)); // ReferenceError: foo is not deﬁned
```

在调用foo时，会直接抛出foo未定义的异常。


**3、Function()构造函数**

```js
// 其中的参数，除了最后一个参数是执行的函数体，其他参数都是函数的形参。
var add = new Function("a", "b", "return a + b");
```

相比于函数声明和函数表达式这两种方式，Function()构造函数的使用比较少，主要有以下两个原因。

第一个原因是Function()构造函数每次执行时，都会解析函数主体，并创建一个新的函数对象，所以当在一个循环或者频繁执行的函数中调用Function()构造函数时，效率是非常低的。

第二个原因是使用Function()构造函数创建的函数，并不遵循典型的作用域，它将一直作为顶级函数执行。所以在一个函数A内部调用Function()构造函数时，其中的函数体并不能访问到函数A中的局部变量，而只能访问到全局变量。


**4、函数表达式的应用场景**

**（1）函数递归**

**（2）代码模块化**

```js
var person = (function () {
   var _name = "";
   return {
      getName: function () {
          return _name;
      },
      setName: function (newName) {
          _name = newName;
      }
   };
}());
person.setName('kingx');
person.getName();   // 'kingx'
```

**5、函数声明与函数表达式的区别**

**（1）函数名称**

在使用函数声明时，是必须设置函数名称的，这个函数名称相当于一个变量，以后函数的调用也会通过这个变量进行。

而对于函数表达式来说，函数名称是可选的，我们可以定义一个匿名函数表达式，并赋给一个变量，然后通过这个变量进行函数的调用。

```js
// 函数声明，函数名称sum必须设置
function sum(num1, num2) {
   return num1 + num2;
}
// 没有函数名称的匿名函数表达式
var sum = function (num1, num2) {
   return num1 + num2;
};

// 具有函数名的函数表达式，其中foo为函数名称
var sum = function foo(num1, num2) {
   return num1 + num2;
};
```

**（2）函数提升**

对于函数声明，存在函数提升，所以即使函数的调用在函数的声明之前，仍然可以正常执行。

对于函数表达式，不存在函数提升，所以在函数定义之前，不能对其进行调用，否则会抛出异常。

#### 3.1.2 函数的调用

**函数的调用存在5种模式，分别是函数调用模式，方法调用模式，构造器调用模式，call()函数、apply()函数调用模式，匿名函数调用模式。这5种模式在使用时都有很明显的特征**

**1、函数调用模式**
待写

**2、方法调用模式**

方法调用模式会优先定义一个对象obj，然后在对象内部定义值为函数的属性property，通过对象obj.property()来进行函数的调用

```js
// 定义对象
var obj = {
   name: 'kingx',
   // 定义getName属性，值为一个函数
   getName: function () {
      return this.name;
   }
};
obj.getName();  // 通过对象进行调用
```

**3、构造器调用模式**

构造器调用模式会定义一个函数，在函数中定义实例属性，在原型上定义函数，然后通过new操作符生成函数的实例，再通过实例调用原型上定义的函数。

```js
// 定义函数对象
function Person(name) {
   this.name = name;
}
// 原型上定义函数
Person.prototype.getName = function () {
   return this.name;
};
// 通过new操作符生成实例
var p = new Person('kingx');
// 通过实例进行函数的调用
p.getName();
```

**4、call()函数、apply()函数调用模式**

通过call()函数或者apply()函数可以改变函数执行的主体，使得某些不具有特定函数的对象可以直接调用该特定函数。

```js
// 定义一个函数
function sum(num1, num2) {
   return num1 + num2;
}
// 定义一个对象
var person = {};
// 通过call()函数与apply()函数调用sum()函数
sum.call(person, 1, 2);
sum.apply(person, [1, 2]);
```

**5、匿名函数调用模式**

模式一

```js
// 通过函数表达式定义匿名函数，并赋给变量sum
var sum = function(num1, num2){
   return num1 + num2;
};
// 通过sum()函数进行匿名函数调用
sum(1, 2);
```

模式二

```js
(function (num1, num2) {
   return num1 + num2;
})(1, 2);  // 3
```
需要注意的是，如果前半部分的函数声明没有使用小括号括住，则直接进行函数的调用时，会抛出语法异常。例如：

```js
// 因为JavaScript解释器在解析语句时，会将function关键字当作函数声明的开始，
// 函数的声明是需要有函数名称的，而上面的代码却并没有函数名称，所以会抛出语法异常。
function (num1, num2) {
   return num1 + num2;
}(1, 2);   // Uncaught SyntaxError: Unexpected token (


// 不会报错
function sum(num1, num2) {
   console.log(num1 + num2);
}(1, 2);

// 立即执行 (使用小括号将整个语句全部括起来，当作一个完整的函数表达式调用)
(function sum(num1, num2) {
   console.log(num1 + num2);
}(1, 2));

```

#### 3.1.3 自执行函数

自执行函数即函数定义和函数调用的行为先后连续产生。它需要以一个函数表达式的身份进行函数调用，上面的匿名函数调用也属于自执行函数的一种。

接下来我们一起看看自执行函数的多种表现形式。

```js
function (x) {
   alert(x);
}(5); // 抛出异常，Uncaught SyntaxError: Unexpected token (

var aa = function (x) {
   console.log(x);
}(1); // 1

true && function (x) {
   console.log(x);
}(2); // 2

0, function (x) {
   console.log(x);
}(3); // 3

!function (x) {
   console.log(x);
}(4); // 4

~function (x) {
   console.log(x);
}(5); // 5

-function (x) {
   console.log(x);
}(6); // 6

+function (x) {
   console.log(x);
}(7); // 7

new function (){
   console.log(8); // 8
};

new function (x) {
   console.log(x);
}(9); // 9
```

### 3.2 函数参数

#### 3.2.1 形参和实参

形参和实参的区别有以下几点。

1、形参出现在函数的定义中，只能在函数体内使用，一旦离开该函数则不能使用；实参出现在主调函数中，进入被调函数后，实参也将不能被访问

```js
function fn1() {
   var param = 'hello';
   fn2(param);
   console.log(arg);   // 在主调函数中不能访问到形参arg，会抛出异常
}
function fn2(arg) {
   console.log(arg);   // 在函数体内能访问到形参arg，输出“hello”
   console.log(param); // 在函数体内不能访问到实参param，会抛出异常
}
fn1();
```

2、在强类型语言中，定义的形参和实参在数量、数据类型和顺序上要保持严格一致，否则会抛出“类型不匹配”的异常。

3、在函数调用过程中，数据传输是单向的，即只能把实参的值传递给形参，而不能把形参的值反向传递给实参。因此在函数执行时，形参的值可能会发生变化，但不会影响到实参中的值。

```js
var arg = 1;
function fn(param) {
   param = 2;
}
fn(arg);
console.log(arg); // 输出“1”，实参arg的值仍然不变
```

4、当实参是基本数据类型的值时，实际是将实参的值复制一份传递给形参，在函数运行结束时形参被释放，而实参中的值不会变化。当实参是引用类型的值时，实际是将实参的内存地址传递给形参，即实参和形参都指向相同的内存地址，此时形参可以修改实参的值，但是不能修改实参的内存地址。

下面的代码段定义了一个实参arg为一个对象，name属性值为kingx，在调用fn()函数时，首先修改了形参param的name属性值为kingx2，此时形参param与实参arg指向的是同一个内存地址（假设为A），因此arg的值也会发生变化。

然后将形参param重新赋值为一个空对象，表示的是将形参param指向了一个新的内存地址（假设为B），但是这并不会影响实参arg的值，它仍然指向原来的内存地址A，因此最后的输出结果为“{name: "kingx2"}”。

```js
var arg = {name: 'kingx'};
function fn(param) {
   param.name = 'kingx2';
   param = {};
}
fn(arg);
console.log(arg); // {name: "kingx2"}
```

#### 3.2.2 arguments对象性质

arguments对象是所有函数都具有的一个内置局部变量，表示的是函数实际接收的参数，是一个类数组结构。

**之所以说arguments对象是一个类数组结构，是因为它除了具有length属性外，不具有数组的一些常用方法。**

下面会分析arguments对象所具有的性质。

**1、可通过索引访问**

```js
function sum(num1, num2) {
   console.log(arguments[0]);  // 3
   console.log(arguments[1]);  // 4
   console.log(arguments[2]);  // undeﬁned
}

sum(3, 4);
```

**2、由实参决定**

arguments对象的值由实参决定，而不是由定义的形参决定，形参与arguments对象占用独立的内存空间。

· arguments对象的length属性在函数调用的时候就已经确定，不会随着函数的处理而改变。

· 指定的形参在传递实参的情况下，arguments对象与形参值相同，并且可以相互改变。

· 指定的形参在未传递实参的情况下，arguments对象对应索引值返回“undefined”。

· 指定的形参在未传递实参的情况下，arguments对象与形参值不能相互改变。

```js
function foo(a, b, c) {
   console.log(arguments.length);  // 2

   arguments[0] = 11;
   console.log(a);   // 11

   b = 12;
   console.log(arguments[1]);  // 12

   arguments[2] = 3;
   console.log(c);  // undeﬁned

   c = 13;
   console.log(arguments[2]);  // 3

   console.log(arguments.length);  // 2
}

foo(1, 2);
```

**3、特殊的arguments.callee属性**

arguments对象有一个很特殊的属性callee，表示的是当前正在执行的函数，在比较时是严格相等的。

```js
function create() {
   return function (n) {
       if (n <= 1)
           return 1;
       return n * arguments.callee(n - 1);
   };
}
var result = create()(5); // returns 120 (5 * 4 * 3 * 2 * 1)
```

尽管arguments.callee属性可以用于获取函数本身去做递归调用，但是我们并不推荐广泛使用arguments.callee属性，其中有一个主要原因是使用arguments.callee属性后会改变函数内部的this值

```js
var sillyFunction = function (recursed) {
   if (!recursed) {
       console.log(this);  // Window {}
       return arguments.callee(true);
   }
   console.log(this);  // Arguments {}
};
sillyFunction();
```

#### 3.2.3 arguments对象的应用

1、实参的个数判断

```js
function f(x, y, z) {
   // 检查传递的参数个数是否正确
   if (arguments.length !== 3) {
       throw new Error("期望传递的参数个数为3，实际传递个数为" + arguments.length);
   }
   // ...do something
}
f(1, 2); // Uncaught Error: 期望传递的参数个数为3，实际传递个数为2
```

2、任意个数的参数处理

```js
function joinStr(seperator) {
   // arguments对象是一个类数组结构，可以通过call()函数间接调用slice()函数，得到一个数组
   var strArr = Array.prototype.slice.call(arguments, 1);
   // strArr数组直接调用join()函数
   return strArr.join(seperator);
}

joinStr('-', 'orange', 'apple', 'banana'); // orange-apple-banana
joinStr(',', 'orange', 'apple', 'banana'); // orange,apple,banana
```

3、模拟函数重载

**函数重载表示的是在函数名相同的情况下，通过函数形参的不同参数类型或者不同参数个数来定义不同的函数**

我们都知道在JavaScript中是没有函数重载的，主要有以下几点原因。

· JavaScript是一门弱类型的语言，变量只有在使用时才能确定数据类型，通过形参是无法确定数据类型的。

· 无法通过函数的参数个数来指定调用不同的函数，函数的参数个数是在函数调用时才确定下来的。

· 使用函数声明定义的具有相同名称的函数，后者会覆盖前者。

那么遇到这种情况，我们该如何写出一个通用的函数，来实现任意个数字的加法运算求和呢？

答案就是使用arguments对象处理传递的参数。

首先通过call()函数间接调用数组的slice()函数以得到函数参数的数组；

然后调用数组的reduce()函数进行多个值的求和并返回。

```js
// 通用求和函数
function sum() {
   // 通过call()函数间接调用数组的slice()函数得到函数参数的数组
   var arr = Array.prototype.slice.call(arguments);
   // 调用数组的reduce()函数进行多个值的求和
   return arr.reduce(function (pre, cur) {
       return pre + cur;
   }, 0)
}

sum(1, 2);       // 3
sum(1, 2, 3);    // 6
sum(1, 2, 3, 4); // 10
```

### 3.3 构造函数

在函数中存在一类比较特殊的函数——构造函数。当我们创建对象的实例时，通常会使用到构造函数，例如对象和数组的实例化可以通过相应的构造函数Object()和Array()完成。

构造函数与普通函数在语法的定义上没有任何区别，主要的区别体现在以下3点。

**1、 构造函数的函数名的第一个字母通常会大写。**

**2、 在函数体内部使用this关键字，表示要生成的对象实例，构造函数并不会显式地返回任何值，而是默认返回“this”。**

**3、 作为构造函数调用时，必须与new操作符配合使用。**

```js
// 构造函数
function Person(name, age) {
   this.name = name;
   this.age = age;
   this.sayName = function () {
       alert(this.name);
   };
}
var person = new Person('kingx', '12');
person.sayName(); // 'kingx'
```

**一个函数在当作普通函数使用时，函数内部的this会指向window。**

```js
Person('kingx', '12');
window.sayName(); // 'kingx'
```

**使用构造函数可以在任何时候创建我们想要的对象实例，构造函数在执行时会执行以下4步：**

**1、 通过new操作符创建一个新的对象，在内存中创建一个新的地址。**

**2、 为构造函数中的this确定指向。**

**3、 执行构造函数代码，为实例添加属性。**

**4、 返回这个新创建的对象。**

以前面生成person实例的代码为例。

第一步：为person实例在内存中创建一个新的地址。

第二步：确定person实例的this指向，指向person本身。

第三步：为person实例添加name、age和sayName属性，其中sayName属性值是一个函数。

第四步：返回这个person实例。


```js
var person1 = new Person();
var person2 = new Person();
console.log(person1.sayName === person2.sayName); // false
```

**事实上，当我们在创建对象的实例时，对于相同的函数并不需要重复创建，而且由于this的存在，总是可以在实例中访问到它具有的属性。**因此，我们需要使用一种更好的方式来处理函数类型的属性。大家可能会想到设置全局访问的函数，这样就可以被所有实例访问到，而不用重复创建。

但是这样也会存在一个问题，如果为一个对象添加的所有函数都处理成全局函数，这样会污染到全局作用域空间，而且也无法完成对一个自定义类型对象的属性和函数的封装，因此这不是一个好的解决办法。

**这里就要引入原型的概念了!!!**


### 3.4 变量提升与函数提升

在JavaScript中，会存在一些比较奇怪的现象。例如，一个函数体内，变量在定义之前就可以被访问到，而不会抛出异常。

```js
function fn() {
   console.log(a); // 输出“undeﬁned”，不会抛出异常
   var a = 1;
}
```

```js
fn();  // 函数正常执行，输出“函数得到调用”
function fn() {
   console.log('函数得到调用');
}
```

#### 3.4.1 作用域

**在JavaScript中，一个变量的定义与调用都是会在一个固定的范围中的，这个范围我们称之为作用域。作用域可以分为：全局作用域、函数作用域和块级作用域。**

需要注意的是块级作用域是在ES6中新增的，需要使用特定的let或者const关键字定义变量。

```js
// 全局作用域内的变量a
var a = 'global variable';

function foo() {
   // 函数作用域内的变量b
   var b = 'function variable';

   console.log(a);  // global variable
   console.log(b);  // function variable
}

// 块级作用域内的变量c
{
   let c = 'block variable';
   console.log(c);  // block variable
}

console.log(c);  // Uncaught ReferenceError: c is not deﬁned
```

#### 3.4.2 变量提升

变量提升是将变量的声明提升到函数顶部的位置，而变量的赋值并不会被提升。

需要注意的一点是，会产生提升的变量必须是通过var关键字定义的，而不通过var关键字定义的全局变量是不会产生变量提升的。

通过下面的代码可以发现，变量v的定义未使用var关键字，那么它是一个全局变量，不会产生变量提升，直接进行输出，抛出一个变量v未定义的异常。

```js
(function () {
   console.log(v);  // Uncaught ReferenceError: v is not deﬁned
   v = 'Hello JavaScript';
})();
```

**1、代码段1的执行过程**

在全局对象window上定义一个变量v，并赋值为Hello World。然后定义一个立即执行函数，这个立即执行函数的作用域为window。在函数内部引用变量v，然后会顺着作用域寻找，最终会在window上找到这个变量v，因此输出“Hello World”。

**2、代码段2的执行过程**

代码段2中出现了变量提升，在立即执行函数的内部，变量v的定义会提升到函数顶部，实际执行过程的代码如下所示。

```js
var v = 'Hello World';

(function () {
  var v;   // 变量的声明得到提升
  console.log(v);
  v = 'Hello JavaScript';  // 变量的赋值并未提升
})();
```

同代码段1的分析，在window上定义一个变量v，赋值为Hello World，而且在立即执行函数的内部同样定义了一个变量v，但是赋值语句并未提升，因此v为undefined。在输出时，会优先在函数内部作用域中寻找变量，而变量已经在内部作用域中定义，因此直接输出“undefined”。


#### 3.4.3 函数提升

不仅通过var定义的变量会出现提升的情况，使用函数声明方式定义的函数也会出现提升，

```js
show();  // 你好
var show;

// 函数声明，会被提升
function show() {
   console.log('你好');
}

// 函数表达式，不会被提升
show = function () {
   console.log('hello');
};
```

#### 3.4.4 变量提升与函数提升的应用

**1、关于函数提升**

```js
function foo() {
   function bar() {
      return 3;
   }

   return bar();

   function bar() {
       return 8;
   }
}
console.log(foo()); // 8
```

由于变量提升的存在，两段代码都会被提升至foo()函数的顶部，而且后一个函数会覆盖前一个bar()函数，因此最后输出值为“8”。

**2、变量提升和函数提升同时使用**

```js
var a = true;
foo();

function foo() {
   if(a) {
      var a = 10;
   }
   console.log(a); // undeﬁned
}
```

在foo()函数内部，首先判断变量a的值，由于变量a在函数内部重新通过var关键字声明了一次，因此a会出现变量提升，a会提升至foo()函数的顶部，此时a的值为undefined。那么通过if语句进行判断时，返回“false”，并未执行a = 10的赋值语句，因此最后输出“undefined”。

**3、变量提升和函数提升优先级**

```js
function fn() {
   console.log(typeof foo); // function
   // 变量提升
   var foo = 'variable';
   // 函数提升
   function foo() {
      return 'function';
   }
   console.log(typeof foo); // string
}
fn();
```

同时存在变量提升和函数提升，但是变量提升的优先级要比函数提升的优先级高。 上面的代码执行如下：

```js
function fn() {
   // 变量提升至函数顶部
   var foo;
   // 函数提升，但是优先级低，出现在变量声明后面，则foo是一个函数
   function foo() {
      return 'function';
   }
   console.log(typeof foo);  // function

   foo = 'variable';  // 变量赋值
   console.log(typeof foo); // string
}
fn();
```

**4、变量提升和函数提升整体应用**

理解变量提升和函数提升可以使我们更了解这门语言，更好地驾驭它。但是在开发中，我们不应该使用这些技巧，而是要规范我们的代码，尽可能提高代码的可读性和可维护性。

1. 无论变量还是函数，都做到先声明后使用。
2. 对于ES6语法编写的代码，则全部使用let或者const关键字。

### 3.5 闭包
在正常情况下，如果定义了一个函数，就会产生一个函数作用域，在函数体中的局部变量会在这个函数作用域中使用。一旦函数执行完成，函数所占空间就会被回收，存在于函数体中的局部变量同样会被回收，回收后将不能被访问到。那么如果我们期望在函数执行完成后，函数中的局部变量仍然可以被访问到，这能不能实现呢？

#### 3.5.1 执行上下文环境

JavaScript每段代码的执行都会存在于一个执行上下文环境中，而任何一个执行上下文环境都会存在于整体的执行上下文环境中。**根据栈先进后出的特点，全局环境产生的执行上下文环境会最先压入栈中，存在于栈底。当新的函数进行调用时，会产生的新的执行上下文环境，也会压入栈中。当函数调用完成后，这个上下文环境及其中的数据都会被销毁，并弹出栈，从而进入之前的执行上下文环境中。**

**需要注意的是，处于活跃状态的执行上下文环境只能同时有一个，即图所示的深色背景的部分。**

<img src="/img/ctx.jpeg"  alt="执行栈" height = "auto"/>

```js
  var a = 10;    // 1.进入全局执行上下文环境
  var fn = function (x) {
      var c = 10;
      console.log(c + x);
  };
  var bar = function (y) {
      var b = 5;
      fn(y + b);  // 3.进入fn()函数执行上下文环境
  };
 bar(20);  // 2.进入bar()函数执行上下文环境
```

**有另外一种情况，虽然代码执行完毕，但执行上下文环境却被无法干净地销毁，这就是我们要讲到的闭包。**

#### 3.5.2 闭包的概念

在JavaScript中存在一种内部函数，即函数声明和函数表达式可以位于另一个函数的函数体内，在内部函数中可以访问外部函数声明的变量，当这个内部函数在包含它们的外部函数之外被调用时，就会形成闭包。

闭包有两个很明显的特点。

· 函数拥有的外部变量的引用，在函数返回时，该变量仍然处于活跃状态。

· 闭包作为一个函数返回时，其执行上下文环境不会被销毁，仍处于执行上下文环境中。


```js
function fn() {
   var max = 10;
   return function bar(x)
         if (x > max) {
            console.log(x);
         }
   };
}
var f1 = fn();
f1(11);  // 11
```

当代码执行到第10行时，调用f1()函数，注意此时是一个关键的节点，因为f1()函数中包含了对max变量的引用，而max变量是存在于外部函数fn()中的，此时fn()函数执行上下文环境并不会被直接销毁，依然存在于执行上下文环境中。

等到第10行代码执行结束后，bar()函数执行完毕，bar()函数执行上下文环境才会被销毁，同时因为max变量引用会被释放，fn()函数执行上下文环境也一同被销毁。

最后全局上下文环境执行完毕，栈被清空，流程执行结束。

**从分析就可以看出闭包所存在的最大的一个问题就是消耗内存，如果闭包使用越来越多，内存消耗将越来越大**

#### 3.5.3 闭包的用途

##### 3.5.3.1 结果缓存

在开发过程中，我们可能会遇到这样的场景，假如有一个处理很耗时的函数对象，每次调用都会消耗很长时间。

我们可以将其处理结果在内存中缓存起来。这样在执行代码时，如果内存中有，则直接返回；如果内存中没有，则调用函数进行计算，更新缓存并返回结果。

因为闭包不会释放外部变量的引用，所以能将外部变量值缓存在内存中。

```js
var cachedBox = (function () {
   // 缓存的容器
   var cache = {};
   return {
       searchBox: function (id) {
           // 如果在内存中，则直接返回
           if(id in cache) {
              return '查找的结果为:' + cache[id];
           }
           // 经过一段很耗时的dealFn()函数处理
           var result = dealFn(id);
           // 更新缓存的结果
           cache[id] = result;
           // 返回计算的结果
           return '查找的结果为:' + result;
       }
   };
})();

// 处理很耗时的函数
function dealFn(id) {
   console.log('这是一段很耗时的操作');
   return id;
}

// 两次调用searchBox()函数
console.log(cachedBox.searchBox(1)); // 第一次显示：这是一段很耗时的操作
console.log(cachedBox.searchBox(1)); // 第二次： 直接显示结果
```

##### 3.5.3.2 封装

在JavaScript中提倡的模块化思想是希望将具有一定特征的属性封装到一起，只需要对外暴露对应的函数，并不关心内部逻辑的实现。

```js
var stack = (function () {
   // 使用数组模仿栈的实现
   var arr = [];
   // 栈
   return {
       push: function (value) {
           arr.push(value);
       },
       pop: function () {
           return arr.pop();
       },
       size: function () {
           return arr.length;
       }
   };
})();
stack.push('abc');
stack.push('def');
console.log(stack.size());  // 2
stack.pop();
console.log(stack.size());  // 1
```

接下来我们将通过几道练习题加深大家对闭包的理解。

**1、ul中有若干个li，每次单击li，输出li的索引值**

```js
<ul>
   <li>1</li>
   <li>2</li>
   <li>3</li>
   <li>4</li>
   <li>5</li>
</ul>
<script>
   var lis = document.getElementsByTagName('ul')[0].children;
   for (var i = 0; i < lis.length; i++) {
       lis[i].onclick = function () {
           console.log(i);
       };
   }
</script>
```

但是真正运行后却发现，结果并不如自己所想，每次单击后输出的并不是索引值，而一直都是“5”。

**这是为什么呢？因为在我们单击li，触发li的click事件之前，for循环已经执行结束了，而for循环结束的条件就是最后一次i++执行完毕，此时i的值为5，所以每次单击li后返回的都是“5”。**

```js
<script>
   var lis = document.getElementsByTagName('ul')[0].children;
   for (var i = 0; i < lis.length; i++) {
       (function (index) {
           lis[index].onclick = function () {
               console.log(index);
           };
       })(i);
   }
</script>
```

**在每一轮的for循环中，我们将索引值i传入一个匿名立即执行函数中，在该匿名函数中存在对外部变量lis的引用，因此会形成一个闭包。而闭包中的变量index，即外部传入的i值会继续存在于内存中，所以当单击li时，就会输出对应的索引index值。**

**2、定时器问题**

**定时器setTimeout()函数和for循环在一起使用**，总会出现一些意想不到的结果，我们看看下面的代码。

```js
var arr = ['one', 'two', 'three'];
for(var i = 0; i < arr.length; i++) {
   setTimeout(function () {
       console.log(arr[i]);
   }, i * 1000);
}
```

但是运行过后，我们却会发现结果是每隔一秒输出一个“undefined”，这是为什么呢？

通过闭包可以解决这个问题，代码如下所示。

```js
var arr = ['one', 'two', 'three'];
for(var i = 0; i < arr.length; i++) {
   (function (time) {
       setTimeout(function () {
           console.log(arr[time]);
       }, time * 1000);
   })(i);
}
```

**通过立即执行函数将索引i作为参数传入，在立即函数执行完成后，由于setTimeout()函数中有对arr变量的引用，其执行上下文环境不会被销毁，因此对应的i值都会存在内存中。**所以每次执行setTimeout()函数时，i都会是数组对应的索引值0、1、2，从而间隔一秒输出“one”“two”“three”。


**3、作用域链问题**

闭包往往会涉及作用域链问题，尤其是包含this属性时。

```js
var name = 'outer';
var obj = {
   name: 'inner',
   method: function () {
       return function () {
           return this.name;
       }
   }
};
console.log(obj.method()());  // outer
```

在调用obj.method()函数时，会返回一个匿名函数，而该匿名函数中返回的是this.name，因为引用到了this属性，在匿名函数中，this相当于一个外部变量，所以会形成一个闭包。

在JavaScript中，this指向的永远是函数的调用实体，而匿名函数的实体是全局对象window，因此会输出全局变量name的值“outer”。

```js
var name = 'outer';
var obj = {
   name: 'inner',
   method: function () {
       // 用_this保存obj中的this
       var _this = this;
       return function () {
           return _this.name;
       }
   }
};
console.log(obj.method()());  // inner
```

**4、多个相同函数名问题**

```js
// 第一个foo()函数
function foo(a, b) {
   console.log(b);
   return {
      // 第二个foo()函数
       foo: function (c) {
          // 第三个foo()函数
          return foo(c, a);
       }
   }
}
var x = foo(0); x.foo(1); x.foo(2); x.foo(3);
var y = foo(0).foo(1).foo(2).foo(3);
var z = foo(0).foo(1); z.foo(2); z.foo(3);
```
在完成这道题目之前，我们需要搞清楚这3个foo()函数的指向。

首先最外层的foo()函数是一个具名函数，返回的是一个具体的对象。

第二个foo()函数是最外层foo()函数返回对象的一个属性，该属性指向一个匿名函数。

第三个foo()函数是一个被返回的函数，该foo()函数会沿着原型链向上查找，而foo()函数在局部环境中并未定义，最终会指向最外层的第一个foo()函数，因此第三个和第一个foo()函数实际是指向同一个函数。

第一行输出结果为“undefined，0，0，0”。

第二行输出结果为“undefined，0，1，2”。

第三行输出结果为“undefined，0，1，1”。


#### 3.5.4 小结

##### 3.5.4.1 闭包的优点

· 保护函数内变量的安全，实现封装，防止变量流入其他环境发生命名冲突，造成环境污染。

· 在适当的时候，可以在内存中维护变量并缓存，提高执行效率。

##### 3.5.4.2 闭包的缺点

· 消耗内存：通常来说，函数的活动对象会随着执行上下文环境一起被销毁，但是，由于闭包引用的是外部函数的活动对象，因此这个活动对象无法被销毁，这意味着，闭包比一般的函数需要消耗更多的内存。

· 泄漏内存：在IE9之前，如果闭包的作用域链中存在DOM对象，则意味着该DOM对象无法被销毁，造成内存泄漏。

### 3.6 this使用

### 3.7 call、apply、bind函数的使用和区别

## 4、对象

## 5、DOM与事件

待补充


## 6、Ajax

待补充


## 7、ES6