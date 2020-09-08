# Unity3D/Android/Vuforia环境搭建



# 开发环境配置
## 安装包下载
点击下方文字进行安装，请自备梯子
- [Unity3D](https://unity3d.com/cn/get-unity)
- [Android SDK](https://developer.android.com/sdk/installing/index.html)
- [Vuforia SDK](https://developer.vuforia.com/downloads/sdk)

## U3D配置
配置编译安卓的环境，如下图所示
![](https://ww4.sinaimg.cn/large/bca3b20djw1f128ca2kfbj20e20bxt9x.jpg)
![](https://ww4.sinaimg.cn/large/bca3b20djw1f128caerdmj20f108tq3t.jpg)

# 第一个虚拟现实应用

> 参考 https://developer.vuforia.com/library/articles/Solution/Compiling-a-Simple-Unity-Project


## 创建一个新项目
U3D中新建项目，双击下载好的SDK：vuforia-unity-xx-yy-zz.unitypackage 将SDK导入到项目中，完成后如下图
![](https://ww1.sinaimg.cn/large/bca3b20djw1f128v76x00j20fl06lmxq.jpg)

## 导入应用Key
在Vuforia官网中生成一个key，复制key到项目中的ARCamera的属性key中，如图
![](https://ww3.sinaimg.cn/large/bca3b20djw1f128v7pxk6j20aj08jq3v.jpg)

## 导入素材
删除默认Camera，导入AR Camera,导入IamgeTarget。将识别目标图片上传至目标管理页面，下载data双击导入到项目中，将IamgeTarget属性Data Set设置为刚刚导入的data。然后随便新建一个3D素材，此处以立方体为例，将新建的Cube拖至IamgeTarget 的Children中，并调整至ImageTarget的中央，如图
![](https://ww1.sinaimg.cn/large/bca3b20djw1f128v874hfj211y0lcagl.jpg)
随后将AR Camera其中的输入设置如下
![](https://ww3.sinaimg.cn/large/bca3b20djw1f128v8rj7zj20aj02djrg.jpg)

## 应用发布
记得修改Bundle Identifier
![](https://ww4.sinaimg.cn/large/bca3b20djw1f128v9smo8j20q00hpjug.jpg)
然后点击build and run 即可

