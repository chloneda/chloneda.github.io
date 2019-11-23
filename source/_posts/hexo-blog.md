---
title: Github+Hexo一站式部署个人博客
tags: Hexo
categories: Hexo
keywords: Hexo Github
comments: false
---
# 写在前面
**注：本文出自博主 Chloneda**：[个人博客](https://chloneda.github.io/) | [博客园](https://www.cnblogs.com/chloneda) | [Github](https://github.com/chloneda) | [Gitee](https://gitee.com/chloneda) | [知乎](https://www.zhihu.com/people/chl_vip/)

本文档是[Github](https://github.com/) + [Hexo](https://hexo.io/zh-cn/) 的搭建个人博客教程，其中Hexo基于Hexo v3.8.0版本，themes主题基于为NexT v7.0.0版本。

搭建博客前置条件可参考 [如何搭建个人独立博客？](https://www.zhihu.com/question/20463581/answer/25478916)

**个人博客地址： [Chloneda's blog](https://chloneda.github.io/)**

**注： 点开侧栏浏览目录可快速定位内容**

# 安装主题
在 Hexo 项目源码目录下，有两个重要的配置文件，其名称均为 _config.yml 。 其中，一份位于站点根目录下，主要包含 Hexo本身的配置；另一份位于主题目录下，主要用于主题相关的配置。为了描述方便，在以下说明中，将前者称为**站点配置文件**， 后者称为**主题配置文件**。

## 下载NexT主题
```
cd hexo
git clone https://github.com/theme-next/hexo-theme-next themes/next
```

## 启用NexT主题
修改**站点配置文件**，查找关键词theme，并修改为主题名字next：
```
# Extensions	#(注意冒号间的空格)
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: next
```
在切换主题之后、验证之前， 我们最好使用 hexo clean 来清除 Hexo 的缓存。

# 主题设置
## 设置Scheme
在Hexo主题中，有四种不同的模式！进入**主题配置文件**，搜索关键词找到scheme属性，选择自己喜欢的模式：
```
# ---------------------------------------------------------------
# Scheme Settings
# ---------------------------------------------------------------
# Schemes
# scheme: Muse    # 默认 Scheme，这是 NexT 最初的版本，黑白主调，大量留白
scheme: Mist     # Muse 的紧凑版本，整洁有序的单栏外观
# scheme: Pisces  # 默认 Scheme，这是 NexT 最初的版本，黑白主调，大量留白
# scheme: Gemini  # 类似 Pisces
```
## 设置语言
编辑**站点配置文件**，搜索关键词language，并设置成你所需要的语言：
```
language: zh-CN
```

## 设置菜单
进入**主题配置文件**，找到menu字段，菜单内容的设置格式是：item name: link || menu photo，其中item name 是一个名称，link时具体菜单分类，菜单的||后面是菜单的图标,具体菜单图标可参考[Font Awesome](http://fontawesome.dashgame.com/)网站。
```
# 菜单示例配置
menu:
  home: / || home
  reading: /reading/ || book
  archives: /archives/ || archive
  categories: /categories/ || th
  #tags: /tags/ || tags
  about: /about/ || user
```

## 头像设置
在**主题配置文件**，搜索字段avatar，值设置成头像的链接地址。
```
# 将头像放置主题目录下的 source/uploads/ （新建uploads目录若不存在） 配置为：
avatar: /uploads/avatar.png
# 放置在 source/images/ 目录下, 配置为：
avatar: /images/avatar.png
# 完整的互联网 URI
avatar: 
   url: http://example.com/avatar.png
```

## 设置侧边栏
在**主题配置文件**，搜索sidebar关键词，设置为hide模式，如下图所示：
```
sidebar:
  #display: post    // 默认显示方式
  #display: always  // 一直显示
  display: hide     // 初始隐藏
  #display: remove  // 移除侧边栏
```
各位可根据个人喜好进行设置。

## 设置站点描述
在**站点配置文件**中，搜索关键词Site，如下：
```
# Site
title: Chloneda		        #你的站点标题
subtitle: Less is more
description: Less is more	#你的站点描述
keywords: chloneda
author: chloneda	        #站点作者
```

# 进阶设定
## 添加标签页面
hexo根目录下，执行以下命令，新建标签页面。
```
hexo new page tags
```
修改站点目录**source/tags**的 index.md 文件：
```
---
title: 添加标签页面测试
tags: Test	#添加标签
categories: Test	#添加分类
comments: false
---
```
修改**主题配置文件**，搜索关键词menu，取消 **#tags: /tags/ || tags**注释，内容如下:
```
# 菜单示例配置
menu:
  home: / || home
  reading: /reading/ || book
  archives: /archives/ || archive
  categories: /categories/ || th
  tags: /tags/ || tags
  about: /about/ || user
```

## 新添菜单翻译对应的中文

打开 **hexo>theme>next>languages>zh-CN.yml** 文件，在menu下添加 **tags: 标签**：
```
menu:
  home: 首页
  archives: 归档
  categories: 分类
  tags: 标签
  about: 关于
  search: 搜索
  schedule: 日程表
  sitemap: 站点地图
  commonweal: 公益404
  resources: 资源
```
**注：添加其他页面也类似。**

## 首页显示预览
首页显示文章列表，列表里的每一篇文章只显示预览，不显示全文。

进入**主题配置文件**，搜索关键词**auto_excerpt**，把enable对应的false改为true。
```
# Automatically Excerpt. Not recommand.
# Please use <!-- more --> in the post to control excerpt accurately.
auto_excerpt:
  enable: false
  length: 150
```

## 友情链接
打开**主题配置文件**,搜索关键字 Blog rolls,添加自己需要的链接：
```
links: #连接
  baidu: https://www.baidu.com/
  google: https://www.google.com/
```

## 本地搜索
在**Hexo**的根目录下执行以下命令。
```
$ npm install hexo-generator-searchdb --save
```

打开**主题配置文件**,搜索关键字local_search,将enable的值设置为 true：
```
# Local search
# Dependencies: https://github.com/theme-next/hexo-generator-searchdb
local_search:
  enable: true
```
打开**站点配置文件**，搜索关键词search，修改为如下内容：
```
# 本地搜索
search:
  path: search.xml
  field: post
  format: html
  limit: 10000

```

## 添加RSS
在Hexo根目录执行安装指令，安装 **hexo-generator-feed** 插件：
```
npm install hexo-generator-feed --save
```
打开**站点配置文件**，追加feed信息:
```
# 设置RSS
feed:
  type: rss2
  path: rss2.html
  limit: 5
  hub:
  content: 'true'

```
打开**主题配置文件**，找到rss，设置为:
```
rss: /atom.xml
```

## 添加社交链接
在**主题配置文件**中，找到social属性，添加社交链接，步骤如下：
```
social:
  E-Mail: mailto:yourname@gmail.com || envelope
  Google: https://plus.google.com/yourname || google
  Twitter: https://twitter.com/yourname || twitter
  Facebook: https://www.facebook.com/yourname || facebook
```
格式为： 社交平台名称：链接

## 设置代码高亮
首先需要改动的地方有：
+ 站点配置文件_config.yml。
+ 主题配置文件_config.yml。

在**站点配置文件**中，搜索highlight关键词:
```
highlight:
  enable: true
  line_number: true
  auto_detect: true
  tab_replace:
```

文字自动检测默认不启动，改成true使其起作用。

再到**主题配置文件**，搜索highlight_theme关键词，修改代码主题样式：
```
# Code Highlight theme
# Available values: normal | night | night eighties | night blue | night bright
# https://github.com/chriskempson/tomorrow-theme
highlight_theme: night
```

## 添加复制按钮
在**主题配置文件**中，搜索关键词codeblock，将copy_button的enable值修改为true。
```
codeblock:
  # Manual define the border radius in codeblock
  # Leave it empty for the default 1
  border_radius:
  # Add copy button on codeblock
  copy_button:
    enable: true
```

## 添加阅读次数统计
**主题配置文件**中，搜索关键词busuanzi_count，设置文章阅读次数统计及网站访客量:
```
# Show Views/Visitors of the website/page with busuanzi.
# Get more information on http://ibruce.info/2015/04/04/busuanzi
busuanzi_count:
  enable: true
  total_visitors: true
  total_visitors_icon: user
  total_views: true
  total_views_icon: eye
  post_views: true
  post_views_icon: eye
```

## 添加 README.md
每个项目**README.md**文件可以简单说明这个项目的用途。在Hexo目录下的 source 根目录下添加一个 README.md 文件，修改**站点配置文件**，将 skip_render 参数的值设置为：
```
skip_render: README.md
```
再次使用**hexo d**命令部署博客的时候就不会在渲染 README.md 这个文件。

# 进阶配置
## 自定义网站头像
自定义头像可以使用 **[比特虫](http://www.bitbug.net/)** 网站制作！

在**主题配置文件**中，按以下修改：
```
favicon:
  small: /images/favicon-16x16-next.png		#你的头像名称
  medium: /images/favicon-32x32-next.png	#你的头像名称
  apple_touch_icon: /images/apple-touch-icon-next.png
  safari_pinned_tab: /images/logo.svg
  #android_manifest: /images/manifest.json
  #ms_browserconfig: /images/browserconfig.xml
```

## 添加自定义页面[友链]
**设置菜单项的显示中文文本**，打开**themes/next/languages/zh-CN.yml**文件,搜索 menu 关键字，修改对应中文或者新增。
```
menu:
  home: 首页
  archives: 归档
  categories: 分类
  tags: 标签
  about: 关于
  search: 搜索
  # schedule: 日程表
  # sitemap: 站点地图
  # commonweal: 公益404
  # 新增menu
  links: 友链	# 新增该选项表示新增“友链”菜单
```
在**主题配置文件**，搜索menu，新增**links: /links/ || link**：
```
# 菜单示例配置
menu:
  home: / || home
  reading: /reading/ || book
  archives: /archives/ || archive
  categories: /categories/ || th
  #tags: /tags/ || tags
  about: /about/ || user
  links: /links/ || link
```
hexo根目录下，执行以下命令，新建友链页面。
```
hexo new page links
```
修改站点目录下**source/links**的 index.md 文件：
```
---
title: 友链
tags: links
categories: links
comments: false
---
```
**注：其它自定义菜单也是类似步骤**

## 增加背景音乐
在本博客的**侧边栏**增加网易云音乐，生成音乐外链可参考**[链接](https://jingyan.baidu.com/article/4e5b3e19fe033691911e2466.html)** ，复制链接，将外链插入到Hexo根路径的侧边栏文件中：**/themes/next/layout/_macro/sidebar.swig**，即侧边栏友情链接**theme.links**这一项之后。
```
{% if theme.links %}
    <div>
      <div class="links-of-blogroll-title">
       ....省略部分代码
      </div>
      <ul class="links-of-blogroll-list">
        ....省略部分代码
      </ul>
    </div>
{% endif %}

    <div id="music163player">
	<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 
		src="//music.163.com/outchain/player?type=2&id=5239700&auto=0&height=66">
	</iframe>
    </div>
```

## 添加打赏功能
如今已进入知识付费时代，打赏是读者对笔者创造的最大支持，更是对劳动者的尊重。打赏功能具体步骤为：
**获取二维码**
- 微信二维码的获取（可百度）。
- 获取支付宝收款二维码（可百度）。

**添加二维码图片资源**
得到二维码图片资源后，读者们可将二维码图片放到**NexT根目录/source/images/**文件夹下。

**开启打赏功能**
打开**主题配置文件**，搜索reward关键词，添加打赏的配置信息。
```
# Reward
# If true, reward would be displayed in every article by default.
# And you can show or hide one article specially through add page variable `reward: true/false`.
reward:
  enable: true  //默认是false，改为true
  comment: 您的支持是对我最大的鼓励
  wechatpay: /images/wechatpay.jpg	#图片链接或图片相对路径
  alipay: /images/alipay.jpg	  #图片链接或图片相对路径
```

## 开启版权声明
打开**主题配置文件**,搜索关键字 creative_commons , post 改为 true：
```
creative_commons:
  license: by-nc-sa
  sidebar: false
  post: true
```

# 优化及设置
## 优化url
seo搜索引擎优化认为，网站的最佳结构是三层，但是默认hexo编译的站点打开文章的url是：sitename/year/mounth/day/title四层的结构，url不利于搜索引擎搜索。

因此，我们可以将url直接改成sitename/blog/title的形式，同时title最好是用英文，在**站点配置文件**搜索permalink关键词，并修改如下。
```
url: https://chloneda.github.io/
root: /
permalink: /blog/:title.html
permalink_defaults:
```

## Hexo博客备份
利用github分支功能进行博客备份，思路说明:
+ master分支：存放博客的静态网页(默认分支)。
+ hexo分支：存放Hexo博客的源码文件。

**master分支部署**
进入**站点配置文件**编辑，搜索deploy关键词：
```
deploy:
  type: git
  repo: https://github.com/你的github用户名/你的github用户名.github.io.git
  branch: master
```
- 修改更新博客内容并保存。
- 执行hexo clean清除本地旧代码。
- 执行hexo g -d生成静态网站并部署到GitHub的master分支上。

**hexo分支配置**
- hexo分支，该分支为博客源码分支。
- 使用git clone -b hexo 你的github仓库路径， 拷贝源码仓库。
- 修改hexo主配置_config.xml的deploy部分配置，设置静态页面的发布分支为master。
- 添加.gitignore文件，将静态网页的目录及其他无需提交的源文件及目录排除掉。

**博客源码更新**
在本地对博客进行修改后，提交hexo源代码：
```
git checkout hexo
git add .
git commit -m 'Code update'
git push origin hexo
```
**发布hexo静态文件**
hexo根目录依次执行以下命令：
```
hexo clean
hexo generate	或者   hexo g
hexo deploy	或者   hexo d
```
本地资料丢失或其他主机搭建博客步骤：
+ 拷贝hexo分支源码到本地：git clone -b hexo github项目地址.git。
+ 安装hexo及各类插件。
+ 本地安装调试。

## Hexo部署脚本
Hexo修改后利用deploy.sh脚本一键部署，提高部署效率。
```
#!/bin/bash
DIR=`dirname $0`

# Generate blog
hexo clean
hexo generate
sleep 5

# Deploy
hexo deploy
sleep 5

# Push hexo code
git add .
current_date=`date "+%Y-%m-%d %H:%M:%S"`
git commit -m "Blog updated: $current_date"

sleep 2
git push origin hexo

echo "=====>Finish!<====="
```
把该脚本存放至 hexo根目录中，并附加脚本执行权限:
```
chmod 775 deploy.sh
```
在hexo目录根执行脚本:
```
./deploy.sh
```
可一键部署博客及备份博客源码至github的分支hexoCode上。

# 提升你的博客
更多提升NexT主题的方法请参考以下网页。
- [Hexo高阶教程：打造定制化博客](https://juejin.im/post/58eb2fd2a0bb9f006928f8c7#heading-21)
- [让你的hexo博客在搜索引擎中排第一](https://juejin.im/post/590b451a0ce46300588c43a0#heading-0)

# 结束小语
本文采用Github + Hexo搭建的个人博客，在搭建过程优化总结，及时记录，希望对各位有所帮助！本文如有错误，欢迎在 [Github](https://github.com/chloneda/chloneda.github.io) 的issue中提出，非常感谢！！！

更多详情请参考：
- [Hexo官网](https://hexo.io/zh-cn/)
- [NexT主题](http://theme-next.iissnan.com/)
- [Hexo插件](https://hexo.io/plugins/)
- [Markdown](https://markdown-zh.readthedocs.io/en/latest/)



---










