---
layout: post
title: Android单类型抽象适配器
date: 2015-11-14
category: Android
tags: [Android,Adapter]
keywords: 
description: Android单类型抽象适配器
---



## 前言
- 介绍一下适配器的抽象，在我们做项目的时候会有很多很多地方使用ListView，也就意味着需要写很多很多的适配器，当我们写的项目很大时就会很烦很烦，每次都要写同样的代码片实现类似的功能，所以我们就有必要对传统的适配器抽象一下。

## 传统适配器的优化写法
- 核心代码就是getVIew()方法，在里面我们进行Item的复用，而相对于其他方法就会显得很多余，因为每个适配器都在重复相同的代码



```
public class FirstpageGridAdapter extends BaseAdapter {
	/* gridview数据 */
	private String[] choices = new String[] { "新房", "二手房", "租房", "资讯", "打折优惠",
			"最新开盘", "房贷计算", "更多" };
	private int[] images = new int[] { R.drawable.selector_xinfang,
			R.drawable.selector_ershou, R.drawable.selector_zufang,
			R.drawable.selector_zixun, R.drawable.selector_youhui,
			R.drawable.selector_kaipan, R.drawable.selector_calculator,
			R.drawable.selector_more };
	private LayoutInflater layoutInflater;
	private onClickChildIbListener listener;

	public FirstpageGridAdapter(Context context, onClickChildIbListener listener) {
		super();
		this.layoutInflater = LayoutInflater.from(context);
		this.listener = listener;
	}

	public interface onClickChildIbListener {
		public void clickChild(int pos,View view);
	}

	public int getCount() {
		return choices.length;
	}

	public Object getItem(int position) {
		return choices[position];
	}

	public long getItemId(int position) {
		return position;
	}

	public View getView(int position, View convertView, ViewGroup parent) {
		ViewHolder holder = null;
		if (convertView == null) {
			holder = new ViewHolder();
			convertView = layoutInflater.inflate(R.layout.item_main_grid,
					parent, false);
			holder.ib = (ImageButton) convertView
					.findViewById(R.id.item_main_grid_ib);
			holder.tv = (TextView) convertView
					.findViewById(R.id.item_main_grid_tv);
			convertView.setTag(holder);
		} else {
			holder = (ViewHolder) convertView.getTag();
		}
		holder.ib.setBackgroundResource(images[position]);
		holder.tv.setText(choices[position]);
		final int pos = position;
		holder.ib.setOnClickListener(new OnClickListener() {
			public void onClick(View v) {
				listener.clickChild(pos,v);
			}
		});
		return convertView;
	}

	private class ViewHolder {
		ImageButton ib;
		TextView tv;
	}
}

```

### 抽象ViewHolder

- 我们可以分析一下ViewHolder类，它维护一个Item的一组UI组件，如果我们要使用使用通用的ViewHolder就会遇到一个问题，你不知道不同的布局有什么组件在里面，如何维护一组UI呢，数组？链表？或者Map?当查找View时我们使用id来进行查找，那么想在ViewHolder中查找View就需要使用id作为键，我们选择使用SparseArray，也就是稀疏数组，什么是稀疏数组，使用id（大整数）作为键值存储数据会造成大多数的未被使用，如果使用一般的表存储会造成很大的浪费，稀疏数组对数组进行了压缩，节约了很大空间，详见[这里](http://blog.csdn.net/dzweather/article/details/8057834)


```
private SparseArray<View> cacheViews;
```

- 使用稀疏数组存储UI控件以后我们需要一个获取UI控件的方法，方法很清晰的，itemView是传递进来的父控件，就是convertView从中根据id获取控件

```
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
```

- ViewHolder完整代码，这里传入了一个viewCount,为什么呢？就像ArrayList一样，不定义空间大小时初始容量16，超出空间大小时每次增加25，但就是初始容量的16对于我们来说已经是浪费了，一个Item不可能有16个控件那么多。指定大小提高内存使用。

```
/**
	 * 
	 * @author chendong
	 * 用来实现复用加载的单类型ViewHolder
	 *
	 */
	public static class SingleViewHolder{
		/**
		 * 使用SparseArray
		 */
		private SparseArray<View> cacheViews;
		private View itemView;

		public SingleViewHolder(View itemView, int viewCount) {
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

## 抽象适配器
- 传统的适配器有太多的重复代码需要编写，我们可以把重复的代码在父类中编写好，使得子类可以直接复用，！使用抽象父类
- 父类的getView()方法中使用了一个抽象方法，使用了一个设计模式模板方法模式，将方法的实现推迟到了子类中
- 泛型，传递的数据类型是不确定的，使用泛型可以解决这个问题，泛型的使用不了解的建议搜一下，后面深入的介绍会使用更复杂的泛型

```
/**
 1. 抽象适配器，使用了模板方法模式，将设置item显示内容的部分抽象到了类外 这是单类型的抽象适配
 2. 
 3. @author chendong
 4. 
 5. @param <T>
 6.            泛型
 */
public abstract class SingleEasyAdapter<T> extends BaseAdapter {
	private LayoutInflater layoutInflater;
	private int resId;
	private List<T> datas;
	private int viewCount = 5;

	/**
	 * @param context
	 *            上下文对象，建议使用getApplicationContext();
	 * @param resId
	 *            item布局id
	 * @param datas
	 *            数据集
	 * @param viewCount
	 *            item中的view个数，用来优化SparseArray<View>
	 */
	public SingleEasyAdapter(Context context, int resId, List<T> datas,
			int viewCount) {
		super();
		this.layoutInflater = LayoutInflater.from(context);
		this.resId = resId;
		this.datas = datas;
		this.viewCount = viewCount;
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
        if (convertView == null) {
            convertView = layoutInflater.inflate(resId, parent, false);
            holder = new ViewHolder(convertView, viewCount);
            convertView.setTag(holder);
            //在这里绑定监听，避免重复绑定。
            bindListener4View(holder, datas.get(position), position);
        } else {
            holder = (ViewHolder) convertView.getTag();
        }
        bindData4View(holder, datas.get(position), position);
        return convertView;
    }

    /**
     * 绑定数据
     *
     * @param holder
     * @param data
     */
    public abstract void bindData4View(ViewHolder holder, T data, int pos);

    /**
     * 绑定监听
     *
     * @param holder
     * @param pos
     */
    public abstract void bindListener4View(ViewHolder holder, T data, int pos);
	}
```


## 完整代码

```

/**
 * 抽象适配器，使用了模板方法模式，将设置item显示内容的部分抽象到了类外 这是单类型的抽象适配
 *
 * @param <T> 泛型
 * @author chendong
 */
public abstract class SingleEasyAdapter<T> extends BaseAdapter {
    private LayoutInflater layoutInflater;
    private int resId;
    private List<T> datas;
    private int viewCount = 5;
    private Context context;


    /**
     * @param context   上下文对象，建议使用getApplicationContext();
     * @param resId     item布局id
     * @param datas     数据集
     * @param viewCount item中的view个数，用来优化SparseArray<View>
     */
    public SingleEasyAdapter(Context context, int resId, List<T> datas,
                             int viewCount) {
        super();
        this.layoutInflater = LayoutInflater.from(context);
        this.resId = resId;
        this.datas = datas;
        this.viewCount = viewCount;
        this.context = context;
    }

    public Context getContext() {
        return this.context;
    }

    public List<T> getData() {
        return datas;
    }

    public void swapData(List<T> datas) {
        this.datas = datas;
        notifyDataSetChanged();
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
        if (convertView == null) {
            convertView = layoutInflater.inflate(resId, parent, false);
            holder = new ViewHolder(convertView, viewCount);
            convertView.setTag(holder);
            //在这里绑定监听
            bindListener4View(holder, datas.get(position), position);
        } else {
            holder = (ViewHolder) convertView.getTag();
        }
        bindData4View(holder, datas.get(position), position);
        return convertView;
    }

    /**
     * 绑定数据
     *
     * @param holder
     * @param data
     */
    public abstract void bindData4View(ViewHolder holder, T data, int pos);

    /**
     * 绑定监听
     *
     * @param holder
     * @param pos
     */
    public abstract void bindListener4View(ViewHolder holder, T data, int pos);
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

		listview.setAdapter(new SingleEasyAdapter<Student>(
				getApplicationContext(), R.layout.item_type1, list, 4) {

			@Override
			public void bindData4View(SingleViewHolder holder, Student data，int pos) {
				((TextView)holder.getView(R.id.tv_name)).setText(data.getName());
				//此处省略若干代码
			}
@Override
			public void bindListener4View(SingleViewHolder holder, Student data，int pos) {
				//此处省略若干代码
			}

		});
```
## 效果
- 可以看到我们只用了很少的代码就完成了功能，而且很清晰，简单地适配不需要创建适配器类了，使用匿名的也很快就可以实现。演示有点丑，基本都实现了。
![这里写图片描述](http://img.blog.csdn.net/20150929220700791)

## 总结
- 将适配器抽象出来作为一个类库，再使用的时候就会简单很多很多，当然如果你的适配器数据很复杂，那么也可以继承抽象父类生成自己的类。!
