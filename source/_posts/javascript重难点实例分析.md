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



### 1.3 String类型
