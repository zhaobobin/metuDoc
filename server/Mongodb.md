# MongoDb



####[云数据库 MongoDB Atlas](https://www.mongodb.com/cloud/atlas)



## MongoDb4.0 新特性

#### 事务 transaction

* 原子性：指的是一个事务内，有多次操作的情况下，要么都执行，要么都不执行。
* 一致性：数据库事务执行前后保持一致性，也就是事务执行前后都能保证是正确的，并满足所有预先设定的限制条件。
* 隔离型：多个事务之间的执行是相互隔离的，互不影响。
* 持久性：事务处理完毕之后，对数据的改变是永久的，即使发生了系统故障，这些修改也不应该丢失，应该是持久的！



#### 跨文档ACID事务

演示：

```js
ClientSession session = client.startSession();

session.startTransaction();   开启事务

session.commitTransaction();  提交事务
```





###Mongooose

####mongoose定义模型

```js
// app/model/user.js
'use strict';

module.exports = app => {
  const mongoose = app.mongoose;
  const Schema = mongoose.Schema;

  const UserSchema = new Schema({
    username: { type: String },
    password: { type: String },
  }, { timestamps: { createdAt: 'createTime', updatedAt: 'updateTime' } });

  UserSchema.statics = {
    detail: function(id) {

    },
  };

  return mongoose.model('User', UserSchema);
};

```



#### mongoose高级查询

* **$or**：或条件查询

  ```js
  return await ctx.model.User.findOne({
     $or: [{ nickname: path }, { _id: path }],
  });
  ```

* 



#### Model操作

* 查询列表：`Model.find()`
* 查询详情：`Model.findById(id)`
* 添加：`const createUser = new Schame(data); createUser.save()`

