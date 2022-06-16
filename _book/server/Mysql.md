##  Mysql



####安装

[Mysql手动下载地址](https://downloads.mysql.com/archives/community/)



**1 下载安装mysql的repo源**

```
# wget https://cdn.mysql.com//Downloads/MySQL-5.7/mysql-community-server-5.7.36-1.el7.x86_64.rpm
```

**2 安装mysql的repo源**

```
# yum -y install  mysql-community-server-5.7.36-1.el7.x86_64.rpm
```

**3 按照以下顺序开始安装**

```
# yum install mysql-server

# yum install mysql-devel

# yum install mysql

# rpm -qa | grep -i mysql
```

**4 服务开关操作**

```
#执行其中之一就可以

# service mysqld status   查看mysql当前的状态

# systemctl status mysqld

# service mysqld stop    停止mysql

# systemctl stop mysqld

# service mysqld restart   重启mysql

# systemctl restart mysqld

# service mysqld start    启动mysql

# systemctl start mysqld
```

**5 设置MySQL开机启动**

```
# systemctl enable mysqld 
```

**6 启动MySQL服务进程**

```
# 以下二选一

# systemctl start mysqld

# service mysqld start
```



####执行

#####1 登录mysql 二选一

```
sudo mysql -uroot -p
sudo mysql -uroot
```

退出控制台是"\q"， 退出mysql是“exit(回车)“。



#####2 修改密码

```
ALTER USER 'root'@'localhost' IDENTIFIED BY 'Zbb820502!';
```

```
alter user 'root'@'localhost' identified by 'Zbb820502!' password expire never;
alter user 'root'@'localhost' identified with mysql_native_password by 'Zbb820502!';
```

或者

```
set password for 'root'@'localhost'=password('Zbb820502!');
```



##### 3 创建用户

```
create user 'visitor'@'%' identified by 'Visitor820502!';
```



查看mysql版本

```
select version();
```

查看所有用户

```
SELECT User, Host FROM mysql.user;
```





#####4 开启mysql的远程访问

```
grant all privileges on *.* to 'root'@'localhost' identified by 'Zbb820502!' with grant option;

grant all privileges on *.* to 'visitor'@'%' identified by 'Visitor820502!' with grant option;
```

如果授权指定IP，可以将%换成指定IP即可。%代表允许用户远程访问数据库。

赋予用户所有特权

```
grant all on *.* to 'visitor'@'%';
```

显示用户的所有权限

```
show grants for 'visitor'@'%';
```





刷新权限表

```
flush privileges;
```



##### 5 开放端口

```
firewall-cmd --zone=public --add-port=3306/tcp --permanent

firewall-cmd --zone=public --add-port=8080/tcp --permanent
```

重新载入

```
firewall-cmd --reload
```



#####6 常见问题

1) 解决Navicat连接不上MySql服务器报错：Client does not support authentication protocol requested by server; consider upgrading MySQL client.

```
1、mysql -uroot -p

2、ALTER USER 'root'@'localhost' IDENTIFIED BY 'password' PASSWORD EXPIRE NEVER;
	password替换为mysql连接密码

3、FLUSH PRIVILEGES;
	使修改生效
```

2. Linux查看MySQL初始密码

   使用命令 cat /var/log/mysqld.log 查看该文件。

3) Navicat 连接Mysql提示：2003 - Can't connect to MySQL server on 'xx.xx.xx.xx' (60 "Connection refused")

   

