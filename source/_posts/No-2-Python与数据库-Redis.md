---
title: No.2 Python与数据库--Redis
date: 2020-05-13 18:29:47
tags: [Python, Redis]
toc: true
---

`Redis`是一个开源的使用`ANSI C语言`编写、支持网络、可基于内存亦可持久化的日志型、`Key-Value`数据库。它是一种**非关系型数据库**，经常用过缓存，拥有丰富的数据类型：字符串、哈希、列表、集合和有序集合。

### 2.1. `Redis`的安装与连接测试

+ 直接安装：`sudo apt-get install redis-server`，该安装只是`Linux`的稳定版本，如果想安装`Redis`官方稳定版本需要采用源码安装

<!--more-->

+ 源码安装：
  - 官网`redis.io`下载稳定的最新版本，比如`redis-5.0.7.tar.gz`
  - 安装`tcl`：`sudo apt-get install tcl`
  - 解压下载的安装包，将解压后的文件放至指定的安装目录，`sudo mv redis-5.0.7 /usr/local/redis  cd /usr/local/redis`
  - 编译安装：`sudo make`——>`sudo make test`——>`sudo make install`
  - 测试：`/usr/local/redis/src/redis-server`——>`/usr/local/redis/src/redis-cli`——>`set name lajos`——>`get name`
  - 创建相关目录：`sudo makdir /etc/redis`(配置文件路径)——>`sudo mkdir /var/lib/redis`(redis数据库存储路径)
  - 安装服务：`cd /usr/local/redis/utils`——>`sudo ./install_server.sh`
  - 重启服务：`ps ajx | grep redis`——>`sudo kill -9 对应的进程号`
  - 测试：`redis-server`——>`redis-cli`
  - 配置文件：`cd /etc/redis`——>`sudo vim /etc/redis/6379.conf`——>可以修改`bind`、`daemonize`、`requirepass`——>然后重启服务
  - 直接开启客户机：`redis-cli`
+ 连接测试
  - 检查服务是否启动：`ps aux(ajx, -ef) | grep redis`
  - 连接：`redis-cli -h 主机 -p 端口 -a 密码`
  - 退出：`quit`
+ 密码管理：默认没有密码，若设置密码，需要通过`auth`认证
  - 单次有效：
    - 获取密码：`config get requirepass`
    - 设置密码：`config set requirepass 123456`
    - 密码认证：`auth 123456`
  - 永久有效：修改配置文件`/etc/redis/redis.conf`
    - 将`requirepass xxxxx`行修改设置的密码
    - 重新启动`redis`服务

### 2.2. `Redis`命令

+ 常用

```markdown
ping：测试连接情况，回复PONG表示OK
quit：退出连接
auth：密码认证
select：选择库，总共有16个，序号0~15，默认是第0个
info：查看服务器信息
command：查看支持的命令
flushdb：清空当前库
flushall：清空所有库
save：前台进行持久化存储
bgsave：后台进行持久化存储
exists：判断指定键是否存在
del：删除键
```

+ 字符串(`string`)

```markdown
set：设置
get：获取
getset：获取之后再设置
mset：设置多个
mget：获取多个
incr：加1
decr：减1
incrby：加指定的值
decrby：减指定的值
append：追加内容
```

+ 哈希(`hash`)

```markdown
hset：设置单个属性
hget：获取单个属性
hmset：设置多个属性
hmget：获取多个属性
hgetall：获取所有属性
hdel：删除键值对
hexists：判断指定键的指定字段是否存在
hkeys：获取指定键的所有属性名
hvals：获取指定键的所有属性值
hlen：获取指定键的属性个数
```

+ 列表(`list`)

```markdown
lpush：从左边(头部)插入数据
lpop：从左边(头部)删除数据
lrange：获取指定区间范围内的数据，0 -1表示所有
lindex：根据索引获取元素
llen：获取列表长度(元素个数)
rpush：从右边(尾部)插入数据
rpop：从右边(尾部)删除数据
lrem：移除指定个数的指定元素
```

+ 集合(`set`)

```markdown
sadd：添加元素
scard：统计元素个数
sdiff：求差集
sinter：求交集
smembers：获取指定集合的所有成员
sismember：判断是否在集合中
smove：移动元素
spop：随机的删除一个元素
srandmember：随机获取指定个数的元素
srem：移除元素
sunion：求并集
```

+ 有序集合(`sorted set`)

```markdown
zadd：添加元素
zcard：统计个数
zcount：指定分数区间内的元素数量
zrange：返回指定索引范围内的元素
zrangebyscore：返回指定分数区间的元素
zrank：返回指定成员的索引值
zrem：移除元素
zscore：获取指定元素的排序指标(分数)
```

### 2.3. `Python`操作`Redis`

`redis`扩展库中有两个类，`Redis`和`StrictRedis`，`StrictRedis`实现了官方的命令，`Redis`是它的子类，兼容老版本。扩展库没有实现`select`方法，可以通过连接时指定使用的库。

+ 安装扩展：`pip install redis`

#### 2.3.1. 简单连接

+ `redis.Redis(host='localhost', port=6379, db=0, password=None, socket_timeout=None, socket_connect_timeout=None,socket_keepalive=None, socket_keepalive_options=None, connection_pool=None, unix_socket_path=None,encoding='utf-8', encoding_errors='strict', charset=None, errors=None, decode_responses=False, retry_on_timeout=False, ssl=False, ssl_keyfile=None, ssl_certfile=None, ssl_cert_reqs='required', ssl_ca_certs=None, max_connections=None, single_connection_client=False, health_check_interval=0)`：`db`表示选定的数据库，`decode_response`表示解码返回

```python
import redis
# 创建连接对象，所有的命令在代码中都是Redis对象的方法
r = redis.Redis(host='localhost', port=6379, db=0, password=None, decode_response=True)
r.set('name', 'lajos')
print(r.get('name))
```

#### 2.3.2. 连接池

多个`redis`对象使用同一个连接池进行连接，避免了多次连接、断开等操作的系统开销。默认，每个Redis实例都会维护一个自己的连接池。可以直接建立一个连接池，然后作为参数Redis，这样就可以实现多个Redis实例共享一个连接池。

```python
import redis

# 创建连接池
pool = redis.ConnectionPool(host='localhost', port=6379, db=0, decode_responses=True)

# 创建Redis对象
r = redis.Redis(connection_pool=pool)
r.set('name', 'lajos')
print(r.get('name'))
```

#### 2.3.3. 详细操作命令

[`redis`详细操作命令](https://www.cnblogs.com/melonjiang/p/5342505.html)

#### 2.3.4. 管道(`pipeline`)

+ 可以记录多个操作，然后一次将操作发送至数据库，避免了多次连接、断开等操作的系统开效

+ 多个操作可以依次进行保存，然后发送，也可以进行连贯操作

```python
import redis

# 创建连接池
pool = redis.ConnectionPool(host='localhost', port=6379, db=0, decode_responses=True)
# 创建Redis对象
r = redis.Redis(connection_pool=pool)
# 创建管道, 默认支持事务(transaction=True)
pipe = r.pipeline()
# 记录命令
pipe.set('name', 'lajos')
pipe.set('age', 20)
pipe.set('message', 'lajos is a handsome man')
dic = {'a': 'aaa', 'b': 'bbb'}
pipe.hmset('myhash', dic)
# 执行操作
pipe.execute()
print(r.get('message'))
print(r.hget('myhash', 'b'))
```

#### 2.3.5. 发布和订阅

`Redis`发布订阅(`pub/sub`)是一种消息通信模式：发送者(`pub`)发送消息，订阅者(`sub`)接收消息。

```python
import redis

class RedisHelper(object):

    def __init__(self, channel=None):
        self.__connection = redis.Redis(host='127.0.0.1', port=6379, decode_responses=True)
        self.channel = 'lajos'
    # 定义发布方法
    def publish(self, msg):
        self.__connection.publish(self.channel, msg)
        return True
    # 定义订阅方法
    def subscribe(self):
        pub = self.__connection.pubsub()
        pub.subscribe(self.channel)
        pub.parse_response()
        return pub
```

+ 发送者

```python
from pythonRedis import RedisHelper

pub = RedisHelper()
while True:
    msg = input('发布者：')
    pub.publish(msg)
# 运行结果
>>> 发布者：hello
发布者：world
发布者：
```

+ 订阅者

```python
from pythonRedis import RedisHelper

sub = RedisHelper()
redis_sub = sub.subscribe()

while True:
    msg = redis_sub.parse_response()
    print(msg)
# 运行结果
>>> ['message', 'lajos', 'hello']
['message', 'lajos', 'world']

```