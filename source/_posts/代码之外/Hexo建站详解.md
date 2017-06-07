---
layout: post
title: Hexo
date: 2017-03-02
category: 代码之外
tags: []
keywords: [Hexo,NexT]
---

本文主要介绍如何使用Hexo+NexT搭建自己的博客，和在建站期间遇到的一些问题以及优化方式。

<!--more-->



## SEO优化
> [Hexo NexT主题SEO优化](https://lancelot_lewis.coding.me/2016/08/16/blog/Hexo-NexT-SEO/)

> [SEO优化，Google收录](http://www.jianshu.com/p/86557c34b671)



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
很遗憾，现在多说已经不能用了。

添加page，在博客根目录下执行如下命令，会在source目录下面创建`msg/index.md`文件

```bash
hexo new page msg
```

配置·多说·，在`msg/index.md`文件**底部**添加如下html代码

```html
<div class="ds-recent-visitors" data-num-items="28" data-avatar-size="42" id="ds-recent-visitors"></div>
```
进入**多说**站点，打开`设置->自定义CSS`，在输入框内加入如下css代码

```css
#ds-reset .ds-avatar img,
#ds-recent-visitors .ds-avatar img {
    width: 54px;
    height: 54px;     /*設置圖像的長和寬，這裏要根據自己的評論框情況更改*/
    border-radius: 27px;     /*設置圖像圓角效果,在這裏我直接設置了超過width/2的像素，即為圓形了*/
    -webkit-border-radius: 27px;     /*圓角效果：兼容webkit瀏覽器*/
    -moz-border-radius: 27px;
    box-shadow: inset 0 -1px 0 #3333sf;     /*設置圖像陰影效果*/
    -webkit-box-shadow: inset 0 -1px 0 #3333sf;
}
#ds-recent-visitors .ds-avatar {
    float: left
}
/*隱藏多說底部版權*/
#ds-thread #ds-reset .ds-powered-by {
    display: none;
}
```

### 配置文字和图标
配置页面，打开`themes/next/_config.yml`文件，进行如下配置，才能使留言的界面显示侧边栏中

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

## bug fix
错误 `Cannot set property 'lastIndex' of undefined`，修改 `hexo/config.yml` 设置 `auto_detect: false`

```css
highlight:
  enable: true
  line_number: true
  auto_detect: false
  tab_replace: 
```