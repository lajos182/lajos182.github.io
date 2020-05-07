---
title: No.8 Python语言基础--面向对象
date: 2020-05-03 10:51:48
tags: [Python, 类, 对象]
toc: true
---

### 8.1. 类和对象的基本定义

- **类**：具有相同特征（属性和行为）事物的抽象。其实是一种自定义的数据类型。

- **对象**：某个类的具象。其实是某个类型的变量。

  > 定义类需要使用关键字：**`class`**，也可以使用元类进行定义
  >
  > 类名：原则上只要符合标识符的命名规范即可，但是我们通常使用大驼峰风格(每个单词首字母大写)命名
  >
  > <!--more-->
  >
  > 不要忘记类名后面的冒号`:`
  >
  > 类的内容要进行缩进
  >
  > 行为(方法)：通过函数体现，与外面定义函数的方式相似，第一个参数默认是self
  >
  > **属性**：通过变量体现，属性是动态添加的，因此在定义时可以不体现
  >
  > 成员访问：成员属性(对象.属性名)，成员方法(对象.方法名())

  ![类的属性与方法](https://gitee.com/lajos/image_bucket/raw/master/skill/class.png)

- 示例：小明手里有两张牌，左手♥K，右手♠A，问：小明交换两手的牌后，手里分别是什么？

  - 思路：

    > 先找到对象：小明、左手、♥K、♠A、右手
    >
    > 根据对象抽象出来对应的类：人、牌、手
    >
    > 写出对应的逻辑，反过来完善抽象出来的类
    >
    > 按照题目的要求创建对应的对象，调用相关的方法

  - 代码：

    ```python
    class Poker(object):
    
        def __init__(self, color, number):
            self.color = color
            self.number = number
        
        def __str__(self):
            return f'{self.color}{self.number}'
    
    # 创建两张牌
    p1 = Poker('♥', 'K')
    p2 = Poker('♠', 'A')
           
    class Hand(object):
    
        def __init__(self, poker=None):
            self.poker = poker
    
        def hold_poker(self, poker):
            self.poker = poker
    
    # 创建左右手对象
    left_hand = Hand(p1)
    right_hand = Hand(p2)
    
    class Person(object):
    
        def __init__(self, name, left_hand, right_hand):
            self.name = name
            self.left_hand = left_hand
            self.right_hand = right_hand
    
        def show(self):
            print(f'{self.name}张开手', end=':')
            print(f'左手--{self.left_hand.poker}', end=',')
            print(f'右手--{self.right_hand.poker}')
    
        def swap(self):
            self.left_hand.poker, self.right_hand.poker = self.right_hand.poker, self.left_hand.poker
            print(f'{self.name}交换两手的牌')
    
    # 创建小明对象
    xiaoming = Person('小明', left_hand, right_hand)
    # 展示手里的牌
    xiaoming.show()
    # 交换两手牌
    xiaoming.swap()
    # 再次展示手里的牌
    xiaoming.show()
    ```

- **`self`和`cls`区别**：

  - `python`并未对类中方法的第一个参数名字做限制，只不过是开发人员的习惯性用法。
  - `self`一般指类的实例，表示一个具体的实例本身，如果使用`@staticmethod`，就可以无视这个`self`，而将这个方法当成一个普通的函数使用。
  - `cls`表示这个类的本身

### 8.2. 常见的魔法函数

**魔法函数**：在python中，有一些内置好的特定的方法，这些方法在进行特定的操作时会自动被调用，称之为魔法方法。

- **`__init__()`**：**初始化函数**，在创建实例对象为其赋值时使用，在`__new__`之后，`__init__`必须至少有一个参数`self`，这个就是`__new__`返回的实例，`__init__`是在`__new__`的基础上可以完成一些其它初始化的动作，`__init__`不需要返回值。这种方法常被人称为**构造方法**。

- **`__new__()`**：很多人认为`__init__`是类的构造函数，其实不太确切，`__init__`更多的是负责初始化操作，相当于一个项目中的配置文件，`__new__`才是真正的构造函数，创建并返回一个实例对象，如果`__new__`只调用了一次，就会得到一个对象。**继承自`object`的新式类才有`__new__`这一魔法方法**，`__new__`至少必须要有一个参数`cls`，代表要实例化的类，此参数在实例化时由`Python`解释器自动提供，**`__new__`必须要有返回值，返回实例化出来的实例（很重要）**，这点在自己实现`__new__`时要特别注意，可以`return`父类__new__出来的实例，或者直接是`object`的`__new__`出来的实例，若`__new__`没有正确返回**当前类`cls`**的实例，那`__init__`是不会被调用的，即使是父类的实例也不行。

  - 创建对象的步骤：

    ```markdown
    (1)首先调用__new__得到一个对象
    (2)调用__init__为对象添加属性
    (3)将对象赋值给变量
    ```

  - `__init__`和`__new__`两个魔法方法的例子：

    ```python
    class A(object):
        pass
    
    class B(A):
        
        def __init__(self):
            print('__init__被调用......')
    
        def __new__(cls):
            print('__new__被调用......')
            print(id(cls))
            # 此处采用了参数A而不是cls，__new__没有正确返回当前类cls的实例
            return object.__new__(A)
    
    b = B()
    print(b, type(b))
    print(id(A), id(B))
    
    # 运行结果
    >>> __new__被调用......
    >>> 2219588983608
    >>> <__main__.A object at 0x00000204CA658400> <class '__main__.A'>
    >>> 2219588982664 2219588983608
    ```

    - 从运行结果可以看出，`__new__`中的参数`cls`和B的`id`是相同的，表明`__new__`中默认的参数`cls`就是B类本身，而在`return`时，并没有正确返回当前类`cls`的实例，而是返回了其父类A的实例，因此`__init__`这一魔法方法并没有被调用，此时`__new__`虽然是写在B类中的，但其创建并返回的是一个A类的实例对象。

    ```python
    # 现在将return中的参数A变为cls，再来看一下运行结果：
    class A(object):
        pass
    
    class B(A):
        
        def __init__(self):
            print('__init__被调用......')
    
        def __new__(cls):
            print('__new__被调用......')
            print(id(cls))
            # 此处采用了参数A而不是cls，__new__正确返回当前类cls的实例
            return object.__new__(cls)
    
    b = B()
    print(b, type(b))
    print(id(A), id(B))
    
    # 运行结果
    >>> __new__被调用......
    >>> 2483180430712
    >>> __init__被调用......
    >>> <__main__.B object at 0x0000024229AC93C8> <class '__main__.B'>
    >>> 2483180427880 2483180430712
    ```

    - 可以看出，当`__new__`正确返回其当前类`cls`的实例对象时，`__init__`被调用了，此时创建并返回的是一个B类的实例对象。

- **`__class__()`**：获取已知随想的类（对象.`__class__`）

  ```python
  print(b.__class__)
  >>> <class '__main__.B'>
  ```

  - 当一个类中的某个成员属性是所有该类的对象的公共属性时，可以这样使用：

    ```python
    class A(object):
        count = 0
        def add_count(self):
            # self.__class__.count不再是单纯的某个对象私有的属性，而是类的所有实例对象的共有属性,它相当于self.A.count
            self.__class__.count += 1
            # self.count += 1
    
    a = A()
    a.add_count()
    print('*********a***********', a.count)
    >>> *********a*********** 1
    b = A()
    b.add_count()
    print('*********b***********', b.count)
    >>> *********b*********** 2
    ```

- **`__slots__`**: 用来限制实例属性

  ```python
  class Student(object):
    __slots__ = ('name', 'age')
  
  >>> s = Student()
  >>> s.name = 'Michael
  >>> s.age = 20
  >>> s.score = 80
  Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  AttributeError: 'Student' object has no attribute 'score'
  ```

- **`__str__()`**：该函数要求返回一个字符串，当实例对象作为`print`参数打印时会打印该字符串。当打印一个类时，`print`首先调用的时类里面定义的`__str__`

  ```python
  class A(object):
     
      def __init__(self, attr):
          self.attr = attr
  
      def __str__(self):
          return f'a的属性是{self.attr}'
  
  a = A('name')
  print(a)
  >>> a的属性是name
  print(A)
  >>> <class '__main__.A'>
  ```

- **`__repr__()`**：是给机器用的，在Python解释器里面直接敲对象名再回车后调用的方法。在没有str时就相当于它本身的作用

  ![__repr__用法](https://gitee.com/lajos/image_bucket/raw/master/skill/__repr___%20and%20__str__.png)

- **`__del__()`**：对象在程序运行结束之后进行垃圾回收的时候调用这个方法，来释放资源。此时，此方法是被自动调用的。总而言之，`__del__`魔法方法是在对象没有变量再引用，其**引用计数**减为0，进行垃圾回收的时候自动调用的。

  ```python
  class A(object):
      num_count = 0 # 共有属性
      def __init__(self, name):
          self.name = name
          A.num_count += 1
          print(name, A.num_count)
  
      def __del__(self):
          A.num_count -= 1
          print('Del', self.name, A.num_count)
      
      def test(self):
          print('aaa')
  
  aa = A('hello')
  bb = A('world')
  cc = A('!!!!!')
  print('over')
  
  >>> hello 1
  world 2
  !!!!! 3
  over
  Del hello 2
  Del world 1
  Del !!!!! 0
  ```

- **`__delattr__()`**：删除一个对象的属性时调用

- **`__getattr__()`**：当访问不存在的对象属性时，系统会自动调用

- **`__getattribute__()`**：访问存在的属性时调用（先调用该方法，查看是否存在该属性，若不存在，接着去调用`__getattr__`）

- **`__setattr__()`**：设置成员属性时，系统会自动调用

  ```python
  class A(object):
      
      def __getattr__(self, item):
          print('访问不存在属性时调用__getattr__')
          try:
              return item
          except KeyError:
              raise AttributeError(f"'Dict' object has no attribute {item}")
      
      def __getattribute__(self, item):
          print('访问存在属性时调用__getattribute__')
          return object.__getattribute__(self, item) # 非绑定方法要显式传递self
  
      def __setattr__(self, item, value):
          print('设置对象成员属性的时候，自动调用__setattr__')
          self.__dict__[item] = value
  
      def __delattr__(self, item):
          print('销毁对象成员属性的时候，自动调用__delattr__')
          print('del', item)
  
  a = A()
  a.name = 'xiaoming'
  >>> 设置对象成员属性的时候，自动调用__setattr__
  访问存在属性时调用__getattribute__
  print(a.age)
  >>> 访问存在属性时调用__getattribute__
  访问不存在属性时调用__getattr__
  del a.name
  >>> 销毁对象成员属性的时候，自动调用__delattr__
  del name
  ```

- **`__setitem__()`**：当对对象按照字典设置键值对时，会自动触发该方法

- **`__getitem__()`**：当对对象按照字典操作根据键获取值时，会自动触发该方法

- **`__delitem__()`**：当做字典操作，删除键值对时，自动触发该方法

  ```python
  class A(object):
      
      def __setitem__(self, key, value):
          print('调用了__setitem__方法')
          self.__dict__[key] = value
  
      def __getitem__(self, item):
          print('调用了__getitem__方法')
          return self.__dict__[item]
  
      def __delitem__(self, key):
          print('调用了__delitem__方法')
          del self.__dict__[key]
  
  a = A()
  a['name'] = name
  >>> '调用了__setitem__方法'
  print(a['name'])
  >>> '调用了__getitem__方法'
  name
  del a['name']
  >>> '调用了__delitem__方法'
  ```

- **`__iter__()`**：只要定义了`__iter__()`方法对象，就可以使用迭代器访问，这意味着，我们可以迭代我们自己定义的对象。

  ```python
  class A(object):
      
      def __init__(self):
          self._obj = []
      
      def __iter__(self):
          return iter(self._obj)
  
      def add(self, obj):
          self._obj.append(obj)
  
  a = A()
  a.add(1)
  a.add(2)
  for i in a:
      print(i)
  ```

- **`__call__()`**：当实例被当做函数进行调用时会触发

  ```python
  class A(object):
      
      def __init__(self):
          self.name = 'xiaoming'
      
      def __call__(self, age):
          return f'I am {self.name}, my age is {age} years old!'
  
  a = A()
  print(a(18))
  >>> I am xiaoming, my age is 18 years old!
  ```

- **`__len__()`**：`len`调用后会调用对象的`__len__`函数，我们可以为其定制输出

  ```python
  class A(object):
      
      def __init__(self, array):
          self.array = array
  
      def __len__(self):
          return len(self.array) + 100
  
  a = A([1, 2, 3, 4, 5])
  print(len(a))
  >>> 105
  ```

### 8.3. 枚举类的使用

- 当我们需要定义常量时，通常使用大写变量通过整数来定义，例如月份，好处时简单，缺点是类型是`int`，并且仍然是变量。

```python
 JAN = 1
 FEB = 2
 MAR = 3
 ...
 NOV = 11
 DEC = 12
```

- 更好的办法是为这样的枚举类型定义一个`class`类型，然后每个常量都是`class`的一个唯一实例。Python提供了`Enum`类来实现这个功能:

```python
from enum import Enum

Month = Enum('Month', ('Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec'))

# 这样我们就获得了Month类型的枚举类，可以直接使用Month.Jan来引用一个常量，或者枚举它的所有成员：
for name, member in Month.__members__.items():
    print(name, '=>', member, ',', member.value)
```

- 自定义枚举类：更精确的控制枚举类型，可以从`Enum`派生出自定义类,使用`@unique`装饰器可以帮助我们检查保证没有重复值

```python
from enum import Enum, unique

@unique
class Weekday(Enum):
    Sun = 0 # Sun的value被设定为0
    Mon = 1
    Tue = 2
    Wed = 3
    Thu = 4
    Fri = 5
    Sat = 6

# 访问这些枚举类型的方法：
>>> day1 = Weekday.Mon
>>> print(day1)
Weekday.Mon
>>> print(Weekday.Tue)
Weekday.Tue
>>> print(Weekday['Tue'])
Weekday.Tue
>>> print(Weekday.Tue.value)
2
>>> print(day1 == Weekday.Mon)
True
>>> print(day1 == Weekday.Tue)
False
>>> print(Weekday(1))
Weekday.Mon
>>> print(day1 == Weekday(1))
True
>>> Weekday(7)
Traceback (most recent call last):
  ...
ValueError: 7 is not a valid Weekday
>>> for name, member in Weekday.__members__.items():
...     print(name, '=>', member)
...
Sun => Weekday.Sun
Mon => Weekday.Mon
Tue => Weekday.Tue
Wed => Weekday.Wed
Thu => Weekday.Thu
Fri => Weekday.Fri
Sat => Weekday.Sat
```

- 枚举类的使用

```python
from enum import Enum, unique

class Gender(Enum):
    Male = 0
    Female = 1

class Student(object):
    def __init__(self, name, gender):
        self.name = name
        self.gender = gender

# 测试
mary = Student('Mary', name, Gender.Female):
if mary.gender == Gender.Female:
    print('测试通过')
else：
    print('测试失败')
```

### 8.4. 类的继承

- **继承**：父类的属性和方法子类可以直接拥有的称为继承。

  ```python
  class Animal(object):
  
      def __init__(self, name):
          self.name = name
  
      def run(self):
          print('动物一般都喜欢跑！')
  
  class Dog(Animal):
  
      pass
  
  d = Dog('wangcai')
  # 直接继承父类的属性
  print(d.name)
  # 直接拥有父类的方法
  d.run()
  >>> wangcai
  动物一般都喜欢跑！
  ```

- **派生**：子类在父类的基础上衍生出来的新的特征(属性和方法)。

  ```python
  class Animal(object):
  
      def __init__(self, name):
          self.name = name
  
      def run(self):
          print('动物一般都喜欢跑！')
  
  class Rabbit(Animal):
      # 添加方法
      def eat(self):
          print('爱吃萝卜和青菜！')
  
  rabbit = Rabbit('xiaobai')
  rabbit.eat()
  >>> 爱吃萝卜和青菜！
  # 添加属性
  rabbit.color = 'white'
  print(rabbit.color)
  >>> white
  ```

- **重写**：父类的方法完全不合适，子类需要全部的覆盖。或者是父类的方法合适，但需要完善。

  ```python
  class Animal(object):
  
      def run(self):
          print('动物一般都喜欢跑！')
  
      def eat(self):
          print('小动物也是一天三顿！')
  
  class Cat(Animal):
      # 当父类的方法完全不合适时，可以覆盖重写
      def run(self):
          print('喜欢走猫步！')
      # 父类的方法合适，但是子类需要添加内容完善功能
      def eat(self):
          # 调用父类方法，不建议使用
          # Animal.eat(self)
          # super(Cat, self).eat()
          # 类名及self可以不传
          super().eat()
          print('最喜欢吃鱼！')
  
  jerry = Cat()
  jerry.run()
  >>> 喜欢走猫步！
  jerry.eat()
  >>> 小动物也是一天三顿！
  最喜欢吃鱼！
  ```

### 8.5. 多继承

- **多继承**：继承多个父类，会涉及到查找顺序(MRO)、重复调用(钻石继承，也叫菱形继承问题)等。

  ```python
  class Father(object):
      def __init__(self, money):
          self.money = money
  
      def play(self):
          print('play basketball')
  
      def eat(self):
          print('喜欢吃红烧肉！')
  
  class Mother(object):
      def __init__(self, face_value):
          self.face_value = face_value
  
      def makeup(self):
          print('like make up')
  
      def eat(self):
          print('喜欢吃饺子！')
  
  class Son(Father, Mother):
      def __init__(self, money, face_value, hobby):
          # 多继承时无法使用super()，因为super()只可以再单继承中使用，当然子类也可以扩展自己的属性
          Father.__init__(self, money)
          Mother.__init__(self, face_value)
          self.hobby = hobby
  
  son = Son(500, 100, 'IT')
  print(son.money)
  >>> 500
  print(son.face_value)
  >>> 100
  # eat继承的父类都有该方法，会根据继承的顺序谁在前继承谁
  son.eat()
  >>> 喜欢吃红烧肉！
  son.makeup()
  >>> like make up
  son.play()
  >>> play basketball
  ```

- `MRO`即`method resolution order`，用于判断子类调用的属性来自于哪个父类。在Python2.3之前，`MRO`是基于**深度优先算**法的，自2.3开始使用`C3`算法，定义类时需要继承`object`，这样的类称为**新式类**，否则为**旧式类**。案例可查看[https://www.lajos.top/2020/04/27/No-1-Python%E7%AE%80%E4%BB%8B%E4%B8%8E%E7%89%88%E6%9C%AC%E4%BB%8B%E7%BB%8D/](https://www.lajos.top/2020/04/27/No-1-Python简介与版本介绍/)

  ![旧式类和新式类](https://gitee.com/lajos/image_bucket/raw/master/skill/MRO%20and%20C3.png)

### 8.6. 多态性

- **多态**：一种食物的多种形态。

- 案例：

  - 如果要定义两个类(猫和老鼠)，他们分别都有`name`属性和`eat`方法，代码实现：

    ```python
    class Cat(object):
        def __init__(self, name):
            self.name = name
        def eat(self):
            print(f'{self.name}在吃！')
            
    class Mouse(object):
        def __init__(self, name):
            self.name = name
        def eat(self):
            print(f'{self.name}在吃！')
    tom = Cat('tom')
    jerry = Mouse('jerry')
    tom.eat()
    >>> tom在吃！
    jerry.eat()
    >>> jerry在吃！
    ```

  - 如果有100种动物，他们都有`name`属性和`eat`方法，当然不需要这样写，就把这些动物都继承自一个基类。(如果一种植物也有这个属性和方法时，也可以继承自这个基类，这就是**鸭子模型**)。

    ```python
    class Animal(object):
         def __init__(self, name):
            self.name = name
        def eat(self):
            print(f'{self.name}在吃！')
            
    class Cat(Animal):
        def __init__(self, name):
            return super(Cat, self).__init__(name)
    
    class Mouse(Animal):
        def __init__(self, name):
            return super().__init__(name)
    tom = Cat('tom')
    jerry = Mouse('jerry')
    tom.eat()
    >>> tom在吃！
    jerry.eat()
    >>> jerry在吃！   
    ```

  - 如果人要给100种动物喂食物，如何实现呢？

    ```python
    class Person(object):
        '''
        def feedCat(self, cat):
            print('给你食物！')
            cat.eat()
        
        def feedMouse(self, mouse):
            print('给你食物！')
            mouse.eat()
        '''
        # 只需要写一种方法，直接给这些动物继承的基类作为一个对象
        def feed_animal(self, animal):
            print('给你食物')
            animal.eat()
    per = Person()
    # tom和jerry都继承自动物
    per.feed_animal(tom)
    per.feed_animal(jerry)     
    ```

### 8.7. 封装性

**封装**：隐藏对象的属性和实现细节，仅对外提供公共访问方式。这样的好处：**将变化隔离**、**便于使用**、**提高复用性**、**提高安全性**。

**封装原则**：①将不需要对外提供的内容都隐藏起来；②把属性都隐藏，提供公共方法对其访问

**广义的封装**：把属性函数都放到类里。

**侠义的封装**：定义私有成员。

- **访问权限**：

  - 共有的：类中的普通属性和方法，默认都是公有的，可以在类的内部、外部、子类中使用
  - 私有的：其实仅仅是一种变形操作，类中所有双下划线开头的名称(`__x`)都会自动变形成：`_类名__x`的形式。只能在本类的内部使用，不能再外部及子类中使用，且有些方法或属性 不希望被子类继承。

  ```python
  class Person(object):
  
      def __init__(self, name, age):
          self.name = name
          # 在属性的前面添加两个'_'，外部访问，系统内部的除外
          # 默认的属性名：__age => _Person__age
          self.__age = age
  
      def eat(self):
          print('民以食为天！')
      
      #  在方法的前面添加两个'_'
      def __inner(self):
          print('私有方法，不想让外部调用！')
  
      def test(self):
          # 在类的内部可以访问私有属性和方法
          print(self.__age)
          self.__inner()
          self.eat()
  
  p = Person('老王', 50)
  # 默认所有的属性和方法都是公有的，就是在类的外面可以直接使用
  p.eat()
  >>> 民以食为天！
  print(p.name)
  >>> 老王
  # 以下两句会报错，提示没有相关的属性或方法
  # p.__inner()
  # print(p.__age)
  # 可以通过系统修改的名称找到，但是强烈建议不要这样使用
  print(p._Person__age)
  >>> 18
  p._Person__inner()
  >>> 私有方法，不想让外部调用！
  # 调用类的内部方法可以访问
  p.test()
  
  class Man(Person):
      
      def __inner(self):
          print('我想覆盖父类的属性，但没有办法！')
  
      def eat(self):
          print('我就是爱吃肉！')
  
  m = Man('lajos', 18)
  m.test()
  >>> 18
  私有方法，不想让外部调用！ # 子类无法修改父类的方法
  我就是爱吃肉！ # 这个eat()不是私有方法，被覆盖
  
  ```

- **封装与扩展性**

  > 封装在于明确区分内外，使得类实现者可以修改封装内的东西而不影响外部调用者的代码；而外部使用者只需要知道一个接口(函数)，只要接口(函数)名、参数不变，合作基础使用者的代码永远无需改变。这就是提供一个良好的——或者说，只要接口这个基础约定不变，则代码改变不足为虑。

  ```python
  # 类的设计者
  class Room(object):
  
      def __init__(self, name, owner, width, length, high):
          self.name = name
          self.owner = owner
          self.__width = width
          self.__length = length
          self.__high = high
      
      def tell_area(self): # 对外提供的接口，隐藏了内部的实现细节，此时我们想求的是面积
          return self.__width * self.__length
  
  r1 = Room('卧室', 'egon', 20, 20, 20)
  print(r1.tell_area()) # 使用者调用接口tell_area
  >>> 400
  
  # 类的设计者，轻松的扩展了功能，而类的使用者完全不需要改变自己的代码
  class Room:
      def __init__(self,name,owner,width,length,high):
          self.name=name
          self.owner=owner
          self.__width=width
          self.__length=length
          self.__high=high
      def tell_area(self): 
          # 对外提供的接口，隐藏内部实现，此时我们想求的是体积,内部逻辑变了,只需求修该下列一行就可以很简答的实现,而且外部调用感知不到,仍然使用该方法，但是功能已经变了
          return self.__width * self.__length * self.__high
  
  
  # 对于仍然在使用tell_area接口的人来说，根本无需改动自己的代码，就可以用上新功能
  prnit(r1.tell_area())
  >>> 8000
  ```

### 8.8. 类属性

**类属性**：定义类时，卸载类中，但是方法外的属性，通常放在类的开头位置。

```python
class Person(object):
    nation = 'China'

    def __init__(self, name):
        self.name = name
        self.nation = '中国'
# 通过类名访问类属性
print(Person.nation)
# print(Person.name)
# 通过对象也可一访问类属性，但是不建议
# 当对象有同名的成员属性时，使用的就是成员属性
p = Person('xiaoming')
print(p.nation)

# 也可以动态添加
Person.hello = 'hello'
print(Person.hello)

# 特殊的类属性
# 表示的类名字字符串，对象没有该属性
print(Person.__name__)
# 表示父类构成的元组，对象没有该属性
print(Person.__bases__)

# 存储类相关的信息
print(Person.__dict__)
print(p.__dict__)
```

### 8.9. 类方法(`classmethod`)

**类方法**：定义时使用装饰器`@classmethod`，通过类名进行调用。可以创建对象或者简介的创建对象，对外提供简单易用的接口。传入的第一个参数为`cls`，是类本身，并且，类方法可以通过类直接调用，或通过实例直接调用。但无论哪种调用方式，最左侧传入的参数一定是类本身。

```python
class A(object):

    @classmethod
    def func(cls):
        print('我是类方法！')

A.func() # 通过类直接调用
a = A()
a.func() # 通过实例对象调用
```

### 8.10. 静态方法(`staticmethod`)

**静态方法**：使用装饰器`@staticmethod`进行装饰，定义是不需要第一个参数`cls`。调用过程中，无需进行实例化。

```python
class A(object):

    @staticmethod
    def func():
        print('我是静态方法！')
A.func() # 通过类直接调用
a = A()
a.func() # 通过实例对象调用
```

> 重点来了：[**Python中的实例方法、类方法和静态方法有什么区别？**](https://blog.csdn.net/sinat_41898105/article/details/80661798)
>
> 三种方法都可以通过对象进行调用，但类方法和静态方法无法访问对象属性，类方法通过对象调用获取的仍是类属性（并非对象属性）；普通方法无法通过类名调用，类方法和静态方法可以，但静态方法不能进行访问，仅仅只是通过传值得方式（与函数调用相同）

### 8.11. 属性函数

**属性函数**：将方法当作属性一样调用，保护或处理特定属性。

```python
import hashlib
class User(object):
    
    def __init__(self, username, password):
        self.username = username
        self.__password = password

    # 使用property装饰的方法，可以当作属性访问
    # 可以保护特定的属性
    @property
    def password(self):
        print('有人想直接获取密码！')
        # 可以给出警告提醒
        return self.__password
    
    # 当设置password属性是，会自动调用该方法
    @password.setter
    def password(self, password):
        print('注意了，有人想修改密码', password)
        # 可以对密码进行特定处理，然后保存
        md = hashlib.md5()
        md.update(password.encode('utf-8'))
        self.__password = md.hexdigest()

user = User('xiaoming', '123456')
print(user.username)
# 很多时候直接访问密码是不需要的，也是不安全的
# 直接访问password属性，自动回调用使用property修饰后的password方法
print(user.password)
# 修改属性时会调用setter装饰的方法
user.password = '654321'
```

### 8.12. 对象判断

- **判断一个对象是否可以像函数一样进行调用？**

  ```python
  class A(object):
      def __call__(self, *args, **kwargs):
          print('++++')
  
  def test():
      pass
  
  
  # 不能使用isinstance方法判断一个对象是否是函数
  # isinstance(test, function)
  
  # 打印时仍然是function类型
  # print(type(test))
  
  # 判断一个对象是否可以像函数一样调用
  print(callable(test))
  >>> True
  a = A()
  print(callable(a))
  >>> True
  
  # 判断对象是否拥有call属性
  print(hasattr(test, 'call'))
  >>> False
  print(hasattr(a, 'call'))
  >>> False
  
  # 判断是否是函数
  from inspect import isfunction
  print(isfunction(test))
  >>> True
  print(isfunction(a))
  >>> False
  ```