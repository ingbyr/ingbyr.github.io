# Mosquitto证书双向验证


使用openssl生成自签名证书并配置Mosquitto双向验证

<!--more-->

## Openssl 自签CA证书

### 生成根证书和密钥

```bash
openssl req -new -x509 -newkey rsa:2048 \
-keyout caKey.pem -out caCrt.pem -days 365 \
-subj "/C=CN/ST=BJ/L=BJ/O=ZJ/OU=ROOT/CN=localhost/emailAddress=hello@world.com"
```

### 客户端生成密钥及证书签名请求

```bash
# mosquitto server 所需证书
openssl req -new -newkey rsa:2048 \
-keyout hubKey.pem -out hub.csr \
-subj "/C=CN/ST=BJ/L=BJ/O=ZJ/OU=HUB/CN=localhost/emailAddress=hello@world.com"

# 设备所需证书
openssl req -new -newkey rsa:2048 \
-keyout fakeMqttKey.pem -out fakeMqtt.csr \
-subj "/C=CN/ST=BJ/L=BJ/O=ZJ/OU=FakeMqtt/CN=localhost/emailAddress=hello@world.com"
```

### 使用 CA 根证书签发客户端证书

将两个csr文件上传至CA服务器后，由CA服务器执行签发

```bash
openssl x509 -req -days 365 \
-in hub.csr -out hubCrt.pem \
-CA caCrt.pem -CAkey caKey.pem \
-CAcreateserial
```

同理对 fakeMqtt.csr 进行签发

```bash
openssl x509 -req -days 365 \
-in fakeMqtt.csr -out fakeMqttCrt.pem \
-CA caCrt.pem -CAkey caKey.pem
```

### 验证生成的证书

```bash
openssl verify -CAfile caCrt.pem hubCrt.pem
openssl verify -CAfile caCrt.pem fakeMqttCrt.pem
```

> TODO：证书吊销 CRL

此时目录如下（测试时CA服务和客户端位于同一目录）：

```bash
> tree .
├── caCrt.pem
├── caCrt.srl
├── caKey.pem
├── fakeMqttCrt.pem
├── fakeMqttKey.pem
├── hubCrt.pem
└── hubKey.pem
```



## 配置Mosquitto

### 启用双向验证

原始配置文件注释比较多，因此我新建一份配置文件用于配置

```bash
# 查看可配置的选项
cat /usr/local/etc/mosquitto/mosquitto.conf
```

新建 hub.conf 配置文件，其中3个file文件选项指向上文中创建的文件：

```bash
port 1883
cafile certs/caCrt.pem
certfile certs/hubCrt.pem
keyfile certs/hubKey.pem
require_certificate true
use_identity_as_username true
```

### 启动

```bash
mosquitto -c ./hub.conf
```

### 订阅发布测试

```bash
# 订阅
mosquitto_sub -h localhost -p 1883 -t /t \
--cafile certs/caCrt.pem \
--cert certs/fakeMqttCrt.pem \
--key certs/fakeMqttKey.pem

# 发布
mosquitto_pub -h localhost -p 1883 -t /t -m "hello" \
--cafile certs/caCrt.pem \
--cert certs/fakeMqttCrt.pem \
--key certs/fakeMqttKey.pem
```


