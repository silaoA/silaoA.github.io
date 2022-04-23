---
title: Linux Cygwin知识库：图形技术栈
date: 2022-04-23 17:44:55
categories: 技术
tags: [Cygwin, Linux, Windows]
toc: true
comments: false
# layout: false
---
本文共约3000字，预计阅读时间10分钟，本文同步发布于[知乎专栏](https://zhuanlan.zhihu.com/p/503627248)和[微信公众号](/about/index.html)平台。
关注学习了解更多的Cygwin、Linux、Python技术。
应评论区要求，本文对[Cygwin系列（十二）：了解X](/2020/2020-04-25-Cygwin系列（十二）：了解X.html)做更深入介绍。

<!--more-->
<!-- [toc] -->

# 0x00 前言
在[Cygwin系列（十二）：了解X](/2020/2020-04-25-Cygwin系列（十二）：了解X.html)简要介绍了X Window System，并发布到回答[Linux 图形界面的显示原理是什么？](https://www.zhihu.com/question/321725817/answer/2251568546)。简单总结下来就是：**X客户端及窗口管理器知道绘制什么内容，但是没有渲染能力，凡事都要交给X Server；到了Wayland这里，Server和窗口管理器功能合并为Compositor，Wayland客户端也能够直接渲染，Wayland体系中大大减少了进程间的沟通**。
评论区反馈，讲得还不够透，在看了一些资料之后，我力求以“**不求甚解**”的方式把到底层的调用链条说一下，如有错误请轻喷。

# 0x01 几个名词
绘制（draw）、 渲染（render）、显示（display）几个词经常看到，有必要梳理清楚它们分别干了什么事。我自己理解：
* 绘制：在内存数据结构中计算UI元素的关键信息，比如点的坐标、线的端点位置，线的颜色、矩形长宽等；
* 渲染：在内存数据结构中生成图像像素数据，比如一块区域480×270个像素点颜色值、某个字体灰度像素、图片视频编码解码等；
* 显示：将渲染好的数据转换为指令送到显示设备，多块渲染数据还要处理遮挡、重叠等工作，送显之后就是硬件设备的事情了，即在屏幕上生成人眼可见的图像。

# 0x02 Linux图形技术栈
软件系统是分层级的，为完成图形显示功能，整套系统形成技术栈（graphic stack）。
## 以X Server为中心
在Linux早期，图形技术栈的设计以X Server为宇宙中心，渲染、送显的工作都交给X Server，也只有X Server能访问硬件显示设备。X应用程序给出的渲染指令是**硬件无关**的，或者，根本不能叫渲染指令，X Server含有显示设备专门适配的X驱动程序（DDX，Device Dependent X），渲染指令由X Server/DDX来做转换，X应用程序**无法也无需**了解底层硬件规格，从而达到屏蔽硬件细节的目标，保证相同的X程序代码在不同的显示设备上正常运行。这在2D情况还凑合。DDX是X Server的组成部分，运行在用户空间，也被称为视频驱动或图形驱动。
![X Server+DDX 图源：维基百科](https://pic2.zhimg.com/80/v2-caba8ebfbe8289099d035975ad319a5b_720w.png)

早期的显卡功能简单，等同于帧缓冲设备（framebuffer device），Linux内核中有`fbdev`子系统提供一套操作接口。“一切皆文件”，对`/dev/fbN`（N是设备号）的读写操作，会映射为对显示设备的操作。X Server/DDX最后是调用`fbdev`接口写帧缓冲完成显示过程。

`fddev`自身也有局限：
* 一方面`fbdev`满足不了利用GPU硬件加速需求，都得靠CPU执行用户态程序去算，2D的情况勉强还凑合，3D就性能不足了，因为CPU的指令设计是面向通用目的，不适合这种专业活；
* 另一方面，显卡需要管理显存中的绘制命令队列、显存的缓冲等等，而用户态程序有权限直接访问显存，这在X Server独占访问权时还没什么，当多个用户态程序同时使用同一部分显存则会出现冲突，缺少一个像管理进程的调度器角色。

## Indirect Rendering
随着3D图形需求的提高，`OpenGL`定义了一套3D图形接口规范，在Linux中就希望能有支持硬件加速的`OpenGL`库（libGL），但由于X Server是宇宙中心，libGL无法直接操作显示设备，也就不能利用硬件加速特性。Linux图形技术栈搞了个补丁，设计了X协议扩展——`GLX`（OpenGL Extension for X）协议，将OpenGL指令交给X Server，由X Server调OpenGL库（Utah-GLX）再转换为真正的硬件指令给显示设备，如2D渲染一样。这种方式称为间接渲染（Indirect Rendering），3D渲染数据量尤其庞大，每个程序还要等待X Server处理完前一个请求才能连接，中间还需要转换来、转换去，效率低下，无法保证实时性。
![Indirect Rendering技术栈 图源：维基百科](https://pica.zhimg.com/80/v2-f254d5453ccf49a229e75029ca75474c_720w.png)

## DRI
应用程序对直接访问硬件进行渲染有着极高需求，势必要打破这种以X Server为宇宙中心的设计了。与间接渲染相对应的是直接渲染，1999年，XFree86 4.0项目（现归属X.org）中首创了直接渲染框架（Direct Rendering Infrastructure，DRI），具体实现需要X Server、Linux内核子系统、应用程序库配合改造。
* 在渲染工作层面，Linux内核新增了DRM（Direct Rendering Manager）子系统, 即直接渲染管理器，**独占显卡的使用权**，负责维护命令队列、显存等资源，任何程序想利用显卡进行GPU硬件加速3D渲染、视频解码等，得调`DRM`子系统的API，`DRM`子系统做仲裁和调度，避免冲突；它还负责多个GPU切换等。`DRM`完成渲染后的结果，以图像buffer形式返回给应用程序。`DRM`子系统的实现有内核态`drm`驱动和用户态`libdrm`库两级，drm驱动对外暴露`open`、`close`、`ioctl`等标准的驱动接口，`libdrm`对drm驱动的进一步封装，向上暴露DRM API。
![DRM子系统的层级 图源：csdn](https://img-blog.csdnimg.cn/20201126195405424.png#pic_center)
* 在显示层面，仍由X Server（或Wayland Compositor）统一管理，原来`DRM`子系统中的kms功能在Linux 3.1.2内核中分离出来，形成了`KMS`（kernel mode-setting）子系统，X Server（或Wayland Compositor）处理所有应用程序送来的图像buffer，处理遮挡、重叠工作，通过系统调用进入内核`KMS`子系统完成显示。

插播一句，drm驱动在Linux内核中基本等同为显卡驱动，随同内核树发布，这意味着需要开源。作为显卡大厂中的死硬分子，英伟达（NVIDIA）就是不提供开源驱动，于是一群爱好者通过闭源驱动进行逆向，搞出了开源的[nouveau项目](https://nouveau.freedesktop.org)，后来还有了英伟达自家员工的参与。这种搞法，性能、稳定性毕竟不如人家原来的闭源驱动好，只能凑合用。这也引发了Linus大神的愤怒，于是诞生了那张著名的竖中指照片。
![Linus大神大骂英伟达](https://pic1.zhimg.com/80/v2-715b1b290220a7638c959f75b2815a38_720w.png)

作为DRI框架中的主要部分，`DRM`的功能范围在多年的发展中扩大了很多，并且覆盖了很多以前在用户空间程序处理的功能，例如framebuffer管理、GEM（Graphics Execution Manager）、KMS（kernel mode-setting），这其中有的在后来分离出去形成了新的内核模块等。

在DRI框架之下，应用程序调用“DRI驱动”（用户态），即OpenGL API的具体实现库，支持硬件加速特性，由设备厂家提供或者开源的[Mesa](https://en.wikipedia.org/wiki/Mesa_(computer_graphics))，再由“DRI驱动”调`libdrm`进入内核DRM模块，完成渲染并返回图像buffer给应用程序；X Server原先的DDX驱动做2D渲染，也可以/必须使用DRM子系统。应用程序**通知**X Server（或Wayland Compositor）渲染已完成，由后者进入内核`KMS`模块完成显示。在DDX时代通过`fbdev`模块操作显示设备的方式基本不使用了，被整合到了`DRM`中，除了嵌入式场景依然保留framebuffer操作方式（配置X.org的driver为`fbdev`）。
![DRI中的X 图源：维基百科](https://pic3.zhimg.com/80/v2-bff87da92c45a76ecd9347575b912e2b_720w.png)
![DRI中的Wayland 图源：维基百科](https://pic3.zhimg.com/80/v2-c9569f4ce07d5626118fe7ae229b138b_720w.png)

* 在渲染过程，对于X，X客户端的3D渲染指令仍然绕不开X Server，必须经过X Server做指令转换后调“DRI驱动”（OpenGL API实现库，如[Mesa](https://en.wikipedia.org/wiki/Mesa_(computer_graphics))），仍然是间接渲染方式，无法直接利用硬件加速特性；而对于Wayland，客户端可以直接调“DRI”驱动下发3D渲染指令，也可以转交给Wayland Compositor。
* 在显示过程，对于X，X客户端、X Server、X窗口管理器之间有繁琐的双向通信流程；而对于Wayland，Wayland Compositor相当于合并了X窗口管理与X Server的功能，就省事多了。
这两方面的一对比，可见出Wayland比X效率提高不少。

DRI框架虽然是为3D图像渲染而生，但其用途已远不止于此。目前Linux系统中图形技术栈格局如下，可以看到支撑图形渲染的驱动主体分为了用户态、内核态两部分，原来X Server（用户态）中的部分功能被剥离出来，**如果将来用户态的驱动也能进入到核内，那性能将进一步飞跃**。
![目前Linux系统图形技术栈格局 图源：维基百科](https://pic1.zhimg.com/80/v2-7422e6d3063b1a11917e904c344faa2d_720w.png)


# 参考
* [linux图形栈的演进](https://blog.csdn.net/ytfy339784578/article/details/103946773)
* [https://en.wikipedia.org/wiki/Direct_Rendering_Infrastructure](https://en.wikipedia.org/wiki/Direct_Rendering_Infrastructure)
* [A brief introduction to the Linux graphics stack](https://blogs.igalia.com/itoral/2014/07/29/a-brief-introduction-to-the-linux-graphics-stack)
* [linux驱动之framebuffer](https://www.cnblogs.com/gzqblogs/p/10105804.html)
* [Linux DRM（一）](https://blog.csdn.net/dearsq/article/details/78312052)
* [Linux DRM（二）](https://blog.csdn.net/dearsq/article/details/78394388)
* [Libdrm 库](https://blog.csdn.net/weixin_41028621/article/details/110202300)
* [Free and open-source graphics device driver](http://en.wikipedia.org/wiki/Free_and_open-source_graphics_device_driver)
* [Linux graphic subsytem(1)_概述](http://www.wowotech.net/graphic_subsystem/graphic_subsystem_overview.html)
* [Linux graphic subsystem(2)_DRI介绍](http://www.wowotech.net/linux_kenrel/dri_overview.html)
  
# 更多阅读
* [Cygwin系列（十二）：了解X](/2020/2020-04-25-Cygwin系列（十二）：了解X.html)
* [Cygwin系列（十三）：折腾X](/2021/2021-10-30-Cygwin系列（十三）：折腾X.html)
* [WSLg：为WSL增光添彩](/2021/2021-06-02-WSLg：为WSL增光添彩.html)
* [微软WSL——Linux桌面版未来之光](/2019/2019-05-08-微软WSL——Linux桌面未来之光.html)
* [Cygwin系列（九）：Cygwin学习路线](/2019/2019-06-16-Cygwin系列（九）：Cygwin学习路线.html)
* [Python自动操作GUI神器PyAutoGUI](/2020/2020-11-27-Python自动操作GUI神器PyAutoGUI.html)
* [伪码人专栏目录导航](https://zhuanlan.zhihu.com/p/102460964)
* [silaoA的博客.https://silaoa.github.io](https://silaoa.github.io)

---
**如本文对你有帮助，或内容引起极度舒适，欢迎分享转发或点击下方捐赠按钮打赏** ^_^