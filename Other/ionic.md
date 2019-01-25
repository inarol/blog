# Windows下搭建Ionic开发环境

## 目录

* [搭建工具和环境](#搭建工具和环境)
* [如何创建一个demo](#如何创建一个demo)

## 搭建工具和环境

**1**. 首先下载安装Ant（cordova工具生成android项目时，需要的编辑工具），到[apache官网](http://ant.apache.org/)下载最新版的zip包，解压到某目录下（例如我是解压缩到D盘，D:\apache-ant-1.9.4）。然后根据自己的目录配置环境变量Path：D:\apache-ant-1.9.4\bin;  以及CLASSPATH： D:\apache-ant-1.9.4\lib;
验证是否安装配置成功（可能需要重启），在命令提示符窗口输入 ant 命令，如果出现 Buildfile: build.xml does not exist!则表示配置成功！

**2**. 安装JAVA环境，网上很多教程，这里不多介绍。

**3**. 安装nodejs环境（安装项目各种依赖包，主要用到npm命令），到[nodejs官网](https://nodejs.org/)下载安装即可，安装非常简单，不多说。验证是否安装配置成功，在命令提示符窗口输入 node -v 如果出现版本号，则表示安装成功。

**4**. 安装Android SDK，最折腾的就是这个，需要翻墙才能把整个过程安装配置完，这里提供国内别人已经下载好的SDK包[百度网盘地址](http://pan.baidu.com/s/1miaWxDI)。把安装包解压到某目录下（例如我是解压缩到D盘，D:\android-sdk-windows），接着配置系统环境变量：

```
ANDROID_HOME=D:\android-sdk-windows
PATH=%ANDROID_HOME%\tools;%ANDROID_HOME%\platform-tools;
```

**5**. 安装Ionic，在命令提示符窗口输入命令：

```
npm install -g cordova ionic 
```

即可安装ionic和Cordova。

## 如何创建一个demo

**1.** 创建一个示例项目，输入

```
ionic start myApp tabs
```

**2.** 在浏览器调试示例项目，输入

```
ionic serve
```

**3.** 添加android平台，输入

```
ionic platform add android
```

**4.** 打包apk包，输入

```
ionic build android
```

**5.** 启动模拟器调试（自带的android模拟器可能会略卡，推荐使用Genymotion来替代Android模拟器作为模拟平台，启动速度增加不少），输入

```
ionic run android
```