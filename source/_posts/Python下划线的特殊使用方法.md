---
title: Python下划线的特殊使用方法
date: 2018-08-11 15:29:01
tags:
categories:
- 技术笔记
---
这里的“特殊”指的是下划线出现在变量名开头或结尾的情况。
<!--more-->

### 单下划线
1. `_single_leading_underscore`： **以一个下划线起始**。表示该变量为模块的内部变量，不要从外部进行访问。这是程序员之间的约定，通过`import a_module`和`a_module._single_leading_underscore`依然可以访问到。但使用`from a_module import *`将不会访问到内部变量。
2. `single_trailing_underscore_`：**以一个下划线结尾**。用于避免和Python的关键词发生冲突，是使用惯例，不是规定，比如`Tkinter.Toplevel(master, class_='ClassName')`。

### 双下划线
1. `__double_leading_underscore`：**以两个下划线起始**。用于命名类属性，避免和子类的变量名发生冲突。比如在`FooBar`类中有一个`__boo`属性，从外部进行访问需要带上类名，比如`_FooBar__boo`。
2. `__double_leading_and_trailing_underscore__`：**前后分别有两个下划线**。属于 Python 的内置属性或对象，比如`__init__`，`__file__`和`__import__`。使用已有的名称即可，不要用这个规则发明新的名称。

### 参考资料
> https://www.python.org/dev/peps/pep-0008/?#naming-conventions