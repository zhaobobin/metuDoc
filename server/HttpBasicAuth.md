# HttpBasicAuth 校验



#### 依赖

```js
const auth = require('basic-auth');
const jwt = require('jsonwebtoken');
```



## 服务端

#### 生成Token

- 用户登录成功后，等到数据user
- 加密方法

```js
// 生成Token
exports.getToken = function(user) {
  return jwt.sign(
    { data: { _id: user._id } },
    PrivateKey,
    { expiresIn: '1d' }
  );
};
```

- 生成token

```js
const token = getToken(user)
```



#### 后端返回Token

```js
// 接口返回
res.json({
    code: 0,
    msg:'请求成功',
    data: {
        currentUser: user,
        userType: userType,
        token: token
    }
});

// 保存token
Storage.set(ENV.storageToken, res.data.token);
```



#### 接口保护

```js
router.post('/api/v1/user/current', verifyToken, controller.access.current);
```



#### 验证Token - 中间件

```js
// 校验Token
exports.verifyToken = async function(ctx, next) {
  const result = auth(ctx.request);
  // console.log(result)
  try {
    ctx.state.user = jwt.verify(result.name, PrivateKey);
  } catch (err) {
    ctx.throw(401, err.message);
  }
  await next();
};
```



## 客户端

#### 生成Token并上传

```js
newOptions.headers['Authorization'] = 'Basic ' + Base64.encode(Storage.get(ENV.storageToken) + ':')
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

