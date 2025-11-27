# 使用ffmpeg压缩60FPS视频并上传至Bilibili


## 样例

我们要达到的视频效果是[这样的](https://www.bilibili.com/video/av17425062/)

## 确定压缩目标

bilibili不进行二压的参数要求如下：

>视频码率最高1800kbps（H264/AVC编码）
音频码率最高192kbps（AAC编码）
分辨率最大支持1920x1080
level≤4.1
关键帧平均至少10秒一个
色彩空间yuv420
位深8bit
声道数≤2
采样率=44100
（不满足条件的视频会被系统二次压制）

## FFMPEG压制

这里先给出全部命令：

```bash
ffmpeg -i $argv[1] -vcodec libx264 -preset veryslow -profile:v high -level:v 4.1 -pix_fmt yuv420p -b:v 1780k -r 60 -acodec aac -strict -2 -ac 2 -ab 128k -ar 44100 -pass 1 -f flv /dev/null; and ffmpeg -i $argv[1] -vcodec libx264 -preset veryslow -profile:v high -level:v 4.1 -pix_fmt yuv420p -b:v 1780k -r 60 -acodec aac -strict -2 -ac 2 -ab 128k -ar 44100 -pass 2 -f flv $argv[2]
```

这里总共执行了两条命令，下面详细说一下。

### -i $argv[1]

这里指定了目标视频路径，也就是要处理的视频文件

### -vcodec libx264

使用X264编码器

### -preset veryslow

使用h.264的最佳编码，牺牲了编码速度。因为b站1800的码率如果不采用最佳编码，会导致画面极度模糊。

### -profile:v high -level:v 4.1

设备兼容性，这里不需要修改

### -pix_fmt yuv420p

色彩空间yuv420p，b站要求

### -b:v 1780k

码率采用1780，如果采用上限1800实际结果将有个可能超过这个值，从而被二压

### -r 60

视频帧率为60FPS

### -pass 1

说明当前处理为第一次处理，为了达到稳定的视频目标参数我们需要进行两次压制，第二条命令就是第二次压制

### acodec aac -strict -2 -ac 2 -ab 128k -ar 44100

音频参数，说明使用acc解码器，双声道，128K码率，44.1k采样率，都是b站的上限数值

### -f flv

视频格式为flv

## 上传视频
经过本地的两次压制，上传到b站后就不会被二压，从而保证了60fps的帧率。

---

> 作者: ingbyr  
> URL: https://ingbyr.github.io/posts/compress-video-by-ffmpeg/  

