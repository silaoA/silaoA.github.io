---
title: WSLg：为WSL增光添彩
date: 2021-06-02 12:35:27
categories: 技术
tags: [Linux, Windows, WSL]
toc: true
comments: false
---

本文共1800余字，预计阅读时间6分钟，本文知乎连接：[WSLg：为WSL增光添彩](https://zhuanlan.zhihu.com/p/377263437)，本文同步发布于[silaoA的博客](https://silaoa.github.io)。
关注学习了解更多的Cygwin、Linux技术。

2021年微软的[build大会](https://register.build.microsoft.com)没有刻意抢[Google IO 2021](https://events.google.com/io/)之前，这一次微软继续带来了Visual Studio、Teams、.NET、VS Code、MAUI等一系列新内容，不忘践行“Microsoft ❤ Developer”、“Microsoft ❤ Linux”，还带来`WSLg`。

<!--more-->
<!-- [toc] -->

# 0x00 回顾WSL雄心
**2016年**，微软在Windows 10周年更新中，引入了新的子系统`WSL`（Windows Subsystem for Linux），刚出来的时候叫“Bash on Ubuntu on Windows”还是测试版，到Windows 10 1709版时变为正式版并更名为WSL，把基本的linux系统带到了Windows 10中。其中的黑科技，莫如用Windows的API构建一套Linux模拟器（模拟机制），动态地将Linux程序中的system call生生翻译成Windows API调用，达到二进制兼容Linux程序的目标。毕竟是内核设计的差异，这种模拟无法实现100%全部翻译，即使100%翻译了性能也可能打折扣。

**2019年**，微软官宣了`WSL2`，翻译系统调用的坑太大填不下去了，第二代`WSL`改成了真Linux内核套虚拟机的方式。从模拟到虚拟，一字之差，技术路线迥异，魔改起来更得心应手。微软调教的Linux内核配上自己家HyperV，比起普通虚拟机搭配Linux发行版的效率高不少。`WSL2`所用的Linux内核，也是开源的，项目地址：[https://github.com/microsoft/WSL2-Linux-Kernel](https://github.com/microsoft/WSL2-Linux-Kernel)，适配`WSL2`的Linux发行版也越来越多。

**2020年**build大会，微软立下了在`WSL`中支持Linux GUI程序的目标，并做了初步的演示。前面的`WSL`和`WSL2`已经很好地解决了在Windows中运行Linux命令行程序的问题，配上漂亮大气的`Windows Terminal`终端，大大提高Windows 10生产力，从Mac阵营多多少少争取了些粉丝过来。但微软有着更远的目标，要做就做全，大厂就是这么大气。

**2021年**4月，微软放出了在`WSL2`中支持Linux GUI程序技术的预览版产品，叫做`WSLg`，并且开源，项目地址：[https://github.com/microsoft/wslg](https://github.com/microsoft/wslg)，此次[build 2021大会](https://register.build.microsoft.com)只是再次介绍了下。

`WSLg`是Windows Subsystem for Linux GUI的缩写，意图在WSL中支持直接运行Linux GUI程序，界面和Windows桌面环境无缝集成。`WSLg`在预览版之后不久就发布了正式版。

# 0x01 无缝的桌面体验
在`WSLg`之前，只能通过终端（比如微软自家的`Windows Terminal`）运行WSL发型版中的命令行程序，常见的如运行vim/emacs写代码，然后编译、调试、部署一把梭过去，同时Windows上可能开着QQ、微信、Excel...，也不用虚拟机来回切换了，就在一台Windows 10机器上搞定。
![一把梭就是干](../pic/一把梭老人.png)

有了`WSLg`后，可以再大胆点在终端敲`gedit`或者其他Linux GUI程序名，用来跑Linux系统中专有的GUI程序，让干活更加顺手。微软宣传的“无缝体验”，“无缝”在哪儿呢？先看下面两个动图。第1个是执行`gvim`和`gedit`，第2个执行的是一个机器人仿真软件，利用了GPU进行3D加速，跑大型软件也毫无压力。

![运行gedit和gvim（图源：devblogs.microsoft.com）](https://devblogs.microsoft.com/commandline/wp-content/uploads/sites/33/2021/04/GUIAppsBlogPostDemo-GIF1-Editor.gif)

![利用GPU 3D加速跑机器人仿真软件（图源：devblogs.microsoft.com）](https://devblogs.microsoft.com/commandline/wp-content/uploads/sites/33/2021/04/GUIAppsBlogPostDemo-GIF4-ROS.gif)

从上可见，Linux GUI程序的窗口就像是Windows中的普通程序一样显示、叠加，在底下的任务栏也有图标，基本上与Windows桌面环境融为一体，如果不是Linux GUI程序风格与Windows差异过于明显，压根看不出来这是Linux GUI程序。这，是怎么做到的？

# 0x02 WSLg怎么做到的
了解Linux GUI程序机制就势必绕不开`X Window System`，可以参考[Cygwin系列（十二）：了解X](https://zhuanlan.zhihu.com/p/134325713)。要让图形窗口显示出来，光有X客户端程序是不够的，还得要一个X Server来负责显示。按照一般想法，有下面两个技术路线：
1. 在Windows上部署一个的Win32版X Server，比如开源的`Xming`、`vcXsrv`或者`Cygwin/X`、商业的`xshell`、`mobaxterm`等，已有他山之石在前，只是X Server本身还要占个窗口，以及X窗口管理器与Windows的窗口管理之间平衡等等问题，达到“无缝”的体验得微软独家调教、继续魔改；
2. 在`WSL`里塞一个X Server补齐Linux桌面环境，这样会造成WSL发行版不够轻量，也许和普通虚拟机搭完整Linux发行版就差不多了，达到“无缝”体验嘛同样得魔改。

![WSLg架构（图源：devblogs.microsoft.com）](https://devblogs.microsoft.com/commandline/wp-content/uploads/sites/33/2021/04/diagram-description-automatically-generated.png)

毕竟微软不一般，**微软另辟了第三条路线**。微软做了个和WSL用户发行版（User Distro，如Ubuntu、Debian、openSUSE、CentOS等）平级的“WSLg System Distro”，这个Linux发行版原本是微软内部在Azure云上的Linux系统，叫做“CBL-Mariner”，现在经过精心剪裁一番魔改之后，专门干一件事——跑X Server。好了现在X Server有了，但图形界面是显示在`WSLg System Distro`里，怎么进到这个系统里看呢？微软在`WSLg System Distro`里加了远程桌面服务套件——`FreeRDP`，一个支持Windows远程桌面协议（RDP）的服务端，如果是在Windows里装Linux虚拟机的话，可以用Windows自带的远程桌面客户端和虚拟Linux系统中的`FreeRDP`通信。就这样，通过Windows -> RDP -> X SerVer一层套一层，把图形窗口“无缝”地集成到了Windows桌面环境中，整个形成了上图的架构。

上面提到`WSLg`里的X Server其实说法不准确，应该叫Wayland Compositor。Wayland作为后起之秀，大有赶超X取而代之之势，很多Linux发行版都在把桌面环境默认显示服务从X换成Wayland。`WSLg`用到的其实是一个Wayland Compositor，就是Wayland官方给出的参考实现`Weston`。`Weston`可以对接Wayand客户端程序，但传统的X11客户端程序也不可忽视，可以通过[XWayland](https://wayland.freedesktop.org/xserver.html)进行协议转换，最后都交到`Weston`来显示图形窗口，这样同时支持Wayland和X11客户端程序。 

整个`WSLg System Distro`、Windows远程桌面客户端（mstcs.exe）的窗口都是对用户隐藏的，猜测是用户开启WSL或者敲完Linux GUI程序名的时候，这些隐藏的组件就已经在后台运行了。具体来说就是上图的`WSLGd`，微软解释说一个类似守护进程的程序，负责启动`Weston`、建立RDP通信等工作，如果它们挂了的话还要负责重启它们。

这样完整看下来，把整个`WSLg System Distro`称作`WSLg`才是恰如其分的，支持Linux GUI程序的核心都在于此。那么问题来了，绕一大圈专门做`WSLg`，并不比上面提的路线2先进，反有舍近求远的意思，为啥不直接走路线2呢？其实也好理解，个人认为是职责分工造成的：
1. 走路线2的话需要各User Distro的厂商去魔改、适配WSL和Windows，他们没有这个义务配合微软（商业利益分成另说），要么就只能反过来微软一个个去适配他们；
2. 即使各User Distro的厂商有这个意向，需要深入Windows API的时候，不如微软自家的工程师团队熟悉，何况Windows中有大量不对外公开的API，只有自家人最了解Windows，而且也是微软最有动力（利益导向）去做Linux GUI程序支持，理应出力最多；
3. 将`WSLg`独立出来，有利于微软对`WSLg`的把控，保持在不同User Distro有一致性体验，专注`WSLg`的开发和快速迭代升级，回到第1条的话易造成碎片化，而且基于微软自家现有的CBL-Mariner，`WSLg`不必另起炉灶，减少成本。

至少目前来看，`WSLg`主要是作为显示服务器，并没有把全部的桌面环境要素放进来，WSL中也没有，因此**不能像在完整Linux发行版中那样点桌面图标来启动程序**，Linux GUI程序（客户端）仍要在终端敲命令启动。

# 0x03 后记
`WSLg`获得了虚拟GPU（vGPU）支持了，还有了硬件加速的`OpenGL`图形库支持，机器配置兼容WDDM v3.0（Windows Display Driver Model）的GPU的话，跑3D程序将更为流畅。要体验`WSLg`，可参照项目地址[https://github.com/microsoft/wslg](https://github.com/microsoft/wslg)安装或升级。

# 参考
- <https://devblogs.microsoft.com/commandline/the-initial-preview-of-gui-app-support-is-now-available-for-the-windows-subsystem-for-linux-2/>
- <https://devblogs.microsoft.com/commandline/wslg-architecture/>
- <https://x.cygwin.com>
- <https://sourceforge.net/projects/xming>
- <https://sourceforge.net/projects/vcxsrv>
- <https://mobaxterm.mobatek.net>
- <http://www.xshellcn.com>

# 更多阅读
- [微软WSL——Linux桌面版未来之光](/2019/2019-05-08-微软WSL——Linux桌面未来之光.html)
- [Cygwin系列（十一）：折腾终端2](/2020/2020-01-13-Cygwin系列（十一）：折腾终端2.html)
- [Cygwin系列（十）：折腾终端1](/2019/2019-12-29-Cygwin系列（十）：折腾终端1.html)
- [Cygwin系列（五）：Shell命令行初体验](/2019/2019-03-13-Cygwin系列（五）：Shell命令行初体验.html)
- [Python操作Excel文件（0）：盘点](/2019/2019-12-03-Python操作Excel文件（0）：盘点.html)
- [Cygwin系列（九）：Cygwin学习路线](/2019/2019-06-16-Cygwin系列（九）：Cygwin学习路线.html)
- [伪码人专栏目录导航](https://zhuanlan.zhihu.com/p/102460964)

---
**如本文对你有帮助，或内容引起极度舒适，欢迎分享转发或点击下方捐赠按钮打赏** ^_^