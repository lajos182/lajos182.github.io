---
title: No.6 Python语言基础--IO编程
date: 2020-05-03 10:27:56
tags: [Python, IO]
toc: true
---

### 6.1. 文件读写

读写文件是最常见的`IO`操作。`Python`内置了读写文件的函数，用法和C是兼容的。

读写文件前，我们先必须了解一下，在磁盘上读写文件的功能都是由操作系统提供的，现代操作系统不允许普通的程序直接操作磁盘，所以，读写文件就是请求操作系统打开一个文件对象（通常称为文件描述符），然后，通过操作系统提供的接口从这个文件对象中读取数据（读文件），或者把数据写入这个文件对象（写文件）。

<!--more-->

- 字符编码

要读取非`UTF-8`编码的文本文件，需要给`open()`函数传入`encoding`参数，例如，读取`GBK`编码的文件, 遇到有些编码不规范的文件，你可能会遇到`UnicodeDecodeError`，因为在文本文件中可能夹杂了一些非法编码的字符。遇到这种情况，`open()`函数还接收一个`errors`参数，表示如果遇到编码错误后如何处理。最简单的方式是直接忽略.

```python
>>> f = open('/Users/Administrator/gbk.txt', 'r', encoding='gbk')
>>> f.read()
'测试'
>>> f = open('/Users/Administrator/gbk.txt', 'r', encoding='gbk', errors='ignore)
```

### 6.2. `StringIO`和`BytesIO`

很多时候，数据读写不一定是文件，也可以在内存中读写。

- `StringIO`：在内存中读写`str`

  - 要把`str`写入`StringIO`，我们需要先创建一个`StringIO`，然后，像文件一样写入即可：

    ```python
    >>> from io import StringIO
    >>> f = StringIO()
    >>> f.write('hello')
    5
    >>> f.write(' ')
    1
    >>> f.write('world!')
    6
    # `getvalue()`方法用于获取写入后的`str`
    >>> print(f.getvalue())
    hello world!
    ```

  - 要读取`StringIO`,可以用一个`str`初始化`StringIO`,然后像文件一样读取

    ```python
    >>> from io import StringIO
    >>> f = StringIO('Hello!\nHi!\nGoodbye!')
    >>> while True:
    ...     s = f.readline()
    ...     if s == '':
    ...         break
    ...     print(s.strip())
    ...
    Hello!
    Hi!
    Goodbye!
    ```

- `BytesIO`：`StringIO`操作的只能是`str`，如果要操作二进制数据，就需要使用`BytesIO`。

  - `BytesIO`实现了在内存中读写`bytes`，我们创建一个`BytesIO`，然后写入一些`bytes`。请注意，写入的不是`str`，而是经过`UTF-8`编码的`bytes`。

    ```python
    >>> from io import BytesIO
    >>> f = BytesIO()
    >>> f.write('中文'.encode('utf-8'))
    6
    >>> print(f.getvalue())
    b'\xe4\xb8\xad\xe6\x96\x87'
    ```

  - 和`StringIO`类似，可以用一个`bytes`初始化`BytesIO`，然后，像读文件一样读取。

    ```python
    >>> from io import BytesIO
    >>> f = BytesIO(b'\xe4\xb8\xad\xe6\x96\x87')
    >>> f.read()
    b'\xe4\xb8\xad\xe6\x96\x87'
    ```

### 6.3. `bytes`函数的使用

`bytes`函数返回一个新的`bytes`对象，该对象是一个0<=x<=256区间的整数不可变序列，它是`bytearray`的不可变版本。

- `bytes([source[, encoding[, errors]]])`：返回一个新的`bytes`对象
  - 如果`source`为整数，则返回一个长度为`source`的初始化数组
  - 如果`source`为字符串，则按照指定的`encoding`将字符串转化为字节序列
  - 如果`source`为可迭代类型，则元素必须[0, 255]中的整数
  - 如果`source`为与`buffer`接口一致的对象，则此对象也可以被用于初始化`bytearray`
  - 如果没有输入任何参数，默认就是初始化数组为0个元素

```python
a = bytes([1, 2, 3, 4])
print(a)
print(type(a))
>>>  b'\x01\x02\x03\x04'
<class 'bytes'>

a = [0xe0, 0xb9, 0x80, 0xe0, 0xb8, 0xa7, 0xe0, 0xb9, 0x87, 0xe0, 0xb8, 0x9a, 0xe0,0xb9, 0x84, 0xe0, 0xb8, 0x8b, 0xe0, 0xb8, 0x95, 0xe0, 0xb9, 0x8c]
b  = bytes(a)
print(b)
c = b.decode('utf-8')
print(c)
>>> b'\xe0\xb9\x80\xe0\xb8\xa7\xe0\xb9\x87\xe0\xb8\x9a\xe0\xb9\x84\xe0\xb8\x8b\xe0\xb8\x95\xe0\xb9\x8c'
เว็บไซต์
```

### 6.4. 序列化

- 一般数据持久化存储采用三种方式：**普通文件**、**数据库**、**序列化**

- **`pickle`**：`pickle.dump(obj, fp)`将对象存储到文件，`pickle.load(fp)`将文件转化为对象

  ```python
  import pickle
  class Person(object):
      def __init__(self, name, age):
          self.name = name
          self.age = age
      
  # 存储：对象=>文件
  xiaoming = Person('xiaoming', 18)
  fp = open('text.txt', 'wb')
  pickle.dump(xiaoming, fp)
  print('对象数据存储完毕！')
  fp.close()
  
  # 读取：文件=>对象
  fp = open('text.txt', 'rb')
  xiaoming = pickle.load(fp)
  print(xiaoming.name, xiaoming.age)
  fp.close()
  ```

   `json`：如果我们要在不同的编程语言之间传递对象，就必须把对象序列化为标准格式，比如`XML`，但更好的方法是序列化为`JSON`，因为`JSON`表示出来就是一个字符串，可以被所有语言读取，也可以方便地存储到磁盘或者通过网络传输。`JSON`不仅是标准格式，并且比`XML`更快，而且可以直接在`Web`页面中读取，非常方便。

| **JSON类型** | **Python类型** |
| :----------: | :------------: |
|      {}      |      dict      |
|      []      |      list      |
|    string    |      str       |
|   1234.56    |   int或float   |
|  true/false  |   True/False   |
|     null     |      None      |

`Python`内置的`json`模块提供了非常完善的`Python`对象到`JSON`格式的转换。我们先看看如何把`Python`对象变成一个`JSON`.`dumps()`方法返回一个`str`，内容就是标准的`JSON`。类似的，`dump()`方法可以直接把`JSON`写入一个`file-like Object`。

```python
>>> import json
>>> d = dict(name='Bob', age=20, score=88)
>>> json.dumps(d)
'{"age": 20, "score": 88, "name": "Bob"}'
```

要把`JSON`反序列化为`Python`对象，用`loads()`或者对应的`load()`方法，前者把`JSON`的字符串反序列化，后者从`file-like Object`中读取字符串并反序列化。由于`JSON`标准规定`JSON`编码是`UTF-8`，所以我们总是能正确地在`Python`的`str`与`JSON`的字符串之间转换。

```python
>>> json_str = '{"age": 20, "score": 88, "name": "Bob"}'
>>> json.loads(json_str)
{'age': 20, 'score': 88, 'name': 'Bob'}
```

`Python`的`dict`对象可以直接序列化为`JSON`的{}，不过，很多时候，我们更喜欢用`class`表示对象，比如定义Student类，然后序列化：

```python
import json

class Student(object):
    def __init__(self, name, age, score):
        self.name = name
        self.age = age
        self.score = score

s = Student('Bob', 20, 88)
print(json.dumps(s))

>>> Traceback (most recent call last):
  ...
TypeError: <__main__.Student object at 0x10603cc50> is not JSON serializable
# 这是因为`dumps()`在序列化对象时，需要提供可选参数
# 可选参数default就是把任意一个对象变成一个可序列为JSON的对象，我们只需要为Student专门写一个转换函数，再把函数传进去即可：
def student_to_dict(std):
    return {
        'name': std.name,
        'age': std.age,
        'score': std.score
    }
>>> print(json.dumps(s, default=student_to_dict))
{"age": 20, "name": "Bob", "score": 88}
>>> print(json.dumps(s, default=lambda obj: obj.__dict__))
```

同样的道理，如果我们要把`JSON`反序列化为一个`Student`对象实例，`loads()`方法首先转换出一个`dict`对象，然后，我们传入的`object_hook`函数负责把`dict`转换为`Student`实例：

```python
def dict_to_student(d):
    return Student(d['name'], d['age'], d['score'])
>>> json_str = '{"age": 20, "score": 88, "name": "Bob"}'
>>> print(json.loads(json_str, object_hook=dict2student))
<__main__.Student object at 0x10cd3c190>
```
