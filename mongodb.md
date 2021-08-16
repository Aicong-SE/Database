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

