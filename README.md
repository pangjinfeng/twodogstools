# twodogstools

当前版本 V0.0.0（发布日期2020-10-10）

twodogstools 是基于常用分析功能集合的工具包。在日常分析中，平凡的实现一些功能却没有一个合适的方式进行调用，导致代码冗余，且嵌套入各类型脚本内统一更新不易，为解决此问题而诞生了twodogstools。

初版主要实现功能：

- 常用信息提取：carlisence、phone、idcard、ip、url、email
- 基于cpca进行地址信息提取，区分到省、市、区、街道
- 分组统计
- 基于已有词组进行词云提取
- 加载并处理文件
- 其他

## Installation

[pypi (点击下载)](https://pypi.org/manage/project/twodogstools/releases/)

```python
pip install twodogstools
```

## Introduction

```python
from twodogs import *
from sklearn import datasets
iris_datas = datasets.load_iris()
df = pd.DataFrame(iris_datas.data, columns=['SpealLength', 'Spealwidth', 'PetalLength', 'PetalLength'])
df.head(1)
```

![iris](https://github.com/pangjinfeng/twodogstools/blob/main/image/iris_datas.png)

样例数据使用常用的数据集iris, 目前主要为三大模块函数：

- pjftools    常用工具
- load_data    加载指定格式数据
- re_    识别信息

### pjftools

#### 1、value_count

主要实现分组统计，对col_key进行分组，计算col_value出现的总次数、不同数、出现详情，此功能在数值类型上效果不明显，但对于文本类型的数据进行快速把控各分组数据概况具有显著意义。

- data: dataframe, 需要进行处理的dataframe
- col_key: 索引列，对其进行分组聚合
- col_value: 值列，对其进行计算

```python
pjftools().value_count(data=df, col_key='SpealLength', col_value='Spealwidth')
```

![value_counts](https://github.com/pangjinfeng/twodogstools/blob/main/image/value_counts.png)

#### 2、extract_address

基于cpca对于字符串类型地址进行提取，精确到区，目前支持 list 与 DataFrame 作为输入，输出为DataFrame,感谢DQinYuan编写的cpca, [项目地址](https://github.com/DQinYuan/chinese_province_city_area_mapper)

- data: list or DataFrame, 存储地址的list或者DataFrame
- column: 地址列所在的DataFrame列，当type(data)==list时，自动转为DataFrame，列名默认为0
- index: 指定自定义的list or DataFrame的index值
- cut: 是否使用分词匹配模式，默认是True，会提高处理速度，若指定False，则会采用“全文匹配的模型”，该模型下的精度会高些，但处理速度会慢些
- lookahead: 默认为8个字符，可以理解为窗口大小
- pos_sensitive: 默认为False，改为True时，则在输出的DataFrame中将新增三列，分别表示抽取省、市、地区的起始位置，若值为-1，表示推断出来的
- open_warning: 是否显示警告信息，默认True，建议打开（当发现重名区并且不知道将其映射到哪一个市时，会将其加入警告信息并显示，打开次功能可以帮助解决数据集中的重名区问题）

```python
location_str = ['徐汇区虹漕路461号58号楼5楼', '泉州市洛江区万安塘西工业区', '朝阳区北苑华贸城']
pjftools.extract_address(location_str)
```

![address_1](https://github.com/pangjinfeng/twodogstools/blob/main/image/address_1.png)

```python
extract_address(pd.DataFrame(location_str, columns=['address']), column='address')
```

![address_2](https://github.com/pangjinfeng/twodogstools/blob/main/image/address_2.png)

#### 3、mergeplus

merge、join都只支持两个DataFrame连接，而append与concat都只是纯粹增添数据，故而有了基于merge的mergeplus，使用reduce进行批量合并。

- data: 需要合并的DataFrame存储到一个list中，[DataFrame, DataFrame, DataFrame]
- how:  连接方式，默认为'inner'
- left_on: 左连接列名，默认为None
- right_on: 右连接列名，默认为None,
- left_index: 左索引连接，默认为False
- right_index: 右索引连接，默认为False
- sort: 排序，默认为False,
- suffixes:  连接后重复列重命名，默认为('_x','_y')

```python
df1.head(1)
```

![mergeplus_1](https://github.com/pangjinfeng/twodogstools/blob/main/image/mergeplus_1.png)

```python
df2.head(2)
```

![mergeplus_2](https://github.com/pangjinfeng/twodogstools/blob/main/image/mergeplus_2.png)

```python
df3.head(1)
```

![mergeplus_3](https://github.com/pangjinfeng/twodogstools/blob/main/image/mergeplus_3.png)

```python
pjftools.mergeplus([df1, df2, df3], left_on='id', right_on='id')
```

![mergeplus_4](https://github.com/pangjinfeng/twodogstools/blob/main/image/mergeplus_4.png)

#### 4、add_split

基于分隔符切分数据并连接回原来的数据，可增加行或列

- df: 需要处理的DataFrame
- column: 需要处理的列名
- sep: 分隔符，默认为 ','
- axis: 增加行或列，0:行,1:列，默认为0

```python
df.head(2)
```

![add_split_1](https://github.com/pangjinfeng/twodogstools/blob/main/image/add_split_1.png)

```python
pjftools.add_split(df, column='content', sep=',', axis=1)
```

![add_split_2](https://github.com/pangjinfeng/twodogstools/blob/main/image/add_split_2.png)

```python
pjftools.add_split(df, column='content', sep=',', axis=0)
```

![add_split_3](https://github.com/pangjinfeng/twodogstools/blob/main/image/add_split_3.png)

#### 5、textrank

基于textrank对文本进行关键词信息提取，在长文本中快速了解大致内容具有重要意义。

- sentence：需要分析的文本
- topK：提取的关键词数，默认为20
- withWeight：是否显示权重，默认为False
- allowPOS：提取的关键词词性，默认为('ns','n','vn','v'),
- withFlag：是否显示词性，默认为False

```python
text = '【#小伙辞去高薪工作放弃读研支教11年#】凌晨4点起床，挑水、做饭、去上课；晚上批改作业，打着手电家访…2009年，浙江小伙杨明辞去收入过万元的工作来到贵州支教，这样的日子一过就是十多年。2012年他曾考研成功，却因孩子们哭着挽留而坚持了下来。11年来，杨明瘦了，头发白了，他却说一切都值得'
pjftools.textrank(sentence=text, withWeight=True,, topK=3)
```

[('工作',1.0), ('支教',0.8561024152117869), ('挽留',0.8238886839907752)]

#### 6、connectfile

对指定目录下的文件进行批量合并，核心函数为 shutil.copyfileobj 不区分文件类型，常用于多层嵌套文本内容合并，利用tqdm进行进度展示。

- filedir：需要合并的文件夹目录，默认为 './'
- save：合并后的文件路径，默认为 './allfile'
- pattern：筛选文件名，输入正则表达式，默认为全量合并

```python
pjftools.connectfile(filedir='./testfilepath/', save='./newfile')
```

![connetfile](https://github.com/pangjinfeng/twodogstools/blob/main/image/connetfile.png)

#### 7、filedir_to_df

对指定目录下的指定文件类型进行批量合并，并生成 DataFrame，新增sourcefile列存储原始文件路径

- filedir：需要合并的文件夹路径，默认为 './'
- filetype：需要合并的文件类型，默认为 ’csv‘
- sep：分隔符，默认为 '\t'
- sheet_name：sheet名称，录入文件为xlsx使用，默认为 0

```python
pjftools.filedir_to_df('./testfilepath/', filetype='xlsx')
```

![filedir_to_df_1](https://github.com/pangjinfeng/twodogstools/blob/main/image/filedir_to_df_1.png)

![filedir_to_df_2](https://github.com/pangjinfeng/twodogstools/blob/main/image/filedir_to_df_2.png)

#### 8、transip

intip与dotip互项转换工具

- data: str or int，需要转换的ip
- transtype: str, 输入数据的数据类型，目前仅支持['intip', 'dotip']，默认为 ’intip'

```python
pjftools.transip('183.39.22.1',transtype='dotip')
pjftools.transip('3072792065',transtype='intip')
pjftools.transip(3072792065)
```

![transip](https://github.com/pangjinfeng/twodogstools/blob/main/image/transip.png)

#### 9、ip2address

通过API接口ip查询地址，并区分是否为境外

- ip: str，需要查询的ip
- api: str，默认为纯真库api='http://ipaddr.cz88.net/data.php?ip='

```python
pjftools.ip2address('58.217.112.26')
```

{'ip': '58.217.112.26', 'address': '江苏省镇江市',  'operators': '电信', 'abroad': False}

#### 10、phone2message

基于公共手机号，短信验证码接收，短信存在他人登录状况，仅做测试交流使用，严禁非法调用，未经本人许可，不得转载与使用

```python
# 加载数据,只保留1小时以内的数据
pm = phone2message()
```

********    禁止非法使用,此工具仅为测试交流使用   ********

```python
# 查看可用手机号活跃状况
pm.phoneinfo.head(n=2) # 更换 n 的数值可以获取到更多的可用手机号，已按照活跃降序
```

![phone2message_1.png](https://github.com/pangjinfeng/twodogstools/blob/main/image/phone2message_1.png)

```python
# 提取短信信息
phone = 17085464353  # 选择上述的其中一个手机号，用来获取短信
message = pm.extract_message(phone=phone)
message
```

![phone2message_2.png](https://github.com/pangjinfeng/twodogstools/blob/main/image/phone2message_2.png)

```python
message.img[0]  # 如果验证码为转换成功，则存为图片形式 [n] n为索引，即第几行，0开始计数
```
验证码地址为：
http://8.210.115.9/img.php?num=OTU1ODQw&x=MzM2NzkxMjI5NzI=&s=39196045440






