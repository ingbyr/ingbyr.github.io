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
├── fakeMqttKeyNoPass.pem
├── hubCrt.pem
└── hubKey.pem
```



## 配置Mosquitto

### 启用双向验证

原始配置文件注释比较多，因此可以新建一份配置文件用于配置

```bash
# 查看可配置的选项
cat /usr/local/etc/mosquitto/mosquitto.conf
```

新建 hub.conf 配置文件，其中3个file文件选项指向自己创建的密钥文件：

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





### Go测试

Golang中的私钥读取无法传入密码选项，因此可以不添加密码或使用以下命令附加密码至pem文件

```bash
openssl rsa -in fakeMqttKey.pem -out fakeMqttKeyNoPass.pem -passin pass:your_password
```



```go
package main

import (
	"crypto/tls"
	"crypto/x509"
	"fmt"
	"io/ioutil"
	"time"

	mqtt "github.com/eclipse/paho.mqtt.golang"
)

func NewTLSConfig() *tls.Config {
	// Import trusted certificates from CAfile.pem.
	// Alternatively, manually add CA certificates to
	// default openssl CA bundle.
	certPool := x509.NewCertPool()
	pemCerts, err := ioutil.ReadFile("certs/caCrt.pem")
	if err == nil {
		certPool.AppendCertsFromPEM(pemCerts)
	}

	// Import client certificate/key pair
	cert, err := tls.LoadX509KeyPair(
		"certs/fakeMqttCrt.pem",
		"certs/fakeMqttKeyNoPass.pem")
	if err != nil {
		panic(err)
	}

	// Just to print out the client certificate..
	cert.Leaf, err = x509.ParseCertificate(cert.Certificate[0])
	if err != nil {
		panic(err)
	}
	fmt.Println(cert.Leaf)

	// Create tls.Config with desired tls properties
	return &tls.Config{
		// RootCAs = certs used to verify server cert.
		RootCAs: certPool,
		// ClientAuth = whether to request cert from server.
		// Since the server is set up for SSL, this happens
		// anyways.
		ClientAuth: tls.NoClientCert,
		// ClientCAs = certs used to validate client cert.
		ClientCAs: nil,
		// InsecureSkipVerify = verify that cert contents
		// match server. IP matches what is in cert etc.
		InsecureSkipVerify: true,
		// Certificates = list of certs client sends to server.
		Certificates: []tls.Certificate{cert},
	}
}

var f mqtt.MessageHandler = func(client mqtt.Client, msg mqtt.Message) {
	fmt.Printf("TOPIC: %s\n", msg.Topic())
	fmt.Printf("MSG: %s\n", msg.Payload())
}

func main() {
	tlsconfig := NewTLSConfig()

	opts := mqtt.NewClientOptions()
	opts.AddBroker("ssl://localhost:1883")
	opts.SetClientID("ssl-sample").SetTLSConfig(tlsconfig)
	opts.SetDefaultPublishHandler(f)

	// Start the connection
	c := mqtt.NewClient(opts)
	if token := c.Connect(); token.Wait() && token.Error() != nil {
		panic(token.Error())
	}

	c.Subscribe("/go-mqtt/sample", 0, nil)

	i := 0
	for range time.Tick(time.Duration(1) * time.Second) {
		if i == 5 {
			break
		}
		text := fmt.Sprintf("this is msg #%d!", i)
		c.Publish("/go-mqtt/sample", 0, false, text)
		i++
	}

	c.Disconnect(250)
}
```



---

> 作者: ingbyr  
> URL: https://ingbyr.github.io/posts/mosquitto%E8%AF%81%E4%B9%A6%E5%8F%8C%E5%90%91%E9%AA%8C%E8%AF%81/  

