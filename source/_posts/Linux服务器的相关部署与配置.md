---
title: Linux服务器的相关部署与配置
date: 2020-08-13 14:05:50
tags: [Linux, Python, Django]
toc: True
---

## 1. 用户的创建与`Python`环境安装

### 1.1. 创建用户并授予权限
+ 新的虚拟机或服务器，首先添加用户：`useradd -m lajos`(位于/home/下)

+ 为用户添加密码：`passwd lajos`

+ 新建的用户不能使用`sudo`，为创建的普通用户添加sudo权限：
    ```shell
    usermod -a -G adm lajos
    usermod -a -G sudo lajos
    ```

<!--more-->

+ 修改用户权限：在`root`用户下进入`/etc/sudoers`添加并使用`wq!`保存：
  
    ```shell
    lajos   ALL=(ALL:ALL) ALL
    ```
    
+ 修改`~/.vimrc`配置，常用的配置如下：
    ```shell
    syntax on
    set nu
    set autoindent
    set smartindent
    set tabstop=4
    set shiftwidth=4
    set showmatch
    set ruler
    set cindent
    set background=dark
    set mouse=a
    set mouse=h
    ```
### 1.2. 安装`Python`相关环境
+ `Linux 16.04`中默认安装的是`Python2.7`和`Python3.5`，`Linux 18.04`中默认安装的是`Python3.6`, `Linux 20.04`默认安装的是`Pyhton3.8`
+ 如果选择的系统中要安装大于`Python3.5`的版本，可以选用下面的方法安装：
    ```shell
    sudo apt-get update
    sudo apt-get install software-properties-common
    sudo add-apt-repository ppa:jonathonf/python-3.6
    sudo apt-get update
    sudo apt-get install python3.6
    cd /usr/bin
    ls | grep python
    sudo rm python
    sudo ln -s python3.6 python
    sudo apt-get install python3-pip
    cd /usr/bin
    ls | grep pip
    sudo ln -s pip3 pip
    pip --version
    sudo python pip install --upgrade pip
    ```
+ 如果是`Linux 18.04`版本以上可以直接创建软连接，`pip`的安装可使用`sudo apt-get install python3-pip`，此时可以创建一个`pip`的软连接
+ 安装虚拟环境
    ```shell
    sudo pip install virtualenv
    sudo pip install virtualenvwrapper
    mkdir  ~/.virtualenvs
    sudo vim  ~/.bashrc 
    # 在结尾添加下面两行的内容
    export WORKON_HOME=$HOME/.virtualenvs
    source /usr/local/bin/virtualenvwrapper.sh
    # 执行.bashrc文件
    source ~/.bashrc
    ```

## 2. `Nginx`的安装

+ 安装`zlib`依赖库：`sudo apt-get install zlib1g-dev`

+ 进入解压相关文件：

  ![img](https://gitee.com/lajos/image_bucket/raw/master/skill/nginx_yilai.png)

  ```shell
  tar -xzvf openssl-1.0.1.tar.gz
  # 官网下载地址：http://nginx.org/en/download.html
  tar -xzvf nginx-1.11.3.tar.gz
  tar -xzvf pcre-8.41. tar.gz  
  # 进入Nginx解压目录
  cd /home/lajos/ nginx-1.11.3/
  ```

+ 配置环境：

  ```shell
  ./configure  --prefix=/usr/local/nginx  --with-http_ssl_module  --with-http_flv_module  --with-http_stub_status_module   --with-http_gzip_static_module --with-pcre=../pcre-8.41  --with-openssl=../openssl-1.0.1
  ```

+ 编译：`make`(如果出现`”pcre.h No such file or directory”`，安装`”sudo apt-get install libpcre3-dev”`)

+ 安装：`sudo make install`

+ 说明：`nginx`会被安装在`/usr/local/nginx`目录下

  `conf`：存放配置文件   `html`：静态网页  

  `logs`：存放日志文件   `sbin`：存放可执行文件

+ 相关命令

  - 启动`Nginx`服务：` sudo /usr/local/nginx/sbin/nginx`

  - 关闭`Nginx`服务： `sudo /usr/local/nginx/sbin/nginx -s stop`

  - 重新加载配置：`sudo /usr/local/nginx/sbin/nginx -s reload`

  - 指定配置文件：`sudo /usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf`

  - 查看版本信息：

    ```shell
    sudo /usr/local/nginx/sbin/nginx -v
    sudo /usr/local/nginx/sbin/nginx -V
    ```

  - 查看80端口的程序：`nesta –ano | grep 80`

  - 关闭占用80端口的程序：`sudo fuser -k 80/tcp`

+ 测试：打开浏览器，输入`Nginx`服务器`IP`地址，如果显示`Welcome to Nginx`，表示已经安装成功

+ 配置：

  - 打开配置文件：`/usr/local/nginx/conf/nginx.conf`

    ![img](https://gitee.com/lajos/image_bucket/raw/master/skill/nginx_conf.png)

  - 全局设置：定义全局错误日志文件，需要什么等级可以设置开启

    ```shell
    error_log logs/error.log;
    #error_log logs/error.log notice;
    #error_log logs/error.log info;
    worker_rlimit_nofile：
    # 指定一个nginx可以打开的最多文件描述符，可以使用“ulimit –n 65535”进行设置（虚拟机默认设置1024），阿里云服务器默认就是65535
    ```

  + `events`(`nginx`工作模式)

    ```shell
    events {
        use epoll;  # linux标准的工作模式，nginx高效的基石
        worker_connections 1024;  #定义nginx每个进程的最大连接数
    }
    ```

  + `http`设置

    ```shell
    sendfile   on;   # 开启高效文件传输模式
    tcp_nopush  on;   # 防止网络阻塞
    tcp_nodelay  on;
    keepalive_timeout  65;    # 设置客户端连接活动的超时时间
    gzip on;   #使用压缩模块
    ```

  + `server`（主机设置）

    ```shell
    server{
        listen          80;
        server_name    localhsot www.lajos.top 39.105.61.52;
        charser utf-8;
        # 负载均衡模块，upstream是负载均衡器
        upstream lajos {
            server 39.105.61.52:8000 weight=1 max_fails=1 fail_timeout=300s;
            server 39.105.61.53:8000 weight=1 max_fails=1 fail_timeout=300s;
        }
    
        # 反向代理配置， 
        location / {
            #适用于django自带的runserver方式启动
            #proxy_pass http://www.lajos.top:8000;
            #proxy_pass http://www.lajos.top:8000;
            #proxy_set_header Host $http_host;
            # 设置uwsgi启动
            include uwsgi_params;
            uwsgi_pass lajos;
        }
    }
    
    ```

+ 重启`nginx`服务：`sudo /usr/local/nginx/sbin/nginx`

## 3. 部署`Django`项目

+ 创建虚拟环境并安装项目运行环境：

  ```shell
  mkvirtualenv testenv
  cd ~/test
  pip install -r requirements.txt
  ```

+ 修改`Django`项目的相关配置：

  ```python
  DEBUG = False
  ALLOWED_HOST = ['*']
  # ...............................................
  # 缓存设置
  CACHES = {
      'default': {
          'BACKEND': 'django_redis.cache.RedisCache',
          'LOCATION': 'redis://127.0.0.1:6379/0',
          'OPTIONS': {
              'CLIENT_CLASS': 'django_redis.client.DefaultClient',
          },
      },
      'verify_sms_codes': {
          'BACKEND': 'django_redis.cache.RedisCache',
          'LOCATION': 'redis://127.0.0.1:6379/1',
          'OPTIONS': {
              'CLIENT_CLASS': 'django_redis.client.DefaultClient',
          }
      }
  }
  
  # SESSION设置
  SESSION_ENGINE = 'django.contrib.sessions.backends.cache'
  SESSION_CACHE_ALIAS = 'default'
  # ...............................................
  # 配置静态文件
  STATIC_URL = '/static/'
  STATICFILES_DIRS = [
      os.path.join(BASE_DIR, 'static')
  ]
  STATIC_ROOT = '/var/www/test/static/'
  # ..............................................
  # 跨域配置
  # CORS
  CORS_ORIGIN_ALLOW_ALL = True  # 允许所有域名跨域(优先选择)
  # CORS_ORIGIN_WHITELIST = (
  #     '127.0.0.1:8080',
  #     'localhost:8080',
  # )
  CORS_ALLOW_CREDENTIALS = True  # 允许携带cookie
  CORS_ALLOW_HEADERS = (
      'XMLHttpRequest',
      'X_FILENAME',
      'accept',
      'accept-encoding',
      'authorization',
      'content-type',
      'dnt',
      'origin',
      'user-agent',
      'x-csrftoken',
      'x-requested-with',
      'Cache-Control'
  )
  # ...............................................
  # 日志配置
  LOGGING = {
      'version': 1,
      'disable_existing_loggers': False,  # 是否禁用已经存在的日志器
      'formatters': {  # 日志信息显示的格式
          'verbose': {
              'format': '%(levelname)s %(asctime)s %(module)s %(lineno)d %(message)s'
          },
          'simple': {
              'format': '%(levelname)s %(module)s %(lineno)d %(message)s'
          },
      },
      'filters': {  # 对日志进行过滤
          'require_debug_true': {  # django在debug模式下才输出日志
              '()': 'django.utils.log.RequireDebugTrue',
          },
      },
      'handlers': {  # 日志处理方法
          'console': {  # 向终端中输出日志
              'level': 'WARNING',
              'filters': ['require_debug_true'],
              'class': 'logging.StreamHandler',
              'formatter': 'simple'
          },
          'file': {  # 向文件中输出日志
              'level': 'INFO',
              'class': 'logging.handlers.RotatingFileHandler',
              'filename': os.path.join(os.path.dirname(BASE_DIR), 'logs/project.log'),  # 日志文件的位置
              'maxBytes': 300 * 1024 * 1024,
              'backupCount': 10,
              'formatter': 'verbose'
          },
          'file_DB': {  # 向文件中输出日志
              'level': 'INFO',
              'class': 'logging.handlers.RotatingFileHandler',
              'filename': os.path.join(os.path.dirname(BASE_DIR), 'logs/DB.log'),  # 日志文件的位置
              'maxBytes': 300 * 1024 * 1024,
              'backupCount': 10,
              'formatter': 'verbose'
          }
      },
      'loggers': {  # 日志器
          'web': {
              'handlers': ['console', 'file'],  # 可以同时向终端与文件中输出日志
              'propagate': True,  # 是否继续传递日志信息
              'level': 'DEBUG',  # 日志器接收的最低日志级别
          },
          'db': {
              'handlers': ['file_DB'],  # 可以向文件中输出日志
              'propagate': True,  # 是否继续传递日志信息
              'level': 'INFO',  # 日志器接收的最低日志级别
          }
      }
  }
  # ...............................................
  # REST配置
  REST_FRAMEWORK = {
      # 配置异常处理器
      'EXCEPTION_HANDLER': 'device_manage.utils.exceptions.exception_handler',
      # 认证
      'DEFAULT_AUTHENTICATION_CLASSES': (
          'rest_framework_jwt.authentication.JSONWebTokenAuthentication',
          'rest_framework.authentication.SessionAuthentication',
          'rest_framework.authentication.BasicAuthentication',
      ),
      'DEFAULT_FILTER_BACKENDS': (
          'django_filters.rest_framework.DjangoFilterBackend', # 过滤功能
          'rest_framework.filters.SearchFilter', # 搜索功能
          'rest_framework.filters.OrderingFilter', # 排序功能
      ),
      'DEFAULT_THROTTLE_CLASSES': (
          # 'rest_framework.throttling.AnonRateThrottle',  # 开启匿名用户接口请求频率限制
          'rest_framework.throttling.UserRateThrottle',  # 开启授权用户接口请求频率限制
      ),
      'DEFAULT_THROTTLE_RATES': {
          # 'anon': '2/second',  # 匿名用户请求频率
          'user': '5/second',  # 授权用户请求频率
      }
  }
  
  # Internationalization
  # https://docs.djangoproject.com/en/3.0/topics/i18n/
  
  LANGUAGE_CODE = 'zh-hans'
  
  TIME_ZONE = 'Asia/Shanghai'
  
  USE_I18N = True
  
  USE_L10N = True
  
  USE_TZ = False
  ```

+ 安装`uwsgi.ini`：

  ```shell
  sudo apt-get install libpython3.6-dev # (2.7不用安装)
  pip install uwsgi
  ```

+ 配置`uwsgi.ini`：

  ```shell
  [uwsgi]
  #注意多个项目的时候不能true哦，否则多个项目都共用这个配置参数了
  vhost = false
  
  #将要让nginx采用8001端口与uWSGI通讯，请确保此端口没有被其它程序采用。
  socket = 127.0.0.1:8001
  #http = 0.0.0.0:8000
  
  master = True
  
  #django项目的路径
  chdir = /home/lajos/test
  
  #进程数
  processess = 2
  
  threads = 4
  
  #这是项目wsgi.py文件的路径
  wsgi-file = test/wsgi.py
  
  #虚拟环境的路径
  virtualenv = /home/test/.virtualenvs/testenv
  
  pidfile = uwsgi.pid
  # 日志按天切割，且可以设置保留时长
  touch-logreopen = scripts/cut_uwsgi_logs.sh
  
  daemonize = uwsgi.log
  ```

+ 添加日志切分脚本：`cut_uwsgi_logs.sh`

  ```shell
  #!/bin/bash
  EXPIRE=7 # 设置日志保留7天
  DIR=`echo $(cd "$(dirname "$0")"; pwd)`/..
  sourcelogfile="${DIR}/uwsgi.log"
  touchfile="$0"
  DATE=`date -d "yesterday" +"%Y-%m-%d"`
  destlogpath="${DIR}/logs/uwsgi_${DATE}.log"
  echo $destlogpath
  mv $sourcelogfile $destlogpath
  touch $touchfile
  find ${DIR} -mtime +${EXPIRE} -name "uwsgi*.log" -exec rm -rf {} \;
  ```

+ 配置`nginx`：

  ```shell
  ...
  http {
  	...
  	upstream test {
                  server 127.0.0.1:8001;
      }
      ...
      server {
          listen       9000;
          server_name  localhost 47.111.97.146;
  
      	charset utf-8;
  
  		#access_log  logs/host.access.log  main;
  
          location / {
              include uwsgi_params;
              uwsgi_pass test;
          }
  
          location /static {
          	alias /var/www/test/static/;
          }
  
          #error_page  404             /404.html;
          #
          error_page  500 502 503 504  /50x.html;
          location = /50x.html {
          root   html;
          }
      }
      ...
  }
  ...
  ```

+ 创建静态文件的存储文件

  ```shell
  sudo mkdir -vp /var/www/test/static/
  sudo chmod 777 /var/www/test/static/
  ```

+ 在项目工程根目录下生成迁移数据库文件、静态文件并执行迁移

  ```shell
  python manage.py makemigrations
  python manage.py migrate
  python manage.py collectstatic
  ```

## 4. 常见的问题

+  `pip install MysqlClient`无法执行？

  - 在安装`mysqlclient`可能会出现下面的问题：

    ```shell
    WARNING: Running pip install with root privileges is generally not a good idea. Try pip3 install --user instead.
    Collecting mysqlclient
    Using cached https://files.pythonhosted.org/packages/4d/38/c5f8bac9c50f3042c8f05615f84206f77f03db79781db841898fde1bb284/mysqlclient-1.4.4.tar.gz
    Installing collected packages: mysqlclient
    Running setup.py install for mysqlclient … error
    Complete output from command /usr/bin/python3 -u -c “import setuptools, tokenize;file='/tmp/pip-build-m2dhr53z/mysqlclient/setup.py';f=getattr(tokenize, 'open', open)(file);code=f.read().replace('\r\n', '\n');f.close();exec(compile(code, file, 'exec'))” install --record /tmp/pip-nspnr_as-record/install-record.txt --single-version-externally-managed --compile:
    running install
    running build
    running build_py
    creating build
    creating build/lib.linux-x86_64-3.6
    creating build/lib.linux-x86_64-3.6/MySQLdb
    copying MySQLdb/init.py -> build/lib.linux-x86_64-3.6/MySQLdb
    copying MySQLdb/exceptions.py -> build/lib.linux-x86_64-3.6/MySQLdb
    copying MySQLdb/compat.py -> build/lib.linux-x86_64-3.6/MySQLdb
    copying MySQLdb/connections.py -> build/lib.linux-x86_64-3.6/MySQLdb
    copying MySQLdb/converters.py -> build/lib.linux-x86_64-3.6/MySQLdb
    copying MySQLdb/cursors.py -> build/lib.linux-x86_64-3.6/MySQLdb
    copying MySQLdb/release.py -> build/lib.linux-x86_64-3.6/MySQLdb
    copying MySQLdb/times.py -> build/lib.linux-x86_64-3.6/MySQLdb
    creating build/lib.linux-x86_64-3.6/MySQLdb/constants
    copying MySQLdb/constants/init.py -> build/lib.linux-x86_64-3.6/MySQLdb/constants
    copying MySQLdb/constants/CLIENT.py -> build/lib.linux-x86_64-3.6/MySQLdb/constants
    copying MySQLdb/constants/CR.py -> build/lib.linux-x86_64-3.6/MySQLdb/constants
    copying MySQLdb/constants/ER.py -> build/lib.linux-x86_64-3.6/MySQLdb/constants
    copying MySQLdb/constants/FIELD_TYPE.py -> build/lib.linux-x86_64-3.6/MySQLdb/constants
    copying MySQLdb/constants/FLAG.py -> build/lib.linux-x86_64-3.6/MySQLdb/constants
    running build_ext
    building 'MySQLdb.mysql' extension
    creating build/temp.linux-x86_64-3.6
    creating build/temp.linux-x86_64-3.6/MySQLdb
    gcc -pthread -Wno-unused-result -Wsign-compare -DNDEBUG -O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector-strong --param=ssp-buffer-size=4 -grecord-gcc-switches -m64 -mtune=generic -D_GNU_SOURCE -fPIC -fwrapv -fPIC -Dversion_info=(1,4,4,'final',0) -D__version=1.4.4 -I/usr/include/mysql -I/usr/include/mysql/mysql -I/usr/include/python3.6m -c MySQLdb/_mysql.c -o build/temp.linux-x86_64-3.6/MySQLdb/_mysql.o
    gcc -pthread -shared -Wl,-z,relro -g build/temp.linux-x86_64-3.6/MySQLdb/_mysql.o -L/usr/lib64/ -L/usr/lib64 -lmariadb -lpython3.6m -o build/lib.linux-x86_64-3.6/MySQLdb/_mysql.cpython-36m-x86_64-linux-gnu.so
    /usr/bin/ld: cannot find -lmariadb
    collect2: error: ld returned 1 exit status
    error: command 'gcc' failed with exit status 1
    ```

  - 解决方法：

    ```shell
    sudo apt-get install python3.6-dev
    sudo apt-get install libmysqlclient-dev
    ```

+ `Nginx`安装依赖`openssl-1.0.1.tar.gz`，`pcre-8.41. tar.gz`的下载地址在哪里？
  
- https://download.csdn.net/download/sinat_41898105/10482731
  
+ `Nginx`做日志切割如何实现？

  - `cut_nginx_log.sh`脚本：

    ```shell
    #!/bin/bash
    YESTERDAY=$(date -d "yesterday" +"%Y-%m-%d")
    EXPIRE=7
    LOGPATH=/usr/local/nginx/logs/
    PID=${LOGPATH}nginx.pid
    mv ${LOGPATH}access.log ${LOGPATH}access_${YESTERDAY}.log
    mv ${LOGPATH}error.log ${LOGPATH}error_${YESTERDAY}.log
    find ${LOGPATH} -mtime +${EXPIRE} -name "*.log" -exec rm -rf {} \;
    kill -USR1 `cat ${PID}`
    ```

+ `uwsgi`日志切割和`Nginx`日志切割都未生效？

  + 这是因为这两个的脚本都是利用`Linux crontab`定制任务实现，切换至`root`用户下，输入`crontab -l`

    ```shell
    # m h  dom mon dow   command
    0 0 * * * /bin/bash /usr/local/nginx/logs/cut_nginx_logs.sh
    0 0 * * * /bin/bash /home/lajos/test/scripts/cut_uwsgi_logs.sh
    ```

  + 注意一定要在超级用户`root`，否则可能不会生效，使用命令`crontab -e`可以进入编辑模式：

    ```
    # m h  dom mon dow   command
    0 0 * * * /bin/bash /usr/local/nginx/logs/cut_nginx_logs.sh
    0 0 * * * /bin/bash /home/lajos/test/scripts/cut_uwsgi_logs.sh
    ```

  + 编辑完成后，需要重启`cron`服务：`service cron restart`

