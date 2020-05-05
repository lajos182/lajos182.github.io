---
title: No.10 Python语言基础--线程
date: 2020-05-05 22:32:18
tags: [Python, 线程]
toc: true
---

### 10.1. 线程简介

- 在一个进程中，若想做多个子任务，我们把每个子任务称为线程
- 线程可以理解为轻量级的进程
- 进程之间的数据是独立，而一个进程下的线程数据是共享的
- **线程**是CPU分配时间的最小单位，进程和线程的调度都是操作系统的事
- 一个进程默认都有一个线程，我们称为主线程

<!--more-->

### 10.2. 线程模块

`Python`的标准库提供了两个模块：`_thread`和`threading`，`_thread`是**低级模块**，`threading`是**高级模块**，对`_thread`进行了封装。绝大多数情况下，我们只需要使用`threading`这个高级模块。

- 低级模块(`_thread`)：非常简陋，不建议使用

```python
import _thread
import time

def loop():
    print('子线程开始')
  	print('子线程结束')
    
if __name__ == '__main__':  
    print('主线程开始')
    # 创建线程
    _thread.start_new_thread(loop, ())
    # 主线程结束，子线程立即结束，通过演示测试
    time.sleep(3)
    print('主线程结束')

```

- 高级模块(`threading`)

```python
import threading
import time

def run(num):
    c = threading.current_thread()
    print('子线程开始：', c.name)
    time.sleep(3)
    print(num)
    print(threading.active_count())
    print('子线程结束：', c.name)

if __name__ == "__main__":
    # 获取主线程
    t = threading.main_thread()
    print('主线程开始：', t.name)
    # 获取当前线程
    c = threading.current_thread()
    print('当前线程：', c.name)
    
    # 创建子线程
    sub = threading.Thread(target=run, args=(250,), name='subthread')

    # 启动子线程
    sub.start()

    # 活跃线程个数
    print(threading.active_count())
    # 线程列表
    print(threading.enumerate())

    time.sleep(1)
    # 判断线程是否活着
    print(sub.is_alive())
    # 等待子线程
    sub.join()
    print(sub.is_alive())
    print('主线程结束：', t.name)
```

- `threading.main_thread()`: 返回主线程对象。可以通过`name`属性获取主线程名称，默认`MainThread`
- `threading.current_thread()`：返回当前线程对象，可以通过`name`属性获取当前线程，默认`Thread-N`
- `threading.active_count()`：返回当前存活的线程个数
- `threading.enumrate()`：以列表形式返回当前所有存活的`Thread`对象
- `threading.Thread(group=None, target=None, name=None, args=(), kwargs={}, *, daemon=None)`：构造线程对象
  - `group`：应该为`None`，为了日后扩展`ThreadGroup`类实现而保留
  - `target`：用于任务函数`run()`调用的可调用对象，默认为`None`，表示不需要调用任何方法
  - `name`：线程名称，默认情况下，由`Thread-N`格式构成一个唯一的名称，其中N是小的十进制数。
  - `args`：用于调用目标函数的参数元组，默认为`()`
  - `kwargs`：是用于调用目标函数的关键字参数字典，默认为`{}`
  - `daemon`：默认为`None`，线程将继承当前线程的守护模式属性；不是`None`，线程将被显式的设置为守护模式，不管该线程是否是守护模式
- `sub.start()`：开始线程活动
- `sub.run()`：代表线程活动的方法。也可以在子类型里重载这个方法，标准的`run()`方法对作为`target`参数传递给该对象构造器的可调用对象（如果存在）发起调用，并附带从`args`和`kwargs`参数分别获取的位置和关键字参数。
- `sub.join(timeount=None)`：等待，直到线程结束。这会阻塞调用这个方法的线程，直到被调用`join()`的线程终结。当`timeout`不为`None`时，因为`join()`总是返回`None`，所以你一定要在`join()`后调用`is_alive()`才能判断是否发生超时。
- `sub.is_active()`：判断线程是否存活着
- `os.exit()`：用于退出当前线程
- `os._exit()`：用于退出当前进程中的主线程

### 10.3. 数据共享

全局变量是可以共享的。

```python
import threading
import time

num = 250

def thread_one():
    global num
    num += 10

def thread_two():
    global num
    num -= 10

if __name__ == "__main__":
    print('主线程开始：', num)
    t1 = threading.Thread(target=thread_one)
    t1.start()
    t1.join()
    print('主线程：', num)
    t2 = threading.Thread(target=thread_two)
    t2.start()
    t2.join()
    print('主线程结束：', num)
# 运行结果如下：
>>> 主线程开始： 250
主线程： 260
主线程结束： 250
```

> 线程之间共享进程的所有数据

### 10.4. 线程锁

多线程和多进程最大的不同在于，多进程中，同一个变量，各自有一份拷贝存在于每个进程中，互不影响，而多线程中，所有变量都由所有线程共享，所以，任何一个变量都可以被任何一个线程修改，因此，线程之间共享数据最大的危险在于多个线程同时改一个变量，把内容给改乱了。

```python
import threading
import time

# 账户余额
balance = 250

def operate(num):
    global balance
    for i in range(1000000):
        balance += num
        balance -=num

if __name__ == "__main__":
    print('账户初始余额为：', balance)
    while True:
        t1 = threading.Thread(target=operate, args=(10,))
        t2 = threading.Thread(target=operate, args=(10,))
        t1.start()
        t2.start()
        t1.join()
        t2.join()
        if balance != 250:
            break
    print('账户最终余额为：', balance)
# 运行结果如下：
>>> 账户初始余额为： 250
账户最终余额为： 240
```

在上面的例子中，两个线程同时一存一取，最终导致余额不对，所以我们必须要保证一个线程在修改`balance`的时候，其他的线程一定不能改。所以，这时候就需要**线程锁**来做这件事情，当某个线程获取锁之后，除非锁被释放才可以继续执行其他线程，保证同意时刻最多只有一个线程持有该所，这样才不会造成冲突。

- `threading.Lock()`：创建线程锁

```python
import threading
import time

# 账户余额
balance = 250

def operate(num, lock):
    global balance
    for i in range(1000000):
        # lock.acquire()
        # try:
        #     balance += num
        #     balance -=num
        # finally:
        #     lock.release()
        # 可以直接采用with实现锁的创建和释放
        with lock:
            balance += num
            balance -=num

if __name__ == "__main__":
    print('账户初始余额为：', balance)
    # 创建线程锁对象
    lock = threading.Lock()
    t1 = threading.Thread(target=operate, args=(10, lock))
    t2 = threading.Thread(target=operate, args=(10, lock))
    t1.start()
    t2.start()
    t1.join()
    t2.join()
    print('账户最终余额为：', balance)
# 运行结果
>>> 账户初始余额为： 250
账户最终余额为： 250
```

### 10.5. 自定义线程

自定义线程，需要继承自`Thread`类，还需要重写`run()`方法，开启线程时会自动调用`run()`方法

```python
import threading
import time

class MyThread(threading.Thread):

    def __init__(self, time, name):
        super().__init__()
        self.time = time
        self.name = name
    
    def run(self):
        print(f'Sub thread {self.name} start >>>')
        time.sleep(self.time)
        print(f'Sub thread {self.name} end <<<')

if __name__ == "__main__":
    print('Main thread start...')
    record = []
    for i in range(5):
        p = MyThread(2, i)
        p.start()
        record.append(p)
    for p in record:
        p.join()
    print('Main thread end...')

# 运行结果
>>> Main thread start...
Sub thread 0 start >>>
Sub thread 1 start >>>
Sub thread 2 start >>>
Sub thread 3 start >>>
Sub thread 4 start >>>
Sub thread 2 end <<<
Sub thread 3 end <<<
Sub thread 0 end <<<
Sub thread 1 end <<<
Sub thread 4 end <<<
Main thread end...
```

### 10.6. 定时线程

在`threading`模块中，有一个`Timer`类，继承自`Thread`类，可以延迟执行线程任务。

- `threading.Timer(interval, function[, args, kwargs])`：设置一定时间后执行任务，`interval`表示设置的时间(`s`)，`function`表示要执行的任务，`args`与`kwargs`表示传入的参数
- `timer.start()`：开启定时任务
- `timer.end()`：取消定时任务

```python
import threading
import time

def run(name):
    print(f'Sub thread {name} start time：{time.ctime()}')
    time.sleep(1)
    print(f'Sub thread {name} end time：{time.ctime()}')

if __name__ == "__main__":
    print('Main thread start...')
    record = []
    for i in range(3):
        sub = threading.Timer(3, run, args=(i,))
        sub.start()
        record.append(sub)
    for sub in record:
        sub.join()
    print(f'Main thread end time：{time.ctime()}')
    print('Main thread end...')
# 运行结果如下：
>>> Main thread start...
Sub thread 0 start time：Tue Nov 19 09:53:58 2019
Sub thread 1 start time：Tue Nov 19 09:53:58 2019
Sub thread 2 start time：Tue Nov 19 09:53:58 2019
Sub thread 0 end time：Tue Nov 19 09:53:59 2019
Sub thread 1 end time：Tue Nov 19 09:53:59 2019
Sub thread 2 end time：Tue Nov 19 09:53:59 2019
Main thread end time：Tue Nov 19 09:53:59 2019
Main thread end...
```

### 10.7. 信号传递(`Event`)

在`threading`模块中，有一个`Event`类，可以用来控制线程的执行。

```python
import threading
import time

def run(num):
    for i in range(num):
        # 等待条件成立，会阻塞
        e.wait()
        print(f'第{i+1}步...')
        # 清除条件
        e.clear()

if __name__ == "__main__":
    print('Main thread start...')
    e = threading.Event()
    sub = threading.Thread(target=run, args=(3,))
    sub.start()
    for i in range(3):
        time.sleep(1)
        # 设置后，wait处将不在阻塞
        e.set()
        print('走你', end=' ')
```

### 10.8. 多核CPU(Python无法利用多线程实现多核任务)

如果你用于一个多核CPU，你肯定回想，多核应该可以同时执行多个线程，可以写一个死循环，在N核CPU上执行N个死循环线程，结果会发生什么？

```python
import threading, multiprocessing

def loop():
    x = 0
    while True:
        x = x ^ 1

if __name__ == "__main__":
    for i in range(multiprocessing.cpu_count()):
        t = threading.Thread(target=loop)
        t.start()
```

启动上面的程序，双核的CPU上可以监控CPU利用率仅有97%，也就是仅使用了一核。但是用`C`、`C++`或`Java`来改写相同的死循环，直接可以把全部核心跑满，4核就跑到400%，8核就跑到800%，为什么Python不行呢？

> 因为`Python`的线程虽然是真正的线程，但解释器执行时，会存在一个全局解释器锁(`GIL`锁)，任何`Python`线程执行前，必须先获得`GIL`锁，然后每执行100条字节码，解释器就会自动释放`GIL`锁，让别的线程有机会执行，这个`GIL`锁实际上把所有线程的执行代码都给加了锁，所以，多线程在`Python`中只能交替执行，即使100个线程跑在100核CPU上，也只能用到1个核。

> `GIL`是`Python`解释器设计的历史遗留问题，通常我们用的解释器是官方实现的`CPython`，要真正利用多核，除非重写一个不带`GIL`的解释器。

> 所以，在`Python`中，可以使用多线程，但不要指望能有效利用多核。如果一定要通过多线程利用多核，那只能通过`C`扩展来实现，不过这样就失去了`Python`简单易用的特点。

> 不过，也不用过于担心，`Python`虽然不能利用多线程实现多核任务，但可以通过多进程实现多核任务。多个`Python`进程有各自独立的`GIL`锁，互不影响。

### 10.9. 进程 vs 线程

- **进程**
  - 进程是操作系统资源分配的最小单位
  - 进程拥有独立的地址空间，每启动一次进程，系统就会为它分配地址空间
  - 多进程程序更健壮
  - 多进程模式最大的优点就是稳定性高，因为一个子进程崩溃了，不会影响主进程和其他子进程。
  - 多进程模式的缺点是创建进程的代价大，在`Unix/Linux`系统下，用`fork`调用还行，在`Windows`下创建进程开销巨大。另外，操作系统能同时运行的进程数也是有限的，在内存和`CPU`的限制下，如果有几千个进程同时运行，操作系统连调度都会成问题。
- **线程**
  - 程序执行(调度)的最小单位
  - 线程是共享进程中的数据，使用相同的地址空间，CPU切换一个线程的花费远远小于进程
  - 轻量级的进程，进程之间的数据是独立的，而一个进程下的数据是可以共享的，同一进程下的线程共享全局变量、静态变量等数据，而进程之间的通信需要以通信的方式（IPC)进行。不过如何处理好同步与互斥是编写多线程程序的难点。
  - 线程模式致命的缺点就是任何一个线程挂掉都可能直接造成整个进程崩溃，因为所有线程共享进程的内存。