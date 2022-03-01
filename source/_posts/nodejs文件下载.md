---
title: nodejs文件下载
date: 2022-02-27 15:54:21
tags:
  - 文件下载
  - 流式下载
  - 断点续传
---

### 简单文件下载

**服务器上文件系统已经存在了某个文件，客户端请求下载直接把文件读了吐回去。设置 Content-Disposition 头部为 attachment 是关键，告诉浏览器应该下载这个文件**

>Content-Type: application/octet-stream告诉浏览器这是一个二进制文件。 Content-Disposition告诉浏览器这是一个需要下载的附件并告诉浏览器默认的文件名。
如果不添加Content-Disposition响应头，浏览器可能会下载或显示文件内容，不同浏览器的处理有所不同。

```js
import Koa from 'koa';
import Router from 'koa-router';
import * as fs from 'fs/promises'; // node版本必须在12

const app = new Koa();
const router = new Router();

router.get('/download/simple', async (ctx) => {
  const file = await fs.readFile(`${__dirname}/1.txt`, 'utf-8');
  ctx.set({
    'Content-Disposition': `attachment; filename=1.txt`,
  });
  ctx.body = file;
});

app.use(router.routes());
app.listen(80);
```


### 流式下载（大文件）

**Node无法将大文件一次性读取到进程内存里，这时候用流来解决。 此例子不设置 Content-Disposition 头部也是会下载的，因为 Content-Type 被设置为了 application/octet-stream，浏览器认为其是一个二进制流文件所以默认下载处理了**

```js
router.get('/download/stream', async (ctx) => {
  const file = fs.createReadStream(`${__dirname}/1.txt`);
  ctx.set({
    'Content-Disposition': `attachment; filename=1.txt`,
  });
  ctx.body = file;
});
```

### 下载进度条提示

**大文件下载进度显示，浏览器通过Content-Length来识别**

```js
const { PassThrough } = require('stream');
router.get('/download/progress', async (ctx) => {
  const { enable } = ctx.query;
  const buffer = await fsp.readFile(`${__dirname}/1.txt`);
  const stream = new PassThrough();
  const l = buffer.length;
  const count = 4;
  const size = Math.floor(l / count);
  const writeQuarter = (i = 0) => {
    const start = i * size;
    const end = i === count - 1 ? l : (i + 1) * size;
    stream.write(buffer.slice(start, end));

    if (end === l) {
      stream.end();
    } else {
      // Koa 不再知道文件大小和类型，并将文件分为 4 份，每份间隔 3 秒发送来模拟大文件下载。
      setTimeout(() => writeQuarter(i + 1), 3000);
    }
  };
  
  // 通过参数是否显示进度条，给出剩余下载时间
  if (!!enable) {
    ctx.set({
      'Content-Length': `${l}`,
    });
  }

  ctx.set({
    'Content-Type': 'plain/txt',
    'Content-Disposition': `attachment; filename=1.txt`,
    Connection: 'keep-alive',
  });
  ctx.body = stream;
  writeQuarter();
});
```


### 断点续传

**下载文件特别大时，常常也会因为网络不稳定导致下载中途断开而失败，**

```js
function getStartPos(range = '') {
  var startPos = 0;
  if (typeof range === 'string') {
    var matches = /^bytes=([0-9]+)-$/.exec(range);
    if (matches) {
      startPos = Number(matches[1]);
    }
  }
  return startPos;
}

router.get('/download/partial', async (ctx) => {
  const range = ctx.get('range');
  const start = getStartPos(range);
  const stat = await fsp.stat(`${__dirname}/1.txt`);
  const stream = fs.createReadStream(`${__dirname}/1.txt`, {
    start,
    highWaterMark: Math.ceil((stat.size - start) / 4),
  });

  stream.on('data', (chunk) => {
    console.log(`Readed ${chunk.length} bytes of data.`);
    stream.pause();
    setTimeout(() => {
      stream.resume();
    }, 3000);
  });

  console.log(`Start Pos: ${start}.`);
  if (start === 0) {
    ctx.status = 200;
    ctx.set({
      'Accept-Ranges': 'bytes',
      'Content-Length': `${stat.size}`,
    });
  } else {
    ctx.status = 206;
    ctx.set({
      'Content-Range': `bytes ${start}-${stat.size - 1}/${stat.size}`,
    });
  }

  ctx.set({
    'Content-Type': 'application/octet-stream',
    'Content-Disposition': `attachment; filename=1.txt`,
    Connection: 'keep-alive',
  });
  ctx.body = stream;
});

```

### 动态表格下载

**实际业务问题：根据请求参数条件读取数据库的某张表的全部记录并导出为表格**



<br >
[文章来源](https://mp.weixin.qq.com/s/Jz_Xu6Np38TpdwFVUGNGdA)