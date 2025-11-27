# Redis主从与集群模式


Redis集群相关

<!--more-->

## 主从模式

主节点Redis配置 `redis-6090.conf`：

```conf
port 6090
protected-mode no
```

从节点Redis配置 `redis-6091.conf`：

```conf
port 6091
protected-mode no
slaveof 127.0.0.1 6090 
```

从节点Redis配置 `redis-6092.conf`：

```conf
port 6092
protected-mode no
slaveof 127.0.0.1 6090 
```

Redis Sentinel配置模版 `redis-sentinel-cluster.tmpl`：

```conf
port ${PORT}
sentinel monitor mymaster 127.0.0.1 6090 2
sentinel down-after-milliseconds mymaster 5000
sentinel failover-timeout mymaster 60000
```

Redis Sentinel 实例的配置文件 `redis-sentinel-cluster-config.sh`，生成 5000-5002 3个实例：

```sh
#!/bin/bash

for port in `seq 5000 5002`; do \
  mkdir -p ./redis-sentinel-cluster/${port}/conf
  PORT=${port} envsubst < ./redis-sentinel-cluster.tmpl > ./redis-sentinel-cluster/${port}/conf/sentinel.conf
  chmod 777 ./redis-sentinel-cluster/${port}/conf/sentinel.conf
  mkdir -p ./redis-sentinel-cluster/${port}/data
done
```

使用 Docker Compose 部署：

```yaml
version: "3"

services:
  redis7000:
    image: redis:alpine
    container_name: redis6090
    command: "redis-server /usr/local/etc/redis/redis.conf"
    network_mode: host
    volumes:
      - ./redis-6090/conf:/usr/local/etc/redis
      - ./redis-6090/data:/data
  redis7001:
    image: redis:alpine
    container_name: redis6091
    command: "redis-server /usr/local/etc/redis/redis.conf"
    network_mode: host
    volumes:
      - ./redis-6091/conf:/usr/local/etc/redis
      - ./redis-6091/data:/data
  redis7002:
    image: redis:alpine
    container_name: redis6092
    command: "redis-server /usr/local/etc/redis/redis.conf"
    network_mode: host
    volumes:
      - ./redis-6092/conf:/usr/local/etc/redis
      - ./redis-6092/data:/data

  sentinel5000:
    image: redis:alpine
    container_name: sentinel5000
    command: "redis-server /usr/local/etc/redis/sentinel.conf --sentinel"
    network_mode: host
    volumes:
      - ./redis-sentinel-cluster/5000/conf:/usr/local/etc/redis
      - ./redis-sentinel-cluster/5000/data:/data
  sentinel5001:
    image: redis:alpine
    container_name: sentinel5001
    command: "redis-server /usr/local/etc/redis/sentinel.conf --sentinel"
    network_mode: host
    volumes:
      - ./redis-sentinel-cluster/5001/conf:/usr/local/etc/redis
      - ./redis-sentinel-cluster/5001/data:/data
  sentinel5002:
    image: redis:alpine
    container_name: sentinel5002
    command: "redis-server /usr/local/etc/redis/sentinel.conf --sentinel"
    network_mode: host
    volumes:
      - ./redis-sentinel-cluster/5002/conf:/usr/local/etc/redis
      - ./redis-sentinel-cluster/5002/data:/data

```

## Cluster 模式

> [参考资料](https://redis.io/learn/operate/redis-at-scale/scalability/exercise-1)

### Redis集群配置文件

Redis配置模版 `redis-cluster.tmpl`：

```conf
port ${PORT}
protected-mode no
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
appendonly yes
```

各个Redis实例配置生成脚本 `redis-cluster-config.sh`，脚本生成端口号 7000-7005 6个Redis实例：

```sh
#!/bin/bash

for port in `seq 7000 7005`; do
  mkdir -p ./redis-cluster/${port}/conf
  PORT=${port} envsubst < ./redis-cluster.tmpl > ./redis-cluster/${port}/conf/redis.conf
  chmod 666 ./redis-cluster/${port}/conf/redis.conf
  mkdir -p ./redis-cluster/${port}/data
done
```

### Redis集群启动

使用 Docker Compose 部署：

```yaml
version: "3"

services:
  redis7000:
    image: redis:alpine
    container_name: redis7000
    command: "redis-server /usr/local/etc/redis/redis.conf"
    network_mode: host
    volumes:
      - ./redis-cluster/7000/conf:/usr/local/etc/redis
      - ./redis-cluster/7000/data:/data
  redis7001:
    image: redis:alpine
    container_name: redis7001
    command: "redis-server /usr/local/etc/redis/redis.conf"
    network_mode: host
    volumes:
      - ./redis-cluster/7001/conf:/usr/local/etc/redis
      - ./redis-cluster/7001/data:/data
  redis7002:
    image: redis:alpine
    container_name: redis7002
    command: "redis-server /usr/local/etc/redis/redis.conf"
    network_mode: host
    volumes:
      - ./redis-cluster/7002/conf:/usr/local/etc/redis
      - ./redis-cluster/7002/data:/data
  redis7003:
    image: redis:alpine
    container_name: redis7003
    command: "redis-server /usr/local/etc/redis/redis.conf"
    network_mode: host
    volumes:
      - ./redis-cluster/7003/conf:/usr/local/etc/redis
      - ./redis-cluster/7003/data:/data
  redis7004:
    image: redis:alpine
    container_name: redis7004
    command: "redis-server /usr/local/etc/redis/redis.conf"
    network_mode: host
    volumes:
      - ./redis-cluster/7004/conf:/usr/local/etc/redis
      - ./redis-cluster/7004/data:/data
  redis7005:
    image: redis:alpine
    container_name: redis7005
    command: "redis-server /usr/local/etc/redis/redis.conf"
    network_mode: host
    volumes:
      - ./redis-cluster/7005/conf:/usr/local/etc/redis
      - ./redis-cluster/7005/data:/data

```

容器启动完毕后，连接至任意一个Redis示例，开始初始化集群：

```sh
redis-cli --cluster create 127.0.0.1:7000 127.0.0.1:7001 \
                           127.0.0.1:7002 127.0.0.1:7003 \
                           127.0.0.1:7004 127.0.0.1:7005 \
                           --cluster-replicas 1
```

执行完毕后将构建一个3主3从的Redis集群。

### 测试集群

> 对于Redis集群，命令行模式需要增加 -c 选项，否则无法操作Redis，即 `redis-cli -c -p 7000`

连接到端口号为7000的Redis，查看节点信息：

```sh
127.0.0.1:7000> cluster nodes

output> 2fc17db8bb8818ec8ac50f40f07bfbdf1ee8d56c 127.0.0.1:7002@17002 master - 0 1629082981000 3 connected 10923-16383
fc92c1709a88bdca8e86b9b5a27d035609e46059 127.0.0.1:7001@17001 master - 0 1629082982000 2 connected 5461-10922
3bd1b8cbe2e0e30a77dfa2b92dd70ed834bfcba4 127.0.0.1:7000@17000 myself,master - 0 1629082981000 1 connected 0-5460
cf361fc2b72ca2bfd070b282754ad19da7640485 127.0.0.1:7003@17003 slave 3bd1b8cbe2e0e30a77dfa2b92dd70ed834bfcba4 0 1629082982590 1 connected
fddc21542112cdcd8857bbfc77481b673e3a543e 127.0.0.1:7004@17004 slave fc92c1709a88bdca8e86b9b5a27d035609e46059 0 1629082981586 2 connected
2a5e09014c852fd15c2e54d500c532ec6ee2490e 127.0.0.1:7005@17005 slave 2fc17db8bb8818ec8ac50f40f07bfbdf1ee8d56c 0 1629082982088 3 connected
```

现在让7000的实例离线30秒：

```sh
redis-cli -p 7000 DEBUG sleep 30
```

现在看一下集群信息，可以看到主节点已经由7000自动切换到了7003，：

```sh
127.0.0.1:7000> cluster nodes
2fc17db8bb8818ec8ac50f40f07bfbdf1ee8d56c 127.0.0.1:7002@17002 master - 0 1629083362653 3 connected 10923-16383
fc92c1709a88bdca8e86b9b5a27d035609e46059 127.0.0.1:7001@17001 master - 0 1629083361000 2 connected 5461-10922
3bd1b8cbe2e0e30a77dfa2b92dd70ed834bfcba4 127.0.0.1:7000@17000 myself,slave cf361fc2b72ca2bfd070b282754ad19da7640485 0 1629083362000 7 connected
cf361fc2b72ca2bfd070b282754ad19da7640485 127.0.0.1:7003@17003 master - 0 1629083362000 7 connected 0-5460
fddc21542112cdcd8857bbfc77481b673e3a543e 127.0.0.1:7004@17004 slave fc92c1709a88bdca8e86b9b5a27d035609e46059 0 1629083362553 2 connected
2a5e09014c852fd15c2e54d500c532ec6ee2490e 127.0.0.1:7005@17005 slave 2fc17db8bb8818ec8ac50f40f07bfbdf1ee8d56c 0 1629083361647 3 connected
```

可以看到7000已经切换为了salve模式，7003切换成了master模式。

使用Go语言测试读写：

```go
package main

import (
	"context"
	"fmt"
	"github.com/go-redis/redis/v8"
)

var ctx = context.Background()

func redisAddrList() []string {
	ip := "localhost"
	addrList := []string{}
	for port := 7000; port < 7006; port++ {
		addrList = append(addrList, fmt.Sprintf("%s:%d", ip, port))
	}
	return addrList
}

func NewRedisClient() *redis.ClusterClient {
	return redis.NewClusterClient(&redis.ClusterOptions{
		Addrs: redisAddrList(),
	})
}

func main() {
	rdb := NewRedisClient()
	err := rdb.Set(ctx, "foo", "bar", 0).Err()
	if err != nil {
		panic(err)
	}

	res, err := rdb.Get(ctx, "foo").Result()
	if err != nil {
		panic(err)
	}

	fmt.Println(res)
}
```

### 扩充节点

当前节点和槽分配情况：

```shell
~/docker-app/redis-cluster » redis-cli -p 7000 cluster nodes     
9a353c214cb677b1e996573b4550875fb0754b65 127.0.0.1:7000@17000 myself,slave 0a50949e09aa0ebce0dbad1c8b0f301adbd62780 0 1736083233000 7 connected
0a50949e09aa0ebce0dbad1c8b0f301adbd62780 127.0.0.1:7003@17003 master - 0 1736083234560 7 connected 0-5460
3b8b46eac23b396e1d0b1adcfb70d086ee539da5 127.0.0.1:7004@17004 slave bd393aeacf7de92c8a3816cbfb1c497a310a418b 0 1736083235665 2 connected
bd393aeacf7de92c8a3816cbfb1c497a310a418b 127.0.0.1:7001@17001 master - 0 1736083235000 2 connected 5461-10922
3c21e101ea340698769de95812a98275693d3b2d 127.0.0.1:7002@17002 slave 57011e62426c833268c7f01b248b0e8f44af443a 0 1736083234660 8 connected
57011e62426c833268c7f01b248b0e8f44af443a 127.0.0.1:7005@17005 master - 0 1736083235563 8 connected 10923-16383
```

在工作目录下复制一个redis配置 `cp -r 7005 7006`，并将其端口号配置为7006，使用docker启动临时redis

```shell
docker run -d \
    --name redis7006 \
    --network host \
    -v ./redis-cluster/7006/conf:/usr/local/etc/redis \
    -v ./redis-cluster/7006/data:/data \
    redis:6.2 redis-server /usr/local/etc/redis/redis.conf
```

确认启动成功后，执行命令添加新的 redis node 到集群中去：

```shell
~/docker-app/redis-cluster » redis-cli --cluster add-node 127.0.0.1:7006 127.0.0.1:7000                                                                                               ing@ing-pc
>>> Adding node 127.0.0.1:7006 to cluster 127.0.0.1:7000
>>> Performing Cluster Check (using node 127.0.0.1:7000)
S: 9a353c214cb677b1e996573b4550875fb0754b65 127.0.0.1:7000
   slots: (0 slots) slave
   replicates 0a50949e09aa0ebce0dbad1c8b0f301adbd62780
M: 0a50949e09aa0ebce0dbad1c8b0f301adbd62780 127.0.0.1:7003
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
S: 3b8b46eac23b396e1d0b1adcfb70d086ee539da5 127.0.0.1:7004
   slots: (0 slots) slave
   replicates bd393aeacf7de92c8a3816cbfb1c497a310a418b
M: bd393aeacf7de92c8a3816cbfb1c497a310a418b 127.0.0.1:7001
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
S: 3c21e101ea340698769de95812a98275693d3b2d 127.0.0.1:7002
   slots: (0 slots) slave
   replicates 57011e62426c833268c7f01b248b0e8f44af443a
M: 57011e62426c833268c7f01b248b0e8f44af443a 127.0.0.1:7005
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
>>> Send CLUSTER MEET to node 127.0.0.1:7006 to make it join the cluster.
[OK] New node added correctly.
```

此时再次查看节点信息，可以看到7006已经加入到cluster中，：

```shell
~/docker-app/redis-cluster » redis-cli -p 7000 cluster nodes                                                                                                                          ing@ing-pc
9a353c214cb677b1e996573b4550875fb0754b65 127.0.0.1:7000@17000 myself,slave 0a50949e09aa0ebce0dbad1c8b0f301adbd62780 0 1736083995000 7 connected
0a50949e09aa0ebce0dbad1c8b0f301adbd62780 127.0.0.1:7003@17003 master - 0 1736083997000 7 connected 0-5460
3b8b46eac23b396e1d0b1adcfb70d086ee539da5 127.0.0.1:7004@17004 slave bd393aeacf7de92c8a3816cbfb1c497a310a418b 0 1736083995380 2 connected
 127.0.0.1:7006@17006 master - 0 1736083996384 0 connected
bd393aeacf7de92c8a3816cbfb1c497a310a418b 127.0.0.1:7001@17001 master - 0 1736083997390 2 connected 5461-10922
3c21e101ea340698769de95812a98275693d3b2d 127.0.0.1:7002@17002 slave 57011e62426c833268c7f01b248b0e8f44af443a 0 1736083996585 8 connected
57011e62426c833268c7f01b248b0e8f44af443a 127.0.0.1:7005@17005 master - 0 1736083996000 8 connected 10923-16383
```

如果需要给7006指定一个slave，则启动7007端口的redis后，执行命令以下命令：

```shell
$ redis-cli -p 7000 --cluster add-node 127.0.0.1:7007 127.0.0.1:7000 --cluster-slave --cluster-master-id d1c7e05fefad29888e1e4eb3a19574b94a280932
```

其中 --cluster-slave 指明新节点为slave，d1c7e05fefad29888e1e4eb3a19574b94a280932是7007节点id

新加入的节点是没有自动分配slot的，可以查看确认：

```shell
~/docker-app/redis-cluster » redis-cli  -p 7000 -c cluster slots                                                                                                                  1 ↵ ing@ing-pc
1) 1) (integer) 0
   2) (integer) 5460
   3) 1) "127.0.0.1"
      2) (integer) 7003
      3) "0a50949e09aa0ebce0dbad1c8b0f301adbd62780"
   4) 1) "127.0.0.1"
      2) (integer) 7000
      3) "9a353c214cb677b1e996573b4550875fb0754b65"
2) 1) (integer) 5461
   2) (integer) 10922
   3) 1) "127.0.0.1"
      2) (integer) 7001
      3) "bd393aeacf7de92c8a3816cbfb1c497a310a418b"
   4) 1) "127.0.0.1"
      2) (integer) 7004
      3) "3b8b46eac23b396e1d0b1adcfb70d086ee539da5"
3) 1) (integer) 10923
   2) (integer) 16383
   3) 1) "127.0.0.1"
      2) (integer) 7005
      3) "57011e62426c833268c7f01b248b0e8f44af443a"
   4) 1) "127.0.0.1"
      2) (integer) 7002
      3) "3c21e101ea340698769de95812a98275693d3b2d"
```

开始为7006节点分配slot

```shell
~/docker-app/redis-cluster » redis-cli -p 7000 --cluster reshard 127.0.0.1:7000                                                                                                       ing@ing-pc
>>> Performing Cluster Check (using node 127.0.0.1:7000)
S: 9a353c214cb677b1e996573b4550875fb0754b65 127.0.0.1:7000
   slots: (0 slots) slave
   replicates 0a50949e09aa0ebce0dbad1c8b0f301adbd62780
M: 0a50949e09aa0ebce0dbad1c8b0f301adbd62780 127.0.0.1:7003
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
S: 3b8b46eac23b396e1d0b1adcfb70d086ee539da5 127.0.0.1:7004
   slots: (0 slots) slave
   replicates bd393aeacf7de92c8a3816cbfb1c497a310a418b
M: d1c7e05fefad29888e1e4eb3a19574b94a280932 127.0.0.1:7006
   slots: (0 slots) master
M: bd393aeacf7de92c8a3816cbfb1c497a310a418b 127.0.0.1:7001
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
S: 3c21e101ea340698769de95812a98275693d3b2d 127.0.0.1:7002
   slots: (0 slots) slave
   replicates 57011e62426c833268c7f01b248b0e8f44af443a
M: 57011e62426c833268c7f01b248b0e8f44af443a 127.0.0.1:7005
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
How many slots do you want to move (from 1 to 16384)?
```

接下来会要求输入分配多少slot到哪个node，我们选择slot数量4096，分配到7006节点d1c7e05fefad29888e1e4eb3a19574b94a280932

```shell
[OK] All 16384 slots covered.
How many slots do you want to move (from 1 to 16384)? 4096
What is the receiving node ID? d1c7e05fefad29888e1e4eb3a19574b94a280932
Please enter all the source node IDs.
  Type 'all' to use all the nodes as source nodes for the hash slots.
  Type 'done' once you entered all the source nodes IDs.
Source node #1: all

Ready to move 4096 slots.
  Source nodes:
    M: 0a50949e09aa0ebce0dbad1c8b0f301adbd62780 127.0.0.1:7003
       slots:[0-5460] (5461 slots) master
       1 additional replica(s)
    M: bd393aeacf7de92c8a3816cbfb1c497a310a418b 127.0.0.1:7001
       slots:[5461-10922] (5462 slots) master
       1 additional replica(s)
    M: 57011e62426c833268c7f01b248b0e8f44af443a 127.0.0.1:7005
       slots:[10923-16383] (5461 slots) master
       1 additional replica(s)
  Destination node:
    M: d1c7e05fefad29888e1e4eb3a19574b94a280932 127.0.0.1:7006
       slots: (0 slots) master
  Resharding plan:
    Moving slot 5461 from bd393aeacf7de92c8a3816cbfb1c497a310a418b
    Moving slot 5462 from bd393aeacf7de92c8a3816cbfb1c497a310a418b
    ....
```

执行完毕再次确认slot分配情况

```shell
~/docker-app/redis-cluster » redis-cli -p 7000 -c cluster slots                                                                                                                       ing@ing-pc
1) 1) (integer) 0
   2) (integer) 1364
   3) 1) "127.0.0.1"
      2) (integer) 7006
      3) "d1c7e05fefad29888e1e4eb3a19574b94a280932"
2) 1) (integer) 1365
   2) (integer) 5460
   3) 1) "127.0.0.1"
      2) (integer) 7003
      3) "0a50949e09aa0ebce0dbad1c8b0f301adbd62780"
   4) 1) "127.0.0.1"
      2) (integer) 7000
      3) "9a353c214cb677b1e996573b4550875fb0754b65"
3) 1) (integer) 5461
   2) (integer) 6826
   3) 1) "127.0.0.1"
      2) (integer) 7006
      3) "d1c7e05fefad29888e1e4eb3a19574b94a280932"
4) 1) (integer) 6827
   2) (integer) 10922
   3) 1) "127.0.0.1"
      2) (integer) 7001
      3) "bd393aeacf7de92c8a3816cbfb1c497a310a418b"
   4) 1) "127.0.0.1"
      2) (integer) 7004
      3) "3b8b46eac23b396e1d0b1adcfb70d086ee539da5"
5) 1) (integer) 10923
   2) (integer) 12287
   3) 1) "127.0.0.1"
      2) (integer) 7006
      3) "d1c7e05fefad29888e1e4eb3a19574b94a280932"
6) 1) (integer) 12288
   2) (integer) 16383
   3) 1) "127.0.0.1"
      2) (integer) 7005
      3) "57011e62426c833268c7f01b248b0e8f44af443a"
   4) 1) "127.0.0.1"
      2) (integer) 7002
      3) "3c21e101ea340698769de95812a98275693d3b2d"
```

### 删除节点

停止7006容器 `docker stop 7006`，使用`CLUSTER FORGET`移除节点：

```shell
~/docker-app/redis-cluster » redis-cli -p 7000 -c CLUSTER FORGET d1c7e05fefad29888e1e4eb3a19574b94a280932                                                                             ing@ing-pc
OK
```

```shell
~/docker-app/redis-cluster » redis-cli -p 7000 -c CLUSTER NODES                                                                                                                       ing@ing-pc
9a353c214cb677b1e996573b4550875fb0754b65 127.0.0.1:7000@17000 myself,slave 0a50949e09aa0ebce0dbad1c8b0f301adbd62780 0 1736085038000 7 connected
0a50949e09aa0ebce0dbad1c8b0f301adbd62780 127.0.0.1:7003@17003 master - 0 1736085039571 7 connected 1365-5460
3b8b46eac23b396e1d0b1adcfb70d086ee539da5 127.0.0.1:7004@17004 slave bd393aeacf7de92c8a3816cbfb1c497a310a418b 0 1736085039673 2 connected
bd393aeacf7de92c8a3816cbfb1c497a310a418b 127.0.0.1:7001@17001 master - 0 1736085038667 2 connected 6827-10922
3c21e101ea340698769de95812a98275693d3b2d 127.0.0.1:7002@17002 slave 57011e62426c833268c7f01b248b0e8f44af443a 0 1736085039068 8 connected
57011e62426c833268c7f01b248b0e8f44af443a 127.0.0.1:7005@17005 master - 0 1736085039000 8 connected 12288-16383
```

```shell
~/docker-app/redis-cluster » redis-cli -p 7000 -c CLUSTER SLOTS                                                                                                                       ing@ing-pc
1) 1) (integer) 1365
   2) (integer) 5460
   3) 1) "127.0.0.1"
      2) (integer) 7003
      3) "0a50949e09aa0ebce0dbad1c8b0f301adbd62780"
   4) 1) "127.0.0.1"
      2) (integer) 7000
      3) "9a353c214cb677b1e996573b4550875fb0754b65"
2) 1) (integer) 6827
   2) (integer) 10922
   3) 1) "127.0.0.1"
      2) (integer) 7001
      3) "bd393aeacf7de92c8a3816cbfb1c497a310a418b"
   4) 1) "127.0.0.1"
      2) (integer) 7004
      3) "3b8b46eac23b396e1d0b1adcfb70d086ee539da5"
3) 1) (integer) 12288
   2) (integer) 16383
   3) 1) "127.0.0.1"
      2) (integer) 7005
      3) "57011e62426c833268c7f01b248b0e8f44af443a"
   4) 1) "127.0.0.1"
      2) (integer) 7002
      3) "3c21e101ea340698769de95812a98275693d3b2d"
```

发现此时slot缺失了一部分，查看cluster状态会发现集群此时处于 fail

```shell
~/docker-app/redis-cluster » redis-cli -p 7000 -c CLUSTER INFO                                                                                                                        ing@ing-pc
cluster_state:fail
cluster_slots_assigned:12288
cluster_slots_ok:12288
cluster_slots_pfail:0
cluster_slots_fail:0
cluster_known_nodes:6
cluster_size:3
cluster_current_epoch:9
cluster_my_epoch:7
cluster_stats_messages_ping_sent:7170
cluster_stats_messages_pong_sent:7271
cluster_stats_messages_update_sent:2
cluster_stats_messages_sent:14443
cluster_stats_messages_ping_received:7270
cluster_stats_messages_pong_received:11255
cluster_stats_messages_meet_received:1
cluster_stats_messages_fail_received:3
cluster_stats_messages_auth-req_received:1
cluster_stats_messages_received:18530
```

原因是7006在持有slot的状态下被移除了集群，此时我们需要再次reshard
> 当然你可以设置 cluster-require-full-coverage=no 让集群强制在缺失slot的情况下继续提供服务

执行命令修复集群

```shell
~/docker-app/redis-cluster » redis-cli --cluster fix 127.0.0.1:7000                                                                                                               1 ↵ ing@ing-pc
127.0.0.1:7003 (0a50949e...) -> 0 keys | 4096 slots | 1 slaves.
127.0.0.1:7001 (bd393aea...) -> 0 keys | 4096 slots | 1 slaves.
127.0.0.1:7005 (57011e62...) -> 1 keys | 4096 slots | 1 slaves.
[OK] 1 keys in 3 masters.
0.00 keys per slot on average.
>>> Performing Cluster Check (using node 127.0.0.1:7000)
S: 9a353c214cb677b1e996573b4550875fb0754b65 127.0.0.1:7000
   slots: (0 slots) slave
   replicates 0a50949e09aa0ebce0dbad1c8b0f301adbd62780
M: 0a50949e09aa0ebce0dbad1c8b0f301adbd62780 127.0.0.1:7003
   slots:[1365-5460] (4096 slots) master
   1 additional replica(s)
S: 3b8b46eac23b396e1d0b1adcfb70d086ee539da5 127.0.0.1:7004
   slots: (0 slots) slave
   replicates bd393aeacf7de92c8a3816cbfb1c497a310a418b
M: bd393aeacf7de92c8a3816cbfb1c497a310a418b 127.0.0.1:7001
   slots:[6827-10922] (4096 slots) master
   1 additional replica(s)
S: 3c21e101ea340698769de95812a98275693d3b2d 127.0.0.1:7002
   slots: (0 slots) slave
   replicates 57011e62426c833268c7f01b248b0e8f44af443a
M: 57011e62426c833268c7f01b248b0e8f44af443a 127.0.0.1:7005
   slots:[12288-16383] (4096 slots) master
   1 additional replica(s)
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[ERR] Not all 16384 slots are covered by nodes.

>>> Fixing slots coverage...
The following uncovered slots have no keys across the cluster:
[0-1364],[5461-6826],[10923-12287]
Fix these slots by covering with a random node? (type 'yes' to accept): yes
>>> Covering slot 5470 with 127.0.0.1:7005
>>> Covering slot 11756 with 127.0.0.1:7005
...
```

执行命令检查集群状态

```shell
~/docker-app/redis-cluster » redis-cli --cluster check 127.0.0.1:7000  
 ...
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```

```shell
~/docker-app/redis-cluster » redis-cli -c -p 7000 CLUSTER INFO                                                                                                                    1 ↵ ing@ing-pc
cluster_state:ok
cluster_slots_assigned:16384
cluster_slots_ok:16384
cluster_slots_pfail:0
cluster_slots_fail:0
cluster_known_nodes:6
cluster_size:3
cluster_current_epoch:14
cluster_my_epoch:13
cluster_stats_messages_ping_sent:8616
cluster_stats_messages_pong_sent:8700
cluster_stats_messages_update_sent:2
cluster_stats_messages_sent:17318
cluster_stats_messages_ping_received:8699
cluster_stats_messages_pong_received:12698
cluster_stats_messages_meet_received:1
cluster_stats_messages_fail_received:3
cluster_stats_messages_auth-req_received:1
cluster_stats_messages_update_received:2
cluster_stats_messages_received:21404
```

---

> 作者: ingbyr  
> URL: https://ingbyr.github.io/posts/redis%E4%B8%BB%E4%BB%8E%E4%B8%8E%E9%9B%86%E7%BE%A4%E6%A8%A1%E5%BC%8F/  

