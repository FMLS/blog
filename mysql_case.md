## 创建新用户
```
CREATE USER 'username'@'host' IDENTIFIED BY 'password';
```
* username: 你将创建的用户名,
* host: 指定该用户在哪个主机上可以登陆，此处的"localhost"，是指该用户只能在本地登录，不能在另外一台机器上远程登录，如果想远程登录的话，将"localhost"改为"%"，表示在任何一台电脑上都可以登录;也可以指定某台机器可以远程登录;
* password: 该用户的登陆密码,密码可以为空,如果为空则该用户可以不需要密码登陆服务器。

## 授权
```
GRANT privileges ON databasename.tablename TO 'username'@'host';
flush privileges;
```

* privileges: 用户的操作权限,如SELECT , INSERT , UPDATE 等, 如果要授予所有的权限则使用ALL
* databasename: 数据库名
* tablename: 表名,如果要授予该用户对所有数据库和表的相应操作权限则可用表示, 如*.*

## 设置密码

```
SET PASSWORD FOR 'username'@'host' = PASSWORD('newpassword');
```

## 删除用户

```
DROP USER 'username'@'host';
```

# 问题

## 创建用户后连接报错 ERROR 1045 (28000): Access denied for user 'bill'@'localhost' (using password: YES)

