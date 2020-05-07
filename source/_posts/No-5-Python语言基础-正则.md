---
title: No.5 Python语言基础--正则
date: 2020-04-30 20:23:12
tags: [Python, 正则]
toc: true
---

### 5.1. 正则基础

**应用场景**：特定规律字符串的查找、替换、切割等；邮箱格式、url等的验证；爬虫项���，提取特定的有效内容；很多应用的配置文件

**使用原则**：只要能够通过自负床等相关函数能够解决的，就不要使用正则；正则的执行效率比较低，会降低代码的可读性；

<!--more-->

**基本使用**：正则式通过`re`模块提供支持的，相关函数包括`match`、`search`、`findall`、`compile`

- `match`：从头惊醒匹配，知道就立即返回正则结果对象，没有就返回`None`
- `search`：匹配全部内容，任意位置，只要找到，立即返回正则结果对象，没有返回`None`
- `findall`：匹配所有内容，返回匹配结构组成的列表，没有返回`None`
- `compile`：根据字符串生成正则表达式的对象，用于特定正则匹配，通过`match`、`search`、`findall`

### 5.2. 正则规则

- 单个字符：

  ```markdown
  普通字符：就是一对一的完全匹配
  []：中间的任意一个字符
  	[a-z]：表示a到z的任意字符
  	[0-9]：表示0到9的任意字符
  	[^abc]：除abc之外的字符
  . ：匹配'\n'以外的任意字符
  \d：所有的数字字符，等价于[0-9]
  \D：所有的非数字字符，等价于[^0-9]
  \w：所有的数字、字母、中文、下划线等 (就是字的意思)
  \W：\w之外的所有字符
  \s：所有的空白字符，如：空格、\t、\n、\r
  \S：\s之外的所有字符
  \b：词边界，如：开头、结尾、标点、空格等
  \B：非词边界
  ```

- 次数控制：

  ```
  *：前面的字符可以是任意次
  +：前面的字符至少出现一次
  ?：至多一次，0次或1次
  {m}：匹配固定的m次
  {m,}：至少m次
  {m,n}：m到n次
  ```

  > 正则的匹配默认是贪婪的
  >
  > 贪婪：最大限度的匹配，正则的匹配默认是贪婪的
  >
  > 非贪婪：只要满足匹配条件，能少匹配就少匹配，通常使用'?'取消贪婪

  ```python
  import re
  
  # c = re.compile(r'a.*?c')
  c = re.compile(r'a.+?c')
  
  s = c.search('abcjsdhfkdkc')
  
  if s:
      print('ok')
      print(s.group())
  >>> ok
  abc
  ```

- 边界限定

  ```markdown
  ^：以指定内容开头
  $：以指定内容结尾
  ```

- 分组匹配

  ```markdown
  |：表示或，具有最低的优先级
  ()：用于表示一个整体，可以确定优先级
  ```

  ```python
  import re
  c = re.compile(r'(\d+)([a-z]+)(\d+)')
  s = c.search('sda123ksdak456sdk')
  if s:
      print('ok')
      print(s.group())   //所有匹配的字符(即匹配正则表达式整体结果)
      print(s.group(0))  //与group()相同
      print(s.group(1))  //列出第一个括号匹配的字符
      print(s.group(2))  //列出第二个括号匹配部分
      print(s.group(3))  //列出第三个括号匹配部分
      print(s.groups())  //返回所有括号匹配的字符，以tuple格式
  ```

  ```python
  import re
  
  # 正则中的\1:表示前面第一个小阔号匹配的结果
  # c = re.compile(r'<([a-z]+)><([a-z]+)>\w*</\2></\1>')
  # c = re.compile(r'<(?P<hello>[a-z]+)><(?P<world>[a-z]+)>\w*</\2></\1>')
  # 可以给指定的分组起个名字
  c = re.compile(r'<(?P<hello>[a-z]+)><(?P<world>[a-z]+)>\w*</(?P=world)></(?P=hello)>')
  
  s = c.search('xxx<div><a>百度一下</a></div>yyy')
  
  if s:
      print('ok')
      print(s)
      print(s.group())
  ```

- 转义字符

  ```markdown
  在匹配所有正则中有特殊含义的字符时，都需要转义。
  正则字符串需要被处理两次，python中处理，正则解析时处理一次。
  通常在写正则字符串时，前面都加一个'r'，表示原始字符，让所有的字符失去意义。
  在匹配有如：'\'的字符时，前面再添加一个'\'就可以了，如果没有添加'r'，通常要写好几个。
  ```

### 5.3. 正则高阶

- 匹配模式

  正则可以对匹配的模式做出整体的修饰处理，如忽略大小写等。

  ```python
  import re
  
  # 忽略大小写
  # c = re.compile(r'hello', re.I)
  
  # 多行匹配
  # c = re.compile(r'^hello', re.M)
  
  # 作为单行处理  或  让 . 匹配 '\n'
  c = re.compile(r'<div>.*?</div>', re.S)
  string = ''''<div>\n
  hello\n</div>'''
  s = c.search(string)
  if s:
      print('ok')
      print(s.group())
  ```

- 字符串切割`split`

  某些情况下，无法通过字符串函数进行切割，可以使用正则的方式处理，如：按照数字切割。可使用`split`切割，返回一个列表

  ```python
  import re
  
  c = re.compile(r'\d')
  string = '正则1其实也不难2但是学完发现自己写不出来3是这样吧'
  ret = c.split(string, maxsplit=2)
  print(ret)
  print(type(ret))
  >>> ['正则', '其实也不难', '但是学完发现自己写不出来3是这样吧']
  <class 'list'>
  ```

- 字符串替换`sub`

  简单的替换可以自己处理，要是按照正则规律进行替换就要使用专用的函数`sub`处理。

  ```python
  import re
  
  c = re.compile(r'\d')
  string = 'sa1erfewr2fsdfs3dfsd'
  ret = c.sub('***', string)
  print(ret)
  ```

