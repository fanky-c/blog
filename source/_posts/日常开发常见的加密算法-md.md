---
title: 日常开发常见的加密算法.md
date: 2024-07-10 20:31:19
tags:
 - AES
 - RSA
 - MD5
 - SHA-3
 - 对称加密算法
 - 非对称加密算法
 - 哈希算法
---

## 一、对称加密
### 1. AES
#### 1.1 介绍
AES（高级加密标准）是一种对称加密算法，即加密和解密使用相同的密钥。它可以加密长度为128、192和256位的数据块，并使用128位的密钥进行加密。AES算法使用了固定的块长度和密钥长度，并且被广泛应用于许多安全协议和标准中，例如SSL/TLS、SSH、IPSec等。

在AES加密中，明文被分成128位的块，每个块使用相同的密钥进行加密。加密过程包括以下步骤：

1. 密钥扩展：将密钥扩展为加密算法所需的轮密钥。
2. 初始轮：将明文分成块，并与第一轮密钥进行异或。
3. 多轮加密：将初始轮产生的结果反复进行多轮加密，每轮使用不同的轮密钥进行加密。
4. 最终轮：在最后一轮加密中，将块进行加密，但是不再进行下一轮加密，而是直接输出密文。

#### 1.2 加解密步骤

前端：

1. 初始化: 引入jsencrypt库并出始化JSEncrypt对象，使用encrypt.setPublicKey()方法设置公钥,公钥是随机任意字符串 。
2. 生成私钥和偏移量: 随机生成aes的私钥key和偏移量 iv，使用encrypt.encrypt()方法对私钥key和偏移量iv进行加密得到code,并将code传给后端, 将私钥与code存入客户端。
3. 加密: 引入CryptoJS插件,使用CryptoJS.AES.encrypt()方法结合生成的私钥key和偏移量iv加密数据。

后端：

1. 初始化: 使用encrypt.setPrivateKey()方法设置私钥,私钥要与前端的公钥一致。
2. 解密code: 使用encryptor.decrypt()方法解密code得到eas的私钥key和偏移量iv。
3. 解密数据: 引入CryptoJS插件, 使用CryptoJS.AES.decrypt()方法结合key和iv解密数据。

#### 1.3 前端代码实现

封装加密/解密：

```js
// 创建key.js文件

import JSEncrypt from 'jsencrypt'
const encrypt = new JSEncrypt();
let publicKey = 'MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQCeCDcnFrS7DIRbvZLHreVUzaMbAFy2DYmioxBK606urY4rVR8IgLgUhnyw2/GQ99pyr8lGtqPeOoapantw1XwEVyi74MDxs4UDL8j4OZR1Es7HVGOB0GwKWobdU9cm/1iDwGyouSmijxKyAePg6KsLNgbjDPYZRS11bYEuZ8/RLQIDAQAB';
// 设置公钥
encrypt.setPublicKey('-----BEGIN PUBLIC KEY-----' + publicKey + '-----END PUBLIC KEY-----')

const random = (length) => {
  var str = Math.random().toString(36).substr(2);
  if (str.length >= length) {
    return str.substr(0, length);
  }
  str += random(length - str.length);
  return str;
}

export const rsaEncode = (src) => {
  // 加密数据
  let data = encrypt.encrypt(src);
  return data
};

export const getKey = () => {
// 生成私钥和偏移量
  let key = { key: random(16), iv: random(16) };
  // 对私钥和偏移量加密
  let code = rsaEncode(key.key + ',' + key.iv);
  window.codeArr = window.codeArr || {};
  // 存入客户端
  codeArr[code] = key;
  return {
    key, code
  }
};

export const getAesKey = (aes) => {
  let key = JSON.parse(JSON.stringify(codeArr[aes]));
  // 从客户端获取到 key
  delete codeArr[aes];
  return key
};

window.getKey = getKey
window.rsaEncode = rsaEncode
```

```js
// 创建 encrypt.js 文件
import CryptoJS from 'crypto-js';

// ------------AES 加密-------------
function getAesString(data, key, iv) {
  let keys = CryptoJS.enc.Utf8.parse(key)
  let vis = CryptoJS.enc.Utf8.parse(iv)
  let encrypt = CryptoJS.AES.encrypt(data, keys, {
    iv: vis, //iv偏移量 CBC需加偏移量
    mode: CryptoJS.mode.CBC, //CBC模式
    padding: CryptoJS.pad.Pkcs7 //padding处理
  });
  return encrypt.toString(); //加密完成后，转换成字符串
}


// ------------AES 解密-------------
function getDAesString(encrypted, key, iv) {
  var key  = CryptoJS.enc.Utf8.parse(key);
  var iv   = CryptoJS.enc.Utf8.parse(iv);
  var decrypted =CryptoJS.AES.decrypt(encrypted,key,{
    iv:iv,
    mode:CryptoJS.mode.CBC,
    padding:CryptoJS.pad.Pkcs7
  });
  return decrypted.toString(CryptoJS.enc.Utf8);
}

// AES 对称秘钥加密
const aes = {
  en: (data, key) => getAesString(data, key.key, key.iv),
  de: (data, key) => getDAesString(data, key.key, key.iv)
};

export { aes };
```

## 二、非对称加密
### 1. RSA

### 2. DSA


## 三、哈希算法
### 1. MD5

### 2. SHA-3