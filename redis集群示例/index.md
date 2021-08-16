# Redis集群示例


Redis集群相关

<!--more-->

## Redis集群配置文件

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

Redis Sentinel配置模版 `redis-sentinel-cluster.tmpl`：
```conf
port ${PORT}
sentinel monitor mymaster 127.0.0.1 7000 2
sentinel down-after-milliseconds mymaster 5000
sentinel failover-timeout mymaster 60000
```

Redis Sentinel 实例的配置文件 `redis-sentinel-cluster-config.sh`，生成 5000-5002 3个实例：
```sh
#!/bin/bash

for port in `seq 5000 5002`; do \
  mkdir -p ./redis-sentinel-cluster/${port}/conf
  PORT=${port} envsubst < ./redis-sentinel-cluster.tmpl > ./redis-sentinel-cluster/${port}/conf/sentinel.conf
  chmod 666 ./redis-sentinel-cluster/${port}/conf/sentinel.conf
  mkdir -p ./redis-sentinel-cluster/${port}/data
done
```

## Redis集群启动
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

容器启动完毕后，连接至任意一个Redis示例，开始初始化集群：

```sh
redis-cli --cluster create 127.0.0.1:7000 127.0.0.1:7001 \
                           127.0.0.1:7002 127.0.0.1:7003 \
                           127.0.0.1:7004 127.0.0.1:7005 \
                           --cluster-replicas 1
```

执行完毕后将构建一个3主3从的Redis集群。

> 对于Redis集群，命令行模式需要增加 -c 选项，否则无法操作Redis，即 `redis-cli -c -p 7001`


## 测试集群

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

连接至端口号为5000的哨兵，查看监控信息：
```sh
127.0.0.1:5000> SENTINEL get-master-addr-by-name mymaster
1) "127.0.0.1"
2) "7000"
```

现在让7000的实例离线30秒：
```sh
redis-cli -p 7000 DEBUG sleep 30
```

在哨兵5000中再次查看监控：
``` sh
127.0.0.1:5000> SENTINEL get-master-addr-by-name mymaster
1) "127.0.0.1"
2) "7003"
```

可以看到主节点已经由7000自动切换到了7003，现在看一下集群信息：
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

