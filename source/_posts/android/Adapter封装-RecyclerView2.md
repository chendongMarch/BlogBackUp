---
layout: post
title: QuickRvAdapter快速适配器
date: 2016-08-05
category: Android
tags: [Android,Adapter,RecyclerView,开源库]
keywords: 
description: QuickRvAdapter快速适配器
---


# RecyclerView Adapter

1. 为RecyclerView提供更简单的适配器实现方式，不断更新完善中。

2. [Demo视频演示](http://7xtjec.com1.z0.glb.clouddn.com/ittleQ.mp4)

3. [GitHub地址](https://github.com/chendongMarch/CommonLib/lib-adapter)

4. [博客](http://blog.csdn.net/chendong_/article/details/50897581)



![](http://7xtjec.com1.z0.glb.clouddn.com/toc.png)


* [使用](#使用)
* [BaseViewHolder 的使用](#baseviewholder-的使用)
* [通用适配器](#通用适配器)
	* [单类型数据适配](#单类型数据适配)
	* [多类型数据适配](#多类型数据适配)
		* [使用 ISectionRule 配置数据](#使用-isectionrule-配置数据)
		* [使用 HashMap 配置数据](#使用-hashmap-配置数据)
	* [九宫格模式适配](#九宫格模式适配)
* [监听事件](#监听事件)
	* [三种事件](#三种事件)
	* [实现需要的事件](#实现需要的事件)
	* [SectionRvAdapter 事件](#sectionrvadapter-事件)
* [数据更新](#数据更新)
	* [内置更新方法](#内置更新方法)
	* [分页更新方法](#分页更新方法)
	* [SectionRvAdapter 追加更新](#sectionrvadapter-追加更新)
* [Module](#module)
	* [添加 Header 和 Footer](#添加-header-和-footer)
	* [预加载更多](#预加载更多)
* [其他](#其他)
	* [adapterId 区分](#adapterid-区分)
	* [Sample](#sample)


## 使用
- 类库还在开发中，暂时没有发布到Jcenter,所以需要在`yourProject.gradle`文件中添加如下代码进行依赖

```
	allprojects {
 	   repositories {
  	       maven { url 'https://dl.bintray.com/chendongmarch/maven' }
	    }
	}
```

- 在`yourApp.gradle`文件中添加依赖

```
	compile 'com.march.lib-adapter:lib-adapter:1.0.0'
```


## BaseViewHolder 的使用
- 通用ViewHolder，内部使用`SparseArray`实现View的缓存。

```java
//BaseViewHolder

//获取控件
public <T extends View> T getView(int resId)
public <T extends View> T getView(String resId)
//设置可见
public RvViewHolder setVisibility(int resId, int v)
//文字
public RvViewHolder setText(int resId, String txt)
//图片
public RvViewHolder setImg(int resId, int imgResId)
//监听
public RvViewHolder setClickLis(int resId, View.OnClickListener listener)
```
## 通用适配器
### 单类型数据适配 
- SimpleRvAdapter

```java
//一个简单的实现,实体类不需要再去实现RvQuickInterface接口
SimpleRvAdapter simpleAdapter =
new SimpleRvAdapter<Demo>(self, demos, R.layout.rvquick_item_a) {
            @Override
            public void onBindView(RvViewHolder holder, Demo data, int pos) {
                holder.setText(R.id.item_a_tv, data.title);
            }
        };
```
### 多类型数据适配 
- TypeRvAdapter

1. 多类型数据适配时需要接受`ITypeAdapterModel(接口)`类型的的数据，因此进行多类型数据适配时，你的Model需要实现`ITypeAdapterModel`告知Adapter数据的type

2. `addType(int type,int res)`方法用来给每种类型的数据添加不同的布局文件，达到多类型适配的目的。

```java
// Demo类必须实现 ITypeAdapterModel 接口
TypeRvAdapter<Demo> typeAdapter =
new TypeRvAdapter<Demo>(context, data) {
            @Override
            public void onBindView(RvViewHolder holder, Demo data, int pos, int type) {
                //根据类型绑定数据
                switch (type) {
                    case Demo.CODE_DETAIL:
                        holder.setText(R.id.item_quickadapter_type_title, data.getmDemoTitle()).setText(R.id.item_quickadapter_desc, data.getmDescStr());
                        break;
                    case Demo.JUST_TEST:
                        holder.setText(R.id.item_quickadapter_title, data.getmDemoTitle());
                        break;
                }
            }

        };
typeAdapter.addType(Demo.CODE_DETAIL, R.layout.item_layout_a)
           .addType(Demo.JUST_TEST, R.layout.item_layout_b);
```

### 九宫格模式适配
- SectionRvAdapter

1. 每个Header下面由多个item。类似微信九宫格照片展示,SectionRvAdapter实际上是多类型适配(TypeRvAdapter)的一种特殊形式，内置了SectionHeader类型的数据，用来更简单的实现九宫格模式的数据适配，因为多类型数据适配的特点他都可以使用，也就是说你可以定义多种类型的数据展示。
2. ![](http://7xtjec.com1.z0.glb.clouddn.com/item_header_small.png)

2. 作为ItemHeader的数据类型需要继承`AbsSectionHeader(抽象类)`

```java
class ItemHeader extends AbsSectionHeader {
        String itemHeaderTitle;
}
```

3. 作为每个Section内容的数据类型，如果他是单类型的不需要做其他操作，如果有多种类型的内容需要实现ITypeAdapterModel，详细参照多类型数据适配[多类型数据适配](#多类型数据适配)

```java
class Content {
		 int contentIndex
        String contentTitle;
}
```

#### 使用 ISectionRule 配置数据
1. `ISectionRule接口`,进行九宫格模式适配使用 ISectionRule 配置数据时，需要添加`ISectionRule`,这是一种规则，adapter会根据你提供的规则自动生成Header

2. 提供了两种构造方法

```java
// 直接配置 item header 和 content 的 layout 资源
public SectionRvAdapter(Context context,List<ID> originDatas,
                        int headerLayoutId, int contentLayoutId)
// 只添加 header 的 layout 资源，content的资源可以使用addType方法添加                        
public SectionRvAdapter(Context context, List<ID> originDatas,
                        int headerLayoutId)
```

```java
// ItemHeader表示header的数据类型，Content表示内部数据的数据类型
// 初始化，添加header 和 content的布局文件
adapter = new SectionRvAdapter<ItemHeader, Content>(
                this,
                contents,
                R.layout.item_header_header,
                R.layout.item_header_content) {
            @Override
            protected void onBindItemHeader(BaseViewHolder holder, ItemHeader data, int pos, int type) {
                holder.setText(R.id.info1, data.getItemHeaderTitle());
            }

            @Override
            protected void onBindContent(BaseViewHolder holder, Content data, int pos, int type) {
                holder.setText(R.id.tv, String.valueOf(data.contentIndex));
            }
        };
		
		 // 添加ISectionRule
        adapter.addItemHeaderRule(new ISectionRule<ItemHeader, Content>() {
            @Override
            public ItemHeader buildItemHeader(int currentPos, Content preData, Content currentData, Content nextData) {
            		// 生成header数据
                return new ItemHeader("create new header " + currentData.contentIndex);
            }

            @Override
            public boolean isNeedItemHeader(int currentPos, Content preData, Content currentData, Content nextData) {
					// 什么时候创建header(当是第一个数据或者index是7的倍数时，插入一个header)
                return currentPos == 0 || currentData.contentIndex % 7 == 1;
            }
        });
mRv.setAdapter(adapter);
```
#### 使用 HashMap 配置数据
1. 一个 ItemHeader 下有多个 Content，类似`Map<ItemHeader,Content>`的数据结构,可以选择在外面构造好数据来进行数据适配

2. HashMap是无序的，为了保证数据的有序性，使用`LinkedHashMap`

3. 同样的也提供了两种构造方法

```java
// 直接配置 item header 和 content 的 layout 资源
public SectionRvAdapter(Context context,
                            LinkedHashMap<IH, List<ID>> originDatas,
                            int headerLayoutId, int contentLayoutId)

// 只添加 header 的 layout 资源，content的资源可以使用addType方法添加 
public SectionRvAdapter(Context context,
                            LinkedHashMap<IH, List<ID>> originDatas,
                            int headerLayoutId) 
```

```java
final LinkedHashMap<ItemHeader, List<Content>> map = new LinkedHashMap<>();
adapter = new SectionRvAdapter<ItemHeader, Content>(this, map,
                R.layout.item_header_header,
                R.layout.item_header_content) {
            @Override
            protected void onBindItemHeader(BaseViewHolder holder, ItemHeader data, int pos, int type) {
                holder.setText(R.id.info1, data.getItemHeaderTitle());
            }

            @Override
            protected void onBindContent(BaseViewHolder holder, Content data, int pos, int type) {
                TextView tv = (TextView) holder.getView(R.id.tv);
            }
        };
```

## 监听事件
支持单击、双击和长按事件
### 三种事件
```java
public interface OnItemListener<D> {
    // 单击事件
    void onClick(int pos, BaseViewHolder holder, D data);
    // 长按事件
    void onLongPress(int pos, BaseViewHolder holder, D data);
    // 双击事件
    void onDoubleClick(int pos, BaseViewHolder holder, D data);
}
```
### 实现需要的事件
抽象类的实现，可以选择性的实现需要的方法

```java
public abstract class SimpleItemListener<D> implements OnItemListener<D> {
    
    @Override
    public void onClick(int pos, BaseViewHolder holder, D data) {

    }

    @Override
    public void onLongPress(int pos, BaseViewHolder holder, D data) {

    }

    @Override
    public void onDoubleClick(int pos, BaseViewHolder holder, D data) {

    }
}

			adapter.setItemListener(new SimpleItemListener <GuideData>() {
            @Override
            public void onClick(int pos, BaseViewHolder holder, GuideData data) {
                 Toast.makeText(mContext, "单击事件", Toast.LENGTH_SHORT).show();
            }

            @Override
            public void onLongPress(int pos, BaseViewHolder holder, GuideData data) {
                Toast.makeText(mContext, "长按事件", Toast.LENGTH_SHORT).show();
            }

            @Override
            public void onDoubleClick(int pos, BaseViewHolder holder, GuideData data) {
                Toast.makeText(mContext,"双击事件", Toast.LENGTH_SHORT).show();
            }
        });
```

### SectionRvAdapter 事件

```java
adapter.setItemListener(new SimpleItemListener<ItemModel>() {
            @Override
            public void onClick(int pos, BaseViewHolder holder, ItemModel data) {
            			//  当是Content数据类型
                if (data.getRvType() == AbsAdapter.TYPE_ITEM_DEFAULT) {
                    Content content = (Content) data.get();
                    Toast.makeText(SectionAdapterTest.this, content.contentTitle, Toast.LENGTH_SHORT).show();
                }else{
                		// 当是ItemHeader数据类型
                }
            }
        });
```

## 数据更新
为了简化数据更新的方法，内置了数据更新的部分方法
### 内置更新方法
```java
// 插入一条数据
public void insert(int pos, D data) 
// 更新数据，isUpdate为false时只会添加数据不会更新显示
public void notifyDataSetChanged(List<D> data, boolean isUpdate)
```
### 分页更新方法
```java
// 简化分页加载的更新，调用该方法实现增量更新，不会全部刷新，isAllData为true时表示data是全部数据，为false时表示是追加的数据
public void appendTailRangeData(List<D> data, boolean isAllData)
```
### SectionRvAdapter 追加更新
// SectionRvAdapter比较特别，需要使用单独的更新方法
```java
// 使用SectionRule配置数据时，使用此方法更新
public void updateDataAndItemHeader(List<ID> data)
// 使用Map配置数据时，使用此方法更新
public void updateDataAndItemHeader(Map<IH, List<ID>> map)
// 分页加载更新数据时调用，仅支持使用SectionRule配置数据
public void appendSectionTailRangeData(List<ID> data)
```


## Module
使用Module配置附加功能，目前有HFModule(添加Header和Footer)、LoadMoreModule(预加载更多)
### 添加 Header 和 Footer
- HFModule

```java
// 生成和添加module
HFModule hfModule = new HFModule(mContext,
                R.layout.header_footer_headerly,
                R.layout.header_footer_footerly, mRv);
adapter.addHFModule(hfModule);
// 更改Header 和 Footer的数据,类似数据的配置,你可以在实现的方法里绑定数据和监听事件
adapter = new SimpleRvAdapter<HFModel>(mContext, hfModels, R.layout.header_footer_item) {
            @Override
            public void onBindHeader(BaseViewHolder header) {
                super.onBindHeader(header);
            }

            @Override
            public void onBindFooter(BaseViewHolder footer) {
                super.onBindFooter(footer);
            }
        };
// 当只想添加Header或Footer时，使用常亮HFModule.NO_RES表示没有资源
HFModule hfModule = new HFModule(mContext,HFModule.NO_RES,
                HFModule.NO_RES, mRv);
                
//  隐藏Header 和 Footer
public void setFooterEnable(boolean footerEnable)
public void setHeaderEnable(boolean headerEnable)
```
### 预加载更多
- LoadMoreModule

`new LoadMoreModule(int preLoadNum, OnLoadMoreListener lis)`,preLoadNum表示提前几个Item进行预加载，preLoadNum越大预加载的越提前
加载数据完成之后需要调用`mLoadMoreModule.finishLoad();`结束本次加载，保证下次加载可以生效

```java
		// 触发之后1500秒后加载数据
		LoadMoreModule loadMoreM = new LoadMoreModule(4, new OnLoadMoreListener() {
            @Override
            public void onLoadMore(final LoadMoreModule mLoadMoreModule) {
                new Handler().postDelayed(new Runnable() {
                    @Override
                    public void run() {
                        List<LoadMoreModel> tempData = new ArrayList<LoadMoreModel>();
                        for (int i = 0; i < 9; i++) {
                            tempData.add(new LoadMoreModel("new is " + i));
                        }
                        adapter.appendTailRangeData(tempData, false);
                        mLoadMoreModule.finishLoad();
                    }
                }, 1500);
            }
        });
        adapter.addLoadMoreModule(loadMoreM);
```
## 其他
### adapterId 区分
为了区分不同的适配器，生成了adapterId,用来检测当前RecyclerView使用的是不是这个adapter

```java
public boolean isThisAdapter(RecyclerView rv)
```
### Sample

```java

//内部类实现
quickAdapter = new TypeRvAdapter<Demo>(self, demos) {
            @Override
            public void onBindView(RvViewHolder holder, Demo data, int pos, int type) {
               // 给控件绑定数据,必须实现
            }

            @Override
            public void onBindHeader(RvViewHolder header) {
               //给Header绑定数据和事件,不需要可以不实现
            }

            @Override
            public void onBindFooter(RvViewHolder footer) {
               //给footer绑定数据和事件,不需要可以不实现
            }
        };





//继承实现
public class MyAdapter extends TypeRvAdapter<Demo> {

    public MyAdapter(Context context, List<Demo> data) {
        super(context, data);
    }

    @Override
    public void onBindView(RvViewHolder holder, Demo data, int pos, int type) {
       // 给控件绑定数据,必须实现
    }

    @Override
    public void onBindHeader(RvViewHolder header) {
       //给Header绑定数据和事件,不需要可以不实现
    }

    @Override
    public void onBindFooter(RvViewHolder footer) {
       //给footer绑定数据和事件,不需要可以不实现
    }
}
```