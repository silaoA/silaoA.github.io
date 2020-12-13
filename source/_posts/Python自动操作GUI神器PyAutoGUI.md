---
title: Python自动操作GUI神器PyAutoGUI
date: 2020-11-27 13:22:19
categories: 技术
tags: [Python]
toc: true
comments: false
# layout: false
---
本文共3500余字，预计阅读时间12分钟，本文同步发布于[知乎专栏](https://zhuanlan.zhihu.com/p/302592540)和微信公众号平台。
关注学习了解更多的Cygwin、Linux、Python技术。

日常使用计算机，命令行程序可以说是为批量操作文件而生，但作为普通用户，最多的还是通过鼠标键盘操作形形色色的图形界面程序。试想下面一个场景：有成千上万个文件，都需要通过图形界面进行同样的一套编辑、保存工作，靠手工一遍一遍地重复做，累死人不说，时间久了必然出现错误，**作为程序猿，怎么能忍重复3次以上的工作，必须利用程序自动化**。要想图形界面也能像命令行程序那样精确控制，就需要GUI自动化工具了。不得不赞Python生态之丰富，GUI自动化自动化工具也有多种库可选，比如Windows平台的pywin32，以及本文主角——跨平台的PyAutoGUI。

<!--more-->

# 0x00 PyAutoGUI简介
`pywin32`直接包装了几乎所有的Windows API，可以方便地从Python直接调用，把Windows API按照功能分了一些大类，每一个大类作为一个模块，常见如`win32api`、`win32gui`、`win32com`等，其中`win32com`使用微软独门的COM接口技术进行进程间通信，可以实现控制GUI程序。但前提是，这些程序得支持COM接口。`win32api`则更加原始，完全通过Win32 API调用获得/生成窗口句柄、发送消息事件，十分繁琐。还有的项目如[pywinauto](https://github.com/pywinauto/pywinauto)，则是在`pywin32`基础上进一步封装，但本质思路还是一样的。

`PyAutoGUI`的思路与此完全不同，它是接管了鼠标、键盘使用权，基本上完全照搬人的操作，底层不必套牢在Windows系统，没错，它是**跨平台**的。官网地址：[https://github.com/asweigart/pyautogui](https://github.com/asweigart/pyautogui)。原本，这类GUI自动化工具的初衷是给GUI程序自动化测试用，产生点击鼠标、敲击键盘的行为，在日志中记录下消息事件和GUI程序的响应结果，事后分析GUI程序可能存在的bug。不过，既然能产生点击鼠标、敲击键盘的行为，我们就可以用来控制GUI程序批量完成文件编辑、保存工作。

按照官方的说法，`PyAutoGUI`是给人类用的GUI自动化神器，简单高效、函数分类清晰，它被[awesome-python](https://github.com/vinta/awesome-python)、[awesome-python-cn](https://github.com/jobbole/awesome-python-cn)收录。

# 0x01 PyAutoGUI使用入门
## 安装
推荐通过pip安装，一行命令搞定。
```bash
python -m pip install -U pyautogui
```
`PyAutoGUI`依赖于`pyscreeze`、`pymsgbox`、`pytweening`，上述命令会自动安装这3个库。安装完成后可以发现，在site-packages\pyautogui有6个文件，名字带java、osx、win、x11的是在不同平台的实现方案，再在__init__.py和__main__.py中检测当前系统平台进行封装。其中，java平台的实现文件为空，猜测是未来计划支持的，先占个坑。
```bash
__init__.py
__main__.py
_pyautogui_java.py  # 空文件，猜测是未来支持
_pyautogui_osx.py
_pyautogui_win.py
_pyautogui_x11.py
```

## 快速上手
`PyAutoGUI`设计简洁，相关符号经过内部`import`之后，被封装在`pyautogui`单个模块中，因此Python程序中只要`import pyautogui`之后便可通过`.`符号访问`pyautogui`中的函数、变量。`pyautogui`中函数大致分为通用功能、鼠标控制、键盘控制、消息窗口、截图5大类。 
1. 通用功能
```python
In [1]: import pyautogui

In [2]: pyautogui.size()  # 获取屏幕尺寸（分辨率×分辨率）
Out[2]: Size(width=1920, height=1080)

In [3]: pyautogui.position() # 获取鼠标当前位置
Out[3]: Point(x=846, y=437)

In [4]: pyautogui.onScreen(100,200) # 判断坐标是否在屏幕范围内
Out[4]: True

In [5]: pyautogui.onScreen(100,2000) # 判断坐标是否在屏幕范围内
Out[5]: False
```

坐标体系至关重要，后续鼠标位置、图片大小都根据这套体系定义。PyAutoGUI沿用了传统的坐标体系，并未重新定义，如下图所示。x的取值范围是[0, 宽度分辨率-1]，y的取值范围是[0, 高度分辨率-1]。
![屏幕坐标体系](https://pic4.zhimg.com/80/v2-23b0c0466ae08e8a3f370c5b46bb1217_720w.jpg)

2. 鼠标控制
鼠标移动，包括绝对位置移动和相对位置移动。
```python
In [7]: sizex,sizey=pyautogui.size() # 保存屏幕尺寸

# 绝对位置移动，移动至屏幕正中心，鼠标移动过渡时间duration设为1秒
In [9]: pyautogui.moveTo(sizex/2,sizey/2,duration=1)

# 相对位置移动，向右100、向上200，鼠标移动过渡时间duration设为0.5秒
In [10]: pyautogui.moveRel(100, -200, duration=0.5)
```
鼠标点击，一个`click()`函数把点鼠标的活包干，过程也可分解为`mouseDown()`、`mouseUp()`；另有在`click()`之上封装的`rightClick()`、`middleClick()`、`doubleClick()`、`tripleClick()`等函数。**鼠标点击之前允许指定要移动的目标位置，若目标位置不在运行当前Python程序的终端/IDE范围内，则可能对其他GUI程序触发鼠标点击事件，从而引起其响应，即焦点移动至其他GUI程序。**
```python
# 移动至屏幕中心点击一下左键，过渡时间0.5秒
In [16]: pyautogui.click(sizex/2,sizey/2, duration=0.5)

# 不指定x、y，在当前位置点击一下右键
In [17]: pyautogui.click(button='right')

# 移动至(100,100)点击3次左键，点击间隔0.1s，鼠标移动过渡时间0.5秒
In [18]: pyautogui.click(100,100, clicks=3,interval=0.1,duration=0.5)

# 移动至(100,100)点击2次右键，点击间隔0.5s，鼠标移动过渡时间0.2秒
In [19]: pyautogui.click(100,100, clicks=2,interval=0.5,button='right',duration=0.2)
```

滚动鼠标滚轮。
```python
# 鼠标位置不动，向上回滚2个单位，项目文档对滚动量参数说明不详
In [22]: pyautogui.scroll(2)

# 鼠标移动至(1000,700)，前下滚动10个单位
# 运行发现鼠标并没有动
In [26]: pyautogui.scroll(-10,1000,700)
```

鼠标拖曳，指从当前位置按下鼠标，移动至目标位置再释放的过程，指定目标位置同样有绝对位置和相对位置两种方式，和移动鼠标函数很像。另外，试用下来，未发现drag()函数和dragRel()的差异。
```python
# 将鼠标从当前位置拖至屏幕中心，默认左键
In [32]: pyautogui.dragTo(sizex/2,sizey/2)

# 将鼠标从当前位置向左100像素、向右200像素拖动，过渡时间0.5秒，指定右键
In [33]: pyautogui.dragRel(-100,200,duration=0.5,button='right')
```

3. 键盘控制
控制按键，也是一个`press()`函数基本把活包干，按键动作往细分解包含`keyDown()`和`keyUp()`两个过程；在此基础上封装，有`typewrite()`和`hotkey()`两个高阶一点的函数，分别用于输入字符串和按快捷键。
```python
# 键名用字符串表示，支持的所有键名，存在pyautogui.KEYBOARD_KEYS变量中，包括26个字母、数字、符号、F1~F20、方向等等所有按键
In [4]: pyautogui.press('a') # 按字母A键，字母支持大小写

# 程序向终端输入了字符a，若程序运行时输入法为中文状态，由于没有继续输入空格或回车，输入法仅列出候选字，并不会输入到终端
In [5]: a 

# 传入键名列表（按键p、按键y、空格），按键之间间隔0.1秒（默认0）
In [6]: pyautogui.press(['p','y','space'], interval=0.1)

# 运行前将输入法切换到中文状态，往终端直接输入了“培养”
In [7]: 培养

# typewrite方式一：传入字符串，不支持中文字符，因为函数无法知道输入法需要什么按键才能得到中文字符
In [9]: pyautogui.typewrite('hello, PyAutoGUI!\n')

# 程序把字符串"'hello, PyAutoGUI!"和换行符输入到了终端
In [10]: hello, PyAutoGUI!
    ...:

# typewrite方式二：传入键名列表，按键之间间隔0.1秒（默认0）
In [11]: pyautogui.typewrite(['s','r','f','space'], interval=0.1)

# 运行前将输入法切换到中文状态，往终端直接输入了“输入法”3个字
In [12]: 输入法

# 大小写字母是自动支持的，仍然尝试一次切换到大写
In [13]: pyautogui.typewrite(['capslock','p','y'])

# CapsLock按键灯被点亮，程序往终端输入了"PY"
In [14]: PY

# hotkey屏蔽了需要反复keyDown、keyUp的细节，参数是任意个键名，而非列表
In [18]: pyautogui.hotkey('ctrl', 'shift', 'esc') #调出任务管理器

In [19]:pyautogui.hotkey('alt','ctrl','delete') # 并未调出重启界面
```

4. 消息窗口
`PyAutoGUI`利用`pymsgbox`的功能，以JavaScript风格函数提供消息框功能，包括`alert()`、`confirm()`、`prompt()`、`password()`，连参数都是一致的，熟悉JavaScript的朋友不会陌生。
```python
In [24]: pyautogui.alert(text='警告',title='PyAutoGUI消息框',button='OK')
Out[24]: 'OK' # 点击的按键被返回

In [28]: pyautogui.confirm(text='请选择',title='PyAutoGUI消息框',buttons=['1','2'
    ...: ,'3'])
Out[28]: '2' # 点击的按键被返回

In [30]: pyautogui.prompt(text='请输入',title='PyAutoGUI消息框',default='请输入')
Out[30]: 'input by 伪码人' # 点OK按钮后返回输入内容

In [32]: pyautogui.password(text='输入密码',title='PyAutoGUI消息框',default='',mask='*')
Out[32]: 'We_Coder' # 点OK按钮后返回输入内容
```
![PyAutoGUI的4种消息框](https://pic1.zhimg.com/80/v2-69dd7705018fbdfd2857c6ffbc53ace8_720w.jpg)

5. 截图相关
PyAutoGUI提供了screenshot()函数进行屏幕截图，返回是Image对象，这是在Pillow库中定义的，因此需要安装Pillow库才能正常工作。
```python
# imageFilename参数，截图要保存的文件全路径名，默认`None`，不保存；
# region参数，截图区域，由左上角坐标、宽度、高度4个值确定，如果指定区域超出了屏幕范围，超出部分会被黑色填充，默认`None`,截全屏
In [41]: pyautogui.screenshot('shot.png',region=(1000,600,600,400))
Out[41]: <PIL.Image.Image image mode=RGB size=600x400 at 0x20C87497390>
```
![PyAutoGUI截图600×400](https://pic2.zhimg.com/80/v2-b6d1990b61b96c81284e9a2b32bc5c4d_720w.jpg)

`PyAutoGUI`还有个图片匹配功能，它是在屏幕按像素匹配，定位图片在屏幕上的坐标位置，`locateOnScreen()`函数返回`region`对象,即左上角坐标、宽度、高度4个值组成的元组，再用`center()`函数计算出中心坐标，`locateCenterOnScreen()`函数则一步到位，返回中心坐标。如果把需要点击的菜单、按钮事先保存成图片，可以用来自动查找菜单、按钮位置，再交由`click()`函数控制鼠标去点击。
```python
loc = pyautogui.locateCenterOnScreen("icon_xx.png", region=(0, 0,sizex/2, sizey/10) ) # region参数限制查找范围，加快查找速度
pyautogui.moveTo(*loc, duration=0.5) # 移动鼠标
pyautogui.click(clicks=1) #点击
```

# 0x02 任务示例
前文只是`PyAutoGUI`相关函数的说明，要真正熟悉它，必须结合具体任务。现有这样一个任务，成千上万个CAD图分布在几十个文件夹中，需要对每个图进行编辑操作，假使这个操作就是简单旋转下视角，再保存。AutoCAD程序本身是有脚本功能的，因此可以利用AutoCAD来完成本任务，综合考虑以下原因促使我转向PyAutoGUI的路线上来：

1. AutoCAD是商业软件，本人希望尽可能地减少对商业软件的依赖；
2. AutoCAD的脚本功能十分有限，没有循环，难以适应如此多数量的文件操作，而且去熟悉AutoCAD脚本可能比PyAutoGUI更花时间；
3. 利用PyAutoGUI熟悉GUI控制后，可以套在别的GUI程序上，未来仍有使用价值，如果使用AutoCAD则始终被限定在AutoCAD的操作范围内，价值低，而且与第一条相悖。

决定走PyAutoGUI的路线，做了如下工作：

1. 找到另一个轻量看图软件，支持简单的编辑功能，正好包括旋转视角；
2. 手工操作键盘鼠标，利用这个看图软件完成1个文件的编辑、保存流程；
3. 将涉及操作的菜单、按钮等截图保存起来备用，将上述流程代码化，落地成果就是1个操作函数；
4. 由于看图软件应需要时间，移动、点击鼠标等操作的过渡时间需要调试确定；
5. 完成1个文件的操作流程后，在主函数编写文件匹配、循环等功能，在循环中调用操作函数，初期只拿10个文件批量操作，调试无误后，主函数的文件匹配修改为整个文件夹，包含成千上万个文件。

操作函数代码。
```python
import os
import pyautogui as pag

__sizex__, __sizey__ = pag.size() #获得屏幕尺寸

def cad_turn(cad_file, outd):
    '''
    cad_file  :  要操作的CAD文件
    outd      :  文件输出路径
    '''

    c = pag.locateCenterOnScreen(__icon_open__, region=(0,0,__sizex__/10,__sizey__/20) ) # 打开图标，图标事先截图保存
    pag.moveTo(*c, duration=0.05)
    pag.click( button="left") # 点击打开按钮

    pag.moveRel(__sizex__/4, __sizey__/4) # 确保文件打开窗口获得焦点    
    pag.typewrite(message=cad_file,interval=0.1) # 输入文件路径
    pag.press('enter')

    c = pag.locateCenterOnScreen(__icon_view__, region=(__sizex__/4, 0,__sizex__/2, __sizey__/10) )  # 旋转视图图标
    pag.moveTo(*c, duration=2)
    pag.click(clicks=7, interval=1, button="left") # 点击旋转视角按钮

    c = pag.locateCenterOnScreen(__icon_saveas__, region=(0, 0, __sizex__/10, __sizey__/20) ) # 另存为图标
    pag.moveTo(*c, duration=0.2)
    pag.click( button="left") # 点击另存为按钮

    pag.moveRel(__sizex__/4, __sizey__/4) # 确保文件打开窗口获得焦点
    pag.typewrite(message=os.path.join(out, fbase+'-se'), interval=0.1) # 输入保存路径
    pag.press('enter')

    # 关闭所有可能存在的窗口，避免占用太多内存
    while( True ): 
        try :
            pag.moveTo(__sizex__/2, __sizey__/2, duration=0.3) # 延时
            c = pag.locateCenterOnScreen(__icon_close__, region=(__sizex__/5, 0, __sizex__*3/5, __sizey__/20) )
            pag.moveTo(*c, duration=0.1) # 移到“x”按钮
            pag.click( button="left") # 点击“x”按钮
        except TypeError:
            break
```

主函数代码。
```python
if __name__ == "__main__":
    p = r'x/y/z/CAD文件所在路径'
    cad_src = glob_file_list( p, r"*.DXF") # 匹配出要操作的文件
    total = len(cad_src)
    if(total == 0):
        print('无文件需处理，退出...')
    try:
        for idx,x in enumerate(cad_src, start=1):
            print("[{}/{}] 处理{}...".format(idx, total, x))
            cad_turn(x, "out")
    except Exception as e:
        print("错误:{}".format(e))
```
开启看图软件，输入法切换到英文状态，再运行Python程序，效果见[视频](https://www.zhihu.com/zvideo/1313113646529048576)。

# 参考

* [https://github.com/asweigart/pyautogui](https://github.com/asweigart/pyautogui)
* [https://github.com/mhammond/pywin32](https://github.com/mhammond/pywin32)
* [https://github.com/pywinauto/pywinauto](https://github.com/pywinauto/pywinauto)
* [https://github.com/vinta/awesome-python](https://github.com/vinta/awesome-python)
* [https://github.com/jobbole/awesome-python-cn](https://github.com/jobbole/awesome-python-cn)

# 更多阅读
* [Python项目如何合理组织规避import天坑](/2019/2019-08-15-Python项目如何合理组织规避import天坑.html)
* [Cygwin系列（九）：Cygwin学习路线](/2019/2019-06-16-Cygwin系列（九）：Cygwin学习路线.html)
* [Linux Cygwin知识库（一）：一文搞清控制台、终端、shell概念](/2019/2019-04-04-Linux Cygwin知识库（一）：一文搞清控制台、终端、shell概念.html) 
* [Linux Cygwin知识库（二）：目录、文件及基本操作](/2019/2019-05-04-Linux Cygwin知识库（二）：目录、文件及基本操作.html)
* [知乎：伪码人专栏目录导航](https://zhuanlan.zhihu.com/p/102460964)

---
**如本文对你有帮助，或内容引起极度舒适，欢迎分享转发或点击下方捐赠按钮打赏** ^_^