---
title: Android02-开发环境
author: MadDonkey
layout: post
categories: [note, Android]
toc: true
---
这是安卓个人学习笔记第二篇。力求每一篇讲清楚，讲明白，讲的不冗长。以Java为主要编程语言。

第二篇介绍Android开发的开发环境相关。内容来自 Mark L. Murphy先生的The Busy Coder’s Guide To Android Development (Final Version)。感谢他的无私分享！

# 使用Android Studio

Android Studio是基于IntelliJ IDEA的IDE（一脉相承的JetBrains风格），现在更是Google官方钦定的开发者工具。界面美观舒适，专为Android而生。

当然也可以用IDEA本身。在Android 开发范畴之外，IDEA远远超过了Android Studio，不过既然是Android的教程，还是老老实实用Android Studio吧！



### 关于模拟器

由于ARM架构CPU类型的模拟器太慢，推荐使用运行在x86的CPU上的模拟器。为此，需要先在开发者的机器上安装一些东西——Linux用户需要安装 KVM，而Mac和Windows用户需要"Intel Hardware Accelerated Execution Manager”(又名HAXM)。而且模拟器这在支持虚拟化技术的硬件上才能运行。

另外，IDE的编译、构建等工作、IDE本身，以及模拟器会非常的占用内存。最好确保内存大小大于8GB，再有一块SSD就再好不过了。否则，使用IDE开发安卓的过程将是一场噩梦！



## 安装Java

AndroidStudio自2.2版本起会自带一份Open JDK 8的复制。最好还是在自己的机器上安自己的JDK，安装JDK的过程就不多赘述。



### 安装Android Studio

可从<a href="https://developer.android.com/studio/index.html">官方下载地址</a>处获取最新IDE。本文以大版本号3的IDE为例。Windows用户下载后得到一个.exe的可执行自我安装文件。Mac用户可以下载到一个DMG磁盘映像，以类似于其他mac software的方式安装之。Linux用户可下载ZIP文件并解压到硬盘目录上，然后可以从安装文件夹下的bin文件夹下的批处理文件或命令行来启动IDE。

第一次运行Android Studio时，会询问是否导入已有的配置文件。第一次可选否。再然后顺着Setup Wizard的提示，选择UI、选择安装模式等，即可完成安装。安装过程中可能会下载Android SDK等内容。

安装好之后打开IDE，可以在settings中找到配置AndroidSDK的地方。

