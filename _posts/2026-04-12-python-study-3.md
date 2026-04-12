---
title: Python 学习笔记 - dataclass
date: 2026-04-12 16:27:00 +0800
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

dataclass有点像实体类的概念。通过装饰器@dataclass修饰一个类，将其转化为一个数据类。

```python
@dataclass
class Student:
    name: str
    age: int
```
使用dataclass修饰后，自动实现\_\_init\_\_、\_\_repr\_\_、\_\_eq\_\_、\_\_hash\_\_等方法。

如下图例子，在定义了类属性后，自动定义对象实例属性，并根据名字和顺序编写\_\_init\_\_方法，并且eq方法也会自动按照对比成员属性值来判断是否相等来实现。由于是数据类，所以eq方法的实现是基于成员属性值的比较，而不是基于对象实例的比较，如果想要基于实例的话，可以设置一个id字段。

![alt text](/assets/blog_res/2026-04-12-python-study-3.assets/dataclass-1.png)

## 缺省值
对象成员属性可以直接定义缺省值，在构造的时候不带age参数，也会默认使用缺省值20。

```python
@dataclass
class Student:
    name: str
    age: int = 20
```

![alt text](/assets/blog_res/2026-04-12-python-study-3.assets/缺省值.png)

## 不可变对象
通过给@dataclass装饰器添加frozen=True参数，可以指定类为不可变对象类。

```python
@dataclass(frozen=True)
class Student:
    name: str
    age: int

student = Student("C9xx", 20)
student.age = 21 # 会产生异常
```
## 使用field定义属性行为

field可以指定成员属性在dataclass的特征，例如特定构造函数，是否参与eq方法的比较，hash方法的计算，对象打印是否打印该属性等等。

![alt text](/assets/blog_res/2026-04-12-python-study-3.assets/field.png)

![alt text](/assets/blog_res/2026-04-12-python-study-3.assets/field-2.png)

 ## \_\_post_init__

\_\_post_init\_\_方法可以在对象实例化后，对对象实例进行初始化操作。

```python
@dataclass
class Student:
    name: str
    age: int
    independent: bool = field(default=False, init=False, repr=False)

    def __post_init__(self):
        self.independent = self.age >= 19

student = Student("C9xx", 20)
print(student.independent) # True

student = Student("C9xx", 18)
print(student.independent) # False
```
## 排序
通过指定order=True参数，可以指定类为可排序对象类。

```python
@dataclass(order=True)
class Student:
    name: str
    age: int
```

指定可排序之后，按照首个成员属性的值进行排序，当第一个属性相同，顺延用第二个属性排序。
也可以使用field指定排序的键，通过post init方法来给排序键赋值。

```python
@dataclass(order=True)
class Student:
    sorted_key: str = field(init=False, repr=False)
    name: str
    age: int

    def __post_init__(self):
        self.sorted_key = self.age
```

但其实更方便的排序方式是使用sort函数的key参数，指定排序的键，这就与order参数无关了。

```python
import operator

students.sort(key=operator.attrgetter("age"))
```

## 结语
pydantic的BaseModel感觉和dataclass也差不太多，而且好像BaseModel用得更多一些。