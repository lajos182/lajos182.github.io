---
title: No.9 Python语言基础--进程
date: 2020-05-04 12:26:11
tags: [Python, 进程]
toc: true
---

### 9.1. 进程简介

- 进程（任务）：
- 在计算机中，其实进程就是一个任务。
- 在操作系统中，进程是程序执行和资源分配的基本单元。
- 单核CPU实现多任务
  - 只是将CPU的时间快速的切换和分配到不同的任务上。
  - 主频足够高，切换足够快，人的肉眼无法分辨而已。

<!--more-->

- 多核CPU实现多任务
  - 如果任务的数量不超过CPU的核心数，完全可以实现一个核心只做一个任务。
  - 在操作系统中几乎是不可能的，任务量往往远远大于核心数。
  - 同样采用轮训的方式，轮流执行不同的任务，只是做任务的'人'有多个而已。

`Unix/Linux`操作系统提供了一个`fork()`系统调用，它非常特殊。普通的函数调用，调用一次，返回一次，但是`fork()`调用一次，返回两次，因为操作系统自动把当前进程（称为父进程）复制了一份（称为子进程），然后，分别在父进程和子进程内返回。

子进程永远返回0，而父进程返回子进程的ID。这样做的理由是，一个父进程可以fork出很多子进程，所以，父进程要记下每个子进程的ID，而子进程只需要调用`getppid()`就可以拿到父进程的ID。

```python
import os

print(f'Process {os.getpid()} start...')
# Only works on Unix/Linux/Mac:
pid = os.fork()
print(pid)
if pid == 0:
    print(f'I am child process ({os.getpid()}) and my parent is {os.getppid()})')
else:
    print(f'I ({os.getpid()}) just created a child process {pid}')

>>> Process (876) start...
I (876) just created a child process (877).
I am child process (877) and my parent is 876.
```

由于Windows没有fork调用，上面的代码在Windows上无法运行。有了fork调用，一个进程在接到新任务时就可以复制出一个子进程来处理新任务，常见的`Apache`服务器就是由父进程监听端口，每当有新的http请求时，就fork出子进程来处理新的http请求。

### 9.2. 进程的并发与并行

- **并发**：并发是指在资源有限的情况下，两者交替轮流使用资源，比如一段路(单核CPU资源)同时只能过一个人，A走一段后，让给B，B用完继续给A，交替使用，目的是提高效率。
- **并行**：并行是指两者同时执行，比如赛跑，两个人都在不停的往前跑（资源够用，比如三个线程，四核的CPU）；
- **区别**：并发是从宏观上，在一个时间段上可以看出是同时执行的，比如一个服务器同时处理多个session。并行是从微观上，也就是在一个精确的时间片刻，有不同的程序在执行，这就要求必须有多个处理器。

### 9.3. 进程管理

```python
from multiprocessing import Process
import os

def run(name):
    print(f'Run child process {name} ({os.getpid()})')

if __name__ == "__main__":
    print(f'Parent process {os.getpid()}')
    # 创建进程，指定任务函数，target就是指定任务的函数，name代表进程名称， args和kwargs表示传递给子进程任务函数的参数
    p = Process(target=run, args=('test',))
    print('Child process will start...')
    p.start()
    p.join()
    print('Child process end...')
```

- `os.getpid()`：获取当前的进程id

- `os.getppid()`：获取主进程id

- `Process([group [, target [, name [, args [, kwargs]]]]])`：由该类实例化得到的对象，表示一个子进程中的任务（尚未启动），需要使用关键字的方式来指定参数，args指定的为传给target函数的位置参数，是一个元组形式，必须有逗号

  -group参数未使用，值始终为None
  -target表示调用对象，即子进程要执行的任务
  -name为子进程的名称
  -args表示调用对象的位置参数元组，args=(1, 2, 'egon',)
  -kwargs表示调用对象的字典，kwargs={'name': 'egon', 'age': 18}

- `p.daemone`：默认值为`False`，如果设为`True`，代表p为后台运行的守护进程，当p的父进程终止时，p也随之终止，并且设定为True后，p不能创建自己的新进程，必须在`p.start()`之前设置

- `p.name`：进程的名称

- `p.pid`：进程pid

- `p.exitcode`：进程在运行时为None、如果为–N，表示被信号N结束(了解即可)

- `p.authkey`：进程的身份验证键,默认是由`os.urandom()`随机生成的32字符的字符串。这个键的用途是为涉及网络连接的底层进程间通信提供安全性，这类连接只有在具有相同的身份验证键时才能成功（了解即可）

- `p.start()`：启动进程，并调用该子进程中的`p.run()`

- `p.run()`：进程启动时运行的方法，正是它去调用`target`指定的函数，我们自定义类的类中一定要实现该方法

- `p.terminate()`：强制终止进程p，不会进行任何清理操作，如果p创建了子进程，该子进程就成了僵尸进程，使用该方法需要特别小心这种情况。如果p还保存了一个锁那么也将不会被释放，进而导致死锁

- `p.is_alive()`：如果p仍然运行，返回True

- `p.join[timeout])`：主线程等待p终止（强调：是主线程处于等的状态，而p是处于运行的状态）。`timeout`是可选的超时时间，需要强调的是，`p.join`只能`join`住`start`开启的进程，而不能`join`住`run`开启的进程

### 9.4. 进程锁

当多个进程操作同一资源时，可能会造成混乱，甚至错误（如写文件），通常采用加锁的方式进行解决

```python
from multiprocessing import Process, Lock
import os
import time

def run(label, lock):
    # 获取锁
    lock.acquire()
    time.sleep(1)
    # 中间的任务，不肯能同时多个进程任务执行
    print(label, os.getpid())
    # 释放锁
    lock.release()

if __name__ == '__main__':
    print('主进程：', os.getpid())
    # 创建进程锁
    lock = Lock()
    # 创建多个子进程
    # 用于存储所有的进程
    recode = []
    for i in range(5):
        p = Process(target=run, args=(f'子进程{i}', lock))
        p.start()
        recode.append(p)
    # 等待进程结束
    for p in recode:
        p.join()
# 运行结果如下：
>>> 主进程： 22224
子进程0 22288
子进程1 22296
子进程2 22312
子进程3 22328
子进程4 22344
```

### 9.5. 自定义进程

采用继承`Process`类开启进程的方式

```python
from multiprocessing import Process, Lock
import os
import time

class MyProcess(Process):

    def __init__(self, name, num=1):
        super().__init__()
        self.name = name
        self.num = num
    
    def run(self):
        for i in range(self.num):
            print(f'子进程({i})运行中...')
            time.sleep(1)
        
if __name__ == "__main__":
    p = MyProcess('test1')
    print(f'主进程{p.name}开始启动...')
    p.start()
    p.join()
```

### 9.6. 进程池

创建少量的进程可以通过创建`Process`对象完成，如果需要大量的进程和管理时就比较费劲。这时，可以通过进程池加以解决，而且可以通过参数控制进程池中进程的并发数，提高CPU利用率（创建进程池-->添加进程-->关闭进程池-->等待进程池结束-->设置回调）

```python
import multiprocessing
import time, random
import os


def callback(num):
    print(num)

def task(num):
    print(f'Run task {num} ({os.getpid()})...')
    start = time.time()
    time.sleep(random.random() * 3)
    end = time.time()
    print(f'Task {num} runs {(end-start):,.2f} seconds.')
    return num

if __name__ == "__main__":
    # 获取cpu核心数
    print('cpu核数：', multiprocessing.cpu_count())
    print(f'Parent process {os.getpid()}')
    # 创建进程池，一般进程池中的进程数不超过cpu核心数
    pool = multiprocessing.Pool(4)
    # 循环创建进程并添加到进程池中
    for i in range(5):
        # 参数：
        # func：任务函数
        # args：任务函数的参数
        # callback：回调函数，进程结束时调用，参数是进程函数的返回值
        pool.apply_async(func=task, args=(i,), callback=callback)
        
    # 如果换用map_async()可以这样来写
    # pool.map_async(task, [i for i in range(5)])
    print('Waiting for all subporcess done...')
    # 关闭进程池，关闭后就不能添加进程了
    pool.close()
    # 等待进程结束
    pool.join()
    print('All subprocess done.')
# 运行结果如下：
>>> cpu核数： 2
Parent process 22840
Waiting for all subporcess done...
Run task 0 (22888)...
Run task 1 (22880)...
Run task 2 (22904)...
Run task 3 (22920)...
Task 2 runs 0.48 seconds.
Run task 4 (22904)...
2
Task 0 runs 0.80 seconds.
0
Task 3 runs 0.71 seconds.
3
Task 1 runs 1.50 seconds.
1
Task 4 runs 1.86 seconds.
4
All subprocess done.
```

- `Pool([numprocess[, initializer[, initargs]]])`：创建进程池，`numprocess`为要创建的进程数，默认使用`multiprocessing.cpu_count()`的CPU核心数；`initializer`是每个工作进程启动时要执行的可调用对象，默认为`None`；`initargs`是要传给`initializer`的参数组。
- `pool.apply(func[, args[, kwargs]])`同步执行(阻塞式)：在**一个池工作进程中**执行`func(*args,**kwargs)`,然后返回结果，**同步运行，阻塞，直到本次任务执行完毕后返回结果**。此操作并不会在所有池工作进程中并执行`func`函数。如果要通过不同参数并发地执行`fun`c函数，必须从不同线程调用`pool.apply()`函数或者使用`pool.apply_async()`。
- `pool.map(func, iterable[, chunksize=None])`：和`pool.apply()`类似，只不过需要接收一个可迭代的参数对象。
- `pool.apply_async(func[, args[, kwargs[, callback=None[, error_callback=None]]]])`异步执行(非阻塞)：在**一个池工作进程中**执行`func(*args,**kwargs)`，然后返回结果。此方法的结果是`AsyncResult`类的实例，callback是可调用对象，接收输入参数。当func的结果变为可用时，将理解传给callback。callback禁止执行任何阻塞操作，否则将接收其他异步操作中的结果。
- `pool.map_async(func, iterable[, chunksize=None[, callback=None[, error_callback=None]]]])`：和`pool.apply_async()`类似，只不过需要接收一个可迭代的参数对象。

### 9.7. 数据共享

`Process`之间肯定是需要通信的，操作系统提供了很多机制来实现进程间的通信。`Python`的`multiprocessing`模块包装了底层的机制，提供了`Queue`、`Pipes`等多种方式来交换数据。但要注意的是，**全局变量不能共享**。

```python
import multiprocessing
import time, random
import os


num = 250
lt = ['hello']

def run():
    global num, lt
    print('子进程开始。。。')
    num += 10
    lt.append('world')
    print('子进程结束：', num, lt)

if __name__ == "__main__":
    print('主进程开始》》》', num, lt)
    p = multiprocessing.Process(target=run)
    p.start()
    p.join()
    print('主进程结束：', num, lt)

# 运行结果如下：
>>> 主进程开始》》》 250 ['hello']
子进程开始。。。
子进程结束： 260 ['hello', 'world']
主进程结束： 250 ['hello']
```

- 管道(Pipe)：创建管道时，得到两个连接；默认`dublex`为`True`，表示全双工，两边都可以收发；若`dublex`为`False`，表示半双工，`p_a`只能收，`p_b`只能发。

  - 全双工通信：

  ```python
  import multiprocessing
  
  def run(p_a):
      # 子进程接收主进程发的数据
      recv = p_a.recv()
      print('子进程收到：', recv)
      print('子进程发送：', ['a', 'b', 'c', 'd'])
      # 子进程给主进程发数据
      p_a.send(['a', 'b', 'c', 'd'])
  
  if __name__ == "__main__":
      # 创建管道, 默认为全双工
      p_a, p_b = multiprocessing.Pipe()
      p = multiprocessing.Process(target=run, args=(p_a,))
      p.start()
      print('主进程发送：', [1, 2, 3, 4, 5])
      # 主进程向子进程发数据
      p_b.send([1, 2, 3, 4, 5])
      p.join()
      # 主进程接收子进程的数据
      recv = p_b.recv()
      print('主进程收到：', recv)
      print('主进程结束')
  
  # 运行结果如下：
  >>> 主进程发送： [1, 2, 3, 4, 5]
  子进程收到： [1, 2, 3, 4, 5]
  子进程发送： ['a', 'b', 'c', 'd']
  主进程收到： ['a', 'b', 'c', 'd']
  主进程结束
  ```

  - 半双工通信：`dublex`为`False`，表示半双工，`p_a`只能收，`p_b`只能发

  ```python
  import multiprocessing
  
   def run(p_a):
       # 子进程接收主进程发的数据
       recv = p_a.recv()
       print('子进程收到(p_a)：', recv)
  
   if __name__ == "__main__":
       # 创建管道, 默认为全双工
       p_a, p_b = multiprocessing.Pipe(duplex=False)
       p = multiprocessing.Process(target=run, args=(p_a,))
       p.start()
       print('主进程发送(p_b)：', [1, 2, 3, 4, 5])
       # 主进程向子进程发数据
       p_b.send([1, 2, 3, 4, 5])
       p.join()
       # 主进程接收子进程的数据
       print('主进程结束')
   # 运行结果如下：
   >>> 主进程发送(p_b)： [1, 2, 3, 4, 5]
   子进程收到(p_a)： [1, 2, 3, 4, 5]
   主进程结束
  ```

- 队列(Queue)

`Queue`本身就是一个消息队列程序，常用的方法有：

- `Queue.qsize()`：返回当前消息队列的消息数量。
- `Queue.empty()`：如果队列为空，返回`True` 否则返回`False`。
- `Queue.full()`：如果队列满了，返回`True`，否则`False`。
- `Queue.get()`：获取队列中的一条消息，然后将其从队列中移除。队列为空时，获取的时候也会阻塞。
- `Queue.put('xxx')`：把内容存放进消息队列, 默认为`True`数据会阻塞，设为`False`时，如果队列已满会报错。
- `Queue.close()`：关闭队列
- `Queue.get_nowait()`相当于`Queue.get(False)`。
- `Queue.put_nowait()`相当于`Queue.put(False)`。

```python
import multiprocessing
import os
import time

def put_data(queue):
    # 拼接数据
    data = str(os.getpid()) + ' ' + str(time.time())
    print('压入数据：', data)
    queue.put(data)

def get_data(queue):
    data = queue.get()
    print('读取数据：', data)

if __name__ == "__main__":
    # 创建队列
    q = multiprocessing.Queue(3)

    # 创建5个进程用于写数据
    record1 = []
    for i in range(5):
        p = multiprocessing.Process(target=put_data, args=(q,))
        p.start()
        record1.append(p)
    
    # 创建5个进程用于读数据
    record2 = []
    for i in range(5):
        p = multiprocessing.Process(target=get_data, args=(q,))
        p.start()
        record2.append(p)

    for p in record1:
        p.join()

    for p in record2:
        p.join()
# 运行结果如下
>>> 压入数据： 6220 1573552927.6151476
压入数据： 5900 1573552927.7400696
压入数据： 19276 1573552927.8130243
压入数据： 17492 1573552927.9059665
压入数据： 16408 1573552927.9929125
读取数据： 6220 1573552927.6151476
读取数据： 5900 1573552927.7400696
读取数据： 19276 1573552927.8130243
读取数据： 17492 1573552927.9059665
读取数据： 16408 1573552927.9929125
```

如果采用进程池创建进程，使用队列进行通讯，可以这样：

```python
def put_data(queue):
    # 拼接数据
    data = str(os.getpid()) + ' ' + str(time.time())
    print('压入数据：', data)
    queue.put(data)

def get_data(queue):
    data = queue.get()
    print('读取数据：', data)

if __name__ == "__main__":
    # 创建队列
    q = multiprocessing.Queue(3)

    # 创建5个进程用于写数据
    pool1 = multiprocessing.Pool(4)
    pool1.map_async(put_data, [i for i in range(5)])
    pool1.close()
    pool1.join()
    # 创建5个进程用于读数据
    pool2 = multiprocessing.Pool(4)
    pool2.map_async(put_data, [i for i in range(5)])
    pool2.close()
    pool2.join()
# 运行结果如下：
>>> 压入数据： 16136 1573552887.7448432
压入数据： 16136 1573552887.7468312
压入数据： 16136 1573552887.74783
压入数据： 16136 1573552887.7488291
压入数据： 16136 1573552887.7718146
压入数据： 17880 1573552888.1625717
压入数据： 17880 1573552888.1655722
压入数据： 17880 1573552888.1665695
压入数据： 17880 1573552888.1795611
压入数据： 19020 1573552888.1855621
```

### 9.8. 共享内存

`Python`中提供了强大的`Manager`类，专门用于实现多进程之间的数据共享；`Mangaer`类支持的类型非常多，如：`Value`, `Array`, `List`, `Dict`, `Queue`, `Lock`等；`Manager`提供的数据不安全，需要通过`Lock`作处理。

```python
import multiprocessing
from ctypes import c_char_p

def run(v, s, a, l, d):
    print('子进程：', v.value)
    v.value += 10
    print('子进程：', s.value)
    s.value = b'world'
    print('子进程：', a[0])
    a[0] = 5
    l.append('subprocess')
    d['name'] = 'xiaoming'

if __name__ == "__main__":
    # 共享内存，可以共享不同类型的数据
    server = multiprocessing.Manager()

    # 整数i, 小数f
    v = server.Value('i', 250)
    # 字符串
    s = server.Value(c_char_p, b'hello')
    # 数组，相当于列表
    a = server.Array('i', range(5))
    # 列表
    l = server.list()
    # 字典
    d = server.dict()

    p = multiprocessing.Process(target=run, args=(v, s, a, l, d))

    p.start()
    p.join()
    print('主进程：', v.value)
    print('主进程：', s.value)
    print('主进程：', a[0])
    print('主进程：', l)
    print('主进程：', d)
# 运行结果如下：
>>> 子进程： 250
子进程： b'hello'
子进程： 0
主进程： 260
主进程： b'world'
主进程： 5
主进程： ['subprocess']
主进程： {'name': 'xiaoming'}
```

### 9.9. 多进程应用举例

使用多进程实现拷贝文件夹，注意直接使用`copy()`效率比较低

```python
import multiprocessing
import os


def run(src, dst):
    src_fp = open(src, 'r')
    dst_fp = open(dst, 'w')
    content = src_fp.read(1024)
    while content:
        dst_fp.write(content)
        content = src_fp.read(1024)
    src_fp.close()
    dst_fp.close()
    print(src, '拷贝到', dst, '已完成')


def copy(src, dst=None):
    if not dst:
        dst = f'{src}_副本'
    if not os.path.exists(src):
        print('源文件不存在！')
        return None
    if os.path.abspath(src) == os.path.abspath(dst):
        print('源文件与目标文件路径相同，无法进行拷贝！')
        return None
    if not os.path.exists(dst):
        os.makedirs(dst)
    record = []
    src_dirs = os.listdir(src)
    for file in src_dirs:
        src_path = os.path.join(src, file)
        dst_path = os.path.join(dst, file)
        if os.path.isfile(src_path):
            p = multiprocessing.Process(target=run, args=(src_path, dst_path))
            p.start()
            record.append(p)
        else:
            copy(src_path, dst_path)
    return record


if __name__ == "__main__":
    src = 'test'
    dst = ''
    record = copy(src, dst)
    if not record:
        print('拷贝异常...')
    else:
        for p in record:
            p.join()
        print('拷贝结束...')
```