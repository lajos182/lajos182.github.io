---
title: hexo+github快速制作个人博客
date: 2020-05-07 09:06:09
tags: [Hexo, 前端, Github]
toc: true
---

## 1. 为何采用`Github pages`服务

`Hexo`是一个简单、快速、强大的基于 `Github Pages `的博客发布工具，支持`Markdown`格式，有众多优秀插件和主题。

使用`github pages`服务搭建博客的好处有：

> （1）全是静态文件，访问速度快；
>
> （2）免费方便，不用花一分钱就可以搭建一个自由的个人博客，不需要服务器不需要后台；
>
> （3）可以随意绑定自己的域名，不仔细看的话根本看不出来你的网站是基于`github`的；
>
> （4）数据绝对安全，基于`github`的版本管理，想恢复到哪个历史版本都行；
>
> （5）博客内容可以轻松打包、转移、发布到其它平台；

<!--more-->

## 2. 准备工作

在开始一切之前，首先要准备以下工作：

+ `Github`账号，[`https://github.com/`](https://github.com/)
+ `node.js`、`npm`，[`http://nodejs.cn/download/`](http://nodejs.cn/download/)
+ `git for windows`(或者其他`git`客户端)，[`https://git-scm.com/downloads`](https://git-scm.com/downloads)

## 3. 搭建博客

### 3.1 创建仓库

（1）直接打开[`Github`官网](https://github.com/)，完成注册

（2） 创建仓库：新建一个名为`github用户名.github.io`的仓库，比如，`github`用户名为`lajos182`，那么就新建`lajos182.github.io`的仓库(其他名称无效)，以后部署上线就可以使用`http://lajos182.github.io`(如果使用`https`，就使用`https://lajos182.github.io`)访问

### 3.2 绑定域名

如果不绑定域名就用默认的 `xxx.github.io` 来访问，如果你想拥有一个属于自己的域名，那也是OK的；

首先注册一个域名，域名注册可以去`godaddy`，阿里云、腾讯云等；

绑定域名分2种情况：带`www`和不带`www`的；

域名配置最常见有2种方式，`CNAME`和`A`记录，`CNAME`填写域名，`A`记录填写`IP`，由于不带`www`方式只能采用`A`记录，所以必须先`ping`一下`github用户名.github.io`的`IP`，然后到你的域名`DNS`设置页，将`A`记录指向你`ping`出来的`IP`，将`CNAME`指向`github用户名.github.io`，这样可以保证无论是否添加`www`都可以访问，如下：

![域名解析](https://gitee.com/lajos/image_bucket/raw/master/skill/lajos_top.png)

然后到你的`github`项目根目录新建一个名为`CNAME`的文件（无后缀），里面填写解析后的域名。注意，在绑定了新域名之后，原来的`github用户名.github.io`并没有失效，而是会自动跳转到你的新域名。

### 3.3 配置`SSH key`

提交代码肯定要拥有`github`权限才可以，但是直接使用用户名和密码太不安全了，所以使用`ssh key`来解决本地和服务器的连接问题。

用`git bash`执行如下命令：

```bash
$ cd ~/. ssh #检查本机已存在的ssh密钥
```

如果提示：`No such file or directory`，就使用下面的命令生成密钥：

```bash
$ ssh-keygen -t rsa -C "邮件地址"
```

然后连续3次回车，最终会在用户目录下生成一个`.ssh`目录，找到`.ssh\id_rsa.pub`文件，记事本打开并复制里面的内容，打开`github`主页，进入个人设置 ->` SSH and GPG keys` -> `New SSH key`，配置好密钥即可。

### 3.4 `hexo`安装并初始化

（1）安装`hexo`：首先确保电脑已经安装`node.js`和`npm`，可以通过下面的命令进行验证是否已经安装：

```bash
$ node -v
$ npm -v
```

如果直接显示出对应的版本号表示已经安装，当然还要确保电脑已经配置好环境变量。然后输入命令安装`hexo`：

```bash
$ npm install -g hexo
```

（2）克隆仓库至本地：如果无法克隆空仓库，可以现在仓库新建一个`README.md`文件，里面可以暂时写一些介绍。

```bash
$ git clone git@github.com:lajos182/lajos182.github.io.git  # 后面是创建的仓库地址
```

（3）找到`lajos182.github.io`所在目录，创建一个`hexo`（可以随便取个名字）分支，这是方便以后多台电脑同步。

```bash
$ cd lajos182.github.io/
$ git branch hexo
```

（4）切换至`hexo`分支，初始化：

```bash
$ hexo init
```

`hexo`会自动下载一些文件至`lajos182.github.io`目录，包含`node_modules`，此时创建一个`.gitignore`文件，里面包含以下内容：

```bash
.DS_Store
Thumbs.db
db.json
*.log
node_modules/
public/
.deploy*/
```

此时的目录结构如下：

![目录结构](https://gitee.com/lajos/image_bucket/raw/master/skill/hexo_dir.png)

这时，使用下面的命令启动服务后，在浏览器访问 [`http://localhost:4000`](http://localhost:4000/) 即可看到内容，可能会碰到浏览器一直在转圈但是就是加载不出来的问题，一般情况下是因为端口4000被占用的缘故，解决端口冲突问题请参考这篇文章：[`http://blog.liuxianan.com/windows-port-bind.html`](http://blog.liuxianan.com/windows-port-bind.html)。

```bash
$ hexo g # 生成
$ hexo s # 启动服务
```

### 3.5 `hexo`修改主题

第一次初始化的时候`hexo`已经帮我们写了一篇名为 `Hello World` 的文章，默认的主题比较丑，所以可以进行修改。`hexo`常用的主题可以看知乎这个推荐[`https://www.zhihu.com/question/24422335/answers/updated`](https://www.zhihu.com/question/24422335/answers/updated)。

博主采用[`hexo-theme-yilia-plus`](https://github.com/JoeyBling/hexo-theme-yilia-plus)这个主题，这个主题是根据[`hexo-theme-yilia`](https://github.com/litten/hexo-theme-yilia)主题做了一些优化和改动，详情可以直接点击上面的链接查看。

预览效果：

电脑端：

![电脑端预览](https://gitee.com/lajos/image_bucket/raw/master/skill/yilla_plus.png)

移动端：

![移动端预览](https://gitee.com/lajos/image_bucket/raw/master/skill/yilia_plus_mobile.png)

（1）下载主题：

```bash
$ cd lajos182.github.io/
$ git clone https://github.com/JoeyBling/hexo-theme-yilia-plus themes/yilia-plus # 注意放到themes/yilia-plus目录下
```

下载后的主题都在目录`themes`下：

![themes预览](https://gitee.com/lajos/image_bucket/raw/master/skill/hexo_themes.png)

（2）修改全局配置`_config.yml`：就是根目录下的这个文件，下面配置中的`xxxxxx`可以根据自己来进行配置。

```bash
# Hexo Configuration
## Docs: https://hexo.io/docs/configuration.html
## Source: https://github.com/hexojs/hexo/

# Site
title: xxxxx
subtitle: xxxxxx
description: xxxxxx
keywords: xxxxxx
author: Lajos
language: zh-Hans
timezone: Asia/Shanghai

# URL
## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
url: xxxxxx  # 例如，https://lajos182.github.io/
root: /
permalink: :year/:month/:day/:title/
permalink_defaults:
pretty_urls:
  trailing_index: true # Set to false to remove trailing 'index.html' from permalinks
  trailing_html: true # Set to false to remove trailing '.html' from permalinks

# Directory
source_dir: source
public_dir: public
tag_dir: tags
archive_dir: archives
category_dir: categories
code_dir: downloads/code
i18n_dir: :lang
skip_render:
  - screenshots/**
  - anires/**
  - screenshots/**
  - gitment/**
  - baidu_sitepush/**
  - canvas_nest/**
  - '*.html'
  - '*.js'
  - README.md
  - '*.sh'
  - '*.txt'
  - '*.md'

# Writing
new_post_name: :title.md # File name of new posts
default_layout: post
titlecase: false # Transform title into titlecase
external_link: true
  # enable: true # Open external links in new tab
  # field: site # Apply to the whole site
  # exclude: ''
filename_case: 0
render_drafts: false
post_asset_folder: false
relative_link: false
future: true
highlight:
  enable: true
  line_number: true
  auto_detect: false
  tab_replace: ''
  # wrap: true
  # hljs: false

# Home page setting
# path: Root path for your blogs index page. (default = '')
# per_page: Posts displayed per page. (0 = disable pagination)
# order_by: Posts order. (Order by date descending by default)
index_generator:
  path: ''
  per_page: 10
  order_by: -date
  #top: 1
  #date: -1

# Category & Tag
default_category: uncategorized
category_map:
tag_map:

# Metadata elements
## https://developer.mozilla.org/en-US/docs/Web/HTML/Element/meta
meta_generator: true

# Date / Time format
## Hexo uses Moment.js to parse and display date
## You can customize the date format as defined in
## http://momentjs.com/docs/#/displaying/format/
date_format: YYYY-MM-DD
time_format: HH:mm:ss
## Use post's date for updated date unless set in front-matter
# use_date_for_updated: false

# Pagination
## Set per_page to 0 to disable pagination
per_page: 10
pagination_dir: page

# Include / Exclude file(s)
## include:/exclude: options only apply to the 'source/' folder
include:
exclude:
ignore:

# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: yilia-plus

# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repository: git@github.com:github用户名/github用户名.github.io.git # 这里是重点，例如git@github.com:lajos182/lajos182.github.io.git，起始就是仓库的地址
  branch: master  # 默认提交的分支是master，所以博主创建了hexo分支保存hexo配置及md博客文件

jsonContent:
  file: content.json
  meta: false
  pages: false
  posts:
    title: true
    date: true
    path: true
    text: true
    raw: false
    content: false
    slug: false
    updated: false
    comments: false
    link: true
    permalink: false
    excerpt: false
    categories: false
    tags: true

# Extensions
plugins:
  hexo-generator-feed

# Feed Atom
feed:
  type: atom
  path: atom.xml
  limit: 20

# https://github.com/JoeyBling/hexo-filter-image
filter_image:
  # 日志是否启用
  log: true
```

（3）修改主题下面的`_config.yml`：这个配置是`themes/yilia-plus/_config.yml`

```bash
# Header-菜单
menu:
  主页: /
  日记: /tags/随笔/
  Python: /tags/Python/


# subNav-子导航
subNav:
  github: "xxxxxx"
  gitee: "xxxxxx" # 码云
  # jianshu: "#" #简书
  # cnblog: "#"
  #blog: "#"
  csdn: "xxxxxx"
  #rss: "#"
  zhihu: "xxxxxx"
  #qq: "img/2434387555.jpg"
  #weixin: "img/weixin_.png"
  weibo: "xxxxxx"
  #douban: "#"
  #segmentfault: "#"
  #bilibili: "#"
  #acfun: "#"
  #mail: "mailto:liujinag_183@qq.com"
  #facebook: "#"
  #google: "#"
  #twitter: "#"
  #linkedin: "#"

# 悬停预览图片效果
hover_effect:
  ## `global` 0: Set separately, 1: Enable global 2: Close global
  ## `global` 0: 分开设置, 1: 全局启用, 2: 全局关闭
  global: 1
  # SubNav-导航
  subNav: true

# RSS订阅(关于如何配置启用:https://www.jianshu.com/p/2aaac7a19736)
rss: /atom.xml

# 是否需要修改 root 路径
# 如果您的网站存放在子目录中，例如 http://yoursite.com/blog，
# 请将您的 url 设为 http://yoursite.com/blog 并把 / 设为 /blog/。
# 新版本已弃用，请在博客根目录文件进行配置
# root: /

# Content

# 文章太长，截断按钮文字(在需要截断的行增加此标记：<!--more-->)
excerpt_link: more
# 文章卡片右下角常驻链接，不需要请设置为false
show_all_link: '展开全文'
# 数学公式
mathjax: false

# Open link in a new tab | 是否在新窗口打开链接
open_in_new: 
  article: false  # 文章链接
  menu: false   # 导航菜单
  subNav: false  # 子菜单

# 打赏
# 打赏type设定：0-关闭打赏； 1-文章对应的md文件里有reward:true属性，才有打赏； 2-所有文章均有打赏
reward_type: 2
# 打赏wording
reward_wording: '谢谢你请我吃糖果'
# 支付宝二维码图片地址，跟你设置头像的方式一样。比如：/assets/img/alipay.jpg
alipay: /img/alipay.jpg
# 微信二维码图片地址
weixin: /img/weixin.png

# 目录
# 目录设定：0-不显示目录； 1-文章对应的md文件里有toc:true属性，才有目录； 2-所有文章均显示目录
toc: 1
# 根据自己的习惯来设置，如果你的目录标题习惯有标号，置为true即可隐藏hexo重复的序号；否则置为false
toc_hide_index: true
# 目录为空时的提示
toc_empty_wording: '目录，不存在的…'

# 是否有快速回到顶部的按钮
top: true

# Miscellaneous
# 百度统计
baidu_analytics: ''
google_analytics: ''

# 网站图标
favicon: /img/favicon.ico

# 你的头像url
avatar: /img/me.png

# 是否开启分享
share_jia: true

# 评论：1、畅言；2、Disqus；3、Gitment；4、Giteement
# 不需要使用某项，直接设置值为false，或注释掉
# 具体请参考wiki：https://github.com/JoeyBling/hexo-theme-yilia-plus/wiki

# 1、畅言
changyan_appid: false
changyan_conf: false

# 2、Disqus 在hexo根目录的config里也有disqus_shortname字段，优先使用yilia-plus的
disqus: false

# 3、Gitment----基于GitHub的评论系统(关闭请设置gitment_owner为false)
# 关于如何集成:https://www.jianshu.com/p/ac7658cc912f
gitment_owner: lajos182      #你的 GitHub ID
# 是否使用官方js(false可以提升访问速度，本地修改过一部分的js，官方js可能会出现服务器不稳定，不太建议使用)
gitment_remote: false
gitment_repo: 'xxxxxx'  #存储评论的 repo name(需要在Github创建),例如'lajos182.github.io'
gitment_oauth:
  client_id: 'xxxxxx'           #client ID
  client_secret: 'xxxxxx'       #client secret

# 4、Giteement----【国内用户建议使用这个，相对比较快】
# 关于如何集成:https://www.jianshu.com/p/f5c4633524c7
# 基于码云的评论系统(https://gitee.com/zhousiwei/giteement)
giteement:
  enable: false  # 是否启用码云评论系统
  # 是否使用官方js(false可以提升访问速度)
  remote: false
  redirect_uri: ''   # 应用回调地址(请和配置的第三方应用保持一致)
  # 不能更改(网上开源项目`https://github.com/Rob--W/cors-anywhere`作者提供的专门用来跨域服务器的配置)
  oauth_uri: https://cors-anywhere.herokuapp.com/https://gitee.com/oauth/token
  giteeID: ''  # 你的码云账号英文名
  # 存储评论的 repo name(需要在码云仓库创建公开仓库)
  repo: ''
  gitment_oauth:
    client_id: ''           #client ID
    client_secret: ''       #client secret

# 访问量统计功能(不蒜子)
busuanzi:
  enable: true
  site_visit: true  # 站点访问量显示
  article_visit: true  # 文章访问量显示

# 网易云音乐插件
music:
  enable: true
  # 播放器尺寸类型(1：长尺寸、2：短尺寸)
  type: 2
  id: 570218574  # 网易云分享的音乐ID(更换音乐请更改此配置项)
  autoPlay: false  # 是否开启自动播放
  # 提示文本(关闭请设置为false)
  text: '这是博主分享的音乐，请尽情的欣赏它吧！'

# 页面点击小红心
clickLove:
  # (关闭请设置为false)
  enable: true

# GitHub Ribbons(https://github.blog/2008-12-19-github-ribbons/)
github:
  # (关闭请设置为false)
  url: xxxxxx

# 页脚 Litten(此配置项已弃用)
# 帮助我们让更多人可以更方便使用Hexo，请尽量不要修改此主题配置
pageFooter:
  litten: GitHub:<a href="https://github.com/JoeyBling/hexo-theme-yilia-plus" target="_blank">hexo-theme-yilia-plus</a>

# 开启百度站长平台自动推送(https://ziyuan.baidu.com/linksubmit/index)
baidu_push: false

# 版权声明
# 版权声明type设定：0-关闭版权声明； 1-文章对应的md文件里有copyright: true属性，才有版权声明； 2-所有文章均有版权声明
copyright_type: 2
# 版权声明自定义文本(关闭请设置为false)
copyright_text: 

# 网站成立年份(默认为 2018，若填入年份小于当前年份，则显示为 2018-2019 类似的格式)
since: 2020

# Progress Bar | 页面加载进度条
# Demo: http://github.hubspot.com/pace/docs/welcome/
# type: barber-shop|big-counter|bounce|center-atom|center-circle|
#       center-radar|center-simple|corner-indicator|flash|flat-top|
#       loading-bar|mac-osx|minimal
# color: black|blue|green|orange|pink|purple|red|silver|white|yellow|
progressBar:
  enable: true
  type: 'corner-indicator'  # Keep Quotes | 保留引号避免出错(某些type会导致样式重叠排版错误)
  color: blue

# Apple Touch icon 苹果图标(关闭请设置为false)
apple_touch_icon: '/apple-touch-icon-180x180.png'

# Tab Title Change | 标签页标题切换
tab_title_change:
  enable: true
  left_tab_title: '(つェ⊂) 我藏好了哦~ '
  return_tab_title: '(*´∇｀*) 被你发现啦~ '

# https://github.com/willin/hexo-wordcount
# 是否开启字数统计(关闭请设置enable为false)
# 也可以单独在md文件里Front-matter设置`no_word_count: true`属性，来自定义关闭字数统计
word_count:
  enable: true
  # 只在文章详情显示(不在首页显示)
  only_article_visit: true

# 文字输入特效
# https://github.com/disjukr/activate-power-mode
activate_power_mode:
  enable: true
  # 使输入模式丰富多彩
  colorful: true
  # 是否开启摇动
  shake: false

# 飘雪特效
# https://github.com/MlgmXyysd/snow.js
snow: true

# 看板娘动态模型插件
## https://github.com/JoeyBling/live2d-widget.js
live2d:
  # (关闭请设置为false)
  enable: true
  # 模型名称(取值请参考：https://github.com/JoeyBling/hexo-theme-yilia-plus/wiki/live2d%E6%A8%A1%E5%9E%8B%E5%8C%85%E5%B1%95%E7%A4%BA)
  model: hibiki
  display:
    position: right # 显示位置：left/right(default: 'right')
    width: 145  # 模型的长度(default: 150)
    height: 315 # 模型的高度(default: 300)
    hOffset: 50 # 水平偏移(default: 0)
    #vOffset: -20 # 垂直偏移(default: -20)
  mobile:
    show: true # 是否在移动设备上显示(default: true)
    scale: 0.6 # 移动设备上的缩放(default: 0.5)
  react:
    opacity: 1 # 模型透明度(default: 0.7)

# 样式定制 - 一般不需要修改，除非有很强的定制欲望…
style:
  # 头像上面的背景颜色
  # header: '#D3D1DC'
  header: '#4d4d4d'
  gif:
    # 是否启用左侧边栏动态图效果
    enable: true
    # 自定义背景图路径(默认可以不设置，提供默认背景图)
    path: /img/biubiubiu.gif
  # 右滑板块背景
  slider: 'linear-gradient(200deg,#a0cfe4,#e8c37e)'

# slider的设置
slider:
  # 是否默认展开tags板块
  showTags: true

# 智能菜单
# 如不需要，将该对应项置为false
# 比如
#smart_menu:
#  friends: false
smart_menu:
  innerArchive: '所有文章'
  friends: '友链'
  aboutme: '关于我'

# 友情链接
friends:
  技术笔记:  #网站名称
    #网站地址
    url: xxxxxx
    #网站图片(可忽略不写)
    # img: https://zhousiwei.gitee.io/ibooks/favicon.ico
    #网站简介(可忽略不写)
    # description: 记录工作和学习过程中的笔记：Java、前端开发、Hexo博客、聚合支付、Linux笔记、ElasticSearch、ELK日志分析
  GitHub:
    url: xxxxxx
  码云:
    url: https://gitee.com/lajos
  # 简书:
  #   url: https://www.jianshu.com/u/02cbf31a043a
  # CSDN:
  #   url: https://blog.csdn.net/sinat_41898105

# 关于我
aboutme: 主要涉及技术：<br>xxxxxx<br>xxxxxx<br><br>很惭愧<br><br>只做了一点微小的工作<br>谢谢大家
```

（4）添加`CNAME`文件：文件内容添加绑定的域名

（5）将上面显示的目录结构中的文件、`CNAME`、`READE.md`提交至远程`hexo`分支：

```bash
$ git add .
$ git commit -m '提交至hexo分支'
$ git push origin hexo
```

（6）`hexo`常见命令

```bash
hexo new "postName" #新建文章
hexo new page "pageName" #新建页面
hexo generate #生成静态页面至public目录
hexo server #开启预览访问端口（默认端口4000，'ctrl + c'关闭server）
hexo deploy #部署到GitHub
hexo help  # 查看帮助
hexo version  #查看Hexo的版本
```

缩写：

```bash
hexo n == hexo new
hexo g == hexo generate
hexo s == hexo server
hexo d == hexo deploy
```

组合命令：

```bash
hexo s -g #生成并本地预览
hexo d -g #生成并上传
```

（7）发布博客并上传

```bash
hexo n "发布第一个博客"
```

进入`lajos182.github.io/source/_posts/`，找到`发布第一个博客.md`这个文件，可采用`Markdown`编辑器进行编写，编写完成后，使用命令生成并上传

（8）提交`hexo`分支的修改内容：

每次发布完，记得将修改的部分提交至`hexo`。

### 3.6 `git`管理与多端同步

为什么会存在这个问题呢？因为刚开始在一台电脑上搭建`hexo`环境并发布博客后，又换用另外一台新设备继续发布博客，此时发现从`github`克隆下来的只有网页样式文件，而`source`目录中的`md`文件就没有，如果直接发布新博文，就会把原来的博文覆盖掉。

为解决这个问题，上面的教程已经实现了：通过`git`建立新建一个`hexo`分支，`hexo`生成的静态博客文件默认放在`master`分支上；`hexo`的源文件（部署环境文件）可以都放在`hexo`分支上，换用新设备时，直接`git clone -b hexo git@github.com:lajos182/lajos182.github.io.git`，然后使用`npm install `安装环境，这样就能看到之前发布的博文`*.md`文件，发布新博文还是一样的。

### 3.7 常见的问题

（1）域名如何使用`https`访问？

进入`github`官网，打开仓库所在目录，点击`settings`，在`Gighub Pages`服务中配置`Enforce HTTPS`，此时绑定的域名就可以使用`https`访问

（2）如何添加`gitment`评论系统？

进入`github`主页，打开`Settings`—>`Developer settings`—>`OAuth Apps`—>`New OAuth App`，进入下面的页面：详细可以参考[`https://www.jianshu.com/p/f5c4633524c7`](https://www.jianshu.com/p/f5c4633524c7)

![themes预览](https://gitee.com/lajos/image_bucket/raw/master/skill/gitment_config.png)

（3）`hexo`的`gitment`评论跳转问题，登录`github`后出现跳转地址`https://www.lajos.top/?error=redirect_uri_mismatch&error_description=The redirect_uri MUST match the registered callback URL for this application.&error_uri=https://developer.github.com/apps/managing-oauth-apps/troubleshooting-authorization-request-errors/#redirect-uri-mismatch`，而且还无法评论

这个问题一般是注册的`gitment`应用有问题，自己查看主页以及授权回调地址是否正确，`http`和`https`也要查看是否与博客的域名地址一致。

（4）上一页和下一页显示问题：使用这个主题可能会出现下面这个问题：

![上一页下一页显示问题](https://gitee.com/lajos/image_bucket/raw/master/skill/prev_next_bug.png)

首先找到`lajos182.github.io/themes/yilia-plus/layout/_partial/archive.ejs`，修改两处，分别在8，9行与37，38行，将`&laquo; Prev`改为`<< 上一页`, `Next &raque;`改为`下一页 >>`，操作如下：

![修改上一页和下一页](https://gitee.com/lajos/image_bucket/raw/master/skill/archive_pre_next.png)

![修改上一页和下一页](https://gitee.com/lajos/image_bucket/raw/master/skill/archive_pre_next1.png)

接着，找到`lajos182.github.io/themes/yilia-plus/layout/_partial/script.ejs`，`crtl + f`搜索`&laquo; Prev`，将改为`<< 上一页`, 搜索`Next &raque;`改为`下一页 >>`，操作如下：

![修改上一页和下一页](https://gitee.com/lajos/image_bucket/raw/master/skill/scipt_pre_next.png)

保存完成，输入一下命令：

```bash
$ hexo clean
$ hexo g
$ hexo d
```

至此完成修改

## 4 本文参考

[`http://blog.haoji.me/build-blog-website-by-hexo-github.html?from=xa`](http://blog.haoji.me/build-blog-website-by-hexo-github.html?from=xa)

[`https://blog.csdn.net/whbk101/article/details/97300209`](https://blog.csdn.net/whbk101/article/details/97300209)

[``https://www.jianshu.com/p/f5c4633524c7``](https://www.jianshu.com/p/f5c4633524c7)