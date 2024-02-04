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

## 2、引用数据类型


## 3、函数


## 4、对象

## 5、DOM与事件


## 6、Ajax


## 7、ES6