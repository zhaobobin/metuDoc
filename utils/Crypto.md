# 加密 Crypto

#### 依赖

```npm
npm i crypto-js --save
```



#### 导入

```js
import ENV from '@/config/env'
const CryptoJS = require('crypto-js');

ENV.web 生成加密key的拼接字符串
ENV.iv 生成加密偏移量的字符串
```



#### 加密

```js
/**
 * 加密方法
 * @param k
 * @param text
 * @returns {string}
 * @constructor
 */
export function Encrypt(k, text){
  const DecryptKey = (k + ENV.web).substring(0, 16);
  const key = CryptoJS.enc.Utf8.parse(DecryptKey);              //十六位十六进制数作为密钥
  const iv = CryptoJS.enc.Utf8.parse(ENV.iv);                   //十六位十六进制数作为密钥偏移量

  let encrypted = CryptoJS.AES.encrypt(
    text.toString(),
    key,
    {
      iv: iv,
      mode: CryptoJS.mode.CBC,
      padding: CryptoJS.pad.Pkcs7
    }
  );
  return encrypted.ciphertext.toString();
}
```



#### 解密

```js
/**
 * 解密方法
 * @param k
 * @param text
 * @returns {string}
 * @constructor
 */
export function Decrypt(k, text){
  const DecryptKey = (k + ENV.web).substring(0, 16);
  const key = CryptoJS.enc.Utf8.parse(DecryptKey);               //十六位十六进制数作为密钥
  const iv = CryptoJS.enc.Utf8.parse(ENV.iv);                    //十六位十六进制数作为密钥偏移量

  let encryptedHexStr = CryptoJS.enc.Hex.parse(text);
  let srcs = CryptoJS.enc.Base64.stringify(encryptedHexStr);

  let decrypt = CryptoJS.AES.decrypt(
    srcs,
    key,
    {
      iv: iv,
      mode: CryptoJS.mode.CBC,
      padding: CryptoJS.pad.Pkcs7
    }
  );
  return decrypt.toString(CryptoJS.enc.Utf8);
}
```

