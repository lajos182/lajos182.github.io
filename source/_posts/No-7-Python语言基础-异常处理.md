---
title: No.7 Python语言基础--异常处理
date: 2020-05-03 10:43:16
tags: Python
toc:
---

- **错误**：程序运行前的语法问题，如：关键字、缩进、括号不成对等。
- **异常**：在程序运行过程中出现的错误，如：变量未定义、除数为0、属性不存在等。
- **异常处理**：异常处理可以认为是一种特殊的路程控制语句，可以提高代码的健壮性。

<!--more-->

- **基本结构**：

  ```python
  try:
      print('正常执行')
      # print(a)
      print(3/0)
  except Exception as e:
      # Exception是所有异常的基类，因此此处可以捕获所有异常
      print('出现异常，进行处理')
      print(e)
  
  print('其他代码')
  ```

- 捕获多个异常：

  ```python
  try:
      # print(a)
      # print(3/0)
      fp = open('123.txt')
  except NameError as e:
      print('NameError:', e)
  except ZeroDivisionError as e:
      print('ZeroDivisionError:', e)
  except Exception as e:
      print('other:', e)
  ```

- 也可以对多种异常进行分组处理：

  ```python
  try:
      # print(a)
      # print(3/0)
      fp = open('123.txt')
  except (NameError, ZeroDivisionError) as e:
      # 将特定的某些异常统一处理，写到元组中即可
      print(e)
  except:
      print('其他异常')
  ```

- `else`和`finally`：

  ```python
  try:
      print('正常执行')
      print(a)
  except:
      print('出现异常')
  else:
      # 出现异常就不执行了
      print('正常结束')
  finally:
      # 无论有误异常都会执行
      print('无论如何都执行')
  ```

  > `else`：在没有异常的时候会执行，出现异常就不执行了
  >
  > `finally`：无论是否有异常，都会执行

- **抛出异常`raise`**：

  ```python
  try:
      print('开始')
      # 根据代码逻辑的需要，手动抛出特定的异常
      raise Exception('手动抛出异常')
      print('一切正常')
  except Exception as e:
      print('出现异常:', e)
  
  print('结束')
  ```

- 异常嵌套：`try-except`结构中嵌套`try-except`结构

  ```python
  print('我要去上班，什么事情都不能阻止我上班的脚步')
  try:
      print('我准备骑电动车')
      raise Exception('昨天晚上那个缺德的家伙把我充电器给拔了，无法骑车')
      print('骑电动车提前到达公司')
  except Exception as e:
      print(e)
      try:
          print('我准备坐公交')
          raise Exception('等了20分钟一直没有公交车，果断放弃')
          print('坐公交准时到达公司')
      except Exception as e:
          print(e)
          print('我准备打车')
          print('打车还是快，一会就到公司了')
  
  print('热情满满的开始一天的工作')
  ```

- 自定义异常类：继承自异常的基类(`Exception`)

  ```python
  # 自定义异常类，名字通常以Exception结尾
  class MyException(Exception):
      def __init__(self, msg):
          self.msg = msg
  
      def __str__(self):
          # repr键变量转换成字符串，若本身是字符串则不必要
          # return repr(self.msg)
          return self.msg
  
      def deal(self):
          print('异常已处理')
  
  try:
      print('正常执行')
      raise MyException('手动抛出定义异常')
  except MyException as e:
      print(e)
      e.deal()
  ```

- 特殊使用：

  - 场景：做文件操作时，中间无论读写，也无论有误异常，最终一定要把文件关闭

  - `with`：使用`with`，不必再关系文件的关闭问题，`with`语句块结束后一定会确保文件关闭

    ```python
    # fp = open('test.txt', 'rb')
    # # 各种读写操作
    # fp.close()
    
    # 使用with，无需关心文件的关闭问题
    with open('test.txt', 'rb') as fp:
        content = fp.read(5)
        print(content)
    ```