# Kubeedge 国内环境安装


国内网络环境下快速安装k8s

<!--more-->

## 安装k8s或k3s

参考官方网站的文档即可，k8s的配置部分参考[kubeedge官方文档](https://kubeedge.io/zh/blog/kubeedge-deployment-manual/)



## 配置安装环境

安装 kubeadm、kubectl：

```bash
gpg --keyserver keyserver.ubuntu.com --recv-keys BA07F4FB
gpg --export --armor BA07F4FB | sudo apt-key add -
echo "deb https://mirrors.tuna.tsinghua.edu.cn/kubernetes/apt kubernetes-xenial main" > /etc/apt/sources.list.d/kubernetes.list 
apt update
apt install kubeadm kubectl ipvsadm
```

下载Golang安装包并解压到 `/usr/local`，添加环境变量：

```bash
# Golang
export GOROOT=/usr/local/go
export GOPATH=/data/gopath
export PATH=$PATH:$GOROOT/bin:$GOPATH/bin

# Kubeedge
export PATH=$PATH:/data/gopath/src/github.com/kubeedge/kubeedge/_output/local/bin
```

source 环境变量文件使之生效。



## 编译Kubeedge

克隆源码（gitclone.com 用于国内加速）：

```bash
git clone https://gitclone.com/github.com/kubeedge/kubeedge $GOPATH/src/github.com/kubeedge/kubeedge 
```

由于kubeedge安编译中需要使用git的历史信息，因此将remote改回github地址：

```bash
git remote set-url origin https://github.com/kubeedge/kubeedge
git remote show origin
```

创建工作目录：

```bash
mkdir -p /data/gopath && cd /data/gopath
mkdir -p src pkg bin
```

编译：

```bash
cd $GOPATH/src/github.com/kubeedge/kubeedge
make all WHAT=keadm
make all WHAT=cloudcore
make all WHAT=edgecore
```

成功编译后的文件位于 `./_output/local/bin`，该目录在之前的环境变量中已添加。



## 创建cloud节点

使用`keadm`进行快速部署，由于安装过程需要访问github相关资源，因此选择手动提前下载所需文件：

> g.ioiox.com 为github资源镜像加速，若失效则需自己替换可用镜像
>
> 版本号根据当前kubeedge版本自行替换

```bash
mkdir /etc/kubeedge
cd  /etc/kubeedge && wget https://g.ioiox.com/https://github.com/kubeedge/kubeedge/releases/download/v1.3.1/kubeedge-v1.3.1-linux-amd64.tar.gz
```

同时可能需要添加 raw.githubusercontent.com 的DNS解析：

```bash
151.101.108.133 raw.githubusercontent.com
```

创建cloud节点：

> IP替换为本机IP

```bash
keadm init --advertise-address="IP"
```

查看日志：

```bash
tail -f /var/log/kubeedge/cloudcore.log
```



## 创建edge节点

在cloud端获取token：

```bash
keadm gettoken
```

将cloud端的`./_output/local/bin`二进制文件拷贝至edge端并加入PATH，随后创建edge节点：

> CLOUD_IP 为前文中的cloud端暴露的IP
>
> TOKEN为获取到的token

```bash
keadm join --cloudcore-ipport=CLOUD_IP:10000 --token=TOKEN
```

查看日志：

```bash
tail -f /var/log/kubeedge/edgecore.log
```



## 删除/重置Kubeedge

停止当前运行的kubeedge组件：

```bash
keadm reset
```

在cloud端：

```bash
rm -r /etc/kubeedge
kubectl delete CustomResourceDefinition $(k get CustomResourceDefinition | grep kubeedge | awk '{print $1}') 
```

> 删除 /etc/kubeedge 目录后需要再次手动下载 kubeedge-v1.3.1-linux-amd64.tar.gz



## 验证

在cloud端：

```bash
root@mq-228 /e/kubeedge# kubectl get no                 
NAME     STATUS   ROLES        AGE   VERSION
mq-228   Ready    master       11d   v1.18.6+k3s1
mq-227   Ready    agent,edge   38m   v1.17.1-kubeedge-v1.3.1
```

**Counter Demo 未更新，按教程无法直接运行**:

- apiVersion需要更新到v1alpha2 
- code.jquery.com无法在国内访问

---

> 作者: ingbyr  
> URL: https://ingbyr.github.io/posts/kubeedge-%E5%9B%BD%E5%86%85%E7%8E%AF%E5%A2%83%E5%AE%89%E8%A3%85/  

