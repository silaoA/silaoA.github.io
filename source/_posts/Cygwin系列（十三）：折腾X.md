---
title: Cygwin系列（十三）：折腾X
date: 2021-10-30 17:05:13
categories: 技术
tags: [Cygwin, Linux, Windows]
toc: true
comments: false
# layout: false
---
本文共2000余字，预计阅读时间8分钟，本文同步发布于[知乎专栏](https://zhuanlan.zhihu.com/p/427637159)和[微信公众号](/about/index.html)平台。
关注学习了解更多的Cygwin、Linux技术。

本篇因各种各样的事情拖了好久。

大多数情况下，我们用Linux系统，是为了发挥命令行程序高效的威力，通过终端远程连接过去，一个黑框框里干完所有的活。但是，偶尔也需要运行一下图形界面程序，比如Web浏览器、Oracle安装程序等。而Linux系统主机通常做服务用，不会在图形支持方面堆很高的配置，这时我们可以利用X11的特性，在远端（Linux主机）运行X Client，但让安装了X Server的本地主机（如Windows主机）负责显示程序界面和交互。

<!--more-->
<!-- [toc] -->

# 0x00 Windows上的X Server
Windows自身的图形界面是内核不可分割的一部分，其实现不遵从X规范，X规范也主要面向UNIX、Linux等符合POSIX标准的系统。那么在Windows上怎么用上X Window System，尤其最关键的X Server？基于`XFree86`、`X.Org Server`，有开发者将其移植到了Windows系统中，比较有影响力的有`Cygwin/X`、`Xming`、`vcXsrv`、`MobaXterm`、`Xmanager`等。

## Cygwin/X
`Cygwin/X`是整个Cygwin项目的一部分，是X Window System在Windows系统上的移植实现，自由开源，初期基于`XFree86`，后来也换到`X.Org Server`。`Cygwin/X`在Cygwin环境中构建，依赖Cygwin项目的UNIX模拟层（cygwin1.dll）而运行。
`Cygwin/X`中的X Server名为`XWin`。
**本文刻意将X Client和X Server分散在两套系统中，不打算用Cygwin/X。**

## Xming
`Xming`基于`Cygwin/X`，最重要的区别是它用`MinGW`交叉工具链重新构建，可以“原生”地运行于Windows系统中，脱离了对Cygwin项目的UNIX模拟层（cygwin1.dll）的依赖。`Xming`旧版本采用GPL授权，代码托管地址 [https://sourceforge.net/projects/xming](https://sourceforge.net/projects/xming)上，最近为2017年11月发布的6.9.0.31，新的版本已停止GPL授权。新版的主页 [http://www.straightrunning.com/XmingNotes](http://www.straightrunning.com/XmingNotes)，是一个开发者个人网站，作者期望给予项目捐赠才允许下载新版本。

`Xming`十分小巧，完全安装也仅占约9MB空间。
![xming界面](https://pic1.zhimg.com/80/v2-71f648c86aee0cd61cf1fcdfc5788f58_720w.jpg)

## vcXsrv
`vcXsrv`基于`X.Org Server`，另有说法是基于`Xming`的老版本，因`Xming`新版本已停止GPL授权，`vcXsrv`图标及关闭提示等多处与`Xming`相同。`vcXsrv`最大特点是，它是切换到Windows本地使用Visual C++ 或 Visual Studio构建，自由开源，开发活跃，代码托管地址 [https://sourceforge.net/projects/vcxsrv](https://sourceforge.net/projects/vcxsrv)。

`vcXsrv`全部安装约占71MB空间，还包含了`xcalc`、`xclock` 2个经典的X客户端程序。`vcXsrv`配置文件名为.XWinrc，看起来与`Xming`、`Cygwin/X`联系密切。
![vcxsrv界面](https://pic2.zhimg.com/80/v2-5d69037eca5af25bf5b7ec1629136a6d_720w.jpg)

## MobaXterm
`MobaXterm`字面意思是一个图形化的ssh客户端，支持多标签页，事实上它还集成了一个X Server（基于`X.Org Server`），同时还集成了Cygwin环境和基本的程序命令。`MobaXterm`为商业软件，Home Edition不收费，Professional Edition收费，详见[https://mobaxterm.mobatek.net](https://mobaxterm.mobatek.net)。

## Xmanager
`Xmanager`是Xmanager公司多个软件产品的合集，包括`Xshell`、`Xftp`、`Xmanager PCX Server`等，其中`Xmanager PCX Server`为Windows平台的一个X Server，为商业软件，详见[http://www.xshellcn.com](http://www.xshellcn.com)。

介绍了这么多Windows平台的X Server，本文觉得选择轻量、开源的`Xming`试玩。

# 0x01 试玩Xming
## step1：开启 Xming Server
`Xming`附带了一个`XLaunch`指引程序，用于简化启用`Xming`过程，说白了就是通过图形界面指引让用户省掉了记忆各种参数选项用法。基本上按照默认选下一步即可，最后一步可以把配置保存起来。
![选择窗口模式——多窗口](https://pica.zhimg.com/80/v2-2d453b50b72d9b5e3d3bc962e98afb68_720w.png) 
![选择启动方式——不需要启动X Client](https://pic3.zhimg.com/80/v2-bccd36448c00ef4b5090f10febc583b3_720w.png)
![附加配置](https://pica.zhimg.com/80/v2-8ab13478904f2598876e37356f54b57d_720w.png)
![完成配置并保存](https://pic1.zhimg.com/80/v2-bf8367158a5e2d9a37179e5f8fe9f4de_720w.png)
最终的效果，和下面的命令等效。
```bash
D:\Program Files (x86)\Xming\Xming.exe :0 -clipboard -multiwindow
```
## step2：启动X Client
Cygwin中X11应用程序很多，以最简单的示例程序xeyes为例。首先需要通过Cygwin的包管理器setup程序或者apt-cyg命令安装xeyes，安装过程可以参考[Cygwin系列（七）：Cygwin软件包管理相关配置](/2019/2019-05-12-Cygwin系列（七）：Cygwin软件包管理相关配置.html)和[Cygwin系列（八）：命令行软件包管理器apt-cyg](/2019/2019-05-25-Cygwin系列（八）：命令行软件包管理器apt-cyg.html)。
![Cygwin软件仓库拥有大量X11软件包](https://pic1.zhimg.com/80/v2-070f58b6973d476b2688c03810848521_720w.png)
接下来，开启终端连到Cygwin shell，运行`xeyes`，毫不意外地，出。。。错。。。了。。。这是因为，`xeyes`并不知道负责显示的X Server在哪里，这需要用户指定，看下一步。
```bash
$ xeyes
Error: cannot open display
```
## step3：配置X应用程序

这一步就是要告知X应用程序，负责显示的X Server在哪里。老规矩先看`xeyes`程序用法，发现第一个选项`-display`就是指定X Server的显示器（display就凑合着这么翻译吧），其他还一些选项比如程序界面尺寸、前景色/背景色啥的。
```bash
$ xeyes --help
usage: xeyes
       [-display [{host}]:[{vs}]]
       [-geometry [{width}][x{height}][{+-}{xoff}[{+-}{yoff}]]]
       [-fg {color}] [-bg {color}] [-bd {color}] [-bw {pixels}]
       [-shape | +shape] [-outline {color}] [-center {color}]
       [-backing {backing-store}] [-distance]
       [-render | +render]
       [-present | +present]
```
`-display`选项值由两部分组成，中间是冒号隔开：
* host，X Server所在的主机名或IP地址，与X Client属同一个主机的话可为空；
* vs，显示器序号及屏幕序号，前面启动Xming时配置了显示器序号为0，一个显示器可能存在多个屏幕，但通常只有一个屏幕，屏幕序号为0，故vs的值为0.0。
再看xeyes -display :0.0效果，成功。鼠标移动，一对眼睛跟着转动。
![xeyes -display :0.0](https://pica.zhimg.com/80/v2-e11bc83454da0143ecf39881ef5a6150_720w.png)
程序那么多，如果运行每个X Client程序都写这么长的命令有点麻烦。好在shell支持`DISPLAY`环境变量，和上边`-display`选项意义一致。如果定义了`DISPLAY`环境变量，`-display`选项就可以跳过了。在.bashrc中写入如下一行，定义`DISPLAY`环境变量，重启Cygwin shell。
```bash
export DISPLAY=:0.0 
# export DISPLAY=localhost:0.0  # 等效
```
这次运行xeyes不加任何参数，效果如下。
![xeyes显示界面](https://pic3.zhimg.com/80/v2-889b502fa8d6e757b5434bb79788c8c6_720w.png)

## 其他话题：X11转发

X Client和X Server的直接通信是不加密的，我们通过终端经由ssh远程登录Linux主机时，可以顺手利用ssh的X11转发（X11 Forwarding）功能，可以减小对配置的修改，也使得运行X Client程序更加安全。远端主机上X Client程序的绘图请求数据，也会被ssh服务器一并转发回来，ssh客户端根据配置的显示器，再交给指定的X Server处理。远端主机并不需要定义或修改`DISPLAY`环境变量，尽可能降低对其他用户、其他程序的影响。

在ssh客户端和服务端，均需要设置“X11Forwarding yes”，ssh客户端还需要指定 x display，同远端主机`DISPLAY`环境变量意义一致。图形界面的ssh客户端，设置操作更为简单。
![putty中设置X11转发](https://datacadamia.com/_media/ssh/x11/putty_x_forwarding.jpg?buster=1274600135)

# 0x02 总结
`xeyes`程序跑通了以后，Cygwin中其他X应用程序都是一样的过程，WSL2中的X应用程序也可以和`Xming`配合着跑，甚至把GTK、KDE这样的桌面环境也可以都跑起来。WSLg更进一步，专门做了一套“WSLg System Distro”专门跑X Server，把X应用程序的图形窗口“无缝”地集成到Windows桌面环境，详见[WSLg：为WSL增光添彩](/2021/2021-06-02-WSLg：为WSL增光添彩.html)。

# 参考
* [https://x.cygwin.com](https://x.cygwin.com)
* [https://sourceforge.net/projects/xming](https://sourceforge.net/projects/xming)
* [https://sourceforge.net/projects/vcxsrv](https://sourceforge.net/projects/vcxsrv)
* [https://mobaxterm.mobatek.net](https://mobaxterm.mobatek.net)
* [http://www.xshellcn.com](http://www.xshellcn.com)

# 更多阅读

* [Cygwin系列（十二）：了解X](/2020/2020-04-25-Cygwin系列（十二）：了解X.html)
* [WSLg：为WSL增光添彩](/2021/2021-06-02-WSLg：为WSL增光添彩.html)
* [微软WSL——Linux桌面版未来之光](/2019/2019-05-08-微软WSL——Linux桌面未来之光.html)
* [Cygwin系列（九）：Cygwin学习路线](/2019/2019-06-16-Cygwin系列（九）：Cygwin学习路线.html)
* [伪码人专栏目录导航](https://zhuanlan.zhihu.com/p/102460964)
* [silaoA的博客.https://silaoa.github.io](https://silaoa.github.io)

---
**如本文对你有帮助，或内容引起极度舒适，欢迎分享转发或点击下方捐赠按钮打赏** ^_^