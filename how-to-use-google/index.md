# 国内谷歌使用指南

目前个人所使用过的方法有三种

# 使用代理
vpn或者ss都是很好的选择，个人正在使用ss，主要是因为IPV6的ss不仅可以实现校园网免流，而且速度也比普通的ipv4快。ss的一些配置教程在网上很容易搜到。
## Windows配置SS
[下载的地址](https://github.com/shadowsocks/shadowsocks-windows/releases), 然后根据wiki进行配置即可
## Ubuntu配置SS
执行命令：
```bash
sudo apt install shadowsocks
```
然后实现一个config文件即可

# 使用第三方hosts
这种方式的优点是操作简单，缺点是有可能不太稳定，需要定期更新列表，下面是本人长期使用的一个源
> [ipv4 hosts](https://serve.netsh.org/pub/ipv4-hosts/)
> [ipv6 hosts](https://serve.netsh.org/pub/ipv6-hosts/)


## Windows修改hosts
修改 C:\Windows\System32\drivers\etc 下的hosts文件，将内容替换为第三方hosts。
## Ubuntu修改hosts
修改 /etc/hosts， 将内容替换为第三方hosts


# 使用国内谷歌镜像
配合chrome可以实现地址栏直接搜索谷歌内容，首先需要搜索一个谷歌镜像网站，比如[这个](http://google.adwiki.cn/)，此处以 https://g.inspire-energy.com.cn/ 这个网址为例
## 配置Chrome搜索
首先在镜像站中随便搜一个关键词，如搜索 ingbyr 得到如下结果 
![](https://ww2.sinaimg.cn/large/bca3b20djw1f7y4gocue7j20nd09vgmr.jpg)


发现其搜索关键词在 #q=ingbyr ，此时打开Chrome设置，找到如下所示的部分，点击管理搜索引擎
![](https://ww1.sinaimg.cn/large/bca3b20djw1f7y4gowbwej20ag0363yg.jpg)


前两项随便填，最后一项填入 https://g.inspire-energy.com.cn/#q=%s ，注意将ingbyr替换为%s，如图
![](https://ww2.sinaimg.cn/large/bca3b20djw1f7y4gpqvvaj20ld0j80uf.jpg)


然后选择设置为默认搜索引擎，如图
![](https://ww4.sinaimg.cn/large/bca3b20djw1f7y4gqsj1jj20lf0jimyq.jpg)


此时就可以直接在地址栏使用谷歌搜索啦~

