---
title: Python操作Excel文件（3）：优雅干将openpyxl
date: 2019-12-20 16:52:13
categories: 技术
tags: [Python]
toc: true
comments: false
# layout: false
---

本文共4000余字，预计阅读时间16分钟，本文同步发布于知乎专栏和微信公众号平台。
关注学习了解更多的Cygwin、Linux、Python技术。

`openpyxl`诞生于Python生态中缺乏原生读写[Office Open XML](http://www.officeopenxml.com/)格式文件（也就是xlsx格式）的背景下，由一群志愿者在业余时间开发维护，项目地址 [http://bitbucket.org/openpyxl/openpyxl](http://bitbucket.org/openpyxl/openpyxl)。相较`pyexcel`、`xlrd/xlwt/xlutils`，`openpyxl`对Excel的功能支持更加丰富，同时在实现上又十分优雅，操作逻辑与直接用Excel软件接近，运行效率也很高，堪称Excel文件操作的优雅干将。
本文所用`openpyxl`版本为3.0，旧版本可能部分API有所不同。

<!--more-->

# 0x00 读文件
`openpyxl`设计比`pyexcel`、`xlrd/xlwt/xlutils`，`openpyxl`更复杂一点，包下有`workbook`、`worksheet`、`cell`、`formattint`、`chart`、`formula`等子包，功能划分清晰，实现了`openpyxl.workbook.workbook.Workbook`（以下简称`Workbook`）、`openpyxl.worksheet.worksheet.Worksheet`（以下简称`Worksheet`）和`openpyxl.cell.cell.Cell`（以下简称`Cell`）类型，与Excel中的工作簿、表单、单元格概念相对应。读写Excel文件，就是围绕这3种对象开展。

## API
`openpyxl`对外的API十分简练，加载已有表格文件就1个函数——`load_workbook`，返回`Workbook`对象，总共接收5个参数：
1. filename，必选参数，指定要打开的Excel文件路径；
2. read_only，默认参数，指定是否只读，默认False，如果不需要编辑内容可设为`True`；
3. keep_vba，默认参数，指定是否保留VBA内容，但即使保留也不能使用，默认False；
4. data_only，默认参数，指定含公式的单元格保留公式内容还是公式值，默认False；
5. keep_links，默认参数，指定单元格中的外部链接是否保存，默认True。

## 示例
假使当前路径下，样例文件名称是`data.xlsx`，有3个表单，仅Sheet1有数据，内容如下图，其中C列2行是公式，`sum(A2,B2)`，C列3行是日期，A列4行是`TRUE`。
![data.xlsx Sheet1](https://pic3.zhimg.com/v2-4678eac6fca7adc6639f7b39c058eaa2_b.png)

可按下述示例代码加载文件。
```python
import openpyxl # 导入包
In [5]: bk = openpyxl.load_workbook('./data.xlsx')

In [6]: type(bk)
Out[6]: openpyxl.workbook.workbook.Workbook

In [7]: bk?
Type:        Workbook
String form: <openpyxl.workbook.workbook.Workbook object at 0x000001F51B0F7128>
File:        d:\program files\python36\lib\site-packages\openpyxl\workbook\workbook.py
Docstring:   Workbook is the container for all other parts of the document.
```

# 0x01 数据访问
## 索引表单
读入Excel文件拿到Workbook后，下一步就是定位到Worksheet。`Workbook`类对象有几个重要的属性和方法，用于索引Sheet。
- active属性，指代工作簿中的激活表单，也就是保存前最后操作的表单;
- get_active_sheet方法，作用与active属性相同，已废弃，**建议直接用active属性**；
- sheetnames属性，即工作簿中所有表单的名称所组成的列表；
- get_sheet_names方法，作用与sheetnames属性相同，已废弃，**建议直接用sheetnames属性**；
- get_sheet_by_name方法，使用名称索引表单，已废弃，**建议直接用名称作关键字，按字典的方式索引**，可以说相当优雅了；
- index、get_index方法，均用于指示`Worksheet`对象的序号，后者已废弃，**建议用index方法**。

`Workbook`类对象还支持直接for循环迭代，遍历`Worksheet`对象。

比较诡异的是，`Workbook`类对象给表单分配了序号，却**不支持通过序号索引表单，也*没有* get_sheet_by_index 方法**，官方认为用名称作关键字、按字典的方式索引更自然。

```python
In [8]: bk.active # 建议使用
Out[8]: <Worksheet "Sheet1">

In [9]: bk.get_active_sheet()
D:\Program Files\Python36\Scripts\ipython3:1: DeprecationWarning: Call to deprecated function get_active_sheet (Use the .active property).
Out[9]: <Worksheet "Sheet1">

In [10]: bk.sheetnames # 建议使用
Out[10]: ['Sheet1', 'Sheet2', 'Sheet3']

In [11]: bk.get_sheet_names()
D:\Program Files\Python36\Scripts\ipython3:1: DeprecationWarning: Call to deprecated function get_sheet_names (Use wb.sheetnames).
Out[11]: ['Sheet1', 'Sheet2', 'Sheet3']

In [12]: sh1 = bk.get_sheet_names('Sheet1')
D:\Program Files\Python36\Scripts\ipython3:1: DeprecationWarning: Call to deprecated function get_sheet_names (Use wb.sheetnames).

In [13]: sh1 = bk.get_sheet_by_name('Sheet1')
D:\Program Files\Python36\Scripts\ipython3:1: DeprecationWarning: Call to deprecated function get_sheet_by_name (Use wb[sheetname]).

In [14]: sh2 = bk['Sheet2']

In [15]: sh4 = bk['Sheet4'] # 不存在的名字作关键字，报错
-----------------------------------------------------
KeyError            Traceback (most recent call last)
KeyError: 'Worksheet Sheet4 does not exist.'

In [18]: bk.index(sh1) # 建议使用
Out[18]: 0

In [19]: bk.get_index(sh2)
D:\Program Files\Python36\Scripts\ipython3:1: DeprecationWarning: Call to deprecated function get_index (Use wb.index(worksheet)).
Out[19]: 1
```
## 索引单元格
拿到`Worksheet`对象后，下一步就是要索引行/列/单元格，并读取数据，`openpyxl`中行/列只不过是`Cell`对象组成的元组。`Worksheet`类对象有几个重要的属性和方法，用于支持后续操作。
- title属性，指示表单名称；
- dimensions属性，指示表单中单元格的范围，风格与Excel一致；
- calculate_dimension方法，作用与dimensions属性相同；
- min_row、max_row、min_column、max_column属性，指示表单当前最小、最大的行序号和列序号，**注意行列序号从1起始**，可以说非常人性化了；
- cell方法，一共接受3个参数，前2个是必选参数，指定行、列序号，第3个是可选参数，指定单元格的值，由此索引或创建1个`Cell`对象；
- rows、columns属性，按行（列）顺序组织的`Cell`对象生成器；
- iter_rows、iter_cols，接受5个参数，前4个参数指定最小、最大行（列）序号和列（行）序号，划出单元格范围，第5个参数values_only指示是否仅返回值，默认False，返回按行（列）排序的单元格生成器；
 
除了上述cell、iter_rows、iter_cols方法，`Worksheet`对象支持通过**切片方式索引**单元格，返回嵌套元组，既接受行（列）序号的形式，也接受Excel风格地址的形式，而且**不必区分大小写**（说的就是`xlrd`/`xlwt`，必须大写，小写错误）。无论切片方式索引或调用iter_rows、iter_cols方法，返回的是可迭代对象，可以直接用在for循环迭代中，可谓相当优雅。
```python
In [23]: sh1.dimensions
Out[23]: 'A1:C5'

In [24]: sh1.max_row,sh1.max_column
Out[24]: (5, 3)

In [25]: sh1.min_row,sh1.min_column
Out[25]: (1, 1)

In [27]: sh1.title
Out[27]: 'Sheet1'

In [32]: sh1.cell(3,2)  # 第3行B列
Out[32]: <Cell 'Sheet1'.B3>

In [33]: sh1['B3']  # 第3行B列
Out[33]: <Cell 'Sheet1'.B3>

# A2至B3，按行排序，结果可直接用在for循环中，如 for row in sh1['A2':'B3']
In [34]: sh1['A2':'B3'] 
Out[34]:
((<Cell 'Sheet1'.A2>, <Cell 'Sheet1'.B2>),
 (<Cell 'Sheet1'.A3>, <Cell 'Sheet1'.B3>))

# 第2至4行，注意4也包括在内，结果可直接用在for循环中，如 for row in sh1[2:4]
In [35]: sh1[2:4] 
Out[35]:
((<Cell 'Sheet1'.A2>, <Cell 'Sheet1'.B2>, <Cell 'Sheet1'.C2>),
 (<Cell 'Sheet1'.A3>, <Cell 'Sheet1'.B3>, <Cell 'Sheet1'.C3>),
 (<Cell 'Sheet1'.A4>, <Cell 'Sheet1'.B4>, <Cell 'Sheet1'.C4>))

# A2至B3，按行排序，结果可直接用在for循环中，如 for row in sh1.iter_rows(2,3,1,2)
In [37]: tuple(sh1.iter_rows(2,3,1,2)) 
Out[37]:
((<Cell 'Sheet1'.A2>, <Cell 'Sheet1'.B2>),
 (<Cell 'Sheet1'.A3>, <Cell 'Sheet1'.B3>))

# A2至B3，按列排序，结果可直接用在for循环中，如 for col in sh1.iter_cols(1,2,2,3)
In [39]: tuple(sh1.iter_cols(1,2,2,3)) 
Out[39]:
((<Cell 'Sheet1'.A2>, <Cell 'Sheet1'.A3>),
 (<Cell 'Sheet1'.B2>, <Cell 'Sheet1'.B3>))

In [40]: sh1.iter_cols(1,2,2,3)  # A2至B3，返回生成器
Out[40]: <generator object Worksheet._cells_by_col at 0x000001F51BB0D830>
```
## 读取单元格的值
`Cell`类对象有个value属性，指示单元格的值，除此外还有类型、显示、风格等属性值得关注。
- row、column属性，指示单元格的行列序号；
- column_letter属性，指示单元格列名称；
- coordinate属性，指示单元格Excel风格的地址名称；
- data_type属性，指示单元格value的类型，`openpyxl`中分类如下图；
- internal_value属性，值在`openpyxl`内部表示；
- hyperlink属性，单元格指向的外部链接；
- is_date属性，指示是否为日期。

![openpyxl单元格值类型](https://pic2.zhimg.com/v2-dacd0a88756c2bcfebc492b5a7c33535_b.png)

```python
In [44]: cC3 = sh1['C3'] # C3单元格

In [45]: cA4 = sh1['A4'] # A4单元格

In [46]: cC3.row,cC3.column
Out[46]: (3, 3)

In [47]: cA4.row, cA4.column
Out[47]: (4, 1)

In [48]: cA4.column_letter, cA4.coordinate
Out[48]: ('A', 'A4')

In [50]: cA4.data_type, cC3.data_type
Out[50]: ('b', 'd')

# 注意含公式的单元格，返回是公式字符串，不是求值
In [53]: sh1['c2'].value # 单元格地址小写也不影响
Out[53]: '=SUM(A2,B2)'

In [54]: sh1['B2'].value
Out[54]: 3

In [56]: sh1['c2'].internal_value
Out[56]: '=SUM(A2,B2)'

In [57]: cC3.internal_value
Out[57]: datetime.datetime(2019, 1, 1, 0, 0)

In [58]: cC3.is_date
Out[58]: True

In [59]: cC3.style_id,cC3.style
Out[59]: (2, '常规')

In [60]: cC3.font, cC3.fill # 字体和填充对象较复杂
Out[60]:
(<openpyxl.styles.fonts.Font object>
 Parameters:
 name='宋体', charset=134, family=None, b=False, i=False, strike=None, outline=None,
 shadow=None, condense=None, color=None, extend=None, sz=12.0, u=None, vertAlign=None, scheme=None,
 <openpyxl.styles.fills.PatternFill object>
 Parameters:
 patternType=None, fgColor=<openpyxl.styles.colors.Color object>
 Parameters:
 rgb='00000000', indexed=None, auto=None, theme=None, tint=0.0, type='rgb', bgColor=<openpyxl.styles.colors.Color object>
 Parameters:
 rgb='00000000', indexed=None, auto=None, theme=None, tint=0.0, type='rgb')
```
除了通过value属性获取单元格的值，还可以在迭代时明确仅返回值，也就是`Worksheet`对象的iter_rows、iter_cols方法第5个参数设为True。
```python
# 按行遍历，生成器转换为元组
In [63]: tuple(sh1.iter_rows(sh1.min_row,sh1.max_row,sh1.min_column,sh1.max_column, True))
Out[63]:
(('列A', '列B', '列C'),
 (1, 3, '=SUM(A2,B2)'),
 (0.5, -100, datetime.datetime(2019, 1, 1, 0, 0)),
 (True, -0.2, 0),
 ('A5', None, None))

# 按列迭代，访问单元格的值
In [64]: for col in sh1.iter_cols(sh1.min_column,sh1.max_column,sh1.min_row,sh1.max_row,True): 
			for val in col:  
 				print(val)

列A
1
0.5
True
A5 
列B
3
-100
-0.2
None
列C
=SUM(A2,B2)
2019-01-01 00:00:00
0
None
```

# 0x02 改写文件
## 流程概述
基本流程包括：
1. 创建工作簿，即`Workbook`对象;
2. 向`Workbook`对象中增加、删除`Worksheet`对象，也可直接索引需要编辑的`Worksheet`；
3. 编辑`Worksheet`对象，包括对单元格赋值，单元格合并、分拆，设置单元格风格，增加、删除行（列），插入图片、图表等；
4. 保存`Workbook`对象至xlsx格式文件。

## 创建工作簿
load_workbook函数从已有Excel文件基础上创建，前文已描述。
Workbook函数创建新的工作簿，有2个可选参数：
- write_only，指示是否仅写入，默认False；
- iso_dates，指示日期格式，默认False。
```python
In [67]: bk2 = openpyxl.Workbook() # 创建新工作簿

In [68]: bk2.sheetnames # 默认带了1个表单
Out[68]: ['Sheet']

In [69]: bk2.active
Out[69]: <Worksheet "Sheet">
```
## 创建、删除表单
`openpyxl`中除了`Worksheet`类，还有个`Chartsheet`类，文档对其如何使用描述不多，`Workbook`对象的create_sheet、create_chartsheet方法，分别用于创建这2种表单，均接受2个可选参数：
- title，指定表单名称，默认None；
- index，指定表单序号，默认None，默认插在最后，0为最前，负数表示逆序位置。
`Workbook`对象还有个`copy_worksheet`方法用于复制表单，但**仅限于复制同一个工作簿内的表单，不可跨Workbook对象，而且表单内图片、图表均忽略**，接受1个必选参数，即源`Worksheet`对象，该`Workbook`自动将复制的表单插在最后。
```python
In [74]: bk2.create_sheet('表单A')
Out[74]: <Worksheet "表单A">

In [75]: bk2.active # 激活表单并未变化
Out[75]: <Worksheet "Sheet">

In [76]: bk2.index(bk2['表单A']) # 递增分配序号
Out[76]: 1

In [77]: bk2.index(bk2.active) 
Out[77]: 0

In [78]: bk2['表单A'].title = 'sheetA' # 通过赋值修改表单名称

In [79]: bk2.sheetnames
Out[79]: ['Sheet', 'sheetA']

# sh1是前文In [13]赋值的表单，即data.xlsx中Sheet1
In [81]: sh1cp = bk.copy_worksheet(sh1)

In [82]: bk.sheetnames # 默认插在最后
Out[82]: ['Sheet1', 'Sheet2', 'Sheet3', 'Sheet1 Copy']

In [83]: sh1cp.title # 默认分配的名称
Out[83]: 'Sheet1 Copy'
```
`Workbook`对象的remove、remove_sheet方法均可删除表单对象，后者已废弃，建议用前者，仅接受1个必选参数，即要删除的`Worksheet`对象。这是`xlwt`所不具备的。
```python
In [92]: bk2.remove(bk2['Sheet'])

In [93]: bk2.sheetnames
Out[93]: ['sheetA']

In [94]: bk.remove(bk['Sheet2'])

In [95]: bk.sheetnames
Out[95]: ['Sheet1', 'Sheet3', 'Sheet1 Copy']
```
## 编辑数据
`Worksheet`对象支持通过赋值直接改写单元格，可以写入数字、字符串、日期、公式等等，merge_cells、unmerge_cells可以合并、拆分单元格，整个操作基本和Excel软件的逻辑相近。支持的公式放在`openpyxl.utils.FORMULAE`变量中，当前3.0版本支持352个公式。
```python
In [97]: for cx in range(2,4):
        	 col = openpyxl.utils.get_column_letter(cx)
        	 sh1[col+'5'] = col+'5' # 第5行B、C列写入字符串

In [100]: sh1['A5'] = '=SUM(A1:A4)' # 写入公式

In [101]: import datetime

In [102]: sh1['B1'] = datetime.datetime(2019,12,31) # 写入日期

In [103]: sh1['C1'] = -0.0001

In [104]: tuple(sh1.values)
Out[104]:
(('列A', datetime.datetime(2019, 12, 31, 0, 0), -0.0001),
 (1, 3, '=SUM(A2,B2)'),
 (0.5, -100, datetime.datetime(2019, 1, 1, 0, 0)),
 (True, -0.2, 0),
 ('=SUM(A1:A4)', 'B5', 'C5'))

In [105]: sh1.merge_cells('B4:C4') #合并单元格，注意值的变化

In [106]: tuple(sh1.values)
Out[106]:
(('列A', datetime.datetime(2019, 12, 31, 0, 0), -0.0001),
 (1, 3, '=SUM(A2,B2)'),
 (0.5, -100, datetime.datetime(2019, 1, 1, 0, 0)),
 (True, -0.2, None),
 ('=SUM(A1:A4)', 'B5', 'C5'))

In [108]: len(openpyxl.utils.FORMULAE)
Out[108]: 352
```
`Worksheet`对象以下方法用于整行（列）增加和删除，但需要注意，**增删行列后，公式所引用的单元格地址并未自动更新，与Excel软件的逻辑 *不一致* ！**
- insert_rows、insert_cols方法，插入空白行（列），接受2个参数，idx为必选参数，指示插入位置，序号从1起始，amount为默认参数，指示插入多少行（列），默认为1；
- delete_rows、delete_cols方法，删除行（列），参数意义与上同。
- append方法，在当前表单末尾追加值，如果传入参数是可迭代对象（如list、tuple）就按序填1行，如果传入参数是字典，则按关键字给指定的列填值，关键字可以是列序号或列名。
```python
In [110]: sh1.insert_rows(1,2)

In [111]: sh1.dimensions
Out[111]: 'A3:C7'

In [112]: sh1.delete_rows(5)

In [113]: sh1.insert_cols(3)

# 注意公式所引用单元格地址并未自动变化，与Excel软件逻辑不一致！
In [114]: tuple(sh1.values) 
Out[114]:
((None, None, None, None),
 (None, None, None, None),
 ('列A', datetime.datetime(2019, 12, 31, 0, 0), None, -0.0001),
 (1, 3, None, '=SUM(A2,B2)'),
 (True, -0.2, None, None),
 ('=SUM(A1:A4)', 'B5', None, 'C5'))

In [115]: sh1.dimensions
Out[115]: 'A1:D6'

In [120]: sh1.append(range(7)) # 末尾插入整行

In [122]: sh1.append({2:'B列',4:'D列'}) # 尾行只在B、D列插入值

In [124]: sh1.append({'B':'同上','D':'同上'}) # 尾行只在B、D列插入值

In [125]: tuple(sh1.values)
Out[125]:
((None, None, None, None, None, None, None),
 (None, None, None, None, None, None, None),
 ('列A',datetime.datetime(2019, 12, 31, 0, 0), None, -0.0001, None, None, None), 
 (1, 3, None, '=SUM(A2,B2)', None, None, None),
 (True, -0.2, None, None, None, None, None),
 ('=SUM(A1:A4)', 'B5', None, 'C5', None, None, None), 
 (0, 1, 2, 3, 4, 5, 6),
 (None, 'B列', None, 'D列', None, None, None),
 (None, '同上', None, '同上', None, None, None))
```
## 其他高阶编辑
`openpyxl.utils`包提供了一些工具函数，辅助数据处理，如前面用到的get_column_letter将列序号转换为列名，还有absolute_coordinate函数将单元格相对引用地址转换为绝对引用地址等等。

`openpyxl.drawing`包可将图片文件转换为`Image`对象，但需要在Python中事先安装`Pillow`做后端支持，否则转换报错。`Worksheet`对象支持通过add_image方法插入图片,接受2个参数：
- img，必选参数，即`Image`对象；
- anchor，可选参数，指定图片锚定坐标。
```python
In [126]: img = openpyxl.drawing.image.Image('../pic/python-logo.jpg')

In [128]: sh1.add_image(img，'F7')
```
`openpyxl.chart`包实现了各种图表类型及对象创建、绘制方法，`Worksheet`对象支持通过add_chart方法插入图表。图表的绘制过程无法交互，不能实时反映出效果，不如直接在Excel软件中绘制。

`openpyxl.style`包实现了字体、颜色、边框、对齐、填充、风格等多种类型及对象创建、制作方法，`Cell`对象的font、fill、alignment、style等属性可以直接赋值，实现单元格风格自定义。

`openpyxl.formatting`包实现了风格、规则等类型及对象创建、制作方法，`Worksheet`对象的conditional_formatting属性，提供了add方法用于添加规格和应用的风格，实现单元格的条件格式。

`openpyxl.comments`包实现了注释类型及对象创建、制作方法，`Cell`对象的comment属性可以直接赋值，实现单元格添加注释。

`openpyxl.worksheet`包实现了数据有效性等类型及对象创建、制作方法，`Worksheet`对象提供了add_data_validation方法进行添加，实现单元格数据有效性提示和检验。

上述高阶编辑属于数据处理之后“锦上添花”的功能，但同时编写代码的逻辑繁琐，不如在Excel软件中交互式操作更直观、方便，本文不推荐也不做描述。

## 保存
将`Workbook`对象保存到文件，就1个方法——save，接受参数就1个——文件名。以下示例将上述编辑过的`Workbook`对象保存至新的Excel文件。
```python
In [136]: bk.save('data-openpyxl.xlsx')

In [138]: bk2.save('bk2-openpyxl.xlsx')
```
保存后`ata-openpyxl.xlsx`文件内容如下图。
![data-openpyxl.xlsx](https://pic1.zhimg.com/v2-a5d25bc66ae959323bd757ad714e6a0c_b.png)

对于保存，有几点需要提醒：
1. **Python所有涉及Excel操作的库都不支持“原地编辑与保存”，`openpyxl`也不例外，“保存”实际上是“另存为”，只是指定保存到原文件的话，原文件被覆盖；**
2. `openpyxl`**仅支持xlsx格式，保存的文件名必须使用.xlsx扩展名；**
3. 如果创建`Workbook`对象时使用了write_only模式，则仅能调用save方法一次，不可多次保存。

# 0x03 总结
从上述读写的示例来看，`openpyxl`实现一套`Workbook`、`Worksheet`、`Cell`对象用于读写操作，对比之下`xlrd/xlwt`就显得精神分裂、要逼死强迫症的节奏了。`openpyxl`对表单、单元格进行索引、访问、赋值等操作的方式和Excel软件的逻辑天然地接近，毫无违和感，覆盖了Excel软件操作的大部分常用功能，同时对外提供的API、参数也不多，易于掌握，对比之下`pyexcel`满天花式操作、数不清的API和参数让人眼花缭乱。

`openpyxl`堪称优雅的得力干将，唯一的遗憾，大概就是缺少对xls格式的支持。

# 参考
* openpyxl Documentation，2019. [https://openpyxl.readthedocs.io](https://openpyxl.readthedocs.io).

# 更多阅读
* [Python操作Excel文件（0）：盘点](2019-12-03-Python操作Excel文件（0）：盘点.html)
* [Python项目如何合理组织规避import天坑](2019-08-15-Python项目如何合理组织规避import天坑.html)
* [Cygwin前传：从割据到互补](2019-02-05-Cygwin前传：从割据到互补.html)
* [Cygwin系列（一）：Cygwin是什么](2019-02-14-Cygwin系列（一）：Cygwin是什么.html)
* [Cygwin系列（九）：Cygwin学习路线](2019-06-16-Cygwin系列（九）：Cygwin学习路线.html)
* [Linux Cygwin知识库（一）：一文搞清控制台、终端、shell概念](2019-04-04-Linux Cygwin知识库（一）：一文搞清控制台、终端、shell概念.html) 
* [Linux Cygwin知识库（二）：目录、文件及基本操作](2019-05-04-Linux Cygwin知识库（二）：目录、文件及基本操作.html)
* [专栏：伪码人We_Coder](https://zhuanlan.zhihu.com/c_1078678205585551360)

---
**如本文对你有帮助，或内容引起极度舒适，欢迎分享转发或点击下方捐赠按钮打赏** ^_^