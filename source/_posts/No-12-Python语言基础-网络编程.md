---
title: No.12 Python语言基础--网络编程
date: 2020-05-12 11:41:14
tags: [Python, 网络编程]
toc: true
---

### 12.1. 相关概念

+ `OSI`七层模型：开放系统互连参考模型，包含物理层、数据链路层、网络层、传输层、会话层、表示层和应用层

<!--more-->

![`OSI`七层模型](https://gitee.com/lajos/image_bucket/raw/master/skill/OSI.png)

+ `TCP/IP`：在`OSI`七层模型的基础进行简化抽象出来的广泛使用的网络协议簇

+ `TCP协议`(传输控制协议)：

  - 完成`OSI`模型中的第四层传输层指定的功能
  - 有链接的，数据是安全的(有保障)
  - 传输的速度相对较慢，三次握手、四次挥手、数据检查

+ `UDP`(用户数据报协议)：

  - 无连接的，数据是不可靠的
  - 传输速度相对较快

![网络编程相关概念汇总](https://gitee.com/lajos/image_bucket/raw/master/skill/network%20program.png)

+ `http`请求的结构及详细解释可以查看[程序猿必须要了解的HTTP协议，今天总结了一份关于HTTP请求的结构与具体内容](https://blog.csdn.net/sinat_41898105/article/details/80951072)

+ `http`响应的结构及详细解释可以查看[程序猿必须要了解的HTTP协议，总结了一份关于HTTP响应的结构及详细解释](https://blog.csdn.net/sinat_41898105/article/details/80952338)

### 12.2. `TCP`编程

`Socket`是网络编程的一个抽象概念。通常我们用一个`Socket`表示打开了一个网络链接，而打开一个`Socket`需要直到目标计算机的`IP`地址和端口号，再指定协议类型即可。

+ `TCP`是建立可靠连接，并且通信双方都可以以流的形式发送数据。

![`TCP`工作原理](https://gitee.com/lajos/image_bucket/raw/master/skill/tcp.png)

+ 客户端：大多数连接都是可靠的`TCP`连接，创建`TCP`连接时，主动发起连接的叫客户端。

+ 服务端：绑定一个端口并监听来自其他客户端的连接。如果某个客户端连接过来了，服务器就与该客户端建立Socket连接，随后的通信就靠这个Socket连接了。

+ `skt = socket.soket(family=AF_INET, type=SOCK_STREAM, proto=0, fileno=None)`: `family`表示协议簇，`socket.AF_INET`表示`IPv4`，`socket.AF_INET6`表示`IPv6`；`type`表示传输协议，`socket.SOCK_STREAM`表示`TCP`, `socket.SOCK_DGRAM`表示`UDP`

+ `skt.connect(address)`：向服务器发出请求，`address`表示(主机/域名, 端口)的一个元组

+ `skt.send()`：发送请求报文，常见的内容`b'GET / HTTP/1.1\r\nHost: www.baodu.com\r\nConnection: close\r\n\r\n'`

+ `skt.recv()`：接收服务器的数据，字节类型，需要`decode`解码

+ `skt.bind()`：绑定主机和端口，接收一个元组类型的数据

+ `skt.listen()`：服务端监听服务

+ `skt.accept()`：服务端等待阻塞，一旦有客户请求数据，便创建新的套接字进行通讯，同时可以获取客户的请求地址

+ `skt.close()`：关闭套接字

> 使用`tcp`模拟客户端连接百度服务器

```python
# tcp-http模拟(tcp传输层, http应用层)
import socket
# 创建套接字
skt = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
# 与服务器建立连接
skt.connect(('www.baodu.com', 80))
# 向服务器发送消息
skt.send(b'GET / HTTP/1.1\r\nHost: www.baodu.com\r\nConnection: close\r\n\r\n')
# 接收服务器的数据
data = []
while True:
    tmp = skt.recv(1024)
    if tmp:
        data.append(tmp)
    else:
        break
headers, content = b''.join(data).decode('utf-8').split('\r\n\r\n', 1)
print(headers)
# 运行结果
>>> HTTP/1.1 200 OK
Server: nginx
Date: Fri, 15 Nov 2019 06:16:49 GMT
Content-Type: text/html
Content-Length: 16831
Last-Modified: Mon, 27 May 2019 05:10:18 GMT
Connection: close
Vary: Accept-Encoding
ETag: "5ceb713a-41bf"
Accept-Ranges: bytes
```

> 使用tcp创建soket服务

```python
import socket
import threading

def tcplink(client_skt, client_addr):
    print(f'接收到来自{client_addr}新的连接')
    while True:
        recv_data = client_skt.recv(1024)
        print('客户端>>>：', recv_data.decode('utf-8'))
        client_skt.send(recv_data)
    client_skt.close()


if __name__ == "__main__":
    # 创建套接字
    skt = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    # 绑定主机和端口
    skt.bind(('192.168.199.178', 8888))
    # 监听服务
    skt.listen()
    # 阻塞，等待客户端的连接，一旦有连接，会创建新的套接字进行通信
    while True:
        client_skt, client_addr = skt.accept()
        # 创建一个新线程处理连接
        t = threading.Thread(target=tcplink, args=(client_skt, client_addr))
        t.start()
        t.join()
    skt.close()
```

> 使用tcp模拟客户端

```python
import socket

# 创建套接字
skt = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
# 建立连接
skt.connect(('192.168.199.178', 8888))
while True:
    data = input('>>>：')
    skt.send(data.encode('utf-8'))
    # 接收服务器返回的消息
    recv_data = skt.recv(1024)
    print('服务器：', recv_data.decode('utf-8'))
skt.close()
```

### 12.3. `UDP`编程

+ `UDP`是面向无连接的协议，使用`UDP`协议时，不需要建立连接，只需要知道对方的`IP`地址和端口号，就可以直接发数据包。

![`UDP`工作原理](https://gitee.com/lajos/image_bucket/raw/master/skill/udp.png)

+ `skt.sendto()`：发送`UDP`数据

+ `skt.bind()`：绑定主机和端口

+ `skt.recvfrom()`：接收`UDP`数据

+ `skt.close()`：关闭`Socket`连接


> 采用`UDP`模拟飞秋发送消息

```python
import socket

# 创建UDP套接字，与TCP基本一样，使用协议: socket.SOCKET_DGRAM
skt = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)

# 版本号:数据包序列号:用户名:主机名:命令:消息内容[:附加数据]
data = '99:12345:lajos:windows:32:您好，欢迎进入飞秋'

for i in range(1, 255):
    ip = '192.168.12.' + str(i)
    addr = (ip, 2425)
    skt.sendto(data.encode('gbk'), addr)

skt.close()
```

> `UDP`协议创建服务端

```python
import socket

# 创建UDP套接字
skt = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
# 绑定IP和端口
skt.bind(('192.168.12.165', 8888))
while True:
    # 阻塞，等待接口客户端的数据
    recv_data, client_addr = skt.recvfrom(1024)
    # 打印客户端发来的数据
    print(client_addr, recv_data.decode('utf-8'))
    # 给客户端发送消息
    skt.sendto(recv_data, client_addr)
skt.close()
```

> `UDP`协议创建客户端

```python
import socket

# 创建UDP套接字
skt = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
while True:
    # 从终端获取输入
    data = input('>>>：')
    # 发送获取的内容
    skt.sendto(data.encode('utf-8'), ('192.168.12.165', 8888))
    # 获取服务端的数据
    recv_data, server_addr = skt.recvfrom(1024)
    print('服务器：', server_addr, recv_data.decode('utf-8'))
# 绑定IP和端口
skt.close()
```

