# Baetyl V2安装指南


Baetyl v2.0.0安装指南

<!--more-->

## Docker

`/etc/docker/daemon.json`中添加/修改：

```json
{
    "exec-opts": ["native.cgroupdriver=systemd"],
    "registry-mirrors": ["https://docker.mirrors.ustc.edu.cn/"],
}
```

重启Docker



## Kubernets

### Rancher

> 参考[官方文档](https://docs.rancher.cn/docs/k3s/installation/_index)和[国内部署](https://docs.rancher.cn/docs/rancher2/best-practices/use-in-china/_index)

```bash
docker run -itd -p 9080:80 -p 9443:443 \
--name rancher \
--restart=unless-stopped \
-e CATTLE_AGENT_IMAGE="registry.cn-hangzhou.aliyuncs.com/rancher/rancher-agent:v2.4.2" \
registry.cn-hangzhou.aliyuncs.com/rancher/rancher:v2.4.2
```

清空数据
```bash
docker stop $(docker ps -aq)
docker system prune -f
docker volume rm $(docker volume ls -q)
docker image rm $(docker image ls -q)
rm -rf /etc/ceph \
       /etc/cni \
       /etc/kubernetes \
       /opt/cni \
       /opt/rke \
       /run/secrets/kubernetes.io \
       /run/calico \
       /run/flannel \
       /var/lib/calico \
       /var/lib/etcd \
       /var/lib/cni \
       /var/lib/kubelet \
       /var/lib/rancher/rke/log \
       /var/log/containers \
       /var/log/pods \
       /var/run/calico
```


###  k3s

使用国内源安装

```bash
curl -sfL http://rancher-mirror.cnrancher.com/k3s/k3s-install.sh | INSTALL_K3S_MIRROR=cn sh -s - server --docker
```

配置config

```bash
cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
```

添加本地存储支持：

```bash
wget https://raw.githubusercontent.com/rancher/local-path-provisioner/master/deploy/local-path-storage.yaml
kubectl create -f local-path-storage.yaml
```

设置该存储为默认存储：

```bash
kubectl patch storageclass local-path -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```



### K8S

```bash
# 安装 kubeadm kubectl kubelet
gpg --keyserver keyserver.ubuntu.com --recv-keys BA07F4FB
gpg --export --armor BA07F4FB | sudo apt-key add -
echo "deb https://mirrors.tuna.tsinghua.edu.cn/kubernetes/apt kubernetes-xenial main" > /etc/apt/sources.list.d/kubernetes.list 
apt update
apt install kubeadm kubectl kubelet

# 查看指定k8s版本需要哪些镜像
kubeadm config images list --kubernetes-version v1.18.3
```

终端输出：

```bash
k8s.gcr.io/kube-apiserver:1.18.3
k8s.gcr.io/kube-controller-manager:v1.18.3
k8s.gcr.io/kube-scheduler:v1.18.3
k8s.gcr.io/kube-proxy:v1.18.3
k8s.gcr.io/pause:3.2
k8s.gcr.io/etcd:3.4.3-0
k8s.gcr.io/coredns:1.6.7
```

新建脚本`get-k8s-images.sh `并替换版本号：

```bash
#!/bin/bash

images=(
    kube-apiserver:v1.18.3
    kube-controller-manager:v1.18.3
    kube-scheduler:v1.18.3
    kube-proxy:v1.18.3
    pause:3.2
    etcd:3.4.3-0
    coredns:1.6.7
)

for imageName in ${images[@]} ; do
    docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName
    docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName k8s.gcr.io/$imageName
    docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName
done
```

执行`get-k8s-images.sh `以便从国内hub获取镜像。修改kubelet配置中的默认cgroup driver：

```bash
cat > /var/lib/kubelet/config.yaml <<EOF
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
cgroupDriver: systemd
EOF
systemctl restart kubelet
```

启动k8s：

```bash
kubeadm init --pod-network-cidr=10.244.0.0/16 --kubernetes-version=v1.18.3
```

启动完毕后有后续步骤的相关提示，具体操作为配置`$HOME/.kube/config`：

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

添加网络组件（Flannel）：

```bash
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
kubectl apply -f kube-flannel.yml
```

添加本地存储支持：

```bash
wget https://raw.githubusercontent.com/rancher/local-path-provisioner/master/deploy/local-path-storage.yaml
kubectl create -f local-path-storage.yaml
```

设置该存储为默认存储：

```bash
kubectl patch storageclass local-path -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```



## 安装Baetyl

参考[官方文档](https://docs.baetyl.io/zh_CN/latest/develop/install.html)

在安装边缘节点时报错：

```bash
curl -d "{\"name\":\"demo-node\"}" -H "Content-Type: application/json" -X POST http://0.0.0.0:30004/v1/nodes
{"code":"UnknownError","message":"nodes.cloud.baetyl.io \"demo-node\" is forbidden: User \"system:serviceaccount:default:baetyl-cloud\" cannot get resource \"nodes\" in API group \"cloud.baetyl.io\" in the namespace \"baetyl-cloud\"","requestId":""}
```

临时的解决办法：为账户baetyl-cloud添加所有相关权限：

```yaml
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: my-baetyl-cr
  labels:
    custom: role-patch
rules:
  - apiGroups:
      - cloud.baetyl.io
    resources:
      - nodes
      - applications
      - configurations
      - nodedesires
      - nodereports
      - secrets
    verbs:
      - get
      - list
      - watch
      - create
      - update
      - patch
      - delete
```

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: my-baetyl-crb
  labels:
    custom: role-patch
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: my-baetyl-cr
subjects:
  - kind: ServiceAccount
    name: baetyl-cloud
    namespace: default
```

---

> 作者: ingbyr  
> URL: https://ingbyr.github.io/posts/baetyl-v2%E5%AE%89%E8%A3%85%E6%8C%87%E5%8D%97/  

