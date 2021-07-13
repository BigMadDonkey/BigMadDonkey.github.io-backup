---
title: Android01-核心概念
author: MadDonkey
layout: post
categories: [note, Android]
toc: true
---
这是安卓个人学习笔记第一篇。力求每一篇讲清楚，讲明白，讲的不冗长。以Java为主要编程语言。

第一篇介绍Android开发的许多核心概念。内容来自 Mark L. Murphy先生的The Busy Coder’s Guide To Android Development (Final Version)。感谢他的无私分享！

# 核心概念

## 组件

Android 应用的编写模式比起传统的 静态main方法作为程序入口，更接近于Servlet。不是写一个main方法，而是创建某些Android提供的基类的子类型，它们定义了各种各样的应用组件。此外，还要创建一些元数据（metadata）去告诉Android那些子类型的信息。

Android的核心是四大组件：

#### Activities

构建用户界面的组成部分是Activity。可以把Activity之于Android视为类似于窗口或对话框之于桌面应用，或者是页面之于经典Web应用。

Activity代表着用户界面的一部分，某些情况下，也代表着一个分离的进入应用程序的入口（一种让其他app链接到你的app的方式）。通常一个Activity会占据屏幕的大部分区域，留下空间给一些像是时钟、信号强度等等的东西。

然而必须记住，在某些设备上用户可以同时与多个Activity交互，例如分屏模式。必须记住，Activity终究不能代表整个屏幕。

#### Services

Activity是短期的，而且可以在任何时候被关闭，例如当用户按下back键时。而另一方面，Service是设计出来为了保持运行，而且独立于任何Activity（如果有必要的话）。即使控制Activity已经不在运行着，仍然可以让Service工作，例如播放后台音乐。

#### Content Providers

Content Provider提供了一层对于任何存储于设备之上的数据的抽象，可以被多个应用访问。Android开发模型鼓励开发者让自己的数据对其他应用可用，对自己也是。Content Provider正是为此而生，而且还能一定程度上控制自己的数据的可访问程度。

举个例子：如果有个app替用户下载好的PDF文件，想要允许用户查看文件，完全可以创建一个Content Provider让那个PDF对其他应用可访问。然后可以打开一个将可以查看PDF的Activity，由系统和用户决定具体使用那个PDF viewer。

#### Broadcast Receiver

系统和app有时会发出广播（Broadcast），从电池要没电啦，到要息屏啦，再到从WiFi切换到移动数据啦，都会发出广播。Broadcast Receiver的作用就是安排如何接收broadcast，并依据其进行响应。

#### Widgets, Containers and Resources

所谓的桌面小部件。绝大多数Android应用开发聚焦于UI层和活动。大多是Android activities使用被称为widget framework的东西去渲染他们的用户界面。也可以使用2D（canvas）和3D（OpenGL）的API来做更加定制化的GUI。

在Android术语中，一个widget是用户界面的一个微单元。Fields、按钮、标签、列表等等全是小部件。activity的UI，是由一个或多个这样的widget构成的。

要组织多个widget在屏幕上显示的央视，就要使用多种多样的容器类，也被称为layout manager。这些布局不安力气允许将东西排成行、列，或是更复杂的安排形式。

要描述如何让容器类和小部件之间联系起来，经常需要创建一个布局资源文件（layout resource file）。Android中的资源文件是指一些像图片、字符串和其他app使用但并不是以源代码的形式存储的材料。UI的布局也是一种资源。要创建这些布局文件，要么借助于结构化工具，例如IDE的拖动GUI builder，要么用手写XML。由于app的UI可能会跨各种设备运行，以及运行在各种不同的环境，很有必要把资源文件放进不同的资源集合resouce sets中，指明那些资源文件在何种情况下可以使用（例如 普通尺寸的屏幕用这个布局，大尺寸屏幕用那个布局...）



## App and Packages

应用程序以来源于APK文件。每个Android应用都会有一个包名，也叫做应用ID。一个包名必须满足3个要求：

1. 是一个合法的Java包名。因为Android build tools会在这个包中生成一些Java源代码。
2. 不允许同一个设备上同时存在两个设备具有相同的应用ID。
3. 应用商店中的任意两个应用都不能以相同的ID上传。

一般包名都以Java中常用的规则——反向域名来命名。

## Android Devices

三类主要的Android设备：

- 智能手机
- 平板电脑
- 笔记本电脑（大多数都是指运行在ChromeOS上的，这样的PC对Android app由一定的支持）

除了这些以外，还有一些不那么常见的设备：

- 电视
- 可穿戴设备
- 桌面设备

然而，Android可没有内置的关于具体设备的概念，什么手机、电脑、电视，都一视同仁。Android只是基于能力和特征来区分不同设备。因此在Android中不会有类似`isPhone()`的方法，倒是可以问问Android关于设备能力的信息。类似地，构建自己的应用时，更多地考虑设备的能力和特征，而非那些具体的类型是很有必要的。这也正是Android希望开发者做的。

## Emulator

即使没有Android设备，也可以借助于模拟器辅助开发。模拟器展现出的效果就像是一块安卓硬件，但实际上他只是运行在开发设备上的程序。借助于模拟器，可以模拟很多设备和很多屏幕尺寸以及很多安卓OS版本，这都是通过创建Android virtual devices，也叫**AVD**来实现的。下一节仔细讨论如何去创建这样的AVD，并运行模拟器。

## OS Versions and API Levels

Android从面世至今已发行大量版本，其能力越来越强，同时又始终努力保持向前/向后兼容。今天写下的app，在未来的版本应该也还能用，尽管可能错过一些新特性，或是在兼容模式下运行。

为了记录所有不同的OS版本以及它们对开发者的影响，Android引入了API Levels的概念。当一个Android的版本包含会影响开发者的改动时，一个新的API Level也得而定义。创建自己的AVD时，也需要选定使用何种API Level。发行app时，也需要指明应用支持的最老的版本，这样该app不会再更老的版本上安装。

一些经典的API Levels如下：

* API Level 19 (Android 4.4)
* API Level 22 (Android 5.1)
* API Level 23 (Android 6.0)
* API Level 24 (Android 7.0)
* API Level 25 (Android 7.1)
* API Level 26 (Android 8.0)
* API Level 28 (Android 9.0)

只需牢记 API Level 20是用在第一代Android可穿戴设备上的Android 4.4版本的。除非专门为可穿戴设备进行开发，否则没必要担心太多关于API Level 20的事。

#### Dalvik and ART

对Android而言，Dalvik和ART是虚拟机。虚拟机用在许多编程语言中，例如Java、Perl、Smalltalk等。Dalvik和ART设计出来更多是为了像JVM一样工作，但又要追求在Linux环境下的最优化。

此二者的区别在于，ART用于Android 5.0及更高版本，而Dalvik用于更旧的版本。

所以一个Android开发应用的过程大体如下：

- 开发者写符合Java语法的源代码，利用Android项目组和第三方发行的类库。
- 开发者把源代码编译成JVM的字节码，使用javac 编译器。
- 开发者把JVM字节码翻译成Dalvik VM字节码，与其他文件一同打包成ZIP格式的归档文件，带apk扩展名。
- Android设备/模拟器运行APK文件，把字节码放到DVM或ART上执行。

不过实际的开发过程中，大量复杂工作被略去，基本上只要编写Java代码，最后就得到APK文件了。

然而总有些地方有时存在些DVM和JVM之间的不同，会影响到的app开发者。

## 进程与线程

当应用运行时，它会在在自己的进程中运行，这一点与其他大部分传统OS无异。Android也会创建一些线程来运行app。大部分时间代码会被执行在一个叫做“主应用线程”或是“UI线程”的地方上。这个线程不用自己创建，开发者应该注意自己的代码对主线程的影响。要做些自己想做的事，最好从主线程中fork自己的线程出来。有些时候Android在屏幕下帮你自动做这样的事情。

