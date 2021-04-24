## Nginx



## 一、简介

　　Nginx是一款轻量级的网页服务器、反向代理服务器。相较于Apache、lighttpd具有占有内存少，稳定性高等优势。**它最常的用途是提供反向代理服务。**

## 二 、安装

1、准备工作

Nginx的安装依赖于以下三个包，意思就是在安装Nginx之前首先必须安装一下的三个包，注意安装顺序如下：

　　1 SSL功能需要openssl库，直接通过yum安装: #yum install openssl

　　2 gzip模块需要zlib库，直接通过yum安装: #yum install zlib

　　3 rewrite模块需要pcre库，直接通过yum安装: #yum install pcre

2、安装Nginx依赖项和Nginx

　　1 使用yum安装nginx需要包括Nginx的库，安装Nginx的库

　　　　#rpm -Uvh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm

　　2 使用下面命令安装nginx

　　　　#yum install nginx

　　3 启动Nginx

　　　　#service nginx start



启动失败：

```
Job for nginx.service failed. See 'systemctl status nginx.service' and 'journalctl -xn' for details.
```

netstat -ntlp | grep 80 查看端口占用情况

```
tcp6       0      0 :::80                   :::*                    LISTEN      10205/node /root/me
```





3、直接浏览器访问localhost就会出现Nginx的欢迎界面表示你安装成功了，否则就是安装失败了

　　查看已开放的端口：firewall-cmd --list-ports

　　默认80端口加入防火墙访问白名单中：firewall-cmd --permanent --zone=public --add-port=80/tcp

　　使用firewall-cmd --reload命令使其生效，之后可以再查看下开放的端口。访问ip:80端口，如下则表示安装成功

![img](https://img2018.cnblogs.com/blog/761230/201910/761230-20191014111047072-649996964.png)

4、配置Nginx

CentOS安装Nginx后，安装在了 /etc/nginx/目录下，你可以打开/etc/nginx/conf.d/default/conf查看里面的配置，包括监听端口，域名和nginx访问的根目录

## 三、配置

以上安装方法nginx的配置文件位于

```
/usr/local/nginx/conf/nginx.conf
```

Nginx配置文件常见结构的从外到内依次是「http」「server」「location」等等，缺省的继承关系是从外到内，也就是说内层块会自动获取外层块的值作为缺省值。

### Server

接收请求的服务器需要将不同的请求按规则转发到不同的后端服务器上，在 nginx 中我们可以通过构建虚拟主机（server）的概念来将这些不同的服务配置隔离。

```
server {
    listen       80;
    server_name  localhost;
    root   html;
    index  index.html index.htm;
}
```

例如我们笔戈玩下的两个子项目 passport 和 wan 就可以通过在 nginx 的配置文件中配置两个 server，servername 分别为 passport.bigertech.com 和 wan.bigertech.com。这样的话不同的 url 请求就会对应到 nginx 相应的设置，转发到不同的后端服务器上。

这里的 **listen** 指监听端口，**server_name** 用来指定IP或域名，多个域名对应统一规则可以空格分开，**index** 用于设定访问的默认首页地址，**root** 指令用于指定虚拟主机的网页跟目录，这个地方可以是相对地址也可以是绝对地址。

通常情况下我们可以在 nginx.conf 中配置多个server，对不同的请求进行设置。就像这样：

```
server {
    listen       80;
    server_name  host1;
    root   html;
    index  index.html index.htm;
}
server {
    listen       80;
    server_name  host2;
    root   /data/www/html;
    index  index.html index.htm;
}
```

但是当 server 超过2个时，建议将不同对虚拟主机的配置放在另一个文件中，然后通过在主配置文件 nginx.conf 加上 include 指令包含进来。更便于管理。

```
include vhosts/*.conf;
```

就可以把vhosts的文件都包含进去啦。

### Localtion

每个 url 请求都会对应的一个服务，nginx 进行处理转发或者是本地的一个文件路径，或者是其他服务器的一个服务路径。而这个路径的匹配是通过 location 来进行的。我们可以将 server 当做对应一个域名进行的配置，而 location 是在一个域名下对更精细的路径进行配置。

以上面的例子，可以将root和index指令放到一个location中，那么只有在匹配到这个location时才会访问root后的内容：

```
    location / {
        root   /data/www/host2;
        index  index.html index.htm;
    }
```

location 匹配规则

```
~      波浪线表示执行一个正则匹配，区分大小写
~*    表示执行一个正则匹配，不区分大小写
^~    ^~表示普通字符匹配，如果该选项匹配，只匹配该选项，不匹配别的选项，一般用来匹配目录
=      进行普通字符精确匹配
```

匹配例子：

```
location  = / {
  # 只匹配"/".
  [ configuration A ] 
}
location  / {
  # 匹配任何请求，因为所有请求都是以"/"开始
  # 但是更长字符匹配或者正则表达式匹配会优先匹配
  [ configuration B ] 
}
location ^~ /images/ {
  # 匹配任何以 /images/ 开始的请求，并停止匹配 其它location
  [ configuration C ] 
}
location ~* \.(gif|jpg|jpeg)$ {
  # 匹配以 gif, jpg, or jpeg结尾的请求. 
  # 但是所有 /images/ 目录的请求将由 [Configuration C]处理.   
  [ configuration D ] 
}

请求:
/ -> 符合configuration A
/documents/document.html -> 符合configuration B
/images/1.gif -> 符合configuration C
/documents/1.jpg ->符合 configuration D
```

#### 静态文件映射

访问文件的配置主要有 **root 和 alias**p's 两个指令。这两个指令的区别容易弄混：

##### alias

alias后跟的指定目录是准确的，并且末尾必须加 /。

```
    location /c/ {
        alias /a/;
    }
```

如果访问站点http://location/c访问的就是/a/目录下的站点信息。

##### root

root后跟的指定目录是上级目录，并且该上级目录下要含有和location后指定名称的同名目录才行。

```
    location /c/ {
        root /a/;
    }
```

这时访问站点http://location/c访问的就是/a/c目录下的站点信息。

如果你需要将这个目录展开，在这个location的末尾加上「autoindex on; 」就可以了

#### 转发

配置起来很简单比如我要将所有的请求到转移到真正提供服务的一台机器的 8001 端口，只要这样：

```
location / {
    proxy_pass 172.16.1.1:8001;
}
```

这样访问host时，就都被转发到 172.16.1.1的8001端口去了。

#### 负载均衡

```
upstream myserver; {
    ip_hash;    
    server 172.16.1.1:8001;
    server 172.16.1.2:8002;
    server 172.16.1.3;
    server 172.16.1.4;
}
location / {
    proxy_pass http://myserver;
}
```

我们在 upstream 中指定了一组机器，并将这个组命名为 myserver，这样在 proxypass 中只要将请求转移到 myserver 这个 upstream 中我们就实现了在四台机器的反向代理加负载均衡。其中的 ip_hash 指明了我们均衡的方式是按照用户的 ip 地址进行分配。另外还有轮询、指定权重轮询、fair、url_hash几种调度算法。

 

## 附注：配置完之后可以使用测试命令验证下配置是否正确

```
nginx -t
```

或者

```
nginx -tc [nginx文件地址]
```





#### 阿里ECS配置

修改：vim /etc/nginx/nginx.conf

重启：service nginx restart

```
# For more information on configuration, see:
#   * Official English Documentation: http://nginx.org/en/docs/
#   * Official Russian Documentation: http://nginx.org/ru/docs/

user root;
worker_processes auto; #启动进程
error_log /var/log/nginx/error.log; #全局错误日志
pid /run/nginx.pid; #PID文件

# Load dynamic modules. See /usr/share/nginx/README.dynamic.
include /usr/share/nginx/modules/*.conf;

events {
    worker_connections 1024; #单个后台worker process进程的最大并发链接数
}

http {

    # 文件扩展名与文件类型映射表
    include       mime.types;

    # 默认文件类型
    default_type  application/octet-stream;

    sendfile        on;

    gzip on; #开启gzip压缩
    gzip_min_length 1k; #设置对数据启用压缩的最少字节数
    gzip_buffers    4 16k;
    gzip_http_version 1.0;
    gzip_comp_level 6; #设置数据的压缩等级,等级为1-9，压缩比从小到大
    gzip_types text/plain text/css text/javascript application/json application/javascript application/x-javascript application/xml; #设置需要压缩的数>据格式
    gzip_vary on;

    #虚拟主机配置
    server {
        listen       80 default_server; #侦听80端口,并为默认服务,default_server只能有一个
        server_name  www.metuwang.com metuwang.com; #服务域名,可以有多个,用空格隔开
        root /root/www;
        index index.html;

        location ^~ / {
                try_files $uri $uri/ /index.html;
        }
       
        location ^~ /api/{
            proxy_pass http://127.0.0.1:7001;
            proxy_send_timeout 1800;
            proxy_read_timeout 1800;
            proxy_connect_timeout 1800;
            client_max_body_size 2048m;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "Upgrade";
            proxy_set_header  Host              $http_host;   # required for docker client's sake
            proxy_set_header  X-Real-IP         $remote_addr; # pass on real client's IP
            proxy_set_header  X-Forwarded-For   $proxy_add_x_forwarded_for;
            proxy_set_header  X-Forwarded-Proto $scheme;
        }
        # 图片缓存时间设置
        location ~ .*.(gif|jpg|jpeg|png|bmp|swf)$ {
            expires 10d;
        }
        # JS和CSS缓存时间设置
        location ~ .*.(js|css)?$ {
            expires 1h;
        }
        # 404定义错误提示页面
        error_page 404             /404.html;
        # 500定义错误提示页面
        error_page   500 502 503 504 /50x.html;
        
    }
    server {
        listen       80;
        server_name  admin.metuwang.com;
        root /root/admin;
        index index.html;

        location ^~ / {
                try_files $uri $uri/ /index.html;
        }

        location ^~ /api/{
            proxy_pass http://127.0.0.1:7001;
            proxy_send_timeout 1800;
            proxy_read_timeout 1800;
            proxy_connect_timeout 1800;
            client_max_body_size 2048m;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "Upgrade";
            proxy_set_header  Host              $http_host;   # required for docker client's sake
            proxy_set_header  X-Real-IP         $remote_addr; # pass on real client's IP
            proxy_set_header  X-Forwarded-For   $proxy_add_x_forwarded_for;
            proxy_set_header  X-Forwarded-Proto $scheme;
        }
        # 图片缓存时间设置
        location ~ .*.(gif|jpg|jpeg|png|bmp|swf)$ {
            expires 10d;
        }
        # JS和CSS缓存时间设置
        location ~ .*.(js|css)?$ {
            expires 1h;
        }
        # 404定义错误提示页面
        error_page 404             /404.html;
        # 500定义错误提示页面
        error_page   500 502 503 504 /50x.html;

    }
```



#### 检测

```
nginx -t
```

#### 启动

```
nginx
```

####重启

```
service nginx restart
```

or

```
nginx -s reload
```

####重启加载配置文件
```
nginx -t -c /etc/nginx/nginx.conf
```

#### 停止

```
nginx -s stop
```

#### 查看进程
```
ps -ax | grep nginx
ps aux | grep nginx
```

#### 开机启动

1、在系统服务目录里创建nginx.service文件

```
vi /usr/lib/systemd/system/nginx.service
```



2、写入内容如下：

```
[Unit]
Description=nginx
After=network.target
  
[Service]
Type=forking
ExecStart=/usr/local/nginx/sbin/nginx
ExecReload=/usr/local/nginx/sbin/nginx -s reload
ExecStop=/usr/local/nginx/sbin/nginx -s quit
PrivateTmp=true
  
[Install]
WantedBy=multi-user.target
```

[Unit]: 服务的说明

Description:描述服务
After:描述服务类别
[Service]服务运行参数的设置
Type=forking是后台运行的形式
ExecStart为服务的具体运行命令
ExecReload为重启命令
ExecStop为停止命令
PrivateTmp=True表示给服务分配独立的临时空间
注意：[Service]的启动、重启、停止命令全部要求使用绝对路径
[Install]运行级别下服务安装的相关设置，可设置为多用户，即系统运行级别为3

3、设置开机自启动

```
systemctl enable nginx.service
```

4、查看nginx状态

```
systemctl status nginx.service
```

很奇怪，明明启动成功了，为什么显示Active: inactive (dead)？

5、杀死nginx重启nginx

```
pkill -9 nginx
```

```
ps aux | grep nginx
systemctl start nginx
```

再次查看状态，变成了active，搞定。

6、重启服务器
`reboot`

7、再次连接后，查看服务状态
`systemctl status nginx.service`

看到nginx已经启动，至此，nginx自启动配置成功。


8、查看错误日志
`vim /var/log/nginx/error.log`


#### 常见问题

1、403 Forbidden

**方案一：确认index文件是否存在，是否有读写权限，若没有则赋权**

1）进入nginx的html路径，cd /home/wl/nginx/html

2）查看目录下文件和权限，ll

3）有权限跳过，没有赋权chmod 777 -R ./

**方案二：确认nginx启动用户和nginx工作用户是否一致**

1）查看linux启动用户，ps -ef |grep nginx，如下启动用户是nobody

 ![img](https://img-blog.csdn.net/20180604232707769)

2）将nginx.config的user改为和启动用户一致，改为root

![img](https://img-blog.csdn.net/20180604232716744)



2、browserHistory路由泛解析

```
location ^~ / {
		try_files $uri $uri/ /index.html;
}
```

3、多个二级域名

```
server_name  ~^(?<subdomain>.+)\.metuwang\.com$; #服务域名,可以有多个,用空格隔开
root /root/$subdomain;
```

4、centos7 nginx Failed to read PID from file /run/nginx.pid: Invalid argument 解决方法

提示错误
Failed to read PID from file /run/nginx.pid: Invalid argument

看到好多说删掉改nginx.pid 文件的，试之，无效。

后来找到了一个方法：
```
mkdir -p /etc/systemd/system/nginx.service.d
printf "[Service]\nExecStartPost=/bin/sleep 0.1\n" > /etc/systemd/system/nginx.service.d/override.conf
```

然后
```
systemctl daemon-reload
systemctl restart nginx.service
```
