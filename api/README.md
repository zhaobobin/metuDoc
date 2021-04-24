## API 接口说明



#### HTTP 请求参数规则

- 【GET】**query**： 如?q=keyword
- 【POST】**body**: 如{name: '谜团'}
- 【PATH】**params**: 如users/:id
- 【Header】**hrader**，如Accept、Cookie



#### [RESTful](server/RESTful.md#restful_status) StatusCode HTTP状态码

```js
// 2xx
200 OK - [GET]：服务器成功返回用户请求的数据，该操作是幂等的（Idempotent）。
201 CREATED - [POST/PUT/PATCH]：用户新建或修改数据成功。
202 Accepted - [*]：表示一个请求已经进入后台排队（异步任务）
204 NO CONTENT - [DELETE]：用户删除数据成功。


// 4xx
400 INVALID REQUEST - [POST/PUT/PATCH]：用户发出的请求有错误，服务器没有进行新建或修改数据的操作，该操作是幂等的。
401 Unauthorized - [*]：表示用户没有权限（令牌、用户名、密码错误）。
403 Forbidden - [*] 表示用户得到授权（与401错误相对），但是访问是被禁止的。
404 NOT FOUND - [*]：用户发出的请求针对的是不存在的记录，服务器没有进行操作，该操作是幂等的。
406 Not Acceptable - [GET]：用户请求的格式不可得（比如用户请求JSON格式，但是只有XML格式）。
409 Resources already exist - [POST]：资源已存在。
410 Gone -[GET]：用户请求的资源被永久删除，且不会再得到的。
422 Unprocesable entity - [POST/PUT/PATCH] 当创建一个对象时，发生一个验证错误。


// 5xx
500 INTERNAL SERVER ERROR - [*]：服务器发生错误，用户将无法判断发出的请求是否成功
533 INTERNAL SERVER ERROR - [*]：keystore 不存在
534 INTERNAL SERVER ERROR - [*]：keystore 已过期
```



#### 接口保护

* 使用[HttpBasicAuth](server/HttpBasicAuth.md)校验：后端依赖 `yarn add basic-auth`
* 使用[JWT](server/Jwt.md)校验：后端依赖 `yarn add jsonwebtoken`
* 使用AES加密：后端依赖 `yarn add crypto-js





##### 参考

* [Postman通用接口加密解决方案](https://www.jianshu.com/p/b107452e607)





## Model 模型

#### 常用方法

1. 列表

   ```js
   // model
     QuestionSchema.statics.list = async function(params) {
       const { per_page = 10 } = params,
         page = Math.max(params.page * 1, 1) - 1, // 页数
         perPage = Math.max(per_page, 1); // 每页数量
       // 查询条件
       let query = {};
       if (params.q) query = { $or: [{ title: params.q }, { description: params.q }] };
       const list = await this.find(query)
         .skip(page * perPage)
         .limit(perPage)
         .exec();
       return { list, count: list.length };
     };
   ```

   ```js
   // service
   async list() {
       const { ctx } = this;
       return await ctx.model.Question.list(ctx.query);
     }
   ```

   

2. 详情

   ```js
   // model
     QuestionSchema.statics.detail = async function({ _id, select, populate }) {
       return await this.findOne({ _id }, select).populate(populate);
     };
   ```

   ```js
   // service
     async detail() {
       const { ctx } = this;
       const topic = await ctx.model.Question.detail({
         _id: ctx.params.id,
         select: this.select,
         populate: 'questioner',
       });
       if (!topic) ctx.throw(404, 'questions 问题不存在');
       return topic;
     }
   ```

3. 创建

   ```js
   // model
     QuestionSchema.statics.add = async function(ctx) {
       const createTopic = new this(ctx.request.body);
       createTopic.createTime = Date.now();
       return await createTopic.save();
     };
   ```

   ```js
   // service
     async create() {
       const { ctx } = this;
       ctx.validate(this.rule.create, ctx.request.body);
       const topic = await ctx.model.Question.add(ctx);
       if (!topic) ctx.throw(401, 'questions 问题创建失败');
       ctx.helper.success({ ctx });
     }
   ```

   

4. 修改

   ```js
   // model
     QuestionSchema.statics.update = async function(id, data) {
       data.updateTime = Date.now();
       return await this.findByIdAndUpdate(id, data, { new: true });
     };
   ```

   ```js
   // service
     async update() {
       const { ctx } = this;
       ctx.validate(this.rule.update, ctx.request.body);
       const res = await ctx.model.Question.update(ctx.params.id, ctx.request.body);
       if (!res) ctx.throw(404, 'questions 问题不存在');
       return res;
     }
   ```

   



