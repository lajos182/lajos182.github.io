---
title: No.1 Python简介与版本介绍
date: 2020-04-27 12:08:33
tags: Python
toc: Toc
---

## 1.1. Python简介

+ Python是一种**解释型**(不需要编译)、**面向对象**、**动态数据类型**的交互式语言，Python是由由荷兰人Guido van Rossum于1989年发明，第一个公开发行版发行于1991年。

+ **Python的优势**：

  + **易于学习**：Python有相对较少的关键字，结构简单，有明确定义的语法，学习起来相对简单。 

 <!--more-->

  + **易于阅读**：Python代码的定义比较清晰，易于阅读。 

  + **易于维护**：Python的成功在于它的源代码是相当容易维护。

  + **具有一个广泛的标准库**：Python的最大优势之一是具有丰富的库，可跨平台，兼容性较好。

  + **互动模式**：互动模式的支持，您可以从终端输入执行代码并获得结果的语言，互动的测试和调试代码的片段。 

  + **可移植**：基于其开放源代码的特性，Python已经被移植到许多平台。 

  + **可扩展性**：如果需要一段运行很快的关键代码，或者想要编程一些不愿开放的算法，你可以使用c或者c++完成那部分程序，然后从你的Python程序中进行调用。 

  + **GUI编程**：Python支持GUI可以创建和移植到许多系统调用。 

  + **可嵌入**：你可以将Python嵌入到c/c++程序中，让你的程序用户得到“脚本”的能力。

+ **缺点**：

  + **运行速度慢**：和C程序相比比较慢，因为Python是解释型语言，代码在执行时会一行一行的编译成CPU能理解的机器码，这个翻译过程非常耗时，所以很慢。而C程序是运行前直接编译成CPU能执行的机器码，所以非常快。

  +  **代码不能加密**：如果要发布Python程序，实际上就是发布源代码，这一点与C语言不同,C语言不能发布源代码，只需要把编译后的机器码（也就是windows上常见的xxx.exe文件）发布出去。要从机器码反推出C代码是不可能的，所以，凡是编译型的语言，都没有这个问题，而解释型的语言，则必

## 1.2. Python的设计模式

**设计模式的定义**：为了解决面向对象系统中重要和重复的设计封装在一起的一种代码实现框架,可以使得代码更加易于扩展和调用。

- **设计模式的基本要素**：模式名称、问题、解决方案、效果。

  **设计模式的六大原则**：	

  - 开闭原则：一个软件实体,如类,模块和函数应该对扩展开发,对修改关闭.既软件实体应尽量在不修改原有代码的情况下进行扩展。
  - 里氏替换原则：所有引用父类的方法必须能透明的使用其子类的对象。
  - 依赖倒置原则：高层模块不应该依赖底层模块,二者都应该依赖其抽象,抽象不应该依赖于细节,细节应该依赖抽象,换而言之,要针对接口编程而不是针对实现编程。
  - 接口隔离原则：使用多个专门的接口,而不是使用单一的总接口,即客户端不应该依赖那些并不需要的接口。
  - 迪米特法则：一个软件实体应该尽可能的少与其他实体相互作用。
  - 单一职责原则：不要存在多个导致类变更的原因.即一个类只负责一项职责。



**Python的设计模式**：

![the design pattern of python](https://gitee.com/lajos/image_bucket/raw/master/skill/the%20design%20pattern%20of%20python.png)

二十三设计模式案例详情，可参考：https://www.cnblogs.com/Liqiongyu/p/5916710.html



## 1.3. 单例模式

> **单利模式**是一种常用的软件设计模式。在它的核心结构中只包含一个被称为单例类的特殊类。通过单例模式可以保证系统中一个类只有一个实例而且该实例易于外界访问，从而方便对实例个数的控制并节约系统资源。如果希望在系统中某个类的对象只能存在一个，单例模式是最好的解决方案。
>
> ```
> `__new__()`在`__init__()`之前被调用，用于生成实例对象。利用这个方法和类的属性的特点可以实现设计模式的单例模式。单例模式是指创建唯一对象，单例模式设计的类只能实例 **这个绝对常考啊.绝对要记住1~2个方法,当时面试官是让手写的.**
> ```



### 1.3.1. 使用`__new__`方法

`__new__`是真正创建实例对象的方法，所以重写基类的`__new__`方法，以此来保证创建对象的时候只生成一个实例 

```python
class Singleton(object):
    _instance = None
    def __new__(cls, *args, **kwargs):
        if cls._instance == super(Singlento, cls).__new__(cls, *args, **kwargs)
        return cls._instance

class Foo(Signleton):
    pass

foo1 = Foo()
foo2 = Foo()

print(foo1 is foo2) # True
```

### 1.3.2. 使用装饰器

装饰器维护一个字典对象instances，缓存了所有单例类，只要单例不存在则创建，已经存在直接返回该实例对象。

```python
def singleton(cls):
    instances = {}
    def wrapper(*args, **kwargs):
        if cls not in instances:
            instances[cls] = cls(*args, **kwargs)
        return instance[cls]
    return wrapper

@singleton
class Foo(object):
    pass

foo1 = Foo()
foo2 = Foo()

print(foo1 is foo2) # True
```

### 1.3.3. 使用元类

元类（参考：[深刻理解Python中的元类](http://blog.jobbole.com/21351/)）是用于创建类对象的类，类对象创建实例对象时一定会调用`__call__`方法，因此在调用`__call__`时候保证始终只创建一个实例即可，`type`是python中的一个元类。

```python
class Singleton(type):
    def __call__(cls, *args, **kwargs):
        if not hasattr(cls, '_instance'):
            cls._instance = super(Singleton, cls).__call__(*args, **kwargs)
        return cls._instance
 
 
class Foo(object):
    __metaclass__ = Singleton
 
 
foo1 = Foo()
foo2 = Foo()
 
print(foo1 is foo2)  # True
```

### 1.3.4. import方法

import作为python的模块，是天然的单例模式，因为模块在第一次导入时，会生成 `.pyc` 文件，当第二次导入时，就会直接加载 `.pyc` 文件，而不会再次执行模块代码。因此，我们只需把相关的函数和数据定义在一个模块中，就可以获得一个单例对象了。如果我们真的想要一个单例类，可以考虑这样做：

```python
class My_Singleton(object):
    def foo(self):
        pass
 
my_singleton = My_Singleton()
```

将上面的代码保存在文件 `mysingleton.py` 中，然后这样使用：

```python
from mysingleton import my_singleton
 
my_singleton.foo()
```

[**单例模式伯乐在线详细解释**](http://python.jobbole.com/87294/)

## 1.4. Python中的元类(metaclass)

这个非常的不常用,但是像ORM这种复杂的结构还是会需要的,详情请看:<http://stackoverflow.com/questions/100003/what-is-a-metaclass-in-python>，中文看这个解释很详细[深刻理解Python中的元类](http://blog.jobbole.com/21351/)

Python处处皆对象，类其实也是对象，使用`class`关键字时，Python会自动创建此对象。但与Python中的大多数内容一样，`type`为您提供了手动执行此操作的方法。[`type`](http://docs.python.org/2/library/functions.html#type)有一个完全不同的能力，它也可以动态创建类。`type`可以将类的描述作为参数，并返回一个类。

```python
type(类名, 类名的元组(针对继承的情况，可以为空), 包含属性的字典(名称和值))
>>> Foo = type('Foo', (), {'bar': True})
>>>	print(Foo)
True
>>> f = Foo()
>>> print(f)
<__main__.Foo object at 0x8a9b84c>
>>> print(f.bar)
True
```

## 1.5. Python自省

这个也是python彪悍的特性.

自省就是面向对象的语言所写的程序在运行时,所能知道对象的类型.简单一句就是运行时能够获得对象的类型.比如`type()`，`dir()`，`getattr()`，`hasattr()`，`isinstance()`

### 1.5.1. 访问对象的属性

`dir([obj])`：调用这个方法将返回包含obj大多数属性名的列表（会有一些特殊的属性不包含在内）。obj的默认值是当前的模块对象。

`hasattr(obj, attr)`：这个方法用于检查obj是否有一个名为attr的值的属性，返回一个布尔值。

`getattr(obj, attr)`：调用这个方法将返回obj中名为attr值的属性的值，例如如果attr为’bar’，则返回obj.bar。

`setattr(obj, attr, val)`：调用这个方法将给obj的名为attr的值的属性赋值为val。例如如果attr为’bar’，则相当于obj.bar = val。

```python
class Cat(object):

    def __init__(self, name, *args, **kwargs):
            self.name = name

    def sayHi(self):
        print(self.name)

cat = Cat('kitty')
print(cat.name)
cat.sayHi()
print(dir(cat)) # 获取实例的属性名，以列表形式返回
>>> ['__class__', '__delattr__', '__dict__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__le__', '__lt__', '__module__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__', 'name', 'sayHi']

if hasattr(cat, 'name'): # 检查实例是否有这个属性
    setattr(cat, 'name', 'tiger') # same as: a.name = 'tiger'
print(getattr(cat, 'name')) # same as: print(a.name)
getattr(cat, 'sayHi')() # same as: cat.sayHi()
```

### 1.5.2. `insepect`模块自省

`is{module|class|function|method|builtin}(obj)`：检查对象是否为模块、类、函数、方法、内建函数或方法。

`isroutine(obj)`：用于检查对象是否为函数、方法、内建函数或方法等等可调用类型。用这个方法会比多个is*()更方便，不过它的实现仍然是用了多个is*()。

`getmembers(object[, predicate])`：这个方法是dir()的扩展版，它会将dir()找到的名字对应的属性一并返回，形如[(name, value), …]。另外，predicate是一个方法的引用，如果指定，则应当接受value作为参数并返回一个布尔值，如果为False，相应的属性将不会返回。使用is*作为第二个参数可以过滤出指定类型的属性。

`getmodule(object)`：返回object的定义所在的模块对象。

`get{file|sourcefile}(object)`：获取object的定义所在的模块的文件名|源代码文件名（如果没有则返回None）。用于内建的对象（内建模块、类、函数、方法）上时会抛出TypeError异常。

`get{source|sourcelines}(object)`：获取object的定义的源代码，以字符串|字符串列表返回。代码无法访问时会抛出IOError异常。只能用于module/class/function/method/code/frame/traceack对象。

`getargspec(func)`：仅用于方法，获取方法声明的参数，返回元组，分别是(普通参数名的列表, *参数名, **参数名, 默认值元组)。如果没有值，将是空列表和3个None。如果是2.6以上版本，将返回一个命名元组(Named Tuple)，即除了索引外还可以使用属性名访问元组中的元素。

`getargvalues(frame)`：仅用于栈帧，获取栈帧中保存的该次函数调用的参数值，返回元组，分别是(普通参数名的列表, *参数名, **参数名, 帧的locals())。如果是2.6以上版本，将返回一个命名元组(Named Tuple)，即除了索引外还可以使用属性名访问元组中的元素。

`getcallargs(func[, *args][, **kwds])`：返回使用args和kwds调用该方法时各参数对应的值的字典。这个方法仅在2.7版本中才有。

`getmro(cls)`：返回一个类型元组，查找类属性时按照这个元组中的顺序。如果是新式类，与cls.`__mro__`结果一样。但旧式类没有`__mro__`这个属性，直接使用这个属性会报异常，所以这个方法还是有它的价值的。

`currentframe()`：返回当前的栈帧对象。

## 1.6. Python中的重载

引自知乎:<http://www.zhihu.com/question/20053359>

函数重载的目的：解决**可变参数类型**、**可变参数个数**两大问题。

另外，一个基本的设计原则是，仅仅当两个函数除了参数类型和参数个数不同以外，其功能是完全相同的，此时才使用函数重载，如果两个函数的功能其实不同，那么不应当使用重载，而应当使用一个名字不同的函数。

函数功能相同，但是参数类型不同，python 如何处理？答案是根本不需要处理，因为 python 可以接受任何类型的参数，如果函数的功能相同，那么不同的参数类型在 python 中很可能是相同的代码，没有必要做成两个不同函数。

函数功能相同，但参数个数不同，python 如何处理？大家知道，答案就是缺省参数。对那些缺少的参数设定为缺省参数即可解决问题。因为你假设函数功能相同，那么那些缺少的参数终归是需要用的。

## 1.7. <a href="#1.7">新式类和旧式类</a>

[stackoverflow]([stackoverflow](http://stackoverflow.com/questions/54867/what-is-the-difference-between-old-style-and-new-style-classes-in-python))

**在Python 2.1中，旧式类是用户可用的唯一风格。**

（旧式）类的概念与类型的概念无关：如果`x`是旧式类的实例，则`x.__class__` 指定类的类`x`，但`type(x)`始终是`<type   'instance'>`。

这反映了这样一个事实，即所有旧式实例独立于其类，都使用一个称为实例的内置类型实现。

**Python 2.2中引入了新式类，以统一类和类型的概念**。新式类只是用户定义的类型，不多也不少。

如果x是新样式类的实例，那么`type(x)`通常是相同的`x.__class__`（尽管不能保证 - 允许新样式类实例覆盖返回的值`x.__class__`）。

**引入新式类的主要动机是提供具有完整元模型的统一对象模型**。

它还具有许多直接的好处，例如能够对大多数内置类型进行子类化，或引入“描述符”，从而启用计算属性。

**出于兼容性原因，默认情况下类仍为旧式**。

通过将另一个新样式类（即类型）指定为父类来创建新样式类，或者如果不需要其他父类，则创建“顶级类型”对象。

除了返回的类型之外，新样式类的行为在许多重要细节中与旧样式类的行为不同。

其中一些更改是新对象模型的基础，就像调用特殊方法的方式一样。其他是针对兼容性问题之前无法实现的“修复”，例如在多重继承的情况下的方法解析顺序。

**Python 3只有新式的类**。

无论你是否是子类`object`，类都是Python 3中的新风格。这篇文章很好的介绍了新式类的特性: <http://www.cnblogs.com/btchenguang/archive/2012/09/17/2689146.html>

> 一个旧式类的深度优先的例子

```python
class A():
    def foo1(self):
        print('A')

class B(A):
    def foo2(self):
        pass
    
class C(A):
    def foo1(self):
        print('C')
        
class D(B, C):
    pass

d = D()
d.foo1()
>>> C
```

**按照经典类的查找顺序从左到右深度优先的规则，在访问d.foo1()的时候,D这个类是没有的..那么往上查找,先找到B,里面没有,深度优先,访问A,找到了foo1(),所以这时候调用的是A的foo1()，从而导致C重写的foo1()被绕过。但在新式类种，按照广度优先的原则，访问d.foo1()时，D没有，继续找B，发现B里面也没有foo1()，再找C， C中有foo1()，直接就调用该方法，输入结果就是C**。

## 1.8. 鸭子模型

`Python`崇尚鸭子类型，即‘如果看起来像、叫声像而且走起路来像鸭子，那么它就是**鸭子**

`python`程序员通常根据这种行为来编写程序。例如，如果想编写现有对象的自定义版本，可以继承该对象。

也可以创建一个外观和行为像，但与它无任何关系的全新对象，后者通常用于保存程序组件的松耦合度。

## 1.9. Python中的作用域

Python中，程序的变量并不是哪个位置都可以访问的，访问权限决定于这个变量是在哪里赋值的。变量的作用域决定了在哪一部分程序可以访问哪个特定的变量名称。Python的作用域一共有4种，分别是：

- L（Local） 局部作用域
- E（Enclosing）闭包函数外的函数中
- G（Global）全局作用域
- B（Bulit-in）内置作用域（内置函数所在模块的范围）

**Python执行时查找作用域的顺序是L-E-G-B**，即：先在局部找，局部找不到去局部外的局部(闭包)，然后是全局再到内建

```python
g_count = 0 # 全局作用域
def outer():
    o_count = 1 # 闭包函数外的函数中
    def inner():
        i_count = 2 # 局部作用域
```



## 1.10. GIL线程全局锁

线程全局锁(Global Interpreter Lock),即Python为了保证线程安全而采取的独立线程运行的限制,说白了就是一个核只能在同一时间运行一个线程.**对于io密集型任务，python的多线程起到作用，但对于cpu密集型任务，python的多线程几乎占不到任何优势，还有可能因为争夺资源而变慢。**

见[Python 最难的问题](http://www.oschina.net/translate/pythons-hardest-problem)

解决办法就是多进程和下面的协程(协程也只是单CPU,但是能减小切换代价提升性能).

## 1.11. Python里的拷贝



## 1.12. Python垃圾回收机制

`Python GC`主要使用引用计数（`reference counting`）来跟踪和回收垃圾。在引用计数的基础上，通过“标记-清除”（`mark and sweep`）解决容器对象可能产生的循环引用问题，通过“分代回收”（`generation collection`）以空间换时间的方法提高垃圾回收效率。

### 1.12.1. 引用计数

`PyObject`是每个对象必有的内容，其中`ob_refcnt`就是做为引用计数。当一个对象有新的引用时，它的`ob_refcnt`就会增加，当引用它的对象被删除，它的`ob_refcnt`就会减少.引用计数为0时，该对象生命就结束了。

优点:

+ 简单

+ 实时性

缺点:

+ 维护引用计数消耗资源

+ 循环引用

### 1.12.2. 标记-清除机制

基本思路是先按需分配，等到没有空闲内存的时候从寄存器和程序栈上的引用出发，遍历以对象为节点、以引用为边构成的图，把所有可以访问到的对象打上标记，然后清扫一遍内存空间，把所有没标记的对象释放。

### 1.12.3. 分代技术

分代回收的整体思想是：将系统中的所有内存块根据其存活时间划分为不同的集合，每个集合就成为一个“代”，垃圾收集频率随着“代”的存活时间的增大而减小，存活时间通常利用经过几次垃圾回收来度量。

Python默认定义了三代对象集合，索引数越大，对象存活时间越长。

举例： 当某些内存块M经过了3次垃圾收集的清洗之后还存活时，我们就将内存块M划到一个集合A中去，而新分配的内存都划分到集合B中去。当垃圾收集开始工作时，大多数情况都只对集合B进行垃圾回收，而对集合A进行垃圾回收要隔相当长一段时间后才进行，这就使得垃圾收集机制需要处理的内存少了，效率自然就提高了。在这个过程中，集合B中的某些内存块由于存活时间长而会被转移到集合A中，当然，集合A中实际上也存在一些垃圾，这些垃圾的回收会因为这种分代的机制而被延迟。

## 1.13. Python2.x与Python3.x区别

[Python2.x与Python3.x区别](https://gitee.com/lajos/image_bucket/raw/master/skill/Python2.x与Python3.x的区别.pdf)