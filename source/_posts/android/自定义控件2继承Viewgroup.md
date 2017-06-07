---
layout: post
title: 自定义控件(2)—继承ViewGroup
date: 2015-11-01
category: Android
tags: [Android]
keywords: 
description: 自定义控件(2)—继承ViewGroup
---

## 前言
- 继承ViewGroup是自定义控件的一般方法，根据需要选择需要继承的ViewGroup的子类，本章将使用纯代码添加控件,方式确实不直观，但是可以用来练习使用代码操作控件的方法，毕竟不是所有的代码都可以用xml文件代替。


## 案例介绍
- 这次的目的实现一个类似这样的组合控件，出现这种需求是因为需要在一个ListView中添加一个类似ListView的部分，但是数量很少，可以不需要使用ListView,而是使用动态添加控件的方式。
![1](http://7xtjec.com1.z0.glb.clouddn.com/purecode_all.png)

![1](http://7xtjec.com1.z0.glb.clouddn.com/purecode.png)


## 代码
- 完全使用代码来写控件，没有xml布局，调试的时候是个大问题，写的代码往往不能实时的显示在预览界面。

 
```java
public class SpecialItemView extends RelativeLayout {

	//分析该布局定义以下组件
	//图片
    private ImageView imgIcon;
    //标题
    private TextView txtTitle;
    //子标题
    private TextView txtSubTitle;
    //专辑数量
    private TextView txtNum;
    //右边更多按钮
    private ImageButton ibArrow;
    //下方横线，因为最后一行是不显示横线的
    private ImageView ivLine;

    /**
     * 代码中new时使用
     *
     * @param context
     */
    public SpecialItemView(Context context) {
        this(context, null);
    }

    /**
     * 在xml文件中使用
     *
     * @param context
     * @param attrs
     */
    public SpecialItemView(Context context, AttributeSet attrs) {
        super(context, attrs);
        init(context, attrs);
    }

    /**
     * 初始化
     *
     * @param context
     * @param attributeSet
     */
    private void init(Context context, AttributeSet attributeSet) {

		//获取屏幕宽度
        int width = context.getResources().getDisplayMetrics().widthPixels;
        //设置当前控件的padding
        this.setPadding(6,10,6,10);
        //初始化控件
        imgIcon = new ImageView(context);
		//LayoutParams对象用来控制控件在ViewGroup中显示的样子
        RelativeLayout.LayoutParams layoutParams =
                new LayoutParams((int) (width*0.2),(int) (width*0.2));
        //垂直居中相当于layout_centervertical=true
        layoutParams.addRule(CENTER_VERTICAL);
        //配置控件
        imgIcon.setScaleType(ImageView.ScaleType.CENTER_CROP);
        imgIcon.setLayoutParams(layoutParams);
        //相当于android:id
        imgIcon.setId(R.id.sp_item_icon);
        imgIcon.setImageResource(R.mipmap.finding_zone_img);
        //添加控件到ViewGroup
        addView(imgIcon);


		//除了可以设置为具体的数值，也可以使用常量，相当于xml文件中使用android：layout_width = "wrap_content"
        layoutParams = new LayoutParams(ViewGroup.LayoutParams.WRAP_CONTENT, ViewGroup.LayoutParams.WRAP_CONTENT);
       //左边margin
        layoutParams.leftMargin = 20;
        //相当于android:aligntop="R.id.sp_item_icon"
        layoutParams.addRule(ALIGN_TOP, R.id.sp_item_icon);
        layoutParams.addRule(RIGHT_OF, R.id.sp_item_icon);
        //以下是textview的设置
        txtTitle.setText("标题");
        txtTitle = new TextView(context);
        txtTitle.setId(R.id.sp_item_title);
        txtTitle.setTextSize(TypedValue.COMPLEX_UNIT_SP,18);
        txtTitle.setTextColor(Color.BLACK);
        txtTitle.setLayoutParams(layoutParams);
        txtTitle.setSingleLine();
        txtTitle.setEllipsize(TextUtils.TruncateAt.END);
        addView(txtTitle);


        txtSubTitle = new TextView(context);
        txtSubTitle.setId(R.id.sp_item_subtitle);
        layoutParams = new LayoutParams(ViewGroup.LayoutParams.WRAP_CONTENT, ViewGroup.LayoutParams.WRAP_CONTENT);
        //todo sp2dp
        layoutParams.addRule(CENTER_VERTICAL);
        layoutParams.addRule(ALIGN_LEFT, R.id.sp_item_title);
        txtSubTitle.setText("字标题");
        //todo sp2dp
        txtSubTitle.setTextSize(TypedValue.COMPLEX_UNIT_SP, 16);
        txtSubTitle.setTextColor(Color.GRAY);
        txtSubTitle.setLayoutParams(layoutParams);
        txtSubTitle.setPadding(0,0,16,0);
        txtSubTitle.setSingleLine();
        txtSubTitle.setEllipsize(TextUtils.TruncateAt.END);
        addView(txtSubTitle);

        txtNum = new TextView(context);
        txtNum.setId(R.id.sp_item_subtitle);
        layoutParams = new LayoutParams(ViewGroup.LayoutParams.WRAP_CONTENT, ViewGroup.LayoutParams.WRAP_CONTENT);
        //todo sp2dp
        layoutParams.addRule(ALIGN_LEFT, R.id.sp_item_title);
        layoutParams.addRule(ALIGN_BOTTOM, R.id.sp_item_icon);
        txtNum.setText("共有几张专辑");
        //todo sp2dp
        txtNum.setTextSize(TypedValue.COMPLEX_UNIT_SP, 16);
        txtNum.setTextColor(Color.GRAY);
        txtNum.setLayoutParams(layoutParams);
        txtNum.setSingleLine();
        txtNum.setEllipsize(TextUtils.TruncateAt.END);
        txtNum.setGravity(CENTER_VERTICAL);
        //图标必须经过setbounds
        Drawable leftD = ContextCompat.getDrawable(getContext(), R.mipmap.finding_album_img);
        leftD.setBounds(0, 0, 20, 0);
        txtNum.setCompoundDrawablesWithIntrinsicBounds(leftD, null, null, null);
        txtNum.setId(R.id.sp_item_num);
        addView(txtNum);


        ibArrow = new ImageButton(context);
        ibArrow.setId(R.id.sp_item_subtitle);
        layoutParams = new LayoutParams(ViewGroup.LayoutParams.WRAP_CONTENT, ViewGroup.LayoutParams.WRAP_CONTENT);
        //todo sp2dp
        layoutParams.addRule(CENTER_VERTICAL);
        layoutParams.addRule(ALIGN_PARENT_RIGHT);

        //todo sp2dp
        layoutParams.rightMargin = 16;
        ibArrow.setBackgroundResource(R.drawable.selector_more);
   
        //todo sp2dp
        ibArrow.setLayoutParams(layoutParams);
        addView(ibArrow);


        ivLine = new ImageView(context);
        layoutParams = new LayoutParams(ViewGroup.LayoutParams.MATCH_PARENT,1);
        layoutParams.addRule(ALIGN_LEFT,R.id.sp_item_title);
        layoutParams.addRule(BELOW,R.id.sp_item_num);
        //todo sp2dp
        layoutParams.topMargin=10;
        ivLine.setLayoutParams(layoutParams);
        ivLine.setBackgroundResource(R.drawable.shape_line);
        addView(ivLine);
    }

    public void setTitle(String txt){
        txtTitle.setText(txt==null?"":txt);
    }
    public void setSubTitle(String txt){
        txtSubTitle.setText(txt==null?"":txt);
    }
    public void setNum(String txt){
        txtNum.setText(txt==null?"":txt);
    }
    public void setShowLine(int xx){
        ivLine.setVisibility(xx);
    }
    public void setImg(String url){
        Picasso.with(getContext()).load(url).into(imgIcon);
    }

    public ImageView getImgIcon(){
        return imgIcon;
    }

   

    public ImageButton getImgArrow(){
        return ibArrow;
    }
    private OnMoreClick onMoreClick;
    public 
    //使用接口将ibArrow点击事件传递出去
    public interface OnMoreClick{
        void click();
    }
}
```




## 在xml文件中使用
```xml
<com.march.himalayasfm.app.widgets.SpecialItemView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            />
```