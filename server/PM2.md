## PM2



1. 安装

```
npm install -g pm2
```

2. 创建软连接

```
ln -s /usr/local/node-v10.15.3-linux-x64/bin/pm2 /usr/local/bin/pm2
```

3. 启动express服务

--watch参数，意味着当你的express应用代码发生变化时，pm2会帮你重启服务

```
pm2 start /root/metuServer/server.js --watch
pm2 list
pm2 save
```

4. 设置pm2 开机启动

```
pm2 startup
```

5. 错误日志

```
pm2 log
```

6. 重启

```
pm2 reload all
```

