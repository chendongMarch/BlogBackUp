---
layout: post
title: Android多类型抽象适配
date: 2015-11-15
category: Android
tags: [Android,Adapter]
keywords: 
description: Android多类型抽象适配
---


## 前言
- 上一篇文章介绍了如何进行[分类适配](http://blog.csdn.net/chendong_/article/details/48814455)，看过的小伙伴可以虽然比较完美的实现了分类适配以及复用，但是代码相当繁琐，我们有两种类型时已经出现了多层if嵌套，如果有三四种类型，估计自己都要转晕了，而且就像我们对[单类型适配器抽象](http://blog.csdn.net/chendong_/article/details/48812405)时做的，避免重复代码！如
果没看过上一篇，建议浏览一下，这里写的很多都是基于第一篇的。

## 抽象ViewHolder

```
public static class MultiViewHolder extends ViewHolder{
		/**
		 * SparseArray
		 */
		private SparseArray<View> cacheViews;
		private View itemView;

		public MultiViewHolder(View itemView, int viewCount) {
			super();
			this.itemView = itemView;
			cacheViews = new SparseArray<View>(viewCount);
		}

		@Override
		public View getView(int resId) {
			View v = cacheViews.get(resId);
			if (v == null) {
				v = itemView.findViewById(resId);
				if (v != null) {
					cacheViews.put(resId, v);
				}
			}
			return v;
		}
}
```


## 使用接口获取对象的类型
- 我们在分类适配的时候不可避免的要获得数据的对象，以此作为根据装载不同的布局文件.

```
/**
 1. 接口，分类适配的对象需要实现的接口，目的是约束实体类实现getType方法
 2. 
 3. @author chendong
 4. 
 */
public interface MultiEasyAdapterInterface {
	public int getType();
}
```

## 抽象分类适配器

 1. 我们需要避免在子类中避免重复编码，同时在子类实现自己的方法，使用抽象父类。
  
 2. 我们不了解传递进来的数据是什么类型，使用泛型。

 3. 我们需要限制传进来的数据必须实现获得获得子类的方法，所以必须要求数据实现MultiEasyAdapterInterface接口，使用泛型限定符确定上限。
 

```
public abstract class MultiEasyAdapter<T extends MultiEasyAdapterInterface>
		extends BaseAdapter {}
```
## 成员变量
```
private LayoutInflater layoutInflater;
private List<T> datas;
```

- 分析一下其他成员，根据[上一篇传统分类适配](http://blog.csdn.net/chendong_/article/details/48814455)的写法，我们姑且忽略类型的差异

 1. 每个类型需要一个布局资源文件id
 2. 每个类型布局文件中字UI控件的个数（用来优化）
 3. 每个类型一个唯一的键值（用来解决复用Item空指针的问题）
 4. 我们用一个实体类存储这些信息
 

```
**
 * 存储类型信息的实体类
 *
 * @author chendong
 * @功能：分类适配器配置信息实体类
 */
public class MultiEasyAdapterEntity {

	/**
	 * @param type
	 *            item类型，int类型变量，Item是什么类型的就填写什么类型
	 * @param resId
	 *            资源id，对应类型的资源id，你需要装载的资源文件的ID
	 * @param viewCount
	 *            资源文件中对应的需要获取的视图的个数
	 */
	public MultiEasyAdapterEntity(int type, int resId, int viewCount) {
		super();
		this.resId = resId;
		this.viewCount = viewCount;
	}

	private int type;
	private int resId;
	private int viewCount;

	public MultiEasyAdapterEntity() {
		super();
	}
}
```
- 可能有点繁琐，type和viewCount 可以省略掉，加入type是为了更灵活的获得数据类型，viewCount则是为了优化SparseArray,综上第三个成员变量，就是使用type作为键，MultiEasyAdapterEntity作为值的一个SparseArray，他的作用就是存储不同类型的数据适配时需要的配置信息，有点类似配置文件的意思。

```
private SparseArray<MultiEasyAdapterEntity> Res4Type;
```
## 完善代码
- 结合分类适配的方法，变量我们已经存储到了SparseArray中，所以代码就很清晰了

```
/**
 * version 2<br/>
注意事项：类型数为n时，那么定义的类型必须在0-n-1之间，这是使用listview自带缓存的要求。
 * 抽象适配器升级版，可以进行分类适配，使用了模板方法模式，将设置item显示内容的部分抽象到了类外<br/>
 * @param <T> <br/>必须实现MultiEasyAdapterInterface接口{@link MultiEasyAdapterInterface}<br/>
 * @author chendong
 */
public abstract class MultiEasyAdapter<T extends MultiEasyAdapterInterface>
        extends BaseAdapter {
    private LayoutInflater layoutInflater;
    private List<T> datas;
    private SparseArray<MultiEasyAdapterEntity> Res4Type;
    private Context context;

    /**
     * @param context  上下文对象
     * @param datas    数据集
     * @param Res4Type 资源配置文件{@link MultiEasyAdapterEntity}this is like a config entity
     */
    public MultiEasyAdapter(Context context, List<T> datas,
                            SparseArray<MultiEasyAdapterEntity> Res4Type) {
        super();
        this.layoutInflater = LayoutInflater.from(context);
        this.datas = datas;
        this.Res4Type = Res4Type;
        this.context = context;
    }


    protected Context getContext(){
        return context;
    }
    protected List<T> getDatas(){
        return datas;
    }
    public void swapData(List<T> datas){
        this.datas = datas;
        notifyDataSetChanged();
    }

    @Override
    public int getViewTypeCount() {
        return Res4Type.size();
    }

    @Override
    public int getItemViewType(int position) {
        return datas.get(position).getType();
    }

    public int getCount() {
        return datas.size();
    }

    public Object getItem(int position) {
        return datas.get(position);
    }

    public long getItemId(int position) {
        return position;
    }

    public View getView(int position, View convertView, ViewGroup parent) {
        ViewHolder holder = null;
        /* get the type*/
        int type = datas.get(position).getType();
        if (convertView == null) {
            int resId = Res4Type.get(type).getResId();
            convertView = layoutInflater.inflate(resId, parent, false);
            holder = new ViewHolder(convertView, Res4Type.get(type)
                    .getViewCount());
            convertView.setTag(holder);
            bindListener4View(holder,datas.get(position), type, position);
        } else {
            holder = (ViewHolder) convertView.getTag();
        }
        bindData4View(holder,datas.get(position), type, position);
        return convertView;
    }

    /**
     * 绑定数据
     * bind data
     *
     * @param holder the viewholder
     * @param type data's type
     * @param data data
     */
    public abstract void bindData4View(ViewHolder holder, T data, int type,int pos);

    /**
     * 绑定监听
     * bind listener
     *
     * @param holder the viewholder
     * @param type data's type
     * @param pos position
     */
    public abstract void bindListener4View(ViewHolder holder, T data, int type,int pos);
}


```

## 测试

```
listview = (ListView) findViewById(R.id.listview);
		list = new ArrayList<Student>();
		for (int i = 0; i < 20; i++) {
			if (i % 5 == 0)
				list.add(new Student("name" + i, "age" + i, "sex", 3));
			else
				list.add(new Student("name" + i, "age" + i, "sex",
						i % 2 == 0 ? 1 : 2));
		}
		SparseArray<MultiEasyAdapterEntity> sparseArray = new SparseArray<MultiEasyAdapterEntity>(
				3);
		sparseArray.put(1, new MultiEasyAdapterEntity(1, R.layout.item_type1,
				4));
		sparseArray.put(2, new MultiEasyAdapterEntity(2, R.layout.item_type2,
				4));
		sparseArray.put(3, new MultiEasyAdapterEntity(3, R.layout.item_type3,
				4));
		listview.setAdapter(new MultiEasyAdapter<Student>(
				getApplicationContext(), list, sparseArray) {

			@Override
			public void bindData4View(ViewHolder holder,
					Student data, int type) {
				((TextView) holder.getView(R.id.tv_name)).setText(data
						.getName());
				((TextView) holder.getView(R.id.tv_sex)).setText(data.getSex());
				((TextView) holder.getView(R.id.tv_age)).setText(data.getAge()
						+ "");
				if (type == 3)
					((TextView) holder.getView(R.id.tv_type)).setText(data
							.getType()+"");
			}
@Override
			public void bindListener4View(ViewHolder holder,
					Student data, int type) {
			//绑定监听事件
			}
		});
```


## 效果
 - 大家可以看到三中布局适配的没有问题，只有蓝色背景的布局显示了type,他是类型3的数据
![图片](http://img.blog.csdn.net/20150930121432551)
		