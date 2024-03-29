<div align="center">
  <a href="https://www.metuwang.com/" target="_blank" rel="noopener noreferrer nofollow">
    <img src="https://www.metuwang.com/images/favicon.png" width="auto" height="32px" alt="去投网logo" style=" position: relative; right: 5px;top:6px;" />
    <strong style="color: #333; font-size: 28px; line-height: 32px;">迷图网</strong>
  </a>
</div>


## 

迷图网，是一个致力于摄影分享、发现、售卖的专业平台，来自世界各地的摄影师是我们忠实的用户。让你与他人因图片相识，世界那么大，我想去看看。



####指南

- [逻辑](logic/README.md)
  * [社区](logic/Community.md)
  * [话题与标签](logic/Topic.md)
  * [消息中心](logic/Message.md)
- [容器](docker/README.md)
- [服务端](server/README.md)
  - [RESTful API](server/RESTful.md)
  - [Jwt](server/Jwt.md)
  - [HttpBasicAuth](server/HttpBasicAuth.md)
  - [角色鉴权](server/Scope.md)
  - [Mongodb](server/Mongodb.md)
  - [Koa](server/Koa.md)
  - [Egg](server/Egg.md)
- [客户端](client/README.md)



####目录

```js

  dist                          打包目录

  public                        公共资源
    ├—— css
    ├—— images
    ├—— markdown

  src                           react代码
    ├—— assets                  图片等资源
    ├—— common                  自定义导航

    ├—— blocks                  通用模块 - 组件综合使用
    ├—— components              通用组件
    ├—— containers              页面组件
    ├—— layouts                 布局组件，嵌套路由
    ├—— locales                 国际化配置

    ├—— models                  数据模型
    ├—— pages                   页面文件
    ├—— routes                  路由配置
    ├—— theme                   主题配置
    ├—— utils                   工具库
    ├—— index.js                项目入口
    └—— index.ejs               模版文件

  .webpackrc                    自动化配置

```



####配置

```js
{

  api: {
    test: 'http://localhost:8080/',
    pro: 'http://www.metuwang.com/',
  },

  appname: '迷图网',
  hometitle: '迷图网 - 摄影图片素材分享社区',
  keywords: '迷图网,摄影,图片,素材,分享,社区。',
  description: '迷图网(www.metuwang.com)，是一个致力于摄影分享、发现、售卖的专业平台，来自世界各地的摄影师是我们忠实的用户。让你与他人因图片相识，世界那么大，我想去看看。',
  author: '迷图网(www.metuwang.com)',
  verification: '',

  address: '',
  hotline: '',
  email: '',
  icp: 'ICP经营许可证 京B2-20160180',
  beian: '京ICP备16058155号-1',
  copyright: '©2015-2018 迷图网 All rights reserved',
  slogan: '影像点亮生活',
  web: 'www.metuwang.com',
  worktime: '9:00-17:30',

  storageToken: 'metuIndex-token',
  storageLastTel: 'metuIndex-lastTel',
  storageRemenber: 'metuIndex-remenber',
  storageHistory: 'metuIndex-history',
  storageTheme: 'metuIndex-theme',
  storageCurrentMenu: 'metuIndex-currentMenu',

  storageWechatLoginState: 'metuIndex-WechatLoginState',    // 微信授权登录state
  storageWeiboLoginState: 'metuIndex-WeiboLoginState',    // 微博授权登录state
  storageQqLoginState: 'metuIndex-QqLoginState',    // QQ授权登录state

}
```
<!-- npm install -g gitbook-cli -->

<!-- Gitbook 一键部署至 GitHub Pages -->

<!-- 1. npm install -g gh-pages -->

<!-- 2. gitbook init -->

<!-- 3. gitbook build -->

<!-- 4. gh-pages -d _book -->