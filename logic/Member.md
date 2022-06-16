## 成员管理



### 团队成员管理



####**一、邀请团队成员**



**1、链接邀请**

1）在页面中设置成员角色 memberRole

```javascript
{
  admin: '管理员',
  user: '普通成员',
  outsource: '外包人员',
}
```

然后生成邀请链接，例如：

```javascript
http://localhost:8080/user/join?invite_code=eyJpbnZpdGVVc2VySWQiOiI5MDg3YjI0Yi00ZWQ2LTQwMmUtYjg4Ni1mMDFjMzg5ZWIyMTkiLCJ0ZWFtSWQiOiI0NTIyODE4Zi1mNzgwLTQyNjgtOTA2ZS1kYTI0NGM2NTZhZWQiLCJtZW1iZXJSb2xlIjoidXNlciIsInJlcXVpcmVkQXBwcm92YWwiOnRydWV9
```

2）链接邀请生成规则：

对象 -> 字符串 -> Base64

```javascript
const params = {
  inviteUserId: this.$store.state.userInfo.detail.id, // 邀请发起人，取自当前登陆人信息
  teamId: this.initialValue.id, // 团队id
  memberRole: this.inviteMemberRole, // 成员角色
  requiredApproval: this.requiredApproval1, // 是否需要审批
};
this.inviteUrl = `${window.location.origin}/user/join?invite_code=${window.btoa(JSON.stringify(params))}`;
```



3）团队管理者复制邀请链接，通过通讯软件将邀请链接分享给他人



4）用户通过邀请链接进入“加入团队”页面：

4-1 在“加入团队”页面，解析邀请链接：Base64 -> 字符串 -> 对象。

4-2 获取本地存储中的token信息，判断当前访问者的登陆状态，进行区分处理：

* 存在token信息并通过验证，则在页面中只展示加入团队按钮；

* 如果没有token或验证失败，则展示引导注册表单，完成输入后提交到server处理，先保存注册信息，再将新用户信息插入到users、teams表的相关字段中；

* Server端检查请求信息，避免用户重复申请加入，校验通过时将信息保存到users、teams表的相关字段中。

​		在team.members中保存requiredApproval，通过status告知团队管理者，申请人的状态。

* 加入成功后的操作：

​		需要审核，显示申请成功的页面提示，页面跳转到首页，在审核通过后收到系统邮件；

​		不需要审核，显示申请成功的页面提示，页面跳转到项目工作区，同时收到系统邮件。



**2、邮箱邀请**

1）在页面中设置成员角色 memberRole

```javascript
{
  admin: '管理员',
  user: '普通成员',
  outsource: '外包人员',
}
```

然后生成邀请链接，例如：

```javascript
http://localhost:8080/user/join?invite_code=eyJpbnZpdGVVc2VySWQiOiI5MDg3YjI0Yi00ZWQ2LTQwMmUtYjg4Ni1mMDFjMzg5ZWIyMTkiLCJ0ZWFtSWQiOiI0NTIyODE4Zi1mNzgwLTQyNjgtOTA2ZS1kYTI0NGM2NTZhZWQiLCJlbWFpbCI6InpoYW9iYjRAbGVub3ZvLmNvbSIsIm1lbWJlclJvbGUiOiJ1c2VyIiwicmVxdWlyZWRBcHByb3ZhbCI6dHJ1ZX0=
```



2）链接邀请生成规则：

对象 -> 字符串 -> Base64

```javascript
const params = {
  inviteUserId: this.$store.state.userInfo.detail.id, // 邀请发起人
  teamId: this.initialValue.id, // 团队id
  email: values.email,					// 邮箱，可输入多个，用 “,” 分隔
  memberRole: values.user_role, // 成员角色
  requiredApproval: this.requiredApproval2, // 是否需要审批
};
const inviteUrl = `${window.location.origin}/user/join?invite_code=${window.btoa(JSON.stringify(params))}`;
```



3）管理者点击邀请按钮后，将邀请链接提交给Server处理，依据填入的email，可以给多人发送邮件。



4）用户通过点击邮件中的邀请链接进入“加入团队”页面：

4-1 在“加入团队”页面，解析邀请链接：Base64 -> 字符串 -> 对象。

4-2 获取本地存储中的token信息，判断当前访问者的登陆状态，进行区分处理：

* 存在token信息并通过验证，则在页面中只展示加入团队按钮；

* 如果没有token或验证失败，则展示引导注册表单，完成输入后提交到server处理，先保存注册信息，再将新用户信息插入到users、teams表的相关字段中；

* Server端检查请求信息，避免用户重复申请加入，校验通过时将信息保存到users、teams表的相关字段中。

​		在team.members中保存requiredApproval，通过status告知团队管理者，申请人的状态。

* 加入成功后的操作：

​		需要审核，显示申请成功的页面提示，页面跳转到首页，在审核通过后收到系统邮件；

​		不需要审核，显示申请成功的页面提示，页面跳转到项目工作区，同时收到系统邮件。



**3、参数说明**



申请加入团队时，参数的提交格式：

```javascript
{
  // 邀请码 invite_code
  inviteUserId [String]
  teamId [String]
  requiredApproval [Boolean]
  // 注册信息 
  email [String]
  emailcode [String]
  username [String]
  password [String]
}
```



数据库team.members中，团队成员的数据模型：

```javascript
{
  user_id [String]
  user_role [String] // owner、admin、user、outsource
  status [String] // PENDING 等待审批、APPROVED 已通过、REJECTED 已拒绝
  username [String]
  email [String]
  mobile [String]
}
```



团队成员角色

```javascript
const TEAM_MEMBER_ROLE = {
  owner: '所有者',
  admin: '管理员',
  user: '普通成员',
  outsource: '外包人员',
};
```

团队成员状态

```javascript
export const TEAM_MEMBER_STATUS = {
  'PENDING': '等待审批',
  'APPROVED': '已通过',
  'REJECTED': '已拒绝'
};
```







***



####**二、编辑团队成员**



***



####**三、删除团队成员**



###项目成员管理

