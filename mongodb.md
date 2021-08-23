[toc]

# 安装

## Centos环境安装

1. 创建仓库文件

```
sudo vi /etc/yum.repos.d/mongodb-org-3.4.repo
```

然后复制下面配置,保存退出

> ```
> [mongodb-org-3.4]
> name=MongoDB Repository
> baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/3.4/x86_64/
> gpgcheck=1
> enabled=1
> gpgkey=https://www.mongodb.org/static/pgp/server-3.4.asc
> ```

2.  yun安装

```
sudo yum install -y mongodb-org
```

完成后修改配置文件,修改配置文件的 `bind_ip,` 默认是 `127.0.0.1 只限于本机连接`。所以安装完成后必须把这个修改为 0.0.0.0 ,否则通过别的机器是没法连接的!

```
sudo vi /etc/mongod.conf
```

3. 启动、停止、重启

MongoDB默认将数据文件存储在`/var/lib/mongo`目录，默认日志文件在`/var/log/mongodb`中。如果要修改,可以在 `/etc/mongod.conf` 配置中指定备用日志和数据文件目录。

```
启动
service mongod start

停止
service mongod stop

重启
service mongod restart
```

查看 mongodb 是否启动成功，查看日志文 /``var``/log/mongodb/mongod.log ，若出现 [initandlisten] waiting for connections on port <port>， 则正常

设置开机启动

```
chkconfig mongod on
```

4. 卸载移除 mongo

```
yum erase $(rpm -qa | grep mongodb-org)
```

5. 移除数据库文件和日志文件

```
rm -r /var/log/mongodb
rm -r /var/lib/mongo
```

## Ubuntu 20.04环境安装

1. 导入包管理系统使用的公钥

```
wget -qO - https://www.mongodb.org/static/pgp/server-5.0.asc | sudo apt-key add -
返回: ok
```

2. 为 MongoDB 创建一个列表文件

```
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/5.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-5.0.list
```

3. 重新加载本地包数据库

```
sudo apt-get update
```

4. 安装 MongoDB 包

```
sudo apt-get install -y mongodb-org
```

5. 启动 MongoDB

```
sudo systemctl start mongod
```

6. 验证 MongoDB 是否已成功启动

```
sudo systemctl status mongod
```

7. 停止 MongoDB

```
sudo systemctl stop mongod
```

8. 重启 MongoDB

```
sudo systemctl restart mongod
```

9. 开始使用 MongoDB

```
mongosh
```

数据目录： /var/lib/mongodb

日志目录： /var/log/mongodb

配置文件： /etc/mongod.conf

```
https://docs.mongodb.com/manual/reference/configuration-options/#std-label-conf-file
```



# 命令行使用

```shell
[root@instance-d0nk2r2c ~]# mongo
 
## 查看数据库
> show dbs;
 
## 查看数据库版本
> db.version();
 
## 常用命令帮助
> db.help();
```

## 创建用户

```shell
> use database_name  # 进入数据库
> db.createUser({  # 创建数据库
	user: "user_name",  # 用户名
	pwd: "password",  # 密码
	roles: [{role: "read", db: "database_name"}, "read"]  # 权限 可指定库，不指定则指定当前库
})
```

| role                 | 说明                                   |
| -------------------- | -------------------------------------- |
| read                 | 可读                                   |
| readWrite            | 可读可写                               |
| dbAdmin              | 允许管理指定数据库                     |
| userAdmin            | 允许在指定数据库中创建和修改用户       |
| clusterAdmin         | 允许对整个集群或数据库系统进行管理操作 |
| readAnyDatabase      | 可读所有数据库，除config 和 local      |
| readWriteAnyDatabase | 可读可写所有数据库，除config 和 local  |
| userAdminAnyDatabase | 允许在所有数据库中创建和修改用户       |
| dbAdminAnyDatabase   | 允许管理所有数据库                     |
| root                 | 超级账户，超级权限                     |

