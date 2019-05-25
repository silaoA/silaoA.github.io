---
title: IEEE-Spectrum-20190415-为太空而生的充气机器人
date: 2019-05-05 10:54:20
categories: 技术
tags: [IEEE, 机器人, 科技]
toc: false
comments: false
---

本文译自[IEEE Spectrum](https://spectrum.ieee.org)，[IEEE Journal Watch](https://spectrum.ieee.org/static/journal-watch)专题追踪工程和计算机科学领域最新成果，如有错误欢迎指正。本文同步发布于知乎（账号silaoA）和微信公众号平台（账号伪码人）。
关注学习了解更多的Cygwin、Linux技术和科技动态。

原文链接：<https://spectrum.ieee.org/automaton/robotics/robotics-hardware/inflatable-robots-for-space>。

在NASA资助下，研究人员探索充气机器人用于未来太空任务。

<!--more-->
<!-- [toc] -->

很遗憾充气机器人并不多见，恰恰是它有几方面出奇的优点而成为大众梦寐以求的款：廉价，几乎由塑胶制成，组装简单。相较于其他机器人，充气机器人重量特别轻，放气后打包尺寸缩减到极小。尽管其貌不扬，充气机器人天然地液压驱动，可以做得很结实且行动迅速，更重要的，在大多数情况下足够安全。充气机器人没有碍事的硬零部件，惯性小。

恰恰由于太松软，造成了充气机器人的缺点，不擅长精确、可重复的控制，难以追踪所有部件的精确位置，使得操纵机器人成为一项挑战。[Brigham Young大学](https://www.byu.edu)在NASA资助下研究这个问题，使用一个名为King Louie的可充气类人机器人。
![Brigham Young大学的充气机器人，图源：https://spectrum.ieee.org](https://spectrum.ieee.org/image/MzI3NDE5NQ.jpeg)

King Louie由[Pneubotics](http://www.pneubotics.com)公司制造，初次面世是在5年前，那时它尚为[Otherlab](https://otherlab.com)公司的一部分。2013年Google I/O大会上也展示过类似的玩意儿，双方竞争都试图干掉对方。这个机器人装满了可膨胀的气舱，起到肌肉的作用，气舱膨胀的时候可以移动手臂，根据填充的气压收缩。King Louie有一对4自由度手臂，NASA演示视频中（机器人Kaa）则是6自由度单臂。
![装满气舱的King Louie，图源：https://spectrum.ieee.org](https://spectrum.ieee.org/image/MzI3NDE4OQ.jpeg)

天生的可塑性让充气机器人好玩的同时也增加了控制难度。要控制机器人，需要在任何给定时刻精确感知四肢、关节的配置，但充气机器人不能使用刚体机器人所用的关节编码器。Brigham Young大学另寻他法，给King Louie装上了HTC Vive（虚拟现实头盔）动作追踪系统的标记器（marker），用来计算机器人手臂关节角度。尽管有了这套外部追踪系统，充气机器人的特质仍带来一些难题：机器人上部分弯曲的造型难以模拟，每次泄气再充气后内部结构都稍微有所不同，会影响到关节性能。

为解决这个问题，研究人员使用了视觉伺服（visual servoing）技术，听起来比较奇特。这项技术基本上是在观测机器人手臂朝向预设位置移动过程中不断给予反馈，如“向左来点儿，更高一点，再高一点儿，移多了，很好完美！”。视觉伺服技术这样有条不紊地工作，但确实非常可靠（robust，鲁棒性好），甚至是在机器人特性重大变化之后仍有效，这种情况是可能发生的，比如充气机器人搬起来一块大石头后再也不能按预期动作了。

NASA资助这项研究是认为充气机器人用在太空探索任务堪称理想：尺寸小、重量轻、耐用、安全。研究人员需要充分了解充气机器人的动力特性，使之保持高可用性，特别是自主性，这还有许多工作要做。另一方面，在航天员指导下，机器人可以提供辅助工作。还有一条可以说明充气机器人是太空任务的绝佳角色：如果所有的手段都失败了，还能呼吸他们（充的气体）。