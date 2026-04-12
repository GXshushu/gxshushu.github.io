---
title: Python 学习笔记 - type类和MetaClass
date: 2026-04-12 15:54:00 +0800
categories: [Python]
tags: [学习]
pin: false
author: C9xx

toc: true
comments: true
typora-root-url: ../../gxshushu.github.io
math: false
mermaid: true

---

# type类

- type是一个元类(metaclass)，用于创建和管理类。
- 任何class在内存中都是一个type类的实例对象。
- Python使用type类来创建其他class。
  - type(class_name, parents, class_dict)
    - class_name: 新创建的类的名称
    - parents: 新创建的类的父类元组
    - class_dict: 新创建的类的属性字典
- 理论上可以用type动态创建class。

```python
class MyClass:
    pass

print(type(MyClass))  # 输出：<class 'type'>
print(isinstance(MyClass, type))  # 输出：True
```
动态创建Class，并设置成员方法：

![alt text](/assets/blog_res/2026-04-12-pythom-study-2.assets/type-make-class.png)

# metaclass

- 一个metaclass就是一个用来创建其他class的类
- type就是所有class默认的metaclass
- 可以在定义class的时候制定metaclass
  - 例如：
  ```python
  class MyClass(object, metaclass=type):
      pass
  ```

## 自定义metaclass
实际上是继承type类，重写__new__方法，来做一些修改。metaclass创建类的时候会执行__new__方法。

![自定义metaclass](/assets/blog_res/2026-04-12-pythom-study-2.assets/metaclass.png)

在这个例子中，创建了一个名为Human的metaclass，继承自type类，重写了__new__方法，来创建Human类的实例对象，我们在__new__方法中，给类添加了一个名为freedom的布尔属性，如果我们使用这个metaclass去构建不同的类，他们都会带有一个freedom的类属性。

- __new__方法的参数：
  - mcs: 就是自己这个metaclass
  - *args: 位置参数
    - classname: 新创建的类的名称
    - parents: 新创建的类的父类元组
    - class_dict: 新创建的类的属性字典

![metaclass_new_args](/assets/blog_res/2026-04-12-pythom-study-2.assets/metaclass_new_args.png)

自定义一个带参数的metaclass：
![metaclass_with_kargs](/assets/blog_res/2026-04-12-pythom-study-2.assets/metaclass_with_kargs.png)
