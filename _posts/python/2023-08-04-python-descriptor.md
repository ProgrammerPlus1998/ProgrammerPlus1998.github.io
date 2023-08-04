---
layout:       post
title:        "解密Python的神秘之门，深入理解描述符"
subtitle:     "Decipher the mysterious door of Python and deeply understand descriptors"
header-img:   "img/in-post/head/2023-07-31-python-global-interpretation-lock.jpg"
date:         2023-08-03 00:00:00
author:       "Xic"
tags:
    - Python
    - 描述符
    - python之巅
---
# 引言
描述符是Python的一个强大特性，但它对许多开发者来说仍然是一片未知的领域。它可以改变Python的属性访问行为，强大到可以做出你无法想象的事情。本文将带你走进这扇神秘之门，深入探索和理解Python描述符。

# 一、描述符协议：描述符的基础
描述符是实现了特定协议的类，这个协议包括`__get__`、`__set__` 和 `__delete__`方法。当一个类的对象被用作另一个类的类属性时，我们可以通过实现这些方法来精细地控制属性的访问。例如，假设我们有一个描述符类：

```python
class MyDescriptor:
    def __get__(self, instance, owner):
        print("Getting...")
        
    def __set__(self, instance, value):
        print("Setting...")
        
    def __delete__(self, instance):
        print("Deleting...")
```
我们可以将其用作另一个类的属性：
```python
class MyClass:
    attr = MyDescriptor()
```

然后，当我们访问、设置或删除MyClass的attr属性时，将调用描述符的`__get__`、`__set__`或`__delete__`方法。

# 二、深入了解描述符类型
描述符分为数据描述符和非数据描述符。数据描述符同时定义了`__get__`和`__set__`，而非数据描述符只定义了`__get__`。这对于解释Python如何解析属性访问非常重要。例如：

```python
class DataDescriptor:
    def __get__(self, instance, owner):
        print("DataDescriptor __get__")
        
    def __set__(self, instance, value):
        print("DataDescriptor __set__")

class NonDataDescriptor:
    def __get__(self, instance, owner):
        print("NonDataDescriptor __get__")

class MyClass:
    data_descriptor = DataDescriptor()
    non_data_descriptor = NonDataDescriptor()

obj = MyClass()
obj.data_descriptor
obj.data_descriptor = 1
obj.non_data_descriptor
obj.non_data_descriptor = 1
```

这段代码将展示出数据描述符和非数据描述符在属性访问中的不同行为。

# 三、描述符的实用性
描述符可以解决多种常见的编程问题。例如，你可以使用描述符来实现惰性属性，即只在首次访问时计算其值。描述符还常常用于实现类型检查和其他形式的验证。例如：

```python
class Integer:
    def __init__(self, name):
        self.name = name

    def __get__(self, instance, owner):
        return instance.__dict__[self.name]

    def __set__(self, instance, value):
        if not isinstance(value, int):
            raise TypeError("Expected an int")
        instance.__dict__[self.name] = value

class MyClass:
    number = Integer('number')

# 创建一个MyClass实例
obj = MyClass()
# 设置number属性
obj.number = "godev.me"
# 获取number属性
print(obj.number)
```

在这个例子中，我们使用描述符实现了类型检查，当我们试图将非整数赋值给MyClass的number属性时，会引发一个类型错误。

# 四、描述符的高级应用
描述符不仅仅可以用于管理单个属性。例如，通过使用描述符，你可以实现Python的对象关系映射，或ORM，这是一种把数据库表映射到Python对象的技术。

描述符可以配合Python装饰器来使用，实现更为强大的功能，例如 @property、@classmethod、@staticmethod等都是基于描述符协议来实现的。

# 五、探讨描述符的陷阱和限制
尽管描述符很强大，但使用它们也需要小心。例如，你需要注意防止实例属性覆盖描述符。一个常见的误区是直接在实例的字典中查找属性，而不是使用内置的getattr和setattr函数。这样做可能会跳过描述符并直接访问实例字典，这可能不是你想要的结果。

# 六、描述符在Python内置库中的应用实例
许多Python内置库都使用了描述符。例如，在Django ORM中，描述符被用来实现了数据库模型字段。在Flask中，描述符被用来处理路由等。

# 七、描述符的实战应用
接下来，让我们通过几个具体的例子来展示描述符的威力。例如，我们可以使用描述符来实现一个简单的类型系统：

```python
class Typed:
    def __init__(self, name, expected_type):
        self.name = name
        self.expected_type = expected_type

    def __get__(self, instance, owner):
        return instance.__dict__[self.name]

    def __set__(self, instance, value):
        if not isinstance(value, self.expected_type):
            raise TypeError(f"Expected {self.expected_type}")
        instance.__dict__[self.name] = value

class MyClass:
    number = Typed('number', int)
    text = Typed('text', str)

# 创建一个MyClass实例
obj = MyClass()
# 设置属性
obj.number = 18
obj.text = 'godev.me'
# 获取属性
print(obj.number)
print(obj.text)
```

# 结尾
描述符是Python的一项强大特性，掌握它可以极大提升你的编程能力。通过理解和使用描述符，我们可以获得更多的编程能力和灵活性。希望通过这篇文章，你能对描述符有更深入的理解，并鼓励你去更多的实践和探索。