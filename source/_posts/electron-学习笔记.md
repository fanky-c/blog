---
title: electron 学习笔记
date: 2018-11-12 11:22:37
tags:
- electron
---

#### 主进程和渲染进程、渲染进程和渲染进程通讯。

##### 主进程和渲染进程通讯。

* 使用 IPC 是很方便的


##### 渲染进程和渲染进程通讯。
* 使用全局共享属性(globald对象和remote.getGlobal方法)。
```
// In the main process.
global.sharedObject = {
  someProperty: 'default value'
}
// In page 1.
require('electron').remote.getGlobal('sharedObject').someProperty = 'new value'
// In page 2.
console.log(require('electron').remote.getGlobal('sharedObject').someProperty)
```

* 利用主进程做消息中转。
```
// In the main process.
ipcMain.on('ping-event', (event, arg) => {
  yourWindow.webContents.send('pong-event', 'something');
}

// In renderer process
// 1
ipcRenderer.send('ping-event', (event, arg) => {
    // do something
  }
)

// 2
ipcRenderer.on('pong-event', (event, arg) => {
    // do something
  }
)
```

* 利用 remote 接口直接获取渲染进程发送消息：
```

```

* ipcRenderer.sendTo接口。
```
ipcRenderer.sendTo(windowId, 'ping', 'someThing')
```