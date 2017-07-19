---
layout: post
title: 自定义控件九宫格滑动解锁
categories:
  - Android
  - View
tags:
  - Android
  - View
keywords:
  - Android
  - 自定义控件
  - 九宫格滑动解锁
abbrlink: 2317215752
date: 2015-08-23 00:00:00
---

## 1. 前言
- 最近想给自己做的的app添加一个滑动解锁的功能，用的是乐视的手机，就模仿它的效果实现.

- [视频演示一下效果](http://39.130.133.37/xdispatch/7xtjec.com1.z0.glb.clouddn.com/lockview.mp4)

- [GitHub](https://github.com/chendongMarch/CommonLib/blob/master/lib-view/src/main/java/com/march/lib/view/LockView.java)


## 2. LockPoint实体
- 每个点是一个实体（LockPoint）用来存储这个点的所有信息，包括点的物理位置(x,y)和点的index位置(0-8)

```java
class LockPoint {
        // 点的位置 0-8
        int index;
        // 点的x,y坐标
        float x, y;
        // 构造方法，初始化一个点
        LockPoint(int index, float x, float y) {
            this.index = index;
            this.x = x;
            this.y = y;
        }
        // 构造方法，从另一个点初始化
        LockPoint(LockPoint p) {
            this.x = p.x;
            this.y = p.y;
            this.index = p.index;
        }
        // 默认构造方法，初始化为一个空的点
        LockPoint() {
            this.x = -1;
            this.y = -1;
            this.index = -1;
        }
        // 判断该点是不是一个空的点
        boolean isEmpty() {
            return this.x == -1 && this.y == -1;
        }
        // 重新给位置赋值
        void init(float x, float y) {
            this.x = x;
            this.y = y;
        }
        // 设置为另一点的值
        void init(LockPoint p) {
            this.x = p.x;
            this.y = p.y;
            this.index = p.index;
        }
        // 判断一个位置是不是在该点触摸范围内,touchSensitiveRange为触摸有效半径
        boolean isTouchIn(float judgeX, float judgeY) {
            return judgeX < x + touchSensitiveRange &&
                    judgeX > x - touchSensitiveRange &&
                    judgeY < y + touchSensitiveRange &&
                    judgeY > y - touchSensitiveRange;
        }

        // 重写equals和hashCode
        @Override
        public boolean equals(Object o) {
            LockPoint p = (LockPoint) o;
            return p.x == x && p.y == y;
        }

        @Override
        public int hashCode() {
            return 2;
        }

        String out(String tag) {
            return tag + " : x = " + x + " , y = " + y;
        }
    }
```


## 3. 初始化
- 初始化九个点的位置，需要根据控件的大小动态计算，因此在onMeare()之后进行
- 需求是需要将九个点放在控件中间，来适应控件大小的变化，首先确定第一个点距离左边的距离startSpace，两个点之间的距离 =（控件宽度 - 2 * startSpace）／2

```java
		int size = getMeasuredWidth();
        // 将宽高设置为一样的，正方形
        setMeasuredDimension(size, size);
        // 初始化屏幕中的九个点的位置和下标
        initLockPointArray = new LockPoint[9];
        // startSpace 为距离左边的距离，计算九个点的位置,保证九个点放在控件中间
        if (startSpace == AUTO_START_SPACING) {
        	//默认是控件的1/4
            startSpace = size / 4;
        }
        // 计算每两个点之间的间隔
        internalSpace = (size - 2 * startSpace) / 2;
```
- 初始化九个点的位置

```java
			// 初始化九个点的位置
            int index = 0;
            for (int i = 0; i < 3; i++) {
                for (int j = 0; j < 3; j++) {
                    initLockPointArray[index] = new LockPoint(index, startSpace + j * internalSpace, startSpace + i * internalSpace);
                    index++;
                }
            }
```
- onMeasure()完整代码

```java
// onMeasure之后初始化数据
    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        int size = getMeasuredWidth();
        // 将宽高设置为一样的，正方形
        setMeasuredDimension(size, size);
        // 初始化屏幕中的九个点的位置和下标
        if (initLockPointArray == null) {
            initLockPointArray = new LockPoint[9];
            // startSpace 为距离左边的距离，计算九个点的位置放在控件中间
            if (startSpace == AUTO_START_SPACING) {
                startSpace = size / 4;
            }
            // 计算每两个点之间的间隔
            internalSpace = (size - 2 * startSpace) / 2;
            // 初始化九个点的位置
            int index = 0;
            for (int i = 0; i < 3; i++) {
                for (int j = 0; j < 3; j++) {
                    initLockPointArray[index] = new LockPoint(index, startSpace + j * internalSpace, startSpace + i * internalSpace);
                    index++;
                }
            }
            // 为了在preview时能看到效果
            if (isInEditMode()) {
                historyPointList.addAll(Arrays.asList(initLockPointArray));
            }
        }
    }
```

## 4. onDraw
- 绘制过程大致分为三个步骤

- <1> 绘制九个点，这是每次都需要绘制的 

```java
        LockPoint tempPoint;
        for (int i = 0; i < initLockPointArray.length; i++) {
            canvas.drawCircle(initLockPointArray[i].x, initLockPointArray[i].y, pointRadius, pointPaint);
        }
```

-  <2> 绘制已经划过的点

```java
		// 绘制之前触过存储起来的的点,绘制第i个点和i+1个点之间的线
        if (historyPointList.size() > 0) {
            for (int i = 0; i < historyPointList.size() - 1; i++) {
                canvas.drawLine(historyPointList.get(i).x, historyPointList.get(i).y, historyPointList.get(i + 1).x, historyPointList.get(i + 1).y, linePaint);
            }
        }
```

-  <3> 绘制触摸点和最后一个点的连线

```java
		// 画最后一个点和触摸的点之间的线
        if (currentLockPoint != null
                && currentLockPoint.x != -1 && currentLockPoint.y != -1
                && touchPoint.x != -1 && touchPoint.y != -1) {
            canvas.drawLine(currentLockPoint.x, currentLockPoint.y, touchPoint.x, touchPoint.y, linePaint);
        }
```


## 5. 事件处理
- 对用户touch事件进行处理
	1. 要记录当前触摸的点，用于绘制跟随手指的连线
	2. 检测触摸的点是不是在九个点中某个点的范围内，如果是的话该点要加入被触摸点的列表中
	3. 当手指抬起时，清除数据,恢复初始状态

```java
	@Override
    public boolean onTouchEvent(MotionEvent event) {

        if (!isEnabled() || isEventOver)
            return false;

        int action = MotionEventCompat.getActionMasked(event);
        switch (action) {
            // 重新初始化触摸点
            case MotionEvent.ACTION_DOWN:
                touchPoint.init(event.getX(), event.getY());
                break;
            // 移动时检测是否在触摸范围内
            case MotionEvent.ACTION_MOVE:
                touchPoint.init(event.getX(), event.getY());
                LockPoint tempPoint;
                for (int i = 0; i < initLockPointArray.length; i++) {
                    tempPoint = initLockPointArray[i];
                    if (!historyPointList.contains(tempPoint)
                            && tempPoint.isTouchIn(event.getX(), event.getY())) {
                        historyPointList.add(new LockPoint(tempPoint));
                        currentLockPoint.init(tempPoint);
                        break;
                    }
                }
                break;
            // 抬起时结束，重新初始化
            case MotionEvent.ACTION_UP:
            case MotionEvent.ACTION_CANCEL:
                touchPoint.init(-1, -1);
                currentLockPoint.init(-1, -1);
                historyPointList.clear();
                break;
        }
        postInvalidate();
        return true;
    }
```

## 6. 优化-多点触控事件处理
- 用户在触摸屏幕时可能有多个手指在操作，上面的代码在单指时没有问题，兼容多点触控的思路是：
	1. 当用户触发down事件时，我们可以获取到一个pointerId，这个id唯一的标志了这个指头，后面发生的所有事件都使用用这个pointerId来获取，只处理这个指头的事件，避免事件的错乱。
	2. 当我们开始的时候标志的那个手指抬起来了怎么办呢，两个解决方法，第一个就是直接结束整个流程，相当于单指时手指抬起。第二个方法就是转移事件，当一个指头抬起时，从该事件中获取还没抬起的手指，更改标志的pointerId,事件就转移到了另一个手指上，我们关心就是新手指的触摸啦
	3. 关于对于事件进行处理的相关机制可以看[Android事件机制，](http://blog.csdn.net/chendong_/article/details/53382175)写的都是比较基本的东西，后面慢慢完善，不过理解获取多指的事件9⃣️绰绰有余啦
- 话不多说，上代码，比较需要注意的地方我都标注在注释中，方便查找

```java
	// 处理触摸事件，支持多点触摸
    @Override
    public boolean onTouchEvent(MotionEvent event) {
        // fast stop
        if (!isEnabled() || isEventOver)
            return false;
        // pointerIndex 是事件的在event中的下标
        int pointerIndex;
        // 获取事件掩码
        int action = MotionEventCompat.getActionMasked(event);
        switch (action) {
            // 重新初始化触摸点
            case MotionEvent.ACTION_DOWN:
                // pointerId 记录当前激活的pointerId
                activePointerId = event.getPointerId(0);
                // 根据pointerId查找事件在event中的位置
                pointerIndex = event.findPointerIndex(activePointerId);
                // 根据位置获取到具体的事件的坐标，这里获得的坐标就是我们要记住的那个指头的坐标
                touchPoint.init(event.getX(pointerIndex), event.getY(pointerIndex));
                break;
            case MotionEvent.ACTION_MOVE:
                // 手指移动时还是根据激活的pointerId获取下标index,来进行后续操作，避免事件错乱
                pointerIndex = event.findPointerIndex(activePointerId);
                // pointerIndex < 0表示手指的事件获取不到了，结束响应事件
                if (pointerIndex < 0) {
                    Log.e(TAG, "Got ACTION_MOVE event but have an invalid active pointer id.");
                    cancelLockDraw();
                    return false;
                }
                // 根据移动的位置获取坐标，初始化touchPoint的值
                touchPoint.init(event.getX(pointerIndex), event.getY(pointerIndex));
                LockPoint tempPoint;
                // 检索触摸点有没有在九个点中的某一个的触摸范围内
                for (int i = 0; i < initLockPointArray.length; i++) {
                    tempPoint = initLockPointArray[i];
                    if (!historyPointList.contains(tempPoint)
                            && tempPoint.isTouchIn(event.getX(pointerIndex), event.getY(pointerIndex))) {
                        LockPoint centerPoint = findCenterPoint(tempPoint);
                        // 优化，查找两个点之间的点，后面会有介绍
                        if (!centerPoint.isEmpty()) {
                            activePoint(centerPoint);
                        }
                        activePoint(tempPoint);
                        break;
                    }
                }
                break;
            case MotionEventCompat.ACTION_POINTER_UP:
                // 多指操作中 非 最后一个手指抬起时触发ACTION_POINTER_UP，此时要获取还在屏幕上的其他手指转移事件的对象
                onSecondaryPointerUp(event);
                break;
            case MotionEvent.ACTION_UP:
                // 最后的手指抬起触发 ACTION_UP
                pointerIndex = event.findPointerIndex(activePointerId);
                if (pointerIndex < 0) {
                    Log.e(TAG, "Got ACTION_UP event but don't have an active pointer id.");
                    activePointerId = INVALID_POINTER;
                    return false;
                }
                // 发布绘制的结果，可能是监听回调之类的
                publishResult();
                // 置为-1
                activePointerId = INVALID_POINTER;
                break;
            case MotionEvent.ACTION_CANCEL:
                // 类似up
                cancelLockDraw();
                activePointerId = INVALID_POINTER;
                break;
        }
        postInvalidate();
        return true;
    }
```

- 转移焦点的方法,在各种控件的源代码中随处可见，我也是拷贝出来直接用的，逻辑不是很复杂

```java
	/**
     * 当一个手机抬起时，转移焦点
     *
     * @param ev 事件
     */
private void onSecondaryPointerUp(MotionEvent ev) {
        final int pointerIndex = MotionEventCompat.getActionIndex(ev);
        final int pointerId = MotionEventCompat.getPointerId(ev, pointerIndex);
        if (pointerId == activePointerId) {
            // This was our active pointer going up. Choose a new
            // active pointer and adjust accordingly.
            final int newPointerIndex = pointerIndex == 0 ? 1 : 0;
            activePointerId = MotionEventCompat.getPointerId(ev, newPointerIndex);
        }
    }
```

- 发布结果

```java
	/**
     * 发布绘制结果
     */
    private void publishResult() {
        if (listener != null) {
            isEventOver = true;
            StringBuilder sb = new StringBuilder();
            for (LockPoint lockPoint : historyPointList) {
                sb.append(lockPoint.index);
            }
            String passWd = sb.toString();
            boolean isFinish = listener.onFinish(LockView.this, passWd, passWd.length());
            if (isFinish) {
                // 输入合法
                touchPoint.init(currentLockPoint);
            } else {
                // 输入不合法
                cancelLockDraw();
                isEventOver = false;
            }
        } else {
            cancelLockDraw();
        }
    }
```

- 回复初始状态,因为在多处调用了，贴一下

```java
	/**
     * 结束绘制，恢复初始状态
     */
    private void cancelLockDraw() {
        touchPoint.init(-1, -1);
        currentLockPoint.init(-1, -1);
        historyPointList.clear();
        postInvalidate();
    }
```

## 7. 优化-自动添加两点之间连线上的点
- 当滑动时越过中间的点之间连接两端，自动查找和添加两点之间的点，手机上的滑动解锁也是这样的逻辑，不然会导致图形很繁琐，不美观而且不符合常见逻辑。也就是说如果当前激发的点和上一个激发的点之间有没有激发的点，那么自动给他激发。

- 首先如果两个点是相邻的或者是对角线上相邻，那么中间一定不会有空下来的点，需要排除这个情况

```java
	
	/**
     * 检测相邻
     *
     * @param p1 点1
     * @param p2 点2
     * @return p1和p2是否相邻，斜对角也算相邻
     */
    private boolean isAdjacentPoint(LockPoint p1, LockPoint p2) {
        // internalSpace是初始化时两个点之间的距离，都是简单的计算和情况罗列
        if ((p1.x == p2.x && Math.abs(p1.y - p2.y) == internalSpace)
                || (p1.y == p2.y && Math.abs(p1.x - p2.x) == internalSpace)
                || (Math.abs(p1.x - p2.x) == internalSpace && Math.abs(p1.y - p2.y) == internalSpace)) {
            Log.e(TAG, "相邻点，不处理");
            return true;
        }
        return false;
    }
```

- 然后如何判断一个点位于首尾两个激发点的中间，思路是当这个点在两个点的连线上时且不是首尾两个点就是中间的点。判断的根据是斜率是不是相等，就是初中的数学问题啦。

```java
	/**
     * 判断c点是不是在p1-p2的直线上
     *
     * @param p1 起始点
     * @param p2 终止点
     * @param c  判断的点
     * @return 是否在该线上
     */
    private boolean isInLine(LockPoint p1, LockPoint p2, LockPoint c) {
        float k1 = (p1.x - p2.x) * 1f / (p1.y - p2.y);
        float k2 = (p1.x - c.x) * 1f / (p1.y - c.y);
        return k1 == k2;
    }
```

- 最后整合一下，去掉不必要的判断，在touch事件中调用

```java
	/**
     * 检测当前激活的点和上一个激活点之间的是否有没有激发的点
     *
     * @param activePoint 当前被激发的点
     * @return 当前激活的点和上一个激活点之间的是否有没有激发的点，没有返回empty的{@link LockPoint#isEmpty()}
     */
    private LockPoint findCenterPoint(LockPoint activePoint) {
        LockPoint rstPoint = new LockPoint();
        // 只有一个点不需要比较
        if (historyPointList.size() < 1) {
            return rstPoint;
        }
        LockPoint tempPoint;
        // 获取上个点
        LockPoint preActivePoint = historyPointList.get(historyPointList.size() - 1);
        // 两个点是不是相邻的，是相邻的是坚决不会中间有点被空出来的
        if (isAdjacentPoint(preActivePoint, activePoint))
            return rstPoint;

        for (int i = 0; i < initLockPointArray.length; i++) {
            tempPoint = initLockPointArray[i];
            // 没有被触摸过 && 不是首点 && 不是尾点
            if (!historyPointList.contains(tempPoint) && !preActivePoint.equals(tempPoint) && !activePoint.equals(tempPoint)) {	
            	// 在连线上
                if (isInLine(preActivePoint, activePoint, tempPoint)) {
                    Log.e(TAG, "点在线上 " + tempPoint.out("temp") + " " + preActivePoint.out("pre") + " " + activePoint.out("active"));
                    rstPoint.init(tempPoint);
                    break;
                }
            }
        }
        return rstPoint;
    }
```
- 在onTouchEvent中调用

```java
		LockPoint centerPoint = findCenterPoint(tempPoint);
      	// 优化，查找两个点之间的点
      	if (!centerPoint.isEmpty()) {
           activePoint(centerPoint);
      	}
	    activePoint(tempPoint);
```


## 8. 优化-给被触摸的点添加动画
- 当手指触摸到一个点时，添加一个缩放动画来反馈触摸操作

- 思路时，当触摸到一个点时使用ValueAnimator开启动画，不断改变半径的值，在绘制时达到实现缩放的效果

```java
	/**
     * 开始缩放动画
     */
    private void startScaleAnimation() {

        if (mScaleAnimator == null) {
            mScaleAnimator = ValueAnimator.ofFloat(1f, scaleMax, 1f);
            mScaleAnimator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
                @Override
                public void onAnimationUpdate(ValueAnimator animation) {
                    float scale = (float) animation.getAnimatedValue();
                    // 不断改变半径的值
                    scalePointRadius = pointRadius * scale;
                    postInvalidate();
                }
            });
            mScaleAnimator.addListener(new Animator.AnimatorListener() {
                @Override
                public void onAnimationStart(Animator animation) {

                }

                @Override
                public void onAnimationEnd(Animator animation) {
                		// 动画结束后初始化回标准半径的值
                    scalePointRadius = pointRadius;
                }

                @Override
                public void onAnimationCancel(Animator animation) {

                }

                @Override
                public void onAnimationRepeat(Animator animation) {

                }
            });
            mScaleAnimator.setDuration(scaleAnimDuration);
        }
        if (mScaleAnimator.isRunning())
            mScaleAnimator.end();
        mScaleAnimator.start();
    }
```

- 同时在onDraw()方法中对刚刚触摸的点要进行绘制,更改onDraw()方法中绘制九个点的部分，对刚刚触摸的点使用缩放后的半径绘制。

```java
		// 绘制九个点，当动画在执行时被激活的点会被放大
        LockPoint tempPoint;
        for (int i = 0; i < initLockPointArray.length; i++) {
            tempPoint = initLockPointArray[i];
            // 最后触摸的点
            if (currentLockPoint != null && currentLockPoint.equals(tempPoint)) {
                canvas.drawCircle(tempPoint.x, tempPoint.y, scalePointRadius, pointPaint);
            } else {
                canvas.drawCircle(tempPoint.x, tempPoint.y, pointRadius, pointPaint);
            }
        }
```

## 9. 回调
- 使用监听将结果回调给使用者，在ACTION_UP时发布结果

```java
		public interface OnLockFinishListener {
        /**
         * 
         * @param lockView 控件
         * @param passWd 密码
         * @param passWsLength 密码长度
         * @return 当返回true时，画面将会定格在绘制结束后的状态，比如当密码输入正确的时候
         * 返回false时，画面会重新初始化回初始状态，比如密码重新二次输入确认或者密码错误的时候
         */
        boolean onFinish(LockView lockView, String passWd, int passWsLength);
    }
    
    
    /**
     * 发布绘制结果
     */
    private void publishResult() {
        if (listener != null) {
            isEventOver = true;
            StringBuilder sb = new StringBuilder();
            for (LockPoint lockPoint : historyPointList) {
                sb.append(lockPoint.index);
            }
            String passWd = sb.toString();
            boolean isFinish = listener.onFinish(LockView.this, passWd, passWd.length());
            if (isFinish) {
                // 画面定格
                touchPoint.init(currentLockPoint);
            } else {
                // 恢复初始化
                cancelLockDraw();
                isEventOver = false;
            }
        } else {
            cancelLockDraw();
        }
    }
```


## 10. 综上
- 还遗留了一个点，就是自动添加中间的点时应该也是有动画效果的，暂时还没做，有空补上吧，希望大家指正。



## 11. 完整代码

```java
/**
 * Project  : CdLibsTest
 * Package  : com.march.cdlibstest.widget
 * CreateAt : 2016/11/26
 * Describe : 自定义控件实现九宫格滑动解锁
 *
 * @author chendong
 */
public class LockView extends View {

    public static final String TAG = "LOCK_VIEW";
    private static final int INVALID_POINTER = -1;
    private static final int AUTO_START_SPACING = -1;
    private static final int DEFAULT_MIN_POINT_NUM = 4;
    // 激活的触摸点id
    private int activePointerId = INVALID_POINTER;


    // 四边的间隔，默认是控件的1／4
    private int startSpace;
    // 两点间隔
    private int internalSpace;

    // 点的半径
    private int pointRadius;
    // 动画scale的半径
    private float scalePointRadius;
    // 触摸半径，在点的一定范围内触发
    private int touchSensitiveRange;
    // 线宽度
    private int lineWidth;
    // 点颜色
    private int pointColor;
    // 线颜色
    private int lineColor;

    // 缩放的大小
    private float scaleMax;
    // 动画时间
    private int scaleAnimDuration = 150;
    // 本次绘制结束，调用init()方法恢复初始化
    private boolean isEventOver = false;


    class LockPoint {
        // 点的位置 0-8
        int index;
        //  点的x,y坐标
        float x, y;
        // 构造方法，初始化一个点
        LockPoint(int index, float x, float y) {
            this.index = index;
            this.x = x;
            this.y = y;
        }
        // 构造方法，从另一个点初始化
        LockPoint(LockPoint p) {
            this.x = p.x;
            this.y = p.y;
            this.index = p.index;
        }
        // 默认构造方法，初始化为一个空的点
        LockPoint() {
            this.x = -1;
            this.y = -1;
            this.index = -1;
        }
        // 判断该点是不是一个空的点
        boolean isEmpty() {
            return this.x == -1 && this.y == -1;
        }
        // 重新给位置赋值
        void init(float x, float y) {
            this.x = x;
            this.y = y;
        }
        // 设置为另一点的值
        void init(LockPoint p) {
            this.x = p.x;
            this.y = p.y;
            this.index = p.index;
        }
        // 判断一个位置是不是在该点触摸范围内,touchSensitiveRange为触摸有效半径
        boolean isTouchIn(float judgeX, float judgeY) {
            return judgeX < x + touchSensitiveRange &&
                    judgeX > x - touchSensitiveRange &&
                    judgeY < y + touchSensitiveRange &&
                    judgeY > y - touchSensitiveRange;
        }

        // 重写equals和hashCode
        @Override
        public boolean equals(Object o) {
            LockPoint p = (LockPoint) o;
            return p.x == x && p.y == y;
        }

        @Override
        public int hashCode() {
            return 2;
        }

        String out(String tag) {
            return tag + " : x = " + x + " , y = " + y;
        }
    }

    // 动画
    private ValueAnimator mScaleAnimator;
    // 初始化的九个点
    private LockPoint[] initLockPointArray;
    // 触摸过的点泪飙
    private List<LockPoint> historyPointList;

    // 触摸的点
    private LockPoint touchPoint;
    // 当前最后一个激活的点
    private LockPoint currentLockPoint;

    // 画线
    private Paint linePaint;
    // 画点
    private Paint pointPaint;

    // 监听
    private OnLockFinishListener listener;

    public interface OnLockFinishListener {
        /**
         *
         * @param lockView 控件
         * @param passWd 密码
         * @param passWsLength 密码长度
         * @return 当返回true时，画面将会定格在绘制结束后的状态
         * 返回false时，画面会重新初始化回初始状态
         */
        boolean onFinish(LockView lockView, String passWd, int passWsLength);
    }

    public LockView(Context context) {
        this(context, null);
    }

    public LockView(Context context, AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public LockView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        TypedArray typedArray = context.obtainStyledAttributes(attrs, R.styleable.LockView);
        float density = getResources().getDisplayMetrics().density;

        pointRadius = (int) typedArray.getDimension(R.styleable.LockView_lock_pointRadius, (8 * density));
        scalePointRadius = pointRadius;
        touchSensitiveRange = (int) typedArray.getDimension(R.styleable.LockView_lock_touchSensitiveRange, pointRadius * 3);
        startSpace = (int) typedArray.getDimension(R.styleable.LockView_lock_startSpace, AUTO_START_SPACING);
        lineWidth = (int) typedArray.getDimension(R.styleable.LockView_lock_lineWidth, (5 * density));

        lineColor = typedArray.getColor(R.styleable.LockView_lock_lineColor, Color.WHITE);
        pointColor = typedArray.getColor(R.styleable.LockView_lock_pointColor, Color.WHITE);

        scaleAnimDuration = typedArray.getInt(R.styleable.LockView_lock_scaleAnimDuration, 180);
        scaleMax = typedArray.getFloat(R.styleable.LockView_lock_scaleMax, 2.5f);
        typedArray.recycle();

        historyPointList = new ArrayList<>();
        touchPoint = new LockPoint();
        currentLockPoint = new LockPoint();

        pointPaint = new Paint();
        pointPaint.setAntiAlias(true);
        pointPaint.setColor(pointColor);
        pointPaint.setStyle(Paint.Style.FILL_AND_STROKE);

        linePaint = new Paint();
        linePaint.setAntiAlias(true);
        linePaint.setStrokeWidth(lineWidth);
        linePaint.setColor(lineColor);
        linePaint.setStyle(Paint.Style.STROKE);
    }

    public void setListener(OnLockFinishListener listener) {
        this.listener = listener;
    }


    /**
     * 开始缩放动画
     */
    private void startScaleAnimation() {

        if (mScaleAnimator == null) {
            mScaleAnimator = ValueAnimator.ofFloat(1f, scaleMax, 1f);
            mScaleAnimator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
                @Override
                public void onAnimationUpdate(ValueAnimator animation) {
                    float scale = (float) animation.getAnimatedValue();
                    scalePointRadius = pointRadius * scale;
                    postInvalidate();
                }
            });
            mScaleAnimator.addListener(new Animator.AnimatorListener() {
                @Override
                public void onAnimationStart(Animator animation) {

                }

                @Override
                public void onAnimationEnd(Animator animation) {
                    scalePointRadius = pointRadius;
                }

                @Override
                public void onAnimationCancel(Animator animation) {

                }

                @Override
                public void onAnimationRepeat(Animator animation) {

                }
            });
            mScaleAnimator.setDuration(scaleAnimDuration);
        }
        if (mScaleAnimator.isRunning())
            mScaleAnimator.end();
        mScaleAnimator.start();
    }

    // 处理触摸事件，支持多点触摸
    @Override
    public boolean onTouchEvent(MotionEvent event) {
        // fast stop
        if (!isEnabled() || isEventOver)
            return false;
        // pointerIndex 是事件的在event中的下标
        int pointerIndex;
        // 获取事件掩码
        int action = MotionEventCompat.getActionMasked(event);
        switch (action) {
            // 重新初始化触摸点
            case MotionEvent.ACTION_DOWN:
                // pointerId 记录当前激活的pointerId
                activePointerId = event.getPointerId(0);
                // 根据pointerId查找事件在event中的位置
                pointerIndex = event.findPointerIndex(activePointerId);
                // 根据位置获取到具体的事件的坐标，这里获得的坐标就是我们要记住的那个指头的坐标
                touchPoint.init(event.getX(pointerIndex), event.getY(pointerIndex));
                break;
            case MotionEvent.ACTION_MOVE:
                // 手指移动时还是根据激活的pointerId获取下标index,来进行后续操作，避免事件错乱
                pointerIndex = event.findPointerIndex(activePointerId);
                // pointerIndex < 0表示手指的事件获取不到了，结束响应事件
                if (pointerIndex < 0) {
                    Log.e(TAG, "Got ACTION_MOVE event but have an invalid active pointer id.");
                    cancelLockDraw();
                    return false;
                }
                // 根据移动的位置获取坐标，初始化touchPoint的值
                touchPoint.init(event.getX(pointerIndex), event.getY(pointerIndex));
                LockPoint tempPoint;
                // 检索触摸点有没有在九个点中的某一个的触摸范围内
                for (int i = 0; i < initLockPointArray.length; i++) {
                    tempPoint = initLockPointArray[i];
                    if (!historyPointList.contains(tempPoint)
                            && tempPoint.isTouchIn(event.getX(pointerIndex), event.getY(pointerIndex))) {
                        LockPoint centerPoint = findCenterPoint(tempPoint);
                        if (!centerPoint.isEmpty()) {
                            activePoint(centerPoint);
                        }
                        activePoint(tempPoint);
                        break;
                    }
                }
                break;
            case MotionEventCompat.ACTION_POINTER_UP:
                // 多指操作中 非 最后一个手指抬起时触发ACTION_POINTER_UP，此时要获取还在屏幕上的其他手指转移事件的对象
                onSecondaryPointerUp(event);
                break;
            case MotionEvent.ACTION_UP:
                // 最后的手指抬起触发 ACTION_UP
                pointerIndex = event.findPointerIndex(activePointerId);
                if (pointerIndex < 0) {
                    Log.e(TAG, "Got ACTION_UP event but don't have an active pointer id.");
                    activePointerId = INVALID_POINTER;
                    return false;
                }
                // 发布绘制的结果，可能是监听回调之类的
                publishResult();
                // 置为-1
                activePointerId = INVALID_POINTER;
                break;
            case MotionEvent.ACTION_CANCEL:
                // 类似up
                cancelLockDraw();
                activePointerId = INVALID_POINTER;
                break;
        }
        postInvalidate();
        return true;
    }


    /**
     * 发布绘制结果
     */
    private void publishResult() {
        if (listener != null) {
            isEventOver = true;
            StringBuilder sb = new StringBuilder();
            for (LockPoint lockPoint : historyPointList) {
                sb.append(lockPoint.index);
            }
            String passWd = sb.toString();
            boolean isFinish = listener.onFinish(LockView.this, passWd, passWd.length());
            if (isFinish) {
                // 输入合法
                touchPoint.init(currentLockPoint);
            } else {
                // 输入不合法
                cancelLockDraw();
                isEventOver = false;
            }
        } else {
            cancelLockDraw();
        }
    }

    /**
     * 当一个手机抬起时，转移焦点
     *
     * @param ev 事件
     */
    private void onSecondaryPointerUp(MotionEvent ev) {
        final int pointerIndex = MotionEventCompat.getActionIndex(ev);
        final int pointerId = MotionEventCompat.getPointerId(ev, pointerIndex);
        if (pointerId == activePointerId) {
            // This was our active pointer going up. Choose a new
            // active pointer and adjust accordingly.
            final int newPointerIndex = pointerIndex == 0 ? 1 : 0;
            activePointerId = MotionEventCompat.getPointerId(ev, newPointerIndex);
        }
    }

    /**
     * 检测当前激活的点和上一个激活点之间的是否有没有激发的点
     *
     * @param activePoint 当前被激发的点
     * @return 当前激活的点和上一个激活点之间的是否有没有激发的点，没有返回empty的{@link LockPoint#isEmpty()}
     */
    private LockPoint findCenterPoint(LockPoint activePoint) {
        LockPoint rstPoint = new LockPoint();
        // 只有一个点不需要比较
        if (historyPointList.size() < 1) {
            return rstPoint;
        }
        LockPoint tempPoint;
        // 获取上个点
        LockPoint preActivePoint = historyPointList.get(historyPointList.size() - 1);
        // 两个点是不是相邻的，是相邻的是坚决不会中间有点被空出来的
        if (isAdjacentPoint(preActivePoint, activePoint))
            return rstPoint;

        for (int i = 0; i < initLockPointArray.length; i++) {
            tempPoint = initLockPointArray[i];
            // 没有被触摸过 && 不是首点 && 不是尾点
            if (!historyPointList.contains(tempPoint) && !preActivePoint.equals(tempPoint) && !activePoint.equals(tempPoint)) {
                if (isInLine(preActivePoint, activePoint, tempPoint)) {
                    Log.e(TAG, "点在线上 " + tempPoint.out("temp") + " " + preActivePoint.out("pre") + " " + activePoint.out("active"));
                    rstPoint.init(tempPoint);
                    break;
                }
            }
        }
        return rstPoint;
    }


    /**
     * 检测相邻
     *
     * @param p1 点1
     * @param p2 点2
     * @return p1和p2是否相邻，斜对角也算相邻
     */
    private boolean isAdjacentPoint(LockPoint p1, LockPoint p2) {
        if ((p1.x == p2.x && Math.abs(p1.y - p2.y) == internalSpace)
                || (p1.y == p2.y && Math.abs(p1.x - p2.x) == internalSpace)
                || (Math.abs(p1.x - p2.x) == internalSpace && Math.abs(p1.y - p2.y) == internalSpace)) {
            Log.e(TAG, "相邻点，不处理");
            return true;
        }
        return false;
    }


    /**
     * 判断c点是不是在p1-p2的直线上
     *
     * @param p1 起始点
     * @param p2 终止点
     * @param c  判断的点
     * @return 是否在该线上
     */
    private boolean isInLine(LockPoint p1, LockPoint p2, LockPoint c) {
        float k1 = (p1.x - p2.x) * 1f / (p1.y - p2.y);
        float k2 = (p1.x - c.x) * 1f / (p1.y - c.y);
        return k1 == k2;
    }

    /**
     * 激活该点，该点将会添加到选中点列表中，然后执行动画
     *
     * @param tempPoint 被激活的点
     */
    private void activePoint(LockPoint tempPoint) {
        historyPointList.add(new LockPoint(tempPoint));
        currentLockPoint.init(tempPoint);
        startScaleAnimation();
        postInvalidate();
    }


    public void init() {
        isEventOver = false;
        cancelLockDraw();
    }

    /**
     * 结束绘制，恢复初始状态
     */
    private void cancelLockDraw() {
        touchPoint.init(-1, -1);
        currentLockPoint.init(-1, -1);
        historyPointList.clear();
        postInvalidate();
    }


    // onMeasure之后初始化数据
    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        int size = getMeasuredWidth();
        // 将宽高设置为一样的，正方形
        setMeasuredDimension(size, size);
        // 初始化屏幕中的九个点的位置和下标
        if (initLockPointArray == null) {
            initLockPointArray = new LockPoint[9];
            // startSpace 为距离左边的距离，计算九个点的位置放在控件中间
            if (startSpace == AUTO_START_SPACING) {
                startSpace = size / 4;
            }
            // 计算每两个点之间的间隔
            internalSpace = (size - 2 * startSpace) / 2;
            // 初始化九个点的位置
            int index = 0;
            for (int i = 0; i < 3; i++) {
                for (int j = 0; j < 3; j++) {
                    initLockPointArray[index] = new LockPoint(index, startSpace + j * internalSpace, startSpace + i * internalSpace);
                    index++;
                }
            }
            // 为了在preview时能看到效果
            if (isInEditMode()) {
                historyPointList.addAll(Arrays.asList(initLockPointArray));
            }
        }
    }

    private void log(Object... objs) {
        StringBuilder sb = new StringBuilder();
        for (Object obj : objs) {
            sb.append(obj.toString()).append("   ");
        }
        Log.e(TAG, sb.toString());
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        // fast stop
        if (initLockPointArray == null)
            return;

        log(currentLockPoint.out("current"), touchPoint.out("touch"));

        // 画最后一个点和触摸的点之间的线
        if (currentLockPoint != null
                && currentLockPoint.x != -1 && currentLockPoint.y != -1
                && touchPoint.x != -1 && touchPoint.y != -1) {
            canvas.drawLine(currentLockPoint.x, currentLockPoint.y, touchPoint.x, touchPoint.y, linePaint);
        }

        // 绘制之前触过存储起来的的点
        if (historyPointList.size() > 0) {
            for (int i = 0; i < historyPointList.size() - 1; i++) {
                canvas.drawLine(historyPointList.get(i).x, historyPointList.get(i).y, historyPointList.get(i + 1).x, historyPointList.get(i + 1).y, linePaint);
            }
        }

        // 绘制九个点，当动画在执行时被激活的点会被放大
        LockPoint tempPoint;
        for (int i = 0; i < initLockPointArray.length; i++) {
            tempPoint = initLockPointArray[i];
            if (currentLockPoint != null && currentLockPoint.equals(tempPoint)) {
                canvas.drawCircle(tempPoint.x, tempPoint.y, scalePointRadius, pointPaint);
            } else {
                canvas.drawCircle(tempPoint.x, tempPoint.y, pointRadius, pointPaint);

            }
        }
    }
}
```