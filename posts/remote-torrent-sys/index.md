# Linux Server使用IPV6 BT资源


## 啰嗦几句
权利的游戏终于又更新啦，然而目前网络没有IPV6所以怎么下载资源成了最大的问题，试了试公网上的bt，速度慢的令人发指，所以决定还是想办法通过IPV6下载资源

## 需要提前准备的资源
必须要有的：
- 一台能访问IPV6资源的服务器（本文使用一台Ubuntu Server 16.04）
- 一个BT账号（北邮人应该都有吧）
- 如果是校内服务器，还需要下载[GlobalProtect](https://vpn.bupt.edu.cn)，后续步骤默认开启VPN

以下资源是非必需的：
- 独立域名（方便自己记住域名）

## 步骤
### 安装Deluge
Deluge后端使用libtorrent，可以在多个平台上使用
```bash
sudo apt install deluge
```
### 安装Deluge-web
Deluge-web可以将Deluge在浏览器中以可视化的方式展现出来，应该没人希望通过复杂的命令行方式操作吧
```bash
sudo apt install deluge-webui
```
### 运行Deluge
启动Deluge后还需要手动启动Deluge-webui，这里使用screen启动webui
```bash
sudo systemctl start deluged
screen -S deluge
deluge-web
```
这时在浏览器中访问 http://your-server-ip:8112 就可以看到熟悉的bt下载界面了，默认的密码是deluge

### 懒人必备（非必须步骤）
一串数字的网址毕竟不太好记忆，况且还有一个8112的端口号，所以接下来使用dnspod和nginx实现域名访问
#### 安装nginx
```bash
sudo apt install nginx
```
#### 创建配置文件
在 /etc/nginx/conf.d/ 目录下新建一个配置文件，名称随意后缀为.conf，针对8112端口的配置实例如下：
```bash
server {
    listen 80;
    server_name [你的域名];
    location / {
        proxy_redirect off;
        proxy_pass http://127.0.0.1:8112;
    }
}
```
如果要配置其他端口的照猫画虎再填一个就行了，因为在 /etc/nginx/nginx.conf 默认读入了conf.d文件夹下的配置文件，所以不再需要动它的默认配置文件
#### 重启nginx
```bash
sudo systemctl restart nginx.service
```
#### 绑定域名
在DNS服务商添加一条A记录，域名为上文中的域名，ip指向服务器IP即可，等DNS生效就可以通过二级域名访问Deluge了

## 资源取回
可以选择把资源从服务器上下下来本地观看，也可以使用Potplayer等支持远程视频播放的软件直接观看。

---

> 作者: ingbyr  
> URL: https://ingbyr.github.io/posts/remote-torrent-sys/  

