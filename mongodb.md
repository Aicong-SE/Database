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

# Mongo配置

## 进程管理

```shell
processManagement: 
	fork: true  # 运行在后台
	pidFilePath: /var/run/mongodb/mongod.pid  # PID 文件路径
```



## 网络

```shell
net:
	port: 27017   # 端口号
	bindIp: 127.0.0.1  # 绑定外网多个 IP 用逗号分割
	maxIncomingConnections: 65536  # 进程允许最大连接数
	wireObjectCheck: true  # 当客户端写入数据时，检测数据的有效性（BSON）
	ipv6: false
```



## 存储

```shell
storage: 
	dbPath: /var/lib/mongo  # mongod 进程存储数据目录，此配置仅对 mongod 进程有效
	indexBuildRetry: true  # 当构建索引时 mongod 意外关闭，那么再次启动是否重新构建索引
	engine: mmapv1  # 存储引擎类型，mongodb 3.0 之后支持 “mmapv1”、“wiredTiger” 两种引擎
	journal: true  # 是否开启 journal 日志持久存储，journal 日志用来数据恢复，是 mongod 最基础的特性，通常用于故障恢复。64 位系统默认为 true，32 位默认为 false，建议开启，仅对 mongod 进程有效。
	directoryPerDB: false  # 是否将不同 DB 的数据存储在不同的目录中
	syncPeriodSecs: 60  # mongod 使用 fsync 操作将数据 flush 到磁盘的时间间隔, 单位：s
	mmapv1:
        quota:
            enforced: false  # 配额管理，是否限制每个 DB 所能持有的最大文件数量
            maxFilesPerDB: 8  # 如果 enforce 开启，每个 DB 所持有的存储文件不会超过此阀值
            smallFiles: false  # 是否使用小文件存储数据；如果此值为 true mongod 将会限定每个数据文件的大小为 512M（默认最大为 2G），journal 降低到 128M（默认为 1G）。如果 DB 的数据量较大，将会导致每个 DB 创建大量的小文件，这对性能有一定的影响。在 production 环境下，不建议修改此值，在测试时可以设置为 true，节约磁盘。
        journal:  # 仅对 MMAPV1 引擎
            commitIntervalMs: 100  # mongod 进程提交 journal 日志的时间间隔，即 fsync 的间隔, 单位：ms
            nsSize: 16  # 每个 database 的 namespace 文件的大小, 单位： M, 最大值可以设置为 2048
    wiredTiger:
        engineConfig: 
            cacheSizeGB: 8  #wiredTiger 缓存工作集（working set）数据的内存大小，单位：GB, 此值决定了 wiredTiger 与 mmapv1 的内存模型不同，它可以限制 mongod 对内存的使用量，而 mmapv1 则不能
            journalCompressor: snappy  # journal 日志的压缩算法，可选值为 “none”、“snappy”、“zlib”。
            directoryForIndexes: false  # 是否将索引和 collections 数据分别存储在 dbPath 单独的目录中。
        collectionConfig:
        	blockCompressor: snappy  # collection 数据压缩算法，可选值 “none”、“snappy”、“zlib”。
       	indexConfig:
       		prefixCompression: true  # 是否对索引数据使用 “前缀压缩”（prefix compression，一种算法）。
```



## 性能分析器

```shell
operationProfiling:
	slowOpThresholdMs: 100  # 数据库 profiler 判定一个操作是 “慢查询” 的时间阀值，单位: ms,
	mode: off  # 数据库 profiler 级别，操作的性能信息将会被写入日志文件中，数据库 profiling 会影响性能，建议只在性能调试阶段开启。此参数仅对 mongod 有效。可选值：1）off：关闭 profiling 2）slowOp：on，只包含慢操作日志 3）all：on，记录所有操作 
```



## 主从复制

```shell
replication:
	oplogSizeMB: 10240  # replication 操作日志的最大尺寸，单位：MB
	enableMajorityReadConcern: false  # 是否开启 readConcern 的级别为 “majority”, 只有开启此选项，才能在 read 操作中使用 “majority”。（3.2 + 版本）
	replSetName: <无默认值>  # “复制集” 的名称，复制集中的所有 mongd 实例都必须有相同的名字，sharding 分布式下，不同的 sharding 应该使用不同的 replSetName
	secondaryIndexPrefetch: all  # 只对 mmapv1 存储引擎有效。复制集中的 secondary，从 oplog 中运用变更操作之前，将会先把索引加载到内存中 1）none：secondaries 不将索引数据加载到内容 2）all：sencondaries 将此操作有关的索引数据加载到内存 3）_id_only：只加载_id 索引
	localPingThresholdMs: 15  # ping 时间，单位：ms, mongos 用来判定将客户端 read 请求发给哪个 secondary。仅对 mongos 有效。
```

## sharding 架构

```shell
sharding:
	clusterRole: <无默认值>  # 在 sharding 集群中，此 mongod 实例的角色，可选值：1、configsvr：此实例为 config server，此实例默认侦听 27019 端口 2、shardsvr：此实例为 shard（分片），侦听 27018 端口
	archiveMovedChunks: true  # 当 chunks 因为 “负载平衡” 而迁移到其他节点时，mongod 是否将这些 chunks 归档，并保存在 dbPath 下 “moveChunk” 目录下，mongod 不会删除 moveChunk 下的文件。
	autoSplit: true  # 是否开启 sharded collections 的自动分裂
	configDB: <无默认值>  # 设定 config server 的地址列表，每个 server 地址之间以 “,” 分割，通常 sharded 集群中指定 1 或者 3 个 config server。所有的 mongos 实例必须配置一样，否则可能带来不必要的问题。
	chunkSize: 64  # sharded 集群中每个 chunk 的大小，单位：MB
```

## 系统日志

```shell
systemLog:
	verbosity: 0  # 日志级别, 0: 包含 “info” 信息，1~5，即大于 0 的值均会包含 debug 信息
	quiet: true  # "安静"，此时 mongod/mongos 将会尝试减少日志的输出量。不建议在 production 环境下开启，否则将会导致跟踪错误比较困难。
	traceAllExceptions: true  # 打印异常详细信息。
	path: logs/mongod.log  # 日志路径
	logAppend: false  # 如果为 true，当 mongod/mongos 重启后，将在现有日志的尾部继续添加日志。否则，将会备份当前日志文件，然后创建一个新的日志文件；默认为 false。
	logRotate: rename  # 日志 “回转”，防止一个日志文件特别大，则使用 logRotate 指令将文件 “回转”，1）rename：重命名日志文件，默认值。 2）reopen：使用 linux 日志 rotate 特性，关闭并重新打开此日志文件，可以避免日志丢失，但是 logAppend 必须为 true。
	destination: file  # 日志输出目的地，可以指定为 “file” 或者“syslog”，表述输出到日志文件，如果不指定，则会输出到标准输出中（standard output）
```



## 与安全有关的配置

```shell
security:
	authorization: disabled  # disabled 或者 enabled, 表示是否开启用户访问控制
	clusterAuthMode: keyFile  # 集群中 members 之间的认证模式，可选值为 “keyFile”、“sendKeyFile”、“sendX509”、“x509”
	keyFile: /srv/mongodb/keyfile  # 当 clusterAuthMode 为 “keyFile” 时，此参数指定 keyfile 的位置，mongodb 需要有访问此文件的权限。
	javascriptEnabled: true  # 表示是否关闭 server 端的 javascript 功能，就是是否允许 mongod 上执行 javascript 脚本，
setParameter:  # 允许指定一些的 Server 端参数，这些参数不依赖于存储引擎和交互机制，只是微调系统的运行状态，比如 “认证机制”、“线程池参数” 等
	enableLocalhostAuthBypass: true  # 表示是否开启 “localhost exception”，对于 sharding cluster 而言，我们倾向于在 mongos 上开启，在 shard 节点的 mongod 上关闭。
	authenticationMechanisms: SCRAM-SHA-1  # 认证机制，可选值为 “SCRAM-SHA-1”、“MONGODB-CR”、“PLAN” 等，建议为“SCRAM-SHA-1”，对 mongod/mongos 有效；一旦选定了认证机制，客户端访问 databases 时需要与其匹配才能有效。
```



## 性能有关的参数

```shell
setParameter: 
	connPoolMaxShardedConnsPerHost: 200   # 表示当前 mongos 或者 shard 与集群中其他 shards 链接的链接池的最大容量，此值我们通常不会调整。连接池的容量不会阻止创建新的链接，但是从连接池中获取链接的个数不会超过此值。维护连接池需要一定的开支，保持一个链接也需要占用一定的系统资源。
	connPoolMaxConnsPerHost: 200   # 同上，表示 mongos 或者 mongod 与其他 mongod 实例之间的连接池的容量，根据 host 限定。
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

