## PM2



#### 安装

```
npm install -g pm2
```

#### 创建软连接

```
ln -s /usr/local/node-v10.15.3-linux-x64/bin/pm2 /usr/local/bin/pm2
```

#### 启动express服务

--watch参数，意味着当你的express应用代码发生变化时，pm2会帮你重启服务

```
pm2 start /root/metuServer/server.js --watch
pm2 list
pm2 save
```

#### 设置pm2 开机启动

```
pm2 startup
```

#### 错误日志

```
pm2 log
```

#### 重启

```
pm2 reload all
```

#### 卸载服务
```
pm2 service uninstall
```



#### 运行多个项目

```
pm2 start /root/api/server/server.js --watch
pm2 list
pm2 save
```



#### 异常问题

1) [mongoose升级导致链接失败](https://github.com/eggjs/egg/issues/4531)

```
exports.mongoose = {
    url:
      process.env.NODE_ENV === "development" ? Server.db.test : Server.db.dev,
    options: {
      useFindAndModify: false,
      useUnifiedTopology: true,
      useNewUrlParser: true,
      useCreateIndex: true,
    },
    plugins: [],
};
```



