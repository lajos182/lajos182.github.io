---
title: No.3 Python与数据库--MongoDB
date: 2020-05-14 09:12:18
tags: [Python, MongoDB]
toc: true
---

### 3.1. `MongoDB`简介

+ MongoDB是一个基于分布式文件存储 的数据库。由`C++`语言编写。旨在为WEB应用提供可扩展的高性能数据存储解决方案。它是一个介于关系数据库和非关系数据库之间的产品，是非关系数据库当中功能最丰富，最像关系数据库的。更多的介绍可查看，[菜鸟教程(https://www.runoob.com/mongodb/mongodb-intro.html](https://www.runoob.com/mongodb/mongodb-intro.html)

  <!--more-->

+ `MongoDB`的安装(`linux`)

  - 直接安装：`sudo apt-get install mongodb`，此安装可能无法安装最新稳定版本

  - 自定义安装：

    - 按转包下载：到`MongoDB`官方网站[https://www.mongodb.com/download-center/community](https://www.mongodb.com/download-center/community)下载对应版本的安装包，例如，下载`mongodb-linux-x86_64-ubuntu 16.04-4.2.1.tgz`，对应的官网链接为`https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-ubuntu1604-4.2.1.tgz`
      ![`MongoDB`的下载](https://gitee.com/lajos/image_bucket/raw/master/skill/mongodb_download.png)

    - 使用`curl`命令下载：`curl -O https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-ubuntu1604-4.2.1.tgz`

    - 解压：`tar -xzvf mongodb-linux-x86_64-ubuntu1604-4.2.1.tgz`

    - 将解压包拷贝至指定没目录：`sudo mv mongodb-linux-x86_64-ubuntu1604-4.2.1 /usr/local/mongodb`

    - 添加至系统环境变量：`sudo vim /etc/profile`——>在最后一行添加`export PATH=/usr/local/mongodb/bin:$PATH`——>`source /etc/profile`

    - 添加相关配置文件：`sudo vim /etc/mongod.conf`

      ```bash
      verbose=true
      port=27017
      logpath=/var/log/mongodb/logs/mongodb.log
      logappend=true
      dbpath=/var/lib/mongodb/db
      directoryperdb=true
      auth=false
      fork=true
      quiet=true
      ```

    - 创建数据库存储、日志文件：`sudo mkdir /var/log/mongodb/logs/ -p`——>`sudo touch /var/log/mongodb/logs/mongodb.log`——>`sudo mkdir /var/lib/mongodb/db –p`
    - 注册开机启动：`sudo vim /ect/init.d/mongodb`

        ```shell
        #!/bin/sh
        ### BEGIN INIT INFO
        # Provides: mongodb
        # Required-Start:
        # Required-Stop:
        # Default-Start: 2 3 4 5
        # Default-Stop: 0 1 6
        # Short-Description: mongodb
        # Description: mongo db server
        ### END INIT INFO
        . /lib/lsb/init-functions
        PROGRAM=/usr/local/mongodb/bin/mongod
        MONGOPID=`ps -ef | grep 'mongod' | grep -v grep | awk '{print $2}'`
        test -x $PROGRAM || exit 0
        case "$1" in
        start)
        ulimit -n 3000
        log_begin_msg "Starting MongoDB server"
        $PROGRAM -f /etc/mongod.conf
        log_end_msg 0
        ;;
        stop)
        log_begin_msg "Stopping MongoDB server"
        if [ ! -z "$MONGOPID" ]; then
        kill -15 $MONGOPID
        fi
        log_end_msg 0
        ;;
        status)
        ;;
        *)
        log_success_msg "Usage: /etc/init.d/mongodb {start|stop|status}"
        exit 1
        esac
        exit 0
        ```

        `sudo chmod +x /etc/init.d/mongodb`

    - 注册开机脚本：`sudo update-rc.d mongodb defaults`(注意：移除使用`sudo update-rc.d –f mongodb remove`)
    - 启动服务：`sudo service mongodb start`
    - 客户端连接：`mongo`(若无法连接，可查看环境变量，或者重新开启会话)

  - 官方安装：

    - 导入`MongoDB`公共`GPG`密钥：`wget -qO - https://www.mongodb.org/static/pgp/server-4.2.asc | sudo apt-key add -`(注意是大写`O`)，如果收到指示`gnupg`未安装的错误，则输入命令`sudo apt-get install gnupg`，安装成功后重新输入`GPG`密钥
    - 创建`MongoDB`相关文件：选择合适的`MongoDB`版本，例如`Ubuntu16.04`，需要执行命令`echo "deb [ arch=amd64 ] https://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/4.2 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.2.list`
    - 重新加载本地软件包数据库：`sudo apt-get update`
    - 安装`MongoDB`：可以选择安装最新版本(`sudo apt-get install -y mongodb-org`)，也可指定版本安装(`sudo apt-get install -y mongodb-org=4.2.1 mongodb-org-server=4.2.1 mongodb-org-shell=4.2.1 mongodb-org-mongos=4.2.1 mongodb-org-tools=4.2.1`)。
    - 配置文件：`/ect/mongod.conf`
    - 启动服务：`sudo service mongod start`

+ `MongoDB`与`MySQL`比较

    | MySQL    | MongoDB          |
    | -------- | ---------------- |
    | database | db(数据库)       |
    | table    | collection(集合) |
    | 一条数据 | document(文档)   |

### 3.2. `MongoDB`基本操作

+ `db`操作

  - 查看所有数据库：`show dbs`
  - 查看当前数据库：`db`或者`db.getName()`
  - 创建或切换数据库：`use python`
  - 若使用的数据库不存在则创建然后切换，存在直接切换
  - 新建的数据库没有数据时，`show dbs`显示不出来
  - 删除当前使用的数据库：`db.dropDatabase()`
  - 退出：`exit`或`quit()`
  - 查看帮助：`help`

+ `collection`操作

  - 查看当前数据库的所有集合：`show collections`
  - 创建集合：
    - 单纯创建：`db.createCollection('star')`，创建一个空集合
    - 按需创建：`db.stu.insert({name:'lufei', age:22})`，直接使用不存在的集合，并插入数据
  - 删除当前数据库指定的集合：`db.stu.drop()`

+ `document`操作

  - 增加文档：

    - `insert`

      - 插入一条：`db.star.insert({name:'linjunjie', age:32, treasure:200, isDelete:0})`
      - 插入多条：

    ```mongodb
    db.star.insert([
    {name:'zhoujielun', age:41, treasure:500, isDelete:0}, 
    {name: 'baijingting', age:26, treasure:80, isDelete:0}
    ])
    ```

    - `save`

    ```moogodb
    # 不指定_id字段，功能等价于insert
    db.star.save({name:'xiaozhan',age:31,treasure:90,isDelete:1})
    # 指定_id字段，相当于更新操作
    db.star.save({_id:ObjectId("5dd63af30246a3087b2054ce"), name:"xiaozhan", age: 31, treasure: 90, isDelete: 0 })
    ```

  - 更新文档：

    - `save`：保存文档时，需要指定`_id`字段
    - `update`：`db.集合.update(query, update, {upsert: <boolean>, multi: <boolean>})`

        ```mongodb
        db.star.update({name: 'zhoujielun'}, {$set: {age: 40}})
        db.star.update({isDelete: 0}, {$inc: {age: 3}})
        db.star.update({isDelete: 0}, {$set: {isDelete: 1}}, {multi: true})
        db.star.update({name: 'zhoujie'}, {$set: {age: 40}}, {upsert: true})
        ```

        > `query`：表示查询条件<br>
        > `update`：表示跟新设置<br>
        > `$set`：表示设置<br>
        > `$inc`：表示增加<br>
        > `upsert`：更新的查询结果不存在，是否作为新数据插入<br>
        > `multi`：符合条件的数据是否全部更新，`true`表示全部更新，`false`只更新一条，默认为`false`

  - 删除文档：`remove`：`db.集合.remove(query, {justOne: <boolean>})`，默认为`false`

      ```mongodb
      # 默认删除所有匹配到的文档
      db.star.remove({name: 'zhoujie'})
      # 只删除一条匹配到的文档
      db.star.remobe({age: 26}, {justOne: true})
      ```

  - 查询文档：`db.star.find()`

    - 格式：`db.集合.find(query, {key1: 0, key2: 0})`

    ```mongodb
    db.star.find({name: 'xiaozhan'})
    db.star.find({name: 'xiaozhan'}, {name: 1, age: 1})
    ```

    > `query`表示查询条件；<br>
    > 第二个参数值为1表示要显示的字段，为0表示不显示的字段；<br>
    > 选择字段时或排序字段只能选择一种情况，要么都选择，要么都排除

    - `pretty()`：格式化显示文档以字典的形式，`db.star.find().pretty()`
    - `findOne()`：查询一条文档，`db.star.findOne()`
    - `count()`：统计查询文档的个数

### 3.3. `MongoDB`进阶操作

+ 查询条件操作符
  - 大于：`$gt`，格式是`db.集合.find({<key>: {$gt: <value>}})`，如`db.star.find({age: {$gt: 35}})`
  - 大于等于：`$gte`
  - 小于：`$lt`
  - 小于等于：`$lte`
  - 等于：不写符合就是等于
  - 多个条件：`db.star.find({age: {$gt: 31, $lt: 40}})`
  - 正则：格式是`db.集合.find({<key>: /正则表达式/})`，如`db.star.find({name: /an$/})`
+ 条件合并`and/or`
  - `and`：表示并且，格式：`db.集合.find({条件1，条件2})`，如，`db.star.find({isDelete: 0, age: {$gt: 35}})`
  - `or`：表示或者，格式：`db.集合.find({$or: [{条件1}，{条件2}])`，如，`db.star.find({$or: [{age: {$gt: 35}}, {age: {$lt: 31}}]})`
+ 限制结果集
  - 限制：`limit`，如，`db.star.find().limit(2)`
  - 跳过：`skip`，如，`db.star.find().skip(1).limit(1)`
+ 结果集排序
  - 格式：`db.集合.find().sort({<key>: 1/-1})`，1表示升序，-1表示降序
  - 示例：`db.star.find().sort({age: 1})`

### 3.4. 权限认证

+ `mongod.conf`配置文件：

  ```shell
  #  mmapv1:
  #  wiredTiger:
  
  # where to write logging data.
  systemLog:
    destination: file
    logAppend: true
    path: /var/log/mongodb/mongod.log
    quiet: true
    # timeStampFormat: iso8601-utc 
  
  # nework interfaces
  net:
   port: 27017
   bindIp：127.0.0.1
   
  # how the process runs
  processManagement:
    timeZoneInfo：/usr/share/zoneinfo
    
  security:
    authorization: enabled
    
  #operationsProfiling:
  
  #replication
  
  #sharding
  
  ## Enterprise-Only Options:
  
  #auditLog
  
  #snmp
  ```

  > - **`systemLog`**：
  >   - `systemLog.verbosity`：`integer`，日志文件输入的级别，越大级别越低
  >   - `systemLog.quiet`：`boolean`，该模式下会限制输出信息，数据库命令输出，副本集活动，连接接受事件，连接关闭事件。
  >   - `systemLog.traceAllExceptions`：`string`，打印`verbose`信息来调试，用来记录额外的异常日志
  >   - `systemLog.syslogFacility`：`string`，默认为`user`，指定`syslog`日志信息的设备级别。需要指定`--syslog`来使用这个选项
  >   - `systemLog.path`：`string`，发送所有的诊断日志，默认重启后会覆盖
  >   - `systemLog.logAppend`：`boolean`，是否启用追加日志
  >   - `systemLog.destination`：`string`，指定一个文件或`syslog`，如果指定为文件，必须同时指定`systemLog.path`
  >   - `systemLog.timeStampFormat`：`string`，默认为`iso8601-local`
  > - **`processManagement`**：
  >   - `processManagement.pidFilePath`：`string`，指定进程的`ID`，与`--fork`配合使用，不指定则不会创建
  >   - `processManagement.fork`：`boolean`，默认为`false`，`true`表示守护进程在后台运行
  >   - `processManagement.timeZoneInfo`：`string`，如，`/usr/share/zoneinfo`，时区
  > - `net`：
  >   - `net.port`：`interger`，默认`27017`，`mongodb`实例监听的端口号
  >   - `net.bindIp`：`string`，2.6版本默认为`127.0.0.1 `，指定`mongodb`实例绑定的`ip`，为了绑定多个`ip`，可以使用逗号分隔
  >   - `net.maxIncomingConnections`：`integer`， 默认为1000000，`mongodb`实例接受的最多连接数，如果高于操作系统接受的最大线程数，设置无效
  >   - `net.wireObjectCheck`：`boolean`，默认为`true `，检查文档的有效性。会稍微影响性能
  >   - `net.http.enabled`：`boolean`，默认为`false`， 打开`http`端口，会导致更多的不安全因素
  >   - `net.unixDomainSocket.enabled`：`boolean`，默认为`false`， 停止`UNIX domain socket`监听。 `mongodb`实例会一直监听`UNIX socket`，除非`net.unixDomainSocket.enabled`设置为`true`，`bindIp`没有设置，`bindIp`没有默认指定为`127.0.0.1`
  >   - `net.unixDomainSocket.pathPrefix`：`string`，默认为`/tmp unix Socket`所在的路径
  >   - `net.ipv6`：`boolean`，默认为`false`，打开`IPV6`功能，默认为关闭的
  >   - `net.http.JSONPEnabled`：`boolean`，默认为`false`，运行`json`访问`http`端口，打开会导致更多的不安全因素
  >   - `net.http.RESTInterfaceEnabled`：`boolean`，默认为`false`， 即使`http`接口选项关闭，打开也会暴露`http`接口，会导致更多的不安全因素。
  > - **`security`**：
  >   - `security.keyFile`：`string`， 指定分片集或副本集成员之间身份验证的`key`文件存储位置
  >   - `security.clusterAuthMode`：`string`， 集群认证中利用到这个模式，如果使用`x.509`安全机制，可以在这里指定
  >   - `keyFile,sendKeyFile,sendX509,x509`：默认的`mongodb`发行版是不支持`ssl`的，可以使用专业版的或重新自行编译`mongodb`
  >   - `security.authorization`：`string`，默认为`disabled` ，`enabled`表示打开访问数据库和进行操作的用户角色认证
  > - **`operationProfiling`**：
  >   - `operationProfiling.slowOpThresholdMs`：`integer`，默认100 指定慢查询时间，单位毫秒，如果打开功能，则向`system.profile`集合写入数据
  >   - `operationProfiling.mode`：`integer`，默认0，改变分析日志输出级别。 0，1，2,分别对应关闭，仅打开慢查询，记录所有操作
  > - **`storage`**：
  >   - `storage.dbPath`：`string `，指定数据文件的路径
  >   - `storage.directoryPerDB`：`boolean`，默认关闭。指定存储每个数据库文件到单独的数据目录。如果在一个已存在的系统使用该选项，需要事先把存在的数据文件移动到目录。
  >   - `storage.indexBuildRetry`：`boolean`，,默认为`true` 。指定数据库在索引建立过程中停止，重启后是否重新建立索引。
  >   - `storage.preallocDataFiles`：`boolean`，默认`true`， 是否预先分片好数据文件。
  >   - `storage.nsSize`：`integer`,默认16 指定命名空间的大小，即`.ns`后缀的文件。最大为2047MB,16M文件可以提供大约24000个命名空间。
  >   - `storage.quota.enforced`：`boolean`,默认`false` 限制每个数据库的数据文件数目。可以通过`maxFilesPerDB`调整数目。
  >   - `storage.quota.maxFilesPerDB`：`integer`,默认为8 限制每个数据库的数据文件数目。
  >   - `storage.smallFiles`：`boolean`,默认为`false` 限制`mongodb`数据文件大小为512MB，减小`journal`文件从1G到128M,适用于有很多数量小的数据文件。
  >   - `storage.syncPeriodSecs`：`number`,默认60。`mongodb`文件刷新频率，尽量不要在生产环境下修改。
  >   - `storage.repairPath`：`string`，默认为指定`dbpath`下的`_tmp`目录。 指定包含数据文件的根目录，进行`--repair`操作。
  >   - `storage.journal.enabled`：`boolean`,默认`64bit`为`true`，`32bit`为`false `记录操作日志，防止数据丢失。
  >   - `storage.journal.debugFlags`：`integer` 提供数据库在非正常关闭下的功能测试。
  >   - `storage.journal.commitIntervalMs`：`number`，默认为100或30，`journal`操作的最大间隔时间。可以是2-300`ms`之间的值，低的值有助于持久化，但是会增加磁盘的额外负担。 如果`journal`和数据文件在同一磁盘上，默认为100`ms`。如果在不同的磁盘上为30`ms`。 如果强制`mongod`提交日志文件，可以指定`journal`: `true`，指定后，时间变为原来的三分之一。
  > - **`replication`**：
  >   - `replication.oplogSizeMB`：`integer`,默认为磁盘的5% 指定`oplog`的最大尺寸。对于已经建立过`oplog.rs`的数据库，指定无效。
  >   - `replication.replSetName`：`string` 指定副本集的名称。
  >   - `replication.secondaryIndexPrefetch`：`string`，默认为all 指定副本集成员在接受`oplog`之前是否加载索引到内存。默认会加载所有的索引到内存。` none`，不加载；`all`，加载所有；`_id_only`，仅加载`_id`。
  > - **`sharding`**：
  >   - `sharding.clusterRole`：`string`，指定分片集的`mongodb`角色。 `configsvr`,配置服务器，端口27019;`shardsvr`,分片实例，端口27018。
  >   - `sharding.archiveMovedChunks`：`integer `在块移动过程中，该选项强制mongodb实例保存所有移动的文档到moveChunk目录。
  > - **`auditLog`**：
  >   - `auditLog.destination`：`string`, `syslog`,以`json`格式保存身份验证到`syslog`，`windows`下不可用，`serverity`级别为`info`，`facility`级别为`user`。`console`以`json`格式输出信息到标准输出。 `file`以`json`格式输出信息到文件。
  >   - `auditLog.forma`t：`string` 指定输出文件的格式` JSON`,输出`json`格式文件;`BSON`,输出`bson`二进制格式文件。
  >   - `auditLog.path`：如果`--auditDestination`的值为`file`，则该选项指定文件路径。
  >   - `auditLog.filter`：`document`, 指定过滤系统身份验证的格式为:
  > - **`snmp`**：
  >   - `snmp.subagent`：`boolean` 运行`SNMP`为一个子代理。
  >   - `snmp.master`：`boolean `运行`SNMP`为一个主进程。

- 在`admin`数据库中创建认证用户：`mongo`—>`use admin`—>`db.createUser({user: 'lajos', pwd: '123456', 'roles': [{role: 'root', 'db': 'admin'}]})`—>`show users`或`db.system.users.find()`

  - `role`：指定用户的角色，可选的角色有以下几种

    - 数据库用户角色：`read`、`readWrite`
    - 数据库管理角色：`dbAdmin`、`dbOwner`、`userAdmin`
    - 集群管理角色：`clusterAdmin`、`clusterManager`、`clusterMonitor`、`hostManager`
    - 备份恢复角色：`backup`、`restore`
    - 所有数据库角色：`readAnyDatabase`、`readWriteAnyDatabase`、`userAdminAnyDatabase`、`dbAdminAnyDatabase`
    - 超级用户角色：`root`
    - 内部角色：`__system`

    > (1)`Read`：允许用户读取指定数据库<br>
    > (2)`readWrite`：允许用户读写指定数据库<br>
    > (3)`dbAdmin`：允许用户在指定数据库中执行管理函数，如索引创建、删除，查看统计或访问`system.profile`<br>
    > (4)`userAdmin`：允许用户向`system.users`集合写入，可以找指定数据库里创建、删除和管理用户<br>
    > (5)`clusterAdmin`：只在`admin`数据库中可用，赋予用户所有分片和复制集相关函数的管理权限<br>
    > (6)`readAnyDatabase`：只在`admin`数据库中可用，赋予用户所有数据库的读权限<br>
    > (7)`readWriteAnyDatabase`：只在`admin`数据库中可用，赋予用户所有数据库的读写权限<br>
    > (8)`userAdminAnyDatabase`：只在`admin`数据库中可用，赋予用户所有数据库的`userAdmin`权限<br>
    > (9)`dbAdminAnyDatabase`：只在`admin`数据库中可用，赋予用户所有数据库的`dbAdmin`权限<br>
    > (10)`root`：只在`admin`数据库中可用。超级账号，超级权限

- 修改配置文件做认证：`vim /etc/mongod.conf`—>将`security`中的`authorization`改为`enabled`

- 重启`mongod`服务：`sudo service mongod restart`

- 认证：`mongo`—>`use admin`—>`db.auth('lajos', '123456')`

### 3.5. `Python`操作`MongoDB`

+ 安装扩展库：`pip install pymongo`

+ 创建连接及相关操作:

    ```python
    import pymogo
    # 创建连接
    connect = pymongo.MongoClient(host='localhost', port=27017)
    # 选择数据库, test是数据库名称
    # 如果mongodb设置账户和密码校验，先要进行校验
    admin = connect.admin
    admin.authentication(username='lajos', password='123456')

    db = connnect.test
    # 查看当前数据库的所有集合
    print(db.collection_names())
    # 查询文档, 得到的结果是一个生成器，需要遍历才可以获取结果
    ret = db.star.find()
    print([i for i in ret])
    # 增加文档，得到的返回值是对应的_id
    ret1 = db.star.insert({'name': 'lajos', age: 18})
    ret2 = db.star.insert_one(({'name': 'lajos', age: 18})
    ret3 = db.star.insert([
        {'name': 'wjk', age: 20},
        {'name': 'yyqx', age: 21},
    ])
    ret4 = db.star.insert_many([
        {'name': 'wjk', age: 20},
        {'name': 'yyqx', age: 21},
    ])
    # 根据_id查询，需要注意要将id值转化为bson类型
    from bson.objectid import ObjectId
    ret = db.star.find({'_id': ObjectId('5dd6518225d1748cf88d0d3b')})
    print([i for i in ret])
    ```
