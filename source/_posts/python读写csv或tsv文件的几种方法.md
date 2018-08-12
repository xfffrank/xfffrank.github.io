---
title: python读写csv或tsv文件的几种方法
date: 2018-07-09 16:57:36
tags:
categories:
- 技术笔记
---
这两天需要将一批数据存储为tsv格式，网上搜了一圈，发现大多文章介绍的是csv文件的读写方法，后来知道可以直接将处理csv文件的方法应用于tsv。因为两种文件的唯一区别在于分隔符，csv是逗号，tsv是制表符。

*说明：将以下代码中的`delimiter`，`sep`参数去掉，即可用于处理 csv 文件，逗号是默认分隔符。*

### 写文件
使用 csv 库。
```python
with open('file.tsv', 'w') as f:
    tsv_w = csv.writer(f, delimiter='\t')
    tsv_w.writerow(['id', 'name', 'score'])  # 单行写入
    tsv_w.writerows([[1, 'Frank', 99], [2, 'John', 70]])  # 多行写入
```

### 读文件

1. 使用 csv 库
```python
import csv

with open('file.tsv') as f:
    tsvreader = csv.reader(f, delimiter='\t')
    for line in tsvreader:
        print(line)
```

2. 使用 DataFrame
```python
from pandas import DataFrame

df = DataFrame.from_csv('file.tsv', sep='\t')
print(df)
```

3. 使用 pandas 库的 csv 读取函数
```python
import pandas as pd

print(pd.read_csv('file.tsv', delimiter='\t'))
```