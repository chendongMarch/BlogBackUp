---
layout: post
title: EditText
categories:
  - Android
  - View
tags: 
  - Android
  - View
keywords:
  - Android
  - EditText
abbrlink: 2515477621
date: 2016-10-31 00:00:00
---

`EditText` 是 `TextView` 的子类，用于进行文字输入等操作，是开发中特别常用的一个控件了，本文主要介绍：  

1. 更改EditText的背景  
2. 设置监听事件
3. 给EditText设置具有带图片的Hint

<!--more-->

## 2. 监听事件
### 2.1 监听焦点变化
```java
etInput.setOnFocusChangeListener(new OnFocusChangeListener() {
    @Override
    public void onFocusChange(View v, boolean hasFocus) {
        // hasFocus 当前是否获得焦点
    }
});
```

### 2.2 文本输入监听事件
```java
etInput.addTextChangedListener(new TextWatcher() {
    @Override
    public void beforeTextChanged(CharSequence s, int start, int count, int after) {
    	// 文本输入之前回调
    }
    @Override
    public void onTextChanged(CharSequence s, int start, int before, int count) {
    	//  输入过程中会频繁回调
    }
    @Override
    public void afterTextChanged(Editable s) {
    	// 输入结束回调
    }
});
```


## 3. 自定义EditText背景

如下图所示，当离开当前EditText时，检查输入是否有问题，输入错误会显示红色

<img src="http://7xtjec.com1.z0.glb.clouddn.com/et_bg_change.jpg"
width ="50%" align=center />


### 3.1 定义背景drawable文件
其他形态的背景也相对简单，只有一条线的这种背景drawable写起来还是需要研究一下的，直接上代码啦,主要是用了一个`selector -> layer-list -> shape`,在`select`属性改变时，改变线条的颜色。原本是一个矩形，然后将另外三个边偏移一下，只留下底边，达到线条显示的效果。

```xml
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:state_selected="true">
        <layer-list xmlns:android="http://schemas.android.com/apk/res/android">
            <item>
                <shape android:shape="rectangle">
                    <solid android:color="@android:color/transparent" />
                </shape>
            </item>
            <item
                android:left="-2dip"
                android:right="-2dip"
                android:top="-2dip">
                <shape>
                    <solid android:color="@android:color/transparent" />
                    <stroke
                        android:width="1dip"
                        android:color="@color/red"
                        android:dashGap="0dp"
                        android:dashWidth="0dip" />
                </shape>
            </item>
        </layer-list>
    </item>


    <item>
        <layer-list xmlns:android="http://schemas.android.com/apk/res/android">
            <item>
                <shape android:shape="rectangle">
                    <solid android:color="@android:color/transparent" />
                </shape>
            </item>
            <item
                android:left="-2dip"
                android:right="-2dip"
                android:top="-2dip">
                <shape>
                    <solid android:color="@android:color/transparent" />
                    <stroke
                        android:width="1dip"
                        android:color="@color/colorPrimary"
                        android:dashGap="0dp"
                        android:dashWidth="0dip" />
                </shape>
            </item>
        </layer-list>
    </item>
</selector>
```


### 3.2 焦点改变时改变背景
```
etInput.setOnFocusChangeListener(new OnFocusChangeListener() {
                @Override
                public void onFocusChange(View v, boolean hasFocus) {
                    String trim = etInput.getText().toString().trim();
                    // 没有焦点&&有文字
                    if (!hasFocus && trim.length() > 0) {
                       etInput.setSelected(true);
                    } else {
                       etInput.setSelected(false);
                    }
                }
            });
```


## 4. 带图片的Hint
如下图所示，实现带有图片的hint，使用 `SpannableString`实现 ，但是开始的效果不是很理想，图片不能很好的居中，使用重写`ImageSpan`的方法解决了这个问题
![](http://olx4t2q6z.bkt.clouddn.com/edittext_img_hint.jpeg)

### 4.1 图片垂直居中的span
```java
class VerticalCenterImageSpan extends ImageSpan {
        VerticalCenterImageSpan(Drawable drawable) {
            super(drawable);
        }

        public int getSize(Paint paint, CharSequence text, int start, int end,
                           Paint.FontMetricsInt fontMetricsInt) {
            Drawable drawable = getDrawable();
            Rect rect = drawable.getBounds();
            if (fontMetricsInt != null) {
                Paint.FontMetricsInt fmPaint = paint.getFontMetricsInt();
                int fontHeight = fmPaint.bottom - fmPaint.top;// 计算文字的高度
                int drHeight = rect.bottom - rect.top;// 计算图片的高度
                int top = drHeight / 2 - fontHeight / 4;
                int bottom = drHeight / 2 + fontHeight / 4;
                fontMetricsInt.ascent = -bottom;
                fontMetricsInt.top = -bottom;
                fontMetricsInt.bottom = top;
                fontMetricsInt.descent = top;
            }
            return rect.right;
        }

        @Override
        public void draw(Canvas canvas, CharSequence text, int start, int end,
                         float x, int top, int y, int bottom, Paint paint) {
            Drawable drawable = getDrawable();
            canvas.save();
            int transY;
            transY = ((bottom - top) - drawable.getBounds().bottom) / 2 + top;
            canvas.translate(x, transY);
            drawable.draw(canvas);
            canvas.restore();
        }
    }
```

### 4.2 设置hint
```java
// 设置图文hint
    private void initEditText(EditText editText) {
        SpannableString msp = new SpannableString("图片 写点什么吧...");
        Drawable drawable = ContextCompat.getDrawable(getContext(), R.drawable.icon_hint_edit);
        drawable.setBounds(0, 0, drawable.getIntrinsicWidth(), drawable.getIntrinsicHeight());
        // 这里从start = 0开始，end = 2结束，图片取代了两个字的位置，上面的字符串中国年的图片两个字会被替代
        msp.setSpan(new VerticalCenterImageSpan(drawable), 0, 2, Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);
        editText.setHint(msp);
    }
```

## 5 修改键盘的回车键
在xml配置

```xml
android:imeOptions="actionSearch"
android:inputType="text"
```
或在java代码中配置

```java
editText.setImeOptions(EditorInfo.IME_ACTION_SEARCH);
editText.setInputType(EditorInfo.TYPE_CLASS_TEXT);
```
实现监听，进行相关操作

```java
editText.setOnEditorActionListener(new TextView.OnEditorActionListener() {
    public boolean onEditorAction(TextView v, int actionId, KeyEvent event) {
        if (actionId == EditorInfo.IME_ACTION_SEND 
                || (event != null && event.getKeyCode() == KeyEvent.KEYCODE_ENTER)) {
            //do something;                
            return true;
        }
        return false;
    }
});
```