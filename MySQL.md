[toc]

# 安装

## ubuntu 20.04

1. sudo apt-get install mysql-server

2. apt-get isntall mysql-client

3. sudo apt-get install libmysqlclient-dev

4. mysql -u root -p

# 忘记密码

## mysql 8.0以上

1. 通过配置文件查看进入 mysql 的 user 和 password 

```shell
> sudo cat /etc/mysql/debian.cnf
[client]
host     = localhost
user     = debian-sys-maint
password = ***
socket   = /var/run/mysqld/mysqld.sock
[mysql_upgrade]
host     = localhost
user     = debian-sys-maint       # 账户
password = ***                    # 密码
socket   = /var/run/mysqld/mysqld.sock
```

2. 进入 mysql 

```
> mysql -u debian-sys-maint -p ***
```

3. 将 root 用户的密码设置为空字符串

```
mysql> update user set authentication_string='' where user='root';
```

4. 刷新权限

```
mysql> flush privileges;
```

5. 退出 mysql

```
mysql> exit
```

6. 重启 mysql 服务

```
> sudo service mysql restart
```

7. 用 root 用户进入 mysql

```
> mysql -u root
```

8. 修改密码

```
mysql> alter user 'root'@'localhost' identified by 'new_password';
```



