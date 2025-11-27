# JavaFX系列-开发环境

本文主要介绍JavaFX环境的搭建和如何部署JavaFX应用
<!--more-->

## 添加JavaFX依赖
在工程 build.gradle 中加入以下内容：
```bash
plugins {
    id 'application'
    id 'org.openjfx.javafxplugin' version '0.0.8' // 引入JavaFX的jar包
}

javafx {
    version = "14" // Java版本
    modules = [ 'javafx.controls' ] // 所需要的模块
    manifest {
        mainClassName = 'com.ingbyr.vdm.gui.Main' // 运行入口
    }
}
```


## 自定义运行时
> Java9引入的Jigsaw允许我们制作自己的JRE，从而有效地减少应用的大小

首先下载JavaFX的jmods保存到 `./jmods/`中
```bash
jmods
├── javafx.base.jmod
├── javafx.controls.jmod
├── javafx.fxml.jmod
├── javafx.graphics.jmod
├── javafx.media.jmod
├── javafx.swing.jmod
└── javafx.web.jmod
```

然后使用jlink生成自定义运行时
```bash
jlink \
--module-path ./jmods \
--add-modules java.base,javafx.base,javafx.graphics,javafx.controls \
--output runtime
```
- **module-path** 第三方模块的目录
- **add-modules** 运行时所需的模块
- **output** 自定义运行时目录名称 

生成的运行时目录如下
```bash
runtime
├── bin
├── conf
├── include
├── legal
├── lib
├── man
└── release
```


## 编译应用
使用gradle插件application中提供的`tasks -> distribution -> installDist`构建任务来编译应用，生成的目录如下
```bash
build/install
└── vdm-gui
    ├── bin
    │   ├── vdm-gui
    │   └── vdm-gui.bat
    └── lib
        ├── javafx-base-14-mac.jar
        ├── javafx-base-14.jar
        ├── javafx-controls-14-mac.jar
        ├── javafx-graphics-14-mac.jar
        ├── javafx-graphics-14.jar
        └── vdm-gui-2.0.0.jar
```

## 部署应用
Java14中正式提供了jpackage工具，该工具可以十分方便的部署跨平台应用安装包，示例如下
```bash
jpackage \
--input vdm-gui/build/install/vdm-gui/lib \
--name vdm14 \
--main-jar vdm-gui-2.0.0.jar \
--main-class com.ingbyr.vdm.gui.Main \
--type dmg \
--runtime-image ~/IdeaProjects/vdm14/runtime
```
- **input** 需要打包的jar目录
- **name** 应用名称
- **main-jar** 主程序jar包
- **main-class** 若jar中没有指定主类，则可以使用此参数指定
- **type** 安装包类型，可选：
  - Windows: exe msi
  - Linux: rpm deb
  - MacOS: pkg dmg
- **runtime-image** 自定义的运行时


---

> 作者: ingbyr  
> URL: https://ingbyr.github.io/posts/javafx%E7%B3%BB%E5%88%97-%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA/  

