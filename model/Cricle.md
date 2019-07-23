# 圈子 Cricle

#### 定义

* 圈子：用户自发形成的话题讨论小组
* 创建：建立圈子的用户可以自行设置圈子的标题、描述、标签等信息。
* 权限：创建者默认成为圈子的管理者，管理者可以添加其他的管理者，或将管理权限转交给其他圈子成员。
* 加入：其他用户可以申请加入圈子，经圈子管理者审核通过后，成为圈子的正式成员。用户可自行退出圈子。



#### Model

```js
//Cricle 圈子
const mongoose = require('mongoose');
const Schema = mongoose.Schema;
const ObjectId = Schema.Types.ObjectId;

let CricleSchema = new Schema({

  name: {type: String, unique: true},                       //圈子名称
  tag: [{type: ObjectId, ref: 'Tag'}],                      //圈子标签
  members: [{type: ObjectId, ref: 'User'}],                 //圈子成员
  desc: String,                                             //圈子简介
  notice: String,                                           //圈子公告
  cover: String,                                            //圈子封面图
  status: {type: Number, default: 1},                       //圈子状态，1 正常 0 关闭
  createtime: {type: Date, default: Date.now()},		        //创建时间

});

//操作
CricleSchema.pre('save', function(next){
  this.createtime = Date.now();
  next();
});

module.exports = mongoose.model('Cricle', CricleSchema);


```

