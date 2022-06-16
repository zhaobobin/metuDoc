### 二进制安装

1.下载解压

```
wget https://cdn.npm.taobao.org/dist/node/v15.8.0/node-v15.8.0-linux-x64.tar.xz
tar -xf node-v15.8.0-linux-x64.tar.xz
cd node-v15.8.0-linux-x64/bin
./node -v
```

2.创建软链接，您就可以在任意目录下直接使用node和npm命令

```
ln -s /root/node-v15.8.0-linux-x64/bin/node /usr/local/bin/node
ln -s /root/node-v15.8.0-linux-x64/bin/npm /usr/local/bin/npm
```

3.查看node、npm版本

```
node -v
npm -v
```



4.软链接pm2

```
ln -s /root/node-v15.8.0-linux-x64/bin/pm2 /usr/local/bin/pm2
```



5.使用 yum 安装 Yarn

添加 yum 源配置

```
curl -sL https://dl.yarnpkg.com/rpm/yarn.repo | sudo tee /etc/yum.repos.d/yarn.repo
```

使用 yum 安装

```
sudo yum -y install yarn
```

查看安装的 Yarn 版本：

```
yarn -v
```

设置 Yarn 全局 bin 路径
查看当前 Yarn 全局 bin 路径：

```
yarn global bin
```

