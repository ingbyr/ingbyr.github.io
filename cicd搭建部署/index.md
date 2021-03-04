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

> `[rpc_secret]` 用于 Drone 和 Runner 验证，具体值为随机字符串

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



## Drone实战

项目中添加`.drone.yml`

```yml
kind: pipeline
type: exec # 指定 Runner 的模式，此处为 Exec Runner
name: default
platform: # Exec Runner 特有设置，提供 Runner 运行的相关信息
  os: linux
  arch: amd64
steps: # 以下为自定义的步骤流程
- name: build
  commands:
  - bash build.sh
- name: deploy
  commands:
  - cd /data/projects/iwop-ui
  - zip -rq dist.zip dist/
  - cp dist.zip /data/storage/iwop-ui/dist-$(date '+%Y%m%d%H%M%S').zip
```

`build.sh` 展示了构建一个外部 nodejs 项目的方式（另一种为个人认为可行的方案是 git submodule，尚未验证）

```sh
#!/bin/bash

# Create project dir and clone it
if [ ! -d [parent-dir]/[project-name] ]; then
  mkdir -p [parent-dir]
  cd [parent-dir]
  git clone -b dev --single-branch --depth=1 [git-url]
fi

# Go to project and update code
cd [parent-dir]/[project-name]
# If has custom we need to restore before update code
git restore .
git pull --rebase

# Custom
custom_files=(
	"file-1"
	"file-2"
)
for i in "${custom_files[@]}"; do
    gsed -i 's/before text/after text/' $i  # For linux add 'alias gsed=sed' / For mac 'brew install gsed'
done

# Check command status
if ! command -v cnpm &> /dev/null; then
    echo "Start to install cnpm ..."
    npm install -g cnpm --registry=https://registry.npm.taobao.org
fi

# Build
cnpm install
npm run build-ws --no-progress
```


