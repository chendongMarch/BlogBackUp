---
layout: post
title: DataBinding
categories:
  - Android
tags:
  - Android
  - DataBinding
abbrlink: 83ba57f4
keywords:
  - Android
  - DataBinding
date: 2017-08-28 08:51:00
---

本文记录如何使用 `Google` 官方库 `DataBinding` 进行数据绑定。

平时编译使用 `Freeline` ，但是很遗憾的是 `Freeline` 在 `DataBinding` 的支持上还有些问题，希望早些解决。

另外 `AndroidStudio` 对 `DataBinding` 的语法支持的不是很好，很多官网文档的语法在项目的编译期都是报错的，后面应该会有改进。

<!--more-->

## 开启 DataBinding

在 `AndroidStudio` 开启 `DataBinding` 十分简单，只需要在 对应的 `build.gradle` 文件中声明

```gradle
android {
    ...
    dataBinding {
        enabled = true
    }
    ...
}
```

开启之后即可使用，这里我遇到了异常如下，`clean` 项目仍不见效果，解决措施在去除下面提到的警告的前提下，由菜单 `File->Invalidate Caches/ReStart`，重启 `IDE` 即可。

```bash
Generated class list does not exist /Users/march/AndroidPro/DevKitSample/devKit/build/intermediates/data-binding-info/release/_generated.txt
```

由于我使用 `ButterKnife` 等注解类框架，遇到 `Warning`，如下：

```bash
Warning:Using incompatible plugins for the annotation processing: android-apt. This may result in an unexpected behavior.
```
这是因为 `android-apt` 的作者已经不再维护它了，解决方案是使用官方的处理器来代替他，即可去除警告信息。

```gradle
// 删除依赖路径
classpath 'com.neenbedankt.gradle.plugins:android-apt:1.8'
// 删除插件引用
apply plugin: 'com.neenbedankt.android-apt'

// 将 android-apt 替换为 annotationProcessor 
annotationProcessor rootProject.ext.support.butterknife840_processor
```

## layout 文件结构

开启的过程还挺麻烦，查到的文章里都只提到了设置 `build.gradle` 文件，遇到的问题耽误了不少时间。

使用 `DataBinding` 时，主要是将数据直接绑定到视图中，对我们来说就是 `layout.xml` 文件了，样式有所更改，最外层需要使用新的 `<layout>` 标签，`<layout>` 标签中分为了 **数据(Model)** 和 **视图(View)** 两部分，`View` 部分自然还是原来的布局文件，在后面的描述中，我都会称他们为 `xml` 的 `Model` 部分和 `View` 部分。

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout
    xmlns:android="http://schemas.android.com/apk/res/android">

    <!--数据部分-->
    <data>

        <import
            alias="User"
            type="com.march.commonlib.activity.DataBindingActivity.User"/>

        <variable
            name="name"
            type="String"/>

        <variable
            name="user"
            type="User"/>
    </data>

    <!--原来的布局文件-->
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical">

        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="@{name}"
            android:textColor="@color/black"
            android:textSize="18sp"/>
            
    </LinearLayout>
</layout>
```

## DataBindingUtil

在使用 `DataBinding` 时，我们需要一个 `ViewDataBinding` 对象，他是连接数据和视图的桥梁，`DataBindingUtil` 作为一个工具类，提供了创建 `ViewDataBinding` 的静态方法。

`setContentView()` 方法，用来替代 `Activity` 的 `setContentView()`，可以在设置视图的同时返回 `ViewDataBinding` 对象。

```java
public static <T extends ViewDataBinding> T setContentView(Activity activity, int layoutId) {
    return setContentView(activity, layoutId, sDefaultComponent);
}
```

`inflate()` 方法，用来代替 `LayoutInflater` 的 `inflate()` 方法，用来在加载一个布局时返回 `ViewDataBinding` 对象。

```java
public static <T extends ViewDataBinding> T inflate(LayoutInflater inflater, int layoutId,
        @Nullable ViewGroup parent, boolean attachToParent) {
    return inflate(inflater, layoutId, parent, attachToParent, sDefaultComponent);
}
```

关于 `ViewDataBinding` 对象，它由插件自动编译生成，类的名字取决于你的 `layout.xml` 文件的名字，例如布局文件名为 `R.layout.activity_databinding` 那么生成对应类则为 `ActivityDatabindingBinding`。



---

<font color='blue'>语法部分⬇️</font>

---

在 `xml` 文件中，我们可以在 `View` 部分使用任何在 `Model` 部分声明的数据，为此，`DataBinding` 提供了丰富而简便的语法支持。

由于 `AndroidStudio` 对 `DataBinding` 的语法支持的没有那么好，所以很多语法在编译期都是错误的，但是运行之后就可以使用，相信在不久的以后这些支持会更加全面。

## Model 部分语法

`Model` 部分被 `<data/>` 标签包含

声明一个数据，使用 `variable` 标签，`type` 声明的是类名，`java.lang` 包下面的类，可以不用写完整路径，但是其他的类，需要声明完整路径。

```xml
<variable
    name="name"
    type="String"/>
    
<variable
    name="user"
    type="com.march.commonlib.activity.DataBindingActivity.User"/>
```

导包，使用 `import` 标签， 然后在声明数据时，可以不用使用完整路径。

```xml
<import
    type="com.march.commonlib.activity.DataBindingActivity.User"/>
<variable
    name="user"
    type="User"/>
```

别名，使用 `import` 标签时，可以指定导入类的别名，如果使用了别名，则声明数据时，`type` 必须指定为别名。

```xml
<import
    alias="Alias_User"
    type="com.march.commonlib.activity.DataBindingActivity.User"/>

<variable
    name="user"
    type="Alias_User"/>
```

当我们声明 `List<T>` 和 `Map<K,V>` 等数据类型时，在 `xml` 中 `<>` 是不允许使用的， 因为我们要使用转义字符 `&lt;` 和 `&gt;` 来表示，不得不说这种写法，可读性很差，而且在编译期是标红的，但是仍然可以运行，绑定成功。

```xml
<!--List-->
<import type="java.util.List"/>
<variable
    name="list"
    type="List&lt;User&gt;"/>
    
    
<!--不导包直接使用也是可以的-->
<variable
    name="list"
    type="java.util.List&lt;User&gt;"/>
    
    
<!--Map-->
<import type="java.util.Map"/>
<variable
    name="map"
    type="Map&lt;String,String&gt;"/>
```

自定义绑定生成的类名，编译生成绑定类的名称是根据 `layout.xml` 文件的名字
来生成的，默认放置在 `包名.databinding` 包下，但是我们通常不希望使用那种很长很没有意义的类名，甚至希望自定义这个绑定类的具体的包名，使用 `<data/>` 标签的 `class` 属性来实现。

```xml
<!--指定绑定类的类名，默认放在 应用包名.databinding 包下-->
<data  class="DiyBinding"></data>

<!--指定绑定类的类名，并放置在应用包名根目录下-->
<data  class=".DiyBinding"></data>

<!--指定绑定类的类名，并指定包名-->
<data  class="com.example.DiyBinding"></data>
```

## View 部分表达式语法

数据绑定的本质其实是在 `View` 部分的 `xml` 中引用 `Model` 部分声明的数据，这样我们使用时只需要设置数据即可根据在 `xml` 中绑定的规则自动完成数据的设置，`DataBinding` 的优势在于它将这一过程使用自动编译生成的类来完成，而我们需要做的只是说明绑定的规则和设置数据即可。

在 `xml` 中绑定数据时使用 `@{expression}` 的形式，`expression` 指的是表达式，表达式中支持丰富的语法。


### 基本语法


一个简单的绑定

```xml
<TextView
    style="@style/TvStyle"
    android:text='@{desc}'/>
```

拼接字符串，在 `xml` 中可以使用 `" "` 和 `' '` 来声明值，如同多数的脚本语言一样，如果字符串中有 `" "`，那么外面就是用 `' '`，反之亦然。


```xml
<TextView
    style="@style/TvStyle"
    android:text='@{"测试字符串拼接 [ name is " + user.name + "，age is " + user.age + "]"}'/>
```

同时也支持对特殊字符的转义，比如对 `" "` 和 `' '` 的转义，来解决多层引号嵌套的问题，比如我们从 `Map` 中取值。

```xml
<TextView
    style="@style/TvStyle"
    android:text='@{"map[\"key\"] = "+map["key"]}'
    />
```

使用 `default` 关键字声明默认值，解决没有绑定数据时的默认显示和 `xml` 预览显示的问题。

```xml
<TextView
    style="@style/TvStyle"
    android:text='@{"带有默认值 - "+user.name,default="默认值"}'
    />
```

支持 `??` 操作符，表达式 `a??b`，当 `a` 为 `null` 时将返回 `b`，这个操作符是 `Java` 不支持的，他用来为值为空时提供默认值显示。

```xml
<TextView
    style="@style/TvStyle"
    android:text='@{"非空判断 - "+user.nullAble??"空值"}'
    />
```

支持简单的表达式计算，下面使用 `VU` 作为了 `View` 类的别名。

```xml
<data>
    <import
        alias="VU"
        type="android.view.View"/>
</data>
    
<TextView
    style="@style/TvStyle"
    android:text='表达式计算'
    android:visibility="@{user.age > 10?VU.VISIBLE:VU.GONE}"
    />
```

使用静态方法

```xml
<data>
  <import type="android.text.TextUtils"/>
</data>

<TextView
    style="@style/TvStyle"
    android:text='@{TextUtils.isEmpty("")?"empty":"have"}'/>
```

访问 `map` 和 `list`

```xml
<data>
    <!--List-->
    <import type="java.util.List"/>
    <variable
      name="list"
      type="List&lt;User&gt;"/>
    
    <!--map-->
    <import type="java.util.Map"/>
    <variable
       name="map"
       type="Map&lt;String,String&gt;"/>
</data>

<TextView
    style="@style/TvStyle"
    android:text='@{"list[0] = " + list[0]}'/>
    
<TextView
    style="@style/TvStyle"
    android:text='@{"map[\"key\"] = "+map["key"]}'
    />
```

支持 `Java` 中大多数的运算符，如 数学计算 `+,-,*,/,%`，使用 `+` 进行字符串拼接，逻辑运算 `&&，||`，位运算 `&,|,^`，单目运算 `+,-,!,~`，位移运算 `>>,>>>,<<,<<<`，比较运算 `==,>,<.>=,<=`，instanceOf，三目运算 `?:`，类型转换等。

```xml
<TextView
    style="@style/TvStyle"
    android:text='@{"" instanceof String + ""}'/>
<TextView
    style="@style/TvStyle"
    android:text='判断并使用不同资源'
    android:textColor="@{user.age%2==0?@color/black:@color/red}"/>
```

### 使用 android 资源

使用资源，在表达式中使用 `andorid` 资源时，稍有不同

Type（类型）|Normal Reference（普通引用）| Expression Reference（表达式引用）
:--|:--|:--
String[]|@array|	@stringArray
int[]|@array|@intArray
TypedArray|@array|@typedArray
Animator|@animator|@animator
StateListAnimator|@animator|@stateListAnimator
color int|@color|@color
ColorStateList|@color|@colorStateList

```xml
<TextView
    style="@style/TvStyle"
    android:text='判断并使用不同资源'
    android:textColor="@{user.age%2==0?@color/black:@color/red}"/>
```

### 支持 Includes

在 `DataBinding` 中不支持 `merge` 节点，但是绑定的数据可以传递到 `include` 的布局中，在 `include` 布局文件中也需要声明数据

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout>
    <data>
        <variable
            name="user"
            type="com.march.commonlib.activity.DataBindingActivity.User"/>
    </data>

    <TextView
        xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="match_parent"
        android:layout_height="50dp"
        android:orientation="vertical"
        android:text="@{user.name}"/>

</layout>
```
我们需要在宿主文件中将数据传递进去

```xml
<include layout="@layout/tv" app:user="@{user}"/>
```

## 绑定方法和监听

方法绑定，`Method Reference`，为控件绑定点击和长按事件，首先我们要先定义这些事件的处理方法，需要注意的是，方法的参数和返回值需要与 `View` 的事件匹配。

```java
public static class EventHandler {
    
    public void clickView(View view) {
        ToastUtils.show("click view");
    }
    
    public boolean longClickView(View view) {
        ToastUtils.show("long click view");
        return true;
    }
}
```
在 `xml` 中进行方法的绑定，使用 `xxx::methodName` 的形式，通常在 `xml` 中是没有 `onLongClick` 属性的，但是在 `DataBinding` 环境下支持，虽然没有高亮显示，点击之后也不能跳转到指定的方法，但是编译可以通过，运行绑定也没有问题。

```xml
<data>
    <variable
        name="handler"
        type="com.march.commonlib.activity.DataBindingActivity.EventHandler"/>
</data>

<Button
    android:id="@+id/myBtn"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:onClick="@{handler::clickView}"
    android:onLongClick="@{handler::longClickView}"
    android:text="方法绑定"/>
```

监听绑定，`Listener Reference`，在控件的被点击后，回调监听，这个和上面方法的绑定有些区别，上面的直接设置控件的事件，现在是控件被点击后回调绑定的方法。

```java
public static class EventHandler {
    
    public void func1() {
        ToastUtils.show("func1");
    }
    
    public void func2(View view, String name) {
        ToastUtils.show("func2 " + name + " " + ((Button) view).getText());
    }
    
    public void func3(View view, Context context, String name) {
        ToastUtils.show("func2 " + name + " " + ((Button) view).getText() + " " + context.getClass().getSimpleName());
    }
}
```

在 `xml` 中绑定监听，使用 `Lambda` 表达式的形式

```xml
<data>
    <variable
        name="handler"
        type="com.march.commonlib.activity.DataBindingActivity.EventHandler"/>
</data>

<Button
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:onClick="@{()->handler.func1()}"
    android:text="监听绑定1"/>
<Button
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:onClick="@{(view)->handler.func2(view,user.name)}"
    android:text="监听绑定2"
    />
```


## 绑定数据

获取到  `ViewDataBinding` 类之后就可以进行数据的设置和绑定，使用编译生成的 `ViewDataBinding` 的子类，会对绑定的数据自动生成 `getter/setter` 方法， 直接调用即可。

绑定对象时，所有的属性都必须是 `public` 或者具有 `public` 的 `getter/setter` 方法。

另外 `DataBinding` 会将所有带有 `id` 的控件获取到，并转换为驼峰命名法，可以直接使用，不需要再进行 `findViewById()`。

```java
DiyBinding binding = getBinding();
// 字符串
binding.setDesc("简单使用");
// 对象
binding.setUser(new User("chendong", 9));
// 方法和监听
binding.setHandler(new EventHandler());
// list
List<User> list = new ArrayList<>();
list.add(new User("test", 10));
binding.setList(list);
// map
Map<String, String> map = new HashMap<>();
map.put("key", "value");
binding.setMap(map);
// 获取控件
Button myBtn = binding.myBtn;
```


## 观察者对象

当数据发生变动时，我们希望 `UI` 作同步的更改。

### BaseObservable

使对象类继承 `BaseObservable` 类，在对应属性声明处或者 `getter/setter` 方法处使用 `@Bindable` 注解，此时，当属性值发生改变时，调用 `notifyPropertyChanged(BR.xxx)` 即可同步 `UI` 更改。

```java
public static class User  extends BaseObservable{
    
    @Bindable
    private String name;
    @Bindable
    public  int    age;
    public  String nullAble;
    
    public String getName() {
        return name;
    }
    
    public void setName(String name) {
        this.name = name;
        notifyPropertyChanged(BR.name);
    }
    
    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }
}
```

在点击事件中更改数据，由于 `age` 属性没添加 `setter` 方法，我们可以在外面调用更新方法。

```java
public void clickView(View view) {
    mUser.setName("newName");
}

public boolean longClickView(View view) {
    mUser.age = 1000;
    mUser.notifyPropertyChanged(BR.age);
    return true;
}
```

弊端也很明显，我们需要所有的对象都是继承 `BaseObservable`，需要手动添加注解，需要手动去调用数据更新的通知，使用 `ObservableField` 可以更方便的完成这个过程。

### ObservableField

定义对象类，并将每个属性都定义为 `ObservableField` 类型，则在属性状态发生更改时，视图会做对应更新；

`ObservableField` 是一个使用范型来包装其他数据类型的类，这样看来我们如果需要一个 `int` 类型属性时，便不得不使用 `ObservableField<Integer>`，不过 `DataBinding` 已经为基本数据类型提供了专门的包装，如 `ObservableInt` 类型，其他基本数据类型也有对应包装。

使用这种方式更新数据时确实简化了很多操作，不过所有的属性都变成了 `ObservableField` 类型，当我们进行数据交互，`json` 转化等操作时，就变得不方便了，暂时不知道是如何解决这个问题。

```java
public static class ObUser {
    public ObservableField<String> name = new ObservableField<>();
    public ObservableInt  age  = new ObservableInt();
}
```
创建 `ObUser` 类型的数据并将它绑定到视图中

```java
ObUser obUser = new ObUser();
obUser.name.set("chendong");
obUser.age.set(12);
binding.setObuser(obUser);
```
当我们更改数据的值时，会自动调用 `notifyChange();` 方法，通知所有数据更新.

```java
public void clickView(View view) {
    mObUser.name.set("newName");
}
```

### ObservableCollections

使用 `ObservableArrayList` 和 `ObservableArrayMap` 创建客观察的 `List` 和 `Map`。

在 `xml` 引用数据

```xml
<data>
    <import type="android.databinding.ObservableArrayList"/>
    <import type="android.databinding.ObservableArrayMap"/>
    <variable
        name="obArrayList"
        type="ObservableArrayList&lt;String&gt;"/>
    <variable
        name="obArrayMap"
        type="ObservableArrayMap&lt;String,String&gt;"/>
</data>

<TextView
    style="@style/TvStyle"
    android:text='@{obArrayList[0]}'/>
<TextView
    style="@style/TvStyle"
    android:text='@{obArrayMap.get("myKey")}'/>
```

创建数据并进行绑定

```java
ObservableArrayList<String> obArrayList = new ObservableArrayList<>();
obArrayList.add("item");

ObservableArrayMap<String, String> obArrayMap = new ObservableArrayMap<>();
obArrayMap.put("myKey", "myValue");

binding.setObArrayList(obArrayList);
binding.setObArrayMap(obArrayMap);
```
当更改对应数据时，视图会得到同步更新

```java
mObArrayList.add(0, "newItem");
mObArrayMap.put("myKey", "myNewValue");
```


## 列表中进行数据绑定

使用 `DataBinding` 可以大大简化列表的使用，再也不需要拿出来所有的控件挨个设置，此时需要使用 `DataBindingUtil.inflate()` 方法获取 `Binding` 对象。

定义 `ViewHolder` 类，持有 `Binding` 对象。

```java
private class BindHolder extends RecyclerView.ViewHolder {
    private ItemBindBinding mItemBindBinding;
    public BindHolder(ItemBindBinding itemBindBinding) {
        super(itemBindBinding.getRoot());
        mItemBindBinding = itemBindBinding;
    }
}
```
定义 `Adapter` 

```java
private class BindAdapter extends RecyclerView.Adapter<BindHolder> {
    ...
    
    @Override
    public BindHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        ItemBindBinding itemBindBinding = DataBindingUtil.inflate(LayoutInflater.from(mContext), R.layout.item_bind, parent, false);
        return new BindHolder(itemBindBinding);
    }
    
    @Override
    public void onBindViewHolder(BindHolder holder, int position) {
        holder.mItemBindBinding.setUser(mUserList.get(position));
    }
    
    ...
}
```

## 任意属性的 setter

在 `xml` 使用 `DataBinding` 语法进行数据绑定时，会自动查找对应的 `setter`，这一过程和命名空间没有关系，也就是说 `app:text="xx"` 和 `android:text="xx"` 是一样的，他们都会去查找这个控件中的 `setText(String text)` 方法调用，由于这一点的支持，我们可以使用很多 `xml` 文件中原先不支持的属性。

比如我们可以如下的语法，设置文字和长按监听事件

```xml
<TextView
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/tv"
    style="@style/TvStyle"
    android:layout_height="50dp"
    app:onLongClickListener="@{handler::longClickView}"
    app:text='@{user.name + "  " + user.age}'/>
```

### 简化自定义属性

通常我们自定义控件时，需要写 `declare-styleable`，这样我们才能在 `xml` 使用自定义的属性，同时还要在自定义控件来获取这些属性，整个过程很繁琐，使用 `DataBinding` 自定义属性就变得很简单了，直接写一个 `setXXX()` 的方法即可，不过缺点就是代码高亮等都不支持了。

定义一个 `TextView`，它有 `setFormatText(String text)` 和 `setMySize(int size)` 方法，这两个方法没什么意义，只是用来测试。

```java
public class MyTv extends android.support.v7.widget.AppCompatTextView {

    public MyTv(Context context) {
        super(context);
    }

    public MyTv(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    public MyTv(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
    }

    public void setFormatText(String text) {
        setText(String.format(Locale.CHINA, "FormatText = %s", text));
    }

    public void setMySize(final int mySize) {
        setOnClickListener(new OnClickListener() {
            @Override
            public void onClick(View v) {
                ToastUtils.show("设置的size= " + mySize);
            }
        });
    }
}
```
在 `xml` 中就可以像普通属性一样使用这两个 `setter` 方法对应的属性。

```xml
<com.march.commonlib.widget.MyTv
    android:id="@+id/tv"
    style="@style/TvStyle"
    android:layout_height="50dp"
    app:formatText="@{user.name}"
    app:mySize="@{user.age}"/>
```

### BindingMethods

不是所有的方法都能刚好是 `setter` 方法的格式，又或者把一个 `setter` 方法转换成 `xml` 属性变得很长很难看，`DataBinding` 提供了将 `setter` 方法和 `xml` 属性映射的方法，使用注解 `BindingMethods` 实现。

还是在自定义控件中，添加方法 `testBindingMethod()` 方法，这不是一个标准的 `setter` 方法，自然无法映射到对应的 `xml` 属性，为了能在 `xml` 中使用，需要使用 `BindingMethods` 注解，`BindingMethods` 是注解在类上的。

下面的代码意味着，对于 `TextView` 的 `my_bind_method` 属性会对应调用 `testBindingMethod()` 方法。

```java
@BindingMethods(
        @BindingMethod(type = TextView.class,
                       attribute = "my_bind_method",
                       method = "testBindingMethod")
)
public class MyTv extends android.support.v7.widget.AppCompatTextView {

    public void testBindingMethod(final String text) {
        setOnLongClickListener(new OnLongClickListener() {
            @Override
            public boolean onLongClick(View v) {
                ToastUtils.show("setTestBindingMethod = " + text);
                return true;
            }
        });
    }
}
```

### BindingAdapter

有时我们需要给已经存在的控件设置新的 `xml` 属性， 甚至更改原来属性的运作方式，此时需要使用 `@BindingAdapter` 注解，这个注解的作用是将属性和方法绑定到一起，当在对应控件中使用对应属性时，会调用该方法。

方法是静态的，如果是 `android` 原先的属性，则需要添加 `android:xxx` 命名空间，其他不需要，可以直接写属性名即可，参数大致分为以下几种类型：

- 注解为一个属性时，第一个参数是对应控件，如果另外只有一个参数，第二个参数为值；
- 注解为一个属性时，第一个参数是对应控件，如果另外有两个参数，则第二个参数为原来的值，第三个参数为新值；
- 注解为多个属性时，第一个参数是对应控件，后面的参数分别为不同属性的值，多个属性时意味着必须当这个控件同时声明了注解中所有的注解属性才会绑定到该方法；


我们可以更改已存在属性的运作方式，如下，当对 `TextView` 使用 `android:text` 属性时，会调用该方法。

```java
public class AttrAdapter {
    @BindingAdapter("android:text")
    public static void test1(TextView tv, String text) {
        tv.setText(String.format(Locale.CHINA, "test1 - %s", text));
    }
}
```
定义一个新的属性

```java
@BindingAdapter("myTest2")
public static void test2(TextView tv, String text) {
    tv.setText(String.format(Locale.CHINA, "test2 - %s", text));
}
```

同时绑定两个属性，只有当该控件同时声明这两个属性时才会绑定到该方法

```java
@BindingAdapter({"android:text", "myTest3"})
public static void test3(TextView tv, String text, String text2) {
    tv.setText(String.format(Locale.CHINA, "test3 - %s - %s", text, text2));
}

@BindingAdapter({"myTest2", "myTest3"})
public static void test4(TextView tv, String text, String text2) {
    tv.setText(String.format(Locale.CHINA, "test4 - %s - %s", text, text2));
}
```

当一个属性，三个参数时，可以获得该属性的旧值和新值。

```java
@BindingAdapter("myTest4")
public static void test5(TextView tv, String oldText, String newText) {
    tv.setText(String.format(Locale.CHINA, "test5 - %s - %s", oldText, newText));
}
```

这些方法都声明在类 `AttrAdapter` 里面，如果要在 `xml` 中使用，需要导入该类。

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">
    
    <data>
        <variable
            name="user"
            type="com.march.commonlib.activity.DataBindingActivity.User"/>

        <import type="com.march.commonlib.activity.DataBindingActivity.AttrAdapter"/>
    </data>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical">

        <TextView
            style="@style/TvStyle"
            android:text="@{user.name}"/>

        <TextView
            style="@style/TvStyle"
            app:myTest2="@{user.name}"/>

        <TextView
            style="@style/TvStyle"
            android:text="@{user.name}"
            app:myTest3="@{user.name}"/>

        <TextView
            style="@style/TvStyle"
            app:myTest2="@{user.name}"
            app:myTest3="@{user.name}"/>

        <TextView
            style="@style/TvStyle"
            app:myTest4="@{user.name}"/>
    </LinearLayout>
</layout>
```

