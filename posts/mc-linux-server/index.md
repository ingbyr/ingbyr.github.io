# Minecraft服务器搭建


## Spigot服务器（类Bukkit）

### 介绍
[spigotmc](https://www.spigotmc.org/)是一个以高性能著称的MC服务器端，Spigot服务端只支持plugin形式的扩展，如果使用人数规模庞大需要追求高性能服务器，Spigot是不二的选择。


### 安装
在官网下载BuildTools.jar文件后，需要自行编译出所需版本的服务器端文件，格式如下：
```bash
java -jar BuildTools.jar --rev [版本号]
```
编译需要一段时间，完成编译后将生成的jar文件拷贝至单独的文件夹。

### 插件配置
将插件（支持Spigot插件和Bukkit插件）拖入plugins文件夹中即可。


### 运行服务器
在jar文件目录下，编写一小段脚本以简化启动命令，内容如下：
```bash
java -Xms1024M -Xmx1024M -jar [your-jar-file] nogui
```
启动一个Screen终端，然后输入
```bash
sh start.sh
```
等待服务器启动完毕即可，要使服务器后台运行，按下ctrl+a,d后screen将处于后台。恢复screen输入：
```bash
screen -r mc
```
终止服务器直接 ctrl+c 即可


## Forge服务器

### 介绍
如果服务端需要运行一些mod（例如著名的暮色森林mod），此时spigot就不再适用了，而[Forge](https://files.minecraftforge.net/)正是为mod而生的服务端和客户端工具。


### 安装
在[Forge下载页面](https://files.minecraftforge.net/)选择需要的forge版本，生成服务端所需的文件要在本地新建一个空文件夹，生成所需文件后整体上传到服务器。


### mod配置
将mod直接拖入`mod`文件夹即可。


### 运行服务器
运行forge*.jar文件即可创建服务器。具体可参考Spigot中的 `运行服务器`一节


## Sponge

### 介绍
[Sponge](https://www.spongepowered.org/)据说是同时支持Bukkit插件和MOD功能的服务器端工具，具体没用使用过，大家可以去官网自行阅读文档安装体验。


## MC客户端
推荐使用[HMCL](https://github.com/huanghongxun/HMCL)。


### 安装optifine
虽然HMCL中有自动安装optifine的功能，但经常会出现依赖配置错误导致的运行失败，所以optifine和forge推荐自行安装。选择对应版本下载[optifine](https://www.optifine.net/downloads)，之后将jar文件导入mod或使用HMCL提供的MOD管理导入即可。


### 安装forge
在[Forge下载页面](https://files.minecraftforge.net/)选择需要的forge版本，安装客户端forge时需要选中游戏所在的`.minecraft`文件夹（不是公共资源文件夹），然后点击安装即可。

---

> 作者: ingbyr  
> URL: https://ingbyr.github.io/posts/mc-linux-server/  

