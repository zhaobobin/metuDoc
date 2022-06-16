# [Egg.js](https://eggjs.org/zh-cn/index.html)



### 快速入门

#### 环境准备

- 操作系统：支持 macOS，Linux，Windows
- 运行环境：建议选择 [LTS 版本](http://nodejs.org/)，最低要求 8.x。

#### 初始化

我们推荐直接使用脚手架，只需几条简单指令，即可快速生成项目（`npm >=6.1.0`）

```markdown
$ mkdir egg-example && cd egg-example
$ npm init egg --type=simple
$ npm i
```

启动项目

```markdown
$ npm run dev
$ open http://localhost:7001
```



###HTTP



#### 在 Egg 中定义 RESTful 风格的 URL

如果想通过 RESTful 的方式来定义路由， 我们提供了 `app.resources('routerName', 'pathMatch', controller)` 快速在一个路径上生成 [CRUD](https://en.wikipedia.org/wiki/Create,_read,_update_and_delete) 路由结构。

```js
// app/router.js
module.exports = app => {
  const { router, controller } = app;
  router.resources('posts', '/api/posts', controller.posts);
  router.resources('users', '/api/v1/users', controller.v1.users); // app/controller/v1/users.js
};
```

上面代码就在 `/posts` 路径上部署了一组 CRUD 路径结构，对应的 Controller 为 `app/controller/posts.js` 接下来， 你只需要在 `posts.js` 里面实现对应的函数就可以了。

| Method | Path            | Route Name | Controller.Action             |
| ------ | --------------- | ---------- | ----------------------------- |
| GET    | /posts          | posts      | app.controllers.posts.index   |
| GET    | /posts/new      | new_post   | app.controllers.posts.new     |
| GET    | /posts/:id      | post       | app.controllers.posts.show    |
| GET    | /posts/:id/edit | edit_post  | app.controllers.posts.edit    |
| POST   | /posts          | posts      | app.controllers.posts.create  |
| PUT    | /posts/:id      | post       | app.controllers.posts.update  |
| DELETE | /posts/:id      | post       | app.controllers.posts.destroy |

```js
// app/controller/posts.js
exports.index = async () => {};

exports.new = async () => {};

exports.create = async () => {};

exports.show = async () => {};

exports.edit = async () => {};

exports.update = async () => {};

exports.destroy = async () => {};
```

如果我们不需要其中的某几个方法，可以不用在 `posts.js` 里面实现，这样对应 URL 路径也不会注册到 Router。





#### 获取查询参数

- 获取url问号参数：**ctx.query**

- 获取路由参数：**ctx.params**

- 获取请求的body参数：**ctx.request.body**；Egg框架内置了bodyParser中间件来对这两类格式的请求 body 解析成 object 挂载到 ctx.request.body 上。

  **框架对 bodyParser 设置了一些默认参数，配置好之后拥有以下特性**：

  - 当请求的 Content-Type 为 `application/json`，`application/json-patch+json`，`application/vnd.api+json` 和 `application/csp-report` 时，会按照 json 格式对请求 body 进行解析，并限制 body 最大长度为 `100kb`。
  - 当请求的 Content-Type 为 `application/x-www-form-urlencoded` 时，会按照 form 格式对请求 body 进行解析，并限制 body 最大长度为 `100kb`。
  - 如果解析成功，body 一定会是一个 Object（可能是一个数组）。

- 获取header参数：**ctx.header**



#### 校验查询参数 ctx.validate

* ctx.validate(createRule, ctx.request.body)		// 参数：校验规则、被校验数据

* 调用 validate 方法对请求参数进行验证。

* 用验证过的参数调用 service 封装的业务逻辑来创建一个 topic。

* 按照接口约定的格式设置响应状态码和内容。

* **中间件**统一处理：app/middleware/error_handler.js

  ```js
  /**
   * 全局错误处理中间件
   * app/middleware/error_handler.js
   */
  'use strict';
  
  module.exports = () => {
    return async function errorHandler(ctx, next) {
      try {
        await next();
      } catch (err) {
        // ctx.app.emit('error', err, ctx); // 所有的异常都在 app 上触发一个 error 事件，框架会记录一条错误日志
        const status = err.status || 500;
        // 生产环境时 500 错误的详细错误内容不返回给客户端，因为可能包含敏感信息
        const err_msg = status === 500 && ctx.app.config.env === 'prod' ?
          'Internal Server Error'
          :
          err.message;
        ctx.status = status;
        ctx.body = { code: 1, data: '', message: err_msg, errors: err.errors };
      }
    };
  };
  
  ```

* 参考文章

  * [eggjs的参数校验模块egg-validate的使用和进一步定制化升级](https://blog.csdn.net/le_17_4_6/article/details/94611179)
  * [parameter](https://github.com/node-modules/parameter)



#### Egg中使用[JWT](Jwt.md)校验

* 修改配置：config/config.default.js

  ```js
  exports.security = {
      csrf: {
        enable: false,
        ignoreJSON: true,
      },
      domainWhiteList: [ '*' ],
    };
  exports.cors = {
      origin: '*',
      allowMethods: 'GET,POST,PUT,PATCH,DELETE,HEAD',
    };
  ```

* 生成token：app/utils/token.js

  ```js
  'use strict';
  
  const auth = require('basic-auth');
  const jwt = require('jsonwebtoken');
  const PrivateKey = '自定义PrivateKey';
  
  /**
   * 生成Token - 默认2小时过期
   * @param user
   */
  exports.getToken = function(user) {
    return jwt.sign(
      { data: { uid: user._id, userType: user.userType } },
      PrivateKey,
      { expiresIn: '2h' }
    );
  };
  
  /**
   * 验证Token
   * 从req.headers.Authorization中解析token
   * @param ctx
   * @param next
   */
  exports.verifyToken = function(ctx, next) {
    // 拿取token 数据 按照自己传递方式写
    const result = auth(ctx.request);
  
    if (result) {
      // 解码 token (验证 secret 和检查有效期（exp）)
      jwt.verify(result.name, PrivateKey, function(err, decoded) {
        if (err) {
          // return res.status(401).json({ status: 9, message: '用户未登录' });
          ctx.body = { code: 9, message: '用户未登录' };
        } else {
          ctx.request.uid = decoded.data.uid;
          ctx.request.userType = decoded.data.userType;
          next();
        }
      });
    } else {
      ctx.body = { code: 1, message: '没有操作权限' };
    }
  };
  ```

  

#### 在Egg中处理异常查询

* 参考：[RESTful API 常用状态码](RESTful.md#restful_status)
* 参数校验失败：
* 查询结果不存在：`ctx.throw(404, '用户不存在')`



#### 常见问题

1、[关于eggjs在Linux上启动时报错的问题及解决办法](https://blog.csdn.net/qq_41966938/article/details/81131058)

2、环境变量NODE_ENV重启后失效
   1) vim /etc/profile
   2) NODE_ENV=production
   3) source /etc/profile

