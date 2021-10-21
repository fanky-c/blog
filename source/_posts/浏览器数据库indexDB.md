---
title: 浏览器数据库IndexedDB
date: 2021-09-29 17:06:52
tags:
 - IndexedDB
---

### IndexedDB介绍
#### 概念
IndexedDB是浏览器提供的本地数据库， 它可以被网页脚本创建和操作

#### 特点
1. 键值对存储。 所有的数据都可以存储， 每一个数据都有主键， 主键是唯一的、不会重复
2. 异步。 IndexedDB操作不会锁死浏览器，用户依然可以进行其他操作； 和localstorage对比， 后者是同步的
3. 支持事务。 意味一系列操作之中， 只要有一步失败， 整个事务就会取消， 数据库回滚到之前的状态， 不存在只改写一部分数据
4. 同源策略。 受同源策略限制， 每一个数据库都有对应创建它的域名， 网页只能访问自身域名下数据库

#### IndexedDB基本概念
1. 数据库 -- IDBDatabase 对象。 每个域名（严格的说，是协议 + 域名 + 端口）都可以新建任意多个数据库；同一个时刻，只能有一个版本的数据库存在。如果要修改数据库结构（新增或删除表、索引或者主键），只能通过升级数据库版本完成  
2. 对象仓库 -- IDBObjectStore 对象。每个数据库包含若干个对象仓库（object store）。它类似于关系型数据库的表
3. 索引 -- IDBIndex 对象
4. 事务 -- IDBTransaction 对象
5. 操作请求 -- IDBRequest 对象
6. 指针 -- IDBCursor 对象
7. 主键集合 -- IDBKeyRange 对象

### 操作流程
#### 1. 打开数据库
```js
var request = window.indexedDB.open(databaseName, version);

request.onerror = function (event) {
  console.log('数据库打开报错');
};

request.onsuccess = function (event) {
  db = request.result;
  console.log('数据库打开成功');
};

// 如果指定的版本号，大于数据库的实际版本号，就会发生数据库升级事件
request.onupgradeneeded = function (event) {
  db = event.target.result;
};
```

#### 2. 新建数据库
```js
request.onupgradeneeded = function (event) {
  db = event.target.result;
  var objectStore;
  if (!db.objectStoreNames.contains('person')) {
    objectStore = db.createObjectStore('person', { keyPath: 'id' });
  }
}
```

#### 3. 新增数据
新增数据指的是向对象仓库写入数据记录。这需要通过事务完成

```js
function add() {
  var request = db.transaction(['person'], 'readwrite')
    .objectStore('person')
    .add({ id: 1, name: '张三', age: 24, email: 'zhangsan@example.com' });

  request.onsuccess = function (event) {
    console.log('数据写入成功');
  };

  request.onerror = function (event) {
    console.log('数据写入失败');
  }
}

add()
```


#### 4. 读取数据
读取数据也是通过事务完成。

```js

function read() {
   var transaction = db.transaction(['person']);
   var objectStore = transaction.objectStore('person');
   var request = objectStore.get(1);

   request.onerror = function(event) {
     console.log('事务失败');
   };

   request.onsuccess = function( event) {
      if (request.result) {
        console.log('Name: ' + request.result.name);
        console.log('Age: ' + request.result.age);
        console.log('Email: ' + request.result.email);
      } else {
        console.log('未获得数据记录');
      }
   };
}

read();
```

#### 5. 遍历数据
遍历数据表格的所有记录，要使用指针对象 IDBCursor。
```js
function readAll() {
  var objectStore = db.transaction('person').objectStore('person');

   objectStore.openCursor().onsuccess = function (event) {
     var cursor = event.target.result;

     if (cursor) {
       console.log('Id: ' + cursor.key);
       console.log('Name: ' + cursor.value.name);
       console.log('Age: ' + cursor.value.age);
       console.log('Email: ' + cursor.value.email);
       cursor.continue();
    } else {
      console.log('没有更多数据了！');
    }
  };
}

readAll()
```


#### 6. 更新数据
更新数据要使用IDBObject.put()方法。
```js
function update() {
  var request = db.transaction(['person'], 'readwrite')
    .objectStore('person')
    .put({ id: 1, name: '李四', age: 35, email: 'lisi@example.com' });

  request.onsuccess = function (event) {
    console.log('数据更新成功');
  };

  request.onerror = function (event) {
    console.log('数据更新失败');
  }
}

update();
```


#### 7. 删除数据
IDBObjectStore.delete()方法用于删除记录。
```js
function remove() {
  var request = db.transaction(['person'], 'readwrite')
    .objectStore('person')
    .delete(1);

  request.onsuccess = function (event) {
    console.log('数据删除成功');
  };
}

remove();
```

<br>
[文章来源](https://www.ruanyifeng.com/blog/2018/07/indexeddb.html)