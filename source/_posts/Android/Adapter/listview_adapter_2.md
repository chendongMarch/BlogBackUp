---
layout: post
title: ListView Adapter 2
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
abbrlink: 1720703307
date: 2015-11-14 00:00:00
---


## 前言
- 在使用适配器为ListView填充数据时，数据类型往往不是单一的，单一的数据显示太单调，对数据格式要求也比较高，我们在项目中往往使用网络请求获取json数据然后将其显示在Item中，此时获取的数据类型可能会有两到三种格式，此时就用到了分类适配，使用分类适配可以更加友好显示ListView。


- 既然需要分类适配，我们就必须拿到数据的类型，你的数据也就必须直接或者间接的实现获取类型的方法，其实ListView中已经包含了这个获取类型的方法，当然你也可以使用自己的获取类型的方法。


```
//重写以下两个方法，listview在内部会准备getViewTypeCount()个缓冲区，用来复用同种类型的View,不同类型的View不会出现复用错误。
@Override
public int getItemViewType(int position) {
		int type = Integer.parseInt(newsEntities.get(position).getType());
		return type;
}
@Override
public int getViewTypeCount() {
        return count;
}
```

## 进行分类适配
```
public View getView(int position, View convertView, ViewGroup parent) {
        if (getItemViewType(position) == 1) {
			ViewHolderVertical holder = null;
			if (convertView == null) {
				convertView = layoutInflater.inflate(
						R.layout.item_firstpagelist_vertical, parent, false);
				holder = new ViewHolderVertical();
				holder.iv = (ImageView) convertView
						.findViewById(R.id.item_firstpagelist_vertical_iv);
						//获取其他的控件....
				convertView.setTag(holder);
			} else {
				holder = (ViewHolderVertical) convertView.getTag();
			}
			NewsEntity entity = newsEntities.get(position);
			//为控件设置显示的数据....
		} else {
			ViewHolder holder = null;
			if (convertView == null) {
				convertView = layoutInflater.inflate(
						R.layout.item_firstpagelist_horizontal, parent, false);
				holder = new ViewHolder();
				holder.iv = (ImageView) convertView
						.findViewById(R.id.item_firstpagelist_iv);
				//获取其他的控件....
				convertView.setTag(holder);
			} else {
				holder = (ViewHolder) convertView.getTag();
			}
			NewsEntity entity = newsEntities.get(position);
			//为控件设置显示的数据....
		}
		return convertView;
    }
```