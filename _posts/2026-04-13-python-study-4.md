---
title: Python 学习笔记 - 变量范围、global与nonlocal
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

# built-in范围

python的所有内建函数和模块都在built-in范围中。比如print，type等等。

# nonlocal关键字

首先需要先拓展一下nonlocal范围。对于前面所讲的变量作用范围，在存在嵌套函数的情况下，表现为如下层次。
![nonlocal范围](/assets/blog_res/2026-04-13-python-study-4.assets/nonlocal.png)

在嵌套函数的情况下

```python
message = "module"
def outer():
    message = "outer"
    def inner():
        message = "inner"

        print(message) # inner
    inner()
    print(message) # outer

outer()
print(message) # module
```

对于print内部的message变量来说，inner内部是local范围，outer与inner之间是nonlocal范围，module字符串是模块范围。

print会依照就近原则获取到`message = "inner"`这个变量。使用nonlocal关键字之后，inner函数里的message就会直接使用outer函数的message变量。这时outer函数里的message变量也会被修改为"inner"。所以第二个print也会打印"inner"。

```python
message = "module"
def outer():
    message = "outer"
    def inner():
        nonlocal message
        message = "inner"

        print(message) # inner
    inner()
    print(message) # inner

outer()
print(message) # module
```

如果outer中不存在message的声明，inner使用nonlocal关键字定义的message变量，就会直接报错。因为模块的message要用global关键字来声明引用。

```python
message = "module"
def outer():
    # message = "outer"
    def inner():
        nonlocal message # error 
        message = "inner"

        print(message) 
    inner()
    print(message) 

outer()
print(message) 
```

```
    nonlocal message
    ^
SyntaxError: no binding for nonlocal 'message' found
```

如果是多层嵌套的情况，最内层的函数通过nonlocal获取变量的时候，会从内到外逐层查找函数范围，直到找到变量为止。以下三个例子可以很好的证明这一点：

```python
message = "module"
def outer():
    message = "outer"
    def inner_1():
        def inner_2():
            nonlocal message
            print(message) # outer
        inner_2()
        print(message) # outer
    inner_1()
    print(message) # outer

outer()
print(message) # module
```

```python
message = "module"
def outer():
    message = "outer"
    def inner_1():
        message = "inner1"
        def inner_2():
            nonlocal message
            print(message) # inner1
        inner_2()
        print(message) # inner1
    inner_1()
    print(message) # outer

outer()
print(message) # module
```

```python
message = "module"
def outer():
    message = "outer"
    def inner_1():
        message = "inner1"
        def inner_2():
            nonlocal message
            message = "inner2"
            print(message) # inner2
        inner_2()
        print(message) # inner2
    inner_1()
    print(message) # outer

outer()
print(message) # module
```
