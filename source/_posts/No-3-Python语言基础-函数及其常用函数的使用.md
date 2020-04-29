---
title: No.3 Python语言基础--函数及其常用函数的使用
date: 2020-04-28 18:35:40
tags: [Python, 函数]
toc: True
---

### 3.1. 函数的基本定义

**函数**：具有特定功能的一段代码。

函数的优点：

+ 解决代码的重复书写问题
+ 可以将功能的实现者和使用者分开，提高开发效率
+ 增加的代码的可移植性

**函数参数**：

- 形参：形式参数，函数定义处的参数
- 实参：实际参数，函数调用处的参数
- 必传参数：也叫位置参数，前面定义的函数中使用的都是必传参数，调用时的形式必须要与定义的一致。
- 默认参数：也称关键字参数，就是有默认值的参数，必须放在最后
- 可变长度参数：函数调用时传递必定义时的参数要多，多出来的参数会保存在`args`和`kwargs`中

`*args与**kwargs`的区别：

+ 用`*args`和`**kwargs`只是为了方便并没有强制使用它们。
+ `*args`接受多余的位置参数，转化程元组`tuple`形式，`**kwargs`接受N个关键字参数，转换成字典`dict`形式

*的特殊用法：

```python
 def show(a, b):
      print(a, b)

  l = [1, 2]
  # 略显啰嗦
  # show(l[0], l[1])
  # 精简方法
  show(*l)


  def show2(a='aa', b='bb'):
      print(a, b)

  d = {'a':'apple', 'b':'banana'}

  # show2(a=d['a'], b=d['b'])
  # 上面方式的简单书写方法
  show2(**d)
```

位置参数：调用参数时比较灵活，加不加关键字都可以，但python3.8引入新的语法控制这种灵活性，`'/'`之前的参数必须不加关键字，`'*'`之后的参数必须加关键字，其他参数依旧随意。

```python
# python3.8之前定义函数
def center(txt, border='=', width=50):
    return f'{txt}'.center(width, border)

# 调用结果：
>>>center('python')
'=========================python========================'
>>>center('python', '*', 10)
'**python**'
>>>center('python', border='*', width=10)
'**python**'
>>>center(txt='python', border='*', width=10)
'**python**'

# python3.8新特性：
def center(txt, /, border='=', *, width=50):
     return f'{txt}'.center(width, border)
# 调用结果
>>> center(txt = '中心',border='$',width = 10)
Traceback (most recent call last):
  File "<pyshell#68>", line 1, in <module>
    center(txt = '中心',border='$',width = 10)
TypeError: center() got some positional-only arguments passed as keyword arguments: 'txt'
>>>center( '中心','$', 10)
Traceback (most recent call last):
  File "<pyshell#69>", line 1, in <module>
    center( '中心','$', 10)
TypeError: center() takes from 1 to 2 positional arguments but 3 were given
>>> center('中心',border='$',width = 10)
'$$$$中心$$$$'
>>> center('中心','$',width = 10)
'$$$$中心$$$$'
```

### 3.2. 变量作用域（LEGB）

LEGB规则：local > enclosed > global > bulit-in

+ （1）**块级作用域**：Python不存在块级作用域，下民的代码可以正常运行。

  ```python
  if True:
      name = 'xiaoming'
  
  print(name)
  ```

+ （2）**局部作用域**：定义在函数内部的变量叫局部变量，只能在函数内部使用。

  ```python
  def test():
      a = 1
  
  test()
  # 此处会报错，提示变量未定义
  # print(a)
  ```

+ （3）**全局作用域**：定义在函数外部的变量叫全局变量，哪里都可以使用(但是不能修改，除非使用`global`)

  ```python
  a = 10
  def test():
      # 加上这句代码，就可以在函数内部修改全局变量
      global a
      a = 1
  
  test()
  print(a)
  ```

+ （4）**闭包函数外的函数中**：

  ```python
  def wai():
        age = 20
        def nei():
            # 使用外层函数的局部变量，加上nonlocal姐可以修改外部函数的局部变量
            nonlocal age
            age = 30
            print(age)
        nei()
        print(age)
  
    wai()
  ```

### 3.3. 内置函数、模块函数及基本数据结构函数

![内置函数、模块函数及基本数据结构函数](https://github.com/lajos182/python-essay/blob/master/images/builtin%2C%20module%2C%20str%2C%20list%2C%20tuple%2C%20set%2C%20dict.png)

### 3.4. 匿名函数

**匿名函数`lambda`**：就是没有名字的函数，使用`lambda`关键字定义。

特点：

- 以lambda开头
- 后面跟上该匿名函数的参数，多个参数使用逗号隔开
- 最后一个参数的后面跟上冒号':'
- 冒号的后面跟上一个表达式，这个表达式就是返回值，不需要使用`return`

### 3.5. 递归函数

**递归函数**： 函数内部调用函数本身的函数叫递归函数

特点：

- 代码简洁
- 可读性差(不易理解)
- 瞬间占用内存较大，没有终止条件会立即崩溃
- 有些领域禁止使用(安全领域：汽车电子)
- 只有在不得不使用的时候再使用(目录操作)

斐波那契数列(1,1,2,3,5,8,13,21,34,...)

```python
def fibonacci(n):
    if n == 1 or n == 2:
        return 1
    return fibonacci(n-2) + fibonacci(n-1)

print(fibonacci(6))
```

### 3.6. 闭包与装饰器

**闭包**：外部函数中定义一个内部函数，内部函数中使用了外部函数的变量，外部函数将内部函数作为返回值返回。提高了代码的可重复使用性.

```python
def wai(n):
    def nei():
        return n*n
    return nei

f1 = wai(3)
print(f1())
```

**装饰器**：本质就是一个函数，该函数接受一个函数作为参数，返回一个闭包，而且闭包中执行传递进来的函数，闭包中可以在函数执行的前后添加额外的功能。

**装饰器的作用就是为已经存在的对象添加额外的功能**。

```python
def zhuangshiqi(func):
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs) + 10
    return wrapper

@zhuangshiqi
def pingfang(n):
    return n * n

print(pingfang(4))
```

### 3.7. 迭代器与生成器

+ **迭代器(Iterator)**：可以使用`for-in`进行遍历，并且可以使用next依次获取元素的对象。

  可以使用`isinstance`判断是否是迭代器。

  ```python
  from collections import Iterator
  
  l = (i for i in range(10))
  print(isinstance(l, Iterator))
  ```

+ **生成器(generator)**：为了解决内存突然变大的问题，Python引入了生成器。它是一种他叔的迭代器。

  + 产生条件：

    + 将列表生成式中的`[]`改为`()`

      ```python
      # 数据量特别大时，会造成内存占用突然增大
      # l2 = [i for i in range(10000)]
      # 生成器
      l2 = (i for i in range(2))
      # 使用next获取生成器中值，一次一个，遍历结束会报错StopIteration
      print(next(l2))
      ```

    + 通常在函数中使用`yield`关键字

      ```python
      def test(n):
      	for i in range(1, n+1):
            	yield i
      ```

  + 特性

    - 可以使用`next`获取数据，一次一个，结束时会报错
    - 只能遍历一遍
    - 可以转换为列表
    - 可以使用`for-in`遍历

### 3.8. 高级函数

+ **可迭代对象**：可以使用`for-in`遍历的对象，我们都称之为可迭代对象。字符串、列表、元组、集合、字典等都不是迭代器，他们都是可迭代对象。

可以使用`isinstance`判断是否是可迭代对象：

```python
from collections import Iterable
print(isinstance(l, Iterable))
print(isinstance(lt, Iterable))
```

+ **高级函数map**：格式`map(func, lt)`，接受两个参数，一个函数和一个可迭代对象，返回一个生成器，将`func`依次作用于`lt`

  ```python
  l = [1,2,3,4,5]
  m = map(lambda x:x*x, l) # 迭代器
  ```

+ **高级函数filter**：格式`filter(funct, lt)`，使用`func`依次作用于每个元素，处理结果为`True`保留下来。

  ```python
  l = [1,2,3,4,5]
  
  # 提取偶数
  f = filter(lambda x:x%2==0, l)
  
  print(list(f))
  ```

+ **高级函数reduce**：格式`reduce(func, lt)`, 接受两个参数，一个函数和一个可迭代对象，首先取两个元素，使用`func`处理，结果和第三个元素继续使用`func`处理，直到结束，返回处理的结果。

  ```python
  from functools import reduce
  
  l = [1,2,3,4,5]
  
  # 求和
  print(reduce(lambda x, y:x+y, l))
  
  # 转换为12345
  print(reduce(lambda x, y:x*10+y, l))
  ```

+ **高级函数sorted**：格式`sorted(func, lt)`, 内建函数，用于对可迭代快递的每个元素进行排序，生成新的对象。

  ```python
  l = [
      {'name':'xiaowang', 'age':18,'height':150},
      {'name':'xiaogang', 'age':20,'height':140},
      {'name':'xiaohong', 'age':19,'height':145},
  ]
  
  l2 = sorted(l, key=lambda x:x['age'], reverse=True)
  
  print(l2)
  ```

  注意：`sort`也可用来排序，它是`list`的排序方法，用于对列表的成员进行排序，而且改变的是原列表。

### 3.9. 匿名、递归、闭包、高级函数总结

![ 匿名、递归、闭包、高级函数总结](https://github.com/lajos182/python-essay/blob/master/images/recursive%2C%20lamada%2C%20closure%2C%20iteator%2C%20decorator%2C%20generator%2C%20iterable.png)

### 3.10.目录管理与文件操作相关模块函数

+ **目录管理(`os`)**

  ```python
  # system, 执行系统命令
  os.system('cls')
  
  # name, 获取操作系统名称，nt代表Windows, posix代表unix
  print(os.name)
  
  # environ, 获取环境变量
  os.environ.get('path')
  os.environ.get(['path'])
  os.getenv('path')
  
  # getcwd, 获取当前工作目录
  os.getcwd()
  
  # mkdir, 创建目录，该方法创建中间目录会报错
  os.mkdir('hello')
  
  # makedirs, 创建目录，会创建中间目录
  os.makedirs('a/b/c')
  
  # rmdir, 删除目录，只能删除空目录
  os.rmdir('a')
  
  # rename, 修改文件名(可以是目录)
  os.rename('a', 'c')
  
  # stat, 查看文件信息
  os.stat('123.py')
  
  # listdir, 列出直接子文件
  os.listdir('c')
  ```

  `path`函数

  ```python
  from os import path
  # 提取文件后缀（切割文件名与后缀）
  name, ext = path.splitext('789.py')
  print(name, ext)
  
  # 提取目录名（最后一个目录分隔符的前面内容）
  print(path.dirname('123/456/789.py'))
  
  # 提取文件名(包括后缀)
  print(path.basename('123/456/789.py'))
  
  # 切割文件名和目录
  print(path.split('123/456/789.py'))
  
  # 判断文件是否存在（可以是目录）
  print(path.exists('123.py'))
  
  # 判断是否是目录文件
  print(path.isdir('c'))
  
  # 判断是否是普通文件
  print(path.isfile('123.py'))
  
  # 获取普通文件大小
  print(path.getsize('01-os.py'))
  # 不可以获取目录大小，始终是0
  print(path.getsize('c'))
  ```

+ **文件管理**

  + 打开文件（`open`）：`fp = open('00-test.txt', 'r')`

    ```python
    # 参数1：文件路径名；参数2：打开方式；参数3：编码格式，一般不用指定，系统会自动识别处理
    # 打开方式：
    r：只读方式，文件不存在会报错
    w：只写方式，文件不存在创建文件，文件存在清空内容
    a：追加方式，文件不存在则创建，文件存在直接打开(不会清空内容)，只能向最后追加内容
    r+：在r方式下添加写的功能
    w+：在w方式下添加读的功能
    a+：在a方式下添加读的功能
    
    在上面的模式上添加b，表示二进制方式打开：rb、wb、ab、rb+、wb+、ab+
    1.文件的读写数据全部是bytes类型，没有添加b的方式全部是str类型
    
    # 编码方式
    ascii：美国信息交换标准代码
    ansi：扩展的ascii
    gb2312：中国的ansi
    gbk：扩展的gb2312
    
    unicode：是一套理论，实现方式不限
    utf-8：可变长度的unicode实现，对中文的支持比较友好
    ```

  + 关闭文件（`close`）：`fp.close()`

  + 文件读写

    ```python
    # 读取指定长度内容
    # ret = fp.read(3)
    # 写入内容
    fp.write('hello')
    
    # 读取一行，包括换行符
    print(fp.readline())
    
    # 读取所有行，返回一个列表
    print(fp.readlines())
    
    # 是否可读
    print(fp.readable())
    
    # 是否可写
    print(fp.writable())
    ```

  + **`read`、`readline`和`readlines`的区别，如何读取大文件**

    > **`read()方法`**：读取整个文件，将文件内容放到一个字符串变量中，如果需要对文件按行进行处理，则不可用该方法。如果文件大于可用内存(好几个G的)，不可能使用这种处理，系统会报错：`MemoryError`
    >
    > **`readline()方法`**： 读取下一行，每只读取文件的一行，通常也是读取到的一行内容放到一个字符串变量中，返回`str`类型。内存不够时使用，一般不太用。
    >
    > **`readlines()方法`**：**读取整个文件所有行，保存在一个列表(`list`)变量中，每行作为一个元素，但读取大文件会比较占内存。**
    >
    > **大文件读取数据**：处理大文件是很容易想到的就是将大文件分割成若干小文件处理，处理完每个小文件后释放该部分内存（分块读取）。这里用了**`iter` & `yield`**

    + 分块读取（`yield`）：将大文件分割成若干小文件处理，处理完每个小文件后释放该部分内存

      ```python
      # 分块读取
      def read_chunk(file_name, trunk_size):
          fp = open(file_name, 'rb') # 创建文件对象，也是一个可迭代对象
          # while True:
          #     content = fp.read(trunk_size)
          #     if not content:
          #         break
          #     yield content
          content = fp.read(trunk_size)
          while content:
              content = fp.read(trunk_size)
              yield content
            
      if __name__ == '__main__':
          file_name = r'F:\AngularJS学习资料\第一章  AngularJS教程.md'
          for i in read_chunk(file_name, 50):
              print(i)
      ```

    + 使用`with`（操作可迭代对象）：对可迭代对象 `fp`，进行迭代遍历：`for line in fp`，会自动地使用缓冲`IO`（`buffered IO`）以及内存管理，而不必担心任何大文件的问题。

      ```python
      with open(r'F:\AngularJS学习资料\第一章  AngularJS教程.md', 'rb') as fp:
          for line in fp:
              print(line)
      ```


  + 文件指针

    ```python
    # 返回文件指针的操作位置
    print(fp.tell())
    
    # 设置偏移
    # 参数1：偏移量
    # 参数2：参考位置，0：开头，1：当前，2：末尾
    # 定位到末尾
    fp.seek(0, 2)
    ```

    > 带`b`的方式`seek`没有异常；不带`b`的时候，相对于当前位置无法偏移，相对于末尾只能偏移0

  + 文件删除

    ```python
    import os
    os.remove('02-test.py')
    ```

+ **文件高级操作`shutil`**:

  + **`copyfileobj(fsrc, fdst, length=16*1024)`**：将`fsrc`文件内容复制至`fdst`文件，`length`为`fsrc`每次读取的长度，用做缓冲区大小

    ```python
    import shutil
    fp1 = open('file.txt', 'r')
    fp2 = open('file_copy.txt', 'a+')
    shutil.copyfileobj(fp1, fp2, length=1024)
    ```

  + **`copyfile(src, dst)`**：将`src`文件内容复制至`dst`文件，若`dst`文件不存在，将会生成一个`dst`文件；若存在将会被覆盖

    ```python
    import shutil
    shutil.copyfile("file.txt","file_copy.txt")
    ```

  + **`copymode(src, dst, follow_symlinks=True)`**：将`src`文件权限复制至`dst`文件，文件内容，所有者和组不受影响。`dst`路径必须是真实的路径，并且文件必须存在，否则将会报文件找不到错误。`follow_symlinks`为`Python3`新增参数，设置为`False`时，`src`, `dst`皆为软连接，可以复制软连接权限，如果设置为`True`，则当成普通文件复制权限。默认为`True`。

    ```python
    import shutil
    shutil.copymode("file.txt","file_copy.txt")
    ```

  + **`copystat(src, dst, follow_symlinks=True)`**：将权限，上次访问时间，上次修改时间以及`src`的标志复制到`dst`。文件内容，所有者和组不受影响。`follow_symlinks`设置为`False`时，`src`, `dst`皆为软连接，可以复制软连接权限、上次访问时间，上次修改时间以及`src`的标志，如果设置为`True`，则当成普通文件复制权限。默认为`True`。

    ```python
    import shutil
    shutil.copystat("file.txt","file_copy.txt")
    ```

  + **`copy(src, dst, follow_symlinks=True)`**： 将文件`src`复制至`dst`。`dst`可以是个目录，会在该目录下创建与`src`同名的文件，若该目录下存在同名文件，将会报错提示已经存在同名文件。权限会被一并复制。本质是先后调用了`copyfile`与`copymode`而已。`follow_symlinks`和上面作用相同。

    ```python
    improt shutil,os
    shutil.copy("file.txt","file_copy.txt")
    # 或者
    shutil.copy("file.txt",os.path.join(os.getcwd(),"copy"))
    ```

  + **`copy2(src, dst, follow_symlinks=True)`**：将文件`src`复制至`dst`。`dst`可以是个目录，会在该目录下创建与`src`同名的文件，若该目录下存在同名文件，将会报错提示已经存在同名文件。权限、上次访问时间、上次修改时间和`src`的标志会一并复制至`dst`。本质是先后调用了`copyfile`与`copystat`方法而已。`follow_symlinks`和上面作用相同。

    ```python
    improt shutil,os
    shutil.copy2("file.txt","file_copy.txt")
    # 或者
    shutil.copy2("file.txt",os.path.join(os.getcwd(),"copy"))
    ```

  + **`ignore_patterns(*patterns)`**：忽略模式，用于配合`copytree()`方法，传递文件将会被忽略，不会被拷贝。`patterns`是文件名称、元组。

  + **`copytree(src, dst, symlinks=False, ignore=None, `copy_function=copy2,
                 ignore_dangling_symlinks=False`)`**：拷贝文档树，将`src`文件夹里的所有内容拷贝至`dst`文件夹，文件夹不存在会自动创建。`symlinks`代表是否复制软连接，`True`复制软连接，`False`不复制，软连接会被当成文件复制过来，默认`False`。`ignore`为忽略模式，可传入`ignore_patterns()`。`copy_function`为拷贝文件的方式，可传入一个可执行的处理函数，默认为`copy2`，`ignore_dangling_symlinks`设置为`False`，拷贝指向文件已删除的软连接时，将会报错，如果想消除这个异常，可以设置此值为True，默认为`False`。

    ```python
    import shutil,os
    folder1 = os.path.join(os.getcwd(),"aaa")
    # bbb与ccc文件夹都可以不存在,会自动创建
    folder2 = os.path.join(os.getcwd(),"bbb","ccc")
    # 将"abc.txt","bcd.txt"忽略，不复制
    shutil.copytree(folder1,folder2,ignore=shutil.ignore_patterns("abc.txt","bcd.txt")
    ```

  + **`rmtree(path, ignore_errors=False, onerror=None)`**：移除文档树，将文件夹目录删除。`ignore_errors`代表是否忽略错误，默认`False`，`onerror`定义错误处理函数，需传递一个可执行的处理函数，该处理函数接收三个参数：函数、路径和`excinfo`

    ```python
    import shutil,os
    folder1 = os.path.join(os.getcwd(),"aaa")
    shutil.rmtree(folder1)
    ```

  + **`move(src, dst, copy_function=copy2)`**：将`src`移动至`dst`目录下。若`dst`目录不存在，则效果等同于`src`改名为`dst`。若`dst`目录存在，将会把`src`文件夹的所有内容移动至该目录下面。`copy_function`为拷贝文件的方式，可传入一个可执行的处理函数，默认为`copy2`。

    ```python
    import shutil,os
    # 示例一，将src文件夹移动至dst文件夹下面，如果bbb文件夹不存在，则变成了重命名操作
    folder1 = os.path.join(os.getcwd(),"aaa")
    folder2 = os.path.join(os.getcwd(),"bbb")
    shutil.move(folder1, folder2)
    # 示例二，将src文件移动至dst文件夹下面，如果bbb文件夹不存在，则变成了重命名操作
    file1 = os.path.join(os.getcwd(),"aaa.txt")
    folder2 = os.path.join(os.getcwd(),"bbb")
    shutil.move(file1, folder2)
    # 示例三，将src文件重命名为dst文件(dst文件存在，将会覆盖)
    file1 = os.path.join(os.getcwd(),"aaa.txt")
    file2 = os.path.join(os.getcwd(),"bbb.txt")
    shutil.move(file1, file2)
    ```

  + **`disk_usage(path)`**：获取当前目录所在硬盘使用情况。`path`表示文件夹或文件路径，`windows`中必须是文件夹路径，`linux`中可以是文件路径和文件夹路径。

    ```python
    import shutil.os
    path = os.path.join(os.getcwd(),"aaa")
    info = shutil.disk_usage(path)
    print(info)   # usage(total=95089164288, used=7953104896, free=87136059392)
    ```

  + **`chown(path, user=None, group=None)`**：修改路径指向的文件或文件夹的所有者或分组。

    ```python
    import shutil,os
    path = os.path.join(os.getcwd(),"file.txt")
    shutil.chown(path, user="root", group="root")
    ```

  + **`which(cmd, mode=os.F_OK | os.X_OK, path=None)`**： 获取给定的`cmd`命令的可执行文件的路径。

    ```python
    import shutil
    print(shutil.which('pip'))  # D:\Python\Python36\Scripts\pip.EXE
    ```

  + **`make_archive(base_name, format, root_dir, …)`**：生成压缩文件。`base_name`指压缩文件的文件名，不允许有扩展名(因为会根据压缩格式生成相应的扩展名)，`format`表示压缩格式，`root_dir`表示将指定文件夹进行压缩。

    ```python
    import shutil,os
    base_name = os.path.join(os.getcwd(),"aaa")
    format = "zip"
    root_dir = os.path.join(os.getcwd(),"aaa")
    # 将会root_dir文件夹下的内容进行压缩，生成一个aaa.zip文件
    shutil.make_archive(base_name, format, root_dir)
    ```

  + **`get_archive_formats()`**：获取支持的压缩文件格式。目前支持的有：`tar`、`zip`、`gztar`、`bztar`。在Python3还多支持一种格式`xztar`。

  + **`unpack_archive(filename, extract_dir=None, format=None)`**：解压操作，`extract_dir`解压至的文件夹路径，文件夹可以不存在，会自动生成。`format`解压格式，默认为`None`，会根据扩展名自动选择解压格式。

    ```python
    import shutil,os
    zip_path = os.path.join(os.getcwd(),"aaa.zip")
    extract_dir = os.path.join(os.getcwd(),"aaa")
    shutil.unpack_archive(zip_path, extract_dir)
    ```

  + **`get_unpack_formats()`**：获取支持的解压文件格式。目前支持的有：`tar`、`zip`、`gztar`、`bztar`和`xztar`。

![目录及文件管理](https://github.com/lajos182/python-essay/blob/master/images/os%20and%20shutil.png)

+ **经典练习**

  + 实现一个拷贝文件的功能，提醒：要考虑超大文件问题，如:依次读取1024字节，循环读取

    ```python
    import os
    def copy_file(source, target):
        if not os.path.exists(source):
            print('源文件不存在！')
            return
        if os.path.isdir(source):
            print('源文件是目录文件，无法拷贝！')
            return
        if os.path.abspath(source) == os.path.abspath(target):
            print('复制的地址与源文件相同，无法进行复制！')
            return 
        if os.path.isdir(target):
            target = os.path.join(target, os.path.basename(source))
        source_fp = open(source, 'rb')
        target_fp = open(target, 'wb')
        content = source_fp.read(1024)
        while content:
            target_fp.write(content)
            content = source_fp.read(1024)
        source_fp.close()
        target_fp.close()
    ```

  + 递归拷贝一个文件夹

    ```python
    def copy_dir(source, target):
        if not os.path.exists(source):
            print('原文件不存在！')
            return
        if os.path.isfile(target):
            print('源文件不是目录文件！')
        if os.path.isfile(target):
            print('目标地址不是目录，无法进行拷贝')
            return
        if os.path.abspath(source) == os.path.abspath(target):
            print('复制的地址与源文件相同，无法进行复制')
            return
        if not os.path.exists(target):
            os.makedirs(target)
        source_list = os.listdir(source)
        for item in source_list:
            source_name = os.path.join(source, item)
            target_name = os.path.join(target, item)
            if os.path.isfile(source_name):
                copy_file(source_name, target_name) # 拷贝单个文件
            else:
                copy_dir(source_name, target_name) # 递归拷贝
     copy_dir('a', 'b')         
    ```

  + 递归删除一个文件夹(或文件)

    ```python
    import os
    def delete_dir(source):
        if not os.path.exists(source):
            print('要删除的文件不存在！')
            return
        if os.path.isfile(source):
            os.remove(source)
        source_list = os.listdir(source)
        for item in source_list:
            source_path = os.path.join(source, item)
            if os.path.isfile(source_path):
                os.remove(source_path)
            else:
                delete_dir(source_path)
        os.rmdir(source)
    delete_dir('../ccc')
    ```

  + 递归统计一个文件夹的大小

    ```python
    import os
    def get_size(source):
        size = 0
        if not os.path.exists(source):
            print('当前文件(夹)不存在！')
            return
        if os.path.isfile(source):
            size += os.path.getsize(source)
            return size
        source_list = os.listdir(source)
        for item in source_list:
            source_path = os.path.join(source, os.path.basename(item))
            if os.path.isfile(source_path):
                size += os.path.getsize(source_path)
            else:
                size += get_size(source_path)
        return size
    
    print(get_size('a_a.py'))
    ```

  + 移动文件夹

    ```python
    def move_dir(source, target):
        if not os.path.exists(source):
            print('不存在该文件，无法移动')
            return
        if os.path.abspath(source) == os.path.abspath(target):
            print('源文件地址与指定的地址相同，无法进行移动')
            return
        if os.path.isfile(source):
            source_fp = open(source, 'r')
            target_fp = open(target, 'w')
            content = source_fp.read(1024)
            while content:
                target_fp.write(content)
                content = source_fp.read(1024)
            source_fp.close()
            target_fp.close()
            os.remove(source)
        else:
            if not os.path.exists(target):
                os.makedirs(target)
            source_list = os.listdir(source)
            for item in source_list:
                source_item = os.path.join(source, item)
                target_item = os.path.join(target, item)
                move_dir(source_item, target_item)
            os.rmdir(source)
    
    move_dir('a', 'c')
    ```

  + 目录整理

    ```python
    # 将目录按后缀进行分类，后缀相同的放到后缀字母大写的文件中，没有后缀的放置在'OTHERS'中，文件夹放在'DIRS'中
    import os, shutil
    def clean_dir(source):
        if os.path.isfile(source) or not os.path.exists(source):
            print('当前目录无法整理，请查询你要整理的目录！') 
            return
        source_list = os.listdir(source)
        for item in source_list:
            source_path = os.path.join(source, item)
            if os.path.isfile(source_path):
                suffix = source_path.rsplit('.', 1)
                if len(suffix) == 2:
                    SUFFIX = os.path.join(source, suffix[1].upper())
                    if not SUFFIX:
                        os.mkdir(SUFFIX)
                    shutil.move(source_path, SUFFIX)
                else:
                    OTHERS = os.path.join(source, 'OTHERS')
                    if not OTHERS:
                        os.mkdir(OTHERS)
                    shutil.move(source_path, OTHERS)
            else:
                DIRS = os.path.join(source, 'DIR')
                if not DIRS:
                    os.mkdir(DIRS)
                shutil.move(source_path, DIRS)
    ```

### 3.11. 时间、日期和日历相关模块函数

+ **`time`模块**：

  + `sleep`：休眠指定的秒数(可以是小数)

  + `localtime`： 将一个时间戳转换为`time.struct_time`类型的对象(类似于元组)

    ```python
    import time
    print(time.localtime())
    >>> time.struct_time(tm_year=2019, tm_mon=4, tm_mday=28, tm_hour=9, tm_min=25, tm_sec=34, tm_wday=6, tm_yday=118, tm_isdst=0)
    ```

    > 年、月、日、时、分、秒、星期(0~6)、今年的第几天、夏令时

  + `mktime`：根据元组形式的时间生成一���时间戳

    ```python
    import time
    print(time.mktime((2019, 4, 28, 9, 25, 34, 6, 118, 0))) # 注意一定是一个元组
    >>> 1556414734.0
    ```

  + `strftime`：将一个元组形式的时间格式化为字符串，不传时间默认转换当前时间

    ```python
    time.strftime('%Y-%m-%d %H:%M:%S %w %W', local_time)
    >>> '2019-04-28 09:34:53 0 16'
    # 注意与datetime.datetime.strftime的区别，datetime.datetime.strftime(time, '%Y-%m-%d %H:%M:%S:%f')
    ```

  + `gmtime`：将一个时间戳转换为元组形式，不传默认转换当前时间

    ```python
    print(time.gmtime())
    # time.struct_time(tm_year=2019, tm_mon=4, tm_mday=28, tm_hour=1, tm_min=50, tm_sec=27, tm_wday=6, tm_yday=118, tm_isdst=0)
    ```

  + `asctime`：将一个元组形式的时间转换为标准格式字符串，不传参数转换当前时间

    ```python
    print(time.asctime())
    # 'Sun Apr 28 09:49:13 2019'
    ```

  + `timezone`：0时区减去当前时区的秒数

    ```python
    print(time.timezone)  #  -28800
    ```

+ **`datetime`模块**：

  + `date()`：获取一定格式的时间，`print(date(2019, 5. 3)), # 2019-05-03`

    + `d = date.today()`：获取今天的时间
    + `d.fromtimestamp()`：将时间戳转化为日期， `print(d.fromtimestamp(time.time()))`

    + `d.isoformat()`：标准格式字符
    + `d.isocalendar()`：日历显示形式，如(年，第几周， 星期几)
    + `d.isoweekday()`：获取星期(1~7)
    + `d.weekday()`: 获取星期(0~6)
    + `d.strftime()`：格式化，`print(d.strftime('%y-%m-%d'))`
    + `d.timetuple()`：转化为元组

  + `time()`：获取一定格式的时间，`tm = time()`
    + `tm.hour`：获取小时
    + `tm.minute`：获取分钟
    + `tm.second`：获取秒数
    + `tm.strftime()`：时间格式化，`print(tm.strftime('%H:%M:%S'))`

  + `datetime`：
    + `datetime.now()`：获取本地时间
    + `datetime.utcnow()`：获取UTC时间(格林尼治时间)
    + `datetime.fromtimestamp()`：将一个时间戳转化为`datetime`
    + `datetime.fromtimestamp.strftime('%Y-%m-%d %H:%M:%S')`：格式化显示

  + `timedelta()`:
    + `delta = timedelta()`：获取时间差，`print(timedelta(days=2, hours=2, seconds=56))`
    + `delta.days`：获取天数
    + `delta.seconds`：获取天数外的秒数
    + `delta.total_seconds()`：获取总秒数

+ **`calendar`模块**：

  + `calendar`：获取某一年的日历

    ```python
    # 获取某一年的日历, 参数：年份， w =列宽， l= 行宽，m = 控制每行打印月份数， c=每个月份之间的间距
    c = calendar.calendar(2018, w=3, l=2, m=2, c=10)
    ```

  + `month`：获取指定年指定月份的日历：

    ```python
    # 获取指定年指定月的日历
    m = calendar.month(2018, 3)
    ```

  + `isleap`：判断是否是闰年

    ```python
    print(calendar.isleap(2062))
    ```

  + `leapdays`：判断两个年份之间的瑞年的个数([起始，结束))

    ```python
    print(calendar.leapdays(1992, 2018))
    ```


![时间、日期、日历](https://github.com/lajos182/python-essay/blob/master/images/time%2C%20calendar%20and%20datemate.png)

### 3.12. 短信邮件相关模块函数

+ **`hashlib`**：用来进行`hash`或者`md5`加密，而且这种加密是不可逆的，所以这种算法又被称为摘要算法。其支持`Openssl`库提供的所有算法，包括`md5`、`sha1`、`sha224`、`sha256`、`sha512`等。

  + `md5()`：创建一个`md5`加密模式的`hash`对象，可以指定需要加密的字符串

  + `update()`：设置加密字符串，创建`md5`对象就不必指定了，不能两个地方都指定

  + `hexdigest()`：获取加密后的摘要(32位)

    ```python
    import hashlib
    md = hashlib.md5() 
    md.update('123456'.encode('utf-8'))
    print(md.hexdigest()) # e10adc3949ba59abbe56e057f20f883e
    
    md = hashlib.md5('123456'.encode('utf-8')) # 如果添加参数，就相当于多进一层加密
    md.update('123456'.encode('utf-8'))
    print(md.hexdigest()) # ea48576f30be1669971699c09ad05c94
    
    ```

+ **`urlib`**：提供了一系列用于操作`URL`的功能，是`Python`的标准库

  + `request`：发送`http`请求

    + `urlopen()`：访问目标网址，结果是一个`http.client.HTTPResponse`对象，默认访问方法是`GET`，当在该方法中传入`data`参数时，则会发起`POST`请求。

      ```markdown
      urlopen(url, data=None, [timeout, ]*, cafile=None, capath=None, cadefault=False, context=None)
      ```

      ```python
      from urllib import request
      rep = request.urlopen('http://www.baidu.com')
      print(rep.read().decode('utf-8'))
      
      # urlopen返回对象提供方法：
      　　# read() , readline() ,readlines() , fileno() , close() ：对HTTPResponse类型数据进行操作。
      　　# info()：返回HTTPMessage对象，表示远程服务器返回的头信息。
      　　# getcode()：返回Http状态码。
      　　geturl()：返回请求的url。
      ```

      ```python
      from urllib import request
      # 设置timeout，如果请求时间超出，那么就会抛出异常
      resp = request.urlopen('http://httpbin.org', data='word=hello'.encode('utf-8'), timeout=10)
      print(resp.read().decode('utf-8'))
      ```

    + `Reuqest()`：传入的`request`对象，而不是一个`url`

      ```markdown
      urllib.request.Request(url, data=None, headers={}, origin_req_host=None, unverifiable=False, method=None)
      
      url--访问的地址。
      data--此参数为可选字段，其中传递的参数需要转为bytes，如果是字典我们只需要通过urlencode转换即可。
      headers--http相应headers传递的信息，构造方法：headers参数传递，通过调用Request对象的add_header()方法来添加请求头。
      origin_req_host--指的是请求方的host名称或者IP地址。
      unverifiable--用来表明这个请求是否是无法验证的，默认是False。意思就是说用户没有足够权限来选择接收这个请求的结果。如果没有权限，这时unverifiable的值就是True 。
      method--用来指示请求使用的方法，比如GET，POST，PUT等
      ```

      ```python
      # 使用urllib模拟微博登录
      from urllib import request
      from urllib.parse import urlencode
      print('Login to weibo.cn...')
      email = input('Email: ')
      passwd = input('Password: ')
      login_data = urlencode([
          ('username', email),
          ('password', passwd),
          ('entry', 'mweibo'),
          ('client_id', ''),
          ('savestate', '1'),
          ('ec', ''),
          ('pagerefer', 'https://passport.weibo.cn/signin/welcome?entry=mweibo&r=http%3A%2F%2Fm.weibo.cn%2F')
      ])
      req = request.Request('https://passport.weibo.cn/sso/login')
      req.add_header('Origin', 'https://passport.weibo.cn')
      req.add_header('User-Agent', 'Mozilla/6.0 (iPhone; CPU iPhone OS 8_0 like Mac OS X) AppleWebKit/536.26 (KHTML, like Gecko) Version/8.0 Mobile/10A5376e Safari/8536.25')
      req.add_header('Referer', 'https://passport.weibo.cn/signin/login?entry=mweibo&res=wel&wm=3349&r=http%3A%2F%2Fm.weibo.cn%2F')
      rep = request.urlopen(req, data=login_data.encode('utf-8'))
      print(rep.read().decode('utf-8'))
      ```

    + 添加`Cookie`：使用`request.build_opener`方法来进行构造`opener`

      ```python
      from http import cookiejar
      from urllib import request
      
      url = 'http://httpbin.org/cookies'
      # 创建一个cookiejar对象
      cookie = cookiejar.CookieJar()
      # 使用HTTPCookieProcessor创建cookie处理器
      cookies = request.HTTPCookieProcessor(cookie)
      # 并以它为参数创建Opener对象
      opener = request.build_opener(cookies)
      # 使用这个opener来发起请求
      resp = opener.open(url)
      print(resp.read().decode())
      ```

      或者也可以把这个生成的`opener`使用`install_opener`方法来设置为全局的。则之后使用`urlopen`方法发起请求时，都会带上这个`cookie`。

      ```python
      # 将这个opener设置为全局的opener
      request.install_opener(opener)
      resp = request.urlopen(url)
      ```

    + 设置`Proxy`代理：使用爬虫来爬取数据的时候，常常需要使用代理来隐藏我们的真实`IP`。

      ```python
      from urllib import request
      
      url = 'http://httpbin.org/ip'
      proxy = {'http':'50.233.137.33:80','https':'50.233.137.33:80'}
      # 创建代理处理器
      proxies = requet.ProxyHandler(proxy)
      # 创建opener对象
      opener = request.build_opener(proxies)
      resp = opener.open(url)
      print(resp.read().decode())
      ```

  + `error`：处理请求过程中出现的异常

    用 try-except来捕捉异常,主要的错误方式就两种` URLError`（错误信息）和`HTTPError`(错误编码)。

    ```python
    from urllib import request, error
    try:
        data= request.urlopen(url)
        print(data.read().decode('utf-8'))
    except error.HTTPError as e:  # 错误编码
        print(e.code)
    except error.URLError as e:   # 错误信息
        print(e.reason)
    ```

  + `parse`：解析`url`

    + `urlencode()`将字典转化为`key1=value1&key2=value2`格式

      ```python
      from urllib.parse import urlencode
      data = {
          'name': 'lajos',
          'age': 18
      }
      print(urlencode(data))  # name=lajos&age=18
      ```

    + `urlparse()`：解析`url`中的所有参数，得到的是`urllib.parse.ParseResult`对象

    + `parse_qs()`：将`url`请求参数转化为字典

      ```python
      from urllib.parse import parse_qs, urlparse
      url = 'http://www.baidu.com/abc/def?name=lajos&age=18'
      url_parse = urlparse(url)
      print(url_parse) # ParseResult(scheme='http', netloc='www.baidu.com', path='/abc/def', params='', query='name=lajos&age=18', fragment='')
      print(url_parse.query) # name=lajos&age=18
      print(parse_qs(url_parse.query)) # {'name': ['lajos'], 'age': ['18']}
      ```

  + `robotparser`：解析`robots.txt`文件

+ **`http.client`**：网络请求

  ```python
  import http.client
  
  # 创建连接(相当于浏览器)
  connect = http.client.HTTPConnection('www.baidu.com')
  # 发送请求(GET/POST)
  connect.request(method='GET', url='http://www.baidu.com')
  # 获取响应
  resp = connect.getresponse()
  # 打印响应内容，读取并解码
  print(resp.read().decode('utf-8'))
  ```

+ **`邮件发送smtplib`**：提供了一种很方便的途径发送电子邮件。它对smtp协议进行了简单的封装。

  + `SMTP()`:（`Simple Mail Transfer Protocol`)即简单邮件传输协议,它是一组用于由源地址到目的地址传送邮件的规则，由它来控制信件的中转方式。

    ```python
    import smtplib
    smtp_obj = smtplib.SMTP([host, [, port [, local_hostname]]])
    # host: SMTP 服务器主机。 你可以指定主机的ip地址或者域名如: runoob.com，这个是可选参数。
    # port: 如果你提供了host参数, 你需要指定SMTP服务使用的端口号，一般情况下SMTP端口号为25。
    # local_hostname: 如果 SMTP 在你的本机上，你只需要指定服务器地址为 localhost 即可。
    ```

  + `SMTP.sendmail()`：发送邮件

    ```python
    SMTP.sendmail(from_addr, to_addrs, msg[, mail_options, rcpt_options])
    # from_addr: 邮件发送者地址。
    # to_addrs: 字符串列表，邮件发送地址。
    # msg: 发送消息，格式是字符串，一般包含标题，发信人，收件人，邮件内容，附件等
    ```

  ```python
  import smtplib
  from email.mime.text import MIMEText
  from email.header import Header
  from email.mime.multipart import MIMEMultipart
  import os
  
  # 邮箱服务器
  mail_server  = 'smtp.qq.com'
  # 用户名
  mail_user = '13249732498@qq.com'
  # 密码或授权码，为了不公开可以通过环境变量的方式获取
  mail_password = os.environ.get('MAIL_PASSWORD') or '123456'
  # 邮件消息
  def send_mail(to_addrs):
      try:
          message = '你好，欢迎注册xxx平台，激活请点击右边链接 <a href="http://www.baidu.com">点击激活</a>'
          # 将邮件字符串消息转换邮件格式，若内容是HTML需要指定第二个参数为'html'
          message = MIMEText(message, 'html')
          # 设置主题
          message['Subject'] = Header('账户激活')
          # 设置显示的发件人
          message['From'] = Header(mail_user)
          # 设置显示的收件人，可以给多个人发邮件
          message['To'] = Header(to_addrs)
          # 创建邮件对象
          mail = smtplib.SMTP(mail_server, 25)
          # 登录服务器
          mail.login(mail_user, mail_password)
          # 发送邮件
          mail.sendmail(mail_user, to_addrs, message.as_string())
          mail.quit()
      except Exception:
          ret = False
      return ret
  
  ret = send_mail('1823423231@qq.com, 23423412@163.com')
  if ret:
      print('邮件发送成功！')
  else:
      print('邮件发送失败！')
  ```

  ```python
  import smtplib
  from email.mime.text import MIMEText
  from email.header import Header
  from email.mime.multipart import MIMEMultipart
  import os
  
  # 邮箱服务器
  mail_server  = 'smtp.qq.com'
  # 用户名
  mail_user = '1287174078@qq.com'
  # 密码或授权码，为了不公开可以通过环境变量的方式获取
  mail_password = os.environ.get('MAIL_PASSWORD') or 'qwewqrewr'
  # 邮件消息
  def send_mail(to_addrs):
      try:
          message = MIMEMultipart()
          # 设置主题
          message['Subject'] = Header('账户激活')
          # 设置显示的发送人
          message['From'] = Header(mail_user)
          # 设置显示的收件人，可以给多个人发邮件
          message['To'] = Header(to_addrs)
          body = '这是Python邮件发送测试.....'
          # 邮件正文内容
          message.attach(MIMEText(body, 'plain'))
          # 构造附件1，传送当前目录下的 test.txt 文件
          att1 = MIMEText(open('test1.txt', 'rb').read(), 'base64', 'utf-8')
          att1['Content-Type'] = 'application/octet-stream'
          att1["Content-Disposition"] = 'attachment; filename="test.txt"'
          message.attach(att1)
          # 构造附件2，传送当前目录下的 runoob.txt 文件
          att2 = MIMEText(open('yunda.py', 'rb').read(), 'base64', 'utf-8')
          att2["Content-Type"] = 'application/octet-stream'
          att2["Content-Disposition"] = 'attachment; filename="yunda.py"'
          message.attach(att2)
          # 创建邮件对象
          mail = smtplib.SMTP(mail_server, 25)
          # 登录服务器
          mail.login(mail_user, mail_password)
          
          # 发送邮件
          mail.sendmail(mail_user, to_addrs, message.as_string())
          mail.quit()
      except Exception:
          ret = False
      return ret
  
  ret = send_mail('1825871201@qq.com')
  if ret:
      print('邮件发送成功！')
  else:
      print('邮件发送失败！')
  ```

+ **短信发送**：常使用短信发送平台，如阿里、云之讯、秒滴

  + 该案例使用阿里短信平台对接，需要的参数如下：

    > **安装Python环境的依赖包**：`pip install aliyun-python-sdk-core`，使用于**`python2.6.5`**及其更高版本
    >
    > `accessKeyId`：主账号`AccessKey`的ID
    > `accessSecret`：主账号的密钥
    > `SignName`：短信签名名称
    > `PhoneNumbers`：要发送的电话号码
    > `TemplateCode`：短信模板ID，例如`'SMS_153055065'`
    > `TemplateParam`：短信模板变量对应的实际值，`JSON`格式，如`'{"code": "1234"}'`

    ```python
    # 将对应的参数替换即可进行调用阿里云短信服务平台
    from aliyunsdkcore.client import AcsClient
    from aliyunsdkcore.request import CommonRequest
    client = AcsClient('<accessKeyId>', '<accessSecret>', 'default')
    
    request = CommonRequest()
    request.set_accept_format('json')
    request.set_domain('dysmsapi.aliyuncs.com')
    request.set_method('POST')
    request.set_protocol_type('https') # https | http
    request.set_version('2017-05-25')
    request.set_action_name('SendSms')
    
    request.add_query_param('SignName', 'lajos')
    request.add_query_param('PhoneNumbers', '15299013421')
    request.add_query_param('TemplateCode', 'SMS_135695051')
    request.add_query_param('TemplateParam', '{"code": "1123"}')
    
    response = client.do_action(request)
    print(str(response, encoding='utf-8'))
    ```

![短信邮件相关](https://github.com/lajos182/python-essay/blob/master/images/hashlip%2C%20urllib%2C%20http.client.png)