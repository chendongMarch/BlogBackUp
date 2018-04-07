---
layout: post
title: Hexo
categories:
  - Hexo
tags:
  - Hexo
keywords:
  - Hexo
  - NexT
abbrlink: 185805107
date: 2017-03-02 00:00:00
---

本文主要介绍如何使用 `Hexo+NexT` 搭建自己的博客之后，和在建站期间遇到的一些问题以及优化方式。

<!--more-->

## 推荐阅读

[Hexo+Next主题优化](https://zhuanlan.zhihu.com/p/30836436?utm_medium=social&utm_source=wechat_session)

## 弄懂 NexT
`post.md` 中 `bool` 属性默认值是 `false`

## BugFix

错误 `Cannot set property 'lastIndex' of undefined`，修改 `hexo/config.yml` 设置 `auto_detect: false`

```css
highlight:
  enable: true
  line_number: true
  auto_detect: false
  tab_replace: 
```

异常 `Cannot find module './build/Release/DTraceProviderBindings'` 不影响使用，但是看着难受，尝试

```bash
npm install hexo --no-optional
// 如果上面的命令有效，可以不执行以下两条
npm uninstall hexo-cli -g
npm install hexo-cli -g
```




## SEO优化

> [Hexo NexT主题SEO优化](https://lancelot_lewis.coding.me/2016/08/16/blog/Hexo-NexT-SEO/)

> [SEO优化，Google收录](http://www.jianshu.com/p/86557c34b671)


## 加快访问速度

因为是放在 GitHub 上面的，所以访问难免会慢。

可以发布以后检查一下网页加载的瓶颈在哪里，然后做相应的优化。

需要注意的是，放在本地的图片访问会变的很慢，因为资源在 GitHub 上面，所以尽量用七牛等的云存储，文章中使用 url。

fontawsome 访问慢，替换 _config.yml 中的源，效果会好很多

```
vendors:
  # Internal path prefix. Please do not edit it.
  _internal: vendors
  # Internal version: 2.1.3
  jquery: //cdn.bootcss.com/jquery/2.1.3/jquery.min.js
  # Internal version: 2.1.5
  # Fancybox: http://fancyapps.com/fancybox/
  fancybox: //cdn.bootcss.com/fancybox/2.1.5/jquery.fancybox.pack.js
  fancybox_css: //cdn.bootcss.com/fancybox/2.1.5/jquery.fancybox.min.css
  # Internal version: 1.0.6
  fastclick: //cdn.bootcss.com/fastclick/1.0.6/fastclick.min.js
  # Internal version: 1.9.7
  lazyload: //cdn.bootcss.com/jquery_lazyload/1.9.7/jquery.lazyload.min.js
  # Internal version: 1.2.1
  velocity: //cdn.bootcss.com/velocity/1.3.1/velocity.min.js
  # Internal version: 1.2.1
  velocity_ui: //cdn.bootcss.com/velocity/1.3.1/velocity.ui.min.js
  # Internal version: 0.7.9
  ua_parser: //cdn.bootcss.com/UAParser.js/0.7.12/ua-parser.min.js
  # Internal version: 4.4.0
  # http://fontawesome.io/
  fontawesome: //cdn.bootcss.com/font-awesome/4.6.2/css/font-awesome.min.css
```

## 页面简单加密访问    

找到文件`themes->next->layout->_partials->head.swig`在`<meta>`标签之后添加如下js代码

```js
<script>
    (function(){
        if('{{ page.password }}'){
            if (prompt('请输入文章密码') !== '{{ page.password }}'){
                alert('密码错误！');
                history.back();
            }
        }
    })();
</script>
```

在文章的`Front-matter`定义`password`

```
---
layout: post
title: Hexo加密文章示例
date: 2017-03-02
category: 
tags: 
keywords:
password: 123456
---
```


## 添加留言界面

添加page，在博客根目录下执行如下命令，会在source目录下面创建`msg/index.md`文件。

```bash
hexo new page msg
```
 
配置文字和图标，打开`themes/next/_config.yml`文件，进行如下配置，才能使留言的界面显示

```
menu:
  home: /
  #about: /about
  archives: /archives
  tags: /tags
  categories: /categories
  guestbook: /guestbook
```
配置图标，打开`themes/next/_config.yml`文件，进行如下配置，可以自定义显示在侧边栏的留言的图标，图标的配置使用key-value的形式，下面代码中`home``calendar`等都是key，使用这个key可以从[这个网站-FontAwesome](http://www.bootcss.com/p/font-awesome/#icons-web-app)获取图标，如果想更换图标，只需要去那个网站获取图标的名字去掉头部的icon作为key添加到下面代码中即可。

```
menu_icons:
  enable: true
  home: home
  about: user
  categories: th
  schedule: calendar
  tags: tags
  guestbook : comments
  archives: archive
  commonweal: heartbeat
```
打开`themes/next/language/zh-CN.yml`文件，进行如下配置，才能显示为 留言 字样

```
menu:
  home: 首页
  archives: 归档
  categories: 分类
  tags: 标签
  about: 关于
  search: 搜索
  commonweal: 公益404
  guestbook: 留言
```


## 过滤文件不进行渲染

在给站点添加 `README.md` 和 Google，百度相关的验证文件时，我们希望这个文件不要被主题渲染，而是源文件直接拷贝到 `public` 文件夹中，只需要在 `hexo/_config.yml` 中进行如下配置。

```yml
skip_render:
    - 'README.*'
    - 'google26933bad87c2b3ba.*'
    - 'baidu_verify_HnYctWkkrH.*'
```


## category 分级

在头部如下写入 categories 标签

```
categories: 
		- Extensions
		- Hexo
```

##  链接唯一化

安装 `hexo-abbrlink` 插件

```bash
npm install hexo-abbrlink --save
```

配置 `hexo/_config.yml`，添加如下，完成后打开文件，保存几次，会自动添加 `abbrlink` 属性，如果已经有值了则不会更改。

```
permalink: article/:abbrlink/  # article/ 可自行更换

# abbrlink config
abbrlink:
  alg: crc32  # 算法：crc16(default) and crc32
  rep: hex    # 进制：dec(default) and hex
```

## 文章压缩

安装插件 `hexo-all-minifier`

```bash
npm install hexo-all-minifier
```

配置 `hexo/_config.yml`，添加如下

```
# 文章压缩
html_minifier:
  enable: true
  ignore_error: false
  exclude:

css_minifier:
  enable: true
  exclude:
    - '*.min.css'

js_minifier:
  enable: true
  mangle: true
  output:
  compress:
  exclude:
    - '*.min.js'

image_minifier:
  enable: true
  interlaced: false
  multipass: false
  optimizationLevel: 2
  pngquant: false
  progressive: false
```


## 添加 fork me on github

修改 `hexoBlog/themes/next/layout/_layout.swig` 文件，在 `header`
标签之前添加图片，图片样式可以到[丝带状的Fork](https://github.com/blog/273-github-ribbons) 或者是 [三角形的的Fork](http://tholman.com/github-corners/)选择颜色和位置等。`href` 中需要修改你为的 `github` 地址
```
 <div class="{{ container_class }} {% block page_class %}{% endblock %} ">
    <div class="headband"></div>

  <!--- add Fork me on Github  -->
    <div class="forkme">
   <a target="_blank" href="https://github.com/chendongMarch"><img style="position: absolute; top: 0; left: 0; border: 0;" src="https://camo.githubusercontent.com/567c3a48d796e2fc06ea80409cc9dd82bf714434/68747470733a2f2f73332e616d617a6f6e6177732e636f6d2f6769746875622f726962626f6e732f666f726b6d655f6c6566745f6461726b626c75655f3132313632312e706e67" alt="Fork me on GitHub" data-canonical-src="https://s3.amazonaws.com/github/ribbons/forkme_left_darkblue_121621.png"></a>
    </div>
    <!--- add Fork me on Github -->   
    
    <header id="header" class="header" itemscope itemtype="//schema.org/WPHeader">
```
在 `head` 标签内添加 `style` 使小屏幕访问时不显示这个图片

```html
<head>
  {% include '_partials/head.swig' %}
  <title>{% block title %}{% endblock %}</title>
<style>
  .forkme{
    display: none;
  }
@media (min-width: 768px) {
  .forkme{
    display: inline;
  }
}
  </style>
  {% include '_third-party/analytics/index.swig' %}
</head>
```

## 代码风格处理

查看 `hexoBlog/themes/next/source/css/_common/components/highlight/highlight.styl` 文件，是对高亮代码的相关配置。找到标签修改既可。
 
```
// 行内代码
code{
	// 分行时是不是根据单词划分
	word-wrap: break-all/break-word;
	// 代码颜色
	color:#fff;
}
```


## 添加本地搜索

安装插件 `hexo-generator-searchdb`

```
npm install hexo-generator-searchdb --save
```
配置 `hexo/_config.yml`

```
search:
  path: search.xml
  field: post
  format: html
  limit: 10000
```
配置 `next/_config.yml`

```
# Local search
local_search:
  enable: true
```



## 添加加载进度

在 `Next` 主题下打开 `/next/layout/_partials/head.swig` 文件，在最后追加进度条

下面的代码只在浏览器显示，手机上不显示进度条

```css
{# 添加进度条 #}
<script src="//cdn.bootcss.com/pace/1.0.2/pace.min.js"></script>
<link href="//cdn.bootcss.com/pace/1.0.2/themes/pink/pace-theme-flash.css" rel="stylesheet">
<style>
      .pace .pace-progress {
        background: #0000; /*进度条颜色*/
        height: 0px;
      }
      .pace .pace-progress-inner {
         box-shadow: 0 0 0px #0000, 0 0 0px     #0000; /*阴影颜色*/
      }
      .pace .pace-activity {
        display: none;
        border-top-color: #00d9cf;    /*上边框颜色*/
        border-left-color: #00d9cf;    /*左边框颜色*/
      }

    @media screen and (min-width: 800px) {
      .pace .pace-progress {
        background: #00d9cf; /*进度条颜色*/
        height: 2px;
      }
      .pace .pace-progress-inner {
         box-shadow: 0 0 10px #00d9cf, 0 0 5px #00d9cf; /*阴影颜色*/
      }
      .pace .pace-activity {
        border-top-color: #00d9cf;    /*上边框颜色*/
        border-left-color: #00d9cf;    /*左边框颜色*/
      }
    }
 
</style>
```

## 解决表格被等分

文件 `themes/next/source/css/_common/scaffolding/tables.styl` 

```
table-layout: auto;
```


## 添加 Leancloud 统计
今天发现网页访问很慢，查了之后发现不蒜子统计访问太慢，换成leancloud,但是只有pv统计。

[参考文章](https://notes.wanghao.work/2015-10-21-%E4%B8%BANexT%E4%B8%BB%E9%A2%98%E6%B7%BB%E5%8A%A0%E6%96%87%E7%AB%A0%E9%98%85%E8%AF%BB%E9%87%8F%E7%BB%9F%E8%AE%A1%E5%8A%9F%E8%83%BD.html#%E9%85%8D%E7%BD%AELeanCloud)


## 指定文章不在首页显示
文件 `/Users/march/Documents/hexoBlog/themes/next/layout/index.swig` 修改。


```
<section id="posts" class="posts-expand">
    {% for post in page.posts %}
      {%if !post.hide %}
      	{{ post_template.render(post, true) }}
      {% endif %}
    {% endfor %}
</section>

// render 之前判断
{%if !post.hide %}
   {{ post_template.render(post, true) }}
{% endif %}

```
然后在文章文件头部，增加

```
---
...
hide: true
---
```


## 网站背景 

/Users/march/Documents/hexoBlog/themes/next/source/css/_custom/custom.styl

```css
body{
	background-image: url(/images/background.png);
}
```


## 头部显示

/Users/march/Documents/hexoBlog/themes/next/source/css/_schemes/Mist/_header.styl

```css
.header { background: $whitesmoke; }
```

## 更改全局字体

/Users/march/Documents/hexoBlog/themes/next/source/css/_variables/custom.styl

```

$font-size-base = 15.5px; // 默认是 16px 大小
$font-family-headings = Georgia, sans // 标题，修改成你期望的字体族
$font-family-base = "Microsoft YaHei", Verdana, sans-serif // 修改成你期望的字体族
```

## 文章中加图片，首页不显示图片

/Users/march/Documents/hexoBlog/themes/next/layout/_macro/post.swig

上面有个 gallery support，加上 not is_index 就可以不再首页显示，然后 md 文件顶部使用 photos 声明图片链接 photos: './img.png'，可以多张

```
{% if not is_index and post.photos and post.photos.length %}
        <div class="post-gallery" itemscope itemtype="http://schema.org/ImageGallery">
          {% set COLUMN_NUMBER = 3 %}
          {% for photo in post.photos %}
            {% if loop.index0 % COLUMN_NUMBER === 0 %}<div class="post-gallery-row">{% endif %}
              <a class="post-gallery-img fancybox" href="{{ url_for(photo) }}" rel="gallery_{{ post._id }}" itemscope itemtype="http://schema.org/ImageObject" itemprop="url">
                <img src="{{ url_for(photo) }}" itemprop="contentUrl"/>
              </a>
            {% if loop.index0 % COLUMN_NUMBER === 2 %}</div>{% endif %}
          {% endfor %}

          {# Append end tag for `post-gallery-row` when (photos size mod COLUMN_NUMBER) is less than COLUMN_NUMBER #}
          {% if post.photos.length % COLUMN_NUMBER > 0 %}</div>{% endif %}
        </div>
      {% endif %}
```



## 头像为圆形，可移动
themes/next/source/css/_common/components/sidebar/sidebar-author.syl

```css
.site-author-image {
  display: block;
  margin: 0 auto;
  padding: $site-author-image-padding;
  max-width: $site-author-image-width;
  height: $site-author-image-height;
  border: $site-author-image-border-width solid $site-author-image-border-color;
  
  // 修改头像边框
  border-radius: 50%;
  -webkit-border-radius: 50%;
  -moz-border-radius: 50%;
  // 设置旋转
  transition: 1.4s all;
}


// 可旋转的圆形头像,`hover`动作
.site-author-image:hover {
    -webkit-transform: rotate(360deg);
    -moz-transform: rotate(360deg);
    -ms-transform: rotate(360deg);
    -transform: rotate(360deg);
}
```

## 主页文章添加卡片背景
themes/next/source/css/_custom/custom.styl

```
// 主页文章添加阴影效果
.post{
   margin-top: 0px;
   margin-bottom: 60px;
   padding: 25px;
   -webkit-box-shadow: 0 0 5px rgba(180, 180, 180, .5);
   -moz-box-shadow: 0 0 5px rgba(180, 180, 180, .5);
}
```

## copyright
themes/next/layout/_macro/post-copyright.swig

```
{{ 
       __('post.copyright.license_content', theme.post_copyright.license_url, theme.post_copyright.license)
       }}
```


## 文章内链接

themes/next/source/css/_custom/custom.styl

```
// 文章内链接文本样式
.post-body p a{
  color: #0593d3;
  border-bottom: none;
  border-bottom: 1px solid #0593d3;
  padding-left:5px;
  padding-right:5px;
  &:hover {
    color: #fc6423;
    border-bottom: none;
    border-bottom: 1px solid #fc6423;
    text-decoration: none;
    border-radius:0;
  }
}
```


## 统一添加“本文结束”标记

在路径 /themes/next/layout/_macro 中新建 passage-end-tag.swig 文件,并添加以下内容：

```
<div>
    {% if not is_index %}
        <div style="text-align:center;color: #aaa;font-size:14px;margin-top:2rem;">------ 本文结束 🎉🎉 谢谢观看  ------</div>
    {% endif %}
</div>
```

打开 themes/next/layout/_macro/post.swig 文件,添加：

```
<div>
    {% if not is_index %}
    {% include 'passage-end-tag.swig' %}
    {% endif %}
 </div>
```
然后打开主题配置文件 _config.yml,在末尾添加：

```
# 文章末尾添加“本文结束”标记
passage_end_tag:
enabled: true
```


## 键盘触发侧栏

/Users/march/Documents/hexoBlog/themes/next/source/js/src/motion.js

```js
$(document).ready(function () {
  NexT.motion = {};

  window.onkeydown=function(){
　　　　if(13 == event.keyCode){
　　　　　　NexT.utils.displaySidebar();
　　　　}
  };
}
```

## 添加 live2d 看板娘

添加一个小姐姐

https://github.com/EYHN/hexo-helper-live2d/blob/master/README.zh-CN.md

```bash
npm install --save hexo-helper-live2d
```

hexo/_config.yml

```bash
live2d:
  enable: true
  scriptFrom: local
  pluginRootPath: live2dw/
  pluginJsPath: lib/
  pluginModelPath: assets/
  model:
    use: live2d-widget-model-z16
    scale: 1
    hHeadPos: 1
    vHeadPos: 1
  display:
    superSample: 3
    width: 60
    height: 120
    position: right
    hOffset: 120
    vOffset: -40
  mobile:
    show: false
  react:
    opacityDefault: 1
    opacityOnHover: 1
```

