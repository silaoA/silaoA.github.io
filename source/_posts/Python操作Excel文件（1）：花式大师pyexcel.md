---
title: Python操作Excel文件（1）：花式大师pyexcel
date: 2019-12-10 15:37:45
categories: 技术
tags: [Python]
toc: true
comments: false
# layout: false
---

`pyexcel`是开源的Excel操作库，项目地址为[https://github.com/pyexcel/pyexcel](https://github.com/pyexcel/pyexcel)。它包装了一套API用于读和写文件数据，这一套API接受的参数包括2个关键字集合，一个指定数据来源，另一个指定目的文件，每个集合里都有很多关键字参数控制读写细节。`pyexcel`包还实现了工作簿、表单类型，用于访问、操作和保存数据，读写操作十分花式。

本文评述`pyexcel`库用于读写Excel文件的方法，并做总结。

<!--more-->
<!-- [toc] -->

# 0x00 读文件
## API
读文件包括`pyexcel`包级别get系列封装函数：`get_array`、`get_dict`、`get_record`、`get_book`、`get_book_dict`、`get_sheet`，可以将文件内容转换为array、dict、sheet/book等多种类型，屏蔽了文件介质是csv（comma separated values）/tsv（tab separated values）文本、xls/xlsx表格文件、dict/list类型、sql数据库表等的细节。同时还有一套同等的iget系列函数，唯一的不同是返回生成器，以提高效率。

## 示例
假使当前路径下，样例文件名称是`data.csv`（也可以是`data.xls`,但需要安装`pyexcel-xls`插件包），其内容如下:
```
col1	col2	col3
v11	v12	v13
v21	v22	v23
v31	v32	v33
v41		v43
```
可按下述示例代码读入数据内容，推荐首选`get_sheet`函数读Excel文件。
```python
# 导入库
import pyexcel as pe

In [47]: sh = pe.get_sheet(file_name='./data.csv', delimiter='\t')
# csv全称是comma separated values，默认用逗号（comma）作列分割符，其他分割符要指定，否则会被当成同一列

In [48]: type(sh)
Out[48]: pyexcel.sheet.Sheet #没有看错，直接就转成了Sheet类型

In [49]: sh # 查看sh内容
Out[49]:
data.csv: # 第一行显示sheet名称，然后是数据
+------+------+------+
| col1 | col2 | col3 |
+------+------+------+
| v11  | v12  | v13  |
+------+------+------+
| v21  | v22  | v23  |
+------+------+------+
| v31  | v32  | v33  |
+------+------+------+
| v41  |      | v43  |
+------+------+------+
```

也可以仅读取文件的一部分，传入更多参数控制行列范围，**注意行列序号从0开始**！
```python
# 范围控制：第2行开始限定最多3行、第1列开始限定最多3列
In [57]: sh_mini = pe.get_sheet(file_name='./data.csv', delimiter='\t', start_row=2, row_limit=3,start_column=1, column_limit=3)

In [58]: sh_mini
Out[58]:
data.csv:
+-----+-----+
| v22 | v23 |
+-----+-----+
| v32 | v33 |
+-----+-----+
|     | v43 |
+-----+-----+
```

`get_sheet`函数接收sheet_name参数，对于有多个sheet的Excel表格，用于指定要读取的sheet，如果缺省，则读取第1个sheet。`get_sheet`函数还接收name_columns_by_row/name_rows_by_column参数，用于将指定的行/列作为列/行名称，**默认值为0，代表第1行**，`pyexcel.sheet.Sheet`类有同名的方法进行同样的操作。

其他几个函数与`get_sheet`较相似，接受的参数也相同。
- `get_array`函数将文件数据转换为数组，即嵌套的列表，列表每个元素对应表格一行。
```python
In [65]: arr = pe.get_array(file_name='./data.csv', delimiter='\t')

In [66]: type(arr)
Out[66]: list

In [67]: arr
Out[67]:
[['col1', 'col2', 'col3'],
 ['v11', 'v12', 'v13'],
 ['v21', 'v22', 'v23'],
 ['v31', 'v32', 'v33'],
 ['v41', '', 'v43']]
``` 
- `get_dict`函数将文件数据转换为有序字典，用第1行的字段作为key，后续行值组成列表作为value。
```python
In [68]: d = pe.get_dict(file_name='./data.csv', delimiter='\t')

In [69]: type(d)
Out[69]: collections.OrderedDict

In [70]: d
Out[70]:
OrderedDict([('col1', ['v11', 'v21', 'v31', 'v41']),
             ('col2', ['v12', 'v22', 'v32', '']),
             ('col3', ['v13', 'v23', 'v33', 'v43'])])

In [71]: d['col2']
Out[71]: ['v12', 'v22', 'v32', '']
```
- `get_record`函数将文件数据转换为有序字典形成的列表，每行数据对应一个有序字典，字典将文件首行字段作为key、数据行作为value。
```python
In [72]: rec = pe.get_records(file_name='./data.csv', delimiter='\t')

In [73]: type(rec)
Out[73]: list

In [74]: rec
Out[74]:
[OrderedDict([('col1', 'v11'), ('col2', 'v12'), ('col3', 'v13')]),
 OrderedDict([('col1', 'v21'), ('col2', 'v22'), ('col3', 'v23')]),
 OrderedDict([('col1', 'v31'), ('col2', 'v32'), ('col3', 'v33')]),
 OrderedDict([('col1', 'v41'), ('col2', ''), ('col3', 'v43')])]

In [75]: rec[2]
Out[75]: OrderedDict([('col1', 'v31'), ('col2', 'v32'), ('col3', 'v33')])
```
- `get_book`函数将文件转换为`pyexcel.book.Book`对象，如果从csv文件读取，则只包含1个sheet，名称就是文件名；如果从xls文件读取，则包含xls文件中所有sheet。
```python
In [79]: bk = pe.get_book(file_name='./data.csv', delimiter='\t')

In [80]: type(bk)
Out[80]: pyexcel.book.Book

In [81]: bk
Out[81]:
data.csv: #仅有这一个sheet
+------+------+------+
| col1 | col2 | col3 |
+------+------+------+
| v11  | v12  | v13  |
+------+------+------+
| v21  | v22  | v23  |
+------+------+------+
| v31  | v32  | v33  |
+------+------+------+
| v41  |      | v43  |
+------+------+------+

In [3]: bk2 = pe.get_book(file_name='./data.xls')

In [4]: type(bk2)
Out[4]: pyexcel.book.Book

In [5]: bk2
Out[5]:
Sheet1:
+------+------+------+
| col1 | col2 | col3 |
+------+------+------+
| v11  | v12  | v13  |
+------+------+------+
| v21  | v22  | v23  |
+------+------+------+
| v31  | v32  | v33  |
+------+------+------+
| v41  |      | v43  |
+------+------+------+
Sheet2: # Sheet2是空的

Sheet3: # Sheet3是空的

```
- `get_book_dict`函数将文件数据转换为多sheet的有序字典，sheet名称作为key，sheet数据以嵌套列表形式作为value，这在含有多sheet的Excel表格中较有用；对于csv文件，由于只有1个sheet，返回的有序字典仅有1项。
```python
In [36]: bd = pe.get_book_dict(file_name='./data.xls')

In [37]: type(bd)
Out[37]: collections.OrderedDict

In [38]: bd2 = pe.get_book_dict(file_name='./data.csv')

In [39]: bd
Out[39]:
OrderedDict([('Sheet1',
              [['col1', 'col2', 'col3'],
               ['v11', 'v12', 'v13'],
               ['v21', 'v22', 'v23'],
               ['v31', 'v32', 'v33'],
               ['v41', '', 'v43']]),
             ('Sheet2', []),
             ('Sheet3', [])])

In [40]: bd2
Out[40]:
OrderedDict([('data.csv',
              [['col1\tcol2\tcol3'],
               ['v11\tv12\tv13'],
               ['v21\tv22\tv23'],
               ['v31\tv32\tv33'],
               ['v41\t\tv43']])])
```

上述示例都是指定file_name参数从文件中读取数据，事实上`pyexcel`的get系列函数还支持指定其他参数从嵌套列表、字典等类型变量甚至sql连接中获得数据，不在本文示例范围之内。

# 0x01 数据访问
## Book和Sheet
`pyexcel`中实现了`pyexcel.book.Book`和`pyexcel.book.Sheet`类型，和Excel表格文件的book、sheet概念相对应，可以通过上述get系列函数得到book/sheet对象，也可以通过`pyexcel.Book()`/`pyexcel.Sheet()`函数创建。

拿到book对象后，下一步就是访问book中的sheet。`pyexcel.book.Book`类对象可以按序号索引对应的sheet，也可以调用`sheet_by_index`和`sheet_by_name`方法获得指定的sheet内容，调用`sheet_names`方法返回book对象包含的所有sheet名称。
```python
In [45]: bk2[1]  # 通过序号访问book包含的sheet
Out[45]: Sheet2:

In [46]: bk2[0] # 通过序号访问book包含的sheet
Out[46]:
Sheet1:
+------+------+------+
| col1 | col2 | col3 |
+------+------+------+
| v11  | v12  | v13  |
+------+------+------+
| v21  | v22  | v23  |
+------+------+------+
| v31  | v32  | v33  |
+------+------+------+
| v41  |      | v43  |
+------+------+------+

In [47]: bk2.sheet_by_index(1) # 调用方法访问book包含的sheet，和序号索引等效
Out[47]: Sheet2:
```
`pyexcel.sheet.Sheet`类对象有个`texttable`属性，即表示文本，除了sheet名称，还有绘制表格边框的虚线符，直接**打印变量sh和打印sh.texttable效果相同**。
```python
In [62]: sh0 = bk2[0]

In [63]: sh0.texttable
Out[63]: 'Sheet1:\n+------+------+------+\n| col1 | col2 | col3 |\n+------+------+------+\n| v11  | v12  | v13  |\n+------+------+------+\n| v21  | v22  | v23  |\n+------+------+------+\n| v31  | v32  | v33  |\n+------+------+------+\n| v41  |      | v43  |\n+------+------+------+'

In [64]: print(sh0.texttable)
Sheet1:
+------+------+------+
| col1 | col2 | col3 |
+------+------+------+
| v11  | v12  | v13  |
+------+------+------+
| v21  | v22  | v23  |
+------+------+------+
| v31  | v32  | v33  |
+------+------+------+
| v41  |      | v43  |
+------+------+------+
```
此外，`pyexcel.sheet.Sheet`类还有几个十分有用的属性。
- content属性，与直接显示sheet相比，就少了第一行的sheet名称。
- csv属性，sheet数据的csv形式，没有表格框线。
- array属性，sheet数据的array形式（嵌套列表），与`get_array`函数返回结果一样。
- row/column属性，非常类似嵌套list，支持通过下标访问指定行/列，序号从0起始。

## 行和列
拿到`pyexcel.sheet.Sheet`对象后，除了用row/column属性获得所有行/列对象集合，进一步迭代遍历，也可以通过序号索引任意行/列，序号从0起始。当序号超出表格行/列范围时便抛出`IndexError`错误，可以用sheet对象的row_range/column_range方法查看行/列范围。`pyexcel.sheet.Sheet`对象的row_at/column_at方法，与直接通过row/column属性序号索引等效。
```python
In [57]: sh0.row
Out[57]: <pyexcel.internal.sheets.row.Row at 0x2cd5e69bcc0>

In [58]: sh0.column
Out[58]: <pyexcel.internal.sheets.column.Column at 0x2cd5e5ef6d8>

In [59]: r1 =sh0.row[1] # 序号索引

In [60]: r1
Out[60]: ['v11', 'v12', 'v13']

In [61]: c3 = sh0.column[3] #超出范围报错
---------------------------------------------------------------------------
IndexError                                Traceback (most recent call last)
<ipython-input-61-0f107a9c4c17> in <module>

In [62]: sh0.column_range() #查看列范围
Out[62]: range(0, 3)

In [63]: c2 = sh0.column[2] # 序号索引

In [64]: c2
Out[64]: ['col3', 'v13', 'v23', 'v33', 'v43']

In [65]: sh0.row_range() #查看行范围
Out[65]: range(0, 5)

In [66]: sh0.row_at(1)
Out[66]: ['v11', 'v12', 'v13']
```

## 单元格
`pyexcel.sheet.Sheet`对象支持二元序号索引任意单元格，或者用行/列名称替代序号（**请注意下述代码注释**）。也可以整体用Excel表单元格地址形式索引，无需任何转换。
```python
In [72]: c = sh0[3,1]  # 行列序号索引

In [73]: c
Out[73]: 'v32'

In [75]: sh0['B4'] # 单元格地址索引
Out[75]: 'v32'

# 设定第1行内容作为列名称，行序号0则定位到第1个数据行，行头被跳过，使用中注意！
In [78]: sh0.name_columns_by_row(0) 

In [79]: sh0[3,"col2"] #3定位到第4个数据行，对应表格的第5行
Out[79]: ''

In [80]: sh0[3,"col1"] #3定位到第4个数据行，对应表格的第5行
Out[80]: 'v41'
```

# 0x02 改写文件
改写文件包括改写变量值、将变量对象写入文件两个步骤，推荐通过`pyexcel.book.Sheet`或`pyexcel.book.Book`类对象进行。

## API
对于`pyexcel.book.Sheet`类对象，`row`、`column`属性支持像`list`那样增删改操作，而且均有save_as方法用于将对象写入文件。此外，`pyexcel`提供了save系列封装函数：`save_as`、`save_book_as`写入文件，指定目的文件时，使用的参数名称与get系列相比较多了"dest_"前缀。如，get系列用file_name指定数据文件来源，save系列用dest_file_name指定目的文件路径；get系列用delimiter参数指定csv分隔符，save系列用dest_delimiter指定写入到csv文件时使用的分隔符。

## 改写示例
`pyexcel.book.Sheet`类对象可以整行/列地增、删、改，也可以定位到具体单元格赋值，使用列表整行/列赋值时要注意元素个数与列/行数一致。对于`pyexcel.book.Book`类对象，既可以把整个book像操作列表那样整体扩展，也可以只索引出某些sheet再进行整体拼接、赋值。
```python
In [85]: sh0.column[1] = ['v12_new','v22_new',0.0,128]

In [86]: sh0
Out[86]:
Sheet1: #注意将首行当作列名称后，首行框线发生变化
+------+---------+------+
| col1 |  col2   | col3 |
+======+=========+======+
| v11  | v12_new | v13  |
+------+---------+------+
| v21  | v22_new | v23  |
+------+---------+------+
| v31  | 0.0     | v33  |
+------+---------+------+
| v41  | 128     | v43  |
+------+---------+------+

In [87]: sh0.row[-1] = [] #注意将首行当作列名称后，行索引将忽略首行，易造成混乱

In [88]: sh0
Out[88]:
Sheet1:
+------+---------+------+
| col1 |  col2   | col3 |
+======+=========+======+
| v11  | v12_new | v13  |
+------+---------+------+
| v21  | v22_new | v23  |
+------+---------+------+
| v31  | 0.0     | v33  |
+------+---------+------+
|      |         |      |
+------+---------+------+

In [101]: sh0.column['col3'] = [1,2,3,4] # 对列命名后，尽量使用字典的索引方式

In [103]: sh0[3,'col3'] = '4C' # 对第3数据行、'col3'列单元格赋值

In [104]: sh0['C4'] # 使用Excel单元格地址索引
Out[104]: '4C'

In [107]: sh0[3,2] # 使用行列序号索引
Out[107]: '4C'

In [118]: del sh0.column[2] # 删除指定列

In [121]: bk3 = bk2+bk2[-1] # 对book整体赋值

In [124]: bk3 = bk2+bk2  # 对book整理赋值

In [125]: bk3.sheet_names()
Out[125]: ['Sheet1', 'Sheet1_2', 'Sheet2', 'Sheet2_3', 'Sheet3', 'Sheet3_4']
```
`pyexcel.book.Sheet`还实现了表单转置（transpose）、截取（region）、剪切（cut）、粘贴（paste）、map应用（map）、行列筛选（filter）、格式化（format）等花式操作方法，本文均不一一论述。

`pyexcel`各种花式操作简化了代码逻辑，让用户专注于处理数据，但对于表格边框风格/字体/颜色等均不支持，对公式也不支持。

## 保存示例
写入文件，即可用`pyexcel.book.Sheet`或`pyexcel.book.Book`类对象的save_as方法，操作简单直接，操作Excel推荐首选。也可以用`pyexcel`包级别的save系列封装函数，更适合进行文件类型转换，同时还有一套同等的isave系列函数，主要的不同是只在写时读入变量，以提高效率。
以上这些save方法/函数会**根据目的文件扩展名自动判别格式类型**。
```python
In [128]: sh0.save_as("./工作簿sh0.xls") #xls文件中含1个sheet

In [130]: bk2.save_as("./工作簿bk2.xls")

In [131]: bk3.save_as("./工作簿bk3.xls")

In [132]: bk2.save_as("./工作簿bk2.xlsx") #需要 pyexcel-xlsx,pyexcel-xlsxw支持

# 直接进行类型换行，省去读、写的繁琐操作，需要 pyexcel-xlsx,pyexcel-xlsxw支持
In [136]: pe.save_as(file_name="./data.xls", dest_file_name="./data.xlsx")
```
`pyexcel.book.Sheet`或`pyexcel.book.Book`类还实现save_to系列函数，将对象写入数据库、ORM、内存等。

# 0x03 总结
- `pyexcel`包封装了get系列函数用于从文件中读取、转换数据，均能灵活支持多种读文件方式，对于操作Excel文件，推荐首选get_sheet函数。
- `pyexcel`包对不同格式文件的支持依赖不同的插件包。
- `pyexcel`包内部实现了`pyexcel.book.Sheet`或`pyexcel.book.Book`类型，与Excel文件的工作簿、表单概念对应，提供了多种灵活的数据访问、删改的方法，以及可视化的方法，让用户专注于处理数据，但很遗憾不支持Excel公式，也不支持图表、表格边框/字体/颜色等风格设定。
- `pyexcel.book.Sheet`或`pyexcel.book.Book`类型有save系列方法将对象变量写入文件、数据库、内存等，推荐首选；同时`pyexcel`包级别的save_as系列封装函数，对于转换文件类型十分方便；这些写入文件的方法/函数，根据目的文件扩展名自动判别格式类型。
- `pyexcel.cookbook`包封装了一些实用工具函数，如多类型文件合并、表格分拆;
- `pyexcel.book.Sheet`和`pyexcel.book.Book`等类的方法并未实现全，调用时有的会抛出错误，要注意这个大坑，本文在ipython环境写示例时错误的未给出，这也是提示符编号不连续的原因。

# 参考
* pyexcel Documentation，2018. [http://docs.pyexcel.org](http://docs.pyexcel.org)

# 更多阅读
* [Python操作Excel文件（0）：盘点](2019-12-03-Python操作Excel文件（0）：盘点.html)
* [Python项目如何合理组织规避import天坑](2019-08-15-Python项目如何合理组织规避import天坑.html)
* [Cygwin前传：从割据到互补](2019-02-05-Cygwin前传：从割据到互补.html)
* [Cygwin系列（一）：Cygwin是什么](2019-02-14-Cygwin系列（一）：Cygwin是什么.html)
* [Cygwin系列（九）：Cygwin学习路线](Cygwin系列（九）：Cygwin学习路线.html)
* [Linux Cygwin知识库（一）：一文搞清控制台、终端、shell概念](2019-04-04-Linux Cygwin知识库（一）：一文搞清控制台、终端、shell概念.html) 
* [Linux Cygwin知识库（二）：目录、文件及基本操作](2019-05-04-Linux Cygwin知识库（二）：目录、文件及基本操作.html)
* [专栏：伪码人We_Coder](https://zhuanlan.zhihu.com/c_1078678205585551360)

---
如本文对你有帮助，或内容引起极度舒适，欢迎分享转发与留言交流
![](https://pic3.zhimg.com/80/v2-2c41616595fb74ec5acfbabe9e0e125a_hd.jpg) 
![](https://pic2.zhimg.com/80/v2-2e15831707f8c58cbb6cfbf0df5a7b41_hd.jpg)
►本文为原创文章，如需转载请私信知乎账号silaoA或联系公众号伪码人（We_Coder）。

**都看这里了，不妨点个赞再走呗**