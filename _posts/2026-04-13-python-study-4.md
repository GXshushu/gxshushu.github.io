---
title: Python 学习笔记 - 变量范围与nonlocal
date: 2026-04-13 14:21:00 +0800
categories: [Python]
tags: [Python 学习]
pin: false
author: C9xx

toc: true
comments: true
typora-root-url: ../../gxshushu.github.io
math: false
mermaid: true

---

# 变量范围
在python中，变量的查找范围是从小到大的，本地范围（函数内）->模块范围->Built-in范围。
![alt text](/assets/blog_res/2026-04-13-python-study-4.assets/变量范围.png)
python在**编译期**确定变量的范围。
python虽然是解释型语言，直接执行py文件，但是也是分为编译期和运行期的。

看看下面几道题

题目一:

```python
count = 10
print(count)

def greeting():
    print(count)

greeting()
```
上面这个题目中，模块范围中有个count变量，第一个print是10肯定没问题

第二个print在函数greeting中，执行到函数里的print的时候

1. 首先寻找本地范围（函数）的count是否有count的定义
2. 没有就去找模块范围，存在count，所以也能正常打印10。


---
题目二:
```python
count = 10
print(count) # 还是10

def greeting():
    count = 20
    print(count)

greeting()
print(count)
```
题目二进行了小小的扩展
- greeting的本地范围中定义了变量count，那么对于greeting函数来说，会先去寻找本地范围是否存在count，发现有，直接使用本地范围的count。
- 由于greeting里的count与模块范围的count是不同的变量，所以不会影响模块范围的count，第三个print还是10。

---
题目三:
```python
count = 10
print(count) # 还是10

def greeting(flag: bool):
    if flag:
        count = 20

    print(count)

greeting(True)
```

对于题目三的改编，可能有人会先入为主的认为，count在if代码块里面，print在外面，所以应该不在内部count的作用域里面。

但python和其他语言不同，没有按代码块来定义作用域的说法，count仍然在greeting的本地范围里面，print仍然能够调用到。所以第二个print还是20。

那如果flag是false呢↓

---
题目四:
```python
count = 10
print(count) # 还是10

def greeting(flag: bool):
    if flag:
        count = 20

    print(count)

greeting(False)
```

对于题目四，可能很多人认为，如果flag是False，那么进不去if块，执行不到count = 20，那么就定义不到count了，那第二个print调用的还是模块范围的count，也就是10。

但是实际结果就是第二个print报错了。

```
UnboundLocalError: local variable 'count' referenced before assignment
```

这就是因为一开始说的，python是先编译后运行。在编译的过程中，编译器不知道这个if语句是否会执行，但已经确定了本地范围有一个count变量，那么执行时候这里的print就会去找本地范围的那个变量了，但是由于执行流程没有经过count定义语句，所以报错了未声明。

# global关键字

从上面可以看出来函数里面只要有对count的赋值，那么函数里的count就只能调到函数本地范围的count，那如果函数中想要调用模块范围的count就要使用global关键字。

使用global关键字之后，函数里的count就会直接使用模块范围的变量count。

如果想要引用别的文件（模块）中的变量，就要使用import关键字，直接导入别的模块，就相当于合并了模块的变量范围。