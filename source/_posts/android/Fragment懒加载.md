---
layout: post
title: Fragment懒加载
date: 2017-03-14
category: Android
tags: Android
keywords: 
---

> ViewPager结合Fragment使用时，由于ViewPager的特性，会预先加载当前显示页面左右两边的页面，也就是说默认会缓存3个页面，（也可以使用  mViewPager.setOffscreenPageLimit(3); 这个方法来改变这个设置。）
