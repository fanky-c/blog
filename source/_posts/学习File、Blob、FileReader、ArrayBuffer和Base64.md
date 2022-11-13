---
title: 学习File、Blob、FileReader、ArrayBuffer和Base64
date: 2022-10-24 20:26:34
tags:
- Blob
- FileReader
- ArrayBuffer
- Base64
---

## Blob
Blob 全称为 binary large object ，即二进制大对象。blob对象本质上是js中的一个对象，里面可以储存大量的二进制编码格式的数据。Blob 对象一个不可修改，从Blob中读取内容的唯一方法是使用 FileReader。

### 1. 实例属性和方法
```js
// 属性
1. Blob.prototype.size //只读
2. Blob.prototype.type //只读

// 方法
Blob.prototype.arrayBuffer() // 返回一个 promise，其会兑现一个包含 Blob 所有内容的二进制格式的 ArrayBuffer

Blob.prototype.slice() // 返回一个新的 Blob 对象，包含了源 Blob 对象中指定范围内的数据。

Blob.prototype.stream() // 返回一个能读取 Blob 内容的 ReadableStream。

Blob.prototype.text() // 返回一个 promise，其会兑现一个包含 Blob 所有内容的 UTF-8 格式的字符串。
```

### 2、创建
```js
//  type：默认值为 ''，表示将会被放入到 blob 中的数组内容的 MIME 类型
const obj = {hello: 'world'};
const blob = new Blob(
    [JSON.stringify(obj, null, 2)], 
    {type : 'application/json'}
);
```
<img src="/img/blob1.png" width="90%" />


### 3、读取
```js
const reader = new FileReader();
reader.addEventListener('loadend', () => {
   // reader.result 包含被转化为类型化数组的 blob 中的内容
   console.log(reader.result);
});
reader.readAsText(blob);
```

[Blob相关文档]((https://developer.mozilla.org/zh-CN/docs/Web/API/Blob)


## File
File 对象是特殊类型的 Blob

在 JavaScript 中，主要有两种方法来获取 File 对象：
1. 1）<input\> 元素上选择文件后返回的 FileList 对象；
2. 2）文件拖放操作生成的 DataTransfer 对象；

```js
/*
<input type="file" name="" id="fileId">
<script>
 let file = document.getElementById("fileId");
 file.onchange = function(e){
    console.log(e.target.files);
 }
</script>
*/

// 属性和方法
name: "CI流程.md"
size: 1356
type: "text/markdown"
```

## FileReader
通过上面我都知道了blob是不可修改也是无法读取里面的内容的。无法读取里面的内容肯定是不可行的。所以Filereader就提供了读取blob里面内容的方法。

FileReader 对象允许 Web 应用程序异步读取存储在用户计算机上的文件（或原始数据缓冲区）的内容，
使用 File 或 Blob 对象指定要读取的文件或数据。

### 1、实例属性和方法
```js
let reader = new FileReader();

// 属性
reader.error 

reader.readyState // 0:EMPTY 1:LOADING 2:DONE

reader.result  // 文件的内容。该属性仅在读取操作完成后才有效，数据的格式取决于使用哪个方法来启动读取操作。

// 事件处理
reader.onabort  // 该事件在读取操作被中断时触发。

reader.onload  // 该事件在读取操作完成时触发。 


// 方法
reader.readAsArrayBuffer() //开始读取指定的 Blob中的内容，一旦完成，result 属性中保存的将是被读取文件的 ArrayBuffer 数据对象

reader.readAsText() // 开始读取指定的Blob中的内容。一旦完成，result属性中将包含一个字符串以表示所读取的文件内容。

reader.readAsDataURL() // 开始读取指定的Blob中的内容。一旦完成，result属性中将包含一个data: URL 格式的 Base64 字符串以表示所读取文件的内容。
```

### 2、将文件读取为base64
```js
<input type="file" name="" id="fileId">
<script>
 let file = document.getElementById("fileId");
 file.onchange = function(e){
    let file = e.target.files[0];
    let reader = new FileReader();
    reader.readAsDataURL(file);
    reader.onload = function (e) {
        console.log(e.target.result);
    }
 }
</script>
```

### 3、将blob对象读出来
```js
<script>
  const obj = {hello: 'world'};
  const blob = new Blob(
      [JSON.stringify(obj, null, 2)], 
      {type : 'application/json'}
  );
  let render = new FileReader();
  render.readAsText(blob);
  render.onload = (e) => {
      console.log(e.target.result); //  {"hello": "world"}
  }
</script>
```

[FileReader相关文档](https://developer.mozilla.org/zh-CN/docs/Web/API/FileReader)

## ArrayBuffer
ArrayBuffer 对象用来表示通用的、固定长度的原始二进制数据缓冲区。

它是一个字节数组，通常在其他语言中称为“byte array”。

你不能直接操作 ArrayBuffer 的内容，而是要通过类型数组对象或 DataView 对象来操作，它们会将缓冲区中的数据表示为特定的格式，并通过这些格式来读写缓冲区的内容。

<img src="/img/blob3.png" width="90%" />

### 1、创建buffer
```js
const buffer = new ArrayBuffer(8);
```

### 2、TypedArray读写buffer
```js
let buffer = new ArrayBuffer(8);
let wrapBuffer = new Int8Array(buffer);
```

### 3、DataView读写buffer
```js
let buffer = new ArrayBuffer(8);
let wrapBuffer = new DataView(buffer);
console.log(wrapBuffer.getInt8());
wrapBuffer.setInt(0, 100);
console.log(wrapBuffer.getInt8());
```


[ArrayBuffer相关文档](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/ArrayBuffer)

## Object URL
它是一个用来表示File Object 或Blob Object 的URL
```js
    <input type="file" name="" id="fileId">
    <script>
     let file = document.getElementById("fileId");
     file.onchange = function(e){
       let file = e.target.files[0];
       console.log(URL.createObjectURL(file))
        // blob:http://127.0.0.1:5500/57c6bf38-c83c-4f1b-a936-da9bf00d9ecf
     }
    </script>
```

[URL参考文档](https://developer.mozilla.org/zh-CN/docs/Web/API/URL/URL)

## base64
在 JavaScript 中，有两个函数被分别用来处理解码和编码 base64 字符串：

1. btoa()：编码，从一个字符串或者二进制数据编码一个 Base64 字符串；
2. atob()：解码，解码一个 Base64 字符串。

```js
btoa('java')     // 编码  'amF2YQ=='
atob('amF2YQ==') // 解码  java
```

### 使用场景
1. 将canvas画布内容生成base64的图片；
2. 将获取的图片文件，生成base64图片。

## Blob与其他类型关系
<img src="/img/blob2.png" width="90%" />

<br />
[文章来源](https://blog.csdn.net/qq_35577655/article/details/127169333)
