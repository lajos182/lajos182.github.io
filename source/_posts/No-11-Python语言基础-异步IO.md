---
title: No.11 Python语言基础--异步IO
date: 2020-05-05 22:55:39
tags: [Python, 异步]
toc: true
---

### 11.1. 计算密集型与`IO`密集型

- 计算密集型：要进行大量的计算，消耗CPU资源，比如计算圆周率、对视频进行高清解码等等，全靠CPU的运算能力。这种计算密集型任务虽然也可以用多任务完成，但是任务越多，花在任务切换的时间就越多，CPU执行任务的效率就越低，所以，要最高效地利用CPU，计算密集型任务同时进行的数量应当等于CPU的核心数。计算密集型任务由于主要消耗CPU资源，因此，代码运行效率至关重要。Python这样的脚本语言运行效率很低，完全不适合计算密集型任务。对于计算密集型任务，最好用C语言编写。

<!--more-->

- `IO`密集型：涉及到网络、磁盘`IO`的任务都是`IO`密集型任务，这类任务的特点是CPU消耗很少，任务的大部分时间都在等待`IO`操作完成（因为`IO`的速度远远低于CPU和内存的速度）。对于`IO`密集型任务，任务越多，CPU效率越高，但也有一个限度。常见的大部分任务都是`IO`密集型任务，比如`Web`应用。`IO`密集型任务执行期间，99%的时间都花在`IO`上，花在`CPU`上的时间很少，因此，用运行速度极快的`C`语言替换用`Python`这样运行速度极低的脚本语言，完全无法提升运行效率。对于IO密集型任务，最合适的语言就是开发效率最高（代码量最少）的语言，脚本语言是首选，`C`语言最差。

多线程和多进程的模型虽然解决了并发问题，但是系统不能无上限地增加线程。由于系统切换线程的开销也很大，所以，一旦线程数量过多，CPU的时间就花在线程切换上了，真正运行代码的时间就少了，结果导致性能严重下降。

由于我们要解决的问题是CPU高速执行能力和`IO`设备的龟速严重不匹配，多线程和多进程只是解决这一问题的一种方法。

另一种解决`IO`问题的方法是异步`IO`。当代码需要执行一个耗时的`IO`操作时，它只发出`IO`指令，并不等待`IO`结果，然后就去执行其他代码了。一段时间后，当`IO`返回结果时，再通知CPU进行处理。

### 11.2. 协程

协程(Corontine)，又称微线程、纤程。

子程序，或者称为函数，在所有语言中都是层级调用，比如A调用B，B在执行过程中又调用了C，C执行完毕返回，B执行完毕返回，最后是A执行完毕。所以子程序调用是通过栈实现的，一个线程就是执行一个子程序。

子程序调用总是一个入口，一次返回，调用顺序是明确的。而协程的调用和子程序不同。

协程看上去也是子程序，但执行过程中，在子程序内部可中断，然后转而执行别的子程序，在适当的时候再返回来接着执行。

注意，在一个子程序中中断，去执行其他子程序，不是函数调用，有点类似CPU的中断。比如子程序A、B：

```python
def A():
    print('1')
    print('2')
    print('3')

def B():
    print('x')
    print('y')
    print('z')
```

假设由协程执行，在执行A的过程中，可以随时中断，去执行B，B也可能在执行过程中中断再去执行A，结果如下。可以看出，在A中是没有调用B的，所以协程的调用比函数调用理解起来要难一些。

```python
1
2
x
y
3
z
```

- 看起来A、B的执行有点像多线程，但协程的特点在于是一个线程执行，那和多线程比，协程有何优势？

> **协程执行效率极高**。因为子程序切换不是线程切换，而是由程序自身控制，因此，没有线程切换的开销，和多线程比，线程数量越多，协程的性能优势就越明显。

> **不需要多线程的锁机制**，因为只有一个线程，也不存在同时写变量冲突，在协程中控制共享资源不加锁，只需要判断状态就好了，所以执行效率比多线程高很多。

- 因为协程是一个线程执行，那怎么利用多核CPU呢？

> 最简单的方法是**多进程+协程**，既充分利用多核，又充分发挥协程的高效率，可获得极高的性能

- `Python`对协程的支持如何通过`generator`实现的？

> 在`generato`r中，我们不但可以通过`for`循环来迭代，还可以不断调用`next()`函数获取由`yield`语句返回的下一个值。但是`Python`的`yield`不但可以返回一个值，它还可以接收调用者发出的参数。

传统的生产者-消费者模型是一个线程写消息，一个线程取消息，通过锁机制控制队列核等待，但一不小心就可能死锁。如果改用协程，生产者生产消息后，直接通过`yeild`跳转到消费者开始执行，待消费者执行完毕后，切换生产者继续生产，效率极高。

```python
# 消费者
def consumer():
    r = ''
    while True:
        n = yield r  # 此处的yield接收调用者发出的参数n，并把处理后的结果r返回
        if not n:
            return 
        print(f'[CONSUMER] Consumering {n}...')
        r = '200 OK'

# 生产者
def produce(c):
    c.send(None) # 启动生成器consumer
    n = 0
    while n < 5:
        n += 1
        print(f'[PRODUCER] Producing {n}...')
        r = c.send(n)  # 切换至consumer执行
        print(f'[PRODUCER] Consumer return {r}')
    c.close()

c = consumer()
produce(c)
# 运行结果如下：
>>> [PRODUCER] Producing 1...
[CONSUMER] Consumering 1...
[PRODUCER] Consumer return 200 OK
[PRODUCER] Producing 2...
[CONSUMER] Consumering 2...
[PRODUCER] Consumer return 200 OK
[PRODUCER] Producing 3...
[CONSUMER] Consumering 3...
[PRODUCER] Consumer return 200 OK
[PRODUCER] Producing 4...
[CONSUMER] Consumering 4...
[PRODUCER] Consumer return 200 OK
[PRODUCER] Producing 5...
[CONSUMER] Consumering 5...
[PRODUCER] Consumer return 200 OK
```

> 运行分析：注意到`consumer`函数是一个`generator`，把一个`consumer`传入`produce`后：
>
> > (1)首先调用`c.send(None)`启动生成器；<br/>
> > (2)然后，一旦生产了东西，通过`c.send(n)`切换到`consumer`执行；<br/>
> > (3)`consumer`通过`yield`拿到消息，处理，又通过`yield`把结果传回；<br/>
> > (4)`produce`拿到`consumer`处理的结果，继续生产下一条消息；<br/>
> > (5)`produce`决定不生产了，通过`c.close()`关闭`consumer`，整个过程结束。<br/>

> 整个流程无锁，由一个线程执行，`produce`和`consumer`协作完成任务，所以称为“协程”，而非线程的抢占式多任务。

### 11.3. `asyncio`

`asynio`是`Python`3.4版本引入的标准库，直接内置了对异步`IO`的支持。它的编程模型就是一个消息循环。我们从`asyncio`模块中直接获取一个`EventLoop`的引用，然后把需要执行的协程扔到`EventLoop`中执行，就实现了异步`IO`。

- `asyncio`提供了完善的异步IO支持；
- 异步操作需要在`coroutine`中通过`yield from`完成；
- 多个`coroutine`可以封装成一组`Task`然后并发执行。

```python
import asyncio

# @asyncio.coroutine把一个generator标记为coroutine类型，然后，我们就把这个coroutine扔到EventLoop中执行。
@asyncio.coroutine
def hello():
    print('hello world!')
    # 异步调用asynico.sleep(1)，也是一个coroutine, 所以线程不会等待asynico.sleep()，而是直接中断并执行下一个消息循环。当asyncio.sleep()返回时，线程就可以从yield from拿到返回值（此处是None），然后接着执行下一行语句。asyncio.sleep(1)看成是一个耗时1秒的IO操作，在此期间，主线程并未等待，而是去执行EventLoop中其他可以执行的coroutine了，因此可以实现并发执行。
    r = yield from asyncio.sleep(10)
    print('hello world again!')

# 获取EventLoop
loop = asyncio.get_event_loop()
# 执行coroutine
loop.run_until_complete(hello())
loop.close()
>>> hello world!
hello world again!
```

使用`task`封装两个`coroutine`:

```python
import asyncio
import threading

@asyncio.coroutine
def hello():
    print(f'hello world! ({threading.current_thread()})')
    yield from asyncio.sleep(1)
    print(f'hello world agagin! ({threading.current_thread()})')

loop = asyncio.get_event_loop()
tasks = [hello(), hello()]
loop.run_until_complete(asyncio.wait(tasks))
loop.close()
# 运行结果如下：
>>> hello world! (<_MainThread(MainThread, started 4020)>)
hello world! (<_MainThread(MainThread, started 4020)>)
hello world agagin! (<_MainThread(MainThread, started 4020)>)
hello world agagin! (<_MainThread(MainThread, started 4020)>)
# 由打印的当前线程名称可以看出，两个coroutine是由同一个线程并发执行的。
```

案例：用`asyncio`的异步网络连接`sina`、`souhu`和163的网站首页

```python
import asyncio

@asyncio.coroutine
def wget(host):
    print(f'wget {host}...')
    # 创建TCP客户端并连接服务器，或者说创建一个TCP连接对象
    # open_connection接收两个参数：主机和端口号
    # connect是协程，这步仅是创建协程对象，立即返回，不阻塞
    connect = asyncio.open_connection(host, 80)
    # 连接创建成功后，asyncio.open_connection方法的返回值就是读写对象
    # 读写对象分别为StreamReader和StreamWriter实例
    # 它们也是协程对象，底层调用socket模块的send和recv方法实现读写
    reader, writer = yield from connect
    # header 是发送给服务器的消息，意为获取页面的 header 信息
    # 这个格式是固定的，见下图
    header = f'GET / HTTP/1.0\r\nHost: {host}\r\n\r\n'
    # 给服务器发消息，注意消息是二进制的
    writer.write(header.encode('utf-8'))
    # 这是一个与底层IO输入缓冲区交互的流量控制方法
    # 当缓冲区达到上限时，drain()阻塞，待到缓冲区回落到下限时，写操作恢复
    # 当不需要等待时，drain()会立即返回，例如上面的消息内容较少，不会阻塞
    # 这就是一个控制消息的数据量的控制阀
    yield from writer.drain()
    # 给服务器发送消息后，就等着读取服务器返回来的消息
    while True:
        # 读取数据是阻塞操作，释放CPU
        # reader相当于一个水盆，服务器发来的数据是水流
        # readline表示读取一行，以\n作为换行符
        # 如果在出现\n之前，数据流中出现EO（End Of File文件结束符）也会返回
        # 相当于出现\n或EOF时，拧上水龙头，line就是这盆水
        line = yield from reader.readline()
        # 数据接收完毕，会返回空字符串\r\n，退出while循环，结束数据接收
        if line.decode('utf-8') == '\r\n':
            break
        # 接收的数据是二进制数据，转换为 UTF-8 格式并打印
        # rstrip 方法删掉字符串的结尾处的空白字符，也就是 \n
        print(f'{host} header > {line.decode("utf-8").rstrip()}')
    writer.close()   # 关闭数据流，可以省略

loop = asyncio.get_event_loop()
tasks = [wget(host) for host in ['www.sina.com.cn', 'www.sohu.com', 'www.163.com']]
loop.run_until_complete(asyncio.wait(tasks))
loop.close()
# 运行结果如下(3个连接由一个线程通过coroutine并完成)：
>>> wget www.163.com...
wget www.sohu.com...
wget www.sina.com.cn...
www.sohu.com header > HTTP/1.1 200 OK
www.sohu.com header > Content-Type: text/html;charset=UTF-8
www.sohu.com header > Connection: close
www.sohu.com header > Server: nginx
www.sohu.com header > Date: Thu, 14 Nov 2019 03:07:07 GMT
www.sohu.com header > Cache-Control: max-age=60
www.sohu.com header > X-From-Sohu: X-SRC-Cached
www.sohu.com header > Content-Encoding: gzip
www.sohu.com header > FSS-Cache: HIT from 111148311605210.19052918
www.sohu.com header > FSS-Proxy: Powered by 9410870.10197312.17348930
www.163.com header > HTTP/1.1 200 OK
www.163.com header > Date: Thu, 14 Nov 2019 03:07:56 GMT
www.163.com header > Content-Type: text/html; charset=GBK
www.163.com header > Connection: close
www.163.com header > Expires: Thu, 14 Nov 2019 03:09:17 GMT
www.163.com header > Server: nginx
www.163.com header > Cache-Control: no-cache,no-store,private
www.163.com header > Vary: Accept-Encoding
www.163.com header > X-Ser: BC51_dx-lt-yd-shandong-jinan-5-cache-6, BC51_dx-lt-yd-shandong-jinan-5-cache-6, BC138_dx-zhejiang-zhou-1-cache-3
www.163.com header > cdn-user-ip: 60.176.229.93
www.163.com header > cdn-ip: 115.231.128.142
www.163.com header > X-Cache-Remote: HIT
www.163.com header > cdn-source: baishan
www.sina.com.cn header > HTTP/1.1 302 Moved Temporarily
www.sina.com.cn header > Server: nginx
www.sina.com.cn header > Date: Thu, 14 Nov 2019 03:07:56 GMT
www.sina.com.cn header > Content-Type: text/html
www.sina.com.cn header > Content-Length: 154
www.sina.com.cn header > Connection: close
www.sina.com.cn header > Location: https://www.sina.com.cn/
www.sina.com.cn header > X-Via-CDN: f=edge,s=ctc.nanjing.ha2ts4.63.nb.sinaedge.com,c=60.176.229.93;
www.sina.com.cn header > X-Via-Edge: 15737008765135de5b03c7c5e66ca1bf1580f
```

### 11.4. `async`/`await`

为了简化并更好地标识异步`IO`，从`Python`3.5开始引入新的语法`async`和`await`，可以让`corountine`的代码更简洁易读。与`Python`3.4的`asyncio`和`yield from`的区别：

- 把`@asyncio.coroutine`替换为`async`
- 把`yield from`替换为`await`。

```python
import asyncio

async def hello():
    print('hello world!')
    r = await asyncio.sleep(10)
    print('hello world again!')

# 获取EventLoop
loop = asyncio.get_event_loop()
# 执行coroutine
loop.run_until_complete(hello())
loop.close()
>>> hello world!
hello world again!
```

### 11.5. `aiohttp`

`aiohttp`可以实现单线程并发`IO`操作。如果仅用在客户端，发货的威力不大。如果把`asyncio`用在服务端，例如`Web`服务器，由于`HTTP`连接就是`IO`操作，因此可以用单线程+`coroutine`实现多用户的高并发。

`asyncio`实现了`TCP`、`UDP`、`SSL`等协议，`aiohttp`则是基于`asyncio`实现的`HTTP`框架。

- 安装`aiohttp`：`pip install aiohttp`
- 编写`HTTP`服务器，分别处理以下`URL`:
  - `/`-首页返回`b'<h1>Index</h1>'`；
  - `/hello/{name}`-根据`URL`参数返回文本`hello, %s!`。
- 用`aiohttp`创建服务

```python
import asyncio

from aiohttp import web

async def index(request):
    await asyncio.sleep(0.5)
    return web.Response(body=b'<h1>Index</h1>', content_type='text/html')

async def hello(request):
    await asyncio.sleep(0.5)
    text = f'<h1>hello, {request.match_info["name"]}</h1>'
    return web.Response(body=text.encode('utf-8'), content_type='text/html')

async def init(loop):
    app = web.Application(loop=loop)
    app.router.add_route('GET', '/', index)
    app.router.add_route('GET', '/hello/{name}', hello)
    runner = web.AppRunner(app)
    srv = await loop.create_server(runner, '127.0.0.1', 9000)
    print('Server started at http://127.0.0.1:9000...')
    return srv
```