---
title: No.4 Python语言基础--模块使用
date: 2020-04-29 22:04:41
tags: Python
toc:
---

**模块**：模块就像一个工具包一样，里面有很多工具(函数、类)，使用时需要通过`import`导入。

- 分类：
  - ①标准库：`random`、`sys`、`os`、`time`；
  - ②第三方：已经写好的特定功能的模块，你可以直接使用pip命令安装；
  - ③自定义：自己写的

<!--more-->

```
import random					# 导入
import random as rdm			# 导入并起别名
from time import sleep			# 指定导入
from time import sleep as sp	# 指定导入并起别名
from random import *            # 模糊导入
```

**自定义模块**：新建一个文件，不与其他模块同名，该文件名就是模块，导入方式于官方的相同。文件名(模块名)就是命名空间，不同命名空间下的标识符可以同名，当使用几个模块中相同的（函数）标识符时，可以通过命名空间或起别名解决。

**测试模块**：当一个模块作为主模块运行时，`__ name __ `的值为` '__ main __'`，当被其他模块导入使用时，值为模块名。

```
if __name__ == '__main__':
    print('测试代码')
```

**包**：多个模块放在同一目录下，目录下有一个`__ init __.py`文件，这个目录就是一个包。一个目录要想成为一个包，必须包含一个` __ init __.py`文件，即使该文件为空(可以简化导入书写)

**修改pip源**：

- `Windows`：进入用户的家目录(在`windows`文件管理器中,输入` %APPDATA%`)，在该目录下新建`pip`文件夹，并创建`pip.ini`，然后添加对应的镜像源；

- `Linux`：进入用户的加目录(`cd ~`)，然后创建`.pip`文件夹(`mkdir .pip`)并创建`pip.conf`(`touch pip.conf`)文件，然后对应的镜像源；

- 国内常用的`pip`镜像

  ```text
  清华：https://pypi.tuna.tsinghua.edu.cn/simple
  
  阿里云：http://mirrors.aliyun.com/pypi/simple/
  
  中国科技大学 https://pypi.mirrors.ustc.edu.cn/simple/
  
  华中理工大学：http://pypi.hustunique.com/
  
  山东理工大学：http://pypi.sdutlinux.org/ 
  
  豆瓣：http://pypi.douban.com/simple/
  ```

- 使用方法：

  ```
  [global]
  index-url = http://mirrors.aliyun.com/pypi/simple/
  trusted-host = mirrors.aliyun.com
  ```

**pip命令**：安装软件包，自动会安装相关的依赖

```linxu
安装软件包：pip install 包名
卸载软件包：pip uninstall 包名
列表显示包：pip list
查看指定包：pip show 包
检测哪些包需要更新：pip list --outdated
升级包：pip install --upgrade 包名
输出已安装的包列表：pip freeze > requirements.txt
pip帮助：pip help
```