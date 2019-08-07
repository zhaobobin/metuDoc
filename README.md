<div align="center">
  <a href="https://www.metuwang.com/" target="_blank" rel="noopener noreferrer nofollow">
    <img src="http://www.metuwang.com/favicon.png" width="auto" height="32px" alt="去投网logo" style=" position: relative; right: 5px;top:6px;" />
    <strong style="color: #333; font-size: 28px; line-height: 32px;">迷图网</strong>
  </a>
</div>

## 介绍

迷图网(www.metuwang.com)，是一个致力于摄影分享、发现、售卖的专业平台，来自世界各地的摄影师是我们忠实的用户。让你与他人因图片相识，世界那么大，我想去看看。


## 指南

- [区块](block/README.md)
  - [用户登录](block/User/UserLogin.md)
  - [用户注册](block/User/UserRegister.md)
  - [找回密码](block/User/PsdReset.md)
- [组件](components/README.md)
  - [表单](components/Form/README.md)
    - [InputText](components/Form/InputText.md)
    - [InputMobile](components/Form/InputMobile.md)
    - [InputPassword](components/Form/InputPassword.md)
    - [InputSmscode](components/Form/InputSmscode.md)
    - [InputEmail](components/Form/InputEmail.md)
    - [InputCaptcha](components/Form/InputCaptcha.md)
- [模型](model/README.md)
- [工具](utils/README.md)


## 结构

```

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


## 配置

```
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

<!-- Gitbook 一键部署至 GitHub Pages -->

<!-- 1. npm install g gh-pages -->

<!-- 2. gitbook build -->

<!-- 3. gh-pages -d _book -->