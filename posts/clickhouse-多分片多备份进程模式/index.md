# ClickHouse 多分片多备份进程模式


基于原生安装方式进行部署

<!--more-->

部署进程模式下的ClickHouse

<!--more-->

## 服务器

- s1r1（分片1副本1）、s3r2（分片3副本2） 位于 s1 服务器
- s2r1（分片2副本1）、s1r2 （分片1副本2）位于 s2 服务器
- s3r1（分片3副本1）、s2r2（分片2副本2） 位于 s3 服务器
- zoo1（位于s1）、zoo2（位于s2）、zoo3 （位于s3）为 zookeeper集群


> 注意端口限制，有些可能需要手动设置防火墙放行端口


## 设置Host

在 s1、s2、s3中的/etc/hosts 中添加以下内容（需替换为自己的IP）

```bash
s1-ip s1
s2-ip s2
s3-ip s3
```


## 配置Zookeeper

### 下载

[Zookeeper官网](https://zookeeper.apache.org/releases.html)

```bash
# 使用国内镜像源，下载前请确认当前版本在镜像中存在
wget https://mirrors.bfsu.edu.cn/apache/zookeeper/zookeeper-3.6.2/apache-zookeeper-3.6.2-bin.tar.gz
tar -zxf apache-zookeeper-3.6.2-bin.tar.gz && mv apache-zookeeper-3.6.2-bin zookeeper
cd zookeeper && cp conf/zoo_sample.cfg conf/zoo.cfg && vim conf/zoo.cfg
```

### 配置

修改 `conf/zoo.cfg` 内容为：

```properties
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just
# example sakes.
dataDir=/data/zookeeper
# the port at which the clients will connect
clientPort=2181
# the maximum number of client connections.
# increase this if you need to handle more clients
#maxClientCnxns=60
#
# Be sure to read the maintenance section of the
# administrator guide before turning on autopurge.
#
# http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
#
# The number of snapshots to retain in dataDir
#autopurge.snapRetainCount=3
# Purge task interval in hours
# Set to "0" to disable auto purge feature
#autopurge.purgeInterval=1

## Metrics Providers
#
# https://prometheus.io Metrics Exporter
#metricsProvider.className=org.apache.zookeeper.metrics.prometheus.PrometheusMetricsProvider
#metricsProvider.httpPort=7000
#metricsProvider.exportJvmInfo=true

## Cluster 如果是云服务器需要使用内网地址ip或在白名单中的hostname
server.1=s1:2888:3888
server.2=s2:2888:3888
server.3=s3:2888:3888
```

- 修改`conf/log4j.properties` `zookeeper.log.dir=/data/log/zookeeper`
- 在`～/.bashrc`中添加 `export ZOO_LOG_DIR=/data/log/zookeeper`
- 在`dataDir`对应目录下新建`myid`文件，分别在s1、s2、s3上填入 1 2 3

### 启动

```bash
bin/zkServer.sh start
```

执行命令验证安装结果：

```bash
bin/zkServer.sh status
```


## 安装ClickHouse

[官方文档](https://clickhouse.tech/docs/zh/getting-started/install/)

```bash
sudo apt-get install apt-transport-https ca-certificates dirmngr
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv E0C56BD4

echo "deb https://repo.clickhouse.tech/deb/stable/ main/" | sudo tee \
    /etc/apt/sources.list.d/clickhouse.list
sudo apt-get update

sudo apt-get install -y clickhouse-server clickhouse-client

sudo service clickhouse-server start
clickhouse-client
```

手动安装
```bash
wget https://mirrors.tuna.tsinghua.edu.cn/clickhouse/deb/stable/main/clickhouse-common-static_20.9.7.11_amd64.deb
wget https://mirrors.tuna.tsinghua.edu.cn/clickhouse/deb/stable/main/clickhouse-client_20.9.7.11_all.deb
wget https://mirrors.tuna.tsinghua.edu.cn/clickhouse/deb/stable/main/clickhouse-server_20.9.7.11_all.deb
dpkg -i clickhouse-common-static_20.9.7.11_amd64.deb
dpkg -i clickhouse-client_20.9.7.11_all.deb
dpkg -i clickhouse-server_20.9.7.11_all.deb
```

## ClickHouse工作目录

```bash
# s1
├── data_p.csv				#（测试用）示例数据
├── s1r1
│   ├── config.xml		# 配置文件
│   ├── metrika.xml		# 分片配置文件
├── s3r2
│   ├── config.xml
│   ├── metrika.xml
└── users.xml

# s2
├── s1r2
│   ├── config.xml
│   ├── metrika.xml
├── s2r1
│   ├── config.xml
│   ├── metrika.xml
└── users.xml

# s3
├── s2r2
│   ├── config.xml
│   ├── metrika.xml
├── s3r1
│   ├── config.xml
│   ├── metrika.xml
└── users.xml
```

配置中tcp访问端口如下：

- s1r1:9111
- s3r2:9132
- s2r1:9121
- s1r2:9112
- s3r1:9131
- s2r2:9122


配置中http访问端口（datagrip工具使用此端口通信）如下：

- s1r1:9011
- s3r2:9032
- s2r1:9021
- s1r2:9012
- s3r1:9031
- s2r2:9022

## 运行ClickHouse

停止默认clickhouse
```bash
sudo service clickhouse stop
```
运行clickhouse
```bash
# s1
nohup clickhouse-server --config-file=/data/ch/s1r1/config.xml > /data/log/ch/s1r1.log 2>&1 &
nohup clickhouse-server --config-file=/data/ch/s3r2/config.xml > /data/log/ch/s3r2.log 2>&1 & 

# s2
nohup clickhouse-server --config-file=/data/ch/s1r2/config.xml > /data/log/ch/s1r2.log 2>&1 &
nohup clickhouse-server --config-file=/data/ch/s2r1/config.xml > /data/log/ch/s2r1.log 2>&1 &

# s3
nohup clickhouse-server --config-file=/data/ch/s2r2/config.xml > /data/log/ch/s2r2.log 2>&1 &
nohup clickhouse-server --config-file=/data/ch/s3r1/config.xml > /data/log/ch/s3r1.log 2>&1 &
```

> 停止运行： ps -A | grep clickhouse | grep -v grep | awk '{print $1}' | xargs kill 



## 自定义用户名和密码

** 在metrika.xml中需要设置密码，注意同步 **

在users.xml中有详细的注释说明了如何添加用户、密码、用户权限等配置方法。（配置文件中已添加了以下内容，需要其他账户可参考以下步骤）

添加defualt用户密码和新建一个tom用户，由于密码存储在文件，因此推荐使用sha256编码后放入文件：

```bash
echo -n "default" | sha256sum | tr -d '-'
# 37a8eec1ce19687d132fe29051dca629d164e2c4958ba141d5f4133a33f0688f  

echo -n "tom-password" | sha256sum | tr -d '-'
# d8c862690b30f6f2add244327715cb08ac926c7c2fb4fcbb7694650bfde5b672  
```

default用户密码为default，tom的密码为tom-password，添加至users.xml，在 users/default中删除password项，添加：

```xml
<password_sha256_hex>37a8eec1ce19687d132fe29051dca629d164e2c4958ba141d5f4133a33f0688f</password_sha256_hex>
```

在users/下添加：

```xml
<tom>
 <password_sha256_hex>d8c862690b30f6f2add244327715cb08ac926c7c2fb4fcbb7694650bfde5b672</password_sha256_hex>
  <profile>default</profile>
  <quota>default</quota>
  <networks incl="networks" replace="replace">
    <ip>::/0</ip>
  </networks>
</tom>
```

登陆数据库：

```bash
clickhouse-client --host s1 -u default --password default --port 9111
clickhouse-client --host s1 --user tom --password tom-password --port 9111
```


## 测试
登陆任意副本数据库，创建测试表
```sql
-- DROP TABLE p_local ON CLUSTER p_3shards_2replicas
CREATE TABLE p_local ON CLUSTER p_3shards_2replicas
(
    ozone Int64,
    particullate_matter Int8,
    carbon_monoxide Int8,
    sulfure_dioxide Int8,
    nitrogen_dioxide Int8,
    longitude Float64,
    latitude Float64,
    timestamp DateTime
) ENGINE = ReplicatedMergeTree('/clickhouse/default/tables/p/{shard}','{replica}')
PARTITION BY toYYYYMM(timestamp)
ORDER BY timestamp
PRIMARY KEY timestamp
```

为表`p_local`创建分布表`p`
```sql
-- DROP TABLE p ON CLUSTER p_3shards_2replicas
CREATE TABLE p ON CLUSTER p_3shards_2replicas AS p_local 
ENGINE = Distributed(p_3shards_2replicas, default, p_local, rand())
```

退出数据库交互界面，在终端中执行语句导入数据
```bash
clickhouse-client --host s1 -u default --password default --port 9111 --query "INSERT INTO p FORMAT CSV" < data_p.csv
```

删除测试表
```sql
drop table p on cluster p_3shards_2replicas
drop table p_local on cluster p_3shards_2replicas
```

## 配置kafka

### 下载

[Kafka官网](https://kafka.apache.org/downloads)

```bash
# 使用国内镜像源，下载前请确认当前版本在镜像中存在
wget https://mirrors.bfsu.edu.cn/apache/kafka/2.6.0/kafka_2.13-2.6.0.tgz
tar -zxf kafka_2.13-2.6.0.tgz && mv kafka_2.13-2.6.0 kafka
```

### 配置

修改配置

```bash
cd kafka && vim config/server.properties
```

通用配置

```properties
# A comma separated list of directories under which to store log files
log.dirs=/data/log/kafka

# root directory for all kafka znodes.
zookeeper.connect=s1:2181,s2:2181,s3:2181
```

分节点配置

- s1

    ```properties
    broker.id=1
    listeners=PLAINTEXT://s1:9092
    ```

- s2

  ```properties
  broker.id=2
  listeners=PLAINTEXT://s2:9092
  ```

- s3

  ```properties
  broker.id=3
  listeners=PLAINTEXT://s3:9092
  ```

### 启动

```bash
./bin/kafka-server-start.sh -daemon ./config/server.properties
```

查看topic
```bash
bin/kafka-topics.sh --list --bootstrap-server s1:9092
```


## 添加kafka缓冲

创建kafka主题
```bash
# ./bin/kafka-topics.sh --delete --bootstrap-server s1:9092 --topic topic-p
./bin/kafka-topics.sh --create --bootstrap-server s1:9092 --replication-factor 2 --partitions 6 --topic topic-p
```

创建kafka消费表
```sql
-- DROP TABLE p_stream ON CLUSTER p_3shards_2replicas
CREATE TABLE p_stream ON CLUSTER p_3shards_2replicas
(
    ozone Int64,
    particullate_matter Int8,
    carbon_monoxide Int8,
    sulfure_dioxide Int8,
    nitrogen_dioxide Int8,
    longitude Float64,
    latitude Float64,
    timestamp DateTime
) ENGINE = Kafka()
SETTINGS
    kafka_broker_list = 's1:9092',
    kafka_topic_list = 'topic-p',
    kafka_group_name = 'clickhouse',
    kafka_format = 'JSONEachRow',
    kafka_row_delimiter = '\n'
```
> 添加 kafka_skip_broken_messages = N 跳过非法消息

创建本地表
```sql
-- DROP TABLE p_local ON CLUSTER p_3shards_2replicas
CREATE TABLE p_local ON CLUSTER p_3shards_2replicas
(
    ozone Int64,
    particullate_matter Int8,
    carbon_monoxide Int8,
    sulfure_dioxide Int8,
    nitrogen_dioxide Int8,
    longitude Float64,
    latitude Float64,
    timestamp DateTime
) ENGINE = ReplicatedMergeTree('/clickhouse/tables/p/{shard}','{replica}')
PARTITION BY toYYYYMM(timestamp)
ORDER BY timestamp
PRIMARY KEY timestamp
```

创建分布表
```sql
-- DROP TABLE p ON CLUSTER p_3shards_2replicas
-- CREATE TABLE p ON CLUSTER p_3shards_2replicas AS p_local ENGINE = Distributed(p_3shards_2replicas, default, p_local, rand())
CREATE TABLE p ON CLUSTER p_3shards_2replicas 
(
    ozone Int64,
    particullate_matter Int8,
    carbon_monoxide Int8,
    sulfure_dioxide Int8,
    nitrogen_dioxide Int8,
    longitude Float64,
    latitude Float64,
    timestamp DateTime
)
ENGINE = Distributed(p_3shards_2replicas, default, p_local, rand())
```

创建物化视图
```sql
-- DROP TABLE p_consumer ON CLUSTER p_3shards_2replicas
CREATE MATERIALIZED VIEW p_consumer ON CLUSTER p_3shards_2replicas TO p AS SELECT * FROM p_stream;
```

发送数据
```bash
./bin/kafka-console-producer.sh --broker-list s1:9092 --topic topic-p
```

```json
{"ozone":1, "particullate_matter":69, "carbon_monoxide":57, "sulfure_dioxide":67, "nitrogen_dioxide":83, "longitude":10.189355309192706, "latitude":56.18210217593679, "timestamp":"2014-09-01 00:05:00"}
```

---

> 作者: ingbyr  
> URL: https://ingbyr.github.io/posts/clickhouse-%E5%A4%9A%E5%88%86%E7%89%87%E5%A4%9A%E5%A4%87%E4%BB%BD%E8%BF%9B%E7%A8%8B%E6%A8%A1%E5%BC%8F/  

