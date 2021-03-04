# CICD搭建部署


基于Gitea和Drone实现CICD

<!--more-->

## 搭建Gitea



### docker-compose部署

```yaml
#docker-compose.yml

version: "3"

networks:
  gitea:
    external: false

services:
  gitea:
    image: gitea/gitea:1.13.2
    environment:
      - USER_UID=1000
      - USER_GID=1000
      - DB_TYPE=postgres
      - DB_HOST=db:5432
      - DB_NAME=gitea
      - DB_USER=gitea
      - DB_PASSWD=gitea
    restart: always
    networks:
      - gitea
    volumes:
      - ./gitea:/data
      - /etc/timezone:/etc/timezone:ro
      - /etc/localtime:/etc/localtime:ro
    ports:
      - "10080:3000"
      - "10022:22"
    depends_on:
     - db

  db:
    image: postgres:9.6-alpine
    restart: always
    environment:
      - POSTGRES_USER=gitea
      - POSTGRES_PASSWORD=gitea
      - POSTGRES_DB=gitea
    networks:
      - gitea
    volumes:
      - ./postgres:/var/lib/postgresql/data
```



### 初始化Gitea

按Gitea的提示填入相应信息即可，需要注意的是

- `HTTP_PORT`的值为gitea默认的`3000`，不是Docker映射后的端口号
- `SSH_PORT`的值为Docker映射后的 `10022`

```ini
[server]
  APP_DATA_PATH    = /data/gitea
  DOMAIN           = ip
  SSH_DOMAIN       = ip
  HTTP_PORT        = 3000								
  ROOT_URL         = http://[gitea-ip]:10080/
  SSH_PORT         = 10022
  SSH_LISTEN_PORT  = 22 
```



### 创建应用

在`个人-设置-应用`中创建`OAuth2 应用`，并记录下`client id`和`client secret`，在后续Drone的部署中会用到



## 搭建Drone

### Docker Drone 和 Docker Drone Runner

使用docker-compose部署：

```yml
# docker-compose.yml

version: "3"

networks:
  drone:
    external: false

services:
  drone-server:
    image: drone/drone:1
    networks:
      - drone
    volumes:
      - ./drone-data:/var/lib/drone/data
    restart: always
    environment:
      - DRONE_GITEA_SERVER=${DRONE_GITEA_SERVER}
      - DRONE_GITEA_CLIENT_ID=${DRONE_GITEA_CLIENT_ID}
      - DRONE_GITEA_CLIENT_SECRET=${DRONE_GITEA_CLIENT_SECRET}
      - DRONE_RPC_SECRET=${DRONE_RPC_SECRET}
      - DRONE_SERVER_HOST=${DRONE_SERVER_HOST}
      - DRONE_SERVER_PROTO=${DRONE_SERVER_PROTO}
    ports:
      - "11080:80"
      - "11443:443"

  drone-runner:
    image: drone/drone-runner-docker:1
    restart: always
    depends_on:
      - drone-server
    networks:
      - drone
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      - DRONE_RPC_PROTO=${DRONE_SERVER_PROTO}
      - DRONE_RPC_HOST=${DRONE_SERVER_HOST}
      - DRONE_RPC_SECRET=${DRONE_RPC_SECRET}
      - DRONE_RUNNER_CAPACITY=2
      - DRONE_RUNNER_NAME=${HOSTNAME}
    ports:
      - "13000:3000"
```

环境变量的设置：

> `[rpc_secret]` 用于 Drone 和 Runner验证，具体值为随机字符串

```ini
# .env
DRONE_GITEA_SERVER=http://[gitea-ip]:10080/
DRONE_GITEA_CLIENT_ID=[gitea oauth2 app client id]
DRONE_GITEA_CLIENT_SECRET=[gitea oauth2 app client secret]
DRONE_RPC_SECRET=[rpc_secret]
DRONE_SERVER_HOST=[ip]:11080
DRONE_SERVER_PROTO=http
```



### Exec Runner

参考[官方文档](https://docs.drone.io/runner/exec/overview/)安装即可
