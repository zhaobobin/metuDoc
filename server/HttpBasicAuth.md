# HttpBasicAuth 校验



#### 依赖

```js
const auth = require('basic-auth');
const jwt = require('jsonwebtoken');
```



#### 生成Token

- 用户登录成功后，等到数据user
- 加密方法

```js
/**
 * 生成Token - 默认2小时过期
 * @param {object} user uid: user._id, userType: user.userType
 * @returns {string}
 */
exports.getToken = function(user){
  return jwt.sign(
    { data: {uid: user._id, userType: user.userType} },
    PrivateKey,
    { expiresIn: '2h' }
  );
};
```

- 生成token

```js
let token = Token.getToken(user);
```



#### 下传Token

```js
// 接口返回
res.json({
    status:1,
    msg:'登录成功',
    data: {
        currentUser: user,
        currentAuthority: body.userType,
        token: token
    }
});

// 保存token
Storage.set(ENV.storageToken, res.data.token);
```



#### 上传Token

```js
newOptions.headers['Authorization'] = 'Basic ' + Base64.encode(Storage.get(ENV.storageToken) + ':')
```



#### 接口保护

```js
router.post('*/UserDel', Token.verifyToken, User.del);
```



#### 验证Token

```js
/**
 * 验证Token
 * 从req.headers.Authorization中解析token
 * @param req
 * @param res
 * @param next
 * @returns {JSON | Promise<any> | *}
 */
exports.verifyToken = function(req, res, next) {
  // 拿取token 数据 按照自己传递方式写
  let result = auth(req);
  console.log(result)
  if (result) {
    // 解码 token (验证 secret 和检查有效期（exp）)
    jwt.verify(result.name, PrivateKey, function(err, decoded) {
      console.log(decoded)
      if (err) {
        return res.status(401).json({ status: 9, message: '用户未登录' });
      } else {
        req.token = decoded;          //在req中写入解密结果
        next();                       //继续下一步路由
      }
    });
  } else {
    res.status(401).json({status: 0, message: '没有操作权限'});   // 没有拿到token 返回错误
  }
};

/*
	decoded示例 
{ 
	data: { uid: '5b6d2a88207f784aed5d9f65', userType: 'admin' },
  iat: 1563091627,
  exp: 1563098827 
}
*/
```



#### 前端处理Token过期

- Admin

```js
// 登录过期等
      if(res.status === 9){
        Storage.remove(ENV.storageToken);              //删除token
        notification.error({
          message: '错误提示',
          description: res.msg,
        });
        setAuthority('guest');
        yield put({
          type: 'changeLoginStatus',
          payload: {
            isAuth: false,
            currentUser: '',
          }
        });
        yield put(routerRedux.push({ pathname: '/user/login' }));
      }else{
        yield callback(res);
      }
```

- Index

```js
//登录过期等
      if(res.status === 9){
        Storage.remove(ENV.storageToken);              //删除token
        notification.error({
          message: '提示',
          description: res.message
        });
        yield put({
          type: 'changeLoginStatus',
          payload: {
            isAuth: false,
            userInfo: '',
          }
        });
        yield put(routerRedux.push({ pathname: '/' }));
      }else{
        yield callback(res);
      }

```

