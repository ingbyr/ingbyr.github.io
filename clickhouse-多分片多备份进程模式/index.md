# ClickHouse 多分片多备份进程模式


基于原生安装方式进行部署

<!--more-->

## 服务器

- s1r1（分片1副本1）、s3r2（分片3副本2） 位于 s1 服务器
- s2r1（分片2副本1）、s1r2 （分片1副本2）位于 s2 服务器
- s3r1（分片3副本1）、s2r2（分片2副本2） 位于 s3 服务器
- zoo1（位于s1）、zoo2（位于s2）、zoo3 （位于s3）为 zookeeper集群

本文中的IP如下：

- s1 -> 10.88.76.227 
- s2 ->10.88.76.228
- s3 -> 10.88.76.229



## 设置Host

在 s1、s2、s3中的/etc/hosts 中添加以下内容（需替换为自己的IP）

```bash
10.88.76.227 s1 
10.88.76.228 s2 
10.88.76.229 s3 

10.88.76.227 s1r1 
10.88.76.228 s1r2

10.88.76.228 s2r1
10.88.76.229 s2r2

10.88.76.229 s3r1
10.88.76.227 s3r2

10.88.76.227 zoo1
10.88.76.228 zoo2
10.88.76.229 zoo3
```



## 部署Zookeeper

> 此处示例为3.6.1版本

```bash
wget https://mirrors.tuna.tsinghua.edu.cn/apache/zookeeper/zookeeper-3.6.1/apache-zookeeper-3.6.1-bin.tar.gz
tar -zxf apache-zookeeper-3.6.1-bin.tar.gz
cd apache-zookeeper-3.6.1-bin
cp conf/zoo_sample.cfg conf/zoo.cfg
vim conf/zoo.cfg # 内容见下方
bin/zkServer.sh start
```

conf/zoo.cfg 内容为：

```properties
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/var/lib/zookeeper
clientPort=2181
server.1=s1:2888:3888
server.2=s2:2888:3888
server.3=s3:2888:3888
```

在s1、s2和s3服务器中分别执行命令验证安装结果：

```bash
bin/zkServer.sh status
```



## 安装ClickHouse

```bash
sudo apt-get install apt-transport-https ca-certificates dirmngr
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv E0C56BD4
echo "deb https://repo.clickhouse.tech/deb/stable/ main/" | sudo tee \
    /etc/apt/sources.list.d/clickhouse.list
sudo apt-get update
sudo apt-get install -y clickhouse-server clickhouse-client
```



## ClickHouse工作目录

```bash
# s1
├── create_table.sh		#（测试用）创建表
├── data_p.csv				#（测试用）示例数据
├── delete_table.sh		#（测试用）删除表
├── insert_data.sh		#（测试用）导入示例数据
├── query_table.sh		#（测试用）查询导入的数据量
├── s1r1
│   ├── config.xml		# 配置文件
│   ├── metrika.xml		# 分片配置文件
│   └── var						# 数据库启动后的相关文件
├── s1r1.log					# 日志
├── s3r2
│   ├── config.xml
│   ├── metrika.xml
│   └── var
├── s3r2.log
└── users.xml

# s2
├── s1r2
│   ├── config.xml
│   ├── metrika.xml
│   └── var
├── s1r2.log
├── s2r1
│   ├── config.xml
│   ├── metrika.xml
│   └── var
├── s2r1.log
└── users.xml

# s3
├── s2r2
│   ├── config.xml
│   ├── metrika.xml
│   └── var
├── s2r2.log
├── s3r1
│   ├── config.xml
│   ├── metrika.xml
│   └── var
├── s3r1.log
└── users.xml
```

配置中tcp访问端口如下：

- s1r1:9111
- s3r2:9132
- s2r1:9121
- s1r2:9112
- s3r1:9131
- s2r2:9122



## 运行ClickHouse

```bash
# s1
nohup clickhouse-server --config-file=/root/iot/chdb/s1r1/config.xml > s1r1.log 2>&1 &
nohup clickhouse-server --config-file=/root/iot/chdb/s3r2/config.xml > s3r2.log 2>&1 & 

# s2
nohup clickhouse-server --config-file=/root/iot/chdb/s1r2/config.xml > s1r2.log 2>&1 &
nohup clickhouse-server --config-file=/root/iot/chdb/s2r1/config.xml > s2r1.log 2>&1 &

# s3
nohup clickhouse-server --config-file=/root/iot/chdb/s2r2/config.xml > s2r2.log 2>&1 &
nohup clickhouse-server --config-file=/root/iot/chdb/s3r1/config.xml > s3r1.log 2>&1 &
```

> 停止运行： ps -A | grep click | awk '{print $1}' | xargs kill 



## 自定义用户名和密码

在users.xml中有详细的注释说明了如何添加用户、密码、用户权限等配置方法。（配置文件中已添加了以下内容，需要其他账户可参考以下步骤）

添加defualt用户密码和新建一个tom用户，由于密码存储在文件，因此推荐使用sha256编码后放入文件：

```bash
echo -n "default" | sha256sum | tr -d '-'···
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
clickhouse-client --host s1r1 -u default --password default --port 9111
clickhouse-client --host s1r1 --user tom --password tom-password --port 9111
```



## 测试（在s1上执行操作）

执行`create_table.sh`来创建表，其内容如下：

```bash
#!/bin/bash

hosts=("s1r1" "s3r2" "s2r1" "s1r2" "s3r1" "s2r2")
ports=("9111" "9132" "9121" "9112" "9131" "9122")

for idx in {0..5}
do
    host=${hosts[${idx}]}
    port=${ports[${idx}]}
    echo "Creating table on $host:$port"
    clickhouse-client --user default --password default --host $host --port $port --query \
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
```

执行`insert_data.sh`来创建分布表并插入数据，其内容如下：

```bash
#!/bin/bash

clickhouse-client --host s1r1 -u default --password default --port 9111 --query "CREATE TABLE p_all AS p ENGINE = Distributed(p_3shards_2replicas, default, p, rand())"
clickhouse-client --host s1r1 -u default --password default --port 9111 --query "INSERT INTO p_all FORMAT CSV" < data_p.csv
```

执行`query_table.sh`来查询各个分片和备份的数据量，其内容如下：

```bash
#!/bin/bash

hosts=("s1r1" "s3r2" "s2r1" "s1r2" "s3r1" "s2r2")
ports=("9111" "9132" "9121" "9112" "9131" "9122")

for idx in {0..5}
do
    host=${hosts[${idx}]}
    port=${ports[${idx}]}
    echo "Data size from $host:$port"
    clickhouse-client --user default --password default --host $host --port $port --query \
    "select count(*) from p"
done
```
