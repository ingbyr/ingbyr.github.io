# VDM中文说明


# 下载
- 需要提前安装[JRE8](http://www.oracle.com/technetwork/java/javase/downloads/jre8-downloads-2133155.html) （已安装JRE8的跳过此步骤）
- [下载页面](https://github.com/ingbyr/VDM/releases)下载`VDM.zip`，解压缩后运行`VDM.jar`


# 简介
VDM是适配多个视频命令行下载工具的GUI程序，已开源至 [GitHub](https://github.com/ingbyr/VDM)。

![](https://ws3.sinaimg.cn/large/bca3b20dly1fusoxry059j20xe0c0jrk.jpg)


# 已适配下载引擎
- [youtube-dl](https://github.com/rg3/youtube-dl)
- [annie](https://github.com/iawia002/annie)


# FAQ

## 自动更新引擎无法正常工作
如果软件内的自动更新总是失败的话（一般是因为访问github release过慢造成的），可以考虑手动更新引擎，方法如下:

1. 在下载引擎发布页面下载对应的可执行文件。
2. 将下载好的可执行文件放入`VDM\package\windows\engines`文件夹内，并将文件名替换为对应名称。
3. 修改`VDM\package\engines.json`中的对应版本号字段。
4. 重启VDM完成更新。

## 下载的视频没有声音
部分下载引擎需要安装[ffmpeg](https://ffmpeg.org/)才能实现视频和音频的自动合并，所以出现此问题时需要手动安装ffmpeg，并且在VDM中设置ffmpeg路径或添加系统PATH。当然也可以更换下载引擎再次尝试。


# 反馈BUG
如果软件运行出错或者无法获取到多媒体资源，请在[GitHub Issues](https://github.com/ingbyr/VDM/issues)进行反馈，我会尽快回复。


# 关于开发者
VDM 由 [@ingbyr](https://www.ingbyr.com/about/) 开发

