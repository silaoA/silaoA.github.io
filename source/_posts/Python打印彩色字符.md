---
title: Python打印彩色字符
date: 2022-12-31 16:24:50
categories: 技术
tags: [Python]
toc: true
comments: false
# layout: false
---

见惯了在黑色终端内输出白色字符，为使输出更有辨识度，我们编写的Python程序如何在终端打印出彩色字符呢？特别是出错的时候，红色字符让用户一下就关注到错误消息，像`pip`在下载安装库的时候，出错都是大段的红色字符。搜索这个问题的时候，不少教程都是介绍Linux终端打印彩色字符，提及Windows终端如何打印彩色字符的较少，这将是本文重点。

<!--more-->
<!-- [toc] -->

# 0x00 Linux终端的颜色
GNU/Linux系统以命令行见长，围绕命令行的生态完备，终端可以配置得花哨绚丽，输出也是多姿多彩。Linux终端对彩色的支持来自于“ANSI Escape Sequence”，即“ANSI转义序列”。
ASCII码中除了字母、数字、符号等文本字符外，还有一些不可见的特殊字符，ASCII码值为0~31和127，被称为“控制字符”，用于换行、回退、产生特殊信号等，其中ASCII码值为27的为`ESC`，即Escape，跟在`ESC`之后的若干字符组成转义序列，可以控制光标移动、闪烁、擦除、**产生颜色**等，关于转义序列，在[Cygwin系列（十）：折腾终端1](/2019/2019-12-29-Cygwin系列（十）：折腾终端1.html) 文中已有介绍，详见[维基百科 ANSI_escape_code](https://en.wikipedia.org/wiki/ANSI_escape_code)。

Linux的终端都能很好地支持ANSI标准，按照标准准备好转义序列和字符就打印出彩色字符，使用Shell、C、Python等任何语言都可以做到。转义序列的起始字符`ESC`怎么表达？显然不是`E`、`S`、`C`三个字符，ANSI规定可以用转义字符`\e`（**又是转义**）来表示，或者用ASCII码值转义形式`\033`表示，`033`代表8进制33，等于10进制27。转义序列`\e[<颜色代码>mtext`，用于控制文本“text”的颜色显示效果，其中<颜色代码>包括前景色、背景色，多个代码用分号隔开，颜色代码取值如下，要设置高亮，在下表前景或背景颜色代码基础上加60即可。
![](https://pic1.zhimg.com/80/v2-8962eb3263f2af33cc2a3cbc6a0b26a3_720w.png)

前景和背景色代码数值范围不同，在转义序列中二者位置互换不影响效果，终端能够根据数值不同区分开来，如`\e[32;40m`和`\e[40;32m`都合法且效果一样，都是设置背景黑色、前景绿色。还有一个特殊颜色代码`\e[0m`表示关闭所有设置**恢复到默认状态**，通常会加在要打印的文本之后，这样不影响下一段输出，做到每个字符独立控制颜色。

有了以上理论基础，就可以使用Python、Shell等来控制字符颜色了，如下可以打印出黑底红色字和白底蓝色字，并且“这是”、“和”为默认色。
```python
print("这是\e[31;40m黑底红色 \e[0m和 \e[34;47m白底蓝色 \e[0m")
```
```bash
echo -e "这是\e[31;40m黑底红色 \e[0m和 \e[34;47m白底蓝色 \e[0m"
```
![](https://pic1.zhimg.com/80/v2-95b7b6a39a0ef537163bce63c1ff0030_720w.webp)
**以上是8色终端的情况，对于支持256色的终端，颜色转义序列更为复杂**，在此不展开说了。

# 0x01 Windows终端颜色控制
Windows以图形界面见长，对命令行的生态支持力度不足，Windows的终端属于Console组件，并**不支持ANSI转义序列**，要打印彩色字符就要另寻他计。
另一方面，个人觉得转义序列可读性太差，且颜色样式与文本混在一起输出，不够优雅。
## 使用库
Python上有些第三方库增加了终端高亮和色彩渲染功能。

第1个`Rich`，字面意思表示“富文本”，文档地址：[https://rich.readthedocs.io/en/latest/introduction.html](https://rich.readthedocs.io/en/latest/introduction.html)。具有以下特性：
- 跨平台，Windows、Linux、MacOS都可用；
- 可与Jupyter Notebook配合使用；
- 导出了同名函数`print`，理论上可以替代Python内置的`print`；
- 支持表格框线、进度线、目录树线等的色彩渲染；
- 支持Markdown渲染、语法高亮（依赖`pygments`库）。
还有更多功能特性不再展开说了，就打印输出而言，是通过成对标签将要颜色渲染的文本包起来，类似于写html那样，可读性是比ANSI转义序列好些了，但仍然是样式与文本混合输出。`Rich`库的功能特性过于繁杂，远远超过“在终端打印彩色字符”的需求，感觉更像是开发文本编辑器、终端模拟器的支持包。

第2个`colorama`，基本上就是ANSI转义序列的包装，`ipython`就用到了它，项目官网[https://github.com/tartley/colorama](https://github.com/tartley/colorama)。`colorama`将颜色样式转义序列做了名称定义，引用时直接用颜色变量即可，前一节的颜色在`colorama`里变成了`Fore.RED`（前景红色）、`Back.GREEN`（背景绿色）等；另外导出了一个`Colored`类，使用`Colored.blue`、`Colored.yellow`等成员函数，可以将传入的字符串文本加上转义序列输出，供下一步`print`。

`colorama`功能极简、用法极简，而且跨平台。在支持ANSI转义序列的平台（Linux、MacOS）`colorama`除了颜色定义替换基本上啥也没做，在不支持ANSI转义序列的平台（Windows），`colorama`将打印文本中的颜色变量剔除掉，变成调用该平台的API来实现颜色样式设定。`colorama`已经很贴近需求了。

## 使用Win32 API
Windows不支持ANSI转义序列，但有Win32 API可以设置终端样式，但Python如何调用呢？Python标准库中，`ctypes`库可以加载外部动态库，这样可以把核心功能用其他语言（比如C/C++）编写提高运行效率，Python程序制作包装，对Python用户而言接口一致。`ctypes`自然能加载Windows dll，实现对Win32 API调用，其中`SetConsoleTextAttribute`函数就是设置终端文本属性，用于控制彩色打印，它有2个参数：
- 参数1：handle，表示要控制谁，包括标准输入、标准输出、标准错误，取值通过`GetStdHandle`函数获得；
- 参数2：color，表示颜色设定，取值为1个字节整数，高4位是背景色、低4位是前景色，含义一致，如下所示。4位设定中，最高位为强调（高亮）位，接下来3位是RGB三原色输出，0表示三原色均无输出，即黑色，7表示三原色均输出，即白色；3位组合共8种颜色取值，与ANSI转义序列一样，只是取值不同，结合高亮位，背景、前景各16种取值。
![](https://pic4.zhimg.com/80/v2-0a631994486d29ba71e9f5b742b6b4ff_720w.webp)
![](https://pic3.zhimg.com/80/v2-65eb1202ecfeef4aaf52f299e17e1e76_720w.webp)

## 初步实现
有了以上理论铺垫，可以写代码尝试了。
```Python
import ctypes
handle = ctypes.windll.kernel32.GetStdHandle(-11)
fg, bg = 6, 7 # 设置背景色、前景色
color =  fg + bg<<4
ctypes.windll.kernel32.SetConsoleTextAttribute(handle, color)
print("彩色文本")
fg, bg = 7, 0 # 恢复背景黑、前景白
color =  fg + bg<<4
ctypes.windll.kernel32.SetConsoleTextAttribute(handle, color)
```
上述代码片段解释如下：
- 第2行获得要控制的handle，-11表示控制标准输出，-10和-12对应标准输入、标准错误；
- 第3、4行设置color，可根据需要调整；
- 第5行设定了终端文本属性；
- 第6行将文本打印出来，终端将按照第5句渲染结果；
- 第7~9行恢复成默认色，背景黑、前景白。

按照过程其实就4步：获得标准输出句柄，设置终端属性，打印，恢复终端属性。把这4步封装成一个函数则更好用。

## print的彩色包装cprint
用“Python打印彩色字符”为关键字搜索，搜索结果千篇一律的介绍都是编写一个类，给每种彩色打印编一个成员函数，前景、背景256种组合难道配256个成员函数？这种设计未免太傻。

打印使用的是`print`函数，不需再实现了，我们所需要的是对`print`包装形成`cprint`函数（c代表colored），为方便独立设置前景、背景色，增加`fg`、`bg`两个参数。捋一下思路：
- 接口设计为`cprint(text, fg=None, bg=None)`，如果`fg`、`bg`都为默认值`None`则不做任何处理直接打印，`text`传给`print`，但这样仅打印字符串，限定了`print`功能；
- 接口设计为`cprint(fg=None, bg=None, *args, **kargs)`，可变参数`args`和关键字参数`kargs`原封不动传给`print`，这样保证`print`接口一致，而且Python规定可变参数和关键字参数必须放在最后，这样只能把`fg`和`bg`放在前面；
- 接口设计为`cprint(*values, sep='', end='\n', file=sys.stdout, flush=False, fg=None, bg=None)`，这样把`print`的5个参数都照顾到了，但在实际使用最多的是只传`values`值其他为默认，`fg`、`bg`位置在最后，要传值则只能使用关键字参数形式。
综合权衡，第2种接口最好，第1种最简单但也最差。

为使传`fg`、`bg`更简便，将颜色值及代码以字典形式存起来，增设了"reset"颜色名称用于指示默认色，函数中仅对有效参数处理，形成代码如下。
```python
import sys
def cprint(fg=None, bg=None, *args, **kargs):
	'''
	print的包装，在终端打印带颜色的文本
	[参数]
		fg：前景色
		bg：背景色，支持颜色字符串和
		其他参数，原封不动传递给print
	'''
	if(sys.platform == "win32"):
		import ctypes
		
		STD_INPUT_HANDLE = -10
		STD_OUTPUT_HANDLE= -11
		STD_ERROR_HANDLE = -12
		fg_colors={
			"black":0, "blue":1, "green":2,"cyan":3,
			"red":4,"purple":5,"yellow":6,"white":7,
			"grey":8, "light blue":9,"light green":10,"light cyan":11,
			"light red":12,"light purple":13,"light yellow":14,"light white":15,
			"reset":7 # 增加"reset"作为恢复状态
			}
		bg_colors = {k:v<<4 for k,v in fg_colors.items()}
		bg_colors["reset"] = 0 # 背景黑
		handle = ctypes.windll.kernel32.GetStdHandle(STD_OUTPUT_HANDLE)
		fcolor, bcolor = fg_colors["reset"], bg_colors["reset"] # 默认背景黑、前景白
		# fg、bg参数，既支持字典key、也支持字典value
		if( (fg,bg) != (None, None) ):
			if( fg in fg_colors.keys() ):
				fcolor = fg_colors[fg]
			elif( fg in fg_colors.values()):
				fcolor = fg
			if( bg in bg_colors.keys() ):
				bcolor = bg_colors[bg]
			elif( bg in bg_colors.values() ):
				bcolor = bg
			color = fcolor | bcolor
			ctypes.windll.kernel32.SetConsoleTextAttribute(handle, color)
	print(*args, **kargs)
	if(sys.platform == "win32"):
		if( (fg,bg) != (None, None) ):
			color = fg_colors["reset"] | bg_colors["reset"]
			ctypes.windll.kernel32.SetConsoleTextAttribute(handle, color)
```
下面来测试，将前景、背景256种组合打印出来，打印内容为整数0~255。假设`cprint`已经被`import`，以下代码利用`y`和`x`分别控制行和列，取值范围都是0~15，同时`x`是前景色、`y<<4`为背景色，`(y<<4)|x`为要打印的整数，每打印一个数不立即换行，而是设置`end="\t"`打印制表符，每行16列打印完后再换行。
```python
for y in range(16):
	for x in range(16):
		cprint(x,y<<4,(y<<4)|x, end="\t")
	print("\n")
```
效果如下，每行应用1种背景色、16种前景色，逐行输出，主对角线上的数值由于背景、前景相同而无法显示出来。从这看出来第2种接口比第1种的好处了，像`end`这样的参数可以自由控制。
![](https://pic1.zhimg.com/80/v2-0f4683f1a42793ca19bc3f74d983b914_720w.webp)
以上实现了`print`轻量级包装，即wrapper。

# 0x02 实现终端彩色打印的讨论
对于`cprint`的接口，还一种设计是将`fg`、`bg`合成为一个参数，需要用户把高4位、低4位或运算后再传值，但这样就不能直接应用颜色字符串，而是要用颜色常量，比如定义为"RED"、"GREEN"等，传值使用“(RED<<4) | GREEN”的形式，函数使用起来稍复杂一点点，也是可以的。如下，`cprint2`接口少一个参数，参数检查也更简单，移位、或运算操作留给用户。
```python
import sys
STD_INPUT_HANDLE = -10
STD_OUTPUT_HANDLE= -11
STD_ERROR_HANDLE = -12
BLACK = 0
BLUE = 1
GREEN = 2
CYAN = 3
RED = 4
PURPLE = 5
YELLOW = 6
WHITE = 7
GREY = 8
LIGHT_BLUE = 9
LIGHT_GREEN = 10
LIGHT_CYAN = 11
LIGHT_RED = 12
LIGHT_PURPLE = 13
LIGHT_YELLOW = 14
LIGHT_WHITE = 15
def cprint2(color=None, *args, **kargs):
	'''
	print的包装，在终端打印带颜色的文本
	[参数]
		color：背景色|前景色形式，8位数字
		其他参数，原封不动传递给print
	'''
	if(sys.platform == "win32"):
		import ctypes
		if(color in range(256)):
			handle = ctypes.windll.kernel32.GetStdHandle(STD_OUTPUT_HANDLE)
			ctypes.windll.kernel32.SetConsoleTextAttribute(handle, color)
	print(*args, **kargs)
	if(sys.platform == "win32"):
		if( color in range(256) ):
			ctypes.windll.kernel32.SetConsoleTextAttribute(handle, (BLACK<<4)|WHITE)
```
`cprint`两种实现，首先检测了系统平台，如果是Windows才会进一步检查参数有效性并使用`ctypes`库调用Win32 API；如果是其他，比如Linux、MacOS，则丢掉颜色参数，直接打印。

**如果程序中除了`print`还有自定义的log等函数，更优雅的思路是装饰器，在需要彩色输出的函数定义前装饰即可。**

综上，终端打印彩色字符，总体来说包括颜色样式与文本分离和混合两大类，但个人认为分离更好：
- **可读性更强**，颜色样式与文本混合，无论是转义序列还是标签，可读性都较差；
- **改造工作量小**，对现有代码中的`print`函数，在`print`前面加上`c`，再在参数列表前头加上前景、背景色即可，比转义序列的；
- **不依赖第三库**，使用标准库`ctypes`即可，方便他人使用和打包；
- 在有重定向、管道输出的场合，比如输出到日志文件，**不会输出颜色样式的转义序列或者标签，不打乱上层的函数调用**。
其中，最后两点个人认为尤需重视。

# 参考
- [Rich Documentation](https://rich.readthedocs.io/en/latest/introduction.html).
- [https://github.com/tartley/colorama](https://github.com/tartley/colorama).
- [维基百科：ANSI_escape_code](https://en.wikipedia.org/wiki/ANSI_escape_code).
- [维基百科：Win32 控制台](https://zh.wikipedia.org/wiki/Win32%E6%8E%A7%E5%88%B6%E5%8F%B0).
- [Windows文档：控制台屏幕缓冲区](https://learn.microsoft.com/zh-cn/windows/console/console-screen-buffers#character-attributes).

# 更多阅读
* [知乎专栏目录导航](https://zhuanlan.zhihu.com/p/102460964)
* [Cygwin系列（十）：折腾终端1](/2019/2019-12-29-Cygwin系列（十）：折腾终端1.html)* [Linux Cygwin知识库（一）：一文搞清控制台、终端、shell概念](/2019/2019-04-04-Linux Cygwin知识库（一）：一文搞清控制台、终端、shell概念.html)
* [Python项目如何合理组织规避import天坑](/2019/2019-08-15-Python项目如何合理组织规避import天坑.html)
* [Cygwin系列（一）：Cygwin是什么](/2019/2019-02-14-Cygwin系列（一）：Cygwin是什么.html)
* [Python操作Excel文件（0）：盘点](/2019/2019-12-03-Python操作Excel文件（0）：盘点.html)
* [Cygwin系列（九）：Cygwin学习路线](/2019/2019-06-16-Cygwin系列（九）：Cygwin学习路线.html)
* [微软WSL——Linux桌面版未来之光](/2019/2019-05-08-微软WSL——Linux桌面未来之光.html)

---
如本文对你有帮助，或内容引起极度舒适，欢迎分享转发与留言交流
![](https://pic3.zhimg.com/80/v2-2c41616595fb74ec5acfbabe9e0e125a_hd.jpg) 
![](https://pic2.zhimg.com/80/v2-2e15831707f8c58cbb6cfbf0df5a7b41_hd.jpg)
►本文为原创文章，如需转载请私信知乎账号silaoA或联系公众号伪码人（We_Coder）。

**都看这里了，不妨点个赞再走呗**