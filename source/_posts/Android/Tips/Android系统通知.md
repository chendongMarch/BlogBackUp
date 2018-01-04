---
layout: post
title: Android 系统通知相关
category: Android
tags:
  - Android
  - AndroidTips
keywords:
  - 系统通知
  - 通知分组
  - 大图通知
abbrlink: d1a02c42
date: 2017-10-15 00:00:00
---


系统通知就是出现在手机通知面板上的消息，点击之后可以触发相应的操作，在接入推送、通知用户下载进度等情况下使用较多，但是通常情况下我们使用的也都是最简单的文字类通知，不过随着 `Android` 新版本的发布，系统通知也变得样式丰富起来。

本文主要记录 `Android` 发送系统通知的相关内容，如发送纯文本、进度条、自定义视图、添加按钮、多行文字模式、收件箱模式、大图模式等；适配 `Android 7.0` 进行通知的分组，避免大量通知占据通知面板；最后封装一个工具类，简化发送各种类型的通知的过程。

[本文相关源码  GitHub](https://github.com/chendongMarch/DevKitSample/tree/master/devKit/src/main/java/com/march/dev/notify)

<!--more-->

## 系统通知属性配置

由于系统通知的相关配置很多，因此用一个类来管理所有的配置选项，并给大多数属性常用的默认值，来简化对通知复杂的配置操作。这样就可以不用再面对 `SDK` 中复杂的 `API`，只需要按照显示的需求来配置对象即可。

由于配置类型繁多，都在注释中体现，不再单独一个一个说了。

```java
public class NotifyModel {
    // 通常使用自动递增的 id 来发送通知
    // 但是有类似进度条这类通知，需要更新原来的通知，此时需要一个不变的 id
    private int notifyId = -1; // 通常使用

    private String title; // 标题
    private String content; // 通知的内容
    private String ticker; // 滚动显示的提示信息

    private int           smallIcon; // 通知小图标资源
    private Bitmap        largeIcon; // 设置显示的大图
    private PendingIntent activeIntent; // 点击通知激活的 intent
    private PendingIntent deleteIntent; // 删除通知时激活的 intent

    private boolean isOnGoing    = false; // false 表示可以滑动删除
    private boolean isAutoCancel = true; // true 表示点击之后自动删除
    // 设置通知优先级别，使用
    // NotificationCompat.PRIORITY_MAX
    // NotificationCompat.PRIORITY_HIGH
    // NotificationCompat.PRIORITY_DEFAULT
    // NotificationCompat.PRIORITY_MIN
    // NotificationCompat.PRIORITY_LOW
    private int     priority     = NotificationCompat.PRIORITY_MAX;
    // 设置提示的方式，响铃或者震动，使用参数
    // Notification.DEFAULT_ALL：铃声、闪光、震动均系统默认。
    // Notification.DEFAULT_SOUND：系统默认铃声。
    // Notification.DEFAULT_VIBRATE：系统默认震动。
    // Notification.DEFAULT_LIGHTS：系统默认闪光。
    private int     remindMode   = NotificationCompat.DEFAULT_VIBRATE | Notification.DEFAULT_SOUND;
    private long[]  vibrate      = null;  // 自定义振动频率

    private int     currentProgress; // 当前进度
    private int     totalProgress; // 总共进度
    private boolean indeterminate; // 是否精确进度，false 为精确进度

    private List<NotificationCompat.Action> actions; // 按钮列表，最多支持 3 个，多了不显示
    private RemoteViews                     remoteViews; // 自定义视图
    private NotificationCompat.Style        notifyStyle; // 通知的风格，收件箱、大图、长文本等

    private String notifyGroup            = DevKit.getContext().getPackageName(); // 分组标记，唯一即可
    private int    summaryIfMoreThanCount = Integer.MAX_VALUE; // 超多多少条就分组显示
}
```

##  发送通知

发送通知需要首先需要使用 `NotificationCompat.Builder` 来创建一个 `Notification`，然后借助 `NotificationManager` 来发送，根据 `NotifyModel` 中的对通知的配置来构建 `Notification`，至于创建哪种类型的通知也是根据 `NotifyModel` 中的参数来，后面会再详细说每种类型关键代码。

```java
//  根据初始化 NotifyModel 构建通知
private static NotificationCompat.Builder initNotificationBuilder(Context context, NotifyModel model) {
    NotificationCompat.Builder builder = new NotificationCompat.Builder(context.getApplicationContext());
    // 设置小图标,这是必须的用于在通知栏进行显示，默认使用了软件的图标
    builder.setSmallIcon(model.getSmallIcon());
    // 设置显示的标题
    builder.setContentTitle(model.getTitle());
    // 设置内容
    builder.setContentText(model.getContent());
    // 设置滚动显示的提示信息
    if (!TextUtils.isEmpty(model.getTicker()))
        builder.setTicker(model.getTicker());
    // 设置大图
    if (model.getLargeIcon() != null)
        builder.setLargeIcon(model.getLargeIcon());
    // 设置优先级
    builder.setPriority(model.getPriority());
    // 设置通知铃声或者震动
    builder.setDefaults(model.getRemindMode());
    // 设置不可滑动删除
    builder.setOngoing(model.isOnGoing());
    // 设置自动删除通知
    builder.setAutoCancel(model.isAutoCancel());
    // 震动
    if (model.getVibrate() != null)
        builder.setVibrate(model.getVibrate());
    // 设置自定义视图
    if (model.getRemoteViews() != null) {
        builder.setContent(model.getRemoteViews());
        // builder.setCustomHeadsUpContentView(model.getRemoteViews());
    }
    // 点击后发送事件
    if (model.getActiveIntent() != null)
        builder.setContentIntent(model.getActiveIntent());
    // 删除后发送事件
    if (model.getDeleteIntent() != null)
        builder.setDeleteIntent(model.getDeleteIntent());
    // 添加按钮及点击事件
    if (model.getActions() != null) {
        for (NotificationCompat.Action action : model.getActions()) {
            builder.addAction(action);
        }
    }
    // 设置进度
    if (model.getTotalProgress() != 0) {
        builder.setProgress(model.getTotalProgress(), model.getCurrentProgress(), model.isIndeterminate());
    }
    // 通知栏风格
    if (model.getNotifyStyle() != null) {
        builder.setStyle(model.getNotifyStyle());
    }
    // 时间，通知栏会按照这个排序
    builder.setWhen(System.currentTimeMillis());
    // 通知分组
    builder.setGroup(model.getNotifyGroup());
    return builder;
}
```

发送通知，每条通知都有一个唯一的 `notifyId`，不同的 `notifyId` 的通知将会排列在通知面板上面，相同 `notifyId` 的通知，后面的会替换前面的， 也就是说面板上面相同 `notifyId` 的通知只能有一条。

```java
public static int notifyNow(Context context, NotifyModel model) {
    Notification notification = initNotificationBuilder(context, model).build();
    // 获取通知管理者，并获得系统通知 的服务
    NotificationManager manager = (NotificationManager) context.getSystemService(Context.NOTIFICATION_SERVICE);
    int notifyId;
    if (model.getNotifyId() != -1)
        notifyId = model.getNotifyId();
    else notifyId = ++sNotificationId;
    manager.notify(notifyId, notification);
    // 兼容 7.0 通知分组
    updateNotificationSummary(context, model);
    return notifyId;
}
```

## 简单的文本通知

这也就是我们最常用的通知类型了，因为大多数的配置都有默认值，如果需要发送一条文本通知只要配置几个关键的属性即可。

```java
NotifyModel notifyModel = new NotifyModel();
Intent intent = new Intent(mContext, HomeActivity.class);
PendingIntent activeIntent = PendingIntent.getActivity(mContext, 100, intent, PendingIntent.FLAG_CANCEL_CURRENT);
notifyModel.setActiveIntent(activeIntent);
notifyModel.setSmallIcon(R.drawable.ic_launcher);
notifyModel.setTitle("测试推送标题");
notifyModel.setTicker("测试推送track");
notifyModel.setContent("测试推送内容");
NotifySender.notifyNow(mActivity, notifyModel);
```

## 进度条通知

进度条的通知同样使用最基本的 `API` 就能实现，这里为了简化，借助 `NotifyModel` 来管理相关参数。

由于进度条类型的通知不能每次更新进度都出现新的通知，因此将 `notifyId` 单独设置成不变的，不然每次发送新的进度都会有一条。

```java
NotifyModel notifyModel = new NotifyModel();
// 初始化类似简单文本通知一些通用基本属性
// ...

progress += 10;
notifyModel.setProgress(0x123, progress, 100, false);

NotifySender.notifyNow(mActivity, notifyModel);
```
在构建通知时，需要设置的有当前进度，进度最大值，是不是精确进度等参数，这些都保存在 `NotifyModel` 中。

```java
// 设置进度
if (model.getTotalProgress() != 0) {
    builder.setProgress(model.getTotalProgress(), model.getCurrentProgress(), model.isIndeterminate());
}
```

## 带按钮的通知

有时候我们需要在进度条上面添加按钮及其相应的触发事件，实现这种效果只需要借助 `NotificationCompat.Action` 即可，`Builder` 可以接受一个 `Action` 的列表，也就代表了多个按钮，测试过程中发现最大支持 3 个按钮。

下面我添加了 4 个按钮，实际上只能显示 3 个，另外需要注意的是创建按钮点击之后触发的 `PendingIntent` 时，`request code` 必须是不同的，否则将失去点击效果。

按钮点击之后，通知并不能自动消除掉，需要后续自己来处理。

```java
NotifyModel notifyModel = new NotifyModel();
// 初始化类似简单文本通知一些通用基本属性
// ...

notifyModel.addAction(new NotificationCompat.Action(R.drawable.ic_launcher, "1", 
        PendingIntent.getActivity(mContext, 101, intent, PendingIntent.FLAG_CANCEL_CURRENT))));
notifyModel.addAction(new NotificationCompat.Action(R.drawable.ic_launcher, "2",
        PendingIntent.getActivity(mContext, 102, intent, PendingIntent.FLAG_CANCEL_CURRENT)));
notifyModel.addAction(new NotificationCompat.Action(R.drawable.ic_launcher, "3",
        PendingIntent.getActivity(mContext, 103, intent, PendingIntent.FLAG_CANCEL_CURRENT)));
notifyModel.addAction(new NotificationCompat.Action(R.drawable.ic_launcher, "4",
        PendingIntent.getActivity(mContext, 104, intent, PendingIntent.FLAG_CANCEL_CURRENT)));
        
NotifySender.notifyNow(mActivity, notifyModel);
```

构建通知时

```java
// 添加按钮及点击事件
if (model.getActions() != null) {
    for (NotificationCompat.Action action : model.getActions()) {
        builder.addAction(action);
    }
}
```

## 自定义视图通知

有时仅仅添加按钮并不能满足我们的要求，需要自定义通知的 `UI` 显示，这一功能需要借助 `RemoteViews` 来实现。

`RemoteViews` 可以加载一个布局文件，不过里面控件相关的设置，都需要借助 `RemoteViews` 的 `API` 来实现，不过好在和控件的属性还是十分类似的。

```java
NotifyModel notifyModel = new NotifyModel();
// 初始化类似简单文本通知一些通用基本属性
// ...

RemoteViews remoteViews = new RemoteViews(mContext.getPackageName(), R.layout.item);
// 设置文字
remoteViews.setTextViewText(R.id.tv, "test");
// 设置图片
remoteViews.setImageViewResource(R.id.iv, R.drawable.ic_launcher);
// 点击事件
remoteViews.setOnClickPendingIntent(R.id.iv,
        PendingIntent.getActivity(mContext, 104, intent, PendingIntent.FLAG_CANCEL_CURRENT));
        
notifyModel.setRemoteViews(remoteViews);

NotifySender.notifyNow(mActivity, notifyModel);
```

在构建通知时，只需要将 `RemoteViews` 作为通知的内容设置进去

```java
// 设置自定义视图
if (model.getRemoteViews() != null) {
    builder.setContent(model.getRemoteViews());
    // builder.setCustomHeadsUpContentView(model.getRemoteViews());
}
```

---

接下来的 3 种展示方式，需要我们关注 `NotificationCompat.Style` 这个类，它确定了通知的风格，比如多行文字风格、收件箱风格、大图片风格，设置方式也很简单只要

```java
// 通知栏风格
if (model.getNotifyStyle() != null) {
    builder.setStyle(model.getNotifyStyle());
}
```

## 多行文字风格通知

通常我们的通知只能显示一行，多了以后会隐藏显示，多行文字可以将所有文字显示出来，为了使用的方便，我将设置类型的方法隐藏在了 `NotifyModel` 类中。

```java
// NotifyModel.java
public void setBigTextStyle() {
    notifyStyle = new NotificationCompat.BigTextStyle().bigText(content);
}
```
因此当发送通知时只需要 `setBigTextStyle()` 即可。

```java
NotifyModel notifyModel = new NotifyModel();
// 初始化类似简单文本通知一些通用基本属性
// ...

notifyModel.setBigTextStyle();

NotifySender.notifyNow(mActivity, notifyModel);
```

## 收件箱风格通知

收件箱风格的通知经常出现在即时通讯类的软件中，比如对方发过来几条消息。

创建 `InBoxStyle`

```java
// NotifyModel.java
public void setInBoxStyle(List<String> msgList, String msgDesc) {
    NotificationCompat.InboxStyle inboxStyle = new NotificationCompat.InboxStyle();
    for (String msg : msgList) {
        inboxStyle.addLine(msg);
    }
    inboxStyle.setSummaryText(msgDesc);
    notifyStyle = inboxStyle;
}
```

发送通知时

```java
NotifyModel notifyModel = new NotifyModel();
// 初始化类似简单文本通知一些通用基本属性
// ...

List<String> msgList = LightConverter.listOf("1", "2", "3", "4", "5");
notifyModel.setInBoxStyle(msgList, "测试 inbox 描述");

NotifySender.notifyNow(mActivity, notifyModel);
```


## 大图风格通知

这种风格的通知我们通常会在使用手机系统截图时出现在通知面板上面，通知上面会显示一张大图片。

创建 `BigPictureStyle`

```java
// NotifyModel.java
public void setBigImageStyle(Bitmap bigImage) {
    notifyStyle = new NotificationCompat.BigPictureStyle().bigPicture(bigImage).bigLargeIcon(bigImage);
}
```

发送通知，注意不要放进去很大的图片。

```java
NotifyModel notifyModel = new NotifyModel();
// 初始化类似简单文本通知一些通用基本属性
// ...

BitmapFactory.Options options = new BitmapFactory.Options();
options.inSampleSize = 4;
String path = new File(Environment.getExternalStorageDirectory(), "1.jpg").getAbsolutePath();
Bitmap bitmap = BitmapFactory.decodeFile(path, options);
notifyModel.setBigImageStyle(bitmap);

NotifySender.notifyNow(mActivity, notifyModel);
```

## 对通知进行分组

当一个软件发送了大量通知，而用户又没有去点击处理，就会堆积在通知面板上，最后被一键清除掉，自 `Android N` 之后支持对通知进行分组，点击分组才会显示全部的通知。

在发送通知时，给通知设置一个 `Group`，这是一个唯一字符串，他用来标记这个通知是属于这个分组的。

```java
// 通知分组
if (model.getNotifyGroup() != null)
    builder.setGroup(model.getNotifyGroup());
```

实际上，所谓的分组，也是一个特殊的通知，只是它在显示在通知面板上面的同时，会将具有相同 `Group` 的通知给收集到一起，如果这个特殊的通知被 `cancel` 掉了，原来的通知又会显示出来。因此我们在每次发送通知或者清除通知之后都去需要检测一下当前通知的数量，以及需要不需要再将通知收起。

检测当前应用通知数量，因为分组通知也是一条通知，因此计算数量时要过滤掉。

```java
private static int getNumberOfNotifications(Context context) {
    if (AppUtils.isOver(Build.VERSION_CODES.M)) {
        // 查询当前展示的所有通知的状态列表
        final StatusBarNotification[] activeNotifications = getManager(context).getActiveNotifications();
        // 获取当前通知栏里头，NOTIFICATION_GROUP_SUMMARY_ID归类id的组别
        // 因为发送分组的通知也算一条通知，所以需要-1
        for (StatusBarNotification notification : activeNotifications) {
            if (notification.getId() == NOTIFICATION_GROUP_SUMMARY_ID) {
                return activeNotifications.length - 1;
            }
        }
        return activeNotifications.length;
    }
    return -1;
}
```

当发现通知数量超出限制时，我们就要额外发送一条通知，将所有通知分组收起。

```java
private static final int NOTIFICATION_GROUP_SUMMARY_ID = 1;

private static void updateNotificationSummary(Context context, NotifyModel model) {
    if (!AppUtils.isOver(Build.VERSION_CODES.N)) return;
    int numberOfNotifications = getNumberOfNotifications(context);
    boolean onlyGroup = isOnlyGroup(context);
    NotificationManager manager = getManager(context);
    // 有多条自己的通知，当数量超过要求的限制
    if (numberOfNotifications >= model.getSummaryIfMoreThanCount()) {
        NotificationCompat.Builder builder = new NotificationCompat.Builder(context);
        // 设置显示的标题
        builder.setContentTitle(context.getString(R.string.app_name));
        // 设置内容
        builder.setContentText("通知已经收起，请查看");
        builder
                .setSmallIcon(model.getSmallIcon())
                .setGroup(model.getNotifyGroup()) //设置类组key，说明此条通知归属于哪一个归类
                .setGroupSummary(true); //这句话必须和上面那句一起调用，否则不起作用
        Notification notification = builder.build();
        manager.notify(NOTIFICATION_GROUP_SUMMARY_ID, notification);
    } else if (numberOfNotifications == 1) {
        // 只有一条自己的通知
        manager.cancel(NOTIFICATION_GROUP_SUMMARY_ID);
    }
}
```


## 清除通知

```java
private static NotificationManager getManager(Context context) {
    return (NotificationManager) context.getSystemService(Context.NOTIFICATION_SERVICE);
}

// 清除指定通知
public static void clearById(Context context, int id) {
    NotificationManager manager = getManager(context);
    if (id != -1)
        manager.cancel(id);
    else
        manager.cancelAll();
}
```




## PendingIntent

这是一个类似 `Intent` 的组件，也是一种 **意图**，不同的是它可以作为通知事件的触发事件，`PendingIntent` 可以在通知事件被触发时启动一个 `Activity`，或者发送一个 `Broadcast`，又或者启动一个 `Service`，这取决于如何构造 `PendingIntent`。

比较来说，使用开启 `Activity` 的 `PendingIntent` 比较简单，点击之后立刻就可以打开界面，缺点就是当通知发送出去时，触发事件已经无法更改，举个例子，比如当用户没有登录时需要打开登录界面，反之进入指定页面；再比如，想在通知被点击时触发网络请求进行数据统计，这些情况都最好选择 `Broadcast` 的 `PendingIntent`，它使得通知的事件触发之后，发送广播到指定接受者，我们可以后续再做更多操作，这样更灵活。

开启 `Activity`

```java
Intent intent = new Intent(mContext, HomeActivity.class);
PendingIntent.getActivity(mContext, 104, intent, PendingIntent.FLAG_CANCEL_CURRENT)
```

发送 `Broadcast`

```java
class MyReceiver extends BroadcastReceiver{
    @Override
    public void onReceive(Context context, Intent intent) {
    }
}

Intent intent = new Intent(mContext,MyReceiver.class);
PendingIntent.getBroadcast(mContext, 104, intent, PendingIntent.FLAG_CANCEL_CURRENT);
```

## 总结

以上，是关于系统通知的内容，关于通知分组在 `Android 7.0` 上面的具体显示，因为没有手机测试，所以没有真的测试过，在 `6.0` 手机上可以将通知收起，但是却没办法再展开了，后面还需要更多的测试来慢慢研究这部分。




