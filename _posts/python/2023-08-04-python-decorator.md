---
layout:       post
title:        "Python装饰器用法大全：初学者到专家的完全指南"
subtitle:     "Python Decorator Usage Encyclopedia: Complete Guide from Beginner to Expert"
header-img:   "img/in-post/head/2023-07-31-python-global-interpretation-lock.jpg"
date:         2023-08-04 00:00:00
author:       "Xic"
tags:
    - Python
    - 装饰器
    - 生成器
    - 异步编程
    - python之巅
---


# 引言
在我们探索Python的世界时，经常会遇到装饰器这个概念。装饰器是一种强大的编程工具，它允许我们以简洁、优雅的方式修改或增强函数的行为。如果你想成为一个高效的Python程序员，理解和掌握装饰器是非常必要的。那么，什么是装饰器呢？

# 一、 Python装饰器基础
装饰器其实就是一个Python函数或类，它能够“装饰”其他函数或类，为其添加额外的功能，比如日志记录、性能测试、事务处理、缓存、权限校验等。最美妙的部分在于，装饰器使你能够在不修改原函数或类的源代码的情况下增加这些功能，这遵循了软件工程中的开闭原则，即对扩展开放，对修改关闭。

本文将详尽地解析Python装饰器的种类和用法，无论你是刚开始接触Python的新手，还是已经有一定经验的专业人士，都能在这篇文章中获得宝贵的知识。我们将从装饰器的基本理论开始讲解，然后逐步深入到更复杂的内容，包括函数装饰器、类装饰器，以及如何使用Python内建的装饰器。同时，我们也将探讨一些常见的装饰器设计模式，通过实例来展示如何在实际项目中应用装饰器。最后，我们会帮助你避开使用装饰器时的常见陷阱。

简单函数装饰器的编写和使用：

```python
def my_decorator(func):
    def wrapper():
        print("Before function execution")
          func()  # 执行被装饰的函数
        print("After function execution")
    return wrapper

@my_decorator
def greet():
    print("Hello, world!")

greet()
```
# 二、 进阶装饰器
在我们了解了函数装饰器的基本概念之后，现在我们将深入探讨更复杂的装饰器类型：类装饰器和带参数的装饰器。此外，我们还会学习如何在一个函数上堆叠多个装饰器。

## 1. 类装饰器

类装饰器和函数装饰器类似，只不过它接受的是一个类而不是函数。类装饰器通常用于为类添加新的属性或方法，或者修改类的行为。
```python
def class_decorator(cls):
    class Wrapper:
        def __init__(self, *args):
            self.wrapped = cls(*args)
        
        def greet(self):
            return "Hello, " + self.wrapped.greet()

    return Wrapper

@class_decorator
class Greeter:
    def __init__(self, name):
        self.name = name

    def greet(self):
        return self.name

g = Greeter("world")
print(g.greet())
```

## 2. 有参装饰器

有时，你可能希望你的装饰器可以接受参数，这就需要使用带参数的装饰器。带参数的装饰器其实是一个返回装饰器的函数。
```python
def repeat(num_times):
    def decorator(func):
        def wrapper(*args, **kwargs):
            for _ in range(num_times):
                func(*args, **kwargs)
        return wrapper
    return decorator

@repeat(3)
def greet(name):
    print(f"Hello {name}")

greet("world")
```
调用 `greet("world")` 会打印三次"Hello world"，这表明 `repeat` 装饰器正确地接收了参数并执行了相应次数的函数。

## 3. 在一个函数上堆叠多个装饰器
Python允许你在一个函数上使用多个装饰器，它们会按照定义的顺序从内到外依次执行。
```python
def decorator1(func):
    def wrapper():
        print("Before decorator 1")
        func()
        print("After decorator 1")
    return wrapper

def decorator2(func):
    def wrapper():
        print("Before decorator 2")
        func()
        print("After decorator 2")
    return wrapper

@decorator1
@decorator2
def greet():
    print("Hello, world!")

greet()
```
运行上面的代码，你将会看到以下的输出：
```mathematica
Before decorator 1
Before decorator 2
Hello, world!
After decorator 2
After decorator 1
```
从输出中我们可以看到，`decorator1`（在外部）的“前”部分先执行，然后是`decorator2`（在内部）的“前”部分，然后是修饰的函数，接着是`decorator2`的“后”部分，最后是`decorator1`的“后”部分。这就清楚地展示了装饰器在修饰的函数前后添加代码的顺序，以及当堆叠多个装饰器时，装饰器的执行顺序。

# 三、 Python内建装饰器
Python内建装饰器包括@staticmethod、@classmethod和@property，我们会介绍它们的使用和区别。
## 1. @staticmethod
`@staticmethod` 是一个内建装饰器，它可以把类中的方法转化为静态方法。静态方法不会接收隐式的第一个参数，无论是从类还是从类的实例调用，其行为都与普通的函数没有区别。
```python
class MyClass:
    @staticmethod
    def my_method(x):
        return x * 2

print(MyClass.my_method(2))  # 输出: 4
```
## 2.@classmethod
`@classmethod`也是一个内建装饰器，它可以把类中的方法转化为类方法。类方法的第一个参数是类本身，通常命名为`cls`，可以被类及其所有实例调用，常常用于创建工厂方法。
```python
class Person:
    def __init__(self, name):
        self.name = name

    def speak(self):
        return f"Hello, my name is {self.name}."

    @classmethod
    def from_dict(cls, data):
        return cls(data['name'])


# 通常的构造方式
person1 = Person('Alice')
print(person1.speak())  # 输出: Hello, my name is Alice.

# 通过工厂方法从字典创建实例
person_data = {'name': 'Bob'}
person2 = Person.from_dict(person_data)
print(person2.speak())  # 输出: Hello, my name is Xic.
```
在上面的代码中，我们定义了一个类`Person`，并通过类方法`from_dict`提供了另一种构造实例的方式。我们可以通过一个字典来创建一个`Person`的实例，而不是直接调用`Person`的构造函数。这对于处理各种数据源（比如从JSON或数据库读取的数据）创建实例非常有用。

## 3. @property
`@property`是一个内建装饰器，它可以将类中的方法转化为属性，这样你就可以用属性的方式来访问这个方法，而不需要显式地调用它。同时，`@property`也提供了一种优雅的方式来实现getter和setter方法。
```python
class MyClass:
    def __init__(self, x):
        self._x = x

    @property
    def x(self):
        return self._x

    @x.setter
    def x(self, value):
        if value < 0:
            raise ValueError("x must be positive")
        self._x = value

instance = MyClass(2)
print(instance.x)  # 输出: 2
instance.x = 3
print(instance.x)  # 输出: 3
```
在上面的代码中，x被定义为一个属性，你可以像访问属性一样来访问x方法，同时，你也可以设置x的值，如果设置的值不符合条件（比如小于0），就会抛出一个异常。

# 四、 常见装饰器设计模式
这部分介绍了一些常见的装饰器设计模式，例如缓存装饰器、计时装饰器和日志装饰器。

计时装饰器：
```python
import time

def timing_decorator(func):
    def wrapper():
        start_time = time.time()
        func()
        end_time = time.time()
        print(f"Execution time: {end_time - start_time} seconds")
    return wrapper

@timing_decorator
def my_function():
    time.sleep(2)

my_function()
```

# 五、 避开装饰器常见的陷阱和错误
## 1. 忘记调用装饰器函数

装饰器是一种特殊的函数，它返回一个新的函数来替换旧的函数。然而，有时我们可能会忘记在装饰器后面加上括号，导致装饰器函数本身被用作结果，而不是调用装饰器函数返回的函数。
```python
@my_decorator  # 正确
def my_func():
    pass

@my_decorator()  # 如果装饰器需要参数，这是正确的。如果不需要，就是错误的。
def my_func():
    pass
```
## 2. 忘记使用@functools.wraps

在定义装饰器时，通常会在内部包装函数上使用@functools.wraps装饰器，以保留原函数的名称、文档字符串等属性。如果忘记使用，可能会导致一些不预期的结果，比如函数的名称不正确。
```python
import functools

def my_decorator(func):
    @functools.wraps(func)  # 保留原函数的名称和文档字符串
    def wrapper(*args, **kwargs):
        # ...
        pass
    return wrapper
```
## 3. 在装饰器中修改函数的参数

装饰器用于修改函数的行为，但是最好避免在装饰器中修改函数的参数。这会使代码更难理解，增加出错的可能性。

## 4. 对装饰器的顺序理解不正确

当一个函数有多个装饰器时，装饰器的顺序很重要。离函数最近的装饰器最先应用，离函数最远的装饰器最后应用。这可能与你的直觉相反。

```python
@decorator1
@decorator2
@decorator3
def my_func():
    pass
```
在上面的代码中，decorator3最先应用，decorator1最后应用。

## 5. 使用装饰器过度

虽然装饰器是一个强大的工具，但是使用过度会使代码变得难以理解。最好在真正需要的时候才使用装饰器，而且使用后要确保代码的可读性和可维护性。
# 结语
学习Python装饰器是一个重要的步骤，但也可能会挑战性，因为它涉及到一些复杂的概念，如闭包、作用域和函数式编程。

**理解函数是一等公民**：在Python中，函数是一等公民，这意味着你可以把函数作为参数传给其他函数，可以从函数中返回函数，也可以把函数赋值给变量。理解这一点是理解装饰器的关键。

**掌握闭包的概念**：闭包是一个函数，它有自己的作用域，并在该作用域内维持一些局部变量。这个函数可以访问它的外部作用域中定义的变量。理解闭包是理解装饰器如何工作的基础。

**从简单开始**：开始时，可以从一些简单的装饰器开始学习，例如，实现一个打印函数执行时间的装饰器或者一个用于缓存函数返回值的装饰器。

**实践**：尝试在自己的代码中使用装饰器，将理论应用到实践中，可以帮助你更好地理解装饰器的工作方式和使用场景。

**阅读Python文档和教程**：Python的官方文档和一些高质量的教程（比如Real Python）有一些关于装饰器的深入解释和示例，可以帮助你更好地理解装饰器。

**学习内建装饰器**：理解Python内建的装饰器（如@property，@classmethod和@staticmethod）可以帮助你看到装饰器在实际编程中的应用，也可以启发你创建自己的装饰器。

**学习使用第三方库的装饰器**：许多Python库，如Flask和pytest，都广泛使用装饰器。研究这些库如何使用装饰器，可以帮助你看到装饰器的实际应用，并可能激发你自己的一些想法。

希望这些建议对你有所帮助！祝你学习顺利！