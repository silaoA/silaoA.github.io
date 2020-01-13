---
title: Python操作Excel文件（0）：盘点
date: 2019-12-3 21:45:33
categories: 技术
tags: [Python]
toc: true
comments: false
# layout: false
---

python的一大优势是生态丰富，各种想要的库都有人做好了，省掉造轮子的成本。对于Excel文件操作，也不例外。在Python中操作Excel有2条思路。
1. 使用`pywin32`，内含`win32com`等多个包，使用微软独门的`COM`接口技术去操控系统中的Excel软件。沿着这条思路，好处是除了Excel软件，可以操控系统中任意支持`COM`接口的软件，微软全家桶是必然在列滴，其他就难说了；劣势就是`COM`接口技术比较老旧了，仅在Windows平台中使用，而且系统中必须已经安装好Excel软件。
2. 使用读写表格文件的库，如`xlrd/xlwt/xlutils`、`openpyxl`、`pyexcel`、`XlsxWriter`等，这些库封装了对Excel文件的读写操作的函数。
![在Python中操作Excel文件](https://pic4.zhimg.com/80/v2-bb7e06dd5c94f113a9d6c8a15bb6a0bb_hd.jpg)

本文对第2条思路中常见的库进行简单评述与总结。

<!--more-->
<!-- [toc] -->

# 0x00 xlrd/xlwt/xlutils
## 简介
这3个必须捆绑到一起说。[Simplistix](www.simplistix.co.uk)公司开发了这3个库，还提供了一个教程[Working with Excel files in Python](http://www.simplistix.co.uk/presentations/europython2009excel.zip)，教程是2009年出的，原网站内容基本都清空了，项目迁移到[http://www.python-excel.org](http://www.python-excel.org/)，教程迁移到[https://github.com/python-excel/tutorial](https://github.com/python-excel/tutorial)。
- `xlrd`用于读取Excel文件表格中的数据，`rd`含义就是read，**同时支持xls和xlsx两种格式标准**；
- `xlwt`用于将数据写入Excel文件，`wt`含义就是write，**仅支持xls格式标准**；
- `xlutils`依赖于`xlrd`和`xlwt`，提供读写相关支持的工具集，比如类型转换、workbook复制等。

吐槽一句，这3个东西原本应该做到一起的，搞成3个麻烦。

## xls和xlsx格式
注意**文件后缀名跟格式标准没关系，不是说文件扩展名用xlsx就一定是xlsx格式、用xls就一定是xls格式，而是两种格式标准文件在数据组织方式、数据量支持等多方面就存在本质差异。微软自Office 2007系列之后开始广泛支持xlsx格式标准，使用xls格式标准的文件逐渐减少。**
xls是微软私有的文件格式标准，以二进制方式直接保存文件，最大支持行数65536行、列数256列，而xlsx是基于Office Open XML标准的格式，实质是以zip压缩包保存文件，只是扩展名使用了xlsx，可以用解压软件查看压缩包内容，最大支持1048576行、16384列。

## 主要特性
`xlrd/xlwt`是以非常原始的方式进行Excel表格数据读写，可以类比为CPU编程的汇编语言、Web开发中的JavaScript，好处嘛就是运行速度相对快，缺点就是代码要写得比较啰嗦。`xlrd/xlwt`实现了Book（工作簿）、Sheet（表单）、Cell（单元格）多级对象，并将单元格分为文本（Text）、数字（Number）、日期（Date）、布尔（Boolean）、错误（Error）、空白（Empty/Blank）几种类型。比较蛋疼的是，`xlrd`中的Book和`xlwt`中Book并非是同一类型，在Book对象中获取Sheet对象的方法也不同，正因为如此，才需要`xlutils`做中间桥梁进行转换。

`xlrd/xlwt`对Excel文件支持较好，除了常规数据读写，还支持单元格字体、边框线风格，还支持写入表格公式，但实际使用表明大量公式不支持，读入表格只能得到公式值，另外不支持图表绘制。

在数据访问方面，`xlrd/xlwt`可以整行/列操作，单元格索引则完全依赖行列序号，但提供了函数用于行列序号与Excel风格单元格地址相互转换。

`xlrd/xlwt/xlutils`对外暴露的类型、方法、函数中规中矩，也没有什么花式操作，用户只了解最常用的小部分也能完成工作。

# 0x01 openpyxl
## 简介
`openpyxl`是由一群志愿者在业余时间开发维护的开源项目，用于在Python中原生地读写Excel xlsx/xlsm/xltx/xltm格式文件，项目地址 [http://bitbucket.org/openpyxl/openpyxl](http://bitbucket.org/openpyxl/openpyxl)。`openpyxl`对Excel文件功能支持十分完备：除了常规数据读写，还支持单元格合并/分拆、插入图片、（有限的）图表绘制、公式写入/解析、单元格注释、只读/只写模式、单元格字体/对齐/填充/边框线风格、条件格式、数据有效性等。

## 主要特性
在数据访问方面，`openpyxl`支持整行/列操作，单元格索引既可以用行列序号，也可以用Excel风格单元格地址，内部自动转换无需用户操心。

Excel功能支持方面，`openpyxl`堪称完美，但最大痛点是不支持xls格式，而目前实际仍有不少xls格式文件在应用中。

# 0x02 XlsxWriter
## 简介
`XlsxWriter`是用于创建Excel xlsx格式文件的包，仅支持写操作，不支持xls格式，非常适合于生成新表格文件的应用场合，对于读出表格数据进行分析则无能为力。`XlsxWriter`开源，项目地址为 [https://github.com/jmcnamara/XlsxWriter](https://github.com/jmcnamara/XlsxWriter)。

在Excel功能支持方面十分完备，`XlsxWriter`支持单元格合并/分拆、图片插入、图表绘制、公式、单元格注释、单元格数字格式/字体/对齐/填充/边框线风格、条件格式、数据有效性、文本框插入、VBA等，还能与`pandas`集成进行数据分析，运行效率也很高。

## 主要特性
`XlsxWriter`在写入数据方面设计很有特点，对外只暴露1个write方法，内部则根据要写入的数据格式（Excel单元格所规定的数字、字符串、公式、空白、日期时间、链接等），回调相应的handler，同时允许用户添加自定义handler，在匹配自定义数据格式时自动回调。这样设计**仅需掌握1个write方法便可搞定所有的写操作**，简洁明了，保持灵活性同时不失统一性。

在数据访问方面，`XlsxWriter`支持整行/列操作，单元格索引既可以用行列序号，也可以用Excel风格单元格地址，内部自动转换无需用户操心。

`XlsxWriter`对外暴露的类型、方法、函数相对较为简洁，数量也不多，风格也很统一，用户上手十分容易，提供的文档和样例十分丰富。

# 0x03 pyexcel
## 简介
与上述几个库所不同的是，`pyexcel`是一个包装库，底层依赖实际是`xlrd/xlwt`、`openpyxl`、`XlsxWriter`、`lxml`等，经过封装后对外提供一套API进行Excel文件读写，可类比为CPU编程的C语言、Web开发中的jQuery等，好处就是简化了代码编写过程，缺点就是运行效率降低了。`pyexcel`开源，项目地址为[https://github.com/pyexcel/pyexcel](https://github.com/pyexcel/pyexcel)。

除了支持Excel的xls、xlsx格式，还支持csv、tsv文本格式，以及sql数据库表、Python内置的dict/嵌套list数据类型等，让用户专注于处理表格数据，不必操心数据存储介质的细节，另外**对字体/颜色/边框风格、图表绘制、公式等均不支持**。

## 主要特性
`pyexcel`被设计为插件式，模块化地支持各种功能，默认安装只有`pyexcel`、`pyexcel-io`两个包，仅支持csv、tsv文本格式，对Excel表格支持就是通过各种插件完成的，用户完全根据自己所需只安装必要的插件而不影响整体功能。
- 对xls的格式读写支持由`pyexcel-xls`插件完成，依赖于`xlrd/xlwt`；
- 对xlsx的格式读写支持由`pyexcel-xlsx`插件完成，依赖于`openpyxl`；
- 只写xlsx的支持由`pyexcel-xlsxw`插件完成，依赖于`XlsxWriter`；
- 只读xlsx的支持由`pyexcel-xlsxr`插件完成，依赖于`lxml`；
- 对ods（Open Document Spreadsheet，开放文档表单，在Open Office等开源办公软件中使用广泛）格式的读写支持由`pyexcel-ods`插件完成，依赖于`odfpy`；
- 其他更多格式支持详情，参考库文档。

在数据访问方面，`pyexcel`支持整个表单、整行/列、矩形区操作，单元格索引既可以用行列序号，也可以用Excel风格单元格地址，内部自动转换无需用户操心。

`pyexcel`对外暴露的类型、方法、函数种类繁多，参数繁多，功能多样，用户可能只需要了解其中少部分就能完成工作。

# 0x04 小结
对以上几个库的主要特性和不足总结对此如下。
![Python中操作Excel的几种常见库对比](https://pic2.zhimg.com/80/v2-c707eda765e51d14233d9df82b8a4411_hd.jpg)

上述几个库对数据分析即使有支持也十分有限，主要用来读出/写入文件，中间的数据分析过程嘛，则可以交给其他库来完成，充分发挥Python生态极度丰富的优势，如`numpy`、`pandas`等。

**若是要处理的表格文件较少，最快捷的方式仍然是各种办公软件**，微软的Office、金山的WPS、开源的Open Office / Libre Office等。遇到大量重复性的表格文件，才有必要通过程序自动化。面对这么多Excel表格文件操作库，该如何选择？如果要格式和功能的广泛支持，首选`xlrd/xlwt/xlutils`或`pyexcel`；如果没有兼容xls格式的包袱，首选`openpyxl`；如果只需要在应用中导出/生成表格，则首选`XlsxWriter`。

# 参考
* Working with Excel files in Python，2009. [www.python-excel.org](www.python-excel.org)
* openpyxl Documentation，2015. [ https://openpyxl.readthedocs.io]( https://openpyxl.readthedocs.io)
* Creating Excel files with Python and XlsxWriter，2019.[https://xlsxwriter.readthedocs.io](https://xlsxwriter.readthedocs.io).
* pyexcel Documentation，2018. [http://github.com/pyexcel/pyexcel](http://github.com/pyexcel/pyexcel)

# 更多阅读
* [Python项目如何合理组织规避import天坑](/2019/2019-08-15-Python项目如何合理组织规避import天坑.html)
* [Cygwin前传：从割据到互补](/2019/2019-02-05-Cygwin前传：从割据到互补.html)
* [Cygwin系列（一）：Cygwin是什么](/2019/2019-02-14-Cygwin系列（一）：Cygwin是什么.html)
* [Cygwin系列（九）：Cygwin学习路线](/2019/2019-06-16-Cygwin系列（九）：Cygwin学习路线.html)
* [Linux Cygwin知识库（一）：一文搞清控制台、终端、shell概念](/2019/2019-04-04-Linux Cygwin知识库（一）：一文搞清控制台、终端、shell概念.html) 
* [Linux Cygwin知识库（二）：目录、文件及基本操作](/2019/2019-05-04-Linux Cygwin知识库（二）：目录、文件及基本操作.html)
* [专栏：伪码人We_Coder](https://zhuanlan.zhihu.com/c_1078678205585551360)

---
**如本文对你有帮助，或内容引起极度舒适，欢迎分享转发或点击下方捐赠按钮打赏** ^_^