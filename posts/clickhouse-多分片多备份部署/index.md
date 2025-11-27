# ClickHouse 多分片多备份部署


基于Docker的部署方式

<!--more-->

## 安装

- 安装 docker 
- 安装 docker-compose
- docker pull yandex/clickhouse-server
- docker pull zookeeper


## 服务器设置

- s1r1（分片1副本1）、s3r2（分片3副本2） 位于 s1 服务器
- s2r1（分片2副本1）、s1r2 （分片1副本2）位于 s2 服务器
- s3r1（分片3副本1）、s2r2（分片2副本2） 位于 s3 服务器
- zoo1（位于s1）、zoo2（位于s2）、zoo3 （位于s3）为 zookeeper集群

/etc/hosts 映射信息，实际部署时需要替换为自己的服务器IP，本文只在s1服务器中添加hosts信息，后文中若未指明服务器名，则默认在s1中执行：

```bash
[S1] s1 
[S2] s2 
[S3] s3
[S1] s1r1 
[S2] s1r2
[S2] s2r1
[S3] s2r2
[S3] s3r1
[S1] s3r2
[S1] zoo1
[S2] zoo2
[S3] zoo3
```

## zookeeper 设置

使用 docker-compose 进行部署，docker-compose.yaml 文件内容如下：

- zoo1（s1服务器）:

  ```yaml
  version: '3.1'
  
  services:
      zoo1:
          image: zookeeper
          restart: always
          hostname: zoo1
          ports:
              - 2181:2181
              - 2888:2888
              - 3888:3888
          environment:
              ZOO_MY_ID: 1
              ZOO_SERVERS: server.1=0.0.0.0:2888:3888;2181 server.2=zoo2:2888:3888;2181 server.3=zoo3:2888:3888;2181
          extra_hosts:
              - "zoo1:[S1]"
              - "zoo2:[S2]"
              - "zoo3:[S3]"
  ```

- zoo2（s2服务器）：

  ```yaml
  version: '3.1'
  
  services:
      zoo2:
          image: zookeeper
          restart: always
          hostname: zoo2
          ports:
              - 2181:2181
              - 2888:2888
              - 3888:3888
          environment:
              ZOO_MY_ID: 2
              ZOO_SERVERS: server.1=zoo1:2888:3888;2181 server.2=0.0.0.0:2888:3888;2181 server.3=zoo3:2888:3888;2181
          extra_hosts:
              - "zoo1:[S1]"
              - "zoo2:[S2]"
              - "zoo3:[S3]"
  ```

- zoo3（s3服务器）:

  ```yaml
  version: '3.1'
  
  services:
      zoo3:
          image: zookeeper
          restart: always
          hostname: zoo3
          ports:
              - 2181:2181
              - 2888:2888
              - 3888:3888
          environment:
              ZOO_MY_ID: 3
              ZOO_SERVERS: server.1=zoo1:2888:3888;2181 server.2=zoo2:2888:3888;2181 server.3=0.0.0.0:2888:3888;2181
          extra_hosts:
              - "zoo1:[S1]"
              - "zoo2:[S2]"
              - "z003:[S3]"
  ```

在s1、s2、s3上分别启动zookeeper：

```bash
docker-compose up -d
```

> 停止容器 docker-compose stop
>
> 删除容器 docker-compose rm
>
> 显示容器 docker-compose ps
>
> 重启容器 docker-compose restart

## 配置数据库

### 导出配置文件模版

首先启动一个临时容器来获取配置文件config.xml：

```bash
docker run -itd --rm --name db yandex/clickhouse-server 
```

拷贝config.xml和users.xml：

```bash
docker cp db:/etc/clickhouse-server/config.xml ./config.xml
docker cp db:/etc/clickhouse-server/users.xml ./users.xml
```

停止容器：

```bash
docker stop db
```

`config.xml`在三台服务器上均需要2份（2备份），`users.xml`在三台服务器中各需要一份（2备份共享该配置）

### 添加用户和密码

在users.xml中有详细的注释说明了如何添加用户、密码、用户权限等配置方法。这里添加defualt用户密码和新建一个tom用户，由于密码存储在文件，因此推荐使用sha256编码后放入文件：

```bash
root@mq-228 ~/i/chdb# echo -n "default" | sha256sum | tr -d '-'···
37a8eec1ce19687d132fe29051dca629d164e2c4958ba141d5f4133a33f0688f  

root@mq-228 ~/i/chdb# echo -n "tom-password" | sha256sum | tr -d '-'
d8c862690b30f6f2add244327715cb08ac926c7c2fb4fcbb7694650bfde5b672  
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

随后即可通过以下命令登陆数据库：

```bash
clickhouse-client -u default --password default --port 9001 
clickhouse-client -u tom --password tom-password --port 9001
```

### 设置监听网段

confix.xml中取消 listen_host 行的注释：

```xml
<!-- Listen specified host. use :: (wildcard IPv6 address), if you want to accept connections both with IPv4 and IPv6 from everywhere. -->
<listen_host>::</listen_host>
```

数据库文件主要由 /etc/clickhouse-server/users.xml 、/etc/clickhouse-server/config.xml 、/etc/metrika.xml 文件构成，其中metrika.xml的内容会覆盖config.xml对应内容，由用户自行创建，主要用于设置分片和备份。metrika.xml的文件名和路径可以在config.xml中自定义。

### s1服务器

工作目录下的文件列表参考：

```bash
root@mq-227 ~/i/chdb# tree .         
.
├── check_table_p.sh
├── config-s1r1.xml
├── config-s3r2.xml
├── create_table.sh
├── data_p.csv
├── delete_table_p.sh
├── docker-compose.yaml
├── metrika-s1r1.xml
├── metrika-s3r2.xml
├── metrika.xml
├── query_table.sh
└── users.xml
```

metrika-s1r1.xml内容（稍后在docker-compose.xml中会映射到容器内的metrika.xml）：

```xml
<yandex>
  
<!-- 集群配置 -->
<clickhouse_remote_servers>
    <!-- 3分片2备份，perftest_3shards_2replicas 为唯一ID-->
    <perftest_3shards_2replicas>
       <shard>
            <internal_replication>false</internal_replication>
            <replica>
                <host>s1r1</host>
                <port>9001</port>
                <user>default</user>
                <password>default</password>
            </replica>
            <replica>
                <host>s1r2</host>
                <port>9002</port>
                <user>default</user>
                <password>default</password>
            </replica>
        </shard>
        <shard>
            <internal_replication>false</internal_replication>
            <replica>
                <host>s2r1</host>
                <port>9001</port>
                <user>default</user>
                <password>default</password>
            </replica>
            <replica>
                <host>s2r2</host>
                <port>9002</port>
                <user>default</user>
                <password>default</password>
            </replica>
        </shard>
        <shard>
            <internal_replication>false</internal_replication>
            <replica>
                <host>s3r1</host>
                <port>9001</port>
                <user>default</user>
                <password>default</password>
            </replica>
            <replica>
                <host>s3r2</host>
                <port>9002</port>
                <user>default</user>
                <password>default</password>
            </replica>
        </shard>
    </perftest_3shards_2replicas>
</clickhouse_remote_servers>

<!-- ZooKeeper 配置 -->
<zookeeper-servers>
    <node index="1">
        <host>zoo1</host>
        <port>2181</port>
    </node>
    <node index="2">
        <host>zoo2</host>
        <port>2181</port>
    </node>
    <node index="3">
        <host>zoo3</host>
        <port>2181</port>
    </node>
		<session_timeout_ms>30000</session_timeout_ms>
    <operation_timeout_ms>10000</operation_timeout_ms> 
</zookeeper-servers>

<!-- 环境变量，不同分片需要替换ID，在创建表时可以引用该变量从而实现建表语句统一-->
<macros>
    <shard>s1</shard>
    <replica>r1</replica>
</macros>

</yandex> 
```

复制之前导出的`config.xml`并重命名为config-s1r1.xml （后文同），修改其TCP连接端口和同步端口：

```xml
<tcp_port>9001</tcp_port>
<interserver_http_port>9011</interserver_http_port>
```

metrika-s3r2.xml 与 `metrika-s1r1.xml` 内容相同，只需修改环境变量配置部分：

```xml
<macros>
    <shard>s3</shard>
    <replica>r2</replica>
</macros>
```

config-s3r2.xml 修改其TCP连接端口和同步端口:

```xml
<tcp_port>9002</tcp_port>
<interserver_http_port>9013</interserver_http_port>
```

docker-compose.xml内容：

```yaml
version: '3.1'
services:

    chdb-s1r1:
        image: yandex/clickhouse-server:latest
        hostname: s1r1
        ports:
            - 9001:9001
            - 9011:9011
        volumes:
            - /root/iot/chdb/users.xml:/etc/clickhouse-server/users.xml
            - /root/iot/chdb/config-s1r1.xml:/etc/clickhouse-server/config.xml
            - /root/iot/chdb/metrika-s1r1.xml:/etc/metrika.xml
            - /root/iot/chdb/s1r1:/var/lib/clickhouse 
        extra_hosts:
            - "s1r1:[S1]"
            - "s1r2:[S2]"
            - "s2r1:[S2]"
            - "s2r2:[S3]"
            - "s3r1:[S3]"
            - "s3r2:[S1]"
            - "zoo1:[S1]"
            - "zoo2:[S2]"
            - "zoo3:[S3]"

    chdb-s3r2: 
        image: yandex/clickhouse-server:latest
        hostname: s3r2
        ports:
            - 9002:9002
            - 9013:9013
        volumes:
            - /root/iot/chdb/users.xml:/etc/clickhouse-server/users.xml
            - /root/iot/chdb/config-s3r2.xml:/etc/clickhouse-server/config.xml
            - /root/iot/chdb/metrika-s3r2.xml:/etc/metrika.xml
            - /root/iot/chdb/s3r2:/var/lib/clickhouse
        extra_hosts:
            - "s1r1:[S1]"
            - "s1r2:[S2]"
            - "s2r1:[S2]"
            - "s2r2:[S3]"
            - "s3r1:[S3]"
            - "s3r2:[S1]"
            - "zoo1:[S1]"
            - "zoo2:[S2]"
            - "zoo3:[S3]"
```

`volumes`选项中的本地路径需要按实际部署路径修改（下同），其中最后一条为数据库数据本地持久化映射，可按需更改文件位置。

### s2服务器

metrika-s2r1.xml 与 metrika-s1r1.xml 类似，只需修改环境变量配置部分：

```xml
<macros>
    <shard>s2</shard>
    <replica>r1</replica>
</macros>
```

config-s2r1.xml 修改其TCP连接端口和同步端口:

```xml
<tcp_port>9001</tcp_port>
<interserver_http_port>9012</interserver_http_port>
```

metrika-s1r2.xml 与 metrika-s1r1.xml 类似，只需修改环境变量配置部分：

```xml
<macros>
    <shard>s1</shard>
    <replica>r2</replica>
</macros>
```

config-s1r2.xml 修改其TCP连接端口和同步端口:

```xml
<tcp_port>9002</tcp_port>
<interserver_http_port>9011</interserver_http_port>
```

docker-compose.xml内容：

```yaml
version: '3.1'
services:

    chdb-s2r1:
        image: yandex/clickhouse-server:latest
        hostname: s2r1
        ports:
            - 9001:9001
            - 9012:9012
        volumes:
            - /root/iot/chdb/users.xml:/etc/clickhouse-server/users.xml
            - /root/iot/chdb/config-s2r1.xml:/etc/clickhouse-server/config.xml
            - /root/iot/chdb/metrika-s2r1.xml:/etc/metrika.xml
            - /root/iot/chdb/s2r1:/var/lib/clickhouse
        extra_hosts:
            - "s1r1:[S1]"
            - "s1r2:[S2]"
            - "s2r1:[S2]"
            - "s2r2:[S3]"
            - "s3r1:[S3]"
            - "s3r2:[S1]"
            - "zoo1:[S1]"
            - "zoo2:[S2]"
            - "zoo3:[S3]"

    chdb-s1r2: 
        image: yandex/clickhouse-server:latest
        hostname: s1r2
        ports:
            - 9002:9002
            - 9011:9011
        volumes:
            - /root/iot/chdb/users.xml:/etc/clickhouse-server/users.xml
            - /root/iot/chdb/config-s1r2.xml:/etc/clickhouse-server/config.xml
            - /root/iot/chdb/metrika-s1r2.xml:/etc/metrika.xml
            - /root/iot/chdb/s1r2:/var/lib/clickhouse
        extra_hosts:
            - "s1r1:[S1]"
            - "s1r2:[S2]"
            - "s2r1:[S2]"
            - "s2r2:[S3]"
            - "s3r1:[S3]"
            - "s3r2:[S1]"
            - "zoo1:[S1]"
            - "zoo2:[S2]"
            - "zoo3:[S3]"
```



### s3服务器

metrika-s3r1.xml 需修改环境变量配置部分：

```xml
<macros>
    <shard>s3</shard>
    <replica>r1</replica>
</macros>
```

config-s3r1.xml 修改其TCP连接端口和同步端口:

```xml
<tcp_port>9001</tcp_port>
<interserver_http_port>9013</interserver_http_port>
```

metrika-s2r2.xml 修改环境变量配置部分：

```xml
<macros>
    <shard>s2</shard>
    <replica>r2</replica>
</macros>
```

config-s2r2.xml 修改其TCP连接端口和同步端口:

```xml
<tcp_port>9002</tcp_port>
<interserver_http_port>9012</interserver_http_port>
```

docker-compose.xml内容：

```yaml
version: '3.1'
services:

    chdb-s3r1:
        image: yandex/clickhouse-server:latest
        hostname: s3r1
        ports:
            - 9001:9001
            - 9013:9013
        volumes:
            - /root/iot/chdb/users.xml:/etc/clickhouse-server/users.xml
            - /root/iot/chdb/config-s3r1.xml:/etc/clickhouse-server/config.xml
            - /root/iot/chdb/metrika-s3r1.xml:/etc/metrika.xml
            - /root/iot/chdb/s3r1:/var/lib/clickhouse
        extra_hosts:
            - "s1r1:[S1]"
            - "s1r2:[S2]"
            - "s2r1:[S2]"
            - "s2r2:[S3]"
            - "s3r1:[S3]"
            - "s3r2:[S1]"
            - "zoo1:[S1]"
            - "zoo2:[S2]"
            - "zoo3:[S3]"

    chdb-s2r2: 
        image: yandex/clickhouse-server:latest
        hostname: s2r2
        ports:
            - 9002:9002
            - 9012:9012
        volumes:
            - /root/iot/chdb/users.xml:/etc/clickhouse-server/users.xml
            - /root/iot/chdb/config-s2r2.xml:/etc/clickhouse-server/config.xml
            - /root/iot/chdb/metrika-s2r2.xml:/etc/metrika.xml
            - /root/iot/chdb/s2r2:/var/lib/clickhouse
        extra_hosts:
            - "s1r1:[S1]"
            - "s1r2:[S2]"
            - "s2r1:[S2]"
            - "s2r2:[S3]"
            - "s3r1:[S3]"
            - "s3r2:[S1]"
            - "zoo1:[S1]"
            - "zoo2:[S2]"
            - "zoo3:[S3]"
```



## 验证

连接至任意数据库，查询集群信息：

```bash
clickhouse-client -u default --password default --host s1r1 --port 9001 
```


## 示例

ClickHouse备份使用的是表级备份，要使用备份需要使用 [Replicated*表引擎](https://clickhouse.tech/docs/en/engines/table-engines/mergetree-family/replication/)，需要注意的是ClickHouse集群在执行 CREATE、 DROP、ATTACH、DETACH和RENAME时不会同步命令，因此创建表时需要在每个数据库实例中都进行创建。

在s1服务器上执行`s1/create_table_p.sh`脚本来创建表：

```bash
ports="9001 9002"
hosts="s1 s2 s3"
for port in $ports
do
    for host in $hosts
    do
        echo "Creating table on $host:$port"
        clickhouse-client -u default --password default --host $host --port $port --query \
        "CREATE TABLE p (
            ozone Int8,
            particullate_matter Int8,
            carbon_monoxide Int8,
            sulfure_dioxide Int8,
            nitrogen_dioxide Int8,
            longitude Float64,
            latitude Float64,
            timestamp DateTime
        ) ENGINE = ReplicatedMergeTree('/clickhouse/tables/p/{shard}','{replica}')
        ORDER BY timestamp
        PRIMARY KEY timestamp"
    done
done
```

为了使用分片，需要一个虚拟的统一入口表，ClickHouse中称为分布表（Distributed Table），可以简单认为是一个数据路由表。创建分布表：

```bash
clickhouse-client --host s1r1 -u default --password default --port 9001 --query "CREATE TABLE p_all AS p
ENGINE = Distributed(perftest_3shards_2replicas, default, p, rand())"
```

其中 perftest_3shards_2replicas 为之前定义的集群ID。然后导入测试数据：

```bash
clickhouse-client --host s1r1 -u default --password default --port 9001 --query "INSERT INTO p_all FORMAT CSV" < data_p.csv
```

查看数据分片存储的情况 `s1/check_table_p.sh`：

```bash
ports="9001 9002"
hosts="s1 s2 s3"
for port in $ports
do
    for host in $hosts
    do
        echo "Data from $host:$port"
        clickhouse-client -u default --password default --host $host --port $port --query "select count(*) from p"
    done
done
```

---

> 作者: ingbyr  
> URL: https://ingbyr.github.io/posts/clickhouse-%E5%A4%9A%E5%88%86%E7%89%87%E5%A4%9A%E5%A4%87%E4%BB%BD%E9%83%A8%E7%BD%B2/  

