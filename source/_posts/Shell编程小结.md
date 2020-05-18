---
title: Shell编程小结
date: 2020-05-18 11:23:50
tags: Shell
toc: true
---

## 1.1. `shell`简介

+ 什么是`shell`？

  - 把在终端下可执行的命令保存到一个文件中，这个文件就是`shell`程序，也就做`shell`脚本。

+ `shell`的类型：

  - `ash`、`bash`、`ksh`、`sch`等
  - 查看默认`shell`使用`echo $SHELL`
  - `/etc/shell`文件中存放了当前系统中可用的`shell`

  <!--more-->

+ `shell`脚本的执行

  - 使用指定`shell`执行指定脚本：`bash hello.sh`，该文件可以没有可执行权限
  - 将`sehll`脚本文件作为可执行程序执行，必须添加可执行权限
    - 添加可执行权限，`sudo chmod +x hello.sh`
    - 在开头指定`shell`：`#!/bin/bash`，其他位置`#`表示注释
    - 执行脚本：
      - 在本目录：`./hello.sh`
      - 不在此目录：`/home/ubuntu/hello.sh`

## 1.2. `shell`变量

+ 定义变量：`name="xiaoming"`
+ 打印变量：`echo $name`或`echo {name}`
+ 销毁变量：`unset name`
+ 声名常量：`readonly name="xiaoming"`
+ 注意事项：使用`=`，两边不能有空格

## 1.3. 变量类型

+ 本地变量：只适用于当前`shell`的变量

+ 环境变量：适用于整个系统，通常都是全大写的

  - 查看环境变量：`env`
  - 打印指定变量：`echo $PATH`
  - `PATH`：若想要你的脚本在哪里都可以使用，需要脚本庐江添加到`PATH`环境变量中
  - 修改：
    - 一次性：`export PATH=$PATH:/home/ubuntu`
    - 永久性：
      - 系统级别：`/ect/profile`
      - 用户级别：`~/.profile`、`~/.bashrc`、`~/.bash_profile`
      - 重新加载(立即生效)：`source file`或`.filename`

+ 位置变量

  - `$0`：表示脚本名
  - `$1 ~ $9`：表示传递给脚本的参数

+ 特殊变量

  - `$#`：传递给脚本或函数的参数个数

  - `$*`：传递给脚本或函数的所有参数

  - `$?`：上次命令执行情况，0表示正确，其他表示错误

  - `$@`：传递给脚本或函数的所有参数，被双引号`" "`包含时，与 `$* `稍有不同

  - `$$`：当前`shell`进程`ID`

    ```shell
    #!/bin/bash
    echo "File Name: $0"
    echo "First Parameter : $1"
    echo "First Parameter : $2"
    echo "Quoted Values: $@"
    echo "Quoted Values: $*"
    echo "Total Number of Parameters : $#"
    # 运行结果如下：
    >>> ./var.sh Zara Ali
    File Name : ./test.sh
    First Parameter : Zara
    Second Parameter : Ali
    Quoted Values: Zara Ali
    Quoted Values: Zara Ali
    Total Number of Parameters : 2
    ```

+ 单双引号的区别

- 双引号：可以是除`$`、`'`、`\`、`"`之外的任意字符

- 单引号：其中的任意字符都不解析，都不会原样输出

  ```shell
  #!/bin/bash
  name="xiaoming"
  echo "hello $namehow are you"
  echo "hello ${name}how are you"
  echo 'hello ${name}how are you'
  # 运行结果
  >>> bash str.sh
  hello are you
  hello xiaominghow are you
  hello ${name}how are you
  ```

- 反引号：会将其中的内容作为命令执行

- 反斜杠：转义特定的字样，如：`&`、`*`、`|`、`?`、`^`、`$`

## 1.4. 字符串操作

+ 计算长度：`${% raw %}{#name}{% endraw %}`
+ 提取字符：`${name:2:3}`，从下标为2的字符开始提取三个字符

## 1.5. 数组操作

+ 定义：`a=(1 2 3)`
+ 访问成员：`${a[0]}`
+ 长度：`${% raw %}{#a[@]}{% endraw %}`
+ 所有元素：`${a[*]}`，使用`for-in`遍历需要提取元素

## 1.6. `seq`命令

+ 说明：生成连续的整数
+ 示例：`seq 1 10`，生成1~10的连续整数

## 1.7. `expr`命令

+ 说明：运算一个表达式

+ 示例：

  ```shell
  expr 1 + 2
  echo `expr 1 + 2` # 打印结果
  expr 3 \* 5  # 需要转义
  ```

## 1.8. 各种运算

+ `test`命令：成功为真，失败为假，测试结果通过`$?`检查，0表示真，1代表假

  ```shell
  #!/bin/bash
  if test 1 -lt 2
  then 
      echo "OK"
  fi
  # 或者可以使用下面的方式，then和if在一行时要使用;
  if test 1 -lt 2; then 
      echo "OK"
  fi
  ```

+ 数值比较

  - `-lt`：小于

  - `-lte`：小于等于

  - `-gt`：大于

  - `-gte`：大于等于

  - `-eq`：相等

  - `-ne`：不相等

    ```shell
    #!/bin/bash
    # []中前后都需要空格，then和if同行需要写分号;
    if [ 1 -lt 2 ]; then 
        echo "OK"
    fi
    ```

+ 字符串测试：

  - `=`：相等
  - `!=`：不相等
  - `-z`：字符串长度是否为0
  - `-n`：字符串长度是否不为0

+ 文件判断

  - `-f`：普通文件
  - `-d`： 目录文件
  - `-w`：文件存在，且可写
  - `-x`：文件存在，且可执行
  - `-s`：文件存在，且至少有一个字符
  - `-c`：字符设备文件
  - `-b`： 块设备文件

+ 逻辑运算

  - `-a`：逻辑与(`and`)，也可以使用`&&`进行转换

  - `-o`：逻辑或(`or`)，也可以使用`|`转换

  - `!`：逻辑非

    ```shell
    #!/bin/bash
    if [ 1 -lt 2 -a 2 -lt 3 ]; then 
        echo "OK"
    fi
    # 可以转化为&&
    if [ 1 -lt 2 ] && [ 2 -lt 3 ]; then 
        echo "OK"
    fi
    # 逻辑非
    if [ ! 3 -lt 2 ]; then 
        echo "OK"
    fi
    ```

## 1.9. 分支结构

+ `if-else`：

  ```shell
  #!/bin/bash
  if [ 1 gt 2 ]; then
      echo "大于"
  elif [ l lt 2 ]; then
      echo "小于"
  else
      echo "等于"
  fi
  ```

+ `case`：

  ```shell
  #!/bin/bash
  # 从终端获取内容，-p后的字符串是提示字符
  read -p 'press any key: ' ch
  case $ch in
      [a-z])
          echo "alpha"
      ;;
      [0-9])
          echo "numberic"
      ;;	
      *)
          echo "other"
      ;;
  esac
  ```

## 1.10. 循环结构

+ `for-in`

  ```shell
  #!/bin/bash
  # 遍历字符串数组
  for x in a b c
  do
      echo $x
  done
  # 遍历目录
  for x in /etc/*
  do
      echo $x
  done
  # 遍历1~10的整数，反引号表示执行
  for x in `seq 1 10`
  do
      echo $x
  done
  # 计算1+2+3+...+100
  for i in `seq 1 100`
  do
      let j+=$i
  done
  echo $j
  # 遍历数组
  a=(1 6 3)
  for x in ${a[*]}
  do
      echo $x
  done
  
  
  ```

+ `for`

  ```shell
  #!/bin/bash
  # 遍历1~10的整数
  for ((i=0;i<=10;i++))
  do
      echo $i
  done
  # 遍历数组
  a=(2 5 6)
  for ((i=0;i<=${#a[@]};i++))
  do
      echo ${a[$i]}
  done
  
  
  ```

+ `while`

  ```shell
  #!/bin/bash
  i=1
  sum=0
  # while (($i<=10))
  while [ $i -le 10 ]
  do
      echo $i
      # ((i++))
      # let i++
      # let i+=1
      i=$[$i+1]
      
      ((sum+=$i))
      # let sum+=$i
      # sum=$[$sum+$i]
  done
  echo $sum
  
  
  ```

+ `until`：条件与`while`相反

  ```shell
  #!/bin/bash
  i=1
  until [ $i -gt 10 ]
  do
      echo $i
      ((i++))
  done
  
  
  ```

+ `break`、`continue`的使用

## 1.11. 函数使用

~~~bash
```shell
demo()
{
    echo "call demo func"
}

demo
# $1,$2接收传递的实参，$?就代表函数的返回值
arg()
{
    echo $1
    echo $2
    echo $#
    echo $*
    return $?
}
arg 1 2 
```
~~~

