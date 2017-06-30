---
layout: post
title: ListView Adapter 4
categories:
  - Android
  - Adapter
tags:
  - Android
  - ListView
  - Adapter
keywords:
  - Android
  - ListView
  - Adapter
abbrlink: 2414640248
date: 2016-03-09 00:00:00
---

## GitHub 
[库代码链接](https://github.com/chendongMarch/QuickAdapter)

## Gradle
`compile 'com.march.adapterlibs:adapterlibs:1.0.6'`


## 前言
- 基于前端时间总结的[EasyAdapter（看原理的可以看这些博客）](http://blog.csdn.net/chendong_/article/details/48812405)重新构建了代码，基本思想还是一样的，目的是对外开放更加简单的API,其中的原理，想知晓的可以去看我以前的博客。现在的版本已经与以前的版本独立开了。


## 单类型抽象适配
### QuickAdapter
```
QuickAdapter<Demo> adapter =
new QuickAdapter<Demo>(BaseApplication.getInst(), R.layout.item_quickadapter, data) {
            @Override
            public void bindData4View(ViewHolder holder, Demo data, int pos) {
                holder.setText(R.id.item_quickadapter_title, data.getmDemoTitle());
            }

            @Override
            public void bindListener4View(ViewHolder holder, Demo data, int pos) {
					  //在这里给内部控件绑定监听不是必须实现的
            }
        };
```


### 使用QuickTypeAdapter
- QuickTypeAdapter的带layout资源参数构造方法,注意不要调用addType了,实体类需要实现QuickInterface接口


```java
QuickTypeAdapter<Demo> typeAdapter =
new QuickTypeAdapter<Demo>(BaseApplication.getInst(), data,R.layout.item_a) {
            @Override
            public void bindData4View(ViewHolder holder, Demo data, int type, int pos) {
                     //单类型,type一直返回0
            }
            @Override
            public void bindListener4View(ViewHolder holder, Demo data, int type, int pos) {
                    //在这里给内部控件绑定监听不是必须实现的
            }
};
```

## 多类型适配
```
//实体类需要实现QuickInterface接口
QuickTypeAdapter<Demo> typeAdapter = 
new QuickTypeAdapter<Demo>(BaseApplication.getInst(), data) {
            @Override
            public void bindData4View(ViewHolder holder, Demo data, int type, int pos) {
                switch (type) {
                    case Demo.CODE_DETAIL:
                        holder.setText(R.id.item_quickadapter_type_title, data.getmDemoTitle());
                        break;
                    case Demo.JUST_TEST:
                        holder.setText(R.id.item_quickadapter_title, data.getmDemoTitle());
                        break;
                }
            }

            @Override
            public void bindListener4View(ViewHolder holder, Demo data, int type, int pos) {
            //在这里给内部控件绑定监听不是必须实现的
            }
        };
//添加每种类型的xml文件
typeAdapter.addType(Demo.CODE_DETAIL, R.layout.item_quickadapter_type)
                .addType(Demo.JUST_TEST, R.layout.item_quickadapter);
```

# 加载网络图片
- 可以提前创建图片加载工具,然后调用ViewHodler.setImg()方法可以直接加载


```java
//你可以在Activity或者Application调用这段代码进行全局配置,第二次调用会将以前的设置覆盖,所以只需要执行一次
Quick.init(new Quick.QuickLoad() {
            @Override
            public void load(Context context, String url, ImageView view) {
                Log.e("chendong","加载图片");
                Glide.with(context).load("http://www.fresco-cn.org/static/fresco-logo.png").into(view);
            }
        });
```

# 更新使用方法
```java
//Demo类是我的实体类
//如果你使用的控件ViewHolder没有为你集成,如何避免强转?使用泛型解决
 holder.<Button>getView(R.id.abc).setText("");

//设置监听事件
public ViewHolder setLis(int resId,View.OnClickListener listener,Object tag)//带有tag监听
public ViewHolder setLis(int resId,View.OnClickListener listener)//不带tag监听
public ViewHolder setTag(int resId, Object tag)//给控件设置tag
public <T> T getTag(int resId)//从ViewHolder获取tag,包含泛型,你可以这样holder.<Demo>getTag(R.id.xxx)

//如果你在控件中设置了tag,当你在适配器外部使用tag时务必使用改方法获取,用来替代view.getTag()方法,
包含泛型,你可以这样用 Quick.<Demo>getTagOutOfAdapter(listView);
Quick.getTagOutOfAdapter(View view)
```
